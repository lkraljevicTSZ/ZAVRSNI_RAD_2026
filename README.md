#define PODATAK_PIN A3
#define CLOCK_PIN A4
#define LATCH_PIN A5

const int ledPinovi[] = {2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13};
const int tipkaStop = A0;
const int tipkaReset = A1;

int trenutnaLed = 0;
int brojPokusaja = 0;
int ukupniBodovi = 0;
bool vrtnja = true;

int bodoviLed[] = {2, 1, 0, 3, 0, 1, 2, 1, 0, 3, 0, 1};

void setup() {
  pinMode(PODATAK_PIN, OUTPUT);
  pinMode(LATCH_PIN, OUTPUT);
  pinMode(CLOCK_PIN, OUTPUT);

  for (int i = 0; i < 12; i++) {
    pinMode(ledPinovi[i], OUTPUT);
    digitalWrite(ledPinovi[i], LOW);
  }

  pinMode(tipkaStop, INPUT_PULLUP);
  pinMode(tipkaReset, INPUT_PULLUP);

  obrisiZaslon();
}

void loop() {
  if (digitalRead(tipkaReset) == LOW) {
    resetirajIgru();
    delay(200);
  }

  if (vrtnja && brojPokusaja < 3) {
    digitalWrite(ledPinovi[trenutnaLed], HIGH);
    prikaziBroj(ukupniBodovi);
    delay(100);
    digitalWrite(ledPinovi[trenutnaLed], LOW);

    trenutnaLed = (trenutnaLed + 1) % 12;
  }

  if (digitalRead(tipkaStop) == LOW && vrtnja && brojPokusaja < 3) {
    vrtnja = false;
    digitalWrite(ledPinovi[trenutnaLed], HIGH);
    ukupniBodovi += bodoviLed[trenutnaLed];

    if (ukupniBodovi > 9) {
      ukupniBodovi = ukupniBodovi % 10;
    }

    prikaziBroj(ukupniBodovi);
    brojPokusaja++;
    delay(1000);

    if (brojPokusaja < 3) {
      digitalWrite(ledPinovi[trenutnaLed], LOW);
      vrtnja = true;
    }

    delay(200);
  }

  if (brojPokusaja >= 3) {
    prikaziBroj(ukupniBodovi);
  }
}

void prikaziBroj(int broj) {
  byte uzorak = 0b00000000;

  if (broj == 0) uzorak = 0b11111100;
  if (broj == 1) uzorak = 0b01100000;
  if (broj == 2) uzorak = 0b11011010;
  if (broj == 3) uzorak = 0b11110010;
  if (broj == 4) uzorak = 0b01100110;
  if (broj == 5) uzorak = 0b10110110;
  if (broj == 6) uzorak = 0b10111110;
  if (broj == 7) uzorak = 0b11100000;
  if (broj == 8) uzorak = 0b11111110;
  if (broj == 9) uzorak = 0b11110110;

  digitalWrite(LATCH_PIN, LOW);
  shiftOut(PODATAK_PIN, CLOCK_PIN, LSBFIRST, uzorak);
  digitalWrite(LATCH_PIN, HIGH);
}

void obrisiZaslon() {
  digitalWrite(LATCH_PIN, LOW);
  shiftOut(PODATAK_PIN, CLOCK_PIN, LSBFIRST, 0b00000000);
  digitalWrite(LATCH_PIN, HIGH);
}

void resetirajIgru() {
  for (int i = 0; i < 12; i++) {
    digitalWrite(ledPinovi[i], LOW);
  }

  brojPokusaja = 0;
  ukupniBodovi = 0;
  trenutnaLed = 0;
  vrtnja = true;

  obrisiZaslon();
}
