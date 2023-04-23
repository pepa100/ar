```
#include <LiquidCrystal.h>

//Deklarace pro klávesnici 4x4
const int pocetRadku = 4;
const int pocetSloupcu = 4;

byte pinROWS[pocetRadku] = { A3, A2, A1, A0 };  //Přiřazení pinů
byte pinCOLS[pocetSloupcu] = { 5, 4, 3, 2 };    //Přiřazení pinů

const char hexaKeys[pocetRadku][pocetSloupcu] = {  //Deklarace keypadu
  { '1', '2', '3', 'A' },
  { '4', '5', '6', 'B' },
  { '7', '8', '9', 'C' },
  { '*', '0', '#', 'D' }
};

//Přiřazení pinů pro lcd displej
LiquidCrystal lcd(7, 8, 9, 10, 11, 12);

//Deklarace proměnných pro kód
String password = "1234";
String userCode = "";
const int pinRelay = A4;
int delayVal = 2000;

//Klavesnice
char keyPress() {
  char key = 0;  // 0 rika, ze zadna klavesa neni stisknuta

  for (int sloupec = 0; sloupec < pocetSloupcu; sloupec++) {
    digitalWrite(pinCOLS[sloupec], LOW);                // aktivace sloupce 0
    for (int radek = 0; radek < pocetRadku; radek++) {  // hleda se radek, na kterem se objevi 0
      if (digitalRead(pinROWS[radek]) == LOW) {         // je rovno 0?
        delay(20);                                      // odstraneni zakmitu
        while (digitalRead(pinROWS[radek]) == LOW)
          ;                              // ceka dokud je klavesa stisknuta
        key = hexaKeys[radek][sloupec];  // ulozi se nactena klavesa
      }
    }
    digitalWrite(pinCOLS[sloupec], HIGH);  // deaktivace aktualniho sloupce 1
    if (key |= 0) {
      return key;
    }
  }
  return key;  // vraci hodnotu klavesy nebo 0
}

void clearDisplay() {
  lcd.clear();
  lcd.write("Password:");
  userCode = "";
  lcd.setCursor(0, 1);
}

void checkPassword() {
  lcd.clear();
  if (userCode == password) {
    // rele
    digitalWrite(pinRelay, HIGH);
    lcd.write("ses buh");
    delay(delayVal);
    digitalWrite(pinRelay, LOW);

    clearDisplay();
  } else {
    lcd.write("ses kokot");
    delay(delayVal);
    clearDisplay();
  }
}

void setup() {
  pinMode(pinRelay, OUTPUT);  //Nastaveni pinu rele
  Serial.begin(9600);
  lcd.begin(16, 2);        //Nadefinujeme kolik sloupců a řádků bude mít display
  lcd.setCursor(0, 0);     //Nastaví kurzor zase podle sloupců a řádků OTESTOVAT S KLAVESNICI
  lcd.write("Password:");  //Vypíše text na display

  for (int radek = 0; radek < pocetRadku; radek++) {
    pinMode(pinROWS[radek], INPUT);      // radky na INPUT
    digitalWrite(pinROWS[radek], HIGH);  // turn on pull-ups
  }

  for (int sloupec = 0; sloupec < pocetSloupcu; sloupec++) {
    pinMode(pinCOLS[sloupec], OUTPUT);     // sloupec na writing
    digitalWrite(pinCOLS[sloupec], HIGH);  // nastavi vsechny sloupce jako neaktivni
  }
}

void loop() {
  char klavesa = keyPress();

  if (klavesa == '*') {
    clearDisplay();
  } else if (klavesa == '#') {
    checkPassword();
  } else if (klavesa) {  //Vyhodnoceni zmacknuteho tlacitka

    userCode += klavesa;
    lcd.clear();
    lcd.print("Password: " + userCode);
  }
}

```
