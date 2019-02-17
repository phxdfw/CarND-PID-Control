# PID Controller Project

## Components
In this project, I used two PID controllers that are working independently but also related to each other at the target values (will explain below), one for steering and one for throttle.

#### Steering controller
This PID controller is very simliar to the example in the video course. There are `P`, `I`, `D` components (see `pid.cpp`). `P` is used to correct proportionally to the error. The target is set as steering =0, and the error summation `I` could help to overcome systematic error in long term. The differentially part `D` can effectively reduce the oscillation and stabilize the control. It is also worthy to note that `PID` controller does not converge as fast as `PD` controller in the short term because of the `I` part.

Also, the final `steering_value` is limited in the range of `[-1, 1]` (line 73-74 in `main.cpp`) as stated in the project comments.

#### Throttle controller
This PID controller is to control the speed of the car. Although we can technically set the target speed at a high value (because the speed limit is 100mph), but we should also take car's status into account, i.e. the target speed should be lower when turning for safety, and be higher when driving in a straight line. Therefore I made the target speed a variable depending on `steering_value`, instead of a constant.

After some attempts and experiments, my model is to split the speed target into a base part of 10 and a variable part up to 90 (see line 84 of `main.cpp`). The variable speed is discounted by a cubic method: `(1 - fabs(steering_value)) ^ 3`. Therefore, if the steering is at its maximum absolute value `1`, the variable part becomes `0` and the controller is targeted to a base speed of 10mph; if the steering is `0`, the variable part becomes `90`, making the controller aiming at a full speed of 10 + 90 = 100mph. With the PID controlling, the car runs towards the target speed depending on its steering status. The reason why I choose a cubic function is: I want to penalize much more when `fabs(steering_value)` grows larger, in order to limit the target speed when the car is making turns. After some testings, I found the cubic function provides me a working solution.

## Hyperparameters (line 38-39 in `main.cpp`)
For the steering controller, `P` is important to correct the direction, `D` is important to kill the oscillation, and `I` is also important to balance any accumulation of systematic errors. In `pid.cpp`, because `Kd` runs on the difference bewteen two errors(which could be small), `Kd`’s value should be relatively large; `Ki` runs on a summation of all errors, `Ki`’s value should be relatively small. I first set them as `Kd=3, Kp=0.3, and Ki=0.003`, and then manually tuned to `Kd=3, Kp=0.15, and Ki=0.002` after simulation runs.

For the throttle controller, `P` and `D` are also very important to help the speed to quickly converge, however, `I` might not be very important here. For example, if our target speed is 40 and it actually converges at 45 because of the systematic error of throttle, it is still acceptable and not end of the world. Another consideration is the acceleration/de-acceleration value should not be too high, so the `Kp` should not be a too large value here. Therefore I set their values to `Kd=3, Kp=0.03, and Ki=0.0003` and ended as `Kd=3, Kp=0.04, and Ki=0.0001` after some manual tunings.

## Question/Bug report
I tried to record some simulation videos to show the car running well in the track, but neither the recording button nor hotkey worked for me in Unity on the VNC box. I attempted multiple times and finally gave up. Maybe this is some bug in this project’s simulation setting?