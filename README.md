# Blind-Curve-detection-project-
This repository includes code which powers the blind curve detection and alert project #include <WiFi.h>
#include <WiFiClientSecure.h>
#include <HTTPClient.h>
#include <Wire.h>
#include <LiquidCrystal_I2C.h>

// --- WiFi ---
const char* ssid = "M35";
const char* password = "ansley1234";

String botToken = "8172178568:AAF9P8OuWXXsdEtD9LRPcbcbujUT8ExQMVQ";
String chatID = "7618044021";

// --- Pins ---
#define PIR_PIN 12
#define RED_LED 4
#define GREEN_LED 14
#define BUZZER 4   // shared with red LED

// --- LCD ---
LiquidCrystal_I2C lcd(0x27, 16, 2);

// --- State variables ---
bool lastState = LOW;
bool telegramSent = false;

// ---------------------

void sendTelegram(String message) {
  WiFiClientSecure client;
  client.setInsecure();
  HTTPClient https;

  String url = "https://api.telegram.org/bot" + botToken +
               "/sendMessage?chat_id=" + chatID +
               "&text=" + message;

  if (https.begin(client, url)) {
    int httpCode = https.GET();
    if (httpCode > 0) {
      Serial.println("Telegram sent");
    }
    https.end();
  }
}

// 🔥 SAME BLINK STYLE FUNCTION
void blinkAlert() {
  digitalWrite(RED_LED, HIGH);
  digitalWrite(BUZZER, HIGH);
  delay(200);
  digitalWrite(RED_LED, LOW);
  digitalWrite(BUZZER, LOW);
  delay(200);
}

void setup() {
  Serial.begin(115200);

  pinMode(PIR_PIN, INPUT);
  pinMode(RED_LED, OUTPUT);
  pinMode(GREEN_LED, OUTPUT);

  digitalWrite(GREEN_LED, LOW);
  digitalWrite(RED_LED, LOW);

  // LCD
  lcd.init();
  lcd.backlight();
  lcd.setCursor(0, 0);
  lcd.print("Connecting WiFi");

  // WiFi connect
  WiFi.begin(ssid, password);
  Serial.print("Connecting");

  int timeout = 0;
  while (WiFi.status() != WL_CONNECTED && timeout < 20) {
    delay(500);
    Serial.print(".");
    timeout++;
  }

  lcd.clear();

  if (WiFi.status() == WL_CONNECTED) {
    Serial.println("\nConnected");
    lcd.print("System Ready");
    sendTelegram("Vehicle Detection System Online 📡");
  } else {
    Serial.println("\nWiFi Failed");
    lcd.print("Offline Mode");
  }

  delay(2000);
  lcd.clear();

  // PIR stabilization
  delay(15000);
}

void loop() {

  int motion = digitalRead(PIR_PIN);

  if (motion == HIGH) {

    // Green OFF
    digitalWrite(GREEN_LED, LOW);

    // LCD update once
    if (lastState == LOW) {
      lcd.clear();
      lcd.setCursor(0, 0);
      lcd.print("Vehicle");
      lcd.setCursor(0, 1);
      lcd.print("Approaching!");

      if (!telegramSent && WiFi.status() == WL_CONNECTED) {
        sendTelegram("⚠️ ALERT: Vehicle Approaching!");
        telegramSent = true;
      }

      lastState = HIGH;
    }

    // 🔴 Blink (same as old code)
    blinkAlert();

  } else {

    // Green ON
    digitalWrite(GREEN_LED, HIGH);
    digitalWrite(RED_LED, LOW);
    digitalWrite(BUZZER, LOW);

    // LCD update once
    if (lastState == HIGH) {
      lcd.clear();
      lcd.setCursor(0, 0);
      lcd.print("No Vehicle");

      telegramSent = false;
      lastState = LOW;
    }

    delay(300);
  }
}
<img width="1326" height="593" alt="2EF592E3-3AAC-494F-BEB0-083D56D1037A_1_105_c" src="https://github.com/user-attachments/assets/cb2c7a95-09c5-430f-b96f-d677b868187d" />
# Solar-Powered Smart Blind Curve Warning System Using ESP32

## 1. Project Overview

The Solar-Powered Smart Blind Curve Warning System is an embedded-system-based safety solution designed to reduce the risk of accidents at blind curves, particularly in hilly regions and roads with limited visibility.

At sharp bends, drivers may not be able to identify vehicles or movement approaching from the opposite direction. This can lead to collisions, especially on narrow roads and in areas with poor visibility.

This project proposes a low-cost warning system using an ESP32 microcontroller and a PIR sensor to detect movement and activate visual and audio alerts. An LCD displays the warning status, while a Telegram bot provides remote notifications through the internet.

The system also incorporates a solar-powered battery arrangement using a 5V solar panel, charging module, rechargeable battery, and boost converter to demonstrate the possibility of operating the system using renewable energy.

---

## 2. Objectives

- Develop a low-cost warning system for blind curves.
- Detect movement using a PIR sensor.
- Provide immediate visual and audio warnings.
- Display system status through an LCD.
- Integrate Telegram notifications for remote monitoring.
- Demonstrate solar-based power generation and battery storage.
- Explore the application of IoT and embedded systems in road safety.

---

## 3. Components Used

| Component | Purpose |
|---|---|
| ESP32 Development Board | Main microcontroller and system controller |
| PIR Sensor | Detects movement in the monitored area |
| 5V Solar Panel | Converts sunlight into electrical energy |
| TP4056 Charging Module | Manages rechargeable battery charging |
| Rechargeable Battery (18650) | Stores electrical energy |
| MT3608 Boost Converter | Boosts battery voltage to the required operating level |
| 16x2 I2C LCD | Displays warning and system status messages |
| Red LED | Indicates a warning condition |
| Green LED | Indicates normal operating conditions |
| Buzzer | Produces an audible warning |
| Connecting Wires | Electrical connections between components |

---

## 4. System Architecture

The system consists of three major sections.

### A. Detection and Processing

The PIR sensor continuously monitors the designated area for movement. When movement is detected, its output signal is sent to the ESP32.

The ESP32 processes the sensor input and determines the appropriate warning condition.

### B. Alert and Notification System

Based on the sensor input, the ESP32 controls the following outputs:

**Normal Condition:**
- Green LED remains ON.
- Red LED and buzzer remain OFF.
- LCD displays "No Vehicle".

**Detection Condition:**
- Green LED turns OFF.
- Red LED starts blinking.
- Buzzer produces an audible alert.
- LCD displays "Vehicle Approaching!".
- A Telegram notification is sent to the configured Telegram chat.

### C. Solar Power Arrangement

The project incorporates a 5V solar panel to demonstrate renewable energy-based operation.

The power arrangement consists of:

1. The solar panel generates electrical energy from sunlight.
2. The TP4056 charging module manages battery charging.
3. The rechargeable battery stores the generated energy.
4. The MT3608 boost converter increases the battery voltage to the required operating level.
5. The regulated output supplies power to the ESP32 system.

This arrangement demonstrates how solar energy can be incorporated into warning systems intended for remote locations.

---

## 5. Telegram Bot Integration

One of the important features of this project is its integration with the Telegram messaging platform.

A Telegram bot was created using BotFather, which provides the bot authentication token required for communication with the Telegram Bot API.

The ESP32 connects to a Wi-Fi network and uses the authentication token to communicate with the Telegram API.

### Working Procedure

1. A Telegram bot is created using BotFather.
2. The bot authentication token is obtained.
3. The Telegram chat ID is configured in the ESP32 program.
4. The ESP32 connects to the internet through Wi-Fi.
5. When the PIR sensor detects movement, the ESP32 sends an HTTP request to the Telegram Bot API.
6. The bot delivers a notification to the configured Telegram chat.

This feature enables remote monitoring without requiring a dedicated mobile application.

**Security Note:** The bot token and Wi-Fi credentials must be stored securely. Never upload actual authentication tokens or passwords to a public GitHub repository. Use placeholders or a separate configuration file excluded through `.gitignore`.

---

## 6. Working Principle

The system starts by initializing the ESP32, PIR sensor, LCD, LEDs, buzzer, and Wi-Fi connection.

During normal operation, the PIR sensor monitors the surrounding area. The green LED indicates that no movement has been detected.

When movement is detected, the PIR sensor sends a HIGH signal to the ESP32. The microcontroller responds by:

- Switching OFF the green LED.
- Activating the red warning LED.
- Activating the buzzer.
- Displaying a warning message on the LCD.
- Sending a Telegram notification through the configured bot.

The warning indicators remain active while the detection condition is present. When movement is no longer detected, the system returns to its normal state.

The Telegram notification mechanism is configured to avoid sending repeated messages continuously during a single detection event.

---

## 7. Applications and Usefulness

The proposed system demonstrates how embedded electronics and IoT communication can be used to improve safety in areas where visibility is limited.

Potential application areas include:

- **Hilly Roads:** Warning systems at sharp bends and blind curves.
- **Narrow Roads:** Alerting road users about movement near restricted-visibility sections.
- **Remote Locations:** Solar-based operation can reduce dependence on conventional electrical infrastructure.
- **Private Roads:** Monitoring movement around restricted or poorly visible sections.
- **Industrial Areas:** Providing warning indications near blind corners in factories and warehouses.
- **Campus and Parking Areas:** Alerting people about movement near obstructed pathways.

The project is particularly useful as a prototype for exploring low-cost, distributed warning systems.

---

## 8. Implementation Challenges

Several practical challenges were encountered during the development and integration of the system.

### 8.1 Sensor Integration

Interfacing the PIR sensor with the ESP32 required proper GPIO configuration and testing. The sensor also required an initial stabilization period before providing reliable readings.

### 8.2 LCD Interfacing

Configuring the I2C LCD involved identifying the correct I2C address and installing a compatible library. Initial display issues required checking the connections and software configuration.

### 8.3 Telegram Bot Configuration

Setting up Telegram communication required creating a bot, obtaining the authentication token, identifying the chat ID, and configuring Wi-Fi connectivity.

Handling network connectivity and HTTP requests was an important part of the implementation.

### 8.4 Hardware Integration

Integrating multiple components with the ESP32 required careful GPIO assignment and proper power distribution. Testing individual components before integrating the complete system helped identify wiring and software-related issues.

### 8.5 Solar Power Integration

Integrating the solar panel with the battery and boost converter required understanding voltage conversion and battery charging.

The solar panel output varies depending on sunlight intensity. Therefore, stable power delivery and proper voltage regulation are important considerations for continuous operation.

---

## 9. Limitations of the Current Prototype

Although the prototype demonstrates the proposed concept, certain limitations must be considered before real-world deployment.

- A PIR sensor detects changes in infrared radiation caused by movement. It does not directly identify or classify vehicles.
- The prototype demonstrates movement detection rather than reliable vehicle-to-vehicle detection.
- Wi-Fi availability is required for Telegram notifications.
- Solar energy generation depends on sunlight intensity and weather conditions.
- Battery capacity and power-management efficiency influence operating duration.
- The current prototype is intended for demonstration and educational purposes, not as a certified road-safety installation.

---

## 10. Areas for Improvement

The following improvements could increase the reliability and practical usefulness of the system.

### Improved Detection Technology

Replace or supplement the PIR sensor with radar, LiDAR, ultrasonic, or suitable vehicle-detection sensors to improve vehicle detection accuracy.

### Better Power Management

Use a dedicated solar charging and power-management circuit with appropriate battery protection and voltage regulation.

### Improved Communication

Introduce LoRa, GSM, or other long-range communication technologies to support locations without conventional Wi-Fi connectivity.

### Weatherproof Enclosure

Design a weather-resistant enclosure to protect the electronics against rain, dust, heat, and other environmental conditions.

### Improved Alert Mechanism

Use high-visibility warning lights and appropriately rated outdoor audio alert devices.

### Intelligent Detection

Introduce filtering and decision-making algorithms to reduce false detections and improve system response.

---

## 11. Future Scalability

The project can be expanded into a distributed smart-road safety network.

Possible future developments include:

1. **Multiple Sensor Nodes:** Install detection units on both sides of a blind curve to identify movement from different directions.

2. **Long-Range Wireless Communication:** Implement LoRa or GSM-based communication to connect multiple warning units over longer distances.

3. **Vehicle Detection and Classification:** Integrate radar or other suitable sensors to distinguish vehicles from pedestrians and other movement.

4. **Solar Power Optimization:** Upgrade the power system with higher-capacity solar panels, suitable battery storage, and intelligent power management.

5. **Cloud-Based Monitoring:** Store detection events and system status on a cloud platform for remote monitoring and maintenance.

6. **GPS Integration:** Include location information in notifications to identify the exact warning-system location.

7. **Advanced Embedded Processing:** Explore edge processing and intelligent algorithms for improved detection and reduced false alerts.

8. **Multi-Node Warning Coordination:** Allow multiple installations to communicate with each other and provide early warnings to approaching road users.

With these improvements, the prototype could serve as a starting point for developing a more reliable and scalable intelligent road-warning system.

---

## 12. Technologies Used

- Embedded C
- Arduino IDE
- ESP32 Microcontroller
- PIR Sensor Interfacing
- I2C Communication
- LCD Interfacing
- Wi-Fi Communication
- Telegram Bot API
- HTTP Requests
- Solar Energy and Battery Storage
- DC-DC Boost Conversion

---

## 13. Conclusion

The Solar-Powered Smart Blind Curve Warning System demonstrates the integration of embedded systems, sensor technology, wireless communication, and renewable energy.

By combining a PIR sensor with an ESP32 microcontroller, the prototype can detect movement and activate visual and audio warnings. The LCD provides local status information, while Telegram integration enables remote notifications. The solar panel and battery arrangement demonstrates the potential for renewable-energy-based operation in locations with limited electrical infrastructure.

Although further improvements in vehicle detection, communication range, power management, and environmental protection are necessary, the project provides a practical foundation for exploring intelligent and low-cost road-safety solutions.

---

## 14. Repository Structure

```text
Smart-Blind-Curve-Warning-System/
│
├── README.md
│
├── Source_Code/
│   └── blind_curve_warning.ino
│
├── Circuit_Diagram/
│   └── circuit_diagram.png
│
├── Block_Diagram/
│   └── block_diagram.png
│
├── Images/
│   ├── prototype.jpg
│   ├── hardware_setup.jpg
│   └── solar_power_setup.jpg
│
└── LICENSE
