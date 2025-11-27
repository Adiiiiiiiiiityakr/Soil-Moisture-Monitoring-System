#include <Wire.h> 
#include <LiquidCrystal_I2C.h>

// --- Pin Definitions (ESP32-C6) ---
#define I2C_SDA      6   // Standard SDA for C6
#define I2C_SCL      7   // Standard SCL for C6
#define SOIL_PIN     2   // Analog Pin (ADC1_CH2)
#define BUZZER_PIN   4   // Digital Output

// --- LCD Setup ---
// Address 0x27 or 0x3F
LiquidCrystal_I2C lcd(0x27, 16, 2);

// --- Calibration Values (12-bit ADC: 0 - 4095) ---
// Since we are powering the sensor with 3.3V, the range is 0-4095.
// Dry = Higher Value (Resistance high)
// Wet = Lower Value (Resistance low)
const int DRY_THRESHOLD = 2800; 
const int WET_THRESHOLD = 1500; 

void setup() {
  // Initialize Serial for debugging
  Serial.begin(115200);
  
  // Initialize Pins
  pinMode(BUZZER_PIN, OUTPUT);
  // Note: INPUT mode is automatic for analogRead, but good practice
  pinMode(SOIL_PIN, INPUT); 

  // --- ADC Configuration ---
  // Ensure we are using 12-bit resolution (0-4095)
  analogReadResolution(12);
  // Set attenuation to 11dB to measure full 0-3.3V range
  analogSetAttenuation(ADC_11db);

  // --- I2C Initialization ---
  // Explicitly define pins for the Wire library
  Wire.begin(I2C_SDA, I2C_SCL);

  // Initialize LCD
  lcd.init();
  lcd.backlight();
  
  // Intro Screen
  lcd.setCursor(0, 0);
  lcd.print("Soil Moisture");
  lcd.setCursor(0, 1);
  lcd.print("Monitor"); // A nod to the architecture
  delay(2000);
  lcd.clear();
}

void loop() {
  // 1. Read Sensor
  int rawValue = analogRead(SOIL_PIN);
  
  Serial.printf("Raw ADC: %d\n", rawValue);

  // 2. Display Setup
  lcd.setCursor(0, 0);
  lcd.print("Moisture: ");
  // Optional: print raw value or percentage for debugging
  // lcd.print(rawValue); 

  lcd.setCursor(0, 1);

  // 3. Logic & Alerts
  if (rawValue > DRY_THRESHOLD) {
    // Condition: DRY
    lcd.print("Dry - WATER ME! ");
    
    // Buzzer Alert
    digitalWrite(BUZZER_PIN, HIGH);
    delay(100);
    digitalWrite(BUZZER_PIN, LOW);
  } 
  else if (rawValue < WET_THRESHOLD) {
    // Condition: HIGH MOISTURE
    lcd.print("High Moisture   ");
    digitalWrite(BUZZER_PIN, LOW);
  } 
  else {
    // Condition: OKAY
    lcd.print("Moderate / OK   ");
    digitalWrite(BUZZER_PIN, LOW);
  }

  delay(2000);
}