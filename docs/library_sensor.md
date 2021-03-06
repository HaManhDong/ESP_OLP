# Docs các thư viện sử dụng để đọc và điều khiển cảm biến

## 1. DHT11(Cảm biến nhiệt độ và độ ẩm)

Giá trị trả về dạng số thực.

```cpp

#include <DHT.h>

#define DHTPIN 4   // Sử dụng pin 4 để đọc dữ liệu từ cảm biến
#define DHTTYPE DHT11 // Loại cảm biến ở đây là DHT11
DHT dht(DHTPIN, DHTTYPE);

void setup(){
    dht.begin();// Trong hàm setup() để khởi tạo.
     Serial.begin(115200);
}

float getTemperature(){
  return dht.readTemperature();
}

float getHumidity(){
  return dht.readHumidity();  // Đọc độ ẩm từ cảm biến
}

void loop(){
  Serial.println(getTemperature());
  Serial.println(getHumidity());
  delay(2000);
}

```

![https://raw.githubusercontent.com/NTT-TNN/ESP_OLP/master/docs/images/dht11_bb.png](https://raw.githubusercontent.com/NTT-TNN/ESP_OLP/master/docs/images/dht11_bb.png)

### Note: Chú ý lúc install library chọn bản 1.2.3 để cài đặt ko sẽ bị lỗi `Adafruit_Sensor.h: No such file or directory`

## 2. DS18B20(Cảm biến nhiệt độ)

```cpp

#include <OneWire.h>
#include <DallasTemperature.h>

OneWire ds(2); // Chân cắm output trên cảm biến có thể là 1 trong các chân sau: 00, 02, 04, 05,  10, 12, 13, 14
DallasTemperature DS18B20(&ds);

void setup(){
  Serial.begin(115200);
  DS18B20.begin();
  pinMode(2, INPUT_PULLUP);
}

//Hàm đọc nhiệt độ
float getTemperature(){
  float temperature;
  do{
    DS18B20.requestTemperatures();
    temperature = DS18B20.getTempCByIndex(0);
    delay(200);
    return temperature;
  }while(temperature == 85.0);
}

void loop(){
  float temp=getTemperature(); // gọi hàm để lấy nhiệt đố gán vào biến temp
  Serial.println(temp);
}


```

### Notes

- Hai chân GPIO00 và GPIO02 ko cần câu lệnh  pinMode(00, INPUT_PULLUP); ở setup các chân khác cần có câu lệnh INPUT_PULLUP tương ứng

![https://raw.githubusercontent.com/NTT-TNN/ESP_OLP/master/docs/images/ds18b20_bb.jpg](https://raw.githubusercontent.com/NTT-TNN/ESP_OLP/master/docs/images/ds18b20_bb.jpg)

## 3. ArduinoJson.h

### 3.1 Decoding/parser

Dùng để nhận một string có dạng giống json sau đó decode/parser thành json. Thường dùng khi nhận dữ liệu đến sau đó chuyển về json để tách các trường dữ liệu.

VD:

```sh
char json[] = "{\"sensor\":\"gps\",\"time\":1351824120,\"data\":[48.756080,2.302038]}";

StaticJsonBuffer<200> jsonBuffer;

JsonObject& root = jsonBuffer.parseObject(json);

const char* sensor = root["sensor"];
long time          = root["time"];
double latitude    = root["data"][0];
double longitude   = root["data"][1];
```

### 3.2 Encoding

```sh
Dùng để tạo một json.

StaticJsonBuffer<200> jsonBuffer

JsonObject& root = jsonBuffer.createObject();
root["sensor"] = "gps";
root["time"] = 1351824120;

JsonArray& data = root.createNestedArray("data");
data.add(48.756080);
data.add(2.302038);


root.printTo(Serial);
// This prints:
// {"sensor":"gps","time":1351824120,"data":[48.756080,2.302038]}
```

### Notes: Có thể tạo một json rồi add làm một phần tử của json khác

## 4. OLED


```cpp
#include "SH1106.h"

SH1106  display(0x3c, 4, 5);

void setup(){
    display.init();
    display.flipScreenVertically();
    display.setFont(ArialMT_Plain_24);
}

void loop(){
    display.clear();
    display.drawString(0, 0, temperatureCString);
    display.display();
}
```

### Note:

- hàm display.drawString nhận vào 2 tham số đầu tiên cho biết tọa độ của điểm bắt đầu vẽ String
- Library sử dụng file Zip
[https://github.com/squix78/esp8266-oled-ssd1306](https://github.com/squix78/esp8266-oled-ssd1306)

## 5. Cảm biến ánh sáng

- Sử dụng digitalRead để đọc dữ liệu trả về trên con cảm biến ánh sáng. Giá trị trả về sẽ là 0 hoặc 1.
- Sử dụng analogRead để đọc dữ liêu analog trả về trên con cảm biến ảnh sáng.

```cpp
int val = 0;         // variable to store the read value
void setup() {
  Serial.begin(115200);
}

void loop() {
  val = analogRead(A0);   // variable to read the value input chân ADC trên adruino
  Serial.println(val);
}

```

![https://raw.githubusercontent.com/NTT-TNN/ESP_OLP/master/docs/images/sensor-photodiode.jpg](https://raw.githubusercontent.com/NTT-TNN/ESP_OLP/master/docs/images/sensor-photodiode.jpg)

## 6. Kết nối wifi

```cpp
#include <ESP8266WiFi.h>

const char* ssid = "NTT_TNN_1";
const char* password = "thao0983451175";

void setup(){
    WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED)
  {
    delay(500);
    Serial.print(".");
  }
  Serial.print("Connected, IP address: ");
}
```

## 7. PubSubClient

Thư viện dùng để connect tới mqtt server

```cpp
#include <PubSubClient.h>
#include <ESP8266WiFi.h>

//MQTT Settings
const char* default_mqtt_server="10.0.0.221";
const char* default_mqtt_port="1883";
const char* topic_pub="icse/sensor";

WiFiClient espClient; // Khai báo wifi

// Hàm callback được goi để xử lý dữ liệu trả về từ mqtt server
void callback(char* topic, byte* payload, unsigned int length) {
  if(strcmp(topic,topic_sub)==0){
     payload[length] = '\0';  // Cắt bỏ dữ liệu thừa
     char inData[80];
     char payload_string[100];
     strncpy(payload_string, (char*)payload,sizeof(payload_string)); // chuyển về dàng char
}
PubSubClient client(espClient); // Gắn PubSub vào wifi


// Connected to mqtt server
void reconnect() {

  while (!client.connected()) {
    Serial.print("Attempting MQTT connection...");
    // Create a random client ID
    String clientId = "ESP8266Client-";
    clientId += String(random(0xffff), HEX);
    // Attempt to connect
    if (client.connect(clientId.c_str())) {
      Serial.println("connected");

      client.subscribe(topic_sub);
    } else {
      Serial.print("failed, rc=");
      Serial.print(client.state());
      Serial.println(" try again in 5 seconds");
      delay(5000);
    }
  }
}
 void setup(){
     client.setServer(default_mqtt_server, 1883);
     client.setCallback(callback);

}

loop(){
    // Kiem tra viec connect toi mqtt server neu ko chua connect se connect
    if (!client.connected()) {
      reconnect();
    }
    client.loop();
    // Publish noi dung theo topic icse/sensor
    char dataMessage[500];
    jsonData.printTo(dataMessage, sizeof(dataMessage));// Chuyen tu json sang String de pub
    client.publish("icse/sensor",dataMessage);
}

```

### Notes:

- Kích thước mặc định của gói tin gửi đi là 128 bytes muốn gửi gói tin có kịch thước lớn hơn cần chỉnh lại `#define MQTT_MAX_PACKET_SIZE 2048` trong file `PubSubClient.h` dòng 23( đường dẫn `libraries/PubSubClient/src/PubSubClient.h`)
- Lưu ý để có thể nhận được hàm callback() cần có `client.loop();`

## 8. Smart Wifi

Sử dụng web server để cho phép nhận vào wifi và mqtt server

```cpp
#include <ESP8266WiFi.h>  
#include <ESP8266WebServer.h>
#include <WiFiManager.h>



const char* default_mqtt_server = "192.168.1.100";
const char* default_mqtt_port = "1883";
https://github.com/NTT-TNN/ESP_OLP/blob/master/docs/images/bh1750_bb.png

void setup() {
    Serial.begin(115200);
    WiFiManager wifiManager;
    WiFiManagerParameter custom_mqtt_server("server", "mqtt server", default_mqtt_server, 40);
    WiFiManagerParameter custom_mqtt_port("port", "mqtt port", default_mqtt_port, 6);
    wifiManager.addParameter(&custom_mqtt_server); 
    wifiManager.addParameter(&custom_mqtt_port);
    Serial.println(WiFi.macAddress());
    pinMode(InputPin, INPUT);  //input declaration
    pinMode(ledPin, OUTPUT);
//    wifiManager.resetSettings();//  xóa cmt 
    if (!wifiManager.autoConnect("THAO_ESP")) {
      Serial.println("failed to connect, we should reset as see if it connects");
      delay(3000);
      ESP.reset();
      delay(5000);
    } 
    Serial.println("connected...yeey :)");
    client.setServer(default_mqtt_server, 1883);
    client.setCallback(callback);
}


void loop() {
   if (!client.connected()) {
      reconnect();
  }
  client.loop();

}

```

### Notes:

#### addParameter

- add các tham số khác như mqtt_server,mqtt_port trước khi autoConnect.

- Sử dụng : 
    `WiFiManagerParameter custom_mqtt_server("server", "mqtt server", default_mqtt_server, 40);`
    Để tạo tham số về server của mqtt.

- Sử dụng: `wifiManager.addParameter(&custom_text)` để thêm giao diện cho than số đó vào giao diện web.

- Sau khi người dùng chỉnh sửa các tham số(như mqtt_server hoặc mqtt_port) và sau khi đã connect thành công vào wifi ta có thể sử dụng câu lệnh để đọc các giá trị được người dùng nhập vào `custom_mqtt_server.getValue()` để lấy giá trị được nhập vào và xử lý.

- Ngoài các tham số như trên có thể thêm các tham số khác do mình tùy chỉnh như:     `WiFiManagerParameter custom_test("port", "mqtt test", default_mqtt_port, 6);` sau đó add vào giao diện web :    `wifiManager.addParameter(&custom_test);`.
- Để hiểu rõ hơn chạy ví dụ :<https://github.com/NTT-TNN/ESP_OLP/blob/master/AutoConnect/AutoConnect.ino>.

#### Autosave

Wifi manager sẽ tự động lưu SSID và  password lần gần nhất để conneect tới nên sẽ không thể tùy chình. Để không lưu gia trị đó thêm dòng `wifiManager.resetSettings();` trước khi autoConnect.

### Notes: Mặc định ở chế độ này arduino sẽ tự động connect đến wifi đã được connect đến ở lần kết nối trước nếu có thể. Nếu muốn mỗi lần nạp firmware đều yêu cầu người dùng config lại wifi (không sử dụng wifi lần gần nhất đã được lưu thì xóa comment dòng     `//wifiManager.resetSettings();`)

## 9. Phát Wifi

Sử dụng để phát wifi

```cpp
IPAddress local_IP(192,168,4,22);
IPAddress gateway(192,168,4,9);
IPAddress subnet(255,255,255,0);

Serial.println(WiFi.softAPConfig(local_IP, gachar dataMessage[500];teway, subnet) ? "Ready" : "Failed!");

Serial.print("Setting soft-AP ... ");
Serial.println(WiFi.softAP("HPCC_ESP_1","hpcc_iot") ? "Ready" : "Failed!");

Serial.print("Soft-AP IP address = ");
Serial.println(WiFi.softAPIP());

```

## 10. Xung PWM

Xung là các trạng thái cao/thấp về mức điện áp được lặp đi lặp lại. Một xung được đặc trưng bởi tần số và chu kỳ.

- Tần số là số lần lặp trong một đơn vị thời gian. Đơn vị là Hz, tức là số lần lặp trong 1 giây.
- Chu kỳ xung cho biết trong 1 dao động có bachar dataMessage[500];o nhiêu thời gian điện áp ở mức cao bao nhiêu thời gian điện áp ở mức thấp. Cụ thể liên hệ với arduino như bảng sau:

AnalogWrite | Tỷ lệ | Chu kỳ xung
---------|----------|---------
 analogWrite(0)	   | 0/255		 | 0%
 analogWrite(64)	 | 64/255	   | 25%
 analogWrite(127)	 | 127/255   | 50%
 analogWrite(191)	 | 191/255	 | 75% 
 analogWrite(255)	 | 255/255	 | 100%

 Hàm analogWrite() trong arduino giúp cho việc tạo xung dễ dàng hơn. Hàm này cho phép bạn thay đổi chu kỳ xung còn tần số là giá trị mặc định của arduino.

Trên hầu hết các Arduino board (ATmega168 or ATmega328), hàm analogWrite hoạt động trên các pins 3, 5, 6, 9, 10, and 11. Sử dụng analogWrite() bạn có thể điều chỉnh đèn LED sáng ở nhiều mức khác nhau hoặc điều chỉnh động cơ quay với nhiều speed khác nhau.

## 11. Memmory

Có 3 bộ nhớ:

- Flash memmory: Nơi mà Arduino sketch được lưu trữ
- SRAM: nơi mà sketch được tạo và chứa các biến khi chạy.
- EEPROM: Là vùng bộ nhớ trống cho phép lập trình viên có thể lưu trữ thông tin lâu dài.

Flash và EEPROM là vùng nhớ tồn tại cả khi mất điện. SRAM sẽ mất khi mất điện.

Dung lượng bộ nhớ ATmega328:
char dataMessage[500];
- Flash  32k bytes (of which .5k is used for the bootloader)
- SRAM   2k bytes
- EEPROM 1k byte

## 12. Sleep

Modules ESP8266 có 4 chế độ Sleep:
- No-sleep
- Modem-sleep
- Light-sleep
- Deep-sleep

Bảng dưới đây cho thấy sự khác nhau giữa 3 cchar dataMessage[500];hế độ sleep
![https://raw.githubusercontent.com/NTT-TNN/ESP_OLP/master/docs/images/esp8266_sleep_options.png](https://raw.githubusercontent.com/NTT-TNN/ESP_OLP/master/docs/images/esp8266_sleep_options.png)

- No-sleep: Chế độ này sẽ giữ mọi thứ ở trạng thái on.
- Modem-sleep: Đây là chế độ mặc định của ESP8266. Tuy nhiên chế độ này chỉ được bật khi ESP8266 đã được kết nối tới một điểm truy cập wifi. Ở chế độ này ESP8266 sẽ disable wifi nhiều nhất có thể. Nó sẽ turn off wifi giữa các lần DTIM beacon.
- Light-sleep: Chế độ này ở mức cao hơn của chế độ modem-sleep. Nó có các tính năng tương tự chế độ modem-sleep nhưng nó cũng tắt system clock và suspend CPU. Chú ý CPU không tắt nó chỉ không hoạt động.
- Deep-sleep: Tất cả mọi thứ sẽ off nhưng Real Time Clock (RTC) nơi mà sẽ lưu trữ thời gian của hệ thống. Khi mọi thứ turn off đây là một giải pháp năng lượng hiệu quả.

### Deepsleep

Với chế độ deep-sleep flow của chương trình sẽ như sau:
1. Thực hiện một vài hành động.
1. Sleep n giây
1. Lặp lại

Để có thể enable chế độ deep-sleep chúng ta cần nối pin GPI016 với pin RST trên port arduino. Cụ thể pin RST sẽ ở trạng thái HIGH trong khi module ESP8266 đang chạy. Khi pin RST nhận được tín hiệu LOW nó sẽ khởi động lại microcontroller. Trong trạng thái deep-sleep, khi thời gian ngủ đã hết nó sẽ gửi tín hiệu LOW tới pin GPIO16. Do vậy cần kết nối pin RST với pin GPIO16 để đánh thức thiết bị khi thời gian deep-sleep kết thúc.

Ví dụ:
```cpp

void setup() {
  Serial.begin(115200);
  Serial.setTimeout(2000);

  // Wait for serial to initialize.
  while(!Serial) { }
  
  Serial.println("I'm awake.");

  Serial.println("Going into deep sleep for 20 seconds");
  ESP.deepSleep(20e6); // 20e6 is 20 microseconds
}

void loop() {char dataMessage[500];
}
```

## 13. Động cơ servo

```cpp
#include <Servo.h>

#define SERVO_PIN 2  // 02, 00, 04, 05, 10, 12, 13, 14, 15 các chân 12- 15 không ổn định động cơ bị giật khi quay
Servo myservo1;
int pos = 0;
void setup() {
  myservo1.attach(SERVO_PIN);
  Serial.begin(115200);

}
void loop() {
  for ( pos = 0; pos <= 180; pos += 5) {
    myservo1.write(pos);
    Serial.println("Pos= ");
    Serial.print(pos);
    delay(100);
  }
  for ( pos = 180; pos >= 0; pos -= 5) {
    myservo1.write(pos);
    Serial.println("Pos= ");
    Serial.print(pos);
    delay(100);
  }
  delay(500);
}

```

![https://raw.githubusercontent.com/NTT-TNN/ESP_OLP/master/docs/images/servo_bb.png](https://raw.githubusercontent.com/NTT-TNN/ESP_OLP/master/docs/images/servo_bb.png)

### 14. Cảm biến mưa

Lượng mưa càng lớn số trả về càng nhỏ

```cpp
#include <Wire.h>

// IoT Wifi Uno-----Cam bien mua
//      ADC---------------A0
//      3.3V--------------VCC
//      GND---------------GND

int nRainIn = 0;          // khai bao chan ADC
int nRainVal;             // bien de doc gia tri ADC
const int timeInterval=1000;
int previousMillis=0;
void setup() {
  Serial.begin(115200);
  pinMode(2, INPUT_PULLUP);
}

void loop()
{
  currentMillis = millis();
  if(currentMillis-previousMillis>=timeInterval){
    previousMillis=currentMillis;
    nRainVal = analogRead(nRainIn);
    Serial.print("\t Moisture Level: ");
    Serial.println(nRainVal);
    
    if(nRainVal < 320)
    {
      Serial.println(":STOP");
    }
    else
    {
      Serial.println( ":Xuong");
    }
  }
}

```

![https://raw.githubusercontent.com/NTT-TNN/ESP_OLP/master/docs/images/mua_bb.png](https://raw.githubusercontent.com/NTT-TNN/ESP_OLP/master/docs/images/mua_bb.png)

### 15. BH1750

Cảm biến ánh sáng trả về cường độ ánh sáng đơn vị lux

```cpp
#include <Wire.h>
#include <BH1750.h>

BH1750 lightMeter;
uint16_t light;

void setup(){
      lightMeter.begin();

}

float getLight(){
  return lightMeter.readLightLevel() ;
}
```
![https://github.com/NTT-TNN/ESP_OLP/blob/master/docs/images/bh1750_bb.png](https://github.com/NTT-TNN/ESP_OLP/blob/master/docs/images/bh1750_bb.png)

### 16. Chuyển Động

Trả lại 1 khi có chuyển động và 0 khi không có chuyển động.

```cpp


int movePin = 02;   //using digital pin10 as input
bool Pin_Status = LOW;

void setup()
{
  pinMode(movePin, INPUT);  //input declaration
  Serial.begin(115200);
}
void loop()
{
  Pin_Status = digitalRead(movePin);
  Serial.println(Pin_Status);
  delay(2000);
}

```

![https://raw.githubusercontent.com/NTT-TNN/ESP_OLP/master/docs/images/chuyen_dong_bb.png](https://raw.githubusercontent.com/NTT-TNN/ESP_OLP/master/docs/images/chuyen_dong_bb.png)

### 17. Relay

```cpp
//Luu y ko cam relay trong luc nap code, chi cam sau khi nap xong

int pin=2;
void setup() {
  // put your setup code here, to run once:
  pinMode(pin,OUTPUT);
}

void loop() {
  // put your main code here, to run repeatedly:
  delay(2000);
  digitalWrite(pin,HIGH);
  delay(2000);
  digitalWrite(pin,LOW);
}

```
## Danh Sách các code đã làm

1. Code đăng ký newDevice lên topic của mqtt sử dụng EEPROM để sẽ chỉ gửi đăng ký topic này một lần duy nhất cho tới khi nhận được trả về xác nhận đã đăng ký thành công.
[https://github.com/NTT-TNN/ESP_OLP/tree/master/ArduinoWindows/newDevice](https://github.com/NTT-TNN/ESP_OLP/tree/master/ArduinoWindows/newDevice)

1. Code update firmware sử dụng OTA trên windows(chưa chạy được trên ubuntu)

[https://github.com/NTT-TNN/ESP_OLP/tree/master/ArduinoWindows/BasicOTA](https://github.com/NTT-TNN/ESP_OLP/tree/master/ArduinoWindows/BasicOTA)

1. Code đọc dữ liệu quan trở(Photodiode)  Giá trị trả về là điện trở tỷ lệ nghịch với cường độ ánh sáng.

[https://github.com/NTT-TNN/ESP_OLP/tree/master/example/anh_sang](https://github.com/NTT-TNN/ESP_OLP/tree/master/example/anh_sang)

1. Code sử dụng blink mức cơ bản chưa làm gì cả

[https://github.com/NTT-TNN/ESP_OLP/blob/master/example/blynk_1/blynk_1.ino](https://github.com/NTT-TNN/ESP_OLP/blob/master/example/blynk_1/blynk_1.ino)

1. Code đọc giá trị cảm biến chuyển động(HC-SR501) . Trả lại 1 có chuyển động 0 nếu không có chuyển động

[https://github.com/NTT-TNN/ESP_OLP/blob/master/example/chuyen_dong/chuyen_dong.ino](https://github.com/NTT-TNN/ESP_OLP/blob/master/example/chuyen_dong/chuyen_dong.ino)

1. Code chế độ deepsleep lưu ý docs ở trên

[https://github.com/NTT-TNN/ESP_OLP/blob/master/example/chuyen_dong/chuyen_dong.ino](https://github.com/NTT-TNN/ESP_OLP/blob/master/example/chuyen_dong/chuyen_dong.ino)

1. Đọc nhiệt độ DS18B20 hiển thị ra OLED

[https://github.com/NTT-TNN/ESP_OLP/blob/master/example/display/display.ino](https://github.com/NTT-TNN/ESP_OLP/blob/master/example/display/display.ino)

1. Đọc giá trị cảm biến mưa giá trị trả về càng lớn mưa càng to

[https://github.com/NTT-TNN/ESP_OLP/blob/master/example/mua/mua.ino4](https://github.com/NTT-TNN/ESP_OLP/blob/master/example/mua/mua.ino)

1. Code điều khiển động cơ servo

[https://github.com/NTT-TNN/ESP_OLP/blob/master/example/servo/servo.ino](https://github.com/NTT-TNN/ESP_OLP/blob/master/example/servo/servo.ino)

1. Code connect to wifi

[https://github.com/NTT-TNN/ESP_OLP/blob/master/example/wifi/wifi.ino](https://github.com/NTT-TNN/ESP_OLP/blob/master/example/wifi/wifi.ino)

1. Quét tìm tất cả các wifi có gần đây

[https://github.com/NTT-TNN/ESP_OLP/blob/master/example/scan/scan.ino](https://github.com/NTT-TNN/ESP_OLP/blob/master/example/scan/scan.ino)

1. Code đọc địa chỉ mac của arduino 

[https://github.com/NTT-TNN/ESP_OLP/blob/master/example/mac/mac.ino](https://github.com/NTT-TNN/ESP_OLP/blob/master/example/mac/mac.ino)

1. Đọc giá trị cảm biến DHT11

[https://github.com/NTT-TNN/ESP_OLP/blob/master/example/test_dht11/test_dht11.ino](https://github.com/NTT-TNN/ESP_OLP/blob/master/example/test_dht11/test_dht11.ino)


```
D0 ~ GPIO16
D1 ~ GPIO05
D2 ~ GPIO04
D3 ~ GPIO00
D4 ~ GPIO02

D5 ~ GPIO14
D6 ~ GPIO12
D7 ~ GPIO13
D8 ~ GPIO15
```


## A0 ~ ADC