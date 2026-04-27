This code for 2time/day for 30days⁉️

>> the VDO about project: https://drive.google.com/file/d/1T9LdPjL_ag954K7VdzNCs7qaCLSzKXxP/view?usp=drivesdk

#include <Keypad.h>


// ================= KEYPAD =================
const byte ROWS = 4;
const byte COLS = 4;


char keys[ROWS][COLS] = {
  {'1','2','3','A'},
  {'4','5','6','B'},
  {'7','8','9','C'},
  {'*','0','#','D'}
};


byte rowPins[ROWS] = {9, 8, 7, 6};
byte colPins[COLS] = {5, 4, 3, 2};


Keypad keypad = Keypad(makeKeymap(keys), rowPins, colPins, ROWS, COLS);


// ================= PIN =================
const int trigPin = 10;
const int echoPin = 11;


const int buzzer = A0;
const int mornLed = 0;
const int eveLed  = 1;


const int segPins[] = {A1, A2, A3, A4, A5, 12, 13};


// ================= 7 SEG =================
byte digits[13][7] = {
  {1,1,1,1,1,1,0},{0,1,1,0,0,0,0},{1,1,0,1,1,0,1},
  {1,1,1,1,0,0,1},{0,1,1,0,0,1,1},{1,0,1,1,0,1,1},
  {1,0,1,1,1,1,1},{1,1,1,0,0,0,0},{1,1,1,1,1,1,1},
  {1,1,1,1,0,1,1},{1,1,1,0,1,1,1},{0,0,1,1,1,1,1},
  {1,0,0,1,1,1,1}
};


// ================= TIME =================
const unsigned long TIME_UNIT = 1000UL;
unsigned long intervalDay = 10 * TIME_UNIT;
unsigned long intervalNight = 14 * TIME_UNIT;


unsigned long targetTime = 0;
unsigned long prevMillis = 0;


// ================= BLINK =================
unsigned long ledPrevMillis = 0;
bool ledState = false;
const unsigned long BLINK_INTERVAL = 400;


// ================= STATE =================
String inputBuffer = "";
char currentMode = ' ';
int doseStep = 0;
bool isAlarm = false;
bool startFromEvening = false;


// ================= SETUP =================
void setup() {
  for (int i = 0; i < 7; i++) pinMode(segPins[i], OUTPUT);


  pinMode(trigPin, OUTPUT);
  pinMode(echoPin, INPUT);


  pinMode(buzzer, OUTPUT);
  digitalWrite(buzzer, HIGH);


  pinMode(mornLed, OUTPUT);
  pinMode(eveLed, OUTPUT);


  digitalWrite(mornLed, HIGH);
  digitalWrite(eveLed, HIGH);
  showDigit(8);
  delay(1000);


  digitalWrite(mornLed, LOW);
  digitalWrite(eveLed, LOW);
  showDigit(0);
}


// ================= DISPLAY =================
void showDigit(int index) {
  for (int i = 0; i < 7; i++) {
    digitalWrite(segPins[i], digits[index][i]);
  }
}


// ================= LOOP =================
void loop() {
  char key = keypad.getKey();
  if (key != NO_KEY) handleKey(key);


  if (doseStep >= 1 && !isAlarm) {
    if (millis() - prevMillis >= targetTime) {
      isAlarm = true;
    }
  }


  if (isAlarm) {
    digitalWrite(buzzer, LOW);
    showDigit(doseStep % 10);


    // ===== BLINK =====
    if (millis() - ledPrevMillis >= BLINK_INTERVAL) {
      ledPrevMillis = millis();
      ledState = !ledState;
    }


    bool isMorning;
    if (!startFromEvening)
      isMorning = (doseStep % 2 != 0);
    else
      isMorning = (doseStep % 2 == 0);


    if (isMorning) {
      digitalWrite(mornLed, ledState ? HIGH : LOW);
      digitalWrite(eveLed, LOW);
    } else {
      digitalWrite(mornLed, LOW);
      digitalWrite(eveLed, ledState ? HIGH : LOW);
    }


    long d = getDistance();


    if (d > 0 && d < 15) {
      delay(300);
      if (getDistance() < 15) {
        confirmDose();
      }
    }


  } else {
    digitalWrite(buzzer, HIGH);
  }
}


// ================= KEYPAD =================
void handleKey(char key) {


  if (key == '*') {
    inputBuffer = "";
    showDigit(0);
  }


  else if (key == 'A') {


    if (inputBuffer.length() > 0 && currentMode == ' ') {
      long val = inputBuffer.toInt();


      if (val > 0) {
        targetTime = val * TIME_UNIT;
        doseStep = 1;
        startFromEvening = false;
        prevMillis = millis();
        showDigit(1);
      }


      inputBuffer = "";
    }
    else {
      currentMode = 'A';
      inputBuffer = "";
      showDigit(10);
    }
  }


  else if (key == 'B') {


    if (inputBuffer.length() > 0 && currentMode == ' ') {
      long val = inputBuffer.toInt();


      if (val > 0) {
        targetTime = val * TIME_UNIT;
        doseStep = 1;
        startFromEvening = true;
        prevMillis = millis();
        showDigit(1);
      }


      inputBuffer = "";
    }
    else {
      currentMode = 'B';
      inputBuffer = "";
      showDigit(11);
    }
  }


  else if (key >= '0' && key <= '9') {
    inputBuffer += key;
    showDigit(key - '0');
  }


  else if (key == '#') {
    long val = inputBuffer.toInt();


    if (currentMode == 'A') {
      intervalDay = val * TIME_UNIT;
    }
    else if (currentMode == 'B') {
      intervalNight = val * TIME_UNIT;
    }


    inputBuffer = "";
    currentMode = ' ';
  }


  else if (key == 'D') {
    if (isAlarm) confirmDose();
  }


  else if (key == 'C') {
    resetSystem();
  }
}


// ================= RESET =================
void resetSystem() {
  doseStep = 0;
  isAlarm = false;
  startFromEvening = false;


  digitalWrite(buzzer, HIGH);
  digitalWrite(mornLed, LOW);
  digitalWrite(eveLed, LOW);


  inputBuffer = "";
  currentMode = ' ';
  showDigit(0);
}


// ================= CONFIRM =================
void confirmDose() {
  isAlarm = false;
  digitalWrite(buzzer, HIGH);


  digitalWrite(mornLed, LOW);
  digitalWrite(eveLed, LOW);


  bool isMorning;


  if (!startFromEvening)
    isMorning = (doseStep % 2 != 0);
  else
    isMorning = (doseStep % 2 == 0);


  if (isMorning) targetTime = intervalNight;
  else targetTime = intervalDay;


  prevMillis = millis();
  doseStep++;


  if (doseStep > 60) {
    doseStep = 0;
    showDigit(12);
  } else {
    showDigit(doseStep % 10);
  }
}


// ================= SENSOR =================
long getDistance() {
  digitalWrite(trigPin, LOW);
  delayMicroseconds(2);
  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin, LOW);


  long duration = pulseIn(echoPin, HIGH, 30000);
  if (duration == 0) return 999;


  return duration / 58;
}

this code for 3 times/days⁉️

The web>> https://wokwi.com/projects/461167741978820609
📘 คู่มือการใช้งานเครื่องเตือนกินยา (เวอร์ชันใช้จริง)
🎯 แนวคิดของเครื่อง
เครื่องนี้ออกแบบมาให้:
* คนตั้งค่า (ลูก/ผู้ดูแล) ตั้งเวลา
* ผู้ใช้งานจริง (เช่น ผู้สูงอายุ) ไม่ต้องตั้งอะไรเลย
* แค่ “รอเครื่องเตือน แล้วหยิบยา”
👉 ใช้ง่ายที่สุด = ลดความผิดพลาด



🔘 อุปกรณ์บนเครื่อง
1. ปุ่มกด (Keypad)
ใช้สำหรับ “ตั้งค่า” เท่านั้น
ปุ่ม	หน้าที่
0–9	ใส่ตัวเลข
A	ตั้งเวลา “มื้อเช้า”
B	ตั้งเวลา “มื้อกลางวัน”
C	ตั้งเวลา “มื้อเย็น”
#	ยืนยัน
*	ล้างค่า
D	ยืนยันตอน alarm


2. 7-Segment (ตัวเลขเดียว)
แสดงผลแบบง่าย:
ตัวเลข	ความหมาย
0	รอเวลา
1	มื้อเช้า
2	มื้อกลางวัน
3	มื้อเย็น
E	ครบกำหนด (จบรอบ)


3. ไฟ LED
สี/ตำแหน่ง	ความหมาย
LED เช้า	มื้อเช้า
LED กลางวัน	มื้อกลางวัน
LED เย็น	มื้อเย็น


4. เสียง (Buzzer)
เสียง	ความหมาย
สูง	เช้า
กลาง	กลางวัน
ต่ำ	เย็น


5. เซ็นเซอร์ (Ultrasonic)
👉 ใช้ตรวจว่า “มีการหยิบยาแล้ว”



⚙️ วิธีตั้งค่า (สำหรับผู้ดูแล)
🟡 ตั้งเวลาแต่ละมื้อ
ตัวอย่าง: ตั้งมื้อเช้า = 6 ชั่วโมง
1. กด A
2. กด 6
3. กด #
👉 เสร็จ



ตั้งครบ 3 มื้อ:
* A → เช้า
* B → กลางวัน
* C → เย็น



▶️ วิธีเริ่มระบบ
เริ่มที่ “มื้อเช้า”
1. กดตัวเลข (เช่น 5)
2. กด #
👉 ระบบเริ่มนับเวลา



เริ่มที่ “มื้อกลางวัน”
1. กดตัวเลข
2. กด B



เริ่มที่ “มื้อเย็น”
1. กดตัวเลข
2. กด D



🚨 ตอนเครื่องเตือน
เมื่อถึงเวลา:
👉 จะเกิด:
* มีเสียง 🔊
* ไฟ LED ติด 💡
* ตัวเลขขึ้น (1 / 2 / 3)



👵 วิธีใช้สำหรับผู้สูงอายุ
แค่:
1. เดินมาที่เครื่อง
2. หยิบยา
👉 เครื่องจะ หยุดเองอัตโนมัติ
(เพราะมี sensor ตรวจจับ)



🔁 การทำงานต่อเนื่อง
* เช้า → กลางวัน → เย็น
* วนไปเรื่อย ๆ
* ครบ 30 วัน → แสดง E



💡 ทำไมใช้ 7-segment ตัวเดียว?
🧠 คำตอบเอาไว้ตอบกรรมการ
✅ เหตุผลด้านการออกแบบ
* ลดความซับซ้อน → ผู้สูงอายุใช้ได้
* ไม่ต้องอ่านตัวหนังสือ
* ใช้ “ตัวเลข + สี + เสียง” แทน



✅ เหตุผลด้านงบประมาณ
👉 พูดได้เลย:
“โปรเจกต์นี้ออกแบบภายใต้งบประมาณจำกัด เพื่อให้สามารถนำไปใช้งานจริงในครัวเรือนได้”



✅ เหตุผลด้านความทนทาน
* 7-seg ทนกว่า LCD
* ใช้ไฟน้อย
* ไม่พังง่าย



✅ UX (สำคัญมาก)
“ผู้ตั้งค่าคือคนที่เข้าใจระบบ แต่ผู้ใช้งานจริงคือผู้สูงอายุ จึงต้องออกแบบให้ใช้งานง่ายที่สุด”

