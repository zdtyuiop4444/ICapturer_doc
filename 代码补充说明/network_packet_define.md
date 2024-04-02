# Network packet formats

- [Network packet formats](#network-packet-formats)
  - [1. General network packet format](#1-general-network-packet-format)
    - [1.1. Packet header](#11-packet-header)
    - [1.2. Packet type](#12-packet-type)
  - [2. Format of individual packet type](#2-format-of-individual-packet-type)
    - [2.1. Server open packet](#21-server-open-packet)
      - [2.1.1. About port](#211-about-port)
      - [2.1.2. Enable diagnostic](#212-enable-diagnostic)
    - [2.2. Device light control packet](#22-device-light-control-packet)
      - [2.2.1. Blink interval](#221-blink-interval)
      - [2.2.2. Color code in RGB](#222-color-code-in-rgb)
      - [2.2.3. Device ID](#223-device-id)
    - [2.3. Device sleep control packet](#23-device-sleep-control-packet)
      - [2.3.1. Device sleep control](#231-device-sleep-control)
        - [2.3.1.1. Auto sleep between transmission](#2311-auto-sleep-between-transmission)
    - [2.4. Device setting packet](#24-device-setting-packet)
      - [2.4.1. Beacon order and Super frame order](#241-beacon-order-and-super-frame-order)
      - [2.4.2. IMU sample rate](#242-imu-sample-rate)
    - [2.5. Station ready packet](#25-station-ready-packet)
      - [2.5.1. Base station type](#251-base-station-type)
    - [2.6. Device data packet for 6 axis IMU](#26-device-data-packet-for-6-axis-imu)
      - [2.6.1. Base station ID](#261-base-station-id)
      - [2.6.2. Frame ID](#262-frame-id)
      - [2.6.3. IMU data sample count](#263-imu-data-sample-count)
      - [2.6.4. IMU data bitwidth](#264-imu-data-bitwidth)
      - [2.6.5. IMU timestamp bitwidth](#265-imu-timestamp-bitwidth)
      - [2.6.6. IMU data contain](#266-imu-data-contain)
      - [2.6.7. Station data contain](#267-station-data-contain)
      - [2.6.8. Device data](#268-device-data)
      - [2.6.9. data decode](#269-data-decode)
      - [2.6.10. Station data](#2610-station-data)
      - [2.6.11. Diagnostic data](#2611-diagnostic-data)
      - [2.6.12. Dummy byte and CIR data](#2612-dummy-byte-and-cir-data)
    - [2.7. Device info packet](#27-device-info-packet)
      - [2.7.1. Device type](#271-device-type)
    - [2.8. Device data count packet](#28-device-data-count-packet)
      - [2.8.1. Device data count](#281-device-data-count)

## 1. General network packet format

The general network packet format for the communication between a UWB base station and
a server is defined in the table below.

**_All values are encoded in little-endian by default._**

| Bytes index: |       0       |   1 -- 2    | 3 -- 3 - 1 + n |
| :----------: | :-----------: | :---------: | :------------: |
|   Content    | Packet header | Packet type | Packet payload |
|    Length    |       1       |      2      |       n        |

### 1.1. Packet header

| Packet header value | Description         |
| :-----------------: | ------------------- |
|        0xFD         | Same for all packet |

### 1.2. Packet type

The Packet type part contains two bytes.
The first byte is either `0xCF` or `0xDF`, defining the usage of the packet.
`0xCF` means the packet is used for control, while `0xDF` means the packet is
used for data transferring.
The second byte defines a specific packet type.

Each type of packet has a defined transfer direction, which is either send from
a base station to a server (mark as ←) or send from a server to a base station
(mark with →).

| Packet type value<br>Byte 1 | <br>Byte 2 | Description                                | Transfer direction | Protocol |
| :-------------------------: | :--------: | ------------------------------------------ | :----------------: | :------: |
|            0xCF             |    0x01    | Server open Packet                         |       &rarr;       |   UDP    |
|            0xCF             |    0x02    | Device light control Packet                |       &rarr;       |   TCP    |
|            0xCF             |    0x03    | Device sleep control Packet                |       &rarr;       |   TCP    |
|            0xCF             |    0x04    | Device setting Packet                      |       &rarr;       |   TCP    |
|            0xCF             |    0x05    | Station ready Packet                       |       &larr;       |   UDP    |
|            0xDF             |    0x01    | Device data Packet                         |       &larr;       |   UDP    |
|            0xDF             |    0x02    | Device data with time stamp and CIR Packet |       &larr;       |   UDP    |
|            0xDF             |    0xF1    | Device info Packet                         |       &larr;       |   UDP    |
|            0xDF             |    0xF2    | Device data count Packet                   |       &larr;       |   UDP    |

**_All the control packets should be transfer with TCP,
except server open packet and station ready packet._**

The server open packet is used to notify all base station that server is working
and tell them the address and opened port of server, so it should be broadcast to all base stations.
The station ready packet is the ACK packet of server open packet.
Other control packets only transfer between main base station and sub base station,
so the **_TCP connection only established between server and main base station_**.

## 2. Format of individual packet type

### 2.1. Server open packet

| Bytes index: |       0       |   1 -- 2    |      3 -- 4      |         5         |
| :----------: | :-----------: | :---------: | :--------------: | :---------------: |
|   Content    | Packet header | Packet type |   Server port    | Enable diagnostic |
|    value     |     0xFD      |   0xCF01    | 2 bytes variable | 1 bytes variable  |

This packet informs all base stations that the server could be access now
and give the open port of server to receive data packet form base stations.

**_This packet that send form server to base station should be boardcast to all
base stations, says sending to address like `192.168.0.255`._**

**_The port for all base stations receiving data from server is 8082._**

#### 2.1.1. About port

All ports are defined to store data in little endian form occupying two bytes.
For example, the port of a server opening at 8086, which is `0x1F96` in Hex,
would be send as `0x96 0x1F`.

#### 2.1.2. Enable diagnostic

If this byte is `0x00`, the diagnostic function is disabled,
otherwise base stations will output CIR along with IMU data.

### 2.2. Device light control packet

| Bytes index: |       0       |   1 -- 2    |        3         |      4 -- 6       |  7 -- 7 - 1 + n  |
| :----------: | :-----------: | :---------: | :--------------: | :---------------: | :--------------: |
|   Content    | Packet header | Packet type |  Blink interval  | Color code in RGB |  Device ID list  |
|    value     |     0xFD      |   0xCF05    | 1 bytes variable | 3 bytes variable  | n bytes variable |

Control the RGB LED on devices in the device ID list.

#### 2.2.1. Blink interval

| Blink interval value | Description              |
| :------------------: | ------------------------ |
|         0x00         | LED off                  |
|      0x01-0xFE       | Blink interval $100x$ ms |
|         0xFF         | LED on without blink     |

This byte control the blink interval of LED.

#### 2.2.2. Color code in RGB

| Bytes index: |   0   |   1   |   2   |
| :----------: | :---: | :---: | :---: |
|   Content    |  Red  | Green | Blue  |

Based on the color will infect the brightness of the RGB LED.
The color should not be larger than 255.

#### 2.2.3. Device ID

The device ID is a 1 byte short ID of UWB device from 0x00 to 0xFF, device ID will be dynamicly allocate to UWB device when it connect to the UWB network.

### 2.3. Device sleep control packet

| Bytes index: |       0       |   1 -- 2    |          3           |  4 -- 4 - 1 + n  |
| :----------: | :-----------: | :---------: | :------------------: | :--------------: |
|   Content    | Packet header | Packet type | Device sleep control |  Device ID list  |
|    value     |     0xFD      |   0xCF06    |   1 bytes variable   | n bytes variable |

Control the sleep and wake up behavior of device.

#### 2.3.1. Device sleep control

|  Bits:  |                1                |          2           |          3           |           4           |    5     |    6     |    7     |    8     |
| :-----: | :-----------------------------: | :------------------: | :------------------: | :-------------------: | :------: | :------: | :------: | :------: |
| Content | Auto sleep between transmission | Auto sleep when idle | Auto sleep when rest | Auto wakeup when move | Reserved | Reserved | Reserved | Reserved |

This byte define the sleep and wake up related behavior of a device.
A `0` bit for disable the function, while a `1` bit for enable the function.

|            Function             | Description                                                                                        | Default value |
| :-----------------------------: | -------------------------------------------------------------------------------------------------- | :-----------: |
| Auto sleep between transmission | The device will enter sleep mode once it successfully transmit a data frame to base stations       |       1       |
|      Auto sleep when idle       | Device will enter sleep when it do not receive any beacon or receive the same beacon for 60 second |       0       |
|      Auto sleep when rest       | Device will enter sleep when it not move for 60 seconds                                            |       0       |
|      Auto wakeup when move      | Device will be waken up from sleep when it is moved                                                |       0       |

##### 2.3.1.1. Auto sleep between transmission

For example, if the sample interval is set to 10 ms, the device will enter sleep for around 8 ms
and use 1ms to wakeup and then transmit the next data frame, which will largely decrease the power consumption.
This configuration only works for sample interval larger than 3ms because it will takes around 1.5 ms to wakeup the device.

### 2.4. Device setting packet

| Bytes index: |       0       |   1 -- 2    |                 3                  |      4 -- 5      |  6 -- 6 - 1 + n  |
| :----------: | :-----------: | :---------: | :--------------------------------: | :--------------: | :--------------: |
|   Content    | Packet header | Packet type | Beacon order and Super frame order | IMU sample rate  |  Device ID list  |
|    value     |     0xFD      |   0xCF04    |          1 bytes variable          | 2 bytes bariable | n bytes variable |

This packet sets the beacon order (the beacon sending interval of main base
station), the super frame order (the UWB sample interval of device) and IMU sample rate for the
devices in device ID list.
The device ID list could be empty to set all device at the same time.

#### 2.4.1. Beacon order and Super frame order

This part is actually pack 2 parts of setting in one byte based on the IEEE
802.15.4 specification.

|  Bits:  |       1 -- 4       |       5 -- 8       |
| :-----: | :----------------: | :----------------: |
| Content |    Beacon order    | Super frame order  |
|  value  | 4 bits of variable | 4 bits of variable |

| Beacon order value |         Description         |
| :----------------: | :-------------------------: |
|        0x0         | Send as the sample interval |
|        0x1         |          1 second           |
|        0x2         |          2 second           |
|        0x3         |          3 second           |
|        0x4         |          4 second           |
|        0x5         |          5 second           |
|        0x6         |          6 second           |
|        0x7         |          7 second           |
|        0x8         |          8 second           |
|        0x9         |          9 second           |
|        0xA         |          10 second          |
|        0xB         |          30 second          |
|        0xC         |         1 miniutes          |
|        0xD         |         5 miniutes          |
|        0xE         |         10 miniutes         |
|        0xF         |         30 miniutes         |

| Super frame order value | Description |
| :---------------------: | :---------: |
|           0x0           |   500 us    |
|           0x1           |    1 ms     |
|           0x2           |    2 ms     |
|           0x3           |    3 ms     |
|           0x4           |    4 ms     |
|           0x5           |    5 ms     |
|           0x6           |    6 ms     |
|           0x7           |    7 ms     |
|           0x8           |    8 ms     |
|           0x9           |    9 ms     |
|           0xA           |    10 ms    |
|           0xB           |    20 ms    |
|           0xC           |    50 ms    |
|           0xD           |   100 ms    |
|           0xE           |   200 ms    |
|           0xF           |   1000 ms   |

#### 2.4.2. IMU sample rate

These two bytes is used to set the sample rate of IMU on device, if the configured sample rate is not supported, the device will pick a sample rate lower than this value.

### 2.5. Station ready packet

| Bytes index: |       0       |   1 -- 2    |         3         |      4 -- 5       |
| :----------: | :-----------: | :---------: | :---------------: | :---------------: |
|   Content    | Packet header | Packet type | Base station type | Base station port |
|    value     |     0xFD      |   0xCF07    | 1 bytes variable  | 2 bytes variable  |

Tell the server that the base station is available for data transmission and
notify the port base station use to send data.

#### 2.5.1. Base station type

| Base station type value | Description       |
| :---------------------: | ----------------- |
|          0x0F           | Main base station |
|          0xF0           | Sub base station  |

The type of this base station.

### 2.6. Device data packet for 6 axis IMU

| Bytes index: |       0       |   1 -- 2    |      3 -- 4      |        5         |        6         |           7           |         8         |           9            |        10        |          11          | 12 -- 12 - 1 + n | 12 + n -- 12 + n - 1 + m |
| :----------: | :-----------: | :---------: | :--------------: | :--------------: | :--------------: | :-------------------: | :---------------: | :--------------------: | :--------------: | :------------------: | :--------------: | :----------------------: |
|   Content    | Packet header | Packet type | Base station ID  |     Frame ID     |    Device ID     | IMU data sample count | IMU data bitwidth | IMU timestamp bitwidth | IMU data contain | Station data contain |   Device data    |       Station data       |
|    value     |     0xFD      |   0xDF01    | 2 bytes variable | 1 bytes variable | 1 bytes variable |   1 bytes variable    | 1 bytes variable  |    1 bytes variable    | 1 bytes variable |   1 bytes variable   | n bytes variable |     n bytes variable     |

Send the data received by the base station to server.

#### 2.6.1. Base station ID

The ID of base station, takes the last two bytes of station's MAC address.

#### 2.6.2. Frame ID

It is a one-byte Frame ID that keep increase from `0x00` to `0xFF`.
For one sample interval, Frame ID should be same for all devices.

#### 2.6.3. IMU data sample count

The number of IMU data in this frame.

#### 2.6.4. IMU data bitwidth

The bitwidth of IMU data.

#### 2.6.5. IMU timestamp bitwidth

The bitwidth of IMU timestamp.

#### 2.6.6. IMU data contain

|  Bits:  |      1       |        2         |      3      |     4     |    5     |        6         |    7     |    8     |
| :-----: | :----------: | :--------------: | :---------: | :-------: | :------: | :--------------: | :------: | :------: |
| Content | Acceleration | Angular velocity | Temperature | Timestamp | Reserved | ReservedReserved | Reserved | Reserved |

Define the types of data from IMU that contains in this packet,
0 means not containd, 1 means contained.

#### 2.6.7. Station data contain

|  Bits:  |     1     |     2      |   3   |    4     |    5     |    6     |    7     |    8     |
| :-----: | :-------: | :--------: | :---: | :------: | :------: | :------: | :------: | :------: |
| Content | Timestamp | Diagnostic |  CIR  | Reserved | Reserved | Reserved | Reserved | Reserved |

Define the types of data from base station that contains in this packet,
0 means not containd, 1 means contained.

#### 2.6.8. Device data

| Bytes length: | IMU data bitwidth / 8 * 3 | IMU data bitwidth / 8 * 3 | IMU data bitwidth / 8 | IMU timestamp bitwidth / 8 |
| :-----------: | :-----------------------: | :-----------------------: | :-------------------: | :------------------------: |
|    Content    |       Acceleration        |     Angular velocity      |      Temperature      |         Timestamp          |

The device data contains different types of data.

For 3 axis data:

| Bytes length: | IMU data bitwidth / 8 | IMU data bitwidth / 8 | IMU data bitwidth / 8 |
| :-----------: | :-------------------: | :-------------------: | :-------------------: |
|    Content    |        data x         |        data y         |        data z         |

**_This part will repeat for IMU data sample count times._**

#### 2.6.9. data decode

For example, the data bitwidth is 16:

| Bytes index: |   0    |   1    |
| :----------: | :----: | :----: |
|   Content    | data L | data H |

To process the data, you need to convert 2 bytes into integer then multiply the
range of sensor.

```c
((int) dataH << 8 | dataL) * resolution
```

Or if the bitwidth is 32:

| Bytes index: |   0    |   1    |   2    |   3    |
| :----------: | :----: | :----: | :----: | :----: |
|   Content    | data 0 | data 1 | data 2 | data 3 |

To process timestamp:

```c
(data 3 << 24 | data 2 << 16 | data 1 << 8 | data 0) * resolution
```

| Sensor type |    Data type     |          Resolution          |         unit         |
| :---------: | :--------------: | :--------------------------: | :------------------: |
|   BMI270    |   Acceleration   |  `16 / 1 << (bitwidth - 1)`  |     $\mathrm{G}$     |
|   BMI270    | Angular velocity | `2000 / 1 << (bitwidth - 1)` | $\mathrm{\degree/s}$ |
|   BMI270    |   Temperature    |         `1 / 1 << 9`         |     $\mathrm{K}$     |
|   BMI270    |    Timestamp     |          `39.0625`           |   $\mathrm{\mu s}$   |
|   DW3000    |    Timestamp     |           `15.65`            |    $\mathrm{ps}$     |

#### 2.6.10. Station data

| Bytes length: |        5         |        24         |     1      |        1152         |
| :-----------: | :--------------: | :---------------: | :--------: | :-----------------: |
|    Content    |    Timestamp     |  Diagnostic Data  | Dummy Byte |      CIR data       |
|     value     | 5 bytes variable | 24 bytes variable |   `0x00`   | 1152 bytes variable |

#### 2.6.11. Diagnostic data

| Bytes index: |   0 -- 3   |   4 -- 7    | 8 -- 11  | 12 -- 15 | 16 -- 19 |   20 -- 21    |     22 -- 23     |
| :----------: | :--------: | :---------: | :------: | :------: | :------: | :-----------: | :--------------: |
|   Content    | ipatovPeak | ipatovPower | ipatovF1 | ipatovF2 | ipatovF3 | ipatovFpIndex | ipatovAccumCount |

The diagnostic data of received UWB frame, refer to user manual
[DW3000 User manual](./../%E6%96%87%E6%A1%A3/DW3000-User-Manual.pdf), and
[api guide p80](./../%E6%96%87%E6%A1%A3/DW3000_Software_API_Guide.pdf).

#### 2.6.12. Dummy byte and CIR data

One byte dummy data and CIR data from index 700 to 891,
referring to [api guide p77](./../文档/DW3000_Software_API_Guide.pdf).

### 2.7. Device info packet

| Bytes index: |       0       |   1 -- 2    |        3         |      4 -- 5      |       6 -- 7       |
| :----------: | :-----------: | :---------: | :--------------: | :--------------: | :----------------: |
|   Content    | Packet header | Packet type |    Device ID     |   Device type    | Battery persentage |
|    value     |     0xFD      |   0xDFF1    | 1 bytes variable | 2 bytes variable |  2 bytes variable  |

The base station will inform a server the device info when it connected, or the device
info is updated.

#### 2.7.1. Device type

| Device type value | Description                                                         |
| :---------------: | ------------------------------------------------------------------- |
|      0x0001       | V0.5 UWB-IMU<br>MCU:CH32V203<br>UWB:DW3210<br>Sensor:<br>BMI270 IMU |

This byte indecate the type of device.

### 2.8. Device data count packet

| Bytes index: |       0       |   1 -- 2    |    3 -- 3 - 1 + 3n     |
| :----------: | :-----------: | :---------: | :--------------------: |
|   Content    | Packet header | Packet type | Device data count list |
|    value     |     0xFD      |   0xDFF2    |   3n bytes variable    |

Tell the server how many UWB frames the base station has received.

#### 2.8.1. Device data count

| Bytes index: |     0     |   1 -- 2   |
| :----------: | :-------: | :--------: |
|   Content    | Device ID | Data count |
