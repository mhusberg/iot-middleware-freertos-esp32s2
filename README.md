## ESP32s2 example using Azure IoT middleware for FreeRTOS

This project contains code neccesary to connect an ESP23s2 microcontroller to Azure IoT hub and use features such as Over the Air (OTA) Azure Device Update (ADU). In this example a temperature/humidity sensor is connected to the microcontroller. With this "real life" telemetry data can be sent in intervals to IoT hub through MQTT. The sleep-interval property can be updated OTA by modifying the device twin desired properties. This way you can remotly define the interval between sending telemetry data.

This source code is built direcly upon the [ESP32](demos/projects/ESPRESSIF/esp32/) sample from the  open source <a href="https://github.com/Azure-Samples/iot-middleware-freertos-samples">azure-iot-middleware-freertos-samples</a> repo. There are multiple different samples in that repo and it can be forked to be used as a starting point for your embedded Azure IoT project.


The different hardware used in this example:

<a href="https://www.fibel.no/product/dht22-temperatur-fuktighetssensor/"><img src="./docs/resources/dht22.jpeg" width="50%" /></a><a href="https://www.wemos.cc/en/latest/s2/s2_mini.html"><img src="./docs/resources/esp32s2.jpg" width="50%" /></a>

## Get Started

```bash
git clone https://github.com/mhusberg/iot-middleware-freertos-esp32s2.git
```
Download and install the <a href="https://docs.espressif.com/projects/esp-idf/en/stable/esp32/get-started/index.html">ESP IoT Development Framework (ESP-IDF)</a>. It is recommended going through some of the get-started guides from their documentation, however, if you're already familiar with ESP-IDF you can ignore it.

To use idf.py commands it must first be exported through the following bash script:

```bash
. $HOME/esp/esp-idf/export.sh
```

The `set-target` command is used based on the device you're using, in this case it's the `esp32s2`. The `menuconfig` command generates and populates a sdkconfig file. It also opens a navigatable UI in which you can change some of the default config settings, enter wifi credentials, and Azure specific credentials (This is further explained down below).

```bash
idf.py set-target esp32s2
idf.py menuconfig
```
When you're in the menuconfig UI, under menu item `Component config --> ESP System Settings` set the `Channel for console output` setting to `USB CDC`.
Under menu item `Azure IoT middleware for FreeRTOS Sample Configuration`, update the following configuration values:

Parameter | Value
---------|----------
 `WiFi SSID` | _{Your WiFi SSID}_
 `WiFi Password` | _{Your WiFi password}_


Under menu item `Azure IoT middleware for FreeRTOS Main Task Configuration`, update the following configuration values:

Parameter | Value
---------|----------
 `Use PnP in Azure Sample` | Enabled by default. Disable this option to build a simpler sample without Azure Plug-and-Play.
 `Azure IoT Device ID` | _{Your Azure IoT Hub device ID}_
 `Azure IoT Module ID` | _{Your Azure IoT Hub Module ID}_ (optional, specify module id if using a device module; else leave blank if not)

Select your desired authentication method with the `Azure IoT Authentication Method () --->`. The default option is `Symmetric Key`:

Parameter | Value
---------|----------
 `Azure IoT Device Symmetric Key` | _{Your Azure IoT Hub device symmetric key}_

If you would like to use x509 certificates, select `X509 Certificates` and update the following values:

Parameter | Value
---------|----------
 `Azure IoT Device Client Certificate` | _{Your Azure IoT Hub device certificate}_
 `Azure IoT Device Client Certificate Private Key` | _{Your Azure IoT Hub device certificate private key}_

```bash
idf.py build
idf.py -p <PORT> flash
```
On Windows this is typically your `COM3` port, on MacOS and linux the port can look like `/dev/cu.usbserial01` or `/dev/cs.usbmodem101`. Once it's finished flashing, use `idf monitor` to read the output of the device over serial. After it's finished initializing the output should write something in the lines of:
<details><summary>See more...</summary>
<br>

```
I (10342) MQTT: Packet received. ReceivedBytes=2.
I (10342) MQTT: CONNACK session present bit set.
I (10342) MQTT: Connection accepted.
I (10342) MQTT: Received MQTT CONNACK successfully from broker.
I (10352) MQTT: MQTT connection established with the broker.
I (10362) AZ IOT: An MQTT connection is established with iot-demo-norwayeast.azure-devices.net
I (10402) MQTT: Packet received. ReceivedBytes=3.
I (10402) AZ IOT: Suback receive context found: 0x00000001
I (10442) MQTT: Packet received. ReceivedBytes=4.
I (10442) AZ IOT: Suback receive context found: 0x00000002
I (10452) AZ IOT: DHT22 sensor read pin: 4
E (10472) DHT: Sensor Timeout

I (10482) AZ IOT: ucScratchBuffer: { "temperature": "0.00", "humidity": "0.00" }
I (10512) AZ IOT: Successfully sent telemetry message
I (10522) AZ IOT: ucReportedPropertiesUpdate:
I (10532) AZ IOT: Attempt to receive publish message from IoT Hub.

I (10572) MQTT: Packet received. ReceivedBytes=721.
I (10572) MQTT: De-serialized incoming PUBLISH packet: DeserializerResult=MQTTSuccess.
I (10582) MQTT: State record updated. New state=MQTTPublishDone.
I (10592) AZ IOT: $iothub/twin/res/200/?$rid=2
I (10592) AZ IOT: Unknown property arrived: skipping over it.
I (10602) AZ IOT: Successfully parsed properties
W (10602) AZ IOT: Incoming sleep interval: 5.000000

I (10612) AZ IOT: Same sleep interval as before, no changes needed.
I (10622) AZ IOT: Current sleep interval: 5.000000
I (10952) MQTT: Packet received. ReceivedBytes=44.
I (10952) MQTT: De-serialized incoming PUBLISH packet: DeserializerResult=MQTTSuccess.
I (10962) MQTT: State record updated. New state=MQTTPublishDone.
I (10972) AZ IOT: $iothub/twin/res/204/?$rid=3&$version=1730
I (11012) MQTT: Packet received. ReceivedBytes=44.
I (11012) MQTT: De-serialized incoming PUBLISH packet: DeserializerResult=MQTTSuccess.
I (11022) MQTT: State record updated. New state=MQTTPublishDone.
I (11032) AZ IOT: $iothub/twin/res/204/?$rid=5&$version=1731
I (11042) AZ IOT: sleepInterval: 500
I (11042) AZ IOT: Keeping Connection Idle...


E (16062) DHT: Sensor Timeout

I (16072) AZ IOT: ucScratchBuffer: { "temperature": "0.00", "humidity": "0.00" }
I (16092) AZ IOT: Successfully sent telemetry message
I (16102) AZ IOT: ucReportedPropertiesUpdate: {"sleepInterval":{"ac":200,"av":20,"ad":"success","value":5}}
I (16132) AZ IOT: Attempt to receive publish message from IoT Hub.

I (16152) MQTT: Packet received. ReceivedBytes=2.
I (16152) MQTT: Ack packet deserialized with result: MQTTSuccess.
I (16162) MQTT: State record updated. New state=MQTTPublishDone.
I (16172) AZ IOT: Puback received for packet id: 0x00000003
I (16202) MQTT: Packet received. ReceivedBytes=2.
I (16202) MQTT: Ack packet deserialized with result: MQTTSuccess.
I (16212) MQTT: State record updated. New state=MQTTPublishDone.
I (16222) AZ IOT: Puback received for packet id: 0x00000004
I (16492) MQTT: Packet received. ReceivedBytes=44.
I (16492) MQTT: De-serialized incoming PUBLISH packet: DeserializerResult=MQTTSuccess.
I (16502) MQTT: State record updated. New state=MQTTPublishDone.
I (16512) AZ IOT: $iothub/twin/res/204/?$rid=7&$version=1732
I (18542) AZ IOT: sleepInterval: 500
I (18542) AZ IOT: Keeping Connection Idle...


I (23552) AZ IOT: ucScratchBuffer: { "temperature": "23.80", "humidity": "39.50" }
I (23582) AZ IOT: Successfully sent telemetry message
I (23592) AZ IOT: ucReportedPropertiesUpdate: {"sleepInterval":5}ac":200,"av":20,"ad":"success","value":5}}
I (23612) AZ IOT: Attempt to receive publish message from IoT Hub.

I (23672) MQTT: Packet received. ReceivedBytes=2.
I (23682) MQTT: Ack packet deserialized with result: MQTTSuccess.
I (23692) MQTT: State record updated. New state=MQTTPublishDone.
I (23702) AZ IOT: Puback received for packet id: 0x00000005
I (23862) MQTT: Packet received. ReceivedBytes=44.
I (23862) MQTT: De-serialized incoming PUBLISH packet: DeserializerResult=MQTTSuccess.
I (23872) MQTT: State record updated. New state=MQTTPublishDone.
I (23882) AZ IOT: $iothub/twin/res/204/?$rid=9&$version=1733
I (25902) AZ IOT: sleepInterval: 500
I (25902) AZ IOT: Keeping Connection Idle...


I (30912) AZ IOT: ucScratchBuffer: { "temperature": "23.90", "humidity": "38.90" }
I (30932) AZ IOT: Successfully sent telemetry message
I (30942) AZ IOT: ucReportedPropertiesUpdate: {"sleepInterval":5}ac":200,"av":20,"ad":"success","value":5}}
I (30972) AZ IOT: Attempt to receive publish message from IoT Hub.

I (31072) MQTT: Packet received. ReceivedBytes=2.
I (31072) MQTT: Ack packet deserialized with result: MQTTSuccess.
I (31082) MQTT: State record updated. New state=MQTTPublishDone.
I (31092) AZ IOT: Puback received for packet id: 0x00000006
I (31182) MQTT: Packet received. ReceivedBytes=45.
I (31182) MQTT: De-serialized incoming PUBLISH packet: DeserializerResult=MQTTSuccess.
I (31192) MQTT: State record updated. New state=MQTTPublishDone.
I (31202) AZ IOT: $iothub/twin/res/204/?$rid=11&$version=1734
I (33222) AZ IOT: sleepInterval: 500
I (33222) AZ IOT: Keeping Connection Idle...
``````
</details>

Exit monitoring with `CTRL-T + CTRL-X`.

## Troubleshooting

If your device is not showing as `COM3` on Windows or `/dev/cu.usbserial01` on MacOS/linux, you might need a serial driver. https://www.silabs.com/developers/usb-to-uart-bridge-vcp-drivers?tab=downloads



If you're having trouble with `idf.py monitor` and output not showing, try pressing the `rst` button on your device after flashing it.



