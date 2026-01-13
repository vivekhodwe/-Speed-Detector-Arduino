# -Speed-Detector-Arduino
Arduino-based speed measurement using ultrasonic sensors
# Speed Detector Using Arduino (Ultrasonic Sensor + 7-Segment Display)

This project measures the speed of a moving object using two ultrasonic sensors and an Arduino Uno. 
When an object crosses the first sensor, the system starts a timer, and when it passes the second sensor,
the timer stops. Using the fixed distance between the sensors, the Arduino calculates the speed and displays it
on a 7-segment display.

## Features
- Detects object motion using HC-SR04 ultrasonic sensors
- Calculates speed using Speed = Distance / Time
- Displays measured speed on 7-segment display
- Accurate timing using Arduino micros()

## Components Used
- Arduino Uno
- 2 × HC-SR04 Ultrasonic Sensors
- 7-Segment Display
- Breadboard / PCB

## Working Principle
1. First ultrasonic sensor detects the object and triggers a timer.
2. Second sensor detects the object and stops the timer.
3. Speed = Distance / Time formula is applied.
4. Speed appears on a 7-segment display.

## Circuit Diagram
![image](https://github.com/user-attachments/assets/024623c3-40cc-4f1b-bf3d-58c22b8ada84)


## Code
// SPEED DETECTION PROJECT
// Arduino UNO + Two HC-SR04 Ultrasonic Sensors + 7-Segment Display
// ------------------------------------------------------------
//
// HOW IT WORKS (simple explanation):
// 1. The first ultrasonic sensor sees the object and starts the timer.
// 2. The second sensor sees the same object and stops the timer.
// 3. Speed = Distance between sensors / Time taken.
//
// The calculated speed (0–9 cm/s range for single digit) is then shown
// on a common-cathode 7-segment display.
//
// ------------------------------------------------------------


// ------------------- PIN SETUP -------------------
#define TRIG1 2
#define ECHO1 3
#define TRIG2 4
#define ECHO2 5

// 7-segment pins (A to G)
int segA = 6;
int segB = 7;
int segC = 8;
int segD = 9;
int segE = 10;
int segF = 11;
int segG = 12;


// ------------------- CONSTANTS -------------------
float sensorGap = 20.0;        // distance between two sensors in cm
int detectLimit = 15;          // object detection threshold


// ------------------- SETUP -------------------
void setup() {
  Serial.begin(9600);

  // Ultrasonic sensor pins
  pinMode(TRIG1, OUTPUT);
  pinMode(ECHO1, INPUT);
  pinMode(TRIG2, OUTPUT);
  pinMode(ECHO2, INPUT);

  // 7 segment pins
  pinMode(segA, OUTPUT);
  pinMode(segB, OUTPUT);
  pinMode(segC, OUTPUT);
  pinMode(segD, OUTPUT);
  pinMode(segE, OUTPUT);
  pinMode(segF, OUTPUT);
  pinMode(segG, OUTPUT);

  Serial.println("System Ready. Waiting for object...");
}


// ------------------- FUNCTION: Measure Distance -------------------
long getDistance(int trigPin, int echoPin) {

  // trigger pulse
  digitalWrite(trigPin, LOW);
  delayMicroseconds(3);

  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin, LOW);

  // time of echo
  long duration = pulseIn(echoPin, HIGH);

  // convert time → distance
  long distance = duration * 0.034 / 2;

  return distance;
}


// ------------------- FUNCTION: Display digit on 7-seg -------------------
void showDigit(int digit) {

  // segment patterns for 0–9
  int numbers[10][7] = {
    {1,1,1,1,1,1,0}, // 0
    {0,1,1,0,0,0,0}, // 1
    {1,1,0,1,1,0,1}, // 2
    {1,1,1,1,0,0,1}, // 3
    {0,1,1,0,0,1,1}, // 4
    {1,0,1,1,0,1,1}, // 5
    {1,0,1,1,1,1,1}, // 6
    {1,1,1,0,0,0,0}, // 7
    {1,1,1,1,1,1,1}, // 8
    {1,1,1,1,0,1,1}  // 9
  };

  digitalWrite(segA, numbers[digit][0]);
  digitalWrite(segB, numbers[digit][1]);
  digitalWrite(segC, numbers[digit][2]);
  digitalWrite(segD, numbers[digit][3]);
  digitalWrite(segE, numbers[digit][4]);
  digitalWrite(segF, numbers[digit][5]);
  digitalWrite(segG, numbers[digit][6]);
}


// ------------------- MAIN LOOP -------------------
void loop() {

  long dist1 = getDistance(TRIG1, ECHO1);

  // If an object comes near sensor 1, start timer
  if (dist1 < detectLimit) {

    Serial.println("Object detected at Sensor 1… Timing started.");
    unsigned long startTime = millis();

    // Wait until sensor 2 detects same object
    long dist2 = 100;
    while (dist2 > detectLimit) {
      dist2 = getDistance(TRIG2, ECHO2);
    }

    unsigned long endTime = millis();
    unsigned long timeTaken = endTime - startTime;

    // Just in case time = 0 ms
    if (timeTaken == 0) timeTaken = 1;

    // Calculate speed in cm/s
    float seconds = timeTaken / 1000.0;
    float speed = sensorGap / seconds;

    Serial.println("-----------------------------");
    Serial.print("Distance between sensors: ");
    Serial.print(sensorGap);
    Serial.println(" cm");

    Serial.print("Time taken: ");
    Serial.print(seconds, 3);
    Serial.println(" s");

    Serial.print("Calculated Speed: ");
    Serial.print(speed, 2);
    Serial.println(" cm/s");
    Serial.println("-----------------------------");

    // Limit displayed speed to 0–9
    int displayValue = (int)speed;
    if (displayValue > 9) displayValue = 9;

    showDigit(displayValue);

    Serial.print("7-Segment Shows: ");
    Serial.println(displayValue);

    delay(2000);  // keep the result visible for 2 sec
  }
}

## Future Improvements
- LCD display
- IR sensors for accuracy
- SD card logging
- Wireless transmission
