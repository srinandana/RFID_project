# RFID_project


// Included Libraries
#include <ESP8266WiFi.h>
#include "HTTPSRedirect.h"
#include <SPI.h>
#include <MFRC522.h>
#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>

const char* ssid = "Galaxy M30sD3D7";
const char* password = "nxen0150";
// pins assignment
constexpr uint8_t RST_PIN = 5;     // Configurable, see typical pin layout above
constexpr uint8_t SS_PIN = 4;     // Configurable, see typical pin layout above
#define screen_width 128
#define screen_height 64

Adafruit_SSD1306 display(-1);
MFRC522 mfrc522(SS_PIN, RST_PIN);   // Create MFRC522 instance.
int j=56;

const char* GScriptId =
"AKfycbw1JtQ4bKM3UgnwLamcFwrKqFgnnw1CLsRO2c7lsQDFMhHgqd2iHCA8qEUGA5ebynoWIA";
const int dataPostDelay = 3000;    // 3 seconds
const char* host = "script.google.com";
const char* googleRedirHost = "script.googleusercontent.com";
const int httpsPort =     443;

HTTPSRedirect client(httpsPort);

// Prepare the url (without the varying data)
String url = String("/macros/s/") + GScriptId + "/exec?";

const char* fingerprint = "F0 5C 74 77 3F 6B 25 D7 3B 66 4D 43 2F 7E
BC 5B E9 28 86 AD";

void setup() {
    Serial.begin(115200);
    Serial.println("Connecting to wifi: ");
    Serial.println(ssid);
    Serial.flush();

  SPI.begin();      // Initiate  SPI bus
  mfrc522.PCD_Init();   // Initiate MFRC522
  Serial.println("Approximate your card to the reader...");
  Serial.println();

  display.begin(SSD1306_SWITCHCAPVCC,0x3C);
  display.clearDisplay();
  display.setTextSize(2);
  display.setTextColor(SSD1306_WHITE);
  display.setCursor(0,0);
  display.print("Count");
  display.display();

    WiFi.begin(ssid, password);
    while (WiFi.status() != WL_CONNECTED) {
            delay(500);
            Serial.print(".");
    }
    Serial.println(" IP address: ");
    Serial.println(WiFi.localIP());


    Serial.print(String("Connecting to "));
    Serial.println(host);
    bool flag = false;
    for (int i=0; i<5; i++){
            int retval = client.connect(host, httpsPort);
            if (retval == 1) {
                        flag = true;
                        break;
            }
            else
                    Serial.println("Connection failed. Retrying…");
    }

    // Connection Status, 1 = Connected, 0 is not.
    Serial.println("Connection Status: " + String(client.connected()));
    Serial.flush();

    if (!flag){
            Serial.print("Could not connect to server: ");
            Serial.println(host);
            Serial.println("Exiting…");
            Serial.flush();
            return;
    }

    // Data will still be pushed even certification don’t match.
   /* if (client.verify(fingerprint, host)) {
            Serial.println("Certificate match.");
    } else {
            Serial.println("Certificate mis-match");
    }*/


}


// This is the main method where data gets pushed to the Google sheet
void postData(String tag, float value){
    if (!client.connected()){
            Serial.println("Connecting to client again…");
            client.connect(host, httpsPort);
    }
    String urlFinal = url + "tag=" + tag + "&value=" + String(value);
    client.printRedir(urlFinal, host, googleRedirHost);
}


// Continue pushing data at a given interval
void loop() {
     // Look for new cards
  if ( ! mfrc522.PICC_IsNewCardPresent())
  {
    return;
  }
  // Select one of the cards
  if ( ! mfrc522.PICC_ReadCardSerial())
  {
    return;
  }
  //Show UID on serial monitor
  Serial.print("UID tag :");
  String content= "";
  byte letter;
  for (byte i = 0; i < mfrc522.uid.size; i++)
  {
     Serial.print(mfrc522.uid.uidByte[i] < 0x10 ? " 0" : " ");
     Serial.print(mfrc522.uid.uidByte[i], HEX);
     content.concat(String(mfrc522.uid.uidByte[i] < 0x10 ? " 0" : " "));
     content.concat(String(mfrc522.uid.uidByte[i], HEX));
  }
  Serial.println();
  Serial.print("Message : ");
  content.toUpperCase();
  if (content.substring(1) == "01 07 16 DB") //change here the UID of
the card/cards that you want to give access
  {
    Serial.println("accepted");
    Serial.println();
    j++;
    Serial.println(j);

    tcount();
    display.display();
    int data = j;
    postData("Count",data);
    delay(dataPostDelay);

  }

 else   {
    Serial.println(" Access denied");
    delay(30);
  }



}


void tcount(void)
{
  display.clearDisplay();
  display.setTextSize(2);
  display.setTextColor(SSD1306_WHITE);
  display.setCursor(0,0);
  display.print("Count");
  display.setCursor(40,40);
  display.print(j);
  display.display();


}
