# Heart-Rate-Monitoring-System-_-IoT
The IoT-Based Heart Rate Monitoring System is a biomedical and IoT project designed to measure a patient’s heart rate (pulse rate) in real time and send the data to the Internet (IoT platform) for remote monitoring.
#include <ESP8266WiFi.h>
#include "ThingSpeak.h"

// WiFi Credentials
const char* ssid = "Your_WiFi_Name";
const char* password = "Your_WiFi_Password";

// ThingSpeak Settings
unsigned long channelID = YOUR_CHANNEL_ID;  // Example: 1234567
const char* writeAPIKey = "YOUR_API_KEY";   // Get this from ThingSpeak

WiFiClient client;

int pulsePin = A0;   // Pulse sensor connected to A0
int signalValue = 0;
int threshold = 550;  // Adjust based on your sensor output

long lastBeatTime = 0;
int beatCount = 0;
int bpm = 0;

void setup() {
  Serial.begin(115200);
  WiFi.begin(ssid, password);
  Serial.print("Connecting to WiFi");

  // Connect to WiFi
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.print(".");
  }
  Serial.println("\nWiFi connected!");
  ThingSpeak.begin(client);
}

void loop() {
  signalValue = analogRead(pulsePin);
  
  // Detect heartbeat (simple peak detection)
  if (signalValue > threshold) {
    long currentTime = millis();
    long delta = currentTime - lastBeatTime;
    if (delta > 300) {   // Minimum time between beats (avoid noise)
      lastBeatTime = currentTime;
      beatCount++;
      bpm = 60000 / delta;   // Convert time gap to BPM
      Serial.print("Heartbeat detected! BPM: ");
      Serial.println(bpm);

      // Upload to ThingSpeak
      ThingSpeak.writeField(channelID, 1, bpm, writeAPIKey);
      Serial.println("Data sent to ThingSpeak\n");
    }
  }

  delay(100);  // Small delay to stabilize reading
}
