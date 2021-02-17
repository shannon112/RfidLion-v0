# RfidLion-v0
This is a fake rfid based badge reader to Lenel OnGuard system.  
Created by ShannonLee to test the Tailgating that is a new feature coming soon to UmboCV!  
The hardware and software were then be refined into SHENEL-1 by @mattwang44.  
**(source code is not open, because it is one of my intern projects at UmboCV.)**
```
LionKing 	Basic, only rfid feature, no http/https request would be sent
LionKingBeta 	Demo, sending http request to generate a tweet (twitter)
LionKingAlpha 	HTTPSver1, sending https request to slackAPI to generate a msg, but very slow
LionKingPlus 	HTTPSver2, after optimizing, sending https request to slackAPI to generate a msg more rapidly
LionKingGet 	HTTPver1, sending http request to test website
LionKingTailgating 	HTTPver2, sending http request to Andy's aws server(with public ip and port number)
```
Our aim is to send a https post request to end point server when users swipe thier badge(rfid card).  
In this program(LionKingPlus) we use slack webhook api as a demo, if someone swipes his badge, our device would send a https post request to url which is given by slack webhook api and then you could see your cardUID is showed on the slack messenge.  
Demo video: https://youtu.be/BVlqO0EteXA  

<img src="https://raw.githubusercontent.com/shannon112/RfidLion-v0/main/doc/IMG_0567.jpg" width="420"> <img src="https://raw.githubusercontent.com/shannon112/RfidLion-v0/main/doc/IMG_0568.png" width="420">
  
## Material
We choose the most commonest and cheapest modules to complete this project.   
The total budget is less than 400 NTD!   

| # | Item name            | Ref / Remarks     | price for one (NTD)
| - | ----------------     | -------- | -----------
| A | NodeMCU v2 Lua * 1   | https://goods.ruten.com.tw/item/show?21521479079344     | 120
|B| RFID module RC522 * 1 | https://goods.ruten.com.tw/item/show?21512663552783     | 73
|C| LCD display IIC * 1  | https://goods.ruten.com.tw/item/show?21408019307836     | 90
|D| USB cable * 1         | With the end point cut preserving Type-A, used as a power line. | 15
|E| LM2596 LM2596S * 1   | https://goods.ruten.com.tw/item/show?21736053587636 | 20
|F| Micro-B USB cable * 1  | With data tranport function. Let NodeMCU communicate with your PC. | -
|G| Transformer * 1      | AC to DC 5V1A | -
|H| LED (greeen) * 1      | - | -
|I| Switch * 2           | https://goods.ruten.com.tw/item/show?21547105965229 | 4*2
|J| Active buzzer * 1    | https://goods.ruten.com.tw/item/show?21504357917958 | 5
|K| Terminal * 2         | https://goods.ruten.com.tw/item/show?21805662456296 | 2*2
|L| Some jumper wire * n | https://goods.ruten.com.tw/item/show?21611084407903 | 65
|M| Breadboard * 2 | https://goods.ruten.com.tw/item/show?21442516284863 | 28*2
|N| Based board * 1 | https://goods.ruten.com.tw/item/show?21617609571746 | 50


## Wiring
<img src="https://raw.githubusercontent.com/shannon112/RfidLion-v0/main/doc/IMG_0565.jpg" width="600">
<img src="https://raw.githubusercontent.com/shannon112/RfidLion-v0/main/doc/IMG_0564.png" width="400">

## Library
Here is the libraries that we have used.  
#include <Wire.h>  //built-in  
#include <MFRC522.h>  
https://github.com/miguelbalboa/rfid  
#include <ESP8266WiFi.h>  
#include <WiFiClientSecure.h>  
#include <WiFiUdp.h>  
https://github.com/esp8266/Arduino  
#include <LiquidCrystal_I2C.h>  
https://github.com/fdebrabander/Arduino-LiquidCrystal-I2C-library or use manage lirbraries in ArduinoIDE to add  
  
## Pipeline
<img src="https://raw.githubusercontent.com/shannon112/RfidLion-v0/main/doc/IMG_0566.png" width="1000">

```c
void loop() {
  turnOffScreen(start_display_time); //lcd
  if ( ! mfrc522.PICC_IsNewCardPresent()) return;// Look for new cards
  if ( ! mfrc522.PICC_ReadCardSerial()) return;// Select one of the cards
  String UID = dump_byte_array(mfrc522.uid.uidByte, mfrc522.uid.size);
  start_display_time=reaction(UID); //lcd, led and breeze
  postSlackMessage(UID);
  turnOffNotice(); //led and breeze
  last_https_complete_time=millis();
}
```
  
## The part that may need tp be modified in future
Change these fields to your wifi.
```c
const char* ssid = "-";
const char* password = "-";
```
You can modify the POST and postDate fields to fit the other https requests you want to send in the future.  
```c
const char* host        = "hooks.slack.com";
const char* fingerprint = "ab f0 5b a9 1a e0 ae 5f ce 32 2e 7c 66 67 49 ec dd 6d 6a 38";
String url = "/services/T02NQ48FJ/BCFK6EB9V/J3Igb3MnDa4liqmzjl8YclFh";
```
```c
void postSlackMessage(String UID) {
  String postData= "{\"attachments\":[{\"fallback\": \"Someone swipes the badge.\",\"color\": \"#29c6a7\",\"pretext\": \"侵入者を検知しました！！！∑(=ﾟωﾟ=;)\",\"author_name\": \"Lenel onGuard Faker\",\"title\": \"CardUID\",\"text\": \""+UID+"\",\"fields\": [{\"title\": \"Authorization\",\"value\": \"Accept\",\"short\": False}]}]}";
  int dataLength = postData.length();

  String POST =    "POST " + url + " HTTP/1.1\r\n"
                   "Host: " + host + "\r\n"
                   "User-Agent: ArduinoIoT/1.0\r\n"
                   "Connection: Keep-Alive\r\n"
                   "Content-Type: application/x-www-form-urlencoded\r\n"
                   "Content-Length: " + dataLength + "\r\n\r\n"
                   "" + postData;
  client.setNoDelay(1);
  client.print(POST);

  while (!(client.available())) {}
  while (client.available()) {
    for(int i=0; i<23;i++) client.readStringUntil('\r');//read them all in once
    //Serial.println(client.readStringUntil('\r'));//read responses by line
    break;
  }
  //Serial.println("POST request complete!");
}
```
  
## Ref
LCD:  
https://www.evernote.com/shard/s315/sh/85438bf7-5950-4c5c-bb26-dea749881ac5/27b0f0befad2824d8bc7ca4caaa36311  
NodeMCU:  
https://www.evernote.com/shard/s315/sh/5a2f6f92-f527-4765-af2e-5adbef4ef920/cb073a22ff95604406b2c9318520b793  
RFID:  
https://www.evernote.com/shard/s315/sh/dd4e8b73-447f-4a97-8f34-b9d5b4655b4e/5618b229cad4f174ea58ad08b172f587  
Https request:  
https://www.evernote.com/shard/s315/sh/9dffc236-a8bb-41ca-a17d-78c6f8d45ab4/f6fa5e0db28d59c5b1e13d946bdaceea  

- SHENEL-1
<img src="https://raw.githubusercontent.com/shannon112/RfidLion-v0/main/doc/SHENEL-1.jpg" width="200"> 
