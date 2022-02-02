# Lab 2: Bluetooth

## Objective: Establish communication between my computer and the Artemis board through the bluetooth stack

## Setup:
To set up for this lab, I first updated Python 3 and pip so that I had the latest releases on my device (Python >= 3.9 and pip >= 21.0). I then created a virtual environment for ECE 4960 and installed the necessary Python and Artemis packages for this course. Fortunately, this all went smoothly, and I was able to open the Lab 2 codebase in Jupyter successfully. Next, I burned the sketch ```ble_arduino.ino``` into the Artemis board from the codebase, and the board output the following in the serial monitor:
```Advertising BLE with MAC: c0:7:f1:98:8a:44```.

I updated ```artemis_address``` in ```connection.yaml``` (Python codebase) to match this MAC address, allowing for a communication channel to be established between my computer and the Artemis board through BLE.

## Task 1: ECHO
For this task, I sent an ECHO command with a string value from the computer to the Artemis board and received an augmented string on the computer. To do this, I wrote the following code with explanations in the ```ble_arduino.ino``` sketch:

```
case ECHO:

    char char_arr[MAX_MSG_SIZE];

    // Extract the next value from the command string as a character array
    success = robot_cmd.get_next_value(char_arr);
    if (!success)
        return;

    // Clear the contents of the EString before using it
    tx_estring_value.clear();
    // Append the string literal "Robot says -> "
    tx_estring_value.append("Robot says -> ");
    // Append the value stored in char_arr, extracted from the command string above
    tx_estring_value.append(char_arr);
    // Append the string literal " :)"
    tx_estring_value.append(" :)");
    // Update the TX String characteristic with the appended string
    tx_characteristic_string.writeValue(tx_estring_value.c_str());

    Serial.println(tx_estring_value.c_str());

    break;
```

After burning the updated sketch to the board, I called the following commands in Jupyter notebook as follows:
![Part 1 Jupyter](images/part 1 jupyter.PNG)

Using the ECHO command, we receive the original string with the prefix "Robot says -> " and the postfix " :)". This output can also be viewed in the Arduion serial monitor:
![Part 1 Output](images/part 1 output.PNG)

## Part 2: Hook the Artemis board up to a computer

## Part 3: Blink it Up!

## Part 4: Serial

## Additional Task 1: 

## Additional Task 2: 

### [Click here to return to homepage](https://lyl24.github.io/lyl24-ece4960)
