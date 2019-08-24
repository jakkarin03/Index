#include <TridentTD_LineNotify.h>
#define SSID        "Admin-Wifi-2.4GHz"      // บรรทัดที่ 11 ให้ใส่ ชื่อ Wifi ที่จะเชื่อมต่อ
#define PASSWORD    "seam2448860"     // บรรทัดที่ 12 ใส่ รหัส Wifi
#define LINE_TOKEN  "rdobj9lcUfqR1y95fejUAoWpWnAY2lXVVXlRgaikxG7"   // บรรทัดที่ 13 ใส่ รหัส TOKEN ที่ได้มาจากข้างบน
#include "DHT.h"
#define DHTPIN D1
#define DHTTYPE DHT11
DHT dht(DHTPIN, DHTTYPE);
void setup() {
  dht.begin();
  Serial.begin(115200); Serial.println();
  Serial.println(LINE.getVersion());

  WiFi.begin(SSID, PASSWORD);
  Serial.printf("WiFi connecting to %s\n",  SSID);
  while (WiFi.status() != WL_CONNECTED) {
    Serial.print(".");
    delay(400);
  }
  Serial.printf("\nWiFi connected\nIP : ");
  Serial.println(WiFi.localIP());

  // กำหนด Line Token
  LINE.setToken(LINE_TOKEN);
  LINE.notify("myarduino.net");
}

void loop() {
  delay(2000);
  float h = dht.readHumidity();
  float t = dht.readTemperature();
  float f = dht.readTemperature(true);
  if (isnan(h) || isnan(t) || isnan(f)) {
    Serial.println(F("Failed to read from DHT sensor!"));
    return;
  }
  float hif = dht.computeHeatIndex(f, h);
  float hic = dht.computeHeatIndex(t, h, false);

  Serial.print(F("Humidity: "));
  Serial.print(h);
  Serial.print(F("%  Temperature: "));
  Serial.print(t);
  Serial.print(F("°C "));
  Serial.print(f);
  Serial.print(F("°F  Heat index: "));
  Serial.print(hic);
  Serial.print(F("°C "));
  Serial.print(hif);
  Serial.println(F("°F"));
  if (t > 40) {
    String LineText;
    String string1 = "อุณหภูมิ เกินกำหนด ";
    String string2 = " °C";
    LineText = string1 + t + string2;
    Serial.print("Line ");
    Serial.println(LineText);
    LINE.notify(LineText);
  }


}
