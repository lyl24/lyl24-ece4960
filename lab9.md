# Lab 9

## Objective:
For this lab, the goal is to build a map of a static room (the front room of the lab). To do this, the first step is to code the robot to turn around in a circle slowly while gathering data from the IMU and TOF sensors, then use transformation matrices to plot the data into a map of the room. This map will be used in future labs on localization and navigation.

## Programming the robot to simultaneously spin and collect data:

### Attempt #1: Stop and Go
In my first attempt, I tried to get the robot to rotate about 20 degrees per turn. In the setup loop, I first obtained the initial angle and TOF sensor reading, and I also set up a setpoint. After obtaining the initial angle, the setpoint would then be set to the value of the initial angle + 20 degrees. In the main loop, the robot continuously gathers IMU data, and if the the robot's heading is not yet at the setpoint, it will spin to the left. As soon as it reaches the setpoint, it will do a hard brake, delay for a bit, then gather the TOF reading. Once complete, the setpoint is incremented by 20 degrees again, and this process is repeated until the robot reaches a full 360 degree turn. The main loop code is as follows:

```ccp
while(central.connected()){
      read_data();
      
      previous_time = current_time;
      myICM.getAGMT();
      current_time = millis();
      previous_gyrX = current_gyrX;
      current_gyrX = get_gyroscope(&myICM);

      error_value = setpoint - current_gyrX;
      
      if(error_value > 1.0 && setpoint < 350.0){
        motorspeed = 120;
        turn_left();     
      }
      else if(error_value <= 1.0 && setpoint < 350.0){
        brake();
        delay(300);
        tof_distance = get_tof_2();
        gyr_array[counter] = current_gyrX;
        tof_array[counter] = tof_distance;
        counter++;
        setpoint = setpoint + 20;
        current_time = millis();
      }
      else{
        brake();
      }
```

To get the robot to start data collection, I ran some commands through Python and had the robot send data to my computer using Bluetooth:

```python
ble = get_ble_controller()
ble.connect()
rc = RobotControl(ble)
rc.setup_notify()
rc.ble.send_command(CMD.SEND_DATA, None)
time.sleep(5)
ble.disconnect()
print(f"IMU: {rc.imu_readings}")
print(f"TOF2: {rc.tof_2_readings}")
```

There was a recurring erorr where the gyroscope would not start at 0 degrees, and it typically would have a number around -10 degrees as the initial angle. I attempted to fix this issue by calibrating for the error: ```output_gyr = current_gyrX - offset_gyr```, where the ```offset_gyr``` variable calibrates all values so that the starting angle is equal to 0. This method did not make the results significantly better, so I eventually got rid of this line.

<iframe width="560" height="315" src="https://www.youtube.com/embed/5Am0pRBLigA" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

As seen in the video above, the robot tends to overshoot each turn by about 5 degrees (leading to a total overshoot of 80 degrees and a total rotation of 440 degrees). While working in the lab, I attempted to add on some "hacks" to minimize the overshoot, and I was only able to reduce the overshoot by a tiny bit. Vivek suggested that I add a PID controller to make the turns more accurate, which leads us to Attempt #2.

### Attempt #2: Stop and GO with PID controller
In my second attempt to get the stop and go code to work better, I implemented basic P control on the angle. Similar to the first attempt, I created setpoints that the robot would aim to stop at. In addition, I added a proportional term, scaled by the constant kp. The value of the proportional term is then scaled to a PWM range for the motors. When the error is small and the proportional term is low, the robot is close to the setpoint and slows down the turn to help prevent overshooting.

```ccp
while(central.connected()){
      read_data();
      
      previous_time = current_time;
      myICM.getAGMT();
      current_time = millis();
      previous_gyrX = current_gyrX;
      current_gyrX = get_gyroscope(&myICM);

      error_value = setpoint - current_gyrX;
      float proportional_term = error_value*kp;
      
      if(error_value > 2.0 && setpoint < 350.0){
        motorspeed = map(proportional_term, 0, 30, 90, 115);
        turn_left(); 
      }
      else if(error_value <= 2.0 && setpoint < 350.0){
        brake();
        delay(300);
        tof_distance = get_tof_2();
        gyr_array[counter] = current_gyrX;
        tof_array[counter] = tof_distance;
        counter++;
        setpoint = setpoint + 20;
        current_time = millis();
      }
      else{
        brake();
      }
```

After implementing the code on the robot, the total overshoot did not decrease significantly. The robot kept overshooting by about 80 degrees, and it spun in a circle similar to the video above. I noticed that in both of these attempts, the wheels would not turn steadily at all at lower PWMs. For example, during each run, one wheel would suddenly be drastically slower or faster than the other, and my attempts to introduce a scaling factor to equalize the wheel speed did not help much. In addition, for each individual run, there would be some turns where neither wheel could turn at all, as well as other turns where it would suddenly find the energy to rotate very quickly and overshoot the turn. 

I implemented Attempts #1 and #2 while in lab before Spring Break, and I ended up collecting data using Attempt #1 just to see if the results would still be okay. The robot kept overshooting to about 440 degrees physically but thinking it rotated 360 degrees, so each of the resulting data points progressively got more and more incorrect. I was worried that the map using this bad data would completely be a hairball, however it looked somewhat reasonable when plotted out (see below).

### Attempt #3: A new approach to Stop and Go
After plotting the data from Attempt #1, I was unhappy with the results, so I wanted to see if I could code up a better stop and go program over Spring Break.



```
def transformation(angle, distance, x_0, y_0):
    measured_distance = np.array([[0],[distance],[1]])
    tf_matrix = np.array([[np.cos(angle), -np.sin(angle), x_0], [np.sin(angle), np.cos(angle), y_0], [0, 0, 1]])
    output_matrix = np.matmul(tf_matrix, measured_distance)
    
    x_1 = output_matrix[0][0]
    x_2 = output_matrix[1][0]
    return x_1, x_2
```

```
def rescaler(x, x_min, x_max):
    x_norm = (x-x_min)/(x_max-x_min)
    x_norm = x_norm*440
    return x_norm
```


![Battery hookup](images/lab4/battery hookup.jpg)

[Motor Driver Datasheet](https://www.ti.com/lit/ds/symlink/drv8833.pdf?HQS=dis-dk-null-digikeymode-dsf-pf-null-wwe&ts=1646507944819&ref_url=https%253A%252F%252Fcei-lab.github.io%252F)

### [Click here to return to homepage](https://lyl24.github.io/lyl24-ece4960)
