
Check out the documentation and the datasheet for the dual motor driver.

Note that to deliver enough current for our robot to be fast, we will parallel couple the two inputs and outputs on each dual motor driver, essentially using two channels two drive each motor. This means that we can deliver twice the average current without overheating the chip. While it is a bad idea to parallel couple motor drivers from separate ICs because their timing might differ slightly, you can often do it when both motor drivers are onboard the same chip with the same clock circuitry.

Discuss/show how you intend to hook up the motor drivers.

What pins will you use for control on the Artemis? (It is worth considering both pin functionality and physical placement on the board/car).
We recommend powering the Artemis and the motor drivers/motors from separate batteries. Why is that?
Consider routing paths given EMI, wire lengths, and color coding. Long wires may not fit in the chassis, and lead to unnecessary noise. Wires that are too short, will make repair harder. Using solid-core wire can cause problems when the car undergoes high accelerations.
