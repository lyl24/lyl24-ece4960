# Lab 11: Grid Localization using Bayes Filter

## Objective: For this lab, the goal is to implement grid localization using Bayes Filter! Robot localization allows the robot to determine where it is located with respect to its environment, and in the previous lab, we found that using non-probabilistic methods leads to terrible results.

In this lab, we are using the same simulator program from Lab 10. The result from the Bayes filter will be displayed on the trajectory plotter alongside the odometry readings and ground truth. When implemented well, the Bayes filter trajectory should follow the ground truth trajectory closely. (Explain Bayes more here)

## Code
Before running the code, I first imported the ```numpy``` and ```math``` modules.

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

The ```arctan2``` function in the numpy library outputs a value in radians in the -pi to pi range. When subtracting angles, I used the ```normalize_angle``` function to convert all angles to a range between -180 and 180 degrees. 

### odom_motion_model
The odomotry motion model function takes the current pose, previous pose, and control input as its arguments, and it returns the probability that the robot moves to a certain pose given the current pose and control input.

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

(Discuss Gaussian)

### [Click here to return to homepage](https://lyl24.github.io/lyl24-ece4960)
