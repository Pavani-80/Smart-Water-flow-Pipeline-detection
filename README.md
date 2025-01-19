#define BLYNK_TEMPLATE_ID           "TMPL3IEZMgNT4"
#define BLYNK_TEMPLATE_NAME         "Quickstart Template"
#define BLYNK_AUTH_TOKEN            "dfrxYr8wS7vtgAuQ3DoCAPQoo-m-tpjC"
/* Comment this out to disable prints and save space */
#define BLYNK_PRINT Serial

#include <ESP8266WiFi.h>
#include <BlynkSimpleEsp8266.h>
#include <Wire.h>
#include <LiquidCrystal_I2C.h>
//#include <BlynkTimer.h>

// WiFi credentials
char ssid[] = "xplore";
char pass[] = "xplore@123";

// Define pins for sensors
#define FLOW_SENSOR_PIN 2   // Digital pin for water flow sensor
#define TRIG_PIN 14          // Trig pin of Ultrasonic sensor
#define ECHO_PIN 12        // Echo pin of Ultrasonic sensor

// Initialize LCD (I2C address: 0x27)
LiquidCrystal_I2C lcd(0x27, 20, 4);

// Variables for Water Flow Sensor
volatile int pulseCount = 0;
float flowRate = 0.0;
float totalLiters = 0.0;

// Ultrasonic Sensor Variables
float distance = 0.0;

// Blynk Timer
BlynkTimer timer;

// Interrupt service routine for flow sensor
void ICACHE_RAM_ATTR pulseCounter() {
  pulseCount++;
}

// Function to calculate water flow rate and total volume
void calculateFlow() {
  // Flow rate in liters/min (pulseCount * calibration factor)
  flowRate = pulseCount / 7.5; // Assuming 7.5 pulses per liter
  totalLiters += (flowRate / 60.0); // Convert flow rate to liters/sec
  pulseCount = 0;

  // Send data to Blynk and display on LCD
  Blynk.virtualWrite(V5, flowRate);    // Send flow rate to Blynk
  Blynk.virtualWrite(V6, totalLiters); // Send total liters to Blynk
  lcd.setCursor(0, 0);
  lcd.print("Flow: ");
  lcd.print(flowRate);
  lcd.print(" L/min");
  lcd.setCursor(0, 1);
  lcd.print("Total: ");
  lcd.print(totalLiters);
  lcd.print(" L");
  delay(1000);
  lcd.clear();
  digitalWrite(TRIG_PIN, LOW);
  delayMicroseconds(2);
  digitalWrite(TRIG_PIN, HIGH);
  delayMicroseconds(10);
  digitalWrite(TRIG_PIN, LOW);

  // Calculate duration and distance
  long duration = pulseIn(ECHO_PIN, HIGH);
  distance = (duration * 0.034) / 2; // Convert to cm

  // Send data to Blynk and display on LCD
  Blynk.virtualWrite(V7, distance); // Send distance to Blynk
  lcd.setCursor(0, 0);
  lcd.print("Distance: ");
  lcd.print(distance);
  lcd.print(" cm");
  delay(1000);
  lcd.clear();
}

void setup() {
  // Debug console
  Serial.begin(115200);

  // Initialize Blynk
  Blynk.begin(BLYNK_AUTH_TOKEN, ssid, pass);

  // Initialize LCD
  lcd.init();
  lcd.backlight();
  lcd.print("Initializing...");

  // Setup ultrasonic sensor
  pinMode(TRIG_PIN, OUTPUT);
  pinMode(ECHO_PIN, INPUT);

  // Setup timers
  timer.setInterval(1000L, calculateFlow);    // Update flow rate every second
  // Setup water flow sensor
  pinMode(FLOW_SENSOR_PIN, INPUT_PULLUP);
  attachInterrupt(digitalPinToInterrupt(FLOW_SENSOR_PIN), pulseCounter, CHANGE);

}

void loop() {
  Blynk.run();
  timer.run();
}
