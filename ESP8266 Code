#include <ESP8266WiFi.h>
#include <FirebaseESP8266.h>

#define WIFI_SSID "YOUR_WIFI"
#define WIFI_PASSWORD "YOUR_PASSWORD"

#define FIREBASE_HOST "parking-system-fb7c4-default-rtdb.firebaseio.com"
#define FIREBASE_AUTH "YOUR_DATABASE_SECRET"

FirebaseData fb;

void setup() {
  Serial.begin(115200);

  WiFi.begin(WIFI_SSID, WIFI_PASSWORD);
  Serial.print("Connecting");

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  Serial.println("\nWiFi Connected");

  Firebase.begin(FIREBASE_HOST, FIREBASE_AUTH);
}

void loop() {
  if (Firebase.getString(fb, "/parking/slot")) {
    String slot = fb.stringData();

    Serial.print("Slot from Firebase: ");
    Serial.println(slot);
  } else {
    Serial.print("Error: ");
    Serial.println(fb.errorReason());
  }

  delay(1000);
}
