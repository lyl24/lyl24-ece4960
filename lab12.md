# Lab 12: Localization (real)

## Objective
For this lab, the goal is to use only the update step of the Bayes filter to perform localization on the real robot. In the lab, there are four poses marked on the ground, and for each pose, we want to run an observation loop to find the distance in front of the robot at each angle. 

The implementation of this lab should be extremely similar to Lab 9, and the main difference is that the data sent back by the robot needs to be in a format that can be run through the Bayes filter code in Python. The overall gist is that the robot turns 20 degrees, takes a distance reading, then keeps looping this process until 18 readings between 0-360 degrees are obtained. Then, the robot sends the angle and TOF distanec readings to my computer using Bluetooth. This sounds easy at first, but I had a feeling this lab would be a struggle considering the fact that my robot's wheels barely work at this point, leading to a lot of inaccuracies in Lab 9.

## Test localization in simulation
After running the notebook with the Bayes filter solution, I obtained the following plot.

![Part 1 plot](images/lab12/part 1 plot.PNG)

As seen in the plot, the ground truth is in green, the odometry readings are in red, and the Bayes filter result is in blue. 

## Implement observation loop functions
As previously mentioned, we want the robot to be able to rotate in a circle and gather 18 distance readings and the corresponding angle/heading. In order to do this, I reused code from Lab 9 and implemented the turning and data-sending functions as commands in Arduino.

```cpp
current_gyrX = 0.0;
tof_distance = get_tof_2();
gyr_array[0] = current_gyrX;
tof_array[0] = tof_distance;

setpoint = 20;
counter = 1;
motorspeed = 100;

while(counter < 18){
    previous_time = current_time;
    myICM.getAGMT();
    current_time = millis();
    previous_gyrX = current_gyrX;
    current_gyrX = get_gyroscope(&myICM);

    error_value = setpoint - current_gyrX;

    if(error_value > 1.0 && setpoint < 350.0){
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
}

for (int i=0; i<18; i++){
  imu_float.writeValue(gyr_array[i]);
  tof_2_float.writeValue(tof_array[i]);
}
break;
```

In the above code, the robot is supposed to get the initial angle and distance reading, then turn 20 degrees counter-clockwise and repeat this process. At each point in this loop, the data is stored in float arrays, and once all the data is collected, it is sent to my computer using Bluetooth.

In the Jupyter notebook, I edited the ```perform_observation_loop``` of class ```RealRobot``` in order to handle the data sent from the robot.

```python
def perform_observation_loop(self, rot_vel=120):
        self.setup_notify()
        self.ble.send_command(CMD.OBSERVATION, None)
        #await asyncio.sleep(30)
        time.sleep(30)

        tof_readings = [x / 1000 for x in self.tof_2_readings]
        imu_readings = [x / 1000 for x in self.imu_readings]
        
        sensor_ranges = np.array(tof_readings[0:18])[np.newaxis].T
        sensor_bearings = np.array(imu_readings[0:18])[np.newaxis].T

        return sensor_ranges, sensor_bearings
```

While the above code was able to get the robot to perform the observation loop, the arrays that were returned were sometimes empty. After discussing with the TAs, we thought that the ```time.sleep()``` function may have messed things up, but after implementing ```await asyncio.sleep()``` I found that the arrays returned were now always empty. In addition, the robot still overshoots each 20 degree turn by about 5 degrees (even when PID control is implemented), adding to all the existing problems. Since the results I was getting were so inconsistent, I realized that I probably had to revamp most of my code. Before embarking on a new path, I managed to squeeze out the following results using the code above.

![Take 1](images/lab12/take 1.PNG)

In the above plots, the blue dots are the results from my localization code, and the red x's are approximately where the robot is actually located. The predicted locations are quite far off from the ground truth.

## Implement observation loop functions: electric boogaloo
yeah


### [Click here to return to homepage](https://lyl24.github.io/lyl24-ece4960)
