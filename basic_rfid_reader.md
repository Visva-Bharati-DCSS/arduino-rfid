## Day 2: A simple RFID authenticator.

### Tools
- Arduino Uno
- Breadboard
- RFID-RC522 sensor
- 1602 LCD
- Connecting wires

### Objective of the day
- Print "Authorized" in LCD if ID is authorized.
- Print "Not authorized" if ID is not authorized.
- If the ID has been scanned recently, print "Already scanned".

### Circuit
<img src="rfid_reader.svg" width=800>

### Code
rfid.cpp
```
#pragma once

#include <MFRC522.h>

const int RFID_MAX = 10;


class Member {
public:
  String name;
  String rfid;

  Member(String name, String rfid)
  : name(name), rfid(rfid) {}
};


class RFID {
private:
  int length;
  byte uid[RFID_MAX];

public:
  RFID() {
    length = 4;
    for (int i = 0; i < RFID_MAX; ++i)
      uid[i] = 0;
  }

  RFID(MFRC522::Uid uid) {
    setUID(uid);
  }

  void setUID(MFRC522::Uid uid) {
    length = uid.size;
    for (int i = 0; i < length; ++i)
      this->uid[i] = uid.uidByte[i];
  }

  String toString() {
    String s;
    if (length <= 0) return s;                    // empty string if no length
    if (uid[0] < 0x10) s += '0';                  // first byte
    s += String(uid[0], 16);
    for (int i = 1; i < length; ++i) {            // subsequent bytes prepended with space
      s += '-';
      if (uid[i] < 0x10) s += '0';
      s += String(uid[i], 16);
    }
    s.toUpperCase();
    return s;
  }

  bool operator==(const RFID rfid) {
    if (length != rfid.length)
      return false;
    for (int i = 0; i < length; ++i)
      if (uid[i] != rfid.uid[i])
        return false;
    return true;
  }

  bool operator==(const String rfid) {
    return rfid == toString();
  }
};
```
rfid_rc522.ino
```
#include <SPI.h>
#include <MFRC522.h>
#include <LiquidCrystal.h>

#include "rfid.cpp"

const int V0 = A5;
const int RS = 6;
const int EN = 7;
const int D4 = 5;
const int D5 = 4;
const int D6 = 3;
const int D7 = 2;

const int MFRC_RST = 9;
const int MFRC_SDA = 10;

/*
For SPI communication with MFRC522
  MOSI -> 11
  MISO -> 12
  SCK  -> 13
*/

MFRC522 mfrc = MFRC522(MFRC_SDA, MFRC_RST);
LiquidCrystal LCD = LiquidCrystal(RS, EN, D4, D5, D6, D7);

RFID rfid = RFID();

const Member authorized[] = {
  Member("Bijon", "13-B3-87-80"),
  Member("Niloy", "65-A1-FC-AA")
};

const int lenAuthorized = sizeof(authorized) / sizeof(Member);


void setup() {
  Serial.begin(9600);
  SPI.begin();
  mfrc.PCD_Init();
  pinMode(V0, OUTPUT);
  Serial.println("\n------ New Session -----\n");
  Serial.println("Initialized RFID: " + rfid.toString() + '\n');
  analogWrite(V0, 100);
  LCD.begin(16, 2);
  LCD.print("Hello!");
}


void loop() {
  // no card
  if (!mfrc.PICC_IsNewCardPresent() || !mfrc.PICC_ReadCardSerial()) {
    delay(500);
    return;
  }
  // same card
  LCD.clear();
  if (rfid == RFID(mfrc.uid)) {
    LCD.print("Already scanned.");
    Serial.println("Already scanned.");
    delay(500);
    return;
  }
  // new card
  rfid.setUID(mfrc.uid);
  Serial.println("Scanned RFID: " + rfid.toString());
  for (int i = 0; i < lenAuthorized; ++i) {
    if (rfid == authorized[i].rfid) {
      LCD.print("Authorized.");
      LCD.setCursor(0, 1);
      LCD.print("Member " + authorized[i].name + ".");
      Serial.println("Authorized.");
      delay(500);
      return;
    }
  }
  // not authorized
  LCD.print("Not authorized.");
  Serial.println("Not authorized.");
  delay(500);
}
```