###  Tracing Script
1) create `cbg_ping_pong.sh` file and paste: 
```bash
#!/bin/bash
# cbg_ping_pong.sh — LTTng tracing for CBG Executor Ping Pong

set -euo pipefail

# ── Prerequisite: root lttng-sessiond must be running ────────────────────────
if ! pgrep -x lttng-sessiond > /dev/null 2>&1; then
    echo "ERROR: lttng-sessiond not running." >&2
    echo "Start it: sudo lttng-sessiond --daemonize" >&2
    exit 1
fi

# ── Session — timestamped to preserve each run ────────────────────────────────
SESSION="ping_pong_trace_$(date +%Y%m%d_%H%M%S)"
OUTPUT="/home/illy/traces/$SESSION"
mkdir -p /home/illy/traces
echo "Creating session: $SESSION → $OUTPUT"
sudo lttng create "$SESSION" --output="$OUTPUT"
# Destroy the session automatically if any subsequent command fails
trap "sudo lttng destroy '$SESSION' 2>/dev/null || true" ERR

# ── Userspace: ROS 2 tracepoints ──────────────────────────────────────────────
sudo lttng enable-event --userspace 'ros2:*'
sudo lttng enable-event --userspace 'rclcpp:*'

# Thread + process name on UST events — links UST to kernel sched events by tid
# Note: UST uses procname (not vprocname — vprocname does not exist in LTTng)
sudo lttng add-context --userspace --type=vtid
sudo lttng add-context --userspace --type=procname

# ── Kernel: scheduler + hrtimer ───────────────────────────────────────────────
sudo lttng enable-event --kernel sched_switch,sched_wakeup
# sched_waking fires just before sched_wakeup — useful for RT scheduling latency;
# not available on all kernel builds, so enable conditionally
if sudo lttng enable-event --kernel sched_waking 2>/dev/null; then
    echo "sched_waking enabled"
else
    echo "WARN: sched_waking not available on this kernel — skipping"
fi
sudo lttng enable-event --kernel timer_hrtimer_start,timer_hrtimer_expire_entry,timer_hrtimer_cancel

# Thread context on kernel events — correlates to UST vtid
# cpu_id is recorded automatically in every kernel event stream; no add-context needed
sudo lttng add-context --kernel --type=tid
sudo lttng add-context --kernel --type=procname

sudo lttng start

echo ""
echo "Tracing active. Now run the ping_pong example (separate terminal)."
echo ""
echo "Stop tracing:"
echo "  sudo lttng stop && sudo lttng destroy $SESSION"
echo ""
echo "Fix ownership (if running ping_pong as root):"
echo "  sudo chown -R illy:illy $OUTPUT"
```
2) make it executable: `chmod +x cbg_ping_pong.sh`

### Optional: CPU Isolation (Cleaner Timing)

CBG_Executor deliberately splits callback groups across threads. Pinning those threads to dedicated CPUs removes scheduler noise from the `sched_switch` events, giving cleaner timing data for the UPPAAL model.

#### Step 1 — Isolate CPUs at boot

Add `isolcpus=2,3` to kernel boot parameters:

```bash
sudo vi /etc/default/grub
# Find: GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"
# Change to: GRUB_CMDLINE_LINUX_DEFAULT="quiet splash isolcpus=2,3"
sudo update-grub
sudo reboot
```

CPUs 2 and 3 will no longer receive normal scheduler tasks. 
**Attention, this needs to be rolled back afterwards. If not, the 2 cores will not be used for normal activities and your machine will have a bottleneck in the performance.** 

#### Step 2 — Pin ping_pong to isolated CPUs

```bash
source ~/ros2_humble/install/local_setup.bash
taskset -c 2,3 chrt -f 98 ros2 run examples_rclcpp_cbg_executor ping_pong
```

High-priority callback group → CPU 2, low-priority → CPU 3 (CBG_Executor assigns these internally). The `cpu_id` context in your kernel traces will now show clean separation between the two groups.

#### Verify isolation

```bash
# Should show only your process on CPUs 2,3
ps -eo pid,psr,comm | grep -E "(PID|ping_pong)"
```


### Full Workflow: 
```bash
# Terminal 1: start root session daemon (once per boot)
sudo lttng-sessiond --daemonize

# Terminal 1: setup and start tracing
./cbg_ping_pong.sh

# Terminal 2: run the example (pinned to isolated CPUs 2,3 at SCHED_FIFO priority 98)
source ~/ros2_humble/install/local_setup.bash
taskset -c 2,3 chrt -f 98 ros2 run examples_rclcpp_cbg_executor ping_pong
# ping_pong runs until you press Ctrl+C — stop it before stopping tracing

# Terminal 1: after you Ctrl+C ping_pong in Terminal 2
sudo lttng stop && sudo lttng destroy ping_pong_trace_<timestamp>
# Replace <timestamp> with the value printed by cbg_ping_pong.sh when it started

# Validate
babeltrace2 ~/traces/ping_pong_trace_<timestamp> | less

# Feed into Stage 1
python3 ~/Desktop/test_rt/traces2xmi.py \
    ~/traces/ping_pong_trace_<timestamp> \
    ROS2DataModel.xmi
```


## Python Script 
create the script: `traces2xmi.py`: 
```python
#!/usr/bin/env python3
"""
traces2xmi.py — ROS 2 CTF traces → EMF XMI (ROS2DataModel.xmi)

Usage:
  python3 traces2xmi.py <trace_dir> <output.xmi>
"""
import argparse
import logging
import sys
from collections import defaultdict
from statistics import mean, stdev
from xml.dom import minidom
from xml.etree.ElementTree import Element, SubElement, tostring


class TraceToXMI:
    """Reads LTTng/CTF traces and writes a ROS2DataModelMM-conformant XMI file."""

    _FIELD_ALIASES = {
        'node':       ['node_handle', 'handle', 'node'],
        'publisher':  ['publisher_handle', 'handle', 'publisher'],
        'subscription': ['subscription_handle', 'handle', 'subscription'],
        'timer':      ['timer_handle', 'handle', 'timer'],
        'group':      ['group_handle', 'callback_group_handle', 'handle'],
        'executor':   ['executor_handle', 'handle', 'executor'],
        'callback':   ['callback', 'callback_handle'],
        'topic':      ['topic_name', 'topic'],
        'namespace':  ['namespace', 'ns'],
        'period':     ['period', 'timer_period'],
        'exclusive':  ['is_mutually_exclusive', 'mutually_exclusive', 'exclusive'],
        'type':       ['type_name', 'message_type', 'type'],
    }

    def __init__(self, debug=False):
        self.debug = debug
        self.log = logging.getLogger(__name__)
        self._reset_state()

    # ── public API ────────────────────────────────────────────────────────────

    def from_trace(self, trace_path: str, output_path: str) -> None:
        events = self._load_trace(trace_path)
        self.log.info(f"Loaded {len(events)} events from {trace_path}")
        model = self._build_model(events)
        root = self._model_to_xmi(model)
        self._write(root, output_path)

    # ── state ─────────────────────────────────────────────────────────────────

    def _reset_state(self):
        self._id_counter = 0
        self._cb_executor: dict = {}
        self._cb_timer: dict = {}
        self._timing: dict = {}

    def _next_id(self) -> str:
        self._id_counter += 1
        return f"_{self._id_counter}"

    # ── trace loading ─────────────────────────────────────────────────────────

    def _load_trace(self, path: str) -> list:
        try:
            import bt2
        except ImportError:
            print("ERROR: babeltrace2 not available. Install: sudo apt install babeltrace2 python3-bt2")
            sys.exit(1)

        events = []
        try:
            for msg in bt2.TraceCollectionMessageIterator(path):
                if type(msg).__name__ != '_EventMessageConst':
                    continue
                ev = msg.event
                events.append({
                    'name': self._native(ev.name),
                    'timestamp': msg.default_clock_snapshot.ns_from_origin,
                    'fields': self._native_dict(getattr(ev, 'payload_field', None) or {}),
                    'context': self._native_dict(getattr(ev, 'common_context_field', None) or {}),
                })
        except Exception as e:
            print(f"ERROR reading trace: {e}")
            sys.exit(1)
        return events

    @staticmethod
    def _native(v):
        if v is None:
            return None
        for attr in ('value', '__str__', '__int__', '__float__'):
            if hasattr(v, attr):
                return getattr(v, attr)()
        return v

    def _native_dict(self, d) -> dict:
        return {self._native(k): self._native(v) for k, v in d.items()}

    # ── field extraction ──────────────────────────────────────────────────────

    def _f(self, fields: dict, key: str):
        for name in self._FIELD_ALIASES.get(key, [key]):
            v = fields.get(name)
            if v is not None:
                return str(v) if isinstance(v, (int, float)) else v
        return None

    @staticmethod
    def _infer_msg_type(topic: str) -> str:
        t = str(topic).lower()
        if 'ping' in t or 'pong' in t:
            return 'example_interfaces/msg/String'
        if 'parameter_events' in t:
            return 'rcl_interfaces/msg/ParameterEvent'
        if 'rosout' in t:
            return 'rcl_interfaces/msg/Log'
        return 'unknown'

    # ── model building ────────────────────────────────────────────────────────

    def _empty_model(self) -> dict:
        return {
            'metadata': {'example': 'ros2_system'},
            'system_structure': {
                'nodes': {},
                'topics': defaultdict(lambda: {
                    'publishers': [], 'subscriptions': [], 'type': 'unknown'}),
                'publishers': {},
                'subscriptions': {},
                'timers': {},
                'callback_groups': defaultdict(lambda: {
                    'callbacks': [], 'mutually_exclusive': False, 'executor_handle': None}),
                'executors': defaultdict(lambda: {
                    'type': 'cbg_executor', 'callback_groups': [], 'node_handles': []}),
            },
            'execution_model': {'callback_assignments': defaultdict(list)},
            'runtime_behavior': {
                'timing_statistics': {
                    'callback_execution_times': defaultdict(list),
                    'periodic_behavior': {},
                },
            },
        }

    def _build_model(self, events: list) -> dict:
        model = self._empty_model()
        node_handles: set = set()
        cb_starts: dict = {}
        timer_periods: dict = {}
        sub_group: dict = {}

        _handlers = {
            'rcl_node_init':                self._ev_node,
            'rcl_publisher_init':           self._ev_publisher,
            'rcl_subscription_init':        self._ev_subscription,
            'rcl_timer_init':               self._ev_timer,
            'callback_group_init':          self._ev_cbg_init,
            'callback_group_add_to_executor': self._ev_cbg_assign,
            'subscription_callback_added':  self._ev_sub_cb_added,
            'timer_callback_added':         self._ev_timer_cb_added,
            'executor_init':                self._ev_executor_init,
            'callback_start':               self._ev_cb_start,
            'callback_end':                 self._ev_cb_end,
        }

        for ev in events:
            name = ev['name']
            handler = next((h for prefix, h in _handlers.items() if prefix in name), None)
            if handler:
                try:
                    handler(ev, model, node_handles, cb_starts, timer_periods, sub_group)
                except Exception as e:
                    if self.debug:
                        print(f"WARN [{name}]: {e}")

        self._post_process(model, node_handles, sub_group)
        return model

    # ── event handlers ────────────────────────────────────────────────────────

    def _ev_node(self, ev, model, node_handles, *_):
        f = ev['fields']
        h = self._f(f, 'node')
        if h:
            model['system_structure']['nodes'].setdefault(h, {
                'name': self._f(f, 'node_name') or f"node_{h}",
                'namespace': self._f(f, 'namespace') or '/',
                'handle': h,
            })
            node_handles.add(h)

    def _ev_publisher(self, ev, model, *_):
        f = ev['fields']
        pub, topic = self._f(f, 'publisher'), self._f(f, 'topic')
        if pub and topic:
            mt = self._f(f, 'type') or self._infer_msg_type(topic)
            model['system_structure']['publishers'][pub] = {
                'handle': pub,
                'node_handle': str(self._f(f, 'node') or ''),
                'topic_name': str(topic),
                'message_type': str(mt),
            }
            model['system_structure']['topics'][str(topic)]['publishers'].append(pub)
            model['system_structure']['topics'][str(topic)]['type'] = str(mt)

    def _ev_subscription(self, ev, model, *_):
        f = ev['fields']
        sub, topic = self._f(f, 'subscription'), self._f(f, 'topic')
        if sub and topic:
            mt = self._f(f, 'type') or self._infer_msg_type(topic)
            model['system_structure']['subscriptions'][sub] = {
                'handle': sub,
                'node_handle': str(self._f(f, 'node') or ''),
                'topic_name': str(topic),
                'message_type': str(mt),
            }
            model['system_structure']['topics'][str(topic)]['subscriptions'].append(sub)
            model['system_structure']['topics'][str(topic)]['type'] = str(mt)

    def _ev_timer(self, ev, model, node_handles, _, timer_periods, *args):
        f = ev['fields']
        h = self._f(f, 'timer')
        if h:
            node = self._f(f, 'node') or (list(node_handles)[0] if node_handles else None)
            period = int(self._f(f, 'period') or 0)
            model['system_structure']['timers'][h] = {
                'handle': h, 'period': period,
                'node_handle': str(node) if node else '',
            }
            if period:
                timer_periods[h] = period

    def _ev_cbg_init(self, ev, model, *_):
        f = ev['fields']
        g = self._f(f, 'group')
        if g:
            model['system_structure']['callback_groups'][g] = {
                'mutually_exclusive': bool(self._f(f, 'exclusive')),
                'callbacks': [],
                'executor_handle': None,
            }

    def _ev_executor_init(self, ev, model, *_):
        f = ev['fields']
        h = self._f(f, 'executor')
        if h:
            model['system_structure']['executors'].setdefault(h, {
                'type': str(self._f(f, 'type') or 'cbg_executor'),
                'callback_groups': [],
                'node_handles': [],
            })

    def _ev_cbg_assign(self, ev, model, *_):
        f = ev['fields']
        g, ex = self._f(f, 'group'), self._f(f, 'executor')
        if g and ex:
            model['system_structure']['executors'].setdefault(ex, {
                'type': 'cbg_executor', 'callback_groups': [], 'node_handles': []})
            model['system_structure']['callback_groups'].setdefault(g, {
                'mutually_exclusive': False, 'callbacks': [], 'executor_handle': None})
            model['system_structure']['executors'][ex]['callback_groups'].append(g)
            model['system_structure']['callback_groups'][g]['executor_handle'] = ex
            model['execution_model']['callback_assignments'][ex].append(g)

    def _ev_sub_cb_added(self, ev, model, _, __, ___, sub_group):
        f = ev['fields']
        cb, sub, g = self._f(f, 'callback'), self._f(f, 'subscription'), self._f(f, 'group')
        if cb and g:
            model['system_structure']['callback_groups'].setdefault(g, {
                'mutually_exclusive': False, 'callbacks': [], 'executor_handle': None,
            })['callbacks'].append(cb)
        if sub and g:
            sub_group[sub] = g

    def _ev_timer_cb_added(self, ev, model, _, __, timer_periods, *args):
        f = ev['fields']
        cb, timer, g = self._f(f, 'callback'), self._f(f, 'timer'), self._f(f, 'group')
        if cb and g:
            model['system_structure']['callback_groups'].setdefault(g, {
                'mutually_exclusive': False, 'callbacks': [], 'executor_handle': None,
            })['callbacks'].append(cb)
        if cb and timer and str(timer) in model['system_structure']['timers']:
            period = model['system_structure']['timers'][str(timer)]['period']
            model['runtime_behavior']['timing_statistics']['periodic_behavior'][cb] = {
                'period': period, 'timer_handle': str(timer), 'callback_handle': cb,
            }

    def _ev_cb_start(self, ev, model, _, cb_starts, *args):
        cb = self._f(ev['fields'], 'callback')
        if cb:
            cb_starts[cb] = ev['timestamp']

    def _ev_cb_end(self, ev, model, _, cb_starts, *args):
        cb = self._f(ev['fields'], 'callback')
        if cb and cb in cb_starts:
            dur = ev['timestamp'] - cb_starts.pop(cb)
            model['runtime_behavior']['timing_statistics']['callback_execution_times'][cb].append({
                'start': cb_starts.get(cb, 0), 'end': ev['timestamp'], 'duration': dur,
            })

    # ── post-processing ───────────────────────────────────────────────────────

    def _post_process(self, model, node_handles: set, sub_group: dict):
        ss = model['system_structure']

        for t in ss['timers'].values():
            if not t['node_handle'] and node_handles:
                t['node_handle'] = list(node_handles)[0]

        for g_h, g_info in ss['callback_groups'].items():
            ex_h = g_info.get('executor_handle')
            if ex_h:
                for cb in g_info['callbacks']:
                    self._cb_executor[cb] = ex_h

        for sub_h, g_h in sub_group.items():
            g_info = ss['callback_groups'].get(g_h, {})
            ex_h = g_info.get('executor_handle')
            if ex_h:
                self._cb_executor[sub_h] = ex_h

        cb_times = model['runtime_behavior']['timing_statistics']['callback_execution_times']
        for cb, execs in cb_times.items():
            durs = [e['duration'] for e in execs if e.get('duration') is not None]
            if durs:
                self._timing[cb] = {
                    'wcet': max(durs),
                    'bcet': min(durs),
                    'avg':  int(mean(durs)),
                    'jitter': int(stdev(durs)) if len(durs) > 1 else 0,
                }

        for info in model['runtime_behavior']['timing_statistics']['periodic_behavior'].values():
            cb_h = info['callback_handle']
            self._cb_timer[cb_h] = info['timer_handle']
            self._timing.setdefault(cb_h, {})['period'] = info['period']
            self._timing[cb_h]['is_periodic'] = True

        if not ss['executors'] and node_handles:
            dex = 'default_cbg_executor'
            ss['executors'][dex] = {
                'type': 'cbg_executor',
                'callback_groups': list(ss['callback_groups'].keys()),
                'node_handles': list(node_handles),
            }
            for g in ss['callback_groups']:
                ss['callback_groups'][g]['executor_handle'] = dex
            model['execution_model']['callback_assignments'][dex] = list(ss['callback_groups'])

        for k in ('topics', 'callback_groups', 'executors'):
            ss[k] = dict(ss[k])
        model['execution_model']['callback_assignments'] = dict(
            model['execution_model']['callback_assignments'])
        model['runtime_behavior']['timing_statistics']['callback_execution_times'] = dict(cb_times)

    # ── XMI generation ────────────────────────────────────────────────────────

    def _model_to_xmi(self, model: dict) -> Element:
        self._reset_state()
        ss   = model.get('system_structure', {})
        rb   = model.get('runtime_behavior', {})
        meta = model.get('metadata', {})

        self._extract_timing(rb)
        self._label_topics(ss.get('topics', {}))

        root = self._mk_root(meta)
        topics_map    = self._mk_topics(root, ss.get('topics', {}))
        nodes_map     = self._mk_nodes(root, ss.get('nodes', {}))
        self._mk_publishers(topics_map, ss.get('publishers', {}))
        timers_map    = self._mk_timers(root, ss.get('timers', {}))
        executors_map = self._mk_executors(
            nodes_map, ss.get('executors', {}), ss.get('callback_groups', {}))
        self._mk_subscriber_callbacks(
            executors_map, ss.get('subscriptions', {}), ss.get('callback_groups', {}))
        self._mk_timer_callbacks(timers_map, executors_map, rb)
        return root

    def _extract_timing(self, rb: dict):
        ts = rb.get('timing_statistics', {})
        for cb, execs in ts.get('callback_execution_times', {}).items():
            if isinstance(execs, list) and execs:
                durs = [e.get('duration', 0) for e in execs
                        if isinstance(e, dict) and e.get('duration') is not None]
                if durs:
                    self._timing.setdefault(cb, {}).update({
                        'wcet': max(durs), 'bcet': min(durs),
                        'avg':  int(mean(durs)),
                        'jitter': int(stdev(durs)) if len(durs) > 1 else 0,
                    })
        for cb_id, info in ts.get('periodic_behavior', {}).items():
            if not isinstance(info, dict):
                continue
            cb_h = info.get('callback_handle', cb_id)
            timer_h = info.get('timer_handle', '')
            if cb_h and timer_h:
                self._cb_timer[cb_h] = timer_h
            self._timing.setdefault(cb_h, {})['period'] = info.get('period', 0)
            self._timing[cb_h]['is_periodic'] = True

    @staticmethod
    def _label_topics(topics: dict):
        for name, info in topics.items():
            if 'interaction_type' not in info or not info['interaction_type']:
                n = name.lower()
                if 'ping' in n:
                    info['interaction_type'] = 'ping'
                elif 'pong' in n:
                    info['interaction_type'] = 'pong'

    def _s(self, elem: Element, key: str, value) -> None:
        elem.set(key, '' if value is None else str(value))

    def _mk_root(self, meta: dict) -> Element:
        root = Element('ROS2DataModelMM:System')
        root.set('xmi:version', '2.0')
        root.set('xmlns:xmi', 'http://www.omg.org/XMI')
        root.set('xmlns:xsi', 'http://www.w3.org/2001/XMLSchema-instance')
        root.set('xmlns:ROS2DataModelMM', 'http://www.example.org/ROS2DataModelMM')
        self._s(root, 'xmi:id', self._next_id())
        self._s(root, 'ID', 'system_1')
        self._s(root, 'Name', meta.get('example', 'ros2_system'))
        root.set('qos_reliability', 'RELIABLE')
        root.set('qos_durability', 'VOLATILE')
        root.set('qos_history', 'KEEP_LAST')
        root.set('qos_depth', '10')
        return root

    def _mk_topics(self, root: Element, topics: dict) -> dict:
        result = {}
        for name, info in topics.items():
            e = SubElement(root, 'topic')
            self._s(e, 'xmi:id', self._next_id())
            self._s(e, 'ID', name)
            self._s(e, 'topic_name', name)
            self._s(e, 'message_type', info.get('type'))
            self._s(e, 'interaction_type', info.get('interaction_type'))
            result[name] = e
        return result

    def _mk_nodes(self, root: Element, nodes: dict) -> dict:
        result = {}
        for handle, info in nodes.items():
            e = SubElement(root, 'node')
            self._s(e, 'xmi:id', self._next_id())
            self._s(e, 'ID', handle)
            self._s(e, 'name', info.get('name', ''))
            self._s(e, 'node_handle', handle)
            self._s(e, 'timestamp', '')
            self._s(e, 'tid', '0')
            self._s(e, 'rmw_handle', '0.0')
            result[handle] = e
        return result

    def _mk_publishers(self, topics_map: dict, publishers: dict) -> dict:
        result = {}
        for handle, info in publishers.items():
            topic = info.get('topic_name')
            if topic not in topics_map:
                continue
            e = SubElement(topics_map[topic], 'publisher')
            self._s(e, 'xmi:id', self._next_id())
            self._s(e, 'ID', handle)
            self._s(e, 'publisher_handle', handle)
            self._s(e, 'timestamp', '')
            self._s(e, 'node_handle', info.get('node_handle', '0'))
            self._s(e, 'rmw_handle', '0.0')
            self._s(e, 'topic_name', topic)
            self._s(e, 'depth', '10')
            self._s(e, 'qos_reliability', 'RELIABLE')
            self._s(e, 'qos_durability', 'VOLATILE')
            self._s(e, 'qos_history', 'KEEP_LAST')
            self._s(e, 'qos_depth', '10')
            self._s(e, 'message_type', info.get('message_type'))
            result[handle] = e
        return result

    def _mk_timers(self, root: Element, timers: dict) -> dict:
        result = {}
        for handle, info in timers.items():
            e = SubElement(root, 'systemtimer')
            self._s(e, 'xmi:id', self._next_id())
            self._s(e, 'ID', handle)
            self._s(e, 'timer_handle', handle)
            self._s(e, 'timestamp', '0.0')
            period = int(info.get('period', 1_000_000))
            self._s(e, 'period', str(period))
            self._s(e, 'period_ms', str(period // 1_000_000))
            self._s(e, 'period_us', str(period // 1_000))
            self._s(e, 'tid', '0')
            result[handle] = e
        return result

    def _mk_executors(self, nodes_map: dict, executors: dict,
                       callback_groups: dict) -> dict:
        result = {}
        node_handles = list(nodes_map.keys())
        if not node_handles:
            return result

        cbg_node: dict = {}
        for cbg_h, cbg_info in callback_groups.items():
            ex_h = cbg_info.get('executor_handle')
            if ex_h and ex_h not in cbg_node:
                parts = cbg_h.split('_')
                for p in reversed(parts):
                    if p in nodes_map:
                        cbg_node[ex_h] = p
                        break

        for i, (handle, info) in enumerate(executors.items()):
            node_h = cbg_node.get(handle) or node_handles[i % len(node_handles)]
            e = SubElement(nodes_map[node_h], 'executor')
            self._s(e, 'xmi:id', self._next_id())
            self._s(e, 'ID', handle)
            self._s(e, 'name', f'executor_{handle}')
            self._s(e, 'executor_type', info.get('type', 'cbg_executor'))
            result[handle] = e
        return result

    def _find_executor(self, handle: str, executors_map: dict,
                        sub_info: dict = None) -> Element:
        ex_h = self._cb_executor.get(handle)
        if ex_h and ex_h in executors_map:
            return executors_map[ex_h]
        if handle in executors_map:
            return executors_map[handle]
        if executors_map:
            return min(
                executors_map.values(),
                key=lambda e: (len(e.findall('subscribercallback')) +
                               len(e.findall('timercallback'))),
            )
        return None

    def _mk_subscriber_callbacks(self, executors_map: dict,
                                   subscriptions: dict,
                                   callback_groups: dict) -> dict:
        for g_info in callback_groups.values():
            ex_h = g_info.get('executor_handle')
            if ex_h:
                for cb in g_info.get('callbacks', []):
                    self._cb_executor.setdefault(cb, ex_h)

        result = {}
        for handle, info in subscriptions.items():
            ex_elem = self._find_executor(handle, executors_map, info)
            if ex_elem is None:
                self.log.warning(f"No executor for subscriber {handle}, skipping")
                continue
            e = SubElement(ex_elem, 'subscribercallback')
            self._s(e, 'xmi:id', self._next_id())
            self._s(e, 'ID', handle)
            self._s(e, 'subscription_name', info.get('topic_name', ''))
            self._s(e, 'timestamp', '0.0')
            self._s(e, 'node_handle', info.get('node_handle', '0'))
            self._s(e, 'depth', '10')
            self._s(e, 'rmw_handle', '0.0')
            self._s(e, 'publisher_handle', '0.0')
            self._s(e, 'message_type', info.get('message_type'))
            result[handle] = e
        return result

    def _mk_timer_callbacks(self, timers_map: dict, executors_map: dict,
                              rb: dict) -> dict:
        result = {}
        cb_times = rb.get('timing_statistics', {}).get('callback_execution_times', {})
        for cb_h, execs in cb_times.items():
            tc = self._timing.get(cb_h, {})
            if not tc.get('is_periodic'):
                continue
            timer_h = self._cb_timer.get(cb_h)
            timer_elem = timers_map.get(timer_h)
            exec_elem = self._find_executor(cb_h, executors_map)
            if timer_elem is None or exec_elem is None:
                self.log.warning(f"Cannot place timer callback {cb_h}, skipping")
                continue

            e = SubElement(exec_elem, 'timercallback')
            self._s(e, 'xmi:id', self._next_id())
            self._s(e, 'ID', cb_h)
            self._s(e, 'callback_object', str(float(cb_h)))

            if execs:
                durs = [x.get('duration', 0) for x in execs if isinstance(x, dict)]
                self._s(e, 'timestamp', str(execs[0].get('start', 0)))
                self._s(e, 'duration', str(int(mean(durs))) if durs else '0')
            else:
                self._s(e, 'timestamp', '0')
                self._s(e, 'duration', '0')

            self._s(e, 'wcet', str(tc.get('wcet', 0)))
            self._s(e, 'bcet', str(tc.get('bcet', 0)))
            self._s(e, 'jitter', str(tc.get('jitter', 0)))
            self._s(e, 'period', str(tc.get('period', 0)))
            self._s(e, 'intra_process', 'false')
            result[cb_h] = e
        return result

    # ── output ────────────────────────────────────────────────────────────────

    @staticmethod
    def _write(root: Element, path: str) -> None:
        pretty = minidom.parseString(tostring(root, 'unicode')).toprettyxml(indent='  ')
        with open(path, 'w', encoding='utf-8') as f:
            f.write(pretty)
        print(f"XMI written: {path}")


# ── CLI ───────────────────────────────────────────────────────────────────────

def main():
    ap = argparse.ArgumentParser(
        description='Convert ROS 2 CTF traces to EMF XMI (ROS2DataModel.xmi)')
    ap.add_argument('input',  help='CTF trace directory')
    ap.add_argument('output', help='Output XMI file path')
    ap.add_argument('--debug', action='store_true')
    args = ap.parse_args()

    logging.basicConfig(
        level=logging.DEBUG if args.debug else logging.INFO,
        format='%(levelname)s %(message)s')

    TraceToXMI(debug=args.debug).from_trace(args.input, args.output)


if __name__ == '__main__':
    main()

```

Run the script: `python3 traces2xmi.py /path/to/ctf_trace_dir/ ROS2DataModel.xmi`

