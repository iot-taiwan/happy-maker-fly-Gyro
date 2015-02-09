# 深圳 Maker 學習之旅

今天(2015.2.4)的目標為串接四軸飛行器所需使用到的Block。而此篇文章為三軸陀螺儀(Grove - 3-Axis Digital Gyro)Block的使用方法，首先讓大家看一下陀螺儀的外貌，如下圖
![陀螺儀](http://i.imgur.com/eUKW5h9.jpg)


## 陀螺儀用處

陀螺儀是一種用來感測與維持方向的裝置，陀螺旋進是日常生活中常見的現象，大家小時候玩過陀螺的，知道在一定速度下，就能一直保持平衡。三軸陀螺儀最大的作用就是測量“角速度”，以判別物體的運動狀態，用於飛行器上，是要讓飛行器可以知道自己目前要去哪，以及如何保持平衡穩定的飛行方向

## 準備工作

這次的Block整合是要將陀螺儀的 Sensor Data 推送至 WebSocket。
首先第一步，我們透過mbed LPC 1768 版子將三軸陀螺儀(Grove - 3-Axis Digital Gyro)與Wify模組串接起，如下圖：
![Wifi + 陀螺儀](http://i.imgur.com/wZotuUa.jpg)

## 開始實作
接下來技術實作的部分就分為三個部分

* Websocket channel server 服務，本書將使用 *sockets.mbed.org*
* ARM mbed 的 Websocket client 實作
* ARM mbed 與 Grove - 3-Axis Digital Gyro實作

```
#include "mbed.h"
#include "WiflyInterface.h"
#include "Websocket.h"
#include "ITG3200.h"
 
 
/* wifly interface:
*     - p9 and p10 are for the serial communication
*     - p19 is for the reset pin
*     - p26 is for the connection status
*     - "mbed" is the ssid of the network
*     - "password" is the password
*     - WPA is the security
*/
WiflyInterface wifly(p13, p14, p19, p26, "WWNet", "mmmmmmmm", WPA);
DigitalOut led1(LED1);
ITG3200 gyro(p9, p10, 0x68);
 
 
 
int main() {
    wifly.init(); //Use DHCP
    //wifly.init("192.168.21.33","255.255.255.0","192.168.21.2");
    while (!wifly.connect());
    led1=1;
    printf("IP Address is %s\n\r", wifly.getIPAddress());
 
    Websocket ws("ws://sockets.mbed.org/ws/mbedschool/viewer");
    //Websocket ws("ws://192.168.199.159:8888");
    while (!ws.connect());
    led1=2;
 
   
    int x = 0, y = 0, z = 0, temp = 0;
    //Set highest bandwidth.
    gyro.setLpBandwidth(LPFBW_42HZ);
 
    while (1) {
        char data[256];
        wait(0.1f);
        x = gyro.getGyroX();
        y = gyro.getGyroY();
        z = gyro.getGyroZ();
        temp = gyro.getTemperature();
  
        printf("Temp: %d, X: %d, Y: %d, Z: %d\n", temp, x, y, z);
        sprintf( data , "Temp: %d, X: %d, Y: %d, Z: %d\n", temp, x, y, z );
        ws.send(data);
        
        //wait(1.0);
    }
}
```

