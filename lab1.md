# Lab 1: The Artemis board

## Objective: Become familiar with the Arduino IDE and Artemis board

## Part 1: Download and Install the Arduino IDE
For this step, I already had the Arduino IDE installed on my laptop. 

## Part 2: Hook the Artemis board up to a computer
Next, I installed the SparkFun Apollo3 Arduino Core.

## Part 3: Blink it Up!
This example code allows the LED on the Artemis board to blink on and off. To load the blink example onto the board, open the Arduino IDE and click on **File->Examples->01.Basics->Blink.** Then, click the upload button and watch the LED blink!

<iframe width="560" height="315" src="https://www.youtube.com/embed/8Zb-Iq6CxyQ" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

As seen in the video, the blue LED on the board turns on for one second, turns off for one second, and repeats this loop endlessly.

## Part 4: Serial
To load this example, click on **File->Examples->Apollo3->Example04_Serial** and then upload to the board. This example demonstrates serial communication, which allows for communication between the computer and the Artemis board.

<iframe width="560" height="315" src="https://www.youtube.com/embed/oDvsFhYsKt0" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

When running the code with the Serial Monitor, we can see that it first counts from 0 up to 9, then allows for users to input characters and "echoes" it back to you.

## Part 5: Analog Read
To load this example, click on **File->Examples->Apollo3->Example02_AnalogRead** and then upload to the board. The Artemis board includes an analog to digital convert (ADC), and this example shows us how to use the analogRead function to read the input on one of the analog pins. Using ADC, we can measure differential pairs, the internal die temperature, the internal VCC voltage, and the internal VSS voltage. The brightness of the built-in LED also depends on the voltage read in the analog pins. For this section, we will use it to measure the temperature of the chip.

<iframe width="560" height="315" src="https://www.youtube.com/embed/WbexvW_9EFU" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

When opening the Serial Monitor, we can see that for every millisecond (displayed on the far right), the program computes the following: the raw ADC counts from die temperature, the VCC across a 1/3 voltage divider, and the VSS. We are interested in the first entry, as it can tell us about temperature changes. When I placed my hand on the chip, this value increased, and the built-in LED got brighter. As soon as I removed my hand, this value decreased, and the built-in LED faded.
  
## Part 6: Microphone Output
To load this example, click on **File->Examples->PDM->Example1_MicrophoneOutput** and then upload to the board. This example demonstrates how to use the pulse density microphone (PDM) on the Artemis board, and it can be used to measure the loudest frequency in the board's surroundings.

<iframe width="560" height="315" src="https://www.youtube.com/embed/T8TzZe56tMs" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

After some initialization steps, the board then starts to read the loudest frequency. This value increases when I make a poor attempt to whistle and snap, and it returns to zero when I stop.

## Additional Task: Turn on the LED when whistling
For this task, the goal is for the built-in LED to blink on when the board detects a sound (such as whistling or snapping), and off otherwise. To tackle this problem, my idea was to use the example code from Part 3 (Blink it up!) and Part 6 (Microphone Output) and combine the two existing programs.

To start off, I took the example code for Part 6 (Microphone Output) and modified the setup loop to include initialization of the built-in LED.

```
void setup()
{
  Serial.begin(115200);
  
  // initialize digital pin LED_BUILTIN as an output.
  pinMode(LED_BUILTIN, OUTPUT);

  if (myPDM.begin() == false) // Turn on PDM with default settings, start interrupts
  {
    Serial.println("PDM Init failed. Are you sure these pins are PDM capable?");
    while (1)
      ;
  }
  Serial.println("PDM Initialized");

  printPDMConfig();
}
```

With the default code, the board only prints out the loudest frequency. I want it to listen for sounds, and if the measured frequency is above a certain threshold, the LED turns on. If the measured frequency is below this threshold (or zero), the LED turns off. In order to figure out the frequency, I had to modify the printLoudest function so that it could return the frequency, rather than just print it out. To do this, I changed the first line of the function:

```
void printLoudest(void) //Original code

int printLoudest(void) //Modified code
```

Then, I added another line to the end of the function to return the loudest frequency:

```
return ui32LoudestFrequency;
```

After modifying the printLoudest function, I made changes to the main loop. I was able to obtain the current frequency readings using printLoudest, and with these readings, I used conditional statements to determine whether or not a sound was detected in the environment. The sensitivity of the microphone can be adjusted by changing the threshold in the if statement, and after playing with the program for a bit, I determined that 100 was a decent threshold for detection of whistling.

```
void loop()
{
  if (myPDM.available())
  {
    myPDM.getData(pdmDataBuffer, pdmDataBufferSize);

    int frequency = printLoudest();

    if (frequency > 100)
    {
      digitalWrite(LED_BUILTIN, HIGH);
    }
    else
    {
      digitalWrite(LED_BUILTIN, LOW);
    }
  }

  // Go to Deep Sleep until the PDM ISR or other ISR wakes us.
  am_hal_sysctrl_sleep(AM_HAL_SYSCTRL_SLEEP_DEEP);
}
```

<iframe width="560" height="315" src="https://www.youtube.com/embed/_VokzF3xRf0" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

### [Click here to return to homepage](https://lyl24.github.io/lyl24-ece4960)
