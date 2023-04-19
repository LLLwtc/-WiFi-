#改造普通灯变成 WiFi 控制灯

\--------------------------------------------------------------------------------------------------------------------------------

##一、 **项目目的**

1. 实现一套系统，允许用户WiFi指令来控制电灯的开关，实现改造普通灯变成 WiFi 控制灯；

##二、 **项目环境**

1. NodeMCU（ESP8266）开发板一块；

2. 高低电平继电器模块一个；

3. 普通 USB 灯一个；

4. 220V-5V 变压器；

5. 手机一台；

6. 杜邦线若干；

7. Arduino IDE开发环境；

##三、 **项目原理**

1. ESP8266模块原理

​		(1) ESP8266是一种低成本、低功耗的Wi-Fi模块，ESP8266模块的原理是通过内置的处理器和Wi-Fi收发器，与外设进行通信，并使用		Flash存储器存储程序代码和数据，以实现连接到Wi-Fi网络并进行数据交换的功能，使其能够在Wi-Fi网络上运行；

​		(2) 芯片：ESP8266模块内置了一个高性能的32位Tensilica L106处理器，其运行频率可达80MHz；

​		(3) Wi-Fi：ESP8266模块集成了802.11 b/g/n Wi-Fi功能，支持AP、STA和AP+STA三种工作模式。该模块还具有多种安全加密机制，		如WPA/WPA2-PSK等；

​		(4) 外设：ESP8266模块还具有多种外设接口，如UART、SPI、I2C、PWM等，可以方便地与其他外设进行通信；

​		(5) Flash存储器：ESP8266模块内置了Flash存储器，可用于存储程序代码和数据；

​		(6) 供电：ESP8266模块的工作电压为3.3V，可以通过USB口或外部电源供电；

​		(7) SDK：ESP8266模块还提供了基于C语言的SDK和完整的开发工具链，使得开发者可以方便地进行开发和调试；

2. HTTP/HTTPS服务器程序原理

​		(1) HTTP和HTTPS是用于在Web浏览器和Web服务器之间传输数据的协议。HTTP是一种无状态协议，而HTTPS则通过加密来保证数		据的安全性；

​		(2) 监听端口：HTTP/HTTPS服务器程序会在指定的端口上监听传入的HTTP/HTTPS请求。通常，HTTP使用端口80，而HTTPS使用端		口443；

​		(3) 接收请求：当HTTP/HTTPS请求到达服务器时，服务器程序会解析请求头和请求体，以确定请求的类型、路径、参数、方法等信		息；

​		(4) 处理请求：服务器程序会根据请求的类型和内容来执行相应的操作，比如获取静态文件、执行动态脚本、查询数据库等；

​		(5) 返回响应：处理请求后，服务器程序会将响应头和响应体发送回客户端。响应头包括状态码、响应类型、内容长度等信息，而响		应体则包含实际的响应数据；

​		(6) 关闭连接：HTTP是一种无状态协议，服务器和客户端之间的连接在每个请求/响应周期结束后都会关闭；

​		(7) 建立SSL/TLS连接：HTTPS服务器程序会在传输层使用SSL/TLS协议来保证数据的安全性。客户端会向服务器发送一个初始的SSL		握手请求，服务器会响应并建立SSL/TLS连接；

​		(8) 握手过程：在SSL/TLS握手过程中，服务器和客户端会交换一些加密密钥和证书，以确保数据的加密和身份验证。一旦握手成功，		数据就会在客户端和服务器之间进行安全传输；

3. 继电器控制原理

​		(1) 继电器是一种电气开关，通常用于控制高电压/大电流的电路，例如控制家庭电器、工业机械、汽车电路等；

​		(2) 继电器结构：继电器由控制电路和负载电路两部分组成。控制电路通常由一个或多个电磁线圈和开关接点组成，当电磁线圈通电		时，接点就会闭合或打开。负载电路则包括一个或多个开关接点和相应的负载设备（例如电灯、电机等）；

​		(3) 控制电路：控制电路通常由一个直流电源、一个电磁线圈和一个开关接点组成。当直流电源施加到电磁线圈上时，电磁线圈就会		产生磁场，吸引开关接点闭合或打开。继电器还可以设置保护电路和灵敏度调节器等功能，以保护开关接点和控制电路；

​		(4) 负载电路：负载电路通常由一个或多个开关接点和相应的负载设备（例如电灯、电机等）组成。当开关接点闭合时，负载电路就		通电，负载设备就开始工作；

​		(5) 应用：继电器可以通过程序控制，例如通过微控制器或单片机控制继电器的开关状态，从而实现自动控制。继电器也可以通过传		感器或其他输入设备控制，例如通过光电传感器、温度传感器等控制继电器的开关状态；

##四、 **项目步骤与结果**

 

1. 路线图，如下：

![img](file:///C:\Users\YIYANG~1\AppData\Local\Temp\ksohtml26388\wps3.jpg) 

2. 配置Arduino；

![img](file:///C:\Users\YIYANG~1\AppData\Local\Temp\ksohtml26388\wps4.jpg) 

 

3. 添加ESP8266库；

![img](file:///C:\Users\YIYANG~1\AppData\Local\Temp\ksohtml26388\wps5.jpg) 

 

4. 为 NodeMCU编写程序，变成HTTP/HTTPS服务器，且有继电器控制功能；

```c++
/*
    This sketch demonstrates how to set up a simple HTTP-like server.
    The                          server will set a GPIO pin depending on the request
      http://server_ip/gpio/0 will set the GPIO2 low,
      http://server_ip/gpio/1 will set the GPIO2 high
    server_ip is the IP address of the ESP8266 module, will be
    printed to Serial when the module is connected.
*/

#include <ESP8266WiFi.h>

#ifndef STASSID
#define STASSID "HONOR 9X"//wifi名称
#define STAPSK "shuoshanibaobei"//WiFi密码
#endif

const char* ssid = STASSID;
const char* password = STAPSK;

// Create an instance of the server
// specify the port to listen on as an argument
WiFiServer server(80); //开启此板子的port 80

void setup() {
  Serial.begin(115200); //开启电脑的序列库，速度设为115200

  // prepare LED
  pinMode(2, OUTPUT);//将脚位2设置为输出，就是板子上的D4脚位
  digitalWrite(2, 0);//先将该脚位设置为低电压

  // Connect to WiFi network
  Serial.println();
  Serial.println();
  Serial.print(F("Connecting to "));
  Serial.println(ssid);

  WiFi.mode(WIFI_STA);
  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(F("."));
  }
  Serial.println();
  Serial.println(F("WiFi connected"));

  // Start the server
  server.begin();
  Serial.println(F("Server started"));

  // Print the IP address
  Serial.println(WiFi.localIP());
}

void loop() {
  // Check if a client has connected
  WiFiClient client = server.accept();
  if (!client) { return; }
  Serial.println(F("new client"));

  client.setTimeout(5000);  // default is 1000

  // Read the first line of the request
  String req = client.readStringUntil('\r');
  Serial.println(F("request: "));
  Serial.println(req);

  // Match the request
  int val;
  if (req.indexOf(F("/gpio/0")) != -1) {
    val = 0;
  } else if (req.indexOf(F("/gpio/1")) != -1) {
    val = 1;
  } else {
    Serial.println(F("invalid request"));
    val = digitalRead(2);
  }

  // Set LED according to the request
  digitalWrite(2, val);

  // read/ignore the rest of the request
  // do not client.flush(): it is for output only, see below
  while (client.available()) {
    // byte by byte is not very efficient
    client.read();
  }

  // Send the response to the client
  // it is OK for multiple small client.print/write,
  // because nagle algorithm will group them into one single packet
  client.print(F("HTTP/1.1 200 OK\r\nContent-Type: text/html\r\n\r\n<!DOCTYPE HTML>\r\n<html>\r\nGPIO is now "));
  client.print((val) ? F("high") : F("low"));
  client.print(F("<br><br>Click <a href='http://"));
  client.print(WiFi.localIP());
  client.print(F("/gpio/1'>here</a> to switch LED GPIO on, or <a href='http://"));
  client.print(WiFi.localIP());
  client.print(F("/gpio/0'>here</a> to switch LED GPIO off.</html>"));

  // The client will actually be *flushed* then disconnected
  // when the function returns and 'client' object is destroyed (out-of-scope)
  // flush = ensure written data are received by the other side
  Serial.println(F("Disconnecting from client"));
}
```

![img](file:///C:\Users\YIYANG~1\AppData\Local\Temp\ksohtml26388\wps6.jpg) 

 

5. 为手机编写APP或网站，具有发送 HTTP/HTTPS 的功能；

 

http://server_ip/gpio/0 will set the GPIO2 low,http://server_ip/gpio/1 will set the GPIO2 high

 

6. 编译、上传程序；

![img](file:///C:\Users\YIYANG~1\AppData\Local\Temp\ksohtml26388\wps7.jpg) 

![img](file:///C:\Users\YIYANG~1\AppData\Local\Temp\ksohtml26388\wps8.jpg) 

7. 通过手机浏览器输入网址，传输指令；

 

![img](file:///C:\Users\YIYANG~1\AppData\Local\Temp\ksohtml26388\wps9.png) 

![img](file:///C:\Users\YIYANG~1\AppData\Local\Temp\ksohtml26388\wps10.jpg) 

![img](file:///C:\Users\YIYANG~1\AppData\Local\Temp\ksohtml26388\wps11.png) 

 

8. 演示视频：改造普通灯变为Wi-Fi灯  https://www.bilibili.com/video/BV1tT411p7Gv/?spm_id_from=333.788.top_right_bar_window_history.content.click&vd_source=64ba9b28cd935af3170b4c9c296c3c3a 

##五、 **项目总结**

1. 原理：
   * 实现了搭建服务器，发送请求，响应请求，高电平接通低电平断开设置继电器的任务；
   * 各个模块的原理参考第三点项目原理所述；
   * 本次实验采用了ESP8266开发板，变压器，高低电平继电器，USB灯共同制作完成了改造普通灯变成 WiFi 控制灯；

2. 实验中遇到的问题与处理：

* 各个模块之间不知如何接线；查询了网上很多教程以及请教了身边同学，最终完成接线；
*  程序烧录成功，USB灯却不能亮；最终排查是接线的问题，杜邦线接触不良导致；
*  在宿舍，程序可以连通手机发出的WIFI，但去到教室却连通不了手机热点；教室WiFi信号不好导致的问题；

 