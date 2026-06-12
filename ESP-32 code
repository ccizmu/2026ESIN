//藍牙EBS
#include "BluetoothSerial.h"

BluetoothSerial SerialBT;

bool ebsTriggered = false;
// =========================
// 感測器
// OUT5 → OUT1
// 左            右
// =========================
const int S5 = 13;
const int S4 = 14;
const int S3 = 27;
const int S2 = 26;
const int S1 = 25;

// =========================
// 馬達
// =========================
const int IN1 = 16;
const int IN2 = 17;

const int IN3 = 21;
const int IN4 = 22;

// =========================
// PWM
// ENA = 右輪
// ENB = 左輪
// =========================
const int ENA = 18;
const int ENB = 19;

// =========================
// 按鈕
// =========================
const int BTN = 15;

bool started = false;
bool lastButtonState = HIGH;

// =========================
// ASL LED
// =========================
const int LED_RED   = 32;
const int LED_GREEN = 33;
const int LED_BLUE  = 4;

// =========================
// 記錄方向
// =========================
int lastDirection = 0;

void setup()
{
  Serial.begin(115200);
  SerialBT.begin("EBS_CAR");

  pinMode(S1, INPUT);
  pinMode(S2, INPUT);
  pinMode(S3, INPUT);
  pinMode(S4, INPUT);
  pinMode(S5, INPUT);

  pinMode(IN1, OUTPUT);
  pinMode(IN2, OUTPUT);

  pinMode(IN3, OUTPUT);
  pinMode(IN4, OUTPUT);

  pinMode(BTN, INPUT_PULLUP);

  pinMode(LED_RED, OUTPUT);
  pinMode(LED_GREEN, OUTPUT);
  pinMode(LED_BLUE, OUTPUT);

  ledcAttach(ENA, 1000, 8);
  ledcAttach(ENB, 1000, 8);

  stopMotor();
  setASLSafe();

  // 開機即進入安全狀態
  started = false;
  ebsTriggered = false;
  checkEBS();
}

void loop()
{
  bool currentButton = digitalRead(BTN);
  checkEBS();
  if (SerialBT.available())
{
    char cmd = SerialBT.read();
//藍牙EBS
    if (cmd == 'S')
    {
        ebsTriggered = true;
    }

    if (cmd == 'R')
    {
        ebsTriggered = false;
    }
}
  if (ebsTriggered)
{
    stopMotor();

    started = false;

    setASLOther();   // 藍燈

    return;
}
//按鈕判斷
  if (lastButtonState == HIGH &&
      currentButton == LOW)
  {
    delay(30);

    if (digitalRead(BTN) == LOW)
    {
      started = !started;

      while (digitalRead(BTN) == LOW);

      delay(50);
    }
  }

  lastButtonState = currentButton;

 if (!started)
{
    stopMotor();

    setASLSafe();

    return;
}

// VLS 已觸發
setASLAutonomous();

  int s1 = digitalRead(S1);
  int s2 = digitalRead(S2);
  int s3 = digitalRead(S3);
  int s4 = digitalRead(S4);
  int s5 = digitalRead(S5);

 
// =========================
// 左90度
// =========================
if (s5 == 0)
{
    lastDirection = -1;
    spinLeft();
}

// =========================
// 右90度
// =========================
else if (s1 == 0)
{
    lastDirection = 1;
    spinRight();
}

// =========================
// 一般左轉
// =========================
else if (s4 == 0)
{
    lastDirection = -1;
    move(100, 140);
}

// =========================
// 一般右轉
// =========================
else if (s2 == 0)
{
    lastDirection = 1;
    move(140, 100);
}

// =========================
// 直走
// =========================
else if (s3 == 0)
{
    lastDirection = 0;
    move(160, 160);
}

else
{
    recoverLine();
}
}

// =========================
// 前進
// =========================

void move(int leftSpeed, int rightSpeed)
{
  digitalWrite(IN1, LOW);
  digitalWrite(IN2, HIGH);

  digitalWrite(IN3, LOW);
  digitalWrite(IN4, HIGH);

  ledcWrite(ENA, rightSpeed);
  ledcWrite(ENB, leftSpeed);
}

// =========================
// 原地左轉
// =========================

void spinLeft()
{
  // 左輪前進
  digitalWrite(IN1, LOW);
  digitalWrite(IN2, HIGH);

  // 右輪倒退
  digitalWrite(IN3, HIGH);
  digitalWrite(IN4, LOW);

  ledcWrite(ENA, 120);
  ledcWrite(ENB, 120);
}

// =========================
// 原地右轉
// =========================

void spinRight()
{
  // 左輪倒退
  digitalWrite(IN1, HIGH);
  digitalWrite(IN2, LOW);

  // 右輪前進
  digitalWrite(IN3, LOW);
  digitalWrite(IN4, HIGH);

  ledcWrite(ENA, 120);
  ledcWrite(ENB, 120);
}

// =========================
// 掉線搜尋
// =========================

void recoverLine()
{
  if (lastDirection == -1)
  {
    spinLeft();
  }
  else if (lastDirection == 1)
  {
    spinRight();
  }
  else
  {
    stopMotor();
  }
}

// =========================
// 停止
// =========================

void stopMotor()
{
  digitalWrite(IN1, LOW);
  digitalWrite(IN2, LOW);

  digitalWrite(IN3, LOW);
  digitalWrite(IN4, LOW);

  ledcWrite(ENA, 0);
  ledcWrite(ENB, 0);
}
// =========================
// ASL 狀態控制
// =========================

// 綠燈：安全狀態
void setASLSafe()
{
  digitalWrite(LED_RED, LOW);
  digitalWrite(LED_GREEN, HIGH);
  digitalWrite(LED_BLUE, LOW);
}

// 紅燈：自主導航
void setASLAutonomous()
{
  digitalWrite(LED_RED, HIGH);
  digitalWrite(LED_GREEN, LOW);
  digitalWrite(LED_BLUE, LOW);
}

// 藍燈：其他狀態
void setASLOther()
{
  digitalWrite(LED_RED, LOW);
  digitalWrite(LED_GREEN, LOW);
  digitalWrite(LED_BLUE, HIGH);
}
// =========================
// EBS 藍牙監聽
// =========================
void checkEBS()
{
    while (SerialBT.available())
    {
        char cmd = SerialBT.read();

        if (cmd == 'S')
        {
            ebsTriggered = true;

            stopMotor();

            started = false;

            setASLOther();
        }

        else if (cmd == 'R')
        {
            ebsTriggered = false;

            stopMotor();

            started = false;

            setASLSafe();
        }
    }
}
