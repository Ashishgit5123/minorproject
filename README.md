# minorproject
vehicle entry and exit managment system
  code of minor project
#include <WiFi.h>
#include <ESP32Servo.h>
#include <SPI.h>
#include <MFRC522.h>
#include <HTTPClient.h>

#define SS_PIN 21  
#define RST_PIN 22  

Servo myservo;  
const int servoPin = 13;  

const char* ssid = "vivo V27"; 
const char* password = "Ashish@...";     
const char* apiKey = "Q1W3UG5D2QJPLWFJ";  
MFRC522 mfrc522(SS_PIN, RST_PIN); 

void setup() {
    Serial.begin(115200); 
    myservo.attach(servoPin);  
    WiFi.begin(ssid, password);  

    while (WiFi.status() != WL_CONNECTED) {
        delay(1000);
        Serial.println("Connecting to WiFi...");
    }
    Serial.println("Connected to WiFi");

    SPI.begin();        
    mfrc522.PCD_Init(); 
    Serial.println("Scan a RFID card/tag...");
}

void loop() {
    // Look for a new card
    if (!mfrc522.PICC_IsNewCardPresent()) {
        return;
    }

    // Select one of the cards
    if (!mfrc522.PICC_ReadCardSerial()) {
        return;
    }

    // Show UID on serial monitor
    String rfid = "";
    Serial.print("UID tag: ");
    for (byte i = 0; i < mfrc522.uid.size; i++) {
        rfid += String(mfrc522.uid.uidByte[i], HEX);
        Serial.print(mfrc522.uid.uidByte[i] < 0x10 ? " 0" : " ");
        Serial.print(mfrc522.uid.uidByte[i], HEX);
    }
    Serial.println();
    rfid.toUpperCase();    
    // Send RFID data to ThingSpeak
    if (WiFi.status() == WL_CONNECTED) {
        HTTPClient http;
        String url = String(server) + "?api_key=" + apiKey + "&field1=" + rfid;
        http.begin(url);
        int httpCode = http.GET();
        if (httpCode > 0) {
            String payload = http.getString();
            Serial.println("Response payload: " + payload);
        } else {
            Serial.println("Error in HTTP request: " + String(http.errorToString(httpCode).c_str()));
        }
        http.end();
    }

    // Activate the servo motor
    myservo.write(0);  
    delay(5000);       
    myservo.write(90);   
    delay(1000);        

    mfrc522.PICC_HaltA();
    delay(10000);  // Wait 10 seconds before next read
}

