# 微软 Azure 物联网平台接入例程

本例程演示如何使用 RT-Thread 提供的 azure-iot-sdk 软件包接入微软物联网平台。初次使用微软物联网平台或者想了解更多使用例程，请阅读[《Azure 云平台软件包用户手册》](https://www.rt-thread.org/document/site/submodules/azure-iot-sdk/docs/introduction/)。

## 简介

Azure 是 RT-Thread 移植的用于连接微软  Azure IoT 中心的软件包，原始 SDK 为：[**azure-iot-sdk-c**](https://github.com/Azure/azure-iot-sdk-c/tree/5299d0010226fdf2dabb3ac3c1f38eabe4500986)。通过该软件包，可以让运行 RT-Thread 的设备轻松接入 Azure IoT 中心。

Azure IoT 中心运行在微软云服务器上，充当中央消息中心，用于 IoT 应用程序与其管理的设备之间的双向通信。 通过 Azure IoT 中心，可以在数百万 IoT 设备和云托管解决方案后端之间建立可靠又安全的通信，生成 IoT 解决方案。几乎可以将任何设备连接到 IoT 中心。

使用 Azure 软件包连接 IoT 中心可以实现如下功能：

- 轻松连入 Azure IoT 中心，建立与 Azure IoT 的可靠通讯
- 为每个连接的设备设置标识和凭据，并帮助保持云到设备和设备到云消息的保密性
- 管理员可在云端大规模地远程维护、更新和管理 IoT 设备
- 从设备大规模接收遥测数据
- 将数据从设备路由到流事件处理器
- 设备上传文件到 IoT 中心
- 将云到设备的消息发送到特定设备

可以使用 Azure IoT 中心来实现自己的解决方案后端。 此外，IoT 中心还包含标识注册表，可用于预配设备、其安全凭据以及其连接到 IoT 中心的权限。

## 硬件说明

Azure 例程需要依赖 IoT Board 板卡上的 WiFi 模块完成网络通信，因此请确保硬件平台上的 WiFi 模组可以正常工作。

## 软件说明

azure 例程位于 `examples/34_iot_cloud_ms_azure` 目录下，重要文件摘要说明如下所示：

| 文件                         | 说明   |
| :-----                       | :-----    |
| applications/main.c     | app 入口 |
| ports                   | 移植文件 |
| packages/azure-iot-sdk-v1.2.8 | azure物联网平台接入软件包 |
| packages/azure-iot-sdk-v1.2.8/samples | azure物联网平台接入示例 |

### 准备工作

在运行本次示例程序之前需要准备工作如下：

1、注册微软 Azure 账户，如果没有 Azure 订阅，请在开始前创建一个[试用帐户](https://www.azure.cn/pricing/1rmb-trial)。 

2、安装 DeviceExplorer 工具，这是一个 windows 平台下测试 Azure 软件包功能必不可少的工具。该工具的安装包为 packages/azure-iot-sdk-v1.2.8/tools 目录下的 SetupDeviceExplorer.msi，按照提示安装即可，成功运行后的界面如下图。

![DeviceExplorer 工具界面](../../docs/figures/34_iot_cloud_ms_azure/setdeviceexplorer.png)

#### 通信协议介绍

目前 RT-Thread Azure 软件包提供的示例代码支持 MQTT 和 HTTP 的通信协议，想要使用哪种协议，只需要在上面选项中选择相应的协议即可。在选择设备端通信协议时，需要注意以下几点：

1、当进行云到设备数据发送时，由于HTTPS 没有用于实现服务器推送的有效方法。 因此，使用 HTTPS 协议时，设备会在 IoT 中心轮询从云到设备的消息。 此方法对于设备和 IoT 中心而言是低效的。 根据当前 HTTPS 准则，每台设备应每 25 分钟或更长时间轮询一次消息。MQTT 支持在收到云到设备的消息时进行服务器推送。 它们会启用从 IoT 中心到设备的直接消息推送。 如果传送延迟是考虑因素，最好使用 MQTT协议。 对于很少连接的设备，HTTPS 也适用。

2、使用 HTTPS 时，每台设备应每 25 分钟或更长时间轮询一次从云到设备的消息。 但在开发期间，可按低于 25 分钟的更高频率进行轮询。 

3、更详细的协议选择文档请参考 Azure 官方文档[《选择通信协议》](https://docs.azure.cn/zh-cn/iot-hub/iot-hub-devguide-protocols)。

#### 创建 IoT 中心

首先要做的是使用 Azure 门户在订阅中创建 IoT 中心。 IoT 中心用于将大量遥测数据从许多设备引入到云中。 然后，该中心会允许一个或多个在云中运行的后端服务读取和处理该遥测数据。 

1、登录到 [Azure 门户](http://portal.azure.cn/)。

2、选择“创建资源” > “物联网” > “IoT 中心”。

![创建物联网中心](../../docs/figures/34_iot_cloud_ms_azure/create_iot_hub.png)

3、在 “IoT 中心” 窗格中，输入 IoT 中心的以下信息：

   - **Subscription（订阅）**：选择需要将其用于创建此 IoT 中心的订阅。
   - **Resource Group（资源组）**：创建用于托管 IoT 中心的资源组，或使用现有的资源组，在这个栏目中填入一个合适的名字就可以了。 有关详细信息，请参阅[使用资源组管理 Azure 资源](https://docs.azure.cn/zh-cn/azure-resource-manager/resource-group-portal)。
   - **Region（区域）**：选择最近的位置。
   - **IoT Hub Name（物联网中心名称）**：创建 IoT 中心的名称，这个名称需要是唯一的。 如果输入的名称可用，会显示一个绿色复选标记。

![填写 IoT 中心资料](../../docs/figures/34_iot_cloud_ms_azure/create_iot_hub_name.png)

4、选择“下一步: **Size and scale（大小和规模）**”，以便继续创建 IoT 中心。

5、选择“**Pricing and scale tier（定价和缩放层）**”。 就测试用途来说，请选择“**F1:Free tier（F1 - 免费）**”层（前提是此层在订阅上仍然可用）。 有关详细信息，请参阅[定价和缩放层](https://www.azure.cn/pricing/details/iot-hub/)。

![选择功能层](../../docs/figures/34_iot_cloud_ms_azure/select_f1_layer.png)

6、选择“**Review + create（查看 + 创建）**”。

7、查看 IoT 中心信息，然后单击“创建”即可。 创建 IoT 中心可能需要数分钟的时间。 可在“**通知**”窗格中监视进度，创建成功后就可以进行下一步注册设备的操作了。

8、为了后续方便查找，可以手动将创建成功后的资源添加到**仪表盘**。

#### 注册设备

要想运行设备端相关的示例，需要先将设备信息注册到 IoT 中心里，然后该设备才能连接到 IoT 中心。 在本次示例中，可以使用 DeviceExplorer 工具来注册设备。

- 获得 IoT 中心的**共享访问密钥**（即IoT 中心连接字符串）

1、IoT 中心创建完毕后，在设置栏目中，点击共享访问策略选项，可以打开 IoT 中心的访问权限设置。打开 iothubowner，在右侧弹出的属性框中获得 IoT 中心的共享访问密钥。

![查看物联网中心共享访问策咯](../../docs/figures/34_iot_cloud_ms_azure/share_access.png)

2、在右侧弹出的属性框中获取 IoT 中心连接字符串：

![复制访问主密钥](../../docs/figures/34_iot_cloud_ms_azure/get_connect_string.png)

- 创建设备

1、有了连接字符串后，我们便可以使用 DeviceExplorer 工具来创建设备，并测试 IoT 中心的功能了。打开测试工具，在配置选项中填入的连接字符串。点击 `update` 按钮更新本地连接  IoT 中心的配置，为下一步创建测试设备做准备。

![配置 DeviceExplorer 工具](../../docs/figures/34_iot_cloud_ms_azure/update_device_explorer.png)

2、打开 Management 选项栏，按照下图所示的步骤来创建测试设备。设备创建成功后，就可以运行设备的功能示例了。

![创建设备](../../docs/figures/34_iot_cloud_ms_azure/management_device.png)

![创建设备成功](../../docs/figures/34_iot_cloud_ms_azure/create_device_done.png)

## 运行

### 编译&下载

- **MDK**：双击 `project.uvprojx` 打开 MDK5 工程，执行编译。
- **IAR**：双击 `project.eww` 打开 IAR 工程，执行编译。

编译完成后，将开发板与 PC 机连接，然后将固件下载至开发板。

程序运行日志如下所示：

```text
 \ | /
- RT -     Thread Operating System
 / | \     4.0.1 build May 30 2019
 2006 - 2019 Copyright by rt-thread team
lwIP-2.0.2 initialized!
[SFUD] Find a Winbond flash chip. Size is 16777216 bytes.
[SFUD] w25q128 flash device is initialize success.
[I/sal.skt] Socket Abstraction Layer initialize success.
[E/main] 1005
[D/FAL] (fal_flash_init:61) Flash device |              w60x_onchip | addr: 0x08000000 | len: 0x00100000 | blk_size: 0x00001000 |initialized finish.
[D/FAL] (fal_flash_init:61) Flash device |                 norflash | addr: 0x00000000 | len: 0x01000000 | blk_size: 0x00001000 |initialized finish.
[D/FAL] (fal_partition_init:176) Find the partition table on 'w60x_onchip' offset @0x0000f0c8.
[I/FAL] ==================== FAL partition table ====================
[I/FAL] | name       | flash_dev   |   offset   |    length  |
[I/FAL] -------------------------------------------------------------
[I/FAL] | easyflash  | norflash    | 0x00000000 | 0x00100000 |
[I/FAL] | app        | w60x_onchip | 0x00010100 | 0x000ed800 |
[I/FAL] | download   | norflash    | 0x00100000 | 0x00100000 |
[I/FAL] | font       | norflash    | 0x00200000 | 0x00700000 |
[I/FAL] | filesystem | norflash    | 0x00900000 | 0x00700000 |
[I/FAL] =============================================================
[I/FAL] RT-Thread Flash Abstraction Layer (V0.3.0) initialize success.
[Flash] (packages\EasyFlash-v3.3.0\src\ef_env.c:152) ENV start address is 0x00000000, size is 4096 bytes.
[Flash] (packages\EasyFlash-v3.3.0\src\ef_env.c:821) Calculate ENV CRC32 number is 0x12D2A372.
[Flash] (packages\EasyFlash-v3.3.0\src\ef_env.c:833) Verify ENV CRC32 result is OK.
[Flash] EasyFlash V3.3.0 is initialize success.
[Flash] You can get the latest version on https://github.com/armink/EasyFlash .
[I/WLAN.dev] wlan init success
[I/WLAN.lwip] eth device init ok name:w0
msh />[I/WLAN.mgnt] wifi connect success ssid:aptest
[I/WLAN.lwip] Got IP address : 192.168.12.26     # 成功自动连接wifi
```

### 连接无线网络

 如果没有连接网络，程序运行后会进行 MSH 命令行，等待用户配置设备接入网络。使用 MSH 命令 `wifi join ssid key` 配置网络，如下所示：

```shell
msh />wifi join ssid_test router_key_xxx
join ssid:ssid_test
[I/WLAN.mgnt] wifi connect success ssid:ssid_test
msh />[I/WLAN.lwip] Got IP address : 152.10.200.224
```

### 运行效果

#### 功能示例一：设备发送遥测数据到物联网中心

**示例文件**

| 示例程序路径                   | 说明      |
| ----                          | ---          |
| samples/iothub_ll_telemetry_sample.c | 从设备端发送遥测数据到 Azure IoT 中心 |

**云端监听设备数据**

- 打开测试工具的 Data 选项栏，选择需要监听的设备，开始监听：

![监听设备遥测数据](../../docs/figures/34_iot_cloud_ms_azure/monitor_device.png)

**修改示例代码中的设备连接字符串**

1、在运行测试示例前需要获取设备的连接字符串。

![获取设备连接字符串](../../docs/figures/34_iot_cloud_ms_azure/get_device_connect_string.png)

2、将连接字符串填入测试示例中的 connectionString 字符串中，重新编译程序，下载到开发板中。

![填入设备连接字符串](../../docs/figures/34_iot_cloud_ms_azure/fillin_device_connect_string.png)

**运行示例程序**

1、在 msh 中运行 azure_telemetry_sample 示例程序：

```c
msh />azure_telemetry_sample
msh />
ntp init
Creating IoTHub Device handle
Sending message 1 to IoTHub
-> 11:46:58 CONNECT | VER: 4 | KEEPALIVE: 240 | FLAGS: 192 | 
USERNAME:
xxxxxxxxxx.azuredevices.cn/devicexxxxxxx9d64648f19e97013bec7ab453
/?api-version=2017-xx-xx-preview&
DeviceClientType=iothubclient%2f1.2.8%20
(native%3b%20xxxxxxxx%3b%20xxxxxx) | PWD: XXXX | CLEAN: 0
<- 11:46:59 CONNACK | SESSION_PRESENT: true | RETURN_CODE: 0x0
The device client is connected to iothub
Sending message 2 to IoTHub
Sending message 3 to IoTHub
-> 11:47:03 PUBLISH | IS_DUP: false | RETAIN: 0 | QOS: DELIVER_AT_LEAST_ONCE |
    TOPIC_NAME: devices/device0f134df9d64648f19e97013bec7ab453/messages/events
    /hello=RT-Thread | PACKET_ID: 2 | PAYLOAD_LEN: 12
-> 11:47:03 PUBLISH | IS_DUP: false | RETAIN: 0 | QOS: DELIVER_AT_LEAST_ONCE | 
    TOPIC_NAME: devices/device0f134df9d64648f19e97013bec7ab453/messages/events
    /hello=RT-Thread | PACKET_ID: 3 | PAYLOAD_LEN: 12
-> 11:47:03 PUBLISH | IS_DUP: false | RETAIN: 0 | QOS: DELIVER_AT_LEAST_ONCE |
    TOPIC_NAME: devices/device0f134df9d64648f19e97013bec7ab453/messages/events
    /hello=RT-Thread | PACKET_ID: 4 | PAYLOAD_LEN: 12
<- 11:47:04 PUBACK | PACKET_ID: 2
Confirmation callback received for message 1 with result IOTHUB_CLIENT_CONFIRMATION_OK
<- 11:47:04 PUBACK | PACKET_ID: 3
Confirmation callback received for message 2 with result IOTHUB_CLIENT_CONFIRMATION_OK
<- 11:47:04 PUBACK | PACKET_ID: 4
Confirmation callback received for message 3 with result IOTHUB_CLIENT_CONFIRMATION_OK
Sending message 4 to IoTHub
-> 11:47:06 PUBLISH | IS_DUP: false | RETAIN: 0 | QOS: DELIVER_AT_LEAST_ONCE | 
    TOPIC_NAME: devices/device0f134df9d64648f19e97013bec7ab453/messages/events
    /hello=RT-Thread | PACKET_ID: 5 | PAYLOAD_LEN: 12
<- 11:47:07 PUBACK | PACKET_ID: 5
Confirmation callback received for message 4 with result IOTHUB_CLIENT_CONFIRMATION_OK
Sending message 5 to IoTHub
-> 11:47:09 PUBLISH | IS_DUP: false | RETAIN: 0 | QOS: DELIVER_AT_LEAST_ONCE | 
    TOPIC_NAME: devices/device0f134df9d64648f19e97013bec7ab453/messages/events
    /hello=RT-Thread | PACKET_ID: 6 | PAYLOAD_LEN: 12
<- 11:47:10 PUBACK | PACKET_ID: 6
Confirmation callback received for message 5 with result IOTHUB_CLIENT_CONFIRMATION_OK
-> 11:47:14 DISCONNECT
Error: Time:Tue Jul 31 11:47:14 2018 File:packages\azure\azure-port\pal\src\
socketio_berkeley.c Func:socketio_send Line:853 
Failure: socket state is not opened.
The device client has been disconnected
Azure Sample Exit
```

2、此时可在 DeviceExplorer 工具的 Data 栏查看设备发到云端的遥测数据：

![收到遥测数据](../../docs/figures/34_iot_cloud_ms_azure/get_data_from_device.png)

示例运行成功，在 DeviceExplorer 工具中看到了设备发送到物联网中心的 5 条遥测数据。

#### 功能示例二：设备监听云端下发的数据

**示例文件**

| 示例程序路径                   | 说明      |
| ----                          | ---          |
| samples/iothub_ll_c2d_sample.c | 在设备端监听 Azure IoT 中心下发的数据 |

**修改示例代码中的设备连接字符串**

- 与上面的示例相同，本示例程序也需要填写正确的设备连接字符串，修改完毕后重新编译程序，下载到开发板中即可。修改内容如下所示：

![修改设备连接字符串](../../docs/figures/34_iot_cloud_ms_azure/change_connect_string.png)

**设备端运行示例程序**

- 在 msh 中运行 azure_c2d_sample 示例程序 ，示例程序运行后设备将会等待并接收云端下发的数据：

```c
msh />azure_c2d_sample
msh />
ntp init
Creating IoTHub Device handle          # 等待 IoT 中心的下发数据
Waiting for message to be sent to device (will quit after 3 messages)
```

**服务器下发数据给设备**

1、打开 DeviceExplorer 工具的 Messages To Device 栏向指定设备发送数据：

![服务器下发数据给设备](../../docs/figures/34_iot_cloud_ms_azure/iothub_send_data2device.png)

2、此时在设备端查看从 IoT 中心下发给设备的数据：

```c
msh />azure_c2d_sample
msh />
ntp init
Creating IoTHub Device handle
Waiting for message to be sent to device (will quit after 3 messages)
Received Binary message                            # 收到二进制数据
Message ID: ea02902d-d6d2-4b7f-9f71-0873ca90b7a3
 Correlation ID: <unavailable>
 Data: <<<hello rt-thread>>> & Size=15
Received Binary message
Message ID: 9b2a9693-ddaf-4222-86b6-4ef8e0fc98d7
 Correlation ID: <unavailable>
 Data: <<<hello rt-thread>>> & Size=15
Received Binary message
Message ID: 48f6a50d-d20a-4483-8e06-41aa75b80aa8
 Correlation ID: <unavailable>
 Data: <<<hello rt-thread>>> & Size=15
Received Binary message
Message ID: 3f15a3ba-5d74-45c8-a714-259d64016515
 Correlation ID: <unavailable>
 Data: <<<hello rt-thread>>> & Size=15
Received Binary message
Message ID: 4313021c-1215-4e77-bb8a-c215c9f0ca38
 Correlation ID: <unavailable>
 Data: <<<hello rt-thread>>> & Size=15
Received Binary message
Message ID: 9be87f25-2a6f-46b5-a413-0fb2f93f85d6
 Correlation ID: <unavailable>
 Data: <<<hello rt-thread>>> & Size=15
Error: Time:Tue Jul 31 13:54:14 2018 
File:packages\azure\azure-port\pal\src\socketio_berkeley.c 
Func:socketio_send Line:853 Failure: socket state is not opened.
Azure Sample Exit     #收到一定数量的下发数据，功能示例自动退出
```

## 注意事项

- 使用本例程前请先阅读[《Azure 云平台软件包用户手册》](https://www.rt-thread.org/document/site/submodules/azure-iot-sdk/docs/introduction/)

## 引用参考

- 《Azure 云平台软件包用户手册》: docs/UM1007-RT-Thread-Azure-IoT-SDK 用户手册.pdf
