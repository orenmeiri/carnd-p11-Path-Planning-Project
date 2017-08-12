
# Reflection on how to generate paths.

The following describes methods and techniques used for the path planning exercise:

## Maintain speed
In order to observe the speed limit (50 mph), I set a hard upper limit of (49.5 mph).  At any given time the _ref___vel_ is maintained with aim to reach this goal as long as no obstacles exist ahead of us.

If obstable is observed ahead in our current lane and lane change is not possible, the target _ref__vel_ is reduced gradually until the obstacle is far enough ahead of us that we can try speeding up again.

When generating x,y points for the planned trajectory, the _ref___vel_ is used to calculate distance of points along the trajectory line.

## Max acceleration and jerk

In order to avoid max acceleration and jerk, changes to _ref___vel_ are done gradually.  This includes starting from stationary state (speed is 0) as well as for slowing down to avoid hitting obstacle head of us in our lane and speeding up after obstacle is passed.

At each step (0.02 seconds) _ref___vel_ is changed a max of 0.112 meters per second.  This is well below acceleration limit of 10 m/s^2.

## Stay in lane and follow the road

Most of the time, the ego car follows current lane and curvature of the road.  This is done by following steps:

* prepare 5 points for fiting smooth line through them with spline library.  These 5 points are in x,y map coodinates system.  When needed points are converted from x,y car coordinate system and s,d coordinate system.
* first 2 points are current car trajectory.  This can be actual current ego car position and 1 extrapolated previous position based on current car heading OR last 2 points in already planned trajectory.
* 3 more points are calculated ahead of us in the current ego car lane.
* When not switching lane, these 5 points should be in middle of current ego car lane and all points along the spline line should smoothly follow the road curvature.
* Trajectory points to send to simulator are calculated from _ref___vel_ at 0.02 seconds intervals on the spline line.

### choose a lane

In order to avoid collisions and perform safe lane switches, sensor_fusion data is analysed.  I evaulate each lane by cost by parsing all other cars on the road around us, adding 1 per obstacle detected the the lane cost.  Obstacles can be:

* car is ahead of us and near in our lane
* car is in other lane near us (close s value)
* car is in other lane, behind us in s but faster than us (i.e. overtaking us).

Once evaulated, each adjacent lane is checked if it has lower cost.  If so, we perform lane change, preferring legal left lane overtaking.

If no better adjacent lane is found but we are getting close to car ahead, we slow down a little bit.

An added optimization: If no obstacles is in the middle lane (the preferred lane), we switch to it.  This is for getting ready to have more options (left and right) in future.


### switch lane

As described above lane costs are calculated and at times, lane switch is best course of action. In such a case, the future 3 pointes (for spline) are marked at future s positions but for the target lane (not current lane).  This produces target trajectory that smoothly leads to the target lane.
