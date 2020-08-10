#include <ESP8266WiFi.h>
 
String apiKey = "VPBBU2ZQN4KOJZES";     //  Enter your Write API key from ThingSpeak
const char *ssid =  "Hp Android";     // replace with your wifi ssid and wpa2 key
const char *pass =  "tulisnama";
const char* server = "api.thingspeak.com";
WiFiClient client;

#define SERIAL Serial
#define sensorPin A0
 
int sensorValue = 0;
float tdsValue = 0;
float Voltage = 0;

void setup() 
{
  SERIAL.begin(115200);
       Serial.println("Connecting to ");
       Serial.println(ssid);
 
 
       WiFi.begin(ssid, pass);
 
      while (WiFi.status() != WL_CONNECTED) 
     {
            delay(1000);
            Serial.print(".");
     }
      Serial.println("");
      Serial.println("WiFi connected");
 
}
 
void loop() 
{
  
      
      float pHValue;
      
              if ( isnan(tdsValue)) 
                 {
                     Serial.println("Failed to read from DHT sensor!");
                      return;
                 }

                         if (client.connect(server,80))   //   "184.106.153.149" or api.thingspeak.com
                      {  
                            
                             String postStr = apiKey;
                             postStr +="&field1=";
                             postStr += String(tdsValue);
                             //postStr +="&field2=";
                             //postStr += String(h);
                             postStr += "\r\n\r\n";
 
                             client.print("POST /update HTTP/1.1\n");
                             client.print("Host: api.thingspeak.com\n");
                             client.print("Connection: close\n");
                             client.print("X-THINGSPEAKAPIKEY: "+apiKey+"\n");
                             client.print("Content-Type: application/x-www-form-urlencoded\n");
                             client.print("Content-Length: ");
                             client.print(postStr.length());
                             client.print("\n\n");
                             client.print(postStr);
                             
sensorValue = analogRead(sensorPin);
    Voltage = sensorValue*3/1024.0; //Convert analog reading to Voltage
    tdsValue=(133.42*Voltage*Voltage*Voltage - 255.86*Voltage*Voltage + 857.39*Voltage)*0.5; //Convert voltage value to TDS value
    SERIAL.print("TDS Value = "); 
    SERIAL.print(tdsValue);
    SERIAL.println(" ppm");
    delay(1000);
                             //Serial.print("Temperature: ");
                             //Serial.print(t);
                             //Serial.print(" degrees Celcius, Humidity: ");
                             //Serial.print(h);
                             Serial.println("%. Send to Thingspeak.");
                        }
          client.stop();
 
          Serial.println("Waiting...");
  
  // thingspeak needs minimum 15 sec delay between updates, i've set it to 30 seconds
  delay(1000);
}
