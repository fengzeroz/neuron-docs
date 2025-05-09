# Modbus TCP

Modbus TCP is a version of the Modbus protocol based on Ethernet, which uses TCP/IP for communication. Unlike the traditional Modbus RTU protocol, Modbus TCP allows devices to be interconnected directly through Ethernet without any special hardware or communication interface. Therefore, Modbus TCP has higher communication speed and wider application range.

In addition to supporting data acquisition and processing via TCP client mode, the Neuron Modbus TCP plugin also supports TCP server mode, which allows devices to connect to Neuron actively. This feature is mainly used for 4G DTU because the IP address of 4G network is a private IP. In this case, the DTU device can only connect to Neuron actively.

## Add Device

Go to **Configuration -> South Devices**, then click **Add Device** to add the driver. Configure the following settings in the popup dialog box.

- Name: The name of this device node.
- Plugin: Select the **Modbus TCP** or **Modbus TCP QH** plugin.

| Plugin | Description |
| --- | --- | 
| **Modbus TCP** |Standard Modbus TCP protocol implementation supports both TCP client and server modes, providing better compatibility with devices. |
| **Modbus TCP QH** | Customized Modbus TCP protocol implementation supports a maximum of 65530 bytes for one read operation, while the standard protocol only allows a maximum of 250 bytes to be read at a time.|

## Device Configuration

After clicking **Create**, you will be redirected to the **Device Configuration** page, where we will set up the parameters required for Neuron to establish a connection with the device. You can also click the device configuration icon on the southbound device card to enter the **Device Configuration** interface.

| Parameter                  | Description                                                    |
| -------------------- | ------------------------------------------------------- |
| **Connection Mode** | Only for **TCP** mode, When selecting TCP, you can choose Neuron as the TCP client or server. |
| **Maximum Retry Times** | The maximum number of retries after a failed attempt to send a read command. |
| **Retry Interval** | Resend reading instruction interval(ms) after a failed attempt to send a read command. |
| **Endianness**    | Byte order of tags with 32 bits, ABCD corresponds to 1234. |
| **Start Address** | Address starts from 1 or 0. |
| **Send Interval** | The waiting time between sending each read/write command. Some serial devices may discard certain commands if they receive consecutive commands in a short period of time. |
| **IP Address** | The IP address of the device when using TCP connection with Neuron as the client, or the IP address of Neuron when using TCP connection with Neuron as the server. The default value is 0.0.0.0. <!--to be confirmed--> |
| **Port** | The port number of the device when using TCP connection with Neuron as the client, or the port number of Neuron when using TCP connection with Neuron as the server.|
| **Connection Timeout** |  The time the system waits for a device to respond to a command. |
| **Check Header** | Choose whether to verify the message header. After selecting True, when encountering packet header errors, the neuron and device will reconnect. |
| **Device Degradation** | Enable or disable device degradation mechanism. |
| **Failure Threshold for Degradation** | The number of consecutive failure cycles required to trigger device degradation. |
| **Recovery Time After Degradation** | The time in seconds after which the device recovers from degradation. |
| **Backup IP Address** | Optional, when the connection is abnormal, it will automatically connect to the backup address. If the backup address connection is abnormal, it will automatically connect to the original address. |
| **Backup Port** | Optional, when the connection is abnormal, it will automatically connect to the backup address. If the backup address connection is abnormal, it will automatically connect to the original address. |

::: tip  
The above configuration can meet the individualized needs of the device:<br>  

1. Retry requests are considered as the same request, i.e., a successful retry is regarded as a successful request.<br>  
2. In the same read cycle, if a certain tag under a slave id does not receive a response, requests for other tags under the same slave id in that cycle will be skipped.<br>  
3. The device degradation is triggered when a certain tag under a slave id fails to respond for multiple consecutive cycles (or can be configured to trigger after just one failed cycle). Once triggered, all requests for that slave will be stopped for the configured period.  
:::

## Configure Data Groups and Tags

After the plugin is added and configured, the next step is to establish communication between your device and Neuron by adding groups and tags to the Southbound driver.

Once device configuration is completed, navigate to the **South Devices** page. Click on the device card or device row to access the **Group List** page. Here, you can create a new group by clicking on **Create**, then specifying the group name and data collection interval.

Upon successfully creating a group, click on its name to proceed to the **Tag List** page. This page allows you to add device tags for data collection. You'll need to provide information such as the tag address, attributes, and data type.

For information on general configuration items, see [Connect to Southbound Devices](../south-devices.md). The subsequent section will concentrate on configurations specific to the driver.

### Data types

* INT16
* UINT16
* INT32
* UINT32
* INT64
* UINT64
* FLOAT
* DOUBLE
* BIT
* STRING
* BYTES

### Address format

> SLAVE!ADDRESS\[.BIT][#ENDIAN]\[.LEN\[H]\[L]\[D]\[E]]\[.BYTES]

#### **SLAVE**

Required, Slave is the slave address or site number.

#### **ADDRESS**

Required, Address is the register address. The Modbus protocol has four areas, each area has a maximum of 65536 registers, and the address range of each area is shown in the table below. It should be noted that a storage area as large as 65536 is generally not required in practical applications. Generally, PLC manufacturers generally use an address range within 10000. Please pay attention to filling in the correct point address according to the area and function code of the device.

| Area                       | Address Range          | Attribute        | Register Size     | Function Code | Data Type|
| ------------------------- | ---------------- | ---------- | ------------- | ------------ | ------- |
| Coil                | 000001 ~ 065536 | Read/Write       | 1Bit          | 0x01, 0x05, 0x0f | BIT     |
| Input          | 100001 ~ 165536 | Read/Write         | 1Bit         | 0x02          | BIT     |
| Input Register| 300001 ~ 365536 | Read/Write         | 16Bit,2Byte         | 0x04          | BIT, INT16, UINT16,<br />INT32, UINT32, INT64,<br />UINT64, FLOAT,<br />DOUBLE, STRING |
| Hold Register  | 400001 ~ 465536 | Read/Write       | 16Bit,2Byte         | 0x03, 0x06, 0x10 | BIT, INT16, UINT16,<br />INT32, UINT32, INT64,<br />UINT64, FLOAT,<br /> DOUBLE, STRING |

#### **.BIT**

Optional, specify a specific bit in a register

| Address         | Data Type | Description                                                |
| ----------- | ------- | --------------------------------------------------- |
| 1!300004.0  | bit     | Refers to station 1, input register area, address 300004, bit 0 |
| 1!400010.4  | bit     | Refers to station 1, hold register area, address 400010, bit 4    |
| 2!400001.15 | bit     | Refers to station 2, hold register area, address 400001, bit 15   |

#### **#ENDIAN**

Optional, byte order, applicable to data types int16/uint16/int32/uint32/float, see the table below for details.
| Symbol | Byte Order | Supported Data Types | Note |
| --- | ------- | ------------------ | ----- |
| #B  | 2,1     | int16/uint16       |       |
| #L  | 1,2     | int16/uint16       | Default byte order if not specified |
| #LL | 1,2,3,4 | int32/uint32/float | Default byte order if not specified |
| #LB | 2,1,4,3 | int32/uint32/float | |
| #BL | 3,4,1,2 | int32/uint32/float | |
| #BB | 4,3,2,1 | int32/uint32/float | |

::: tip
The byte order of a tag has a higher priority than the byte order configuration of a node. That is to say, once the byte order is configured for a tag, it follows the configuration of that tag and ignores the node configuration.
The byte order can be illustrated using the notation ABCD, which corresponds directly to the sequence 1234. As an example, the ABCD designation represents the standard or default Endianness 1234. (#LL).
:::

#### .LEN\[H]\[L]\[D]\[E]

When the data type is STRING, `.LEN` is a required field, indicating the number of bytes the string occupies. Each register contains four storage methods: H, L, D, and E, as shown in the table below.
| Symbol | Description                                 |
| --- | ------------------------------------- |
| H   | One register stores two bytes, with the high byte first |
| L   | One register stores two bytes, with the low byte first |
| D   | One register stores one byte, and it is stored in the low byte      |
| E   | One register stores one byte, and it is stored in the high byte|

#### **.BYTES**

Optional, read and write the length of bytes type data, applicable to bytes data type.

::: tip
A register of the Modbus driver contains 2 bytes. When reading and writing Modbus register data in the bytes data type, please ensure that the bytes parameter is set to an even number.
:::

### Example Addresses

| Address        | Data Type | Description |
| ----------- | ------- | --------- |
| 1!300004    | int16    | Refers to station 1, input register area, address 300004, byte order #L |
| 1!300004#B  | int16    | Refers to station 1, input register area, address 300004, byte order #B |
| 1!300004#L  | uint16   | Refers to station 1, input register area, address 300004, byte order #L |
| 1!400004    | int16    | Refers to station 1, hold register area, address 400004, byte order #L |
| 1!400004#L  | int16    | Refers to station 1, hold register area, address 400004, byte order #L |
| 1!400004#B  | uint16   | Refers to station 1, hold register area, address 400004, byte order #B |
| 1!300004    | int32    | Refers to station 1, input register area, address 300004, byte order #LL |
| 1!300004#BB | uint32   | Refers to station 1, input register area, address 300004, byte order #BB |
| 1!300004#LB | uint32   | Refers to station 1, input register area, address 300004, byte order #LB |
| 1!300004#BL | float    | Refers to station 1, input register area, address 300004, byte order #BL |
| 1!300004#LL | int32    | Refers to station 1, input register area, address 300004, byte order #LL |
| 1!400004    | int32    | Refers to station 1, hold register area, address 400004, byte order #LL|
| 1!400004#LB | uint32   | Refers to station 1, hold register area, address 400004, byte order #LB|
| 1!400004#BB | uint32   | Refers to station 1, hold register area, address 400004, byte order #BB|
| 1!400004#LL | int32    | Refers to station 1, hold register area, address 400004, byte order #LL|
| 1!400004#BL | float    | Refers to station 1, hold register area, address 400004, byte order #BL|
| 1!300001.10  | String  | Refers to station 1, input register area, address 300001, character length 10, byte order L, which occupies addresses 300001 to 300005|
| 1!300001.10H | String  | Refers to station 1, input register area, address 300001, character length 10, byte order H, which occupies addresses 300001 to 300005|
| 1!300001.10L | String  | Refers to station 1, input register area, address 300001, character length 10, byte order L, which occupies addresses 300001 to 300005|
| 1!400001.10  | String  | Refers to station 1, hold register area, address 400001, character length 10, byte order L, which occupies addresses 400001 to 400005|
| 1!400001.10H | String  | Refers to station 1, hold register area, address 400001, character length 10, byte order H, which occupies addresses 400001 to 400005|
| 1!400001.10L | String  | Refers to station 1, hold register area, address 400001, character length 10, byte order L, which occupies addresses 400001 to 400005|
| 1!400001.10D | String  | Refers to station 1, hold register area, address 300001, character length 10, byte order D, which occupies addresses 400001 to 400005|
| 1!400001.10E | String  | Refers to station 1, hold register area, address 300001, character length 10, byte order E, which occupies addresses 400001 to 400005|

## Use Case

This chapter also provides practical examples to facilitate a quick start.

- [Modbus Slave Simulator](./example/modbus-slave/modbus-slave.md)

## Data Monitoring

After completing the point configuration, you can click **Monitoring** -> **Data Monitoring** to view device information and control devices. For details, refer to [Data Monitoring](../../../admin/monitoring.md).
