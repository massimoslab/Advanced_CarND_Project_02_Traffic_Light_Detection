[image1]: ./images/i2c_1.jpg

# Annex 05 : I2C Connection

There are various ways to connect and control an Arduino from a Raspberry Pi. The easiest solution is to use a USB-to-mini-USB cable. The issue with this solution is that an Arduino can be connected only to one Raspberry Pi. Therefore this would be what is also called a single-master to single-slave solution. In this scenario the Raspberry Pi is the master since it is the device that controls the Arduino, which is the slave. What we want and need is a solution that allows us to have:
* a single-master to multi-slave
* a multi-master to single-slave
* a multi-master to multi-slave

A I2C connection solves this problem in an easy way.

## 01. I2C Protocol

I2C or inter-integrated circuit protocol is a hardware protocol designed to allow multiple, slave integrated circuits to communicate with one or more master. It uses two bidirectional open-drain lines, Serial Data Line (SDA) and Serial Clock Line (SCL), pulled up with resistors. Typical voltages used are +5 V or +3.3 V, although systems with other voltages are permitted.


## 02. Devices assembly

xxx

![alt text][image1]


## 03. The Arduino code

What things you need to install the software and how to install them

```
#include <Wire.h>
#include <Servo.h>

#define SLAVE_ADDRESS 12

int enA = 6;
int in1 = 8;
int in2 = 7;
int servoPIN = 10;

Servo steeringServo;


void setup() {
  Serial.begin(9600);
  Wire.begin(SLAVE_ADDRESS);
  Wire.onReceive(receiveData);
  pinMode(enA, OUTPUT);
  pinMode(in1, OUTPUT);
  pinMode(in2, OUTPUT);
  steeringServo.attach(servoPIN);
}


void loop() {
  delay(100);
}


void receiveData(int byteCount) {
  int cmd[3];
  int i = 0;

  while(Wire.available()) {
    cmd[i] = Wire.read();
    i++;
  }

  // send the throttle value to the motor
  analogWrite(enA, cmd[1]);
  digitalWrite(in1, 1);
  digitalWrite(in2, 0);

  // send the steering value to the servo
  steeringServo.write(cmd[2]);
}
```

#### Understanding the code

The **void setup()** function is run first with its content between brackets. The **Serial.begin(9600)** sets up the speed of the serial port to 9600 baud. The baud setting in the serial monitor window must match this value so that the Arduino and serial monitor window are communicating at the same speed.

The **void loop()** function is run second with all its content between brackets.
The **Serial.println("Hello, world!")** sends the text *Hello World!* to the serial / USB port for display in the serial monitor window.

## 04. The Raspberry Pi code

What things you need to install the software and how to install them

```
from smbus import SMBus
import time


bus = SMBus(1)
arduinoAddress = 12


step_total = 20
step_count = 0

throttle_max = 255
throttle_min = 0
throttle_step = int((throttle_max - throttle_min) / step_total)

steering_max = 130
steering_min = 50
steering_step = int((steering_max - steering_min) / step_total)

throttle_val = throttle_min
steering_val = steering_min


while True:

	while step_count < step_total:
		bus.write_i2c_block_data(arduinoAddress, 1, [throttle_val, steering_val])
		throttle_val += throttle_step
		steering_val += steering_step
		step_count += 1
		print("throttle: " + str(throttle_val) + ", steering: " + str(steering_val))
		time.sleep(0.5)

	while step_count > 0:
		bus.write_i2c_block_data(arduinoAddress, 1, [throttle_val, steering_val])
		time.sleep(0.1)
		throttle_val -= throttle_step
		steering_val -= steering_step
		step_count -= 1
		print("throttle: " + str(throttle_val) + ", steering: " + str(steering_val))
		time.sleep(0.5)
```

#### Understanding the code

The **void setup()**

## Author

**Massimo Passamonti**: [email me](me@massimoslab.com)

## License

This project and its source code are licensed under the MIT License. [See MIT License](https://github.com/github/choosealicense.com/blob/gh-pages/LICENSE.md) file for more details.
