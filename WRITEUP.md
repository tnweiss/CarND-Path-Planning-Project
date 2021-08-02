# Project Writeup

## Compilation

### The code compiles correctly

Run the following to build the code 

```bash
mkdir build && cd build
cmake ..
make
```

## Valid Trajectories

### The car is able to drive at least 4.32 miles without incident..

In my test run the car was able to make it 4.32 miles without any incidend

### The car drives according to the speed limit.

The car will hover around 49.5 miles per hour. I use `ref_vel` to track the current speed and adjust 
it based on whether or not i am at the limit or if there's another car in front of me.

```cpp
...
  else if (too_close) {
    ref_vel -= .224; // slow down for this fool
  } else if (ref_vel < 49.5) {
    ref_vel += .224;
  }
```

### Max Acceleration and Jerk are not Exceeded.

The car will accelerate / decellerate at .224 mph^2 as seen in the code snippet above. 

### Car does not have collisions.

The car will slow down or change lanes in order to avoid collisions. The code below detects whether or not a 
car is in front of it or if it is safe to change lanes. These placeholders are determined by what lane the car is in, and how far
away the car is away from cars in adjacent lanes

```cpp
          if (too_close && left_shift_avail) {
            lane -= 1; // pass this fool on the left
            shift_pause_count = 10; //wait 10 steps before shifting again
          } else if (too_close && right_shift_avail) {
            lane += 1; // pass this fool on the right
            shift_pause_count = 10; //wait 10 steps before shifting again
          } else if (too_close) {
            ref_vel -= .224; // slow down for this fool
          } else if (ref_vel < 49.5) {
            ref_vel += .224;
          }
```

### The car stays in its lane, except for the time between changing lanes.

The car will appropriately change lanes in a timely manner. It does this by declaring two variables that 
hold possible outcomes. The pause is used to keep the car in a lane for some amount of time before making 
another lane change. This prevents us from exceeding our jerk.

```cpp
          bool left_shift_avail = true;
          bool right_shift_avail = true;
          int shift_pause_count = 0;
```

In this section we try to see if there are any cars that can make a lane change to the left impossible (same kind of thing for the right).

```cpp
            // if shifting left is still possible then see if this car changes that
            else if (left_shift_avail && d < (2 + 4 * (lane - 1) + 2) && d > ( 2 + 4 * (lane - 1) - 2)) {
              double vx = sensor_fusion[i][3];
              double vy = sensor_fusion[i][4];
              double check_speed = sqrt( vx*vx + vy*vy );
              double check_car_s = sensor_fusion[i][5];
              
              check_car_s += ((double)prev_size * .02* check_speed);
              
              if ((abs(check_car_s - car_s) < 30)) {
                left_shift_avail = false;
              }
              
            }

```

### The car is able to change lanes

The car is able to change lanes if the appropriate criteria is met.

## Reflection

### There is a reflection on how to generate paths.

The provided Q&A tutorial helped me with behavior and trajectory planning which provided a foundation for me to create my own prediction logic. 

On Line `105` I start by declaring variables that help me decide what i can do. On `110` through `119` I eliminate some actions based simply on what lane i'm in
and whether or not i have switched lanes recently. On line `121` through `164` I loop through all of the vehicles my sensors picked up and use their relative distance
and lane position to rule out any lane changing or acceleration. On line `166` through `177` I decide which action i want to take based on what I have determined about
my speed and surroundings.

On Line `185` is when i start planning out my trajectory. On line `193` through `213` I try to get two previous points to help create my spline. I then calculate where i want to 
be 30, 60, and 90 feet out to use in my fit (`216` through `226`). I then convert the coordinate back to relative coordinates. After that i create my spline, feed it my previous points + 
the future points to get a fit. I then use the fit to add on to my previous points so my projection will always be at 50 points out. This smoothes any transitions i want to make and is a 
small enough number that allows me to react to my surroundings.


