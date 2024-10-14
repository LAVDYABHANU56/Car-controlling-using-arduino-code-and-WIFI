# Car-controlling-using-arduino-code-and-WIFI
# Arduino code will be transferred to ESP8266 and it will connected to a motor drive in order control the car.All the runtime commands are provided in the code.A website will be created where we have runtime commands like forward,backward,left and right and even stop etc.
#include <Arduino.h>
#include <ESP8266WiFi.h>
#include <WiFiUdp.h>
#include <ArduinoJson.h>

const char* ssid = "Aether";
const char* password = "QWERTY123";

WiFiUDP Udp;
const int max_size = 255;
unsigned int localUdpPort = 4210;  
char incomingPacket[max_size];
int id = 0, pwmR = 0, pwmL = 0, state = 0;

const int motorR_IN1 = D1 ;
const int motorR_IN2 = D2 ;
const int motorL_IN3 = D3 ;
const int motorL_IN4 = D4 ;
const int motorR_EN1 = D5 ;
const int motorL_EN2 = D6 ;


void motorR(int speed){
    if (speed > 0){
        digitalWrite(motorR_IN1, 1);
        digitalWrite(motorR_IN2, 0);
        // Serial.println("R Forwards");
    }
    else if (speed < 0){
        digitalWrite(motorR_IN1, 0);
        digitalWrite(motorR_IN2, 1);
        // Serial.println("R Backwards");
    }
    else {
        digitalWrite(motorR_IN1, 0);
        digitalWrite(motorR_IN2, 0);
    }
    analogWrite(motorR_EN1, abs(speed));
}

void motorL(int speed){
    if (speed > 0){
        digitalWrite(motorL_IN3, 1);
        digitalWrite(motorL_IN4, 0);
        // Serial.println("L Forwards");
    }
    else if (speed < 0){
        digitalWrite(motorL_IN3, 0);
        digitalWrite(motorL_IN4, 1);
        // Serial.println("L Backwards");
    }
    else {
        digitalWrite(motorL_IN3, 0);
        digitalWrite(motorL_IN4, 0);
    }
    analogWrite(motorL_EN2, abs(speed));
}

// void smoothHalt(int stop_time) {
//     pwmL = pwmR;
//     for (int i = pwmL; i > 0; i--) {
//         motorL(i);
//         motorR(i);
//         delay(stop_time);
//     }
// }

void setup() {
  pinMode(motorR_IN1, OUTPUT);
  pinMode(motorR_IN2, OUTPUT);
  pinMode(motorL_IN3, OUTPUT);
  pinMode(motorL_IN4, OUTPUT);
  pinMode(motorR_EN1, OUTPUT);
  pinMode(motorL_EN2, OUTPUT);
  motorL(0);
  motorR(0);
  Serial.begin(115200);
  Serial.println();

  Serial.printf("Connecting to %s ", ssid);
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println(" connected");

  WiFi.setSleepMode(WIFI_NONE_SLEEP);
  Udp.begin(localUdpPort);
  Serial.printf("Now listening at IP %s, UDP port %d\n", WiFi.localIP().toString().c_str(), localUdpPort);
}


void loop() {
  // pwmL = 0; 
  // pwmR = 0;
  DynamicJsonDocument doc(200);
  int packetSize = Udp.parsePacket();
  if (packetSize) {
    int len = Udp.read(incomingPacket, max_size);
    if (len > 0)
    {
      incomingPacket[len] = 0;
    }
    DeserializationError error = deserializeJson(doc, incomingPacket);
        if (error) {
            Serial.println("parseObject() failed");
        }
    id = doc["id"];
    pwmL = doc["pwmL"];
    pwmR = doc["pwmR"];
    state = doc["state"];
    //Serial.printf("\n id = %d L - %d , R - %d, S - %d \n", id, pwmL, pwmR, state);
    // Serial.println(id);
    motorL(pwmL);
    motorR(pwmR);
    }
}

