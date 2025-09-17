---
layout: default
title: Movement Control
nav_order: 4
grand_parent: Archive
parent: Motor Control
---

**Controlling Linear Actuator by Entering Length**

**A4988 Stepper Motor Pins:**

![A4988 Stepper Motor Pins](/assets/media/image1.png){:width="500px"}

**Vdd and GND:** Should be connected to the 5v and GND parts of the Arduino.

**Vmot and GND:** Should be connected to 12 volt and GND to provide the 12 volt needed by the stepper motor.

**1A,1B,2A,2B:** Pins to which the stepper motor is connected.

**Dır:** Controls the direction of the motor.

**Step:** Controls the steps.

**MS1, MS2, MS3:** Microstep Selection Pins.

**Sleep and Reset:** When they are connected to each other, the controller becomes active.

**En:** When the Enable pin is active, the motor is grounded. We can limit the power usage by making this pin active and passive.

Micro step Mode   | MS1 | MS2 | MS3                                           
-----------------|-----|-----|-----
Full Step        | low | low | low
Half Step        | high| low | low
Quarter Step     | low | high| low
Eighth Step      | high| high| low
Sixteenth Step   | high| high| high

For one revolution needed steps are calculated as:

Full Step mode:

$$\frac{360}{1.8{^\circ}} = 200$$

Half step:

$$\frac{360}{0.9{^\circ}} = 400$$

Quarter step:

$$\frac{360}{0.45{^\circ}} = 800$$

Eighth step:

$$\frac{360}{0.225{^\circ}} = 1600$$

Sixteenth step:

$$\frac{360}{0.1125{^\circ}} = 3200$$

**Stepper Motor Controller Connections:**

Stepper motor model is: 17HS4401S

![Stepper Motor](/assets/media/image2.png){:width="600px"}

**Trapezoidal lead screw motion principle:**

![Lead Screw Principle](/assets/media/image3.png){:width="500px"}

The screw rotates, the nut does not rotate, the nut moves along the screw.

**Linear Actuator:**

Linear actuators are created by properly combining the stepper motor and trapezoidal lead screw. With each step of the stepper motor, the part attached to the screw shaft moves by the length of the Lead. It is used to move a load back and forth.

$$L = p × n_s$$

$$L = Lead of thread.$$

$$P = Thread pitch.$$

$$n_s = Number of thread starts.$$

**Length Calculation in Code:**

$$toplamAdim = (mmFinal / 8) × stepsPerRevolution$$

The specified `stepsPerRevolution` is the number of steps required for the stepper motor to complete one revolution.

The number of `starts` of the used shaft: $$n_s = 4$$

Pitch value of shaft: $$p = 2$$

Lead of thread: $$L = 8$$

So when the `stepsPerRevolution` is completed, the shaft moves 8 mm. By multiplying the formula by 1/8, we ensure that the shaft moves 1 mm when the `stepsPerRevolution` is completed.

**T8 Trapezoidal lead screw:**

![T8 Lead Screw](/assets/media/image4.png){:width="500px"}

**Calculating the force created by the torque applied by the stepper motor on the shaft**

![Inclined Plane 1](/assets/media/image5.jpeg){:width="500px"}

![Inclined Plane 2](/assets/media/image6.jpeg){:width="500px"}

h = length of lead: 8 mm  
D = diameter: 8 mm  
C = Circumference: π × 8 = 25,13 mm  
Length of Helix: √(25.13² + 8²) = 26,34 mm

**Formula for converting stepper motor torque into force**

![Torque to Force Formula](/assets/media/image7.jpg){:width="500px"}

**Torque work formula and force work formula**

![Torque Work Formula](/assets/media/image8.jpg){:width="500px"}

**Equality of the number of turns of the stepper motor and the shaft helix length**

![Helix Length](/assets/media/image9.jpg){:width="500px"}

$$T_M = 430\ Nmm$$

$$h = 8\ mm$$

$$\sin(\alpha) = h / \text{length of helix} = 0.304 \Rightarrow \alpha = 18^\circ$$

$$T_M = \frac{8F_N}{2\pi} \Rightarrow F_N = 338.58\ N$$

![Forces on Screw](/assets/media/image10.jpeg){:width="500px"}

$$F_T = F_N \times \sin(18) = 104.62\ N$$

$$F_\Ö = F_N \times \cos(18) = 322.01\ N$$

![Forces Diagram](/assets/media/image11.jpeg){:width="500px"}

**When friction force acts:**

Real life systems are under the influence of friction force. The situation where the force applied to the nut does not move the nut is called **autoblocking**.

Autoblocking requirement:

**α ≤ p**

**Linear Actuator:**

![Linear Actuator](/assets/media/image12.jpeg){:width="500px"}

**Controlling Linear Actuator by Entering Length in Arduino Uno**

<div class="code-example" markdown="1">
#define dirPin 6
#define stepPin 7
#define controlPin 2
#define dirPin 6
#define stepPin 7
#define controlPin 2
#define MS1 3
#define MS2 4
#define MS3 5
#define stepsPerRevolution 200
#define joyX A0
#define joyY A1
double xValue;
double yValue; 
double toplamAdim;
#define mmFinal 50
int i;
int son;
double tMotor = 430 ; //stepper motor torque
double d = 8 ; //diameter
double h = 8 ; //lead
double circumference; 
double lengthOfHelix;
double fN; //force exerted by stepper motor
double fT; //nut turning force
double fA; //axial force
double angle; 
double value;
double angle2degree;

void setup(){
  pinMode(stepPin, OUTPUT); 
 pinMode(dirPin, OUTPUT); 
  pinMode(MS1, OUTPUT);
 pinMode(MS2, OUTPUT);
  pinMode(MS3, OUTPUT);
  Serial.begin(9600);
}

void loop() {
  // stepper motor torque force relationship formulas
  circumference = (3.1415 * d);
  lengthOfHelix = sqrt((pow(circumference,2) + pow(h,2)));
  value = h/lengthOfHelix;
  angle = asin(value);  // output is radian:
  angle2degree = (angle * 180)/ 3,1415;
  fN = (( tMotor * 2 * (3.1415))/ h); 
  fT = fN * sin(angle); // Input is radian:
  fA = fN * cos(angle); // Input is radian:

  Serial.print(" angle ");
  Serial.print(angle2degree);
  Serial.print("°");
  Serial.print(" fN value ");
  Serial.print(fN);
  Serial.print(" fT value ");
  Serial.print(fT);
  Serial.print(" fA value ");
  Serial.print(fA);

  // Microstep control settings:
 digitalWrite(MS1, LOW);
 digitalWrite(MS2, LOW);
 digitalWrite(MS3, LOW);
 toplamAdim = (mmFinal / 8) * stepsPerRevolution;
 

 // Set the spinning direction counterclockwise:
  
 if(mmFinal > 0){
  digitalWrite(dirPin, HIGH); 
  if(son != 1){
   for( i = 0; i < toplamAdim; i++){

   digitalWrite(stepPin, HIGH); 
   delayMicroseconds(2000); 
   digitalWrite(stepPin, LOW); 
   delayMicroseconds(2000); 
   son = 1;
    }
   }
  }
  if(mmFinal < 0){
   // Set the spinning direction counterclockwise:
  digitalWrite(dirPin, LOW); 
   if(son != 1){
    for( i = 0; i < -toplamAdim; i++){
    digitalWrite(stepPin, HIGH); 
    delayMicroseconds(2000); 
    digitalWrite(stepPin, LOW); 
    delayMicroseconds(2000); 
    son = 1;
    }
   }
  }
 }
