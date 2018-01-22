## CarND-Path-Planning-Project

**Path Planning Project README**

The goal of this project is to design a path planner that is able to create smooth, safe paths for the car to follow along a 3 lane highway with traffic. A successful path planner will be able to keep inside its lane, avoid hitting other cars, and pass slower moving traffic all by using localization, sensor fusion, and map data.

#### How to run

1. Clone the project at https://github.com/bertciccone/CarND-Path-Planning-Project.git

2. cd <project directory>/Debug/

3. ./path_planning

4. Click "Allow" in the pop-up window

5. Download and run the [Udacity/Unity Term 3 Simulator](https://github.com/udacity/self-driving-car-sim/releases/tag/T3_v1.2)
6. Click "Select"

#### [Rubric Points](https://review.udacity.com/#!/rubrics/1020/view)

Here I will consider the rubric points individually and describe how I addressed each point in my implementation.

### The car is able to drive at least 4.32 miles without incident.

Highway incidents in this simulation include:
- Exceeding the speed limit
- Driving too slowly
- Driving outside a lane
- Collision with another car
- Excessive jerk in forward or lateral direction

In the final run of the project as captured in the ![project video](./video/xxxxxxxxxxx), the car drives over 5 minutes without incident.

### The car drives according to the speed limit.

The path planner checks the vehicle velocity every 20 ms and never increments it to be above 49.5 mph.

`const double target_vel = 49.5;`

### Max Acceleration and Jerk are not Exceeded.

To prevent "jerk," or sudden changes in velocity, two methods are used:

- speed changes are made gradually by increasing a fraction of the desired change every 20 ms, for example:

`double delta_vel = 0.336;
...
if (ref_vel < target_vel) { // gradually reach driving speed
  ref_vel += delta_vel;`

- tragectories for lane changes prevent sudden turns by using the spline.h library to create a smooth path for each lane change from five widely-spaced waypoints

`         tk::spline s;
          // Set x, y points to the spline
          s.set_points(ptsx, ptsy);
`

### Car does not have collisions.

The car avoids collisions using a couple of techniques:

- a safety zone of 25 meters is defined for distance checks in front of and behind the vehicle:

`const double s_safety_zone = 25;
`
- vehicle speed is decreased/increased when the vehicle exceeds the safety zone of the vehicle in front/behind (as measured by s coordinate distance), for example:

`if (check_car_s > car_s // other car is ahead of us
    && check_car_s - car_s < s_safety_zone) { // in safety zone
`

- before making lane changes, the code tests for the safety zones to be clear ahead of and behind other vehicles, for example:

`else if (lane > 0 && in_lane(lane - 1, d)) {
    // other car is on our left and between front/rear safety zones
    if (check_car_s > car_s - s_safety_zone
        && check_car_s < car_s + s_safety_zone) {
`

### The car stays in its lane, except for the time between changing lanes.

The vehicle is able to stay in its lane through the use of map waypoints and Spline interpolation to create the path that the vehicle needs to follow (see Reflection). In the final run, the vehicle stayed in its lane over 5 minutes, except for quick lane changes.

### The car is able to change lanes.

When a slower car is ahead of the vehicle, possible actions include (in order of preference):
- lane change to the left, if not blocked
- lane change to the right, if not blocked
- reduce speed

These actions are implemented after receiving the location of other cars from sensor fusion data and checking the 25 meter safety zones, for example:

`if (too_close_front) {
    if (lane > 0 && !blocked_left) {
      lane -= 1;
    }
`

### Reflection

This is a reflection on how paths are generated.

The car's path is extended to a length of 50 steps every 0.02 seconds and is created with these basic steps:

- Identify five widely-spaced waypoints from the road map that will be used to determine the path.
- Two of these are selected using either the direction of the vehicle at its starting point or the last two points from the previous path segment.
- Three more points are added from the road map (adjusted for the correct lane), which correspond to the shape of the current lane or a lane change.

- Use Spline polynomial interpolation to create a smooth line intersecting all of the waypoints. This calculation is performed by the function in spline.h.

- The function in spline.h is used to calculate points along the line separated by the distance traveled by the vehicle in 0.02 seconds at the desired velocity.

### Project Video

Download the following video to see a 5-minute run of the Udacity/Unity Term 3 Simulator running with the project path planner code.

![Project Video](./video/xxxxxxxxxxx)
