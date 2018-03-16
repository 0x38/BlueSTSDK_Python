# BlueST SDK

BlueST is a multi-platform library ([Android](https://github.com/STMicroelectronics-CentralLabs/BlueSTSDK_Android), [iOS](https://github.com/STMicroelectronics-CentralLabs/BlueSTSDK_iOS), and [Linux](https://github.com/STMicroelectronics-CentralLabs/BlueSTSDK_Python) supported) that permits easy access to the data exported by a Bluetooth Low Energy (BLE) device that implements the BlueST Protocol.


## Compatibility
This version of the SDK is compatible with [Python](https://www.python.org/) 2.7 and runs on a Linux system.


## Preconditions
The BlueST SDK makes use of the [bluepy](https://github.com/IanHarvey/bluepy) Python interface to Bluetooth Low Energy on Linux.
Moreover, it uses the [concurrent.futures](https://docs.python.org/3/library/concurrent.futures.html) module to run pools of threads in background, that serve listeners' callbacks.

Please follow the official instructions to install the mentioned libraries.


## How to run the example application
To run the BlueST Example application please follow the steps below:
 1. Download the BlueST SDK and the BlueST Example.
 2. Add the parent folder of the "BlueSTSDK" package to the "PYTHONPATH" environment variable. On Linux:
    ```Shell
    $ export PYTHONPATH=/home/<user>/.../<parent-of-BlueSTSDK>/
    ```
 3. Run the main script:
    ```Shell
    $ python Test_BLE_1.py
    ```

## BlueST Protocol

### Advertising data
The library shows only the devices that have a vendor-specific field formatted in the following way:

|Length|  1       |1           | 1                |1          | 4              | 6                    |
|------|----------|------------|------------------|-----------|----------------|----------------------|
|Name  | Length   | Field Type | Protocol Version | Device Id | Feature Mask   | Device MAC (optional)|
|Value | 0x07/0x0D | 0xFF       | 0x01             | 0xXX      | 0xXXXXXXXX     | 0xXXXXXXXXXXXX       |


 - The Field Length must be 7 or 13 bytes long.
 
 - The Device Id is a number that identifies the type of device:
    - 0x00 for a generic device
    - 0x01 is reserved for the [STEVAL-WESU1](http://www.st.com/en/evaluation-tools/steval-wesu1.html) board
    - 0x02 is reserved for the [STEVAL-STLKT01V1 (SensorTile)](http://www.st.com/content/st_com/en/products/evaluation-tools/solution-evaluation-tools/sensor-solution-eval-boards/steval-stlkt01v1.html) board
    - 0x03 is reserved for the [STEVAL-BCNKT01V1 (BlueCoin)](http://www.st.com/content/st_com/en/products/evaluation-tools/solution-evaluation-tools/sensor-solution-eval-boards/steval-bcnkt01v1.html) board
    - 0x04 is reserved for the [STEVAL-IDB007VX (BlueNRG1)](http://www.st.com/content/st_com/en/products/evaluation-tools/solution-evaluation-tools/communication-and-connectivity-solution-eval-boards/steval-idb007v1.html) and [STEVAL-IDB008VX (BlueNRG2)](http://www.st.com/content/st_com/en/products/evaluation-tools/solution-evaluation-tools/communication-and-connectivity-solution-eval-boards/steval-idb008v1.html) boards
    - 0x80 for a generic Nucleo board

  In case you need to define your own custom boards, you should use Device Id values not yet assigned. Moreover, please note that values between 0x80 and 0xFF are reserved for ST Nucleo boards.
 
 - The feature mask is a bit field that provides information about which features are exported by the board. Currently, bits are mapped in the following way:
  
   |Bit|31|30|29|28|27|26|25|24|23|22|21|20|19|18|17|16|
   |:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
   |Feature|Analog|ADPCM Sync|Switch|Direction of arrival|ADPCM Audio|Microphone Level|Proximity|Luxmeter|Accelerometer|Gyroscope|Magnetometer|Pressure|Humidity|Temperature|Battery|Second Temperature|
   
   |Bit|15|14|13|12|11|10|9|8|7|6|5|4|3|2|1|0|
   |:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
   |Feature|CO Sensor|DC Motor|Stepper Motor|SD Logging|Beam forming|Accelerometer Event|Free Fall|Sensor Fusion Compact|Sensor Fusion|Motion Intensity|Compass|Activity|Carry Position|Proximity Gesture|MEMS Gesture|Pedometer|

To understand the way the data are exported by predefined features, please refer to the method <code> Feature.extractData(self, timestamp, data, offset)</code> within the features class definition.

- The device MAC address is optional, and needed only on the iOS platform.


### Note
Currently only a subset of the features is implemented: Temperature, Humidity, Pressure, Accelerometer, Gyroscope, and Magnetometer.
Future releases of the Python SDK will cover all the abovementioned features.


### Characteristics/Features
The characteristics managed by the SDK must have a UUID as follows: <code>XXXXXXXX-0001-11e1-ac36-0002a5d5c51b</code>.
The SDK scans all the services, searching for characteristics that match the pattern. 

The first part of the UUID has bits set to "1" for each feature exported by the characteristics.

In case of multiple features mapped onto a single characteristic, the data must be in the same order as the bit mask.

A characteristic's data format must be the following:
 
| Length |     2     |         >1         |          >1         |       |
|:------:|:---------:|:------------------:|:-------------------:|:-----:|
|  Name  | Timestamp | First Feature Data | Second Feature Data | ..... |
  
 The first 2 bytes are used to communicate a timestamp. This is especially useful to recognize any loss of data.
 
 Since the BLE packet maximum length is 20 bytes, the maximumm size of a feature's data field is 18 bytes.
 

### Example
The SDK is compatible with the following ST firmware:
 - [FP-SNS-MOTENV1](http://www.st.com/content/st_com/en/products/embedded-software/mcus-embedded-software/stm32-embedded-software/stm32-ode-function-pack-sw/fp-sns-motenv1.html): STM32 ODE function pack for IoT node with BLE connectivity plus environmental and motion sensors
 - [FP-SNS-ALLMEMS1](http://www.st.com/content/st_com/en/products/embedded-software/mcus-embedded-software/stm32-embedded-software/stm32-ode-function-pack-sw/fp-sns-allmems1.html): STM32 ODE function pack for IoT node with BLE connectivity, digital microphone, environmental and motion sensors
 - [FP-SNS-FLIGHT1](http://www.st.com/content/st_com/en/products/embedded-software/mcus-embedded-software/stm32-embedded-software/stm32-ode-function-pack-sw/fp-sns-flight1.html): STM32 ODE function pack for IoT node with BLE connectivity, environmental and motion sensors, time-of-flight sensors (Please remove NFC when used with Python SDK)


## Main actors

### Manager
This is a singleton class that starts/stops the discovery process and stores the retrieved nodes.

Before starting the scanning process, it is also possible to define a new Device Id and to register/add new features to already defined devices.

The Manager notifies a new discovered node through the <code>Manager.ManagerListener</code> class. Each callback is performed asynchronously by a thread running in background.

### Node
This class represents a remote device.

Through this class it is possible to recover the features exported by a node and read/write data from/to the device.

The node exports all the features whose corresponding bit is set to "1" within the advertising data message. Once the device is connected, scanning and enabling the available characteristics can be performed. Then, it is possible to request/send data related to the discovered features.

A node notifies its RSSI (signal strength) when created.

A node can be in one of following status:
- **Init**: dummy initial status.
- **Idle**: the node is waiting for a connection and sending an advertising data message.
- **Connecting**: a connection with the node was triggered, the node is performing the discovery of device services/characteristics.
- **Connected**: connection with the node was successful.
- **Disconnecting**: ongoing disconnection; once disconnected the node goes back to the Idle status.
- **Lost**: the device has sent an advertising data, however it is not reachable anymore.
- **Unreachable**: the connection with the node was in place, however it is not reachable anymore.
- **Dead**: dummy final status.

Each callback is performed asynchronously by a background thread.

### Feature
This class represents the data exported by a node.

Each feature has an array of <code>Field</code> objects that describes the data exported.

Data are received from a BLE characteristic and contained in a <code>Feature.Sample</code> class. The user is notified about new data through a listener.

Note that each callback is performed asynchronously by a thread running in background.

Available features can be retrieved from Features package.

#### How to add a new Feature

 1. Extend the Feature class:
    1.  Create an array of <code>Feature.Field</code> objects that describes the data exported by the feature.
    2.  Create a constructor that accepts only the node as a parameter. From this constructor call the superclass constructor, passing the feature's name and the feature's fields.
    3.  Implement the method <code> Feature.extractData(self, timestamp, data, offset)</code>.
    4.  Implement a class method that allows to get data from a <code>Feature.Sample</code> object.
 2. Register the new feature:
    If you want to use BlueST's bitmask for features within the advertising data, please register the new feature before performing the discovery process, e.g.:
 
    ```Python
    # Adding a 'MyFeature' feature to a Nucleo device, mapping it to a '0x10000000-0001-11e1-ac36-0002a5d5c51b' characteristic.
    mask_to_features_dic = {}
    mask_to_features_dic[0x10000000] = BlueSTSDK.Features.MyFeature.MyFeature
    try:
        Manager.addFeaturesToNode(0x80, mask_to_features_dic)
    except InvalidFeatureBitMaskException as e:
        print e
    # Synchronous discovery of Bluetooth devices.
    manager.discover(False, SCANNING_TIME_s)
    ```

    Otherwise, you can register the feature after discovering a node and before connecting to it:
    ```Python
    map = UUIDToFeatureMap()
    map.put(uuid.UUID('00002a37-0000-1000-8000-00805f9b34fb'), BlueSTSDK.Features.StandardCharacteristics.FeatureHeartRate.FeatureHeartRate)
    node.addExternalFeatures(map)
    node.connect()


## License
  COPYRIGHT(c) 2018 STMicroelectronics                                          

  Redistribution and use in source and binary forms, with or without
  modification, are permitted provided that the following conditions are met:
    1. Redistributions of source code must retain the above copyright notice,
       this list of conditions and the following disclaimer.
    2. Redistributions in binary form must reproduce the above 
       notice, this list of conditions and the following disclaimer in the
       documentation and/or other materials provided with the distribution.
    3. Neither the name of STMicroelectronics nor the names of its
       contributors may be used to endorse or promote products derived from
       this software without specific prior written permission.

  THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
  AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
  IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
  ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE
  LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
  CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
  SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
  INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN
  CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE)
  ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE
  POSSIBILITY OF SUCH DAMAGE.
