# Lab 11: Grid Localization using Bayes Filter

## Objective: The goal is to implement grid localization using Bayes Filter! Robot localization allows the robot to determine where it is located with respect to its environment, and in the previous lab, we found that using non-probabilistic methods lead to terrible results.

In this lab, we are using the same simulator program from Lab 10. The result from the Bayes filter will be displayed on the trajectory plotter alongside the odometry readings and ground truth. When implemented well, the Bayes filter trajectory should follow the ground truth trajectory closely. 

As opposed to the non-probabilistic methods from Lab 10, a Bayes filter is a probabilistic approach for estimating the location of the robot, and it does so recursively over time using a mathematical/statistical model paired with incoming measurements of the robot's environment. The general method for a Bayes filter is as follows:

![prediction step](images/lab11/full algorithm.PNG)

## Code
Before running the code below, I first imported the ```numpy``` and ```math``` modules.

The Bayes filter code consists of five functions: ```compute_control```, ```odom_motion_model```, ```prediction_step```, ```sensor_model```, and ```update_step```.

### compute_control
This function extracts the control information for the robot based on the odometry motion model. It takes the current and previous odometry poses as inputs and returns information for the first rotation, translation, and second rotation needed to get the robot from the previous to the current pose.

```python
def compute_control(cur_pose, prev_pose):
    """
    Args:
        cur_pose  ([Pose]): Current Pose
        prev_pose ([Pose]): Previous Pose 

    Returns:
        [delta_rot_1]: Rotation 1  (degrees)
        [delta_trans]: Translation (meters)
        [delta_rot_2]: Rotation 2  (degrees)
    """
    
    cur_x, cur_y, cur_theta = cur_pose
    prev_x, prev_y, prev_theta = prev_pose
    
    degrees = np.degrees(np.arctan2(cur_y - prev_y, cur_x - prev_x))
    delta_rot_1 = loc.mapper.normalize_angle(degrees - prev_theta)
    delta_trans = np.sqrt((cur_pose[0]-prev_pose[0])**2+(cur_pose[1]-prev_pose[1])**2)
    delta_rot_2 = loc.mapper.normalize_angle(cur_theta - prev_theta - delta_rot_1)
    
    return delta_rot_1, delta_trans, delta_rot_2
```

The ```arctan2``` function in the numpy library outputs a value in radians in the -pi to pi range, and this can be converted to degrees using the ```degrees``` function. When subtracting angles, I used the ```normalize_angle``` function to convert all angles to a range between -180 and 180 degrees. 

### odom_motion_model
The odometry motion model function takes the current pose, previous pose, and control input as its arguments, and it returns the probability that the robot moves to a certain pose given the current pose and control input.

```python
def odom_motion_model(cur_pose, prev_pose, u):
    """
    Args:
        cur_pose  ([Pose]): Current Pose
        prev_pose ([Pose]): Previous Pose
        (rot1, trans, rot2) (float, float, float): A tuple with control data in the format 
                                                   format (rot1, trans, rot2) with units (degrees, meters, degrees)

    Returns:
        prob [float]: Probability p(x'|x, u)
    """
    
    rot1, trans, rot2 = compute_control(cur_pose, prev_pose) #actual movement
    rot1_u, trans_u, rot2_u = u #inputted movement

    rot1_prob = loc.gaussian(rot1, rot1_u, loc.odom_rot_sigma)
    trans_prob = loc.gaussian(trans, trans_u, loc.odom_trans_sigma)
    rot2_prob = loc.gaussian(rot2, rot2_u, loc.odom_rot_sigma)
    prob = rot1_prob*trans_prob*rot2_prob

    return prob
```

The Gaussian distribution is used to model the measurement noise, and it is like a simplified version of the Beam model. In the code above, the ```gaussian``` function helps determine how “probable” the transition of the robot state from the previous pose to the current pose is given the actual control input ("true" mean) and the rotation/translation noise (standard deviation).

### prediction_step
For the prediction step of the Bayes filter, the probabilities stored in bel_bar are updated based on the belief from the previous time step and the odometry motion model.

```python
def prediction_step(cur_odom, prev_odom):
    """ 
    Args:
        cur_odom  ([Pose]): Current Pose
        prev_odom ([Pose]): Previous Pose
    """
    
    u = compute_control(cur_odom, prev_odom)
    for x_prev in range(MAX_CELLS_X):
        for y_prev in range(MAX_CELLS_Y):
            for theta_prev in range(MAX_CELLS_A):
                if loc.bel[(x_prev, y_prev, theta_prev)] < 0.0001:
                    continue
                for x_cur in range(MAX_CELLS_X):
                    for y_cur in range(MAX_CELLS_Y):
                        for theta_cur in range(MAX_CELLS_A):
                            loc.bel_bar[(x_cur, y_cur, theta_cur)] += odom_motion_model(loc.mapper.from_map(x_cur, y_cur, theta_cur), loc.mapper.from_map(x_prev, y_prev, theta_prev), u)*loc.bel[(x_prev, y_prev, theta_prev)]
  
    loc.bel_bar = loc.bel_bar/np.sum(loc.bel_bar)
```

In order to run through every cell, I defined the following variables:

```python
MAX_CELLS_X = loc.mapper.MAX_CELLS_X
MAX_CELLS_Y = loc.mapper.MAX_CELLS_Y
MAX_CELLS_A = loc.mapper.MAX_CELLS_A
```

For all the previous values, if the belief is less than 0.0001, we can assume that the probability is basically zero and therefore ignore it. If the belief is greater than this value, the code then loops through all current values, and it updates bel_bar using the following equation:

![prediction step](images/lab11/prediction step.PNG)

Finally, since we assumed that all values lower than 0.0001 are simply 0, the probabilities across the grid might not perfectly add up to 1 anymore. In the last line of the prediction step, I included a step that normalizes the probabilities and fixes this issue.

### sensor_model
In the sensor model, the observations made in the rotation loop and the current pose are inputs, and the output is an array that stores the likelihood of each individual measurement (equivalent to p(z|x) ).

```python
def sensor_model(obs, cur_pose):
    """ 
    Args:
        obs ([ndarray]): A 1D array consisting of the measurements made in rotation loop

    Returns:
        [ndarray]: Returns a 1D array of size 18 (=loc.OBS_PER_CELL) with the likelihood of each individual measurements
    """
    
    prob_array = []
    for i in range(18):
        prob_value = loc.gaussian(obs[i], cur_pose[i], loc.sensor_sigma)
        prob_array.append(prob_value)
    return prob_array
```

This function uses the Gaussian function to determine the probability that we get a certain distance observation given the current position of the robot and sensor noise.

### update_step
In the final step, the probabilities stored in bel are updated based on bel_bar and the sensor model.

```python
def update_step():
    for x in range(0, MAX_CELLS_X):
        for y in range(0, MAX_CELLS_Y):
            for theta in range(0, MAX_CELLS_A):
                loc.bel[(x, y, theta)] = np.prod(sensor_model(loc.obs_range_data,mapper.get_views(x, y, theta)))*loc.bel_bar[(x, y, theta)] 

    loc.bel = loc.bel/np.sum(loc.bel) 
```

First, I ran through every cell, and for each cell, the belief is updated according to the following equation:

![update step](images/lab11/update step.PNG)

Next, I normalized the probabilities similar to what I did in the prediction step.

## Run the Bayes Filter
It's time to run the code and see how the Bayes filter holds up. I recorded videos of the trajectory plotter with the ground truth (green line), odometry readings (red line), and Bayes filter values (blue line). In the second video, the squares on the map represent the probability that the robot is in a certain grid location, with white being the most probable location.

<iframe width="560" height="315" src="https://www.youtube.com/embed/beD7GwoiV-Y" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

<iframe width="560" height="315" src="https://www.youtube.com/embed/xo_qTzhsBpI" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

Below, I have included the most probable state after each iteration of the Bayes filter as well as its probability. For the most part, the predicted state matches the ground truth pose quite closely. I noticed that the predicted trajectories sometimes end up being more "angular" than the actual trajectory, for example, there are sometimes oddly-straight trajectories as well as perfect 45/90 degree turns. In addition, there may be something off in the angle calculation code because the angular error keeps increasing. However, since I have the normalize_angle function, it doesn't affect the output too much.

```
----------------- 0 -----------------
2022-04-24 17:35:21,268 | INFO     |: ---------- PREDICTION STATS -----------
2022-04-24 17:35:21,272 | INFO     |: GT index         : (6, 3, 6)
2022-04-24 17:35:21,274 | INFO     |: Prior Bel index  : (4, 2, 6) with prob = 0.0903941
2022-04-24 17:35:21,276 | INFO     |: POS ERROR        : (0.596, 0.518, 9.607)
2022-04-24 17:35:21,278 | INFO     |: ---------- PREDICTION STATS -----------
2022-04-24 17:35:24,833 | INFO     |: ---------- UPDATE STATS -----------
2022-04-24 17:35:24,839 | INFO     |: GT index      : (6, 3, 6)
2022-04-24 17:35:24,841 | INFO     |: Bel index     : (6, 4, 6) with prob = 1.0
2022-04-24 17:35:24,843 | INFO     |: Bel_bar prob at index = 5.820437141223614e-05
2022-04-24 17:35:24,844 | INFO     |: GT            : (0.291, -0.092, 319.607)
2022-04-24 17:35:24,845 | INFO     |: Belief        : (0.305, 0.000, -50.000)
2022-04-24 17:35:24,848 | INFO     |: POS ERROR     : (-0.014, -0.092, 369.607)
2022-04-24 17:35:24,850 | INFO     |: ---------- UPDATE STATS -----------
-------------------------------------


----------------- 1 -----------------
2022-04-24 17:35:26,960 | INFO     |: ---------- PREDICTION STATS -----------
2022-04-24 17:35:26,967 | INFO     |: GT index         : (7, 2, 5)
2022-04-24 17:35:26,968 | INFO     |: Prior Bel index  : (7, 3, 5) with prob = 0.1419913
2022-04-24 17:35:26,970 | INFO     |: POS ERROR        : (-0.097, -0.235, 366.306)
2022-04-24 17:35:26,971 | INFO     |: ---------- PREDICTION STATS -----------
2022-04-24 17:35:30,469 | INFO     |: ---------- UPDATE STATS -----------
2022-04-24 17:35:30,487 | INFO     |: GT index      : (7, 2, 5)
2022-04-24 17:35:30,489 | INFO     |: Bel index     : (6, 2, 5) with prob = 1.0
2022-04-24 17:35:30,490 | INFO     |: Bel_bar prob at index = 1.2812168001111827e-05
2022-04-24 17:35:30,493 | INFO     |: GT            : (0.513, -0.540, 656.306)
2022-04-24 17:35:30,494 | INFO     |: Belief        : (0.305, -0.610, -70.000)
2022-04-24 17:35:30,496 | INFO     |: POS ERROR     : (0.208, 0.070, 726.306)
2022-04-24 17:35:30,498 | INFO     |: ---------- UPDATE STATS -----------
-------------------------------------


----------------- 2 -----------------
2022-04-24 17:35:31,596 | INFO     |: ---------- PREDICTION STATS -----------
2022-04-24 17:35:31,612 | INFO     |: GT index         : (7, 2, 4)
2022-04-24 17:35:31,613 | INFO     |: Prior Bel index  : (6, 2, 4) with prob = 0.0884460
2022-04-24 17:35:31,615 | INFO     |: POS ERROR        : (0.208, 0.070, 723.386)
2022-04-24 17:35:31,618 | INFO     |: ---------- PREDICTION STATS -----------
2022-04-24 17:35:35,144 | INFO     |: ---------- UPDATE STATS -----------
2022-04-24 17:35:35,152 | INFO     |: GT index      : (7, 2, 4)
2022-04-24 17:35:35,154 | INFO     |: Bel index     : (7, 2, 4) with prob = 0.9999999
2022-04-24 17:35:35,157 | INFO     |: Bel_bar prob at index = 0.08194671476545762
2022-04-24 17:35:35,159 | INFO     |: GT            : (0.513, -0.540, 993.386)
2022-04-24 17:35:35,162 | INFO     |: Belief        : (0.610, -0.610, -90.000)
2022-04-24 17:35:35,165 | INFO     |: POS ERROR     : (-0.097, 0.070, 1083.386)
2022-04-24 17:35:35,168 | INFO     |: ---------- UPDATE STATS -----------
-------------------------------------


----------------- 3 -----------------
2022-04-24 17:35:36,289 | INFO     |: ---------- PREDICTION STATS -----------
2022-04-24 17:35:36,304 | INFO     |: GT index         : (7, 0, 4)
2022-04-24 17:35:36,305 | INFO     |: Prior Bel index  : (8, 1, 4) with prob = 0.1250435
2022-04-24 17:35:36,307 | INFO     |: POS ERROR        : (-0.378, -0.031, 1083.386)
2022-04-24 17:35:36,309 | INFO     |: ---------- PREDICTION STATS -----------
2022-04-24 17:35:39,888 | INFO     |: ---------- UPDATE STATS -----------
2022-04-24 17:35:39,906 | INFO     |: GT index      : (7, 0, 4)
2022-04-24 17:35:39,907 | INFO     |: Bel index     : (8, 1, 5) with prob = 0.9996129
2022-04-24 17:35:39,909 | INFO     |: Bel_bar prob at index = 0.08720404625441203
2022-04-24 17:35:39,911 | INFO     |: GT            : (0.537, -0.946, 1353.386)
2022-04-24 17:35:39,913 | INFO     |: Belief        : (0.914, -0.914, -70.000)
2022-04-24 17:35:39,914 | INFO     |: POS ERROR     : (-0.378, -0.031, 1423.386)
2022-04-24 17:35:39,917 | INFO     |: ---------- UPDATE STATS -----------
-------------------------------------


----------------- 4 -----------------
2022-04-24 17:35:43,098 | INFO     |: ---------- PREDICTION STATS -----------
2022-04-24 17:35:43,103 | INFO     |: GT index         : (8, 0, 8)
2022-04-24 17:35:43,105 | INFO     |: Prior Bel index  : (6, 0, 7) with prob = 0.1170404
2022-04-24 17:35:43,107 | INFO     |: POS ERROR        : (0.493, 0.127, 1469.806)
2022-04-24 17:35:43,110 | INFO     |: ---------- PREDICTION STATS -----------
2022-04-24 17:35:46,613 | INFO     |: ---------- UPDATE STATS -----------
2022-04-24 17:35:46,630 | INFO     |: GT index      : (8, 0, 8)
2022-04-24 17:35:46,631 | INFO     |: Bel index     : (8, 1, 9) with prob = 0.9999999
2022-04-24 17:35:46,633 | INFO     |: Bel_bar prob at index = 1.0570805414447486e-07
2022-04-24 17:35:46,635 | INFO     |: GT            : (0.798, -1.093, 1799.806)
2022-04-24 17:35:46,637 | INFO     |: Belief        : (0.914, -0.914, 10.000)
2022-04-24 17:35:46,639 | INFO     |: POS ERROR     : (-0.116, -0.178, 1789.806)
2022-04-24 17:35:46,641 | INFO     |: ---------- UPDATE STATS -----------
-------------------------------------


----------------- 5 -----------------
2022-04-24 17:35:52,767 | INFO     |: ---------- PREDICTION STATS -----------
2022-04-24 17:35:52,777 | INFO     |: GT index         : (11, 0, 11)
2022-04-24 17:35:52,778 | INFO     |: Prior Bel index  : (11, 1, 10) with prob = 0.0934682
2022-04-24 17:35:52,779 | INFO     |: POS ERROR        : (-0.247, -0.019, 1819.595)
2022-04-24 17:35:52,782 | INFO     |: ---------- PREDICTION STATS -----------
2022-04-24 17:35:56,222 | INFO     |: ---------- UPDATE STATS -----------
2022-04-24 17:35:56,227 | INFO     |: GT index      : (11, 0, 11)
2022-04-24 17:35:56,229 | INFO     |: Bel index     : (10, 1, 11) with prob = 1.0
2022-04-24 17:35:56,230 | INFO     |: Bel_bar prob at index = 0.01298524398718259
2022-04-24 17:35:56,232 | INFO     |: GT            : (1.582, -0.934, 2209.595)
2022-04-24 17:35:56,234 | INFO     |: Belief        : (1.524, -0.914, 50.000)
2022-04-24 17:35:56,236 | INFO     |: POS ERROR     : (0.058, -0.019, 2159.595)
2022-04-24 17:35:56,238 | INFO     |: ---------- UPDATE STATS -----------
-------------------------------------


----------------- 6 -----------------
2022-04-24 17:35:58,338 | INFO     |: ---------- PREDICTION STATS -----------
2022-04-24 17:35:58,341 | INFO     |: GT index         : (11, 2, 12)
2022-04-24 17:35:58,342 | INFO     |: Prior Bel index  : (7, 0, 15) with prob = 0.1043242
2022-04-24 17:35:58,344 | INFO     |: POS ERROR        : (1.051, 0.678, 2108.726)
2022-04-24 17:35:58,346 | INFO     |: ---------- PREDICTION STATS -----------
2022-04-24 17:36:01,822 | INFO     |: ---------- UPDATE STATS -----------
2022-04-24 17:36:01,828 | INFO     |: GT index      : (11, 2, 12)
2022-04-24 17:36:01,830 | INFO     |: Bel index     : (10, 2, 12) with prob = 0.9999999
2022-04-24 17:36:01,832 | INFO     |: Bel_bar prob at index = 0.00033548181205257416
2022-04-24 17:36:01,834 | INFO     |: GT            : (1.660, -0.541, 2598.726)
2022-04-24 17:36:01,837 | INFO     |: Belief        : (1.524, -0.610, 70.000)
2022-04-24 17:36:01,839 | INFO     |: POS ERROR     : (0.136, 0.068, 2528.726)
2022-04-24 17:36:01,842 | INFO     |: ---------- UPDATE STATS -----------
-------------------------------------


----------------- 7 -----------------
2022-04-24 17:36:03,971 | INFO     |: ---------- PREDICTION STATS -----------
2022-04-24 17:36:03,981 | INFO     |: GT index         : (11, 3, 13)
2022-04-24 17:36:03,982 | INFO     |: Prior Bel index  : (10, 4, 13) with prob = 0.0944704
2022-04-24 17:36:03,983 | INFO     |: POS ERROR        : (0.208, -0.183, 2514.361)
2022-04-24 17:36:03,985 | INFO     |: ---------- PREDICTION STATS -----------
2022-04-24 17:36:07,491 | INFO     |: ---------- UPDATE STATS -----------
2022-04-24 17:36:07,497 | INFO     |: GT index      : (11, 3, 13)
2022-04-24 17:36:07,499 | INFO     |: Bel index     : (11, 3, 13) with prob = 1.0
2022-04-24 17:36:07,501 | INFO     |: Bel_bar prob at index = 8.833263705263988e-05
2022-04-24 17:36:07,502 | INFO     |: GT            : (1.732, -0.183, 2964.361)
2022-04-24 17:36:07,504 | INFO     |: Belief        : (1.829, -0.305, 90.000)
2022-04-24 17:36:07,505 | INFO     |: POS ERROR     : (-0.097, 0.122, 2874.361)
2022-04-24 17:36:07,507 | INFO     |: ---------- UPDATE STATS -----------
-------------------------------------


----------------- 8 -----------------
2022-04-24 17:36:10,647 | INFO     |: ---------- PREDICTION STATS -----------
2022-04-24 17:36:10,659 | INFO     |: GT index         : (11, 5, 14)
2022-04-24 17:36:10,660 | INFO     |: Prior Bel index  : (10, 4, 13) with prob = 0.0734940
2022-04-24 17:36:10,661 | INFO     |: POS ERROR        : (0.207, 0.326, 2897.571)
2022-04-24 17:36:10,664 | INFO     |: ---------- PREDICTION STATS -----------
2022-04-24 17:36:14,137 | INFO     |: ---------- UPDATE STATS -----------
2022-04-24 17:36:14,144 | INFO     |: GT index      : (11, 5, 14)
2022-04-24 17:36:14,146 | INFO     |: Bel index     : (11, 4, 13) with prob = 0.9999999
2022-04-24 17:36:14,148 | INFO     |: Bel_bar prob at index = 0.03597377334175566
2022-04-24 17:36:14,151 | INFO     |: GT            : (1.731, 0.326, 3347.571)
2022-04-24 17:36:14,153 | INFO     |: Belief        : (1.829, 0.000, 90.000)
2022-04-24 17:36:14,154 | INFO     |: POS ERROR     : (-0.098, 0.326, 3257.571)
2022-04-24 17:36:14,157 | INFO     |: ---------- UPDATE STATS -----------
-------------------------------------


----------------- 9 -----------------
2022-04-24 17:36:17,328 | INFO     |: ---------- PREDICTION STATS -----------
2022-04-24 17:36:17,334 | INFO     |: GT index         : (11, 6, 16)
2022-04-24 17:36:17,335 | INFO     |: Prior Bel index  : (11, 6, 15) with prob = 0.0920022
2022-04-24 17:36:17,336 | INFO     |: POS ERROR        : (-0.097, 0.046, 3258.055)
2022-04-24 17:36:17,339 | INFO     |: ---------- PREDICTION STATS -----------
2022-04-24 17:36:20,865 | INFO     |: ---------- UPDATE STATS -----------
2022-04-24 17:36:20,870 | INFO     |: GT index      : (11, 6, 16)
2022-04-24 17:36:20,871 | INFO     |: Bel index     : (11, 7, 16) with prob = 1.0
2022-04-24 17:36:20,873 | INFO     |: Bel_bar prob at index = 0.06560135006548269
2022-04-24 17:36:20,876 | INFO     |: GT            : (1.732, 0.656, 3748.055)
2022-04-24 17:36:20,878 | INFO     |: Belief        : (1.829, 0.914, 150.000)
2022-04-24 17:36:20,880 | INFO     |: POS ERROR     : (-0.097, -0.259, 3598.055)
2022-04-24 17:36:20,882 | INFO     |: ---------- UPDATE STATS -----------
-------------------------------------


----------------- 10 -----------------
2022-04-24 17:36:23,019 | INFO     |: ---------- PREDICTION STATS -----------
2022-04-24 17:36:23,035 | INFO     |: GT index         : (10, 7, 16)
2022-04-24 17:36:23,036 | INFO     |: Prior Bel index  : (9, 8, 17) with prob = 0.1204846
2022-04-24 17:36:23,038 | INFO     |: POS ERROR        : (0.089, -0.299, 3589.708)
2022-04-24 17:36:23,041 | INFO     |: ---------- PREDICTION STATS -----------
2022-04-24 17:36:26,494 | INFO     |: ---------- UPDATE STATS -----------
2022-04-24 17:36:26,498 | INFO     |: GT index      : (10, 7, 16)
2022-04-24 17:36:26,500 | INFO     |: Bel index     : (9, 7, 17) with prob = 1.0
2022-04-24 17:36:26,503 | INFO     |: Bel_bar prob at index = 0.002783559882728099
2022-04-24 17:36:26,504 | INFO     |: GT            : (1.308, 0.920, 4119.708)
2022-04-24 17:36:26,506 | INFO     |: Belief        : (1.219, 0.914, 170.000)
2022-04-24 17:36:26,508 | INFO     |: POS ERROR     : (0.089, 0.006, 3949.708)
2022-04-24 17:36:26,510 | INFO     |: ---------- UPDATE STATS -----------
-------------------------------------


----------------- 11 -----------------
2022-04-24 17:36:29,648 | INFO     |: ---------- PREDICTION STATS -----------
2022-04-24 17:36:29,659 | INFO     |: GT index         : (7, 6, 3)
2022-04-24 17:36:29,661 | INFO     |: Prior Bel index  : (6, 6, 3) with prob = 0.0686627
2022-04-24 17:36:29,662 | INFO     |: POS ERROR        : (0.098, 0.178, 4327.093)
2022-04-24 17:36:29,665 | INFO     |: ---------- PREDICTION STATS -----------
2022-04-24 17:36:33,158 | INFO     |: ---------- UPDATE STATS -----------
2022-04-24 17:36:33,176 | INFO     |: GT index      : (7, 6, 3)
2022-04-24 17:36:33,178 | INFO     |: Bel index     : (7, 7, 3) with prob = 0.9999999
2022-04-24 17:36:33,179 | INFO     |: Bel_bar prob at index = 0.011219027038181771
2022-04-24 17:36:33,181 | INFO     |: GT            : (0.403, 0.788, 4577.093)
2022-04-24 17:36:33,183 | INFO     |: Belief        : (0.610, 0.914, -110.000)
2022-04-24 17:36:33,185 | INFO     |: POS ERROR     : (-0.207, -0.127, 4687.093)
2022-04-24 17:36:33,187 | INFO     |: ---------- UPDATE STATS -----------
-------------------------------------


----------------- 12 -----------------
2022-04-24 17:36:35,295 | INFO     |: ---------- PREDICTION STATS -----------
2022-04-24 17:36:35,309 | INFO     |: GT index         : (6, 4, 6)
2022-04-24 17:36:35,310 | INFO     |: Prior Bel index  : (8, 5, 5) with prob = 0.1020943
2022-04-24 17:36:35,312 | INFO     |: POS ERROR        : (-0.659, -0.161, 4692.939)
2022-04-24 17:36:35,314 | INFO     |: ---------- PREDICTION STATS -----------
2022-04-24 17:36:38,835 | INFO     |: ---------- UPDATE STATS -----------
2022-04-24 17:36:38,848 | INFO     |: GT index      : (6, 4, 6)
2022-04-24 17:36:38,849 | INFO     |: Bel index     : (6, 4, 6) with prob = 0.9333164
2022-04-24 17:36:38,851 | INFO     |: Bel_bar prob at index = 0.0001252879602781938
2022-04-24 17:36:38,853 | INFO     |: GT            : (0.255, 0.143, 4982.939)
2022-04-24 17:36:38,855 | INFO     |: Belief        : (0.305, 0.000, -50.000)
2022-04-24 17:36:38,856 | INFO     |: POS ERROR     : (-0.050, 0.143, 5032.939)
2022-04-24 17:36:38,858 | INFO     |: ---------- UPDATE STATS -----------
-------------------------------------


----------------- 13 -----------------
2022-04-24 17:36:41,126 | INFO     |: ---------- PREDICTION STATS -----------
2022-04-24 17:36:41,145 | INFO     |: GT index         : (6, 3, 2)
2022-04-24 17:36:41,147 | INFO     |: Prior Bel index  : (8, 3, 4) with prob = 0.1001080
2022-04-24 17:36:41,149 | INFO     |: POS ERROR        : (-0.893, 0.124, 5004.196)
2022-04-24 17:36:41,152 | INFO     |: ---------- PREDICTION STATS -----------
2022-04-24 17:36:44,637 | INFO     |: ---------- UPDATE STATS -----------
2022-04-24 17:36:44,645 | INFO     |: GT index      : (6, 3, 2)
2022-04-24 17:36:44,647 | INFO     |: Bel index     : (5, 3, 2) with prob = 1.0
2022-04-24 17:36:44,648 | INFO     |: Bel_bar prob at index = 4.886228318376782e-06
2022-04-24 17:36:44,650 | INFO     |: GT            : (0.021, -0.181, 5274.196)
2022-04-24 17:36:44,651 | INFO     |: Belief        : (0.000, -0.305, -130.000)
2022-04-24 17:36:44,653 | INFO     |: POS ERROR     : (0.021, 0.124, 5404.196)
2022-04-24 17:36:44,655 | INFO     |: ---------- UPDATE STATS -----------
-------------------------------------


----------------- 14 -----------------
2022-04-24 17:36:47,789 | INFO     |: ---------- PREDICTION STATS -----------
2022-04-24 17:36:47,795 | INFO     |: GT index         : (4, 2, 1)
2022-04-24 17:36:47,797 | INFO     |: Prior Bel index  : (4, 3, 16) with prob = 0.1521482
2022-04-24 17:36:47,798 | INFO     |: POS ERROR        : (-0.035, -0.049, 5101.273)
2022-04-24 17:36:47,800 | INFO     |: ---------- PREDICTION STATS -----------
2022-04-24 17:36:51,279 | INFO     |: ---------- UPDATE STATS -----------
2022-04-24 17:36:51,293 | INFO     |: GT index      : (4, 2, 1)
2022-04-24 17:36:51,294 | INFO     |: Bel index     : (4, 3, 1) with prob = 1.0
2022-04-24 17:36:51,296 | INFO     |: Bel_bar prob at index = 0.00018029427748503477
2022-04-24 17:36:51,298 | INFO     |: GT            : (-0.340, -0.354, 5611.273)
2022-04-24 17:36:51,300 | INFO     |: Belief        : (-0.305, -0.305, -150.000)
2022-04-24 17:36:51,302 | INFO     |: POS ERROR     : (-0.035, -0.049, 5761.273)
2022-04-24 17:36:51,304 | INFO     |: ---------- UPDATE STATS -----------
-------------------------------------


----------------- 15 -----------------
2022-04-24 17:36:54,435 | INFO     |: ---------- PREDICTION STATS -----------
2022-04-24 17:36:54,441 | INFO     |: GT index         : (3, 2, 0)
2022-04-24 17:36:54,442 | INFO     |: Prior Bel index  : (2, 2, 0) with prob = 0.0960709
2022-04-24 17:36:54,443 | INFO     |: POS ERROR        : (0.168, 0.237, 5758.350)
2022-04-24 17:36:54,446 | INFO     |: ---------- PREDICTION STATS -----------
2022-04-24 17:36:57,947 | INFO     |: ---------- UPDATE STATS -----------
2022-04-24 17:36:57,961 | INFO     |: GT index      : (3, 2, 0)
2022-04-24 17:36:57,964 | INFO     |: Bel index     : (3, 3, 0) with prob = 0.6750508
2022-04-24 17:36:57,965 | INFO     |: Bel_bar prob at index = 0.0020947943443782586
2022-04-24 17:36:57,967 | INFO     |: GT            : (-0.746, -0.372, 5948.350)
2022-04-24 17:36:57,969 | INFO     |: Belief        : (-0.610, -0.305, -170.000)
2022-04-24 17:36:57,971 | INFO     |: POS ERROR     : (-0.136, -0.067, 6118.350)
2022-04-24 17:36:57,973 | INFO     |: ---------- UPDATE STATS -----------
-------------------------------------
```

For this lab, I worked with Syd Lawrence and Ryan Chan.

### [Click here to return to homepage](https://lyl24.github.io/lyl24-ece4960)
