# Lab 13: Path Planning and Execution

## Objective
For this lab, the goal is to make the robot navigate through a set of waypoints as quickly and accurately as possible. The course is as follows:

```
1. (-4, -3)    <--start
2. (-2, -1)
3. (1, -1)
4. (2, -3)
5. (5, -3)
6. (5, -2)
7. (5, 3)
8. (0, 3)
9. (0, 0)      <--end
```

![Planned Path](images/lab 13/planned path.png)

Before starting this lab, I wanted to code the robot to go around the course in the same way as the simulation robot. At each point, the robot will first do an observation loop to determine its current location. Then, it will calculate the angle and distance it needs to move in order to reach the next point in the trajectory. Once it reaches this point, it will do another observation loop to find its current location, then repeat this process until it reaches all points in the planned trajectory. Unfortunately, I ran out of time to implement this method since I focused so much on Lab 12 (Localization: real). Even though I managed to get good results for Lab 12, the success rate for determining the correct position was at most 50%, which wouldn't cut it for this lab. 

In this lab, I used Jonathan's robot once again, since my robot was still Problematic. I decided to use an open loop method where I control each of the turns and forward/backwards movements. I implemented simple P control on the distance travelled as well as for the angle when turning, then wrote this into commands in Arduino so that I could tell the robot to execute each action whenever I needed.

Arduino code for moving forward, where the input distance is the distance between the front TOF sensor and the nearest obstacle in front of the robot:
```cpp
case MOVE_FORWARD:
    float input_distance;
    success = robot_cmd.get_next_value(input_distance);
    if (!success)
        return;

    current_distance = get_tof_2();
    error_value = current_distance - input_distance;
    proportional_term = error_value*kp;

    while((error_value > 3.0) || (error_value < -3.0)){
        current_distance = get_tof_2();
        error_value = current_distance - input_distance;
        proportional_term = error_value*kp;

        if(error_value > 3.0){
          motorspeed = map(proportional_term, 0, 2000, 40, 80);
          move_forward();
        }
        else if(error_value < -3.0){
          motorspeed = map(proportional_term, 0, -500, 40, 80);
          move_backward();
        }
        else{
          brake();
        }
    }
    break;
 ```

Python code for executing the forward movement:
```python
def move_forward(self, distance):
    print("moving forward")
    self.ble.send_command(CMD.MOVE_FORWARD, distance)
    pass
```
```python
rc.ble.send_command(CMD.MOVE_FORWARD, insert_distance_here)
```

Arduino code for turning left, where the input angle is the angle I want the robot to turn by:
```cpp
case TURN_LEFT:  
    float input_left;
    success = robot_cmd.get_next_value(input_left);
    if (!success)
        return;

    previous_time = current_time;
    myICM.getAGMT();
    current_time = millis();
    previous_gyrZ = current_gyrZ;
    current_gyrZ = get_gyroscope(&myICM);
    offset = -(current_gyrZ);
    current_gyrZ = current_gyrZ + offset;
    first_gyrZ = current_gyrZ;

    setpoint = -(input_left);

    error_value = current_gyrZ - setpoint;
    proportional_term = error_value*kp;

    while((error_value > 1.0)){
      previous_time = current_time;
      myICM.getAGMT();
      current_time = millis();
      previous_gyrZ = current_gyrZ;
      current_gyrZ = get_gyroscope(&myICM);

      error_value = current_gyrZ - setpoint;
      proportional_term = error_value*kp;

      if(error_value > 1.0){
        motorspeed = map(proportional_term, 0, 80, 100, 130);
        turn_left();     
      }
      else if(error_value < 1.0){
        brake();
      }
    }            
    break;
```

Python code for executing the left turn:
```python
def left(self, angle):
    print("turning left")
    self.ble.send_command(CMD.TURN_LEFT, angle)
    pass
```
```python
rc.ble.send_command(CMD.TURN_LEFT, insert_angle_here)
```

The code for moving backward and turning right is implemented in the same way as the code for moving forward and turning left, respectively. With these commands, I'm now able to send individual movement commands to the robot, and I can link the commands together to allow the robot to move through the course.

```python
rc.ble.send_command(CMD.MOVE_FORWARD, 400)
time.sleep(4)
rc.ble.send_command(CMD.TURN_LEFT, 70)
time.sleep(2)
rc.ble.send_command(CMD.MOVE_BACKWARD, 1050)
time.sleep(4)
rc.ble.send_command(CMD.MOVE_BACKWARD, 2000)
time.sleep(4)
rc.ble.send_command(CMD.MOVE_BACKWARD, 2100)
time.sleep(4)
rc.ble.send_command(CMD.TURN_LEFT, 65)
time.sleep(2)
rc.ble.send_command(CMD.MOVE_FORWARD, 400)
time.sleep(4)
rc.ble.send_command(CMD.TURN_LEFT, 65)
time.sleep(2)
rc.ble.send_command(CMD.MOVE_FORWARD, 400)
time.sleep(4)
rc.ble.send_command(CMD.TURN_RIGHT, 80)
time.sleep(2)
rc.ble.send_command(CMD.MOVE_BACKWARD, 700)
time.sleep(4)
rc.ble.send_command(CMD.TURN_LEFT, 65)
time.sleep(2)
rc.ble.send_command(CMD.TURN_LEFT, 65)
time.sleep(2)
rc.ble.send_command(CMD.MOVE_FORWARD, 400)
time.sleep(4)
rc.ble.send_command(CMD.TURN_LEFT, 65)
time.sleep(2)
rc.ble.send_command(CMD.MOVE_FORWARD, 600)
time.sleep(4)
rc.ble.send_command(CMD.TURN_RIGHT, 70)
time.sleep(2)
rc.ble.send_command(CMD.MOVE_BACKWARD, 1300)
time.sleep(4)
```

Using this series of commands, I ran four trials, and the robot was able to make it through the course and reach the final point two times.

<iframe width="560" height="315" src="https://www.youtube.com/embed/JddoyY4h0Jk" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

Since this method doesn't allow the robot to localize, the robot never really knows where it is located. As it makes minor mistakes throughout the course, these errors build up, and it is unable to fix these errors on the go. If the course was longer and more complex, the success rate at which the robot makes it to the final point would plummet due to the amount of error built up over time. If the robot could get robust localization results, it would be able to take previous mistakes into account when moving between points, and it should be able to accurately make it to the end of the course most of the time.

### [Click here to return to homepage](https://lyl24.github.io/lyl24-ece4960)
