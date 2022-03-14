# Lab 6: Closed-loop Control (PID)

## Objective: Get experience with PID control.

## Prelab:
For this lab, I chose to do Task A (Don't Hit the Wall). Before starting to implement PID control, I set up a system for debugging. To do this, I wrote code that would integrate Bluetooth with code for the robot so that the on-board sensors could send readings back to my computer. The best way to do this is to have the robot start on an input from my computer sent over Bluetooth, store data and time stamps for the front TOF sensor in arrays, then send the data back to the computer over Bluetooth. 

First, I combined Arduino code for Bluetooth, the TOF sensors, and motor drivers into one script. I added two commands (which the robot can recognize using the read_data() function in the Arduino code) so that I could tell the car to start and stop through Bluetooth.

```
case MOVE_FORWARD:
    Serial.println("move forward");
    int motorspeed;
    success = robot_cmd.get_next_value(motorspeed);
    if (!success)
      return;
    analogWrite(motor1f, motorspeed);
    analogWrite(motor2f, int(motorspeed*constant));
    break;

case STOP:
    Serial.println("case 2");
    analogWrite(motor1f, 0);
    analogWrite(motor2f, 0);
    analogWrite(motor1b, 0);
    analogWrite(motor2b, 0);
    break;
```

Next, I wrote a function that would allow the robot to get TOF distance readings.

```
float get_tof_2(){
  distanceSensor2.startRanging();
  distance2 = distanceSensor2.getDistance();
  distanceSensor2.clearInterrupt();
  distanceSensor2.stopRanging();

  return distance2;
}
```

The following code can be used to store the readings into arrays.

```
get_current_time = millis();
time_array[counter] = get_current_time;
tof_distance = get_tof_2();
tof_array[counter] = tof_distance;

counter++;
```

To send the arrays to my computer over Bluetooth, I wrote each value in the array to float characteristics, and they are stored in a growing array in the Python code.

```
for (int i=0; i<sizeof(tof_array); i++){
  tof_2_float.writeValue(tof_array[i]);
  tx_characteristic_float.writeValue(time_array[i]);
}
```

In the Jupyter notebook, I created a robot control class, and it also has functions for notification handlers and can send start/stop commands to the robot. However, the code is unable to upload to the Artemis board, and a blue light on the board flashes when I try. I was unable to figure out how to fix this issue, so I ended up using a method in which the board continuously sends data over Bluetooth. This method really slows down the sampling rate because sending data over Bluetooth takes up extra time, but this method is still sufficient when running the robot at a slower speed.

```
get_current_time = millis();
tx_characteristic_float.writeValue(get_current_time);
tof_distance = get_tof_2();
tof_2_float.writeValue(tof_distance);
```



### [Click here to return to homepage](https://lyl24.github.io/lyl24-ece4960)
