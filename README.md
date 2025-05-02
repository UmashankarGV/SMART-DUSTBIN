#include <Servo.h> // Include servo library

Servo servo;     // Create a servo object

int trigPin = 5;    // Ultrasonic sensor trigger pin
int echoPin = 6;   // Ultrasonic sensor echo pin
int servoPin = 7;  // Servo motor control pin

// Assuming the UV sensor is connected to pin 8 (comment out if not used)
// int uvSensorPin = 8;

int ledPin = 10;   // LED pin (not used in this modification)

long duration, dist, average;   
long aver[3];   // Array to store distance readings for averaging

void setup() {
  Serial.begin(9600);
  servo.attach(servoPin);  // Attach servo to control pin

  pinMode(trigPin, OUTPUT);  // Set trigger pin as output
  pinMode(echoPin, INPUT);   // Set echo pin as input
  // pinMode(uvSensorPin, INPUT); // Comment out if not using UV sensor

  servo.write(0);         // Close cap on power on (assuming servo controls a lid)
  delay(100);
  servo.detach(); // Ensure complete detachment after initial movement
}

void measure() {
  digitalWrite(trigPin, LOW);
  delayMicroseconds(5);
  digitalWrite(trigPin, HIGH);
  delayMicroseconds(15);
  digitalWrite(trigPin, LOW);
  pinMode(echoPin, INPUT);
  duration = pulseIn(echoPin, HIGH);

  // Check for timeout (optional): Add a check for a very long duration value 
  // (indicating no signal received) and handle it appropriately (e.g., set dist to 0).
  if (duration > 10000) {
    Serial.println("Ultrasonic sensor timeout!");
    dist = 0;
  } else {
    dist = (duration/2) / 29.1;    // Calculate distance if signal received
  }
}

void loop() {
  for (int i = 0; i <= 2; i++) {  // Measure distance three times and store in array
    measure();
    aver[i] = dist;
    delay(10);
  }

  dist = (aver[0] + aver[1] + aver[2]) / 3; // Calculate average distance

  // UV sensor integration (comment out if not used)
  // int uvValue = analogRead(uvSensorPin); // Read analog value from UV sensor
  // if (uvValue > threshold) { // Replace 'threshold' with a value indicating high UV
  //   // Implement logic for UV detection, like turning on an LED or sending a signal
  //   Serial.println("UV sensor detected high UV levels!");
  // }

  if (dist < 50) { // If object is close (replace 50 with desired distance)
    servo.attach(servoPin);
    delay(1);
    servo.write(0);  // Close lid
    delay(3000);
    servo.write(150); // Open lid
    delay(1000);
    servo.detach(); // Detach servo after movement
  }

  Serial.print("Distance: ");
  Serial.print(dist);
  Serial.println(" cm");
}

