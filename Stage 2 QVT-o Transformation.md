art 1: ROS 2 Data Metamodel (`ROS2DataModelMM`)

### 1. Create the project

1. File → New → Project
2. Select **Eclipse Modeling Framework** → ** Ecore Modeling Project**
3. Name: `ROS2DataModelMM` --> Next
4. Main package name: `ROS2DataModelMM` --> Next
5. Click **Finish**

### 2. Paste the ecore metamodel

1. `ROS2DataModelMM` → `model` → `ROS2DataModelMM.ecore`
2. Right-click `ROS2DataModelMM.ecore` → **Open With** → **Text Editor**
3. Replace the entire content with:

```xml
<?xml version='1.0' encoding='UTF-8'?>
<ecore:EPackage xmlns:ecore="http://www.eclipse.org/emf/2002/Ecore"
    xmlns:xmi="http://www.omg.org/XMI"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmi:version="2.0"
    name="ROS2DataModelMM"
    nsURI="http://www.example.org/ROS2DataModelMM"
    nsPrefix="ROS2DataModelMM">
  <eClassifiers xsi:type="ecore:EClass" name="System">
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="ID" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EString"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="Name" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EString"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="qos_reliability" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EString"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="qos_durability" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EString"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="qos_history" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EString"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="qos_depth" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EInt"/>
    <eStructuralFeatures xsi:type="ecore:EReference" name="topic" upperBound="-1" eType="#//Topic" containment="true"/>
    <eStructuralFeatures xsi:type="ecore:EReference" name="node" upperBound="-1" eType="#//Node" containment="true"/>
    <eStructuralFeatures xsi:type="ecore:EReference" name="service" upperBound="-1" eType="#//Service" containment="true"/>
    <eStructuralFeatures xsi:type="ecore:EReference" name="systemtimer" upperBound="-1" eType="#//SystemTimer" containment="true"/>
  </eClassifiers>
  <eClassifiers xsi:type="ecore:EClass" name="Topic">
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="ID" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EString"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="topic_name" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EString"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="message_type" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EString"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="interaction_type" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EString"/>
    <eStructuralFeatures xsi:type="ecore:EReference" name="publisher" upperBound="-1" eType="#//Publisher" eOpposite="#//Publisher/topic"/>
    <eStructuralFeatures xsi:type="ecore:EReference" name="subscribercallback" upperBound="-1" eType="#//SubscriberCallback" eOpposite="#//SubscriberCallback/topic"/>
  </eClassifiers>
  <eClassifiers xsi:type="ecore:EClass" name="Node">
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="ID" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EString"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="name" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EString"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="node_handle" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EDouble"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="timestamp" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EString"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="tid" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EInt"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="rmw_handle" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EDouble"/>
    <eStructuralFeatures xsi:type="ecore:EReference" name="executor" upperBound="-1" eType="#//Executor" containment="true"/>
  </eClassifiers>
  <eClassifiers xsi:type="ecore:EClass" name="SystemTimer">
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="ID" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EString"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="timer_handle" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EDouble"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="timestamp" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EDouble"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="period" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EInt"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="period_ms" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EString"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="period_us" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EString"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="tid" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EInt"/>
    <eStructuralFeatures xsi:type="ecore:EReference" name="timercallback" upperBound="-1" eType="#//TimerCallback" eOpposite="#//TimerCallback/systemtimer"/>
  </eClassifiers>
  <eClassifiers xsi:type="ecore:EClass" name="Service">
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="ID" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EString"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="service_handle" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EDouble"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="timestamp" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EString"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="node_handle" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EDouble"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="rmw_handle" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EDouble"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="service_name" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EString"/>
    <eStructuralFeatures xsi:type="ecore:EReference" name="servicerequest" eType="#//ServiceRequest" eOpposite="#//ServiceRequest/service"/>
    <eStructuralFeatures xsi:type="ecore:EReference" name="servicecallback" upperBound="-1" eType="#//ServiceCallback" eOpposite="#//ServiceCallback/service"/>
    <eStructuralFeatures xsi:type="ecore:EReference" name="clientcallback" upperBound="-1" eType="#//ClientCallback" eOpposite="#//ClientCallback/service"/>
  </eClassifiers>
  <eClassifiers xsi:type="ecore:EClass" name="Publisher">
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="ID" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EString"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="publisher_handle" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EDouble"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="timestamp" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EString"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="node_handle" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EDouble"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="rmw_handle" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EDouble"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="topic_name" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EString"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="depth" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EInt"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="qos_reliability" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EString"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="qos_durability" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EString"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="qos_history" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EString"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="qos_depth" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EInt"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="message_type" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EString"/>
    <eStructuralFeatures xsi:type="ecore:EReference" name="topic" eType="#//Topic" eOpposite="#//Topic/publisher"/>
    <eStructuralFeatures xsi:type="ecore:EReference" name="subscribercallback" eType="#//SubscriberCallback"/>
    <eStructuralFeatures xsi:type="ecore:EReference" name="timercallback" eType="#//TimerCallback"/>
    <eStructuralFeatures xsi:type="ecore:EReference" name="servicecallback" eType="#//ServiceCallback"/>
    <eStructuralFeatures xsi:type="ecore:EReference" name="clientcallback" eType="#//ClientCallback"/>
  </eClassifiers>
  <eClassifiers xsi:type="ecore:EClass" name="Executor">
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="ID" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EString"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="name" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EString"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="executor_type" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EString"/>
    <eStructuralFeatures xsi:type="ecore:EReference" name="subscribercallback" upperBound="-1" eType="#//SubscriberCallback" containment="true"/>
    <eStructuralFeatures xsi:type="ecore:EReference" name="timercallback" upperBound="-1" eType="#//TimerCallback" containment="true"/>
    <eStructuralFeatures xsi:type="ecore:EReference" name="servicecallback" upperBound="-1" eType="#//ServiceCallback" containment="true"/>
    <eStructuralFeatures xsi:type="ecore:EReference" name="clientcallback" upperBound="-1" eType="#//ClientCallback" containment="true"/>
  </eClassifiers>
  <eClassifiers xsi:type="ecore:EClass" name="TimerCallback">
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="ID" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EString"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="callback_object" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EString"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="timestamp" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EDouble"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="period" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//ELong"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="duration" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//ELong"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="wcet" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//ELong"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="bcet" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//ELong"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="jitter" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//ELong"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="intra_process" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EBoolean"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="initiallyReleased" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EBoolean"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="bufferSize" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EInt"/>
    <eStructuralFeatures xsi:type="ecore:EReference" name="systemtimer" eType="#//SystemTimer" eOpposite="#//SystemTimer/timercallback"/>
    <eStructuralFeatures xsi:type="ecore:EReference" name="publisher" upperBound="-1" eType="#//Publisher" containment="true"/>
    <eStructuralFeatures xsi:type="ecore:EReference" name="executor" eType="#//Executor"/>
    <eStructuralFeatures xsi:type="ecore:EReference" name="servicerequest" upperBound="-1" eType="#//ServiceRequest" containment="true"/>
  </eClassifiers>
  <eClassifiers xsi:type="ecore:EClass" name="ServiceRequest">
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="ID" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EString"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="subscription" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EDouble"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="timestamp" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EDouble"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="subscription_handle" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EDouble"/>
    <eStructuralFeatures xsi:type="ecore:EReference" name="service" eType="#//Service" eOpposite="#//Service/servicerequest"/>
    <eStructuralFeatures xsi:type="ecore:EReference" name="clientcallback" eType="#//ClientCallback"/>
    <eStructuralFeatures xsi:type="ecore:EReference" name="servicecallback" eType="#//ServiceCallback"/>
    <eStructuralFeatures xsi:type="ecore:EReference" name="timercallback" eType="#//TimerCallback"/>
    <eStructuralFeatures xsi:type="ecore:EReference" name="subscribercallback" eType="#//SubscriberCallback"/>
  </eClassifiers>
  <eClassifiers xsi:type="ecore:EClass" name="SubscriberCallback">
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="ID" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EString"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="subscription_name" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EString"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="timestamp" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EDouble"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="node_handle" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EDouble"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="depth" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EInt"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="rmw_handle" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EDouble"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="publisher_handle" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EDouble"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="message_type" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EString"/>
    <eStructuralFeatures xsi:type="ecore:EReference" name="topic" eType="#//Topic" eOpposite="#//Topic/subscribercallback"/>
    <eStructuralFeatures xsi:type="ecore:EReference" name="publisher" upperBound="-1" eType="#//Publisher" containment="true"/>
    <eStructuralFeatures xsi:type="ecore:EReference" name="executor" eType="#//Executor"/>
    <eStructuralFeatures xsi:type="ecore:EReference" name="servicerequest" upperBound="-1" eType="#//ServiceRequest" containment="true"/>
  </eClassifiers>
  <eClassifiers xsi:type="ecore:EClass" name="ServiceCallback">
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="ID" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EString"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="callback_object" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EDouble"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="timestamp" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EString"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="duration" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EInt"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="intra_process" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EBoolean"/>
    <eStructuralFeatures xsi:type="ecore:EReference" name="service" lowerBound="1" eType="#//Service" eOpposite="#//Service/servicecallback"/>
    <eStructuralFeatures xsi:type="ecore:EReference" name="publisher" upperBound="-1" eType="#//Publisher" containment="true"/>
    <eStructuralFeatures xsi:type="ecore:EReference" name="executor" eType="#//Executor"/>
    <eStructuralFeatures xsi:type="ecore:EReference" name="servicerequest" upperBound="-1" eType="#//ServiceRequest" containment="true"/>
  </eClassifiers>
  <eClassifiers xsi:type="ecore:EClass" name="ClientCallback">
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="ID" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EString"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="client_handle" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EDouble"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="timestamp" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EString"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="node_handle" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EDouble"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="rmw_handle" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EDouble"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="service_name" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EString"/>
    <eStructuralFeatures xsi:type="ecore:EReference" name="service" lowerBound="1" eType="#//Service" eOpposite="#//Service/clientcallback"/>
    <eStructuralFeatures xsi:type="ecore:EReference" name="publisher" upperBound="-1" eType="#//Publisher" containment="true"/>
    <eStructuralFeatures xsi:type="ecore:EReference" name="executor" eType="#//Executor"/>
    <eStructuralFeatures xsi:type="ecore:EReference" name="servicerequest" upperBound="-1" eType="#//ServiceRequest" containment="true"/>
  </eClassifiers>
</ecore:EPackage>
```

### 3. Generate Java code from genmodel

1. Open `ROS2DataModelMM.genmodel`
2. Right-click the root element → **Generate Model Code**

### 4. Add the Data Model XMI (Stage 1 output)

1. Right-click `ROS2DataModelMM` → `model` → **New** → **File** → `ROS2DataModel.xmi`
2. Paste the content of `ROS2DataModel.xmi` produced by Stage 1 into this file


## Part 2: ROS 2 Verification Metamodel

### 1. Create the project

1. File → New → Project
2. Select **Eclipse Modeling Framework** → ** Ecore Modeling Project**
3. Name: `ROS2VerificationMetamodel` --> Next 
4. Main package name: `ROS2VerificationModelMM` --> Next
5. Click **Finish**

### 2. Paste the ecore metamodel

1. `ROS2VerificationMetamodel` → `model` → `ROS2VerificationModelMM.ecore`
2. Right-click → **Open With** → **Text Editor**
3. Replace entire content with:

```xml
<?xml version='1.0' encoding='UTF-8'?>
<ecore:EPackage xmlns:ecore="http://www.eclipse.org/emf/2002/Ecore"
    xmlns:xmi="http://www.omg.org/XMI"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmi:version="2.0"
    name="ROS2VerificationModelMM"
    nsURI="http://www.example.org/ROS2VerificationModelMM"
    nsPrefix="ROS2VerificationModelMM">
  <eClassifiers xsi:type="ecore:EClass" name="System">
    <eStructuralFeatures xsi:type="ecore:EReference" name="nodes" upperBound="-1" eType="#//Node" containment="true"/>
    <eStructuralFeatures xsi:type="ecore:EReference" name="executors" upperBound="-1" eType="#//Executor" containment="true"/>
    <eStructuralFeatures xsi:type="ecore:EReference" name="templates" upperBound="-1" eType="#//Template" containment="true"/>
    <eStructuralFeatures xsi:type="ecore:EReference" name="globalDeclarations" upperBound="-1" eType="#//Declaration" containment="true"/>
    <eStructuralFeatures xsi:type="ecore:EReference" name="channels" upperBound="-1" eType="#//Channel" containment="true"/>
    <eStructuralFeatures xsi:type="ecore:EReference" name="clocks" upperBound="-1" eType="#//Clock" containment="true"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="name" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EString"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="stopTime" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EInt"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="maxReleases" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EInt"/>
  </eClassifiers>
  <eClassifiers xsi:type="ecore:EClass" name="Node">
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="name" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EString"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="nodeHandle" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EString"/>
    <eStructuralFeatures xsi:type="ecore:EReference" name="executors" upperBound="-1" eType="#//Executor" eOpposite="#//Executor/node"/>
  </eClassifiers>
  <eClassifiers xsi:type="ecore:EClass" name="Executor">
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="name" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EString"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="executorType" eType="#//ExecutorType"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="version" eType="#//ExecutorVersion"/>
    <eStructuralFeatures xsi:type="ecore:EReference" name="periodicCallbacks" upperBound="-1" eType="#//PeriodicCallback" containment="true"/>
    <eStructuralFeatures xsi:type="ecore:EReference" name="subscriberCallbacks" upperBound="-1" eType="#//SubscriberCallback" containment="true"/>
    <eStructuralFeatures xsi:type="ecore:EReference" name="timerCallbacks" upperBound="-1" eType="#//TimerCallback" containment="true"/>
    <eStructuralFeatures xsi:type="ecore:EReference" name="node" eType="#//Node" eOpposite="#//Node/executors"/>
    <eStructuralFeatures xsi:type="ecore:EReference" name="template" eType="#//Template"/>
  </eClassifiers>
  <eClassifiers xsi:type="ecore:EClass" name="Callback" abstract="true">
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="id" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EString"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="type" eType="#//CallbackType"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="timestamp" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EDouble"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="bufferSize" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EInt"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="wcet" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//ELong"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="bcet" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//ELong"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="duration" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//ELong"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="jitter" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//ELong"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="qosReliability" eType="#//QoSReliability"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="qosDurability" eType="#//QoSDurability"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="qosHistory" eType="#//QoSHistory"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="qosDepth" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EInt"/>
    <eStructuralFeatures xsi:type="ecore:EReference" name="location" eType="#//Location"/>
  </eClassifiers>
  <eClassifiers xsi:type="ecore:EClass" name="PeriodicCallback" eSuperTypes="#//Callback">
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="period" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//ELong"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="initiallyReleased" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EBoolean"/>
  </eClassifiers>
  <eClassifiers xsi:type="ecore:EClass" name="TimerCallback" eSuperTypes="#//PeriodicCallback">
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="callbackObject" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EString"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="intraProcess" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EBoolean"/>
  </eClassifiers>
  <eClassifiers xsi:type="ecore:EClass" name="SubscriberCallback" eSuperTypes="#//Callback">
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="subscriptionName" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EString"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="messageType" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EString"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="nodeHandle" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EString"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="publisherHandle" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EString"/>
  </eClassifiers>
  <eClassifiers xsi:type="ecore:EEnum" name="CallbackType">
    <eLiterals name="TIMER"/>
    <eLiterals name="SUBSCRIBER" value="1"/>
    <eLiterals name="SERVICE" value="2"/>
    <eLiterals name="CLIENT" value="3"/>
  </eClassifiers>
  <eClassifiers xsi:type="ecore:EEnum" name="ExecutorType">
    <eLiterals name="SINGLE_THREADED"/>
    <eLiterals name="MULTI_THREADED" value="1"/>
  </eClassifiers>
  <eClassifiers xsi:type="ecore:EEnum" name="ExecutorVersion">
    <eLiterals name="ExV1"/>
    <eLiterals name="ExV2" value="1"/>
  </eClassifiers>
  <eClassifiers xsi:type="ecore:EEnum" name="QoSReliability">
    <eLiterals name="RELIABLE"/>
    <eLiterals name="BEST_EFFORT" value="1"/>
  </eClassifiers>
  <eClassifiers xsi:type="ecore:EEnum" name="QoSDurability">
    <eLiterals name="VOLATILE"/>
    <eLiterals name="TRANSIENT_LOCAL" value="1"/>
    <eLiterals name="TRANSIENT" value="2"/>
    <eLiterals name="PERSISTENT" value="3"/>
  </eClassifiers>
  <eClassifiers xsi:type="ecore:EEnum" name="QoSHistory">
    <eLiterals name="KEEP_LAST"/>
    <eLiterals name="KEEP_ALL" value="1"/>
  </eClassifiers>
  <eClassifiers xsi:type="ecore:EClass" name="Template">
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="name" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EString"/>
    <eStructuralFeatures xsi:type="ecore:EReference" name="locations" upperBound="-1" eType="#//Location" containment="true"/>
    <eStructuralFeatures xsi:type="ecore:EReference" name="transitions" upperBound="-1" eType="#//Transition" containment="true"/>
    <eStructuralFeatures xsi:type="ecore:EReference" name="localDeclarations" upperBound="-1" eType="#//Declaration" containment="true"/>
    <eStructuralFeatures xsi:type="ecore:EReference" name="parameters" upperBound="-1" eType="#//Variable" containment="true"/>
  </eClassifiers>
  <eClassifiers xsi:type="ecore:EClass" name="Location">
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="name" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EString"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="isInitial" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EBoolean"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="isUrgent" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EBoolean"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="isCommitted" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EBoolean"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="invariant" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EString"/>
  </eClassifiers>
  <eClassifiers xsi:type="ecore:EClass" name="Transition">
    <eStructuralFeatures xsi:type="ecore:EReference" name="source" eType="#//Location"/>
    <eStructuralFeatures xsi:type="ecore:EReference" name="target" eType="#//Location"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="guard" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EString"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="synchronization" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EString"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="update" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EString"/>
  </eClassifiers>
  <eClassifiers xsi:type="ecore:EClass" name="Clock">
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="name" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EString"/>
  </eClassifiers>
  <eClassifiers xsi:type="ecore:EClass" name="Variable">
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="name" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EString"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="type" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EString"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="initialValue" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EString"/>
  </eClassifiers>
  <eClassifiers xsi:type="ecore:EClass" name="Declaration">
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="text" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EString"/>
  </eClassifiers>
  <eClassifiers xsi:type="ecore:EClass" name="Channel">
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="name" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EString"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="isBroadcast" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EBoolean"/>
  </eClassifiers>
</ecore:EPackage>
```

### 3. Generate Java code from genmodel

1. Open `ROS2VerificationModelMM.genmodel`
2. Right-click the root element → **Generate Model Code**

### 4. Add QVT nature to the project

1. Right-click `ROS2VerificationMetamodel` → **Properties** → **Project Natures**
2. Click **Add…** → select **QVT Operational Project Nature** → OK
3. Apply and Close

A `transformations/` folder appears in the project.

### 5. Configure QVT Metamodel Mappings

1. Right-click `ROS2VerificationMetamodel` → **Properties** → **QVT Settings** → **Metamodel Mappings**
2. Click **Add** for `ROS2DataModelMM`:
   - Source model URI: `http://www.example.org/ROS2DataModelMM`
   - Target model URI: browse to `ROS2DataModelMM/model/ROS2DataModelMM.ecore`
3. Click **Add** for `ROS2VerificationModelMM`:
   - Source model URI: `http://www.example.org/ROS2VerificationModelMM`
   - Target model URI: browse to `ROS2VerificationMetamodel/model/ROS2VerificationModelMM.ecore`
4. Apply → Apply and Close

---

## Part 3: RunQVTO.java

This class runs the QVT-O transformation standalone.

1. Right-click `src` → **New** → **Class**
2. Name: `RunQVTO`, package: `ros2verificationmetamodel` → Finish
3. Replace content with:
```java
package ros2verificationmetamodel;

import java.io.File;
import java.util.Collections;

import org.eclipse.emf.common.util.BasicDiagnostic;
import org.eclipse.emf.common.util.Diagnostic;
import org.eclipse.emf.common.util.URI;
import org.eclipse.emf.ecore.EPackage;
import org.eclipse.emf.ecore.resource.Resource;
import org.eclipse.emf.ecore.resource.ResourceSet;
import org.eclipse.emf.ecore.resource.impl.ResourceSetImpl;
import org.eclipse.emf.ecore.xmi.impl.EcoreResourceFactoryImpl;
import org.eclipse.emf.ecore.xmi.impl.XMIResourceFactoryImpl;
import org.eclipse.m2m.qvt.oml.BasicModelExtent;
import org.eclipse.m2m.qvt.oml.ExecutionContextImpl;
import org.eclipse.m2m.qvt.oml.ExecutionDiagnostic;
import org.eclipse.m2m.qvt.oml.ModelExtent;
import org.eclipse.m2m.qvt.oml.TransformationExecutor;

public class RunQVTO {

    public static void main(String[] args) throws Exception {
        // Working dir must be set to ${workspace_loc:ROS2VerificationMetamodel}
        String wsRoot = new File("").getAbsoluteFile().getParent();

        String dataEcore = wsRoot + "/ROS2DataModelMM/model/ROS2DataModelMM.ecore";
        String verEcore  = wsRoot + "/ROS2VerificationMetamodel/model/ROS2VerificationModelMM.ecore";
        String inputXmi  = wsRoot + "/ROS2DataModelMM/model/ROS2DataModel.xmi";
        String qvtoFile  = wsRoot + "/ROS2VerificationMetamodel/transformations/ROS2DM2ROS2VM.qvto";
        String outputXmi = wsRoot + "/ROS2VerificationMetamodel/transformations/ROS2DataToVerification.xmi";

        Resource.Factory.Registry.INSTANCE.getExtensionToFactoryMap().put("ecore", new EcoreResourceFactoryImpl());
        Resource.Factory.Registry.INSTANCE.getExtensionToFactoryMap().put("xmi",   new XMIResourceFactoryImpl());
        Resource.Factory.Registry.INSTANCE.getExtensionToFactoryMap().put("*",     new XMIResourceFactoryImpl());

        ResourceSet rs = new ResourceSetImpl();

        Resource dataEcoreRes = rs.getResource(URI.createFileURI(dataEcore), true);
        EPackage dataPkg = (EPackage) dataEcoreRes.getContents().get(0);
        EPackage.Registry.INSTANCE.put(dataPkg.getNsURI(), dataPkg);

        Resource verEcoreRes = rs.getResource(URI.createFileURI(verEcore), true);
        EPackage verPkg = (EPackage) verEcoreRes.getContents().get(0);
        EPackage.Registry.INSTANCE.put(verPkg.getNsURI(), verPkg);

        Resource inputRes = rs.getResource(URI.createFileURI(inputXmi), true);
        ModelExtent inputExtent  = new BasicModelExtent(inputRes.getContents());
        ModelExtent outputExtent = new BasicModelExtent();

        TransformationExecutor executor = new TransformationExecutor(URI.createFileURI(qvtoFile));
        ExecutionContextImpl ctx = new ExecutionContextImpl();
        ctx.setConfigProperty("keepModeling", true);
        ExecutionDiagnostic result = executor.execute(ctx, inputExtent, outputExtent);

        if (result.getSeverity() == Diagnostic.OK) {
            ResourceSet outRs = new ResourceSetImpl();
            outRs.getResourceFactoryRegistry().getExtensionToFactoryMap().put("xmi", new XMIResourceFactoryImpl());
            Resource outRes = outRs.createResource(URI.createFileURI(outputXmi));
            outRes.getContents().addAll(outputExtent.getContents());
            outRes.save(Collections.emptyMap());
            System.out.println("OK → " + outputXmi);
        } else {
            System.err.println("FAILED:");
            for (Diagnostic d : result.getChildren()) System.err.println("  " + d.getMessage());
        }
    }
}
```
4. Open `META-INF/MANIFEST.MF` → `Require-Bundle` must include:

```
org.eclipse.m2m.qvt.oml,
org.eclipse.m2m.qvt.oml.runtime
```

---

## Part 4: QVT-O Transformation

### 1. Create the transformation file

1. Right-click `transformations/` → **New** → **File** → `ROS2DM2ROS2VM.qvto`
2. Right-click the new file → **Open With** → **Text Editor**
3. Paste:

```
modeltype ROS2DataModelMM uses 'http://www.example.org/ROS2DataModelMM';
modeltype ROS2VerificationModelMM uses 'http://www.example.org/ROS2VerificationModelMM';

transformation ROS2DataToVerification(in dataModel : ROS2DataModelMM, out verificationModel : ROS2VerificationModelMM);

main() {
    dataModel.rootObjects()[ROS2DataModelMM::System]->map transformSystem();
}

mapping ROS2DataModelMM::System::transformSystem() : ROS2VerificationModelMM::System {
    name        := self.Name;
    stopTime    := 1000000;
    maxReleases := 100;

    executors := self.node.executor->map transformExecutor();

    channels += self->map createSyncChannel();
    channels += self.topic->map createTopicChannel();

    templates += self.topic.publisher->map createPublisherTemplate();
    templates += self.node.executor.subscribercallback->map createSubscriberTemplate();
}

mapping ROS2DataModelMM::Executor::transformExecutor() : ROS2VerificationModelMM::Executor {
    name         := self.name;
    executorType := ROS2VerificationModelMM::ExecutorType::SINGLE_THREADED;
    version      := ROS2VerificationModelMM::ExecutorVersion::ExV2;

    timerCallbacks      := self.timercallback->map transformTimerCallback();
    subscriberCallbacks := self.subscribercallback->map transformSubscriberCallback();
}

mapping ROS2DataModelMM::TimerCallback::transformTimerCallback() : ROS2VerificationModelMM::TimerCallback {
    id              := self.ID;
    type            := ROS2VerificationModelMM::CallbackType::TIMER;
    timestamp       := if self.timestamp.oclIsUndefined() then 0.0 else self.timestamp endif;
    period          := if self.period.oclIsUndefined()    then 10000000 else self.period endif;
    duration        := if self.duration.oclIsUndefined()  then 17989    else self.duration endif;
    wcet            := if self.wcet.oclIsUndefined()      then 105297   else self.wcet endif;
    bcet            := if self.bcet.oclIsUndefined()      then 14079    else self.bcet endif;
    jitter          := if self.jitter.oclIsUndefined()    then 6987     else self.jitter endif;
    callbackObject  := if self.callback_object.oclIsUndefined() then '' else self.callback_object.toString() endif;
    intraProcess    := if self.intra_process.oclIsUndefined()   then false else self.intra_process endif;
    initiallyReleased := if self.initiallyReleased.oclIsUndefined() then true else self.initiallyReleased endif;
    bufferSize      := if self.bufferSize.oclIsUndefined() then 1 else self.bufferSize endif;
    qosReliability  := ROS2VerificationModelMM::QoSReliability::RELIABLE;
    qosDurability   := ROS2VerificationModelMM::QoSDurability::VOLATILE;
    qosHistory      := ROS2VerificationModelMM::QoSHistory::KEEP_LAST;
    qosDepth        := 1;
}

mapping ROS2DataModelMM::SubscriberCallback::transformSubscriberCallback() : ROS2VerificationModelMM::SubscriberCallback {
    id               := self.ID;
    type             := ROS2VerificationModelMM::CallbackType::SUBSCRIBER;
    timestamp        := if self.timestamp.oclIsUndefined() then 0.0 else self.timestamp endif;
    subscriptionName := self.subscription_name;
    messageType      := if self.message_type.oclIsUndefined() then '' else self.message_type endif;
    nodeHandle       := self.node_handle.toString();
    publisherHandle  := self.publisher_handle.toString();
    bufferSize       := if self.depth.oclIsUndefined() then 10 else self.depth endif;
    qosReliability   := ROS2VerificationModelMM::QoSReliability::RELIABLE;
    qosDurability    := ROS2VerificationModelMM::QoSDurability::VOLATILE;
    qosHistory       := ROS2VerificationModelMM::QoSHistory::KEEP_LAST;
    qosDepth         := if self.depth.oclIsUndefined() then 10 else self.depth endif;
    wcet             := 1000;
    bcet             := 500;
    duration         := 750;
    jitter           := 100;
}

-- Broadcast sync channel for global coordination
mapping ROS2DataModelMM::System::createSyncChannel() : ROS2VerificationModelMM::Channel {
    name        := 'sync_chan';
    isBroadcast := true;
}

-- One broadcast channel per topic; name convention must match Acceleo template:
--   Publisher_<name> strips 'Publisher_' and appends '_chan!' for the sync label
--   Subscriber_<name> strips 'Subscriber_' and appends '_chan?' for the sync label
mapping ROS2DataModelMM::Topic::createTopicChannel() : ROS2VerificationModelMM::Channel {
    name        := self.topic_name.replaceAll('/', '_') + '_chan';
    isBroadcast := true;
}

-- Acceleo uses template.name only; 'Publisher_' prefix is mandatory for filtering
mapping ROS2DataModelMM::Publisher::createPublisherTemplate() : ROS2VerificationModelMM::Template {
    name := 'Publisher_' + self.topic_name.replaceAll('/', '_');
}

-- Acceleo uses template.name only; 'Subscriber_' prefix is mandatory for filtering
mapping ROS2DataModelMM::SubscriberCallback::createSubscriberTemplate() : ROS2VerificationModelMM::Template {
    name := 'Subscriber_' + self.subscription_name.replaceAll('/', '_');
}
```

### 2. Run the QVT-O transformation

Run via `RunQVTO.java`

1. Right-click `RunQVTO.java` → **Run As** → **Run Configurations…**
2. Make sure that in **Arguments** → **Working Directory** → **Other** → `${workspace_loc:ROS2VerificationMetamodel}`
3. Click **Run**
