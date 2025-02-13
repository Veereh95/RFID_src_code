//#RFID_src_code
//Arduino source code for RFID door lock system

#include <SPI.h>
#include <MFRC522.h>
#include <Servo.h>

#define SS_PIN 10
#define RST_PIN 9
#define LED_G 5 //define green LED pin
#define LED_R 4 //define red LED
#define BUZZER 2 //buzzer pin
MFRC522 mfrc522(SS_PIN, RST_PIN);   // Create MFRC522 instance.
Servo myServo; //define servo name

// Define authorized UIDs (each UID contains 4 bytes, adjust size accordingly)
byte authorizedUIDs[][4] = {
  {0x73, 0xB8, 0xA2, 0x2E}, // UID 1
  {0x1A, 0xB0, 0xAE, 0x80}, // UID 2
  {0x34, 0x56, 0x78, 0x9A}  // UID 3
};
const int numAuthorizedUIDs = sizeof(authorizedUIDs) / sizeof(authorizedUIDs[0]);

void setup() 
{
  Serial.begin(9600);   // Initiate a serial communication
  SPI.begin();      // Initiate  SPI bus
  mfrc522.PCD_Init();   // Initiate MFRC522
  myServo.attach(3); //servo pin
  myServo.write(0); //servo start position
  pinMode(LED_G, OUTPUT);
  pinMode(LED_R, OUTPUT);
  pinMode(BUZZER, OUTPUT);
  noTone(BUZZER);
  Serial.println("Put your card to the reader...");
  Serial.println();
}

void loop() 
{
  // Look for new cards
  if (!mfrc522.PICC_IsNewCardPresent()) 
  {
    return;
  }

  // Select one of the cards
  if (!mfrc522.PICC_ReadCardSerial()) 
  {
    return;
  }

  // Show UID on serial monitor
  Serial.print("UID tag: ");
  byte currentUID[4];
  for (byte i = 0; i < mfrc522.uid.size; i++) 
  {
    Serial.print(mfrc522.uid.uidByte[i] < 0x10 ? " 0" : " ");
    Serial.print(mfrc522.uid.uidByte[i], HEX);
    currentUID[i] = mfrc522.uid.uidByte[i]; // Store current UID
  }
  Serial.println();

  // Check if the UID is authorized
  if (isAuthorized(currentUID)) 
  {
    Serial.println("Authorized access");
    delay(500);
    digitalWrite(LED_G, HIGH);
    tone(BUZZER, 500);
    delay(300);
    noTone(BUZZER);
    myServo.write(90); // Rotate servo by 90 degrees
    delay(5000);
    myServo.write(0);
    digitalWrite(LED_G, LOW);
  } 
  else 
  {
    Serial.println("Access denied");
    digitalWrite(LED_R, HIGH);
    tone(BUZZER, 300);
    delay(1000);
    digitalWrite(LED_R, LOW);
    noTone(BUZZER);
  }
}

// Function to check if the scanned UID matches any authorized UID
bool isAuthorized(byte currentUID[]) 
{
  for (int i = 0; i < numAuthorizedUIDs; i++) 
  {
    bool match = true;
    for (int j = 0; j < 4; j++) 
    {
      if (authorizedUIDs[i][j] != currentUID[j]) 
      {
        match = false;
        break;
      }
    }
    if (match) 
    {
      return true;
    }
  }
  return false;
}
