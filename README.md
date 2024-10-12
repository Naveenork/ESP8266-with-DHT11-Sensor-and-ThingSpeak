# ESP8266-with-DHT11-Sensor-and-ThingSpeak
This project demonstrates how to use an ESP8266 Wi-Fi module with a DHT11 temperature and humidity sensor to send data to ThingSpeak, an IoT cloud platform. The ESP8266 connects to a Wi-Fi network and sends temperature and humidity readings from the DHT11 sensor to ThingSpeak at regular intervals.

# Components
1 x ESP8266 (e.g., NodeMCU)
1 x DHT11 Sensor
1 x 10kΩ resistor (optional, for stable data reading)
Breadboard and jumper wires
Circuit Diagram
Connect VCC of the DHT11 sensor to the 3.3V pin of the ESP8266.
Connect GND of the DHT11 sensor to the GND pin of the ESP8266.
Connect the DATA pin of the DHT11 sensor to digital pin D1 on the ESP8266.
(Optional) Place a 10kΩ resistor between the VCC and DATA pin of the DHT11 for better stability.
# Prerequisites
Arduino IDE with ESP8266 board support installed.
DHT library installed in the Arduino IDE (use the Library Manager to install the DHT sensor library).
A ThingSpeak account with a created channel to store data and a Write API key.

# Code Explanation

#include <ESP8266WiFi.h>
#include <DHT.h>

String apiKey = "HPITSE4SVJY0UODC";  // Enter your Write API key from ThingSpeak

const char* ssid = "NAVEENKUMAR 6966";  // Wi-Fi SSID
const char* pass = "12345678";          // Wi-Fi password
const char* server = "api.thingspeak.com";

#define DHTPIN D1  // Pin where the DHT11 is connected
DHT dht(DHTPIN, DHT11);  // Initialize DHT sensor

WiFiClient client;

void setup() {
  Serial.begin(115200);
  delay(10);
  dht.begin();

  Serial.println("Connecting to ");
  Serial.println(ssid);

  WiFi.begin(ssid, pass);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("");
  Serial.println("WiFi connected");
}

void loop() {
  float h = dht.readHumidity();
  float t = dht.readTemperature();

  if (isnan(h) || isnan(t)) {
    Serial.println("Failed to read from DHT sensor!");
    return;
  }

  if (client.connect(server, 80)) {
    String postStr = apiKey;
    postStr += "&field1=" + String(t);
    postStr += "&field2=" + String(h);
    postStr += "\r\n\r\n";

    client.print("POST /update HTTP/1.1\n");
    client.print("Host: api.thingspeak.com\n");
    client.print("Connection: close\n");
    client.print("X-THINGSPEAKAPIKEY: " + apiKey + "\n");
    client.print("Content-Type: application/x-www-form-urlencoded\n");
    client.print("Content-Length: ");
    client.print(postStr.length());
    client.print("\n\n");
    client.print(postStr);

    Serial.print("Temperature: ");
    Serial.print(t);
    Serial.print(" degrees Celsius, Humidity: ");
    Serial.print(h);
    Serial.println("%. Sent to ThingSpeak.");
  }
  
  client.stop();
  Serial.println("Waiting...");

  // ThingSpeak requires at least 15 seconds delay between updates
  delay(15000);
}
# How to Use
Copy the code into the Arduino IDE.
Replace apiKey with your ThingSpeak Write API key.
Upload the code to the ESP8266 board.
Open the Serial Monitor (set the baud rate to 115200).
The ESP8266 will connect to the specified Wi-Fi network and send temperature and humidity data to ThingSpeak every 15 seconds.
Customization
To change the Wi-Fi credentials, modify the ssid and pass variables.
Adjust the data sending interval by modifying the delay() at the end of the loop() function. Ensure it’s at least 15 seconds to meet ThingSpeak's rate limits.
To change the DHT sensor pin, modify the #define DHTPIN line.
Troubleshooting
If the ESP8266 is unable to connect to Wi-Fi, check the SSID and password.
If you see "Failed to read from DHT sensor!" in the Serial Monitor, verify the wiring and the sensor's connection.
Ensure that the DHT library is installed and up-to-date in the Arduino IDE.
# License
This project is open-source and available for personal and educational use.
