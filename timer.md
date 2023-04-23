```
int prodleva = 75;
const int resetBtn = 2;
const int buttonStop = 11;
int buttonState = 0;
int buttonStopState = 0;

struct disp4x {
  int cislo = 0;
  byte disp[4] = { 0, 0, 0, 0 };
  byte cisloDisp = 0;
  unsigned int portDisp[4] = { A3, A2, A1, A0 };

  unsigned int portSegment[7] = { 3, 4, 5, 6, 7, 8, 9 };               
};

//                     a, b, c, d, e, f, g
int dekoder[10][7] = { { 1, 1, 1, 1, 1, 1, 0 },
                       { 0, 1, 1, 0, 0, 0, 0 },
                       { 1, 1, 0, 1, 1, 0, 1 },
                       { 1, 1, 1, 1, 0, 0, 1 },
                       { 0, 1, 1, 0, 0, 1, 1 },
                       { 1, 0, 1, 1, 0, 1, 1 },
                       { 1, 0, 1, 1, 1, 1, 1 },
                       { 1, 1, 1, 0, 0, 0, 0 },
                       { 1, 1, 1, 1, 1, 1, 1 },
                       { 1, 1, 1, 1, 0, 1, 1 } };

disp4x displej;
unsigned int minuty;
unsigned int sekundy;
unsigned int pocetOpakovani = 0;

void resetSegmentu() {
  for (unsigned int i = 0; i < sizeof(displej.portSegment); i++) {
    digitalWrite(displej.portSegment[i], LOW);
  }
  for (unsigned int i = 0; i < sizeof(displej.portDisp); i++) {
    digitalWrite(displej.portDisp[i], HIGH);
  }
}

void printSegments(int cislo) {
  if (cislo >= 0 && cislo <= 9) {
    for (unsigned int pin = 0; pin < sizeof(displej.portSegment); pin++) {
      digitalWrite(displej.portSegment[pin], dekoder[cislo][pin]);
    }
  }
}

void printCislo(int cislo) {
  unsigned int poradi = 0;
  while (poradi < 4) {
    if (cislo > 0) {
      displej.disp[poradi] = cislo % 10;
      cislo = cislo / 10;
    } else {
      displej.disp[poradi] = 0;
    }
    poradi++;
  }
}

void setCislo(int minuty, int sekundy) {
  displej.cislo = sekundy + 100 * minuty;
}


void setup() {
  Serial.begin(9600);
  for (unsigned int i = 0; i < sizeof(displej.portSegment); i++) {
    pinMode(displej.portSegment[i], OUTPUT);
  }
  for (unsigned int i = 0; i < sizeof(displej.portDisp); i++) {
    pinMode(displej.portDisp[i], OUTPUT);
  }

  setCislo(minuty, sekundy);
  pinMode(resetBtn, INPUT);
  pinMode(buttonStop, INPUT);
  resetSegmentu();

  noInterrupts();
  TCNT1 = 0;
  TCCR1A = 0b00000000;
  TCCR1B = 0b00001101;
  TIMSK1 = 0b00000010;
  OCR1A = prodleva;


  interrupts();
}


void loop() {
  buttonState = digitalRead(resetBtn);
  buttonStopState = digitalRead(buttonStop);
  if (buttonState == LOW) {
    sekundy = 0;
    minuty = 0;
  }

  if (buttonStopState == LOW) {

    static unsigned long startMillis = millis();
    if (millis() - startMillis >= 1000) {
      startMillis = millis();
      sekundy += 1;
      Serial.println(sekundy);


      if (sekundy == 60) {
        minuty += 1;
        sekundy = 0;
      } else if (minuty == 60) {
        minuty = 0;
        sekundy = 0;
      }
    }
  }

  setCislo(minuty, sekundy);
  printCislo(displej.cislo);
}

ISR(TIMER1_COMPA_vect) {
  digitalWrite(displej.portDisp[displej.cisloDisp], HIGH);
  displej.cisloDisp = (displej.cisloDisp < 3 ? displej.cisloDisp + 1 : 0);
  printSegments(displej.disp[displej.cisloDisp]);
  digitalWrite(displej.portDisp[displej.cisloDisp], LOW);
  pocetOpakovani++;
}
```
