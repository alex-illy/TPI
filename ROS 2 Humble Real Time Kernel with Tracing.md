#### Scope
The scope of this documentation and is to establish a fast way to deploy an Ubuntu 22.04 ROS 2 Humble system that has Real Time Kernel capabilities and can be used to trace ROS 2 applications with kernel capabilities.

Research: 
https://documentation.ubuntu.com/real-time/latest/how-to/enable-real-time-ubuntu/

##### Install Ubuntu 22.04 with Real Time Kernel 
1) Start with a clean Ubuntu 22.04 system and make sure that is updated and upgraded. (`sudo apt update` and `sudo apt upgrade`)
2) Go to [Ubuntu Pro](https://ubuntu.com/pro) and create an account. 
3) Log in > Account > Ubuntu Pro Dashboard > Copy the Token or the command to attach a machine. 
4) On the machine, start the terminal and run the command: `sudo pro attact 'token'`
5) Install real time kernel: 
	1) `sudo add-apt-repository universe`
	2) `sudo apt update`
	3) `sudo apt install ubuntu-realtime`
6) Test it: `uname -a`



##### ROS 2 Source Installation with Tracing 
1) Set the a locale that support UTF-8: 
```bash
sudo apt update && sudo apt install locales
sudo locale-gen en_US en_US.UTF-8
sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
export LANG=en_US.UTF-8
```
Check: `locale`
2) Add the ROS 2 apt repository: `sudo apt install software-properties-common` and `sudo add-apt-repository universe`
3) Install the `ros2-apt-source` package:
```bash
sudo apt update && sudo apt install curl -y
export ROS_APT_SOURCE_VERSION=$(curl -s https://api.github.com/repos/ros-infrastructure/ros-apt-source/releases/latest | grep -F "tag_name" | awk -F\" '{print $4}')
curl -L -o /tmp/ros2-apt-source.deb "https://github.com/ros-infrastructure/ros-apt-source/releases/download/${ROS_APT_SOURCE_VERSION}/ros2-apt-source_${ROS_APT_SOURCE_VERSION}.$(. /etc/os-release && echo $VERSION_CODENAME)_all.deb" # If using Ubuntu derivates use $UBUNTU_CODENAME
sudo dpkg -i /tmp/ros2-apt-source.deb
```
4) Install development tools and ROS tools: 
```bash
sudo apt update && sudo apt install -y \
  python3-flake8-docstrings \
  python3-pip \
  python3-pytest-cov \
  ros-dev-tools
```

```bash
sudo apt install -y \
   python3-flake8-blind-except \
   python3-flake8-builtins \
   python3-flake8-class-newline \
   python3-flake8-comprehensions \
   python3-flake8-deprecated \
   python3-flake8-import-order \
   python3-flake8-quotes \
   python3-pytest-repeat \
   python3-pytest-rerunfailures
```
5) Install LTTng and babeltrace `sudo apt-get install -y lttng-tools liblttng-ust-dev lttng-modules-dkms python3-lttng python3-babeltrace babeltrace` 
6) Create a workspace: `mkdir -p ~/ros2_humble/src` and switch to it: `cd ~/ros2_humble`
7) Clone the  Humble repos: `vcs import --input https://raw.githubusercontent.com/ros2/ros2/humble/ros2.repos src`
8) Switch to the source directory: `cd src`
9) Clone the performance test and tracetools analysis repos: 
	1) `git clone https://gitlab.com/ApexAI/performance_test.git`
	2) `git clone https://github.com/ros-tracing/tracetools_analysis.git -b humble`
10) Install the dependencies using rosdep: 
	1) `sudo rosdep init`
	2) `rosdep update`
	3) switch one directory back: `cd ..` so you can install inside of `src`
	4) `rosdep install --rosdistro humble --from-paths src --ignore-src -y --skip-keys "fastcdr rti-connext-dds-6.0.1 urdfdom_headers"`
11) Build the code in the workspace:
	1) Switch to the ros2_humble directory: `cd ~/ros2_humble/`
	2) Build the entire workspace:`colcon build --symlink-install`
	3) Build tracing-specific packages with performance test configuration: `colcon build --packages-up-to ros2trace ros2run tracetools_analysis performance_test --cmake-args -DPERFORMANCE_TEST_RCLCPP_ENABLED=ON`
12) Source the setup file: 
	1) `vi .bashrc`
	2) Add: `source ~/ros2_humble/install/local_setup.bash`
	3) Source: `source .bashrc`
13) Verify if ROS 2 Works
	1) Talker in one terminal: `ros2 run demo_nodes_cpp talker`
	2) Listener in another terminal: `ros2 run  demo_nodes_py listener`
14) Verify if traces are enabled: `ros2 run tracetools status`


##### Tracing Example 
0) If you are using kernel tracing with a non-root user, make sure to add that user to the  `tracing` group
	1) Create the group if it doesn't exist: `sudo groupadd -r tracing`
	2) Add the user: `sudo usermod -aG tracing $USER`
 1) Start the LTTng session daemon: `lttng-sessiond --daemonize`
2) In one terminal, switch to ros2 env: `cd ros2_humble`and  trace `perf_test`:      `ros2 trace --session-name perf-test --kernel --list`
3) In another terminal, run `performance_test` as root for real-time priorities from : `sudo ./install/performance_test/lib/performance_test/perf_test -c rclcpp-single-threaded-executor -p 1 -s 1 -r 0 -m Array1m --reliability RELIABLE --max-runtime 60 --use-rt-prio 98` If you get an error involving shared libraries, run the command like this: `sudo env PATH="$PATH" LD_LIBRARY_PATH="$LD_LIBRARY_PATH" ./install/performance_test/lib/performance_test/perf_test -c rclcpp-single-threaded-executor -p 1 -s 1 -r 0 -m Array1m --reliability RELIABLE --max-runtime 60 --use-rt-prio 98`
4) Validate the trace with babeltrace: `babeltrace ~/.ros/tracing/perf-test | less`


