# 1. UDP packet formats

- [1. UDP packet formats](#1-udp-packet-formats)
  - [1.1. General UDP packet format](#11-general-udp-packet-format)
    - [1.1.1. Packet header](#111-packet-header)
    - [1.1.2. Packet type](#112-packet-type)
  - [1.2. Format of individual packet type](#12-format-of-individual-packet-type)
    - [1.2.1. Server open packet](#121-server-open-packet)
      - [1.2.1.1. About port](#1211-about-port)
    - [1.2.2. Request port packet](#122-request-port-packet)
      - [1.2.2.1. Base station type](#1221-base-station-type)
    - [1.2.3. Open port packet](#123-open-port-packet)
      - [1.2.3.1. Request port result](#1231-request-port-result)
    - [1.2.4. Device setting packet](#124-device-setting-packet)
      - [1.2.4.1. Beacon order and Super frame order](#1241-beacon-order-and-super-frame-order)
      - [1.2.4.2. Device ID](#1242-device-id)
    - [1.2.5. Device light control packet](#125-device-light-control-packet)
      - [1.2.5.1. Blink interval](#1251-blink-interval)
      - [1.2.5.2. Color code in RGB](#1252-color-code-in-rgb)
    - [1.2.6. Device sleep control packet](#126-device-sleep-control-packet)
      - [1.2.6.1. Device sleep control](#1261-device-sleep-control)
    - [1.2.7. Station ready packet](#127-station-ready-packet)
    - [1.2.8. Device data packet](#128-device-data-packet)
      - [1.2.8.1. Frame ID](#1281-frame-id)
      - [1.2.8.2. Device data](#1282-device-data)
    - [1.2.9. Device data with time stamp and CIR packet](#129-device-data-with-time-stamp-and-cir-packet)
      - [1.2.9.1. Time stamp](#1291-time-stamp)
      - [1.2.9.2. Diagnostic data](#1292-diagnostic-data)
      - [1.2.9.3. Dummy byte and CIR data](#1293-dummy-byte-and-cir-data)
    - [1.2.10. Device info packet](#1210-device-info-packet)
      - [1.2.10.1. Device type](#12101-device-type)
    - [1.2.11. Device data count packet](#1211-device-data-count-packet)
      - [1.2.11.1. Device data count](#12111-device-data-count)

## 1.1. General UDP packet format

The general UDP packet format for the communication between UWB base station and server is defined in the Table below.

|Bytes:1|2|n|
|---|---|---|
|Packet header|Packet type|Packet payload|

### 1.1.1. Packet header

|Packet header value|Description|
|:---:|---|
|0xFD|Same for all packet|

### 1.1.2. Packet type

The Packet type part contains two Byte, the first Byte is wether 0xCF or 0xDF, defines the usage of the packet, 0xCF means it used for control, 0xDF means it is for data transfering. second byte defines the specific packet type.

Each type of packet has a defined transfer direction, it is wether send from base station to server(mark with &larr;) or send from server to base station(mark with &rarr;).

|Packet type value<br>Byte 1|<br>Byte 2|Description|Transfer direction|
|:---:|:---:|---|:---:|
|0xCF|0x01|Server open Packet|&rarr;|
|0xCF|0x02|Request port Packet|&larr;|
|0xCF|0x03|Open port Packet|&rarr;|
|0xCF|0x04|Device setting Packet|&rarr;|
|0xCF|0x05|Device light control Packet|&rarr;|
|0xCF|0x06|Device sleep control Packet|&rarr;|
|0xCF|0x07|Station ready Packet|&larr;|
|0xDF|0x01|Device data Packet|&larr;|
|0xDF|0x02|Device data with time stamp and CIR Packet|&larr;|
|0xDF|0xF1|Device info Packet|&larr;|
|0xDF|0xF2|Device data count Packet|&larr;|

## 1.2. Format of individual packet type

### 1.2.1. Server open packet

|Bytes:1|2|2|
|:---:|:---:|:---:|
|Packet header|Packet type|Server port|
|0xFD|0xCF01|2 bytes variable|

Inform all the base station that this server could be access now and give the open port of server to receive data packet form base stations.

***This packet that send form server to base station should be boardcast to all base stations like send to 192.168.0.255***

***The port for all base stations to receive data from server is 8082***

#### 1.2.1.1. About port

All ports is defined to store in little endian use two bytes.
For example is the server open port 8086, which is 0x1F96 in Hex, the two byte for Server port part will be 0x96 0x1F.

### 1.2.2. Request port packet

|Bytes:1|2|1|2|
|:---:|:---:|:---:|:---:|
|Packet header|Packet type|Base station type|Requested port|
|0xFD|0xCF02|0x0F&#124;0xF0|2 bytes variable|

When the port informed by Server open packet is conflict with other application, the base station could send this packet to server to request a new port for data transmittion.

#### 1.2.2.1. Base station type

|Base station type value|Description|
|:---:|---|
|0x0F|Main base station|
|0xF0|Sub base station|

The type of this base station.

### 1.2.3. Open port packet

|Bytes:1|2|1|2|
|:---:|:---:|:---:|:---:|
|Packet header|Packet type|Requset port result|Opened port|
|0xFD|0xCF03|0x00&#124;0x0F&#124;0xFF|2 bytes variable|

This packet tells the result of the Request port packet of base station, and tell it the port opened.

#### 1.2.3.1. Request port result

|Requset port result value|Description|
|:---:|---|
|0x00|Accept|
|0x0F|Open another port|
|0xFF|Reject|

The result of request port.

### 1.2.4. Device setting packet

|Bytes:1|2|1|n|
|:---:|:---:|:---:|:---:|
|Packet header|Packet type|Beacon order and Super frame order|Device ID list|
|0xFD|0xCF04|1 bytes variable|n bytes variable|

Setting the beacon order (The beacon sending interval of main base station), and super frame order (The UWB sample interval of device) for the devices in device ID list. The device ID list could be empty to set all device at the same time.

#### 1.2.4.1. Beacon order and Super frame order

This part is actually pack 2 parts of setting in one byte based on the IEEE802.15.4 specifcation.

|Bits:4|4|
|:---:|:---:|
|Beacon order|Surper frame order|
|4 bits of variable|4 bits of variable|

|Beacon order value|Description|
|:---:|---|
|0x0|Send as the sample interval|
|0x1|1 second|
|0x2|2 second|
|0x3|3 second|
|0x4|4 second|
|0x5|5 second|
|0x6|6 second|
|0x7|7 second|
|0x8|8 second|
|0x9|9 second|
|0xA|10 second|
|0xB|30 second|
|0xC|1 miniutes|
|0xD|5 miniutes|
|0xE|10 miniutes|
|0xF|30 miniutes|

|Super frame order value|Description|
|:---:|---|
|0x0|500 us|
|0x1|1 ms|
|0x2|2 ms|
|0x3|3 ms|
|0x4|4 ms|
|0x5|5 ms|
|0x6|6 ms|
|0x7|7 ms|
|0x8|8 ms|
|0x9|9 ms|
|0xA|10 ms|
|0xB|20 ms|
|0xC|50 ms|
|0xD|100 ms|
|0xE|200 ms|
|0xF|1000 ms|

#### 1.2.4.2. Device ID

The device ID is a 1 byte short ID of UWB device from 0x00 to 0xFF, device ID will be dynamicly allocate to UWB device when it connect to the UWB network.

### 1.2.5. Device light control packet

|Bytes:1|2|1|3|n|
|:---:|:---:|:---:|:---:|:---:|
|Packet header|Packet type|Blink interval|Color code in RGB|Device ID list|
|0xFD|0xCF05|1 bytes variable|3 bytes variable|n bytes variable|

Control the RGB LED on device in device ID list.

#### 1.2.5.1. Blink interval

|Blink interval value|Description|
|:---:|---|
|0x00|LED off|
|0x01-0xFE|Blink interval $100x$ ms|
|0xFF|LED on without blink|

This byte control the blink interval of LED.

#### 1.2.5.2. Color code in RGB

|Bytes:1|1|1|
|:---:|:---:|:---:|
|Red|Green|Blue|

Based on the color will infect the brightness of the RGB LED, the color should not be larger than 255.

### 1.2.6. Device sleep control packet

|Bytes:1|2|1|n|
|:---:|:---:|:---:|:---:|
|Packet header|Packet type|Device sleep control|Device ID list|
|0xFD|0xCF06|1 bytes variable|n bytes variable|

Control the sleep and wakeup behavior of device.

#### 1.2.6.1. Device sleep control

|Bits:1|1|1|1|1|1|1|1|
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
|Auto sleep between transmission|Auto sleep when idle|Auto sleep when rest|Auto wakeup when move|Reserved|Reserved|Reserved|Reserved|

This byte define the sleep and wakeup related behavior of a device. 0 for disable the function, 1 for enable the funciton.

|Function|Description|Default value|
|:---:|---|:---:|
|Auto sleep between transmission|Device will enter sleep mode once it successfully transmit a data frame to base stations, for example, if the sample interval is set to 10 ms, the device will enter sleep for around 8ms and use 1ms to wakeup then transmit the next data frame, this will largely decrease power consumption. This configuration only work for sample interval larger than 3ms because it will takes around 1.5 ms to wakeup the device|1|
|Auto sleep when idle|Device will enter sleep when it do not receive any beacon or receive the same beacon for 60 second|0|
|Auto sleep when rest|Device will enter sleep when it not move for 60 second|0|
|Auto wakeup when move|Device will wakeup from sleep when it is moved|0|

### 1.2.7. Station ready packet

|Bytes:1|2|2|
|:---:|:---:|:---:|
|Packet header|Packet type|Base station port|
|0xFD|0xCF07|2 bytes variable|

Tell the server that the base station is available for data transmition and notify the port base station use to send data.

### 1.2.8. Device data packet

|Bytes:1|2|1|1|24|
|:---:|:---:|:---:|:---:|:---:|
|Packet header|Packet type|Frame ID|Device ID|Device data|
|0xFD|0xDF01|1 bytes variable|1 bytes variable|24 bytes variable|

Send the data received by base station to server without timestamps.

#### 1.2.8.1. Frame ID

It is 1 byte Frame ID that keep increase from 0x00 to 0xFF. For one sample interval, Frame ID should be same for all devices.

#### 1.2.8.2. Device data

|Bytes:6|6|6|6|
|:---:|:---:|:---:|:---:|
|Acceleration|Angular velocity|Angle|Magnetic|
|6 bytes variable|6 bytes variable|6 bytes variable|6 bytes variable|

The device data contains four different types of data. Each contains 3 axis(x, y, z) of data. Every data is 16bit signed int.

|Bytes:1|1|1|1|1|1|
|:---:|:---:|:---:|:---:|:---:|:---:|
|X data L|X data H|Y data L|Y data H|Z data L|Z data H|

To process the data, you need to convert 2 bytes into int then multipily the range of sensor.

For magnetic:

```c
((int)dataH<<8|dataL)*1.0*range
```

For other data type:

```c
((int)dataH<<8|dataL)*1.0/32768*range
```

|Data type|Range|unit|
|:---:|---|---|
|Acceleration|16|$g$|
|Angular velocity|2000|$\degree/s$|
|Angle|180|$\degree$|
|Magnetic|0.98|$mgauss$|

### 1.2.9. Device data with time stamp and CIR packet

|Bytes:1|2|1|1|24|4|24|1|1152|
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
|Packet header|Packet type|Frame ID|Device ID|Device data|Time stamp|Diagnostic data|dummy byte|CIR data|
|0xFD|0xDF02|1 bytes variable|1 bytes variable|24 bytes variable|4 bytes variable|24 bytes variable|0x00|1152 bytes variable|

Send the data received by base station to server with timestamps.

#### 1.2.9.1. Time stamp

|Bytes:1|1|1|1|
|:---:|:---:|:---:|:---:|
|timestamp0|timestamp1|timestamp2|timestamp3|

To process timestamp:

```c
timestamp3<<24|timestamp2<<16|timestamp1<<8|timestamp0*15.65
```

The time unit is $ps$.

#### 1.2.9.2. Diagnostic data

|Bytes:4|4|4|4|4|2|2|
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
|ipatovPeak|ipatovPower|ipatovF1|ipatovF2|ipatovF3|ipatovFpIndex|ipatovAccumCount|

The diag data of received UWB frame, refer to user manual [DW3000 User manual](./../文档/DW3000-User-Manual.pdf), and [api guide p80](./../文档/DW3000_Software_API_Guide.pdf).

#### 1.2.9.3. Dummy byte and CIR data

One byte dummy data and CIR data from index 700 to 891. refer to [api guide p77](./../文档/DW3000_Software_API_Guide.pdf).

### 1.2.10. Device info packet

|Bytes:1|2|1|2|2|
|:---:|:---:|:---:|:---:|:---:|
|Packet header|Packet type|Device ID|Device type|Battery persentage|
|0xFD|0xDFF1|1 bytes variable|2 bytes variable|2 bytes variable|

Base station will inform Server the device info when it connected, or the device info is updated.

#### 1.2.10.1. Device type

|Device type value|Description|
|:---:|---|
|0x0001|V0.5 UWB-IMU<br>MCU:CH32V203<br>UWB:DW3210<br>Sensor:<br>BMI270 IMU|

Indecate the type of device.

### 1.2.11. Device data count packet

|Bytes:1|2|3n|
|:---:|:---:|:---:|
|Packet header|Packet type|Device data count list|
|0xFD|0xDFF2|3n bytes variable|

Tell the server how many UWB frame the base station has received.

#### 1.2.11.1. Device data count

|Bytes:1|2|
|:---:|:---:|
|Device ID|Data count|
