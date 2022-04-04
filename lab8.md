# Lab 8: Stunts!

## Objective
The ultimate goal of this lab is to do flashy stunts! Fast!

## Controlled Stunts
I chose to do Task A, in which the robot drives quickly towards a wall, does a flip right on the sticky mat located ~0.5 m from the wall, then speeds back in the direction it came in. My PID controller from Lab 6 doesn't work quickly enough to do this stunt, and I was not able to get the Kalman Filter to work successfully on the robot. Therefore, I manually tuned the distance to the wall for which the robot initiates the flip. To do this, I first measured the distance to the wall from the starting point.

```python
ble = get_ble_controller()
ble.connect()
rc = RobotControl(ble)
rc.get_distance()
distance = ble.receive_float(ble.uuid['RX_TOF2'])
print(distance)
```

From previous labs, I know that the maximum velocity of the robot is about 2250 mm/sec, and this is the velocity I should use to calculate the time it takes to reach the wall when the PWM value is set to the maximum value of 255. The robot has to flip a little bit before it reaches the wall, and it also takes a bit of time to accelerate to full speed, therefore I incorporated an offset variable to help calibrate the flipping distance.

```python
max_velocity = 2250
offset_time = -500
forward_time = (distance+offset_time)/max_velocity
print(forward_time)
```

While figuring out how to make the robot do a flip, I decided that the best strategy would be to write some driving functions in the Arduino code, then send commands from my computer using Bluetooth to initiate these functions. Some of the basic commands included braking, coasting, moving forward, and moving backward.

```ccp
case STOP:
    analogWrite(motor1f, 255);
    analogWrite(motor2f, 255);
    analogWrite(motor1b, 255);
    analogWrite(motor2b, 255);
    break;

case COAST:
    analogWrite(motor1f, 255);
    analogWrite(motor2f, 255);
    analogWrite(motor1b, 255);
    analogWrite(motor2b, 255);
    break;

case MOVE_FORWARD:
    success = robot_cmd.get_next_value(motorspeed);
    if (!success)
      return;
    analogWrite(motor1f, motorspeed);
    analogWrite(motor2f, int(motorspeed*constant));
    break;
 
 case MOVE_BACKWARD:
    success = robot_cmd.get_next_value(motorspeed);
    if (!success)
      return;
    analogWrite(motor1b, motorspeed);
    analogWrite(motor2b, int(motorspeed*constant));
    break;
```

I also tried to write an entire function that would execute the flip, however, it did not work, and it was difficult to troubleshoot since all of the individual actions that go into a flip were coded into a single command. The best combination of commands that led to a flip included moving forward at full speed, suddenly braking to allow the robot to flip over, then moving forward again until it reaches the starting point. However, when I implemented the following sequence: ```MOVE_FORWARD``` ```STOP``` ```MOVE_FORWARD```, the timing for the sudden brake was not quite right. I realized that when I send commands over Bluetooth, there is a slight lag, and so I created a ```FAST_REVERSE``` command that packages the last two commands together and therefore reduces the delay.

```ccp
case FAST_REVERSE:
    success = robot_cmd.get_next_value(motorspeed);
    if (!success)
      return;
    analogWrite(motor1f, 255);
    analogWrite(motor2f, 255);
    analogWrite(motor1b, 255);
    analogWrite(motor2b, 255);
    analogWrite(motor1b, motorspeed);
    analogWrite(motor2b, int(motorspeed*constant));
    break;
```

With the commands ready to go, I sent the commands to the robot using Python. (*Note: In the code below, a PWM value of 0 actually corresponded to the maximum speed because I accidentally messed up the directionality of the motors in the Arduino code. This caused a lot of confusion, but I eventually fixed the issue afterwards.)

```python
rc.move_forward(0)
time.sleep(forward_time)
rc.fast_reverse(0)
time.sleep(forward_time*2.5)
rc.stop()
```

After many failed runs (some of which were very close to succeeding), I captured a nice video of the robot successfully completing Task A.

<iframe width="560" height="315" src="https://www.youtube.com/embed/S9WTDIpaqSE" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Open Loop, Repeatable Stunts
For the extra stunt, I wanted the robot to be able to avoid and go around obstacles in its environment. While this isn't technically a flashy stunt, I wanted the robot to be able to behave like a Roomba, so I've called this move "The Roomba." To program "The Roomba," the robot first drives forward while continuously gathering distance readings in front of it using a front-mounted TOF sensor. As soon as it detects an obstacle in front of it, it first rotates to the right and collects a new distance reading, then rotates to the left and collects another distance readings. Comparing the two distance readings, it will choose to go in the direction with a higher value, allowing it to travel for longer before hitting another obstacle. 

```ccp
while(central.connected()){
  read_data();
  tof_distance = get_tof_2();
  if(tof_distance > 300){
    motorspeed = 75;
    forward();
  }
  else if(tof_distance <= 300 && tof_distance > 20){
    brake();
    delay(5);
    motorspeed = 180;
    turn_right();
    delay(500);
    brake();
    delay(5);
    float right_distance = get_tof_2();
    delay(5);
    turn_left();
    delay(1000);
    brake();
    delay(5);
    float left_distance = get_tof_2();
    delay(5);
    turn_right();
    delay(500);
    brake();
    delay(5);

    if(right_distance > left_distance){
      turn_right();
      delay(500);
      brake();
    }
    else{
      turn_left();
      delay(500);
      brake();
    }
  }
  else{
    motorspeed = 100;
    backward();
    delay(200);
    brake();
    delay(5);
  }
}
}
else{
brake();
}
```

It was extremely difficult to calibrate the motors so that the turns were the same on both sides of the robot -- within the same run, the robot would sometimes massively overshoot a turn and then get stuck on the next turn, and this problem was exacerbated by a dying battery. In the end, I was able to get a couple of runs that worked decently well, and the robot was able to sense and avoid objects in front of it.

<iframe width="560" height="315" src="https://www.youtube.com/embed/IJqRwO_fuyI" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Bloopers


Anyways, thanks for coming to my TED talk.

### [Click here to return to homepage](https://lyl24.github.io/lyl24-ece4960)
