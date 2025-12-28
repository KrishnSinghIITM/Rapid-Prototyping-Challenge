# Handbook: Rapid Prototyping Session 1

**Project:** The Intelligent Toll Gate System  
**Duration:** 1 Hour  
**Hardware:** ESP32, Sensors, Actuators  



## 1. System Architecture (Wiring Map)
**Strictly follow this map. One wrong wire = Smoke.**

### ⚡ Power Distribution Rules
* **High Power (5V/VIN):** Servo Motor (Red Wire) & Ultrasonic Sensor (VCC).
* **Low Power (3.3V):** OLED Display & RGB LED.
* **Ground (GND):** All components must share a common Ground connection.

### 📌 Pin Connection Table

| Component | Pin Label | ESP32 Pin | Function / Use |
| :--- | :--- | :--- | :--- |
| **Master Switch** | Middle Pin | **GPIO 14** | Logic Input (System ON/OFF) |
| **Ultrasonic Sensor** | Trig | **GPIO 5** | Digital Output (Send Sound) |
| | Echo | **GPIO 18** | Digital Input (Receive Sound) |
| **Servo Motor** | Signal (Orange) | **GPIO 13** | PWM Output (Motor Control) |
| **OLED Display** | SDA | **GPIO 21** | I2C Data Line |
| | SCL | **GPIO 22** | I2C Clock Line |
| **RGB LED** | Blue Leg | **GPIO 4** | Status Indicator |
| | Green Leg | **GPIO 16** | Status Indicator |
| | Red Leg | **GPIO 17** | Status Indicator |

### Wokwi File
[View Project on Wokwi](https://wokwi.com/projects/451427565302923265)
<img width="984" height="553" alt="image" src="https://github.com/user-attachments/assets/bf160dd5-0b45-4a3e-a464-ef1f00617be2" />




## 2. System Logic (State Machine)
The system operates based on distance zones. Program your ESP32 to follow this logic table.

| State | Distance Range | RGB Color | OLED Status (Bold) | Gate Action |
| :--- | :--- | :--- | :--- | :--- |
| **BOOT** | Power ON | 🔵 Blue | (Loading Bar) | Closed |
| **IDLE** | > 200 cm | 🔵+🟢 Cyan | **ACTIVE** | Closed |
| **WARNING** | 100 - 200 cm | 🟡 Yellow | **SLOW DOWN** | Closed |
| **SCANNING** | < 100 cm | 🔴 Red | **SCANNING** | Wait 2 Seconds |
| **GRANTED** | (After Scan) | 🟢 Green | **WELCOME** | **OPEN (90°)** |
| **SAFETY** | Car still < 200cm | 🟢 Green | **CAR PASING** | **STAYS OPEN** |



## 3. AI Code Generation Prompt
You can use the following prompt to generate the firmware using AI tools:

> "Act as an Embedded Engineer. Write ESP32 C++ code for a Smart Toll Gate System.
>
> **Hardware Configuration:**
> * **Switch:** Pin 14 (Input Pull-up).
> * **Sensors:** Trig(5), Echo(18).
> * **Actuators:** Servo(13), RGB LED (Red 17, Green 16, Blue 4).
> * **Display:** SSD1306 OLED (I2C 0x3C).
>
> **Functional Requirements:**
> 1.  **Boot Up:** Show a loading bar animation on OLED and turn on Blue LED.
> 2.  **UI Layout:** Small Title at top, **Large BOLD Status** in the center.
> 3.  **Safety Logic:** The gate must remain open if an object is detected within 200cm (Anti-crush).
> 4.  **Zones:** Cyan Light (>200cm), Yellow Warning (100-200cm), Red Scan (<100cm).
> 5.  **Timing:** Simulate a 2-second scanning delay before opening the gate.
>
> **Output:** Provide the complete, error-free Arduino sketch."



## 4. Source Code (Session 1 Only)
*Flash this code to your ESP32 to run the system.*

```cpp
/*
 * WORKSHOP PROJECT: INTELLIGENT TOLL GATE SYSTEM
 * Platform: ESP32
 * Author: Krishn Singh
 */

#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>
#include <ESP32Servo.h>

// --- PIN DEFINITIONS ---
#define PIN_SWITCH  14  
#define PIN_TRIG    5
#define PIN_ECHO    18
#define PIN_SERVO   13
#define PIN_BLUE    4
#define PIN_GREEN   16
#define PIN_RED     17

// --- CONFIGURATION ---
#define SCREEN_WIDTH 128
#define SCREEN_HEIGHT 64
Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, -1);
Servo gateServo;

// --- VARIABLES ---
long duration;
int distance = 0;
bool systemBooted = false;
unsigned long scanStartTime = 0;
unsigned long gateOpenTime = 0;
bool isScanning = false;
bool isGateOpen = false;

// --- FUNCTION PROTOTYPES ---
int readDistance();
void setRGB(int r, int g, int b);
void updateScreen(String title, String status, int size); 
void centerText(String text, int y, int size);
void playBootAnimation();
void startScanning();
void openGate();
void closeGate();

void setup() {
  Serial.begin(115200);

  // Pin Configuration
  pinMode(PIN_SWITCH, INPUT_PULLUP);
  pinMode(PIN_TRIG, OUTPUT);
  pinMode(PIN_ECHO, INPUT);
  pinMode(PIN_RED, OUTPUT);
  pinMode(PIN_GREEN, OUTPUT);
  pinMode(PIN_BLUE, OUTPUT);

  // Initialize Actuators
  gateServo.attach(PIN_SERVO);
  gateServo.write(0); // Ensure Gate is Closed

  // Initialize Display
  if(!display.begin(SSD1306_SWITCHCAPVCC, 0x3C)) { 
    Serial.println(F("SSD1306 allocation failed"));
    for(;;); 
  }
  display.clearDisplay();
  display.display();
}

void loop() {
  // --- 1. POWER MANAGEMENT ---
  if (digitalRead(PIN_SWITCH) == HIGH) { 
    // System OFF State
    systemBooted = false;
    setRGB(0, 0, 0);
    display.clearDisplay();
    display.display();
    gateServo.write(0);
    return;
  }

  // --- 2. BOOT SEQUENCE ---
  if (!systemBooted) {
    playBootAnimation(); 
    systemBooted = true;
  }

  // --- 3. SENSOR DATA ACQUISITION ---
  distance = readDistance();

  // --- 4. CORE LOGIC ---
  
  // A. Gate Open State (Safety Monitor)
  if (isGateOpen) {
    if (millis() - gateOpenTime > 3000) {
      if (distance > 200) {
        closeGate(); // Safe to close
      } else {
        // Safety Triggered: Object obstructing gate
        // Note: 'PASING' used to fit 128px screen width
        updateScreen("WAITING...", "CAR PASING", 2); 
      }
    }
    return; 
  }

  // B. Scanning State
  if (isScanning) {
    if (millis() - scanStartTime > 2000) {
      openGate(); // Scan Complete
    } else {
      setRGB(1, 0, 0); // Red
      updateScreen("PLEASE WAIT", "SCANNING", 2); 
    }
    return;
  }

  // C. Monitoring State (Idle/Approach)
  if (distance < 100) {
    startScanning();
  } 
  else if (distance >= 100 && distance <= 200) {
    setRGB(1, 1, 0); // Yellow (Red + Green)
    updateScreen("WARNING", "SLOW DOWN", 2); 
  } 
  else {
    setRGB(0, 1, 1); // Cyan (Blue + Green)
    updateScreen("TOLL GATE", "ACTIVE", 2); 
  }
  
  delay(100); // System Stability Delay
}

// --- HELPER FUNCTIONS ---

int readDistance() {
  digitalWrite(PIN_TRIG, LOW);
  delayMicroseconds(2);
  digitalWrite(PIN_TRIG, HIGH);
  delayMicroseconds(10);
  digitalWrite(PIN_TRIG, LOW);
  duration = pulseIn(PIN_ECHO, HIGH);
  return duration * 0.034 / 2;
}

void setRGB(int r, int g, int b) {
  digitalWrite(PIN_RED, r);
  digitalWrite(PIN_GREEN, g);
  digitalWrite(PIN_BLUE, b);
}

void updateScreen(String title, String status, int size) {
  display.clearDisplay();
  display.setTextColor(WHITE);
  centerText(title, 5, 1);                  // Header
  display.drawLine(10, 20, 118, 20, WHITE); // Divider Line
  
  int yPos = (size == 1) ? 35 : 30;         // Dynamic Vertical Centering
  centerText(status, yPos, size);           // Status Text
  display.display();
}

void centerText(String text, int y, int size) {
  display.setTextSize(size);
  int charWidth = (size == 1) ? 6 : 12;
  int textWidth = text.length() * charWidth;
  int x = (SCREEN_WIDTH - textWidth) / 2;
  if (x < 0) x = 0;
  display.setCursor(x, y);
  display.print(text);
}

void playBootAnimation() {
  setRGB(0, 0, 1); // Blue Indicator
  display.clearDisplay();
  display.setTextColor(WHITE);
  centerText("SYSTEM", 5, 1);
  display.drawLine(10, 20, 118, 20, WHITE);
  centerText("LOADING...", 30, 1); 
  display.drawRect(14, 45, 100, 10, WHITE);
  display.display();
  
  // Animate Bar
  for(int i = 0; i <= 96; i+=4) {
    display.fillRect(16, 47, i, 6, WHITE);
    display.display();
    delay(30); 
  }
}

void startScanning() {
  if (!isScanning) {
    isScanning = true;
    scanStartTime = millis();
  }
}

void openGate() {
  isScanning = false;
  isGateOpen = true;
  gateOpenTime = millis();
  gateServo.write(90); 
  setRGB(0, 1, 0);     
  updateScreen("ACCESS GRANTED", "WELCOME", 2);
}

void closeGate() {
  isGateOpen = false;
  gateServo.write(0); 
}
```



## 5. FINAL CHALLENGE NOTE⚠️

**Important:** - For the **Final Challenge in Campus**, the full source code will **NOT** be provided. You will only receive **Boilerplate / Starter Code**. You will be required to write the core logic yourself.

**Allowed Resources:**
✅ You **ARE ALLOWED** to use any **AI Tool** (ChatGPT, Gemini, Claude, etc.) or the **Internet** to help you generate code, debug errors, and solve the problem.


---
**Credits ~ Krishn Singh**
