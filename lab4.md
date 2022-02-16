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
Knowing the dimensions of the car can help us determine where the car can fit. The car is quite short and wide, so it could fit under things very well (like a Roomba).

| Dimension  | Measurement |
| ------------- | ------------- |
| Length (including wheels) | 18 cm |
| Width (including wheels) | 14.2 cm |
| Height (including wheels) | 8 cm |
| Length (chassis) | 14.2 cm |
| Width (chassis) | 7.3 cm |
| Height (chassis) | 5 cm |

In addition, the weight of the car with just the battery is ___ . Given that the car is pretty short and wide, the center of mass is low to the ground, and it is difficult for the car to flip over onto its side. 

### Batteries
Measuring the battery life and time to full charge provides valuable information on how long we can drive the car and how long we have to wait for it to get to full charge again.

| Trial  | Time |
| ------------- | ------------- |
| Battery Life | 8:11 (when only turning in circles) |
| Time to full charge | a really long time, like overnight |

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

### Surfaces
After experimenting on the tiled floors of Phillips Hall, I determined that the car can drive very well on smooth, flat surfaces. Next, I brought the car home and tested it out on an uneven, gravelly surface.

(Insert video)

Analysis

Then, I tested it out on uneven ground with grass, as well as snow.

(Insert video)

Analysis

### Turning around its own axis

### How reliably can we control the car with manual control?
Not well.

#### Stunts
It can kind of do a wheelie for half a second.


### [Click here to return to homepage](https://lyl24.github.io/lyl24-ece4960)
