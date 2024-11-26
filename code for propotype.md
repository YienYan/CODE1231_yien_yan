#include <Servo.h>
#include <MsTimer2.h>

volatile int currAngle;
volatile int targetAngle;
volatile int diffAngle;
volatile int ledState;
unsigned long lightOffTimer = 0; 
Servo servo_A0;

#define PinLED 2
#define PinLED1 3
#define PinLED2 4
#define PinLED3 5
#define PinLED4 6
#define PinSR501 A5
#define ServoOffsets 2          
#define ServoIntervalTimes 100  

void msTimer2_func() {
  if (targetAngle != currAngle) {
    diffAngle = targetAngle - currAngle;
    if (diffAngle > 0) {
      currAngle = currAngle + ServoOffsets;
      if (currAngle > 180) {
        currAngle = 180;
      }
    } else {
      currAngle = currAngle - ServoOffsets;
      if (currAngle < 0) {
        currAngle = 0;
      }
    }
    servo_A0.write(currAngle);

    
    if (currAngle == targetAngle && targetAngle == 180) {
      lightOffTimer = millis(); 
    }

    delay(0);  
  }
}

void setup() {
  currAngle = 0;
  targetAngle = 0;
  diffAngle = 0;
  ledState = 0;
  Serial.begin(9600);
  servo_A0.attach(A0);

  
  pinMode(PinLED, OUTPUT);
  pinMode(PinLED1, OUTPUT);
  pinMode(PinLED2, OUTPUT);
  pinMode(PinLED3, OUTPUT);
  pinMode(PinLED4, OUTPUT);
  pinMode(PinSR501, INPUT);

  servo_A0.write(0);
  delay(0);

  MsTimer2::set(ServoIntervalTimes, msTimer2_func);
  MsTimer2::start();
}

void loop() {
  Serial.println(digitalRead(PinSR501));

  
  if (digitalRead(PinSR501) == 1) {
    ledState = 1;              
    
    digitalWrite(PinLED, 1);
    digitalWrite(PinLED1, 1);
    digitalWrite(PinLED2, 1);
    digitalWrite(PinLED3, 1);
    digitalWrite(PinLED4, 1);

    
    targetAngle = 180;
  }

  
  if (ledState == 1 && lightOffTimer > 0 && millis() - lightOffTimer >= 3000) {
    ledState = 0; 
    
    digitalWrite(PinLED, 0);
    digitalWrite(PinLED1, 0);
    digitalWrite(PinLED2, 0);
    digitalWrite(PinLED3, 0);
    digitalWrite(PinLED4, 0);

    
    targetAngle = 0;
    lightOffTimer = 0; 
  }
}
