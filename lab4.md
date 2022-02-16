# Lab 4: Characterize your Car 

## Objective: Get familiar with and document the capabilities of the hardware.

## Prelab:
Before starting measurements, I checked that the Artemis works while being powered by a 600mAh battery. To do this, I first took the 2mm white JST connector and reversed the positions of the red and black wires so that they would match up with the + and - ports on the board. Next, I soldered the JST connector to a battery connector. Since the wires are also reversed on the battery connect, I put labels on the battery connector wires to mark that the black wire connects to the red wires on the JST connector and the battery, and the red wire connects to the black wires on the JST connector and the battery. The setup can be seen below:

![Battery hookup](images/lab4/battery hookup.jpg)

After carefully verifying that all the wires were set up correctly (despite the confusing colors), I plugged in the battery while the Artemis was connected via USB to my computer, and the yellow LED lit up. This meant that the battery was being charged from the USB port, and everything was working properly.

## Testing!
Next, we can move on to testing the capabilities of the RC car. The car is powered by an 850mAh battery, and it is controlled using a remote control.

![Car](images/lab4/car.jpg)

### Dimensions and Weight

| Length (including wheels) | 18 cm |
| Width (including wheels) | 14.2 cm |
| Height (including wheels) | 8 cm |
| Length (chassis) | 14.2 cm |
| Width (chassis) | 7.3 cm |
| Height (chassis) | 5 cm |
| Weight | ___ |

### Batteries

| Battery Life | 8:11 (when only turning in circles) |
| Time to full charge | a really long time |

### Drifting Distance
I wanted to measure the distance that it takes for the car to drift to a stop after releasing the driving buttons. Understanding how far the car moves after releasing all controls is helpful for improving the accuracy of the car's movements. For example, if the car is driving forward autonomously and we want it to suddenly stop after 5 meters, we would need to tell it to stop before 5 meters so that it can drift the rest of the way and not overshoot the distance.

To do this, I set up a tape measure in the hallway in Phillips Hall, drove the car forward at maximum speed (this required about 5 feet of a head start), and measured the distance between when I released the buttons and when the car came to a full stop. I took five measurements in a row and noticed that the drifting distance decreased over time. This is probably due to the short battery life, and after many trials, the maximum speed of the car was much lower than its maximum speed at full charge.

| Trial  | Distance |
| ------------- | ------------- |
| 1 | 371 cm |
| 2 | 389 cm |
| 3 | 338 cm |
| 4 | 300 cm |
| 5 | 238 cm |

### Braking Distance
Similar to the drifting distance, I wanted to measure how fast the car can to come to a stop. This can also help us improve the accuracy of the car's movements (especially sudden stops) like the drifting distance measurements.

I set up the experiment in the same way as before, however instead of releasing all buttons and letting the car drift to a stop, I would immediately press the back button to see how fast it could reverse its direction. After ramping the car up to full speed, I measured the distance between when I started reversing direction on the remote control and when the car actually reversed direction. (I considered the moment the car fully reversed direction as the stopping point, since the car's velocity momentarily reaches zero.) The measured distances decreased over time, most likely due to the short battery life and lowered maximum speed after many trials.

| Trial  | Distance |
| ------------- | ------------- |
| 1 | 104 cm |
| 2 | 90 cm |
| 3 | 90 cm |
| 4 | 75 cm |
| 5 | 70 cm |



### [Click here to return to homepage](https://lyl24.github.io/lyl24-ece4960)
