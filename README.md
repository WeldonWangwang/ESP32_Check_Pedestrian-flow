# 基于 ESP32 和 OneNET 的人流量检测及数据云端统计  

> TAG: esp32; onenet; wifi; mqtt; 流量检测  
> 
> 本工程源代码托管位置：https://github.com/WeldonWangwang/ESP32_Check_Pedestrian-flow  
> 
> 人流量监控在安保，商场，旅游等诸多行业具有重要作用，人流量数据不仅具有科学研究价值,还具有很强的实用价值。因此选择一种便捷，经济的监控手段很有必要。

ESP32 是乐鑫信息科技设计的集成 2.4 GHz Wi-Fi 和蓝牙双模的单芯片方案,采用台积电(TSMC)超低功耗的 40 纳米工艺,拥有
最佳的功耗性能、射频性能、稳定性、通用性和可靠性,适用于各种应用和不同功耗需求。  

![](https://i.imgur.com/Jsd19Wo.png)

我们利用 ESP32  Wi-Fi 的混杂接收模式，接受全部可以获得空中包，然后对其解析，筛选得到周围无线设备发送的 Probe Request 帧，通过对 Probe Request 帧的来源和强度进行分析和汇总，从而计算出周围一定区域内的设备设备使用量（人流量）。  

在得到基本的人流量数据后，ESP32 通过 MQTT 将数据发送至 OneNET 物联网平台，得到最终的人流量变化曲线图，以从云端方便的进行数据处理和监控。

## 1. 硬件准备  

- ESP32: 可以参考乐鑫的官方资料册，[ESP32-DevKitC 开发板](http://esp-idf.readthedocs.io/en/latest/hw-reference/modules-and-boards.html#esp32-core-board-v2-esp32-devkitc) 或 [ESP-WROVER-KIT 开发板](http://esp-idf.readthedocs.io/en/latest/hw-reference/modules-and-boards.html#esp-wrover-kit)

- 路由器/ Wi-Fi 热点：可以连接外网  

## 2.  快速开始  

### 2.1 ESP32 Wi-Fi 联网配置   
运行 `make menuconfig` -> Demo Configuration -> 输入热点的 Wi-Fi SSID & Wi-Fi Password  

![](https://i.imgur.com/UOA8dm4.png)  

### 2.2 OneNET 创建设备及应用  

- 产品注册  
登录 OneNET 官网，进行设备注册。点击“开发者中心”，进入相应的“产品列表”管理页面，在这里您可以新建并管理您的产品；  
点击右上角的 “创建产品”，在弹出页面中按照提示填写产品的基本信息，进行产品创建；  

![](https://i.imgur.com/B7u07Ks.png)

操作系统选择“其他 -> freertos”，网络运营商选择“其他”，联网方式选择“Wi-Fi”.

  在创建过程最后一步，系统会提示让您选择“设备接入方   式”和“设备接入协议”， 协议选择“公开协议 -> MQTT”，点击“确定”按钮，完成产品创建。   
  
![](https://i.imgur.com/FxMLIj1.png) 

- 创建设备  

 在设备管理页面选择“添加设备”， 输入设备名称和鉴权信息，鉴权信息可以自己定义，相当于通讯双方都知道的“暗号”。  
 
![](https://i.imgur.com/zuGlXgn.png)

 接入设备后可以获得设备的 ID 等设备信息。

- 创建应用   

 设备的应用可以更好的对设备上传的数据进行统计和展示，点击创建应用，选择独立应用，选择产品及名称，点击创建。  
 
![](https://i.imgur.com/XIGPsDs.png)

因为我们需要展示人流量的变化数据，因此在这里选择“折线图”，然后对坐标轴进行设置，得到一个大体的应用框架，在这时还不能对数据流进行选择，因为我们还没有开始进行数据上报，点击保存，则可以在“应用管理中看到刚刚创建的应用”。  

![](https://i.imgur.com/iHAKOzi.png)

### 2.3 程序中设备信息填充  

 在程序中的 `main/include/onenet.h` 中将刚刚在平台上创建得到的“设备 ID”，“产品 ID”，“鉴权信息” 进行填充：  

        #define ONENET_DEVICE_ID    "*****"         // mqtt client id
        #define ONENET_PROJECT_ID   "*****"         // mqtt username
        #define ONENET_AUTH_INFO    "*****"         // mqtt password

### 2.4 编译运行  

- 返回顶层目录  
- 执行 `make` 指令, 编译 demo, 命令如下：  

		make flash monitor

 编译成功后, 开始进行烧写。 
 
 运行后打印输出：    
 
    ```
    Connecting to server 183.230.40.39:6002,29207
    Connected!
    Connected to server 183.230.40.39:6002
    Sending MQTT CONNECT message, type: 1, id: 0000
    Reading MQTT CONNECT response message
    Connected
    mqtt_sending_task
    Queuing publish, length: 27, queue size(27/4096)

    Sending...27 bytes
    mqtt_start_receive_schedule

    Current device num = 1
    MAC: 0x**.0x**.0x**.0x**.0x**.0x**, The time is: 7200, The rssi = -73

    Current device num = 2
    MAC: 0x**.0x**.0x**.0x**.0x**.0x**, The time is: 8880, The rssi = -59  
    ...
     ```
 可以看到，ESP32 成功连接到 MQTT，并且已经开始抓包，并且打印得到无线设备的 MAC 地址和 信号强度。  
 
 我们在程序中设置的是每隔十分钟向云端上报依次信息，所以每隔十分钟会调用一次 `mqtt publish`.  

## 3. OneNET 控制台查看人流量数据  

打开 OneNET 控制台，可以看到设备已经在线，点击设备对应的应用，选择数据流“Pedestrian-flow”，则可以看到应用中数据流的变化曲线图。  

![](https://i.imgur.com/6qoYiRh.png)

## 4. ESP32 抓包详解  

### 4.1 Probe Request 包分析  

Probe Request 包属于 802.11 标准，其基本的帧结构如下：   

![](https://i.imgur.com/q4D4Wn9.jpg) 

通过 wireshark 抓包获得 Probe Request 包如下：  
*注： ubuntu 下 wireshark 抓 802.11教程可参考http://ju.outofmemory.cn/entry/204965*

![](https://i.imgur.com/7Lsdu3L.png)

可以从包中看到该帧的 “Subtype : 4”, 还有信号强度和 MAC 地址等信息。

### 4.2 对应程序  

- 在程序中首先对抓到的包包头进行判定，当包头为 Probe Request 包时进行下一步判定，否则直接丢弃。  

        if (sniffer_payload->header[0] != 0x40) {
            return;
        }
        
- 得到真正的 Probe Request 包后，首先对来源设备的 MAC 地址进行分析，过滤掉一些不需要计算的设备（固定无线装置等）。 
-  
        for (int i = 0; i < 32; ++i) {
            if (!memcmp(sniffer_payload->source_mac, esp_module_mac[i], 3)) {
                return;
            }
        }
        
  在程序中我们过滤掉其他 ESP32 芯片的 MAC 地址。  
  
- 重复设备过滤  
在抓到的包中，有相当一部分是来自于同一设备的 Probe Request 包，因此需要剔除。  

        for (station_info = g_station_list->next; station_info; station_info = station_info->next) {
            if (!memcmp(station_info->bssid, sniffer_payload->source_mac, sizeof(station_info->bssid))) {
                return;
            }
        }  
        
- 我们创建一个存储每一个设备信息的链表，在获得一个有效设备信息后，将该设备信息加入设备链表中。  
- 
        if (!station_info) {
            station_info = malloc(sizeof(station_info_t));
            station_info->next = g_station_list->next;
            g_station_list->next = station_info;
        }

## 5. 人流量计算  

首先需要确定计算人流量的区域范围，我们利用获得的设备信号强度进行划分，从而根据选择不同的信号强度来确定需要监控人流量的区域范围。  

但是 RSSI 并不是协议中的字段，802.11协议中没有给出具体产生RSSI的过程或者说是算法吧。协议中说RSSI值范围0~255，RSSI值随PHY Preamble部分的能量单调递增。即在接收端网卡测量到RSSI的值以后，我们可以从这个程序接口获得RSSI的值。因此在不同的实地环境下，要根据实际测定来确定不同接受距离对应的实际 RSSI 值。

由于无线设备的普及，几乎人手都会持有一具有特定 MAC 地址的无线设备，而无线设备又会发送 Probe Request 包，因此只要得到空中的 Probe Request 包就可以计算出该区域的流动人数。   

人流量是指在一定区域内单位时间的人流总数，因此我们以 10 分钟为侦测区间，通过对在 10 分钟内的所有人数进行计算来获得人流量。
在 10 分钟后将链表清空，开始下一阶段的监控。整个统计流程如下：  
![](https://i.imgur.com/aJdu10E.png)  

## 6. ESP32 -> OneNET 间 MQTT 通信  

### 6.2 ESP32 开启 STA 模式

ESP32 在进行抓包的同时需要开启 STA 模式进行联网，联网流程如下：  

![](https://i.imgur.com/Nu4OMhF.png)

### 6.2 MQTT  
MQTT（Message Queuing Telemetry Transport，消息队列遥测传输）是 IBM 开发的一个即时通讯协议，有可能成为物联网的重要组成部分。该协议支持所有平台，几乎可以把所有联网物品和外部连接起来，被用来当做传感器和制动器（比如通过 Twitter 让房屋联网）的通信协议。[参考](https://baike.baidu.com/item/MQTT/3618851?fr=aladdin)    

MQTT 协议是一个面向物联网应用的即时通信协议，使用 TCP/IP 提供网络连接，能够对负载内容实现消息屏蔽传输，开销小，可以有效降低网络流量，协议的特点和功能包括：

- 长连接协议  

- 终端数据点上报，支持的数据点类型包括：
 - 整型（int）
 - 浮点数（float）
 - 字符串（string）
 - JSON格式  

- 平台消息下发

- 基于 Topic 的订阅、发布以及消息推送，可以实现设备间的消息单播以及组播

ESP32 可以移植并很好的支持 MQTT 协议，设备端通过 pubish 即可讲数据发送到云端。  

        val =  s_device_info_num / 10;
        char buf[128];
        memset(buf, 0, sizeof(buf));
        sprintf(&buf[3], "{\"%s\":%d}", ONENET_DATA_STREAM, val);
        uint16_t len = strlen(&buf[3]);
        buf[0] = data_type_simple_json_without_time;
        buf[1] = len >> 8;
        buf[2] = len & 0xFF;
        mqtt_publish(client, "$dp", buf, len + 3, 0, 0);

## 7. 总结

通过利用 ESP32 进行抓包可以在保证抓包质量的同时承担很多其他功能角色，来最大程度的利用 ESP32.并且 ESP32 也可以利用 MQTT 进行订阅相关 topic， 进而通过云端来控制 ESP32.  

改进: 不同区域内的人流量大小会影响到 ESP32 接收到的信号强度发生变化，因此当使用信号强度来判定选择区域范围时会存在误差，此点需要改进。