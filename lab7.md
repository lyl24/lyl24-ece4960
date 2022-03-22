# Lab 7: Kalman Filter

## Objective: Implement a Kalman Filter to help speed up the TOF sensor for Lab 8 (Stunts)

## Lab Procedure:

## Part 1: Step Response
The first step is to execute a step response by driving the car towards a wall as fast as possible while logging motor input values and the TOF sensor output. To do this, I added the code below to the Artemis board, which allows the robot to drive towards the wall at top speed (PWM input = 255). With this code, I was able to store motor input values, TOF sensor output, and time into arrays, which were then sent over to my computer using bluetooth after the robot finished running.

```
read_data();
  tof_distance = get_tof_2();

  get_current_time = millis();
  time_array[counter] = float(get_current_time);
  tof_array[counter] = float(tof_distance);

  float error = tof_distance - setpoint;
  if(error >= 300){
    motorspeed = 255;
    analogWrite(motor1b, 0);
    analogWrite(motor2b, 0);
    analogWrite(motor1f, motorspeed);
    analogWrite(motor2f, int(constant*motorspeed));
    motor_array[counter] = float(motorspeed);
    Serial.println(motorspeed);
  }
  else{
    analogWrite(motor1f, 255);
    analogWrite(motor1b, 255);
    analogWrite(motor2f, 255);
    analogWrite(motor2b, 255);
    motor_array[counter] = float(0.0);
    Serial.println("stop");
  }
  counter++;
```

From the data I collected, I was able to make graphs for the TOF sensor output (distance vs time), the computed speed (velocity vs time), and the motor input.

TOF sensor output:




Measure both the steady state speed and the 90% rise time and compute the A and B matrix.

![Battery hookup](images/lab4/battery hookup.jpg)

[Motor Driver Datasheet](https://www.ti.com/lit/ds/symlink/drv8833.pdf?HQS=dis-dk-null-digikeymode-dsf-pf-null-wwe&ts=1646507944819&ref_url=https%253A%252F%252Fcei-lab.github.io%252F)

### [Click here to return to homepage](https://lyl24.github.io/lyl24-ece4960)
