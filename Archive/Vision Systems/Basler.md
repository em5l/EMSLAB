---
layout: default
title: "Basler"
nav_order: 8
grand_parent: "Archive"
parent: "Vision Systems"
---

## Using Harvesters with Pylon – Basic Tutorial

This tutorial explains how to use the **Harvesters** Python library with **Basler Pylon cameras** and access camera nodes.

### 1. Install Dependencies

Make sure you have **Pylon SDK** installed and the `harvesters` Python package. Also having opencv is important.

### 2. Tryout example code

This code will start your camera make sure the location of the cti file is correct.

<div class="code-example" markdown="1">
```
from harvesters.core import Harvester
import cv2
import numpy as np

# Initialize Harvester and add the CTI file
h = Harvester()
h.add_file(r"C:\Program Files\Basler\pylon\Runtime\x64\ProducerU3V.cti")
h.update()  # Discover cameras

# Create an image acquirer (first detected camera)
ia = h.create(0)

try:
    # Start acquisition in a background thread
    # Limit the queue to 1 buffer by allowing frame drop
    ia.start(run_as_thread=True)

    print("Press 'q' to quit.")
    while True:
        with ia.fetch() as buffer:
            # Convert raw data to a 2D NumPy array (grayscale)
            component = buffer.payload.components[0]
            img = component.data.reshape(component.height, component.width)

            # Display the live camera feed
            cv2.imshow("Live Camera View", img)

            if cv2.waitKey(1) & 0xFF == ord('q'):
                break

finally:
    ia.stop()         # Stop acquisition
    ia.destroy()      # Release resources
    h.reset()         # Reset Harvester
    cv2.destroyAllWindows()
```
</div>

### 3. Get nodes and Change values.

 This code provide all of the nodes associated with the camera and changes their values.

<div class="code-example" markdown="1">
```
# --- Setup Basler camera ---
"""
h = Harvester()
h.add_file(r"C:\Program Files\Basler\pylon\Runtime\x64\ProducerU3V.cti")  # or ProducerGEV.cti for GigE
h.update()

if not h.device_info_list:
    raise RuntimeError("No Basler cameras detected.")

ia = h.create(0)
nm = ia.remote_device.node_map
# -------------------------------
#ENABLE AcquisitionFrameRate
# -------------------------------
if nm.has_node("AcquisitionFrameRateEnable"):
    node_enable = nm.get_node("AcquisitionFrameRateEnable")
    try:
        node_enable.value = True  # enable frame rate control
        print("AcquisitionFrameRateEnable set to True")
    except Exception as e:
        print("Failed to enable AcquisitionFrameRate:", e)
else:
    print("AcquisitionFrameRateEnable node not available")

# -------------------------------
#Read and change AcquisitionFrameRate
# -------------------------------
if nm.has_node("AcquisitionFrameRate"):
    node_fps = nm.get_node("AcquisitionFrameRate")
    try:
        print("Current FPS:", node_fps.value)
        print("Min FPS:", node_fps.min)
        print("Max FPS:", node_fps.max)

        # Set new frame rate safely within allowed range
        desired_fps = 60
        if node_fps.min <= desired_fps <= node_fps.max:
            node_fps.value = desired_fps
            print(f"AcquisitionFrameRate set to {desired_fps} fps")
    except Exception as e:
        print("Failed to read/set AcquisitionFrameRate:", e)
else:
    print("AcquisitionFrameRate node not available")
# -------------------------------
#ENABLE DeviceLinkThroughputLimitMode
# -------------------------------
if nm.has_node("DeviceLinkThroughputLimit"):
    node_limit = nm.get_node("DeviceLinkThroughputLimit")
    print("Current throughput limit:", node_limit.value, "Bytes/s")

    # Example: limit to 80 MB/s
    desired_limit = 300_000_000
    node_limit.value = desired_limit
    print(f"DeviceLinkThroughputLimit set to {desired_limit} Bytes/s")
else:
    print("DeviceLinkThroughputLimit not available")
# -------------------------------
#Read and change DeviceLinkThroughputLimit
# -------------------------------
if nm.has_node("DeviceLinkThroughputLimit"):
    node_DeviceLinkThroughputLimit = nm.get_node("DeviceLinkThroughputLimit")
    try:
        print("Current Bandwith:", node_DeviceLinkThroughputLimit.value)
        # Set new frame rate safely within allowed range
        desired_DeviceLinkThroughputLimit = 210000000  # e.g. 70 MB/s
        node_DeviceLinkThroughputLimit.value = desired_DeviceLinkThroughputLimit
        print(f"Babdwith set to {desired_DeviceLinkThroughputLimit} ")
    except Exception as e:
        print("Failed to read/set DeviceLinkThroughputLimit:", e)
else:
    print("DeviceLinkThroughputLimit node not available")
# -------------------------------
#Read and change BalanceWhiteAuto
# -------------------------------
if nm.has_node("BalanceWhiteAuto"):
    node_enable = nm.get_node("BalanceWhiteAuto")
    try:
        node_enable.value = "Continuous"
        print("BalanceWhiteAuto set to Continuous")
    except Exception as e:
        print("Failed to enable BalanceWhiteAuto:", e)
else:
    print("BalanceWhiteAuto node not available")

# -------------------------------
#Read and change PixelFormat 
# -------------------------------
if nm.has_node("PixelFormat"):
    node_PixelFormat = nm.get_node("PixelFormat")
    try:
        node_PixelFormat.value = "Mono8"
        print("PixelFormat set to:",node_PixelFormat.value)
    except Exception as e:
        print("Failed to Set PixelFormat:", e)
else:
    print("PixelFormat node not available")

ia.start(run_as_thread=True)

```
</div>

Another example code just to get all avaible nodes.

<div class="code-example" markdown="1">
```
# --- Setup Basler camera ---
from harvesters.core import Harvester

h = Harvester()
h.add_file(r"C:\Program Files\Basler\pylon\Runtime\x64\ProducerU3V.cti")  # or ProducerGEV.cti
h.update()

if not h.device_info_list:
    raise RuntimeError("No Basler cameras detected.")

ia = h.create(0)
nm = ia.remote_device.node_map

# --- List all nodes using dir() ---
print("All available nodes:")
for node_name in dir(nm):
    if not node_name.startswith("_"):
        print(node_name)

# --- OR list nodes more formally ---
print("\nNode names and types:")
for node_name in nm.node_names:
    node = nm.get_node(node_name)
    print(f"{node_name}: {type(node)}")

```
</div>
Ouput should be like:

<div class="code-example" markdown="1">

```
['__class__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__', '__firstlineno__', '__format__', '__ge__', '__getattr__', '__getattribute__', '__getstate__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__le__', '__lt__', '__module__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__static_attributes__', '__str__', '__subclasshook__', '__weakref__', '_attributes', '_create_node_map', '_module', '_node_callback_proxy_dict', '_node_map', '_parent', '_remove_intermediate_file', '_retrieve_file_path', '_source_object', 'deregister_node_callback', 'deregister_node_callbacks', 'module', 'node_map', 'parent', 'port', 'register_event', 'register_node_callback']
<genicam.genapi.NodeMap; proxy of <Swig Object of type 'GENAPI_NAMESPACE::CNodeMapRef *' at 0x0000027013FE6880> >
['AcquisitionBurstFrameCount', 'AcquisitionControl', 'AcquisitionFrameRate', 'AcquisitionFrameRateEnable', 'AcquisitionMode', 'AcquisitionStart', 'AcquisitionStatus', 'AcquisitionStatusSelector', 'AcquisitionStop', 'AnalogControl', 'AutoExposureTimeLowerLimit', 'AutoExposureTimeUpperLimit', 'AutoFunctionControl', 'AutoFunctionProfile', 'AutoFunctionROIControl', 'AutoFunctionROIHeight', 'AutoFunctionROIOffsetX', 'AutoFunctionROIOffsetY', 'AutoFunctionROISelector', 'AutoFunctionROIUseBrightness', 'AutoFunctionROIUseWhiteBalance', 'AutoFunctionROIWidth', 'AutoGainLowerLimit', 'AutoGainUpperLimit', 'AutoTargetBrightness', 'BalanceRatio', 'BalanceRatioSelector', 'BalanceWhiteAuto', 'BinningHorizontal', 'BinningHorizontalMode', 'BinningVertical', 'BinningVerticalMode', 'BlackLevel', 'BlackLevelSelector', 'BslUSBSpeedMode', 'CenterX', 'CenterY', 'ChunkCounterSelector', 'ChunkCounterValue', 'ChunkData', 'ChunkDataControl', 'ChunkEnable', 'ChunkExposureTime', 'ChunkGain', 'ChunkGainSelector', 'ChunkLineStatusAll', 'ChunkModeActive', 'ChunkPayloadCRC16', 'ChunkSelector', 'ChunkSequencerSetActive', 'ChunkTimestamp', 'ColorAdjustmentHue', 'ColorAdjustmentSaturation', 'ColorAdjustmentSelector', 'ColorSpace', 'ColorTransformationSelector', 'ColorTransformationValue', 'ColorTransformationValueSelector', 'CounterAndTimerControl', 'CounterDuration', 'CounterEventSource', 'CounterReset', 'CounterResetActivation', 'CounterResetSource', 'CounterSelector', 'DecimationHorizontal', 'DecimationVertical', 'DemosaicingMode', 'DeviceControl', 'DeviceFirmwareVersion', 'DeviceLinkCurrentThroughput', 'DeviceLinkSelector', 'DeviceLinkSpeed', 'DeviceLinkThroughputLimit', 'DeviceLinkThroughputLimitMode', 'DeviceManufacturerInfo', 'DeviceModelName', 'DeviceReset', 'DeviceSFNCVersionMajor', 'DeviceSFNCVersionMinor', 'DeviceSFNCVersionSubMinor', 'DeviceScanType', 'DeviceSerialNumber', 'DeviceTemperature', 'DeviceTemperatureSelector', 'DeviceUserID', 'DeviceVendorName', 'DeviceVersion', 'DigitalIOControl', 'DigitalShift', 'EventControl', 'EventCriticalTemperature', 'EventCriticalTemperatureData', 'EventCriticalTemperatureTimestamp', 'EventExposureEnd', 'EventExposureEndData', 'EventExposureEndFrameID', 'EventExposureEndTimestamp', 'EventFrameBurstStart', 'EventFrameBurstStartData', 'EventFrameBurstStartFrameID', 'EventFrameBurstStartOvertrigger', 'EventFrameBurstStartOvertriggerData', 'EventFrameBurstStartOvertriggerFrameID', 'EventFrameBurstStartOvertriggerTimestamp', 'EventFrameBurstStartTimestamp', 'EventFrameBurstStartWait', 'EventFrameBurstStartWaitData', 'EventFrameBurstStartWaitTimestamp', 'EventFrameStart', 'EventFrameStartData', 'EventFrameStartFrameID', 'EventFrameStartOvertrigger', 'EventFrameStartOvertriggerData', 'EventFrameStartOvertriggerFrameID', 'EventFrameStartOvertriggerTimestamp', 'EventFrameStartTimestamp', 'EventFrameStartWait', 'EventFrameStartWaitData', 'EventFrameStartWaitTimestamp', 'EventNotification', 'EventOverTemperature', 'EventOverTemperatureData', 'EventOverTemperatureTimestamp', 'EventSelector', 'EventTest', 'EventTestData', 'EventTestTimestamp', 'ExpertFeatureAccess', 'ExpertFeatureAccessKey', 'ExpertFeatureAccessSelector', 'ExpertFeatureEnable', 'ExposureAuto', 'ExposureMode', 'ExposureOverlapTimeMax', 'ExposureOverlapTimeMode', 'ExposureTime', 'FileAccessBuffer', 'FileAccessControl', 'FileAccessLength', 'FileAccessOffset', 'FileOpenMode', 'FileOperationExecute', 'FileOperationResult', 'FileOperationSelector', 'FileOperationStatus', 'FileSelector', 'FileSize', 'Gain', 'GainAuto', 'GainSelector', 'Gamma', 'Height', 'HeightMax', 'ImageFormatControl', 'ImageQualityControl', 'LUTControl', 'LUTEnable', 'LUTIndex', 'LUTSelector', 'LUTValue', 'LUTValueAll', 'LightSourcePreset', 'LineDebouncerTime', 'LineFormat', 'LineInverter', 'LineLogic', 'LineMinimumOutputPulseWidth', 'LineMode', 'LineOverloadStatus', 'LineSelector', 'LineSource', 'LineStatus', 'LineStatusAll', 'NoiseReduction', 'OffsetX', 'OffsetY', 'PGIControl', 'PayloadSize', 'PixelColorFilter', 'PixelDynamicRangeMax', 'PixelDynamicRangeMin', 'PixelFormat', 'PixelSize', 'RemoveParameterLimit', 'RemoveParameterLimitControl', 'RemoveParameterLimitSelector', 'ResultingFrameRate', 'ReverseX', 'ReverseY', 'Root', 'SIPayloadFinalTransfer1Size', 'SIPayloadFinalTransfer2Size', 'SIPayloadTransferCount', 'SIPayloadTransferSize', 'ScalingHorizontal', 'ScalingVertical', 'SensorHeight', 'SensorReadoutMode', 'SensorReadoutTime', 'SensorWidth', 'SequencerConfigurationMode', 'SequencerControl', 'SequencerMode', 'SequencerPathSelector', 'SequencerSetActive', 'SequencerSetLoad', 'SequencerSetNext', 'SequencerSetSave', 'SequencerSetSelector', 'SequencerSetStart', 'SequencerTriggerActivation', 'SequencerTriggerSource', 'SharpnessEnhancement', 'ShutterMode', 'SoftwareSignalControl', 'SoftwareSignalPulse', 'SoftwareSignalSelector', 'TemperatureState', 'TestImageResetAndHold', 'TestImageSelector', 'TestPendingAck', 'TimerDelay', 'TimerDuration', 'TimerSelector', 'TimerTriggerSource', 'TimestampLatch', 'TimestampLatchValue', 'TransportLayerControl', 'TriggerActivation', 'TriggerDelay', 'TriggerEventTest', 'TriggerMode', 'TriggerSelector', 'TriggerSoftware', 'TriggerSource', 'UserDefinedValue', 'UserDefinedValueControl', 'UserDefinedValueSelector', 'UserOutputSelector', 'UserOutputValue', 'UserOutputValueAll', 'UserSetControl', 'UserSetDefault', 'UserSetLoad', 'UserSetSave', 'UserSetSelector', 'Width', 'WidthMax', 'clear_xml_cache', 'concatenated_write', 'connect', 'device_info', 'disconnect', 'get_node', 'has_node', 'load_xml_from_file', 'load_xml_from_string', 'load_xml_from_zip_file', 'nodes', 'parse_swiss_knives', 'pointer', 'poll', 'this', 'thisown']

```
</div>