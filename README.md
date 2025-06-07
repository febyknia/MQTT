
>👩🏻‍💻Feby 🗓 7 June 2025

# MQTT-BASED ENVIRONMENT MONITORING WITH ESP32 AND DHT22


---
## **A. Introduction**

This practicum implements an Internet of Things (IoT)-based weather monitoring system using an ESP32 microcontroller and MQTT protocol for real-time data communication. The ESP32 is connected to a Wi-Fi network and connects to a public MQTT broker (test.mosquitto.org) to send and receive messages on specific topics. 

## **B. Diagram**
This diagram was created using the ESP32 Microcontroller based Wokwi simulation.
<img width="1020" alt="2" src="https://github.com/user-attachments/assets/e32cbcb2-942d-47e2-a364-f467f76bf7bf" />








```json
{
  "version": 1,
  "author": "Subairi",
  "editor": "wokwi",
  "parts": [
    { "type": "wokwi-esp32-devkit-v1", "id": "esp", "top": 0, "left": 0, "attrs": {} },
    { "type": "wokwi-dht22", "id": "dht1", "top": 0.3, "left": -111, "attrs": {} },
    {
      "type": "wokwi-led",
      "id": "led1",
      "top": -3.6,
      "left": 157.8,
      "attrs": { "color": "red", "flip": "1" }
    }
  ],
  "connections": [
    [ "esp:TX0", "$serialMonitor:RX", "", [] ],
    [ "esp:RX0", "$serialMonitor:TX", "", [] ],
    [ "dht1:GND", "esp:GND.2", "black", [ "v0" ] ],
    [ "dht1:VCC", "esp:3V3", "black", [ "v0" ] ],
    [ "dht1:SDA", "esp:D15", "black", [ "v0" ] ],
    [ "led1:A", "esp:D2", "black", [ "v0" ] ],
    [ "led1:C", "esp:GND.1", "black", [ "v0" ] ]
  ],
  "dependencies": {}
}



```
## **C. Program Code**
This program code is for setting the temperature and humidity, as well as the connection to Blynk.

```cpp
#include <WiFi.h>
#include <PubSubClient.h>
#include <DHTesp.h>

const int LED_RED = 2;
const int DHT_PIN = 15;
DHTesp dht; 

const char* ssid = "Wokwi-GUEST";
const char* password = "";
const char* mqtt_server = "test.mosquitto.org";

WiFiClient espClient;
PubSubClient client(espClient);
unsigned long lastMsg = 0;
float temp = 0;
float hum = 0;

void setup_wifi() { 
  delay(10);
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);

  WiFi.mode(WIFI_STA); 
  WiFi.begin(ssid, password); 

  while (WiFi.status() != WL_CONNECTED) { 
    delay(500);
    Serial.print(".");
  }

  randomSeed(micros());
  Serial.println("");
  Serial.println("WiFi connected");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());
}

void callback(char* topic, byte* payload, unsigned int length) { 
  Serial.print("Message arrived [");
  Serial.print(topic);
  Serial.print("] ");
  for (int i = 0; i < length; i++) { 
    Serial.print((char)payload[i]);
  }
  Serial.println();

  if ((char)payload[0] == '1') {
    digitalWrite(LED_RED, HIGH);  
  } else {
    digitalWrite(LED_RED, LOW); 
  }
}

void reconnect() { 
  while (!client.connected()) {
    Serial.print("Attempting MQTT connection...");

    String clientId = "ESP32Client-";
    clientId += String(random(0xffff), HEX);
  
    if (client.connect(clientId.c_str())) {
      Serial.println("Connected");
    
      client.publish("IOT/Test1/mqtt", "Test IOT"); 
      client.subscribe("IOT/Test1/mqtt"); 
    } else {
      Serial.print("failed, rc=");
      Serial.print(client.state());
      Serial.println(" try again in 5 seconds");
      
      delay(5000);
    }
  }
}

void setup() {
  pinMode(LED_RED, OUTPUT);     
  Serial.begin(115200);
  setup_wifi(); 
  client.setServer(mqtt_server, 1883); 
  client.setCallback(callback);
  dht.setup(DHT_PIN, DHTesp::DHT22);
}

void loop() {
  if (!client.connected()) {
    reconnect();
  }
  client.loop();

  unsigned long now = millis();
  if (now - lastMsg > 2000) { 
    lastMsg = now;
    TempAndHumidity  data = dht.getTempAndHumidity();

    String temp = String(data.temperature, 2); 
    client.publish("IOT/Test1/temp", temp.c_str()); 
    
    String hum = String(data.humidity, 1); 
    client.publish("IOT/Test1/hum", hum.c_str()); 

    Serial.print("Temperature: ");
    Serial.println(temp);
    Serial.print("Humidity: ");
    Serial.println(hum);
  }
}

```
## **C. Result**
The ESP32 can connect to the WiFi network with IP address 10.13.37.2 and also successfully connect to the MQTT server.  The information obtained includes the DHT22 sensor consistently sending temperature data of 24.00°C and humidity of 40.0%, which is displayed repeatedly on the terminal. This shows that the sensor readings are stable without interference. 
<img width="1020" alt="1" src="https://github.com/user-attachments/assets/21395080-218c-44b0-9ab9-525b99906c3b" />


For more details, please follow the implementation steps in the MQTT check report (pdf).
