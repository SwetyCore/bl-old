---
layout: post
title: 用ESP32制做网络时钟
categories: [ESP32]
date: 2021-08-01 00:00:00
keywords: ESP32, ILI9225, Arduino GFX
---

# 介绍

+ 效果展示

<iframe src="//player.bilibili.com/player.html?aid=504500657&bvid=BV1qg41177Rj&cid=380856567&page=1" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true"> </iframe>

 + 介绍
   - 显示时间
   - 根据IP定位显示天气
   - 利用Smart Config进行微信扫码联网

# 制作过程

+ **材料表**

  - ESP32开发板 *1
  - 母对母杜邦线 *7
  - 2.0寸主控IC为ILI9225的TFT显示屏 *1
  - Micro USB数据线 *1

+ **工具**

  - Arduino IDE
  - *Visual Studio Code*
  - *Python3 环境*
  - msys2
  - 图片编辑工具（我这里用的是自带的画图）
  - Img2Lcd 某宝客服赠送的工具
  - [FontSmaller 字体文件子集化工具](https://fontsmaller.github.io/)
  - FontLabTypeTool **付费软件**

+ **硬件连接表**

  | ESP32 | TFT屏幕 |
  | :---: | :-----: |
  |  3V3  |   VCC   |
  |  GND  |   GND   |
  |  25   |   RS    |
  |  15   |   CS    |
  |  14   |   CLK   |
  |  13   |   SDA   |
  |  26   |   EST   |

  

+ **库文件**

  ```c++
  #include <Arduino_GFX_Library.h>
  #include <ArduinoJson.h>
  #include <HTTPClient.h>
  #include <time.h>
  #include <SPI.h>
  ```

# 程序编写

  1. **WIFI连接**

     在一般情况下，如果直接将WiFi的SSID和密码写入到代码中将不利于切换网络，因此我们使用WiFi Smart Config来进行配网，代码在Arduino的示例程序里找到。

  2. **时间同步**

     由于ESP32断电并不会保存时间，因此每次上电都需要从网络获取时间。

     利用 `configTime(gmtOffset_sec, daylightOffset_sec, ntpServer);`来同步时间，相关代码如下：

     ```c++
     const char* ntpServer = "pool.ntp.org";
     const long  gmtOffset_sec = 28800;
     const int   daylightOffset_sec = 0;
     
     
     configTime(gmtOffset_sec, daylightOffset_sec, ntpServer);
     
     void GetTime()
     {
         struct tm timeinfo;
       	if (!getLocalTime(&timeinfo)) {
         	return;
       	}
         Serial.println(&timeinfo, "%A, %Y-%m-%d %H:%M:%S");
     }
     ```

  3. **位置获取**

     由于当时并没有申请IP定位的相关API，加上之前学习爬虫时偶然在NetWork里面发现了BiliBili的相关接口：

     > http://api.bilibili.com/x/web-interface/zone

     参考链接

     - API请求方式及响应参考：[通过ip确定地理位置](https://github.com/SocialSisterYi/bilibili-API-collect/blob/master/clientinfo/ip.md)
     - JSON数据解析：[ArduinoJson: Efficient JSON serialization for embedded C++](https://arduinojson.org/)

     相关代码如下：

     ```c++
     float lat = 0.0;	//latitude 唯独
     float lon = 0.0;	//longitude 经度
     String city = "";	//城市名称
     
     void getlocation() {
       DynamicJsonDocument doc(1024);
       HTTPClient http;
       http.begin("http://api.bilibili.com/x/web-interface/zone");
       int httpCode = http.GET();
       if (httpCode == 200)
       {
         Serial.println("Get OK");
         String resBuff = http.getString();
         Serial.println(resBuff);
         deserializeJson(doc, resBuff);
         JsonObject root = doc.as<JsonObject>();
         city = root["data"]["city"].as<String>();
         lat = root["data"]["latitude"];
         lon = root["data"]["longitude"];
         
         Serial.print(lat);
         Serial.print(lon);
         Serial.println(city);
       }
       else {
         Serial.println("Location get error!");
       }
     
     
       http.end(); // 结束当前连接
     }
     ```

  4. **天气获取**

     API来源的原因同上，选自win10上的天气资讯里抓到的API：

     > https://api.msn.cn/weather/current

     *请求方式：GET*

     **查询字符串参数：**

     | KEY          | VALUE                                      | 备注   |
     | ------------ | ------------------------------------------ | ------ |
     | latLongList: | 33.13815,111.49078                         | 经纬度 |
     | locale:      | zh-cn                                      | 地区   |
     | units:       | C                                          |        |
     | appId:       | 9e21380c-ff19-4c78-b4ea-19558e93a5d3       |        |
     | apiKey:      | j5i4gDqHL6nGYwx5wi5kRhXjtf2c5qgFX9fzfk0TOo |        |
     | ocid:        | msftweather                                |        |
     | wrapOData:   | FALSE                                      |        |

     **示例：**
     
     ```shell
     curl 'https://api.msn.cn/weather/current?latLongList=34%2C112&locale=zh-cn&units=C&appId=9e21380c-ff19-4c78-b4ea-19558e93a5d3&apiKey=j5i4gDqHL6nGYwx5wi5kRhXjtf2c5qgFX9fzfk0TOo&ocid=msftweather&wrapOData=false'
     ```
     
     **响应参考**
     
     
     ```json
     {
         "responses": [
             {
                 "weather": [
                     {
                         "current": {
                             "baro": 973, 
                             "cap": "阴", 
                             "capAbbr": "多云", 
                             "daytime": "n", 
                             "dewPt": 22, 
                             "feels": 22, 
                             "rh": 80, 
                             "icon": 32, 
                             "pvdrIcon": "32", 
                             "urlIcon": "http://img-s-msn-com.akamaized.net/tenant/amp/entityid/AAehyQC.img", 
                             "sky": "", 
                             "temp": 25, 
                             "vis": 6, 
                             "windDir": 180, 
                             "windSpd": 61, 
                             "created": "2021-08-02T00:04:38+08:00", 
                             "pvdrCap": "多云", 
                             "pvdrWindDir": "南风", 
                             "pvdrWindSpd": "6-7级", 
                             "aqi": 34, 
                             "aqiSeverity": "空气优", 
                             "richCaps": [ ], 
                             "alertCount": 0
                         }, 
                         "provider": {
                             "name": "中国天气网", 
                             "url": "http://www.weather.com.cn/weather/101180708.shtml"
                         }
                     }
                 ], 
                 "source": {
                     "id": "101180708", 
                     "coordinates": {
                         "lat": 33.1379738, 
                         "lon": 111.491
                     }, 
                     "location": {
                         "Name": "xichuan", 
                         "StateCode": "河南", 
                         "CountryName": "china", 
                         "CountryCode": "CN", 
                         "TimezoneName": "Asia/Shanghai", 
                         "TimezoneOffset": "08:00:00"
                     }, 
                     "utcOffset": "08:00:00", 
                     "countryCode": "CN"
                 }
             }
         ], 
         "units": {
             "system": "Metric", 
             "pressure": "百帕", 
             "temperature": "‎°C", 
             "speed": "公里/小时", 
             "height": "毫米", 
             "distance": "公里", 
             "time": "s"
         }, 
         "copyright": "Copyright © 2021 Microsoft and its suppliers. All rights reserved. This API cannot be accessed and the content and any results may not be used, reproduced or transmitted in any manner without express written permission from Microsoft Corporation.", 
         "latLongIdMap": {
             "33.13815,111.49078": "101180708"
         }
     }
     ```
     
     信息解析&获取方式同IP地理位置查询。
     
  5. **信息显示**

     这里用到了 Arduino GFX 库

     参考链接：[Arduino_GFX](https://github.com/moononournation/Arduino_GFX)

     + 初始化：

       ```c++
       Arduino_DataBus *bus = new Arduino_ESP32SPI(25 /* RS */, 15 /* CS */,  14/* SCK */, 13 /* MOSI */, -1 /* MISO */, HSPI /* spi_num */);
       
       Arduino_GFX *gfx = new Arduino_ILI9225(bus, 26 /* RST */, 1);
       void setup(void)
       {
         Serial.begin(115200);
         gfx->begin();
       }
       ```

       

     + 清除屏幕：`gfx->fillScreen(color);`

     + 输出文字：

       ```c++
         gfx->setCursor(10, 10);	// 设置光标位置，参考点为首个字体的左下角
         gfx->setFont(&pf_min_ys10pt8b);	// 选择字体，若没有则使用默认字体
         gfx->setTextColor(RED);	// 选择字体颜色
         gfx->println("Hello World!"); 	// 输出到屏幕上
       ```

     + 显示图片：

       ```c++
       gfx->draw24bitRGBBitmap(x,y,bitmap,width,height);	// 显示彩图
       gfx->drawXBitmap(x,y,gImage_code,width,height,WHITE);	// 显示黑白二色图（二维码）
       ```

       

     **问题**

     1. 如何显示中文？

        自定义中文字库。

        官方提供了转换的工具：[Adafruit-GFX-Library/fontconvert](https://github.com/adafruit/Adafruit-GFX-Library/tree/master/fontconvert) ，按照[fontconvert_win.md](https://github.com/adafruit/Adafruit-GFX-Library/blob/master/fontconvert/fontconvert_win.md)文档即可编译成功。

        使用方法：`./fontconvert fontfile size [first] [last]`

        但是，由于中文字库太过庞大，想要全部应用单靠小小的ESP32是无法承受的，因此我们需要精简，我的思路如下：

        1. 找出可能要显示的文字：(这里我用Python作为工具进行实现)

           ```python
           a = "朗晴少云晴间多云多云阴有风平静微风和风清风强风劲风疾风大风烈风风暴狂爆风飓风热带风暴霾中度霾重度霾严重霾阵雨雷阵雨雷阵雨并伴有冰雹小雨中雨大雨暴雨大暴雨特大暴雨强阵雨强雷阵雨极端降雨毛毛雨细雨雨小雨中雨中雨大雨大雨暴雨暴雨大暴雨大暴雨特大暴雨雨雪天气雨夹雪阵雨夹雪冻雨雪阵雪小雪中雪大雪暴雪小雪中雪中雪大雪大雪暴雪浮尘扬沙沙尘暴强沙尘暴龙卷风雾浓雾强浓雾轻雾大雾特强浓雾热冷省市南阳镇江优良轻度污染东西北空气"
           # 对字符串a进行去重
           a=set(a)
           b=""
           for i in a:     
               print(i,end="")
               b = b + i
           # 去重之后：少天冻雾冷!西空热云阴狂特间重降度静大爆伴多冰微浮朗扬气雪劲阵中有小染暴阳飓浓严平卷轻极夹毛雹雨省镇江雷清优北沙烈带并污龙东疾良尘强细霾晴风端和市南
           # 对字符串的Unicode码进行排序
           ls=[]
           for i in b:
               ls.append(ord(i))
           ls.sort()
           ```

        2. 由于字体转换工具的局限性，它截取的字体必须是连续的，即按照Unicode码从[first]到[last]进行转换，然而我们需要的字的Unicode码跨度很大（从[" "(32)]到["龙"(40857)]），转换后的.h文件大小高达2Mb，因此还需要再进行缩减。

           由于英文及其符号的Unicode码是连在一起的（从32到126）因此我们可以按照一定的规则将ttf文件中我们需要的字的Unicode码进行修改，在126之后进行追加**我称之为字体映射[doge]**，然后再在程序里写一个解析的函数即可。

           得到映射关系及部分解析代码的python程序如下：

           ```python
           sum=0
           ys={}
           for i in ls:
               if i in range(0x4e1c,ls[len(ls)-1]+1):
                   sum+=1
                   print(f"{chr(i)}:{hex(i)}:{hex(126+sum)} ")		# 映射关系
                   ys[chr(i)]=chr(126 + sum)
                   print(f'txt.replace("{chr(i)}", "\\{str(hex(126+sum))[1:]}");')		# 解析代码
           
           '''
           输出示例:
           东:0x4e1c:0x7f 
           txt.replace("东", "\x7f");
           严:0x4e25:0x80 
           txt.replace("严", "\x80");
           中:0x4e2d:0x81 
           txt.replace("中", "\x81");
           云:0x4e91:0x82 
           txt.replace("云", "\x82");
           ..........
           飓:0x98d3:0xc6 
           txt.replace("飓", "\xc6");
           龙:0x9f99:0xc7 
           txt.replace("龙", "\xc7");
           '''
           # 很明显的从4w多缩减到了199
           ```

           解析的函数：

           ```c++
           void printcn(String txt) {
             // Serial.println(txt);
               
             txt.replace("东", "\x7f");
             txt.replace("严", "\x80");
             txt.replace("中", "\x81");
             txt.replace("云", "\x82");
             txt.replace("优", "\x83");
           
             gfx->println(txt);
           }
           
           ```

           得到映射关系后我们就可以对ttf文件进行修改了，利用[**FontSmaller 字体文件子集化工具**]将需要的文件进行简化，再用[**FontLabTypeTool字体编辑器**]工具对其Unicode进行修改，最后使用编译好的字体转换工具进行转换即可得到.h文件			`./fontconvert fontfile 10 32 199`

     2. 如何得到bitmapArray？

        我用了两张图，第一张是Smart Config的二维码，因为只有黑白，所以很容易就实现了显示：

        设定参考：

        | 选项               | 选择           |
        | ------------------ | -------------- |
        | 输出数据类型:      | C语言数组(*.c) |
        | 扫描模式:          | 水平扫描       |
        | 输出灰度:          | 单色           |
        | 字节内象素数据反序 | 是             |
        
        *需要修改生成的.c中的char[]类型为uint8_t[]类型
        
        第二张是彩色的UI背景图，使用ppt以及画图进行制作。
        
        然而彩色的UI背景图就不是那么容易了，无论我用这个软件怎么设置，在屏幕中显示的都只有混乱，因此只能找别的软件了。在我翻遍了github之后，找到了一个py文件（淦忘了那个py文件的具体出处链接了）运行后终于成功显示出了上下颠倒，红色变蓝色（后来百度到是因为R通道和B通道被交换了）的图片，既然py文件代码看不懂，那就主动转换源文件来 **负负得正** 
        
        转换源图片为负的python代码：(拿出了还没有入门的opencv知识)
        
        ```python
        import cv2
        
        img1 = cv2.imread("./IMG/bg.bmp")
        
        b, g, r = cv2.split(img1)
        img1 = cv2.merge([r, g, b])  # 通道转换
        
        img1 = cv2.flip(img1, 0)  # 垂直翻转
        
        
        cv2.imwrite("out.jpeg", img1)	# 输出转换后的图片
        
        ```
        
        注意：输出的图片还要另存为24色的bmp文件（利用win自带的画图的另存为）
        
        然后利用找到的py程序输出bitmap数组：(程序经过了我部分魔改，很抱歉我找不到程序的原作者了啊啊啊啊)
        
        ```python
        import os
        import shutil
        
        img_out_c = list()
        
        sum = 0
        
        
        def img_to_c_string(image_path, img_name):
            with open(image_path, 'rb') as img_file:
                binary_file = img_file.read()
        
                # img data
                img_out_c.append(
                    "/*********************************************************************\n*\n*  name: %s\n*/" % (img_name))
                img_out_c.append("static const uint8_t %s_%s[%s] = {" % (
                    img_name[-3:], img_name[:-4].replace(' ', '_').replace('-', '_'), os.path.getsize(image_path)))
        
                for x in range(int(len(binary_file) / 20)):
                    part = binary_file[20 * x: 20 * x + 20]
                    img_out_c.append("    0x" + ", 0x".join(format(x, "02X") for x in part) + ',')
                    print("    0x" + ", 0x".join(format(x, "02X") for x in part) + ',')
                    print("\n\n")
                if len(binary_file) % 20 != 0:
                    part = binary_file[int(len(binary_file) / 20) * 20:len(binary_file)]
                    img_out_c.append("    0x" + ", 0x".join(format(x, "02X") for x in part))
                img_out_c.append("};\n")
        
        
        def generate_c_code(env_dir):
            out_path = os.path.join(env_dir, "img_out.h")
            if os.path.isfile(out_path):
                os.remove(out_path)
            with open(out_path, 'w') as c_file:
                for bmp_line in img_out_c:
                    c_file.write(bmp_line + '\n')
                c_file.write('\n')
        
        
        if __name__ == '__main__':
            env_dir = os.path.dirname(os.path.realpath(__file__))
            bmp_dir = os.path.join(env_dir, "IMG")
            if os.path.exists(bmp_dir):
                bmp_list = os.listdir(bmp_dir)
                for img_file in bmp_list:
                    bmp_path = os.path.join(bmp_dir, img_file)
                    print(bmp_path)
                    img_to_c_string(bmp_path, img_file)
                generate_c_code(env_dir)
        
        ```
        
        之后把想要的功能全部实现一波就行了！！！
        
        最后附上我写的虽然满是bug但总算完成了基本功能的项目源码：[SwetyCore/Esp32NetworkClock: 一个基于Esp32制作的可以显示天气的时钟 (github.com)](https://github.com/SwetyCore/Esp32NetworkClock) 欢迎前来Star！！

<small>本项目所用API仅为学习研究所用!</small>

