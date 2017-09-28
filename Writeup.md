# The Goal of this Project
 The goal is to design a path planner that is able to create smooth, safe paths for the car to follow along a 3 lane highway with traffic. A successful path planner will be able to keep inside its lane, avoid hitting other cars, and pass slower moving traffic all by using localization, sensor fusion, and map data.

# Model Documentation
Before wee get started I have to give a huge thanks to David Silver and Aron brown for the the extraordinary walkthrough that they created.

## Acceleration
The jerk induced due to sudden acceleration is fixed by checking the current speed and adding increments of 0.244 to the `ref_vel` until riched the desired speed of 49.5. In case that wee decide that wee have to slow down wee just have to subtract the same quantity of increments from the `ref_vel`.

```
else if (ref_vel < 49.5){
  ref_vel += 0.224;
}
```
## Changing lanes and taking corners
The jerk induced to to the sudden acceleration induced by taking corners or changing the direction with the intent of changing lanes is fixed by using the `splin.h` header. Wee create a spline of points from the current car position until our next desired destination point in the future
- wee decide on the lane that wee intend to be at the end of the spline
- wee read the last 2 point that the car has bean
  - when speed is 0 just get the current position and the immediate next point in the direction the car is heading
- wee decide in the distance where wee want our car to be at and at on a specific lane
- wee take that point and wee adding it to our points
- wee create a spline from the previous points and the last point
- wee select from the spline the number of missing points from our previous path and wee add them to our current path.
Using this strategy our car can keep the same lane or change lanes without creating jerk. ('main.cpp' from line 340 to 441 ). What I like the most about this approach is that wee only care about on what lane wee wild like to be and the algorithm adds smooth point to the current path in order to get there.

## Behavior planning
Wee use the sensor fusion jason object in order to make calculation related to where are the other cars. Wee basically care only of the following 3 things.
- `too_close` If there is a car in front of us that is too close
- `left_allowed` If the left lane exists and is a valid lane and also there is a gap in the lane with sufficient space that wee can change into
- `right_allowed` The same as  with the left lane but this time wee compute the same state values for the right lane
Wee compute this values in main.cpp from line 258 to 323

The decision to change the lane is simple. If no car in front just keep the lane. If car in front check the left lane and change if ok. If left not ok then check right and change if ok. If left and right not ok then just slow down a bit (until things clear out)
```
if (too_close){
  if(left_allowed){
    lane -= 1;
  }else if (right_allowed){
    lane += 1;
  } else{
    ref_vel -= 0.244;
  }
}else if (ref_vel < 49.5){
  ref_vel += 0.224;
}
```

Using this very simple solution wee change lanes successfully only when it improves our speed and wee also behave in the same time as a very responsible driver.
