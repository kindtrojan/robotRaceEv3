# TrojanHorse

A TrojanHorse to beat all other robots in OS race for glory.

# Main source code
 
The source code of the robot can be found in [this](https://gitlab.eurecom.fr/trojanhorse/trojanhorse/-/blob/master/recover_from_stuck.c) file.

# How to build and run

On the robot/build environment(can be cross compiler)

- do "git clone https://gitlab.eurecom.fr/trojanhorse/trojanhorse.git",

- do "make recover_from_stuck"
you will see a binary called "recover_from_stuck", 

- run it on the robot using "./recover_from_stuck"

## Description of the robot

Our robot is using two actuators (sonar and gyroscope), as well as three motors (two for the wheels and one to drop obstacles). We added a bumper to protect the robot from the hits and the obstacles, and a cup to throw our own obstacles.

**The sonar** is placed on the front, so that it can detect what's comming. It is protected by the bumper, so we can go and touch any wall without a problem. It is precise, but can react unexpectedly : with the same distance it can give differentes values if the object is wood or plastic, depending on the shape of the object... For exemple, the fixed obstacles and the corrner of the stadium (when comming straight) are showing the biggest value (bigger than the full lenght of the stadium). Who know how it will react to the obstacles from the others teams ?

**The gyroscope** is placed on the top, parallel to the ground. It can detect quite precisely the angular change while rotating, but it will slowly loose in accuracy as the truns keep going. It dosn't give the value modulo 360, but can gives extremely high or low values. There is a risk of overflow, that can be reseted by changing the mode.

**The wheels** and their's motors are placed under the robot, in the front. A third ball-shaped wheel is placed at the back to keep the balance. Each wheel can be directed precisely by turning the motor by an angular distance or can just be ask to turn until said to stop.

**The bumper** is composed of two parts : 

- The top part is made of two bars to hit the walls and protect the sonar. It also have the adventage of placing the robot in front of the wall that was hit.

- The sides parts are made of two black pieces that protects the wheels from any obstacle. It avoids the wheel touching the wall (that would make the robot turn in front of the wall), and any obstacle that could be a problem for the wheel. It's not perpendicular to the way, so it will push the obstacles away.

**The droping device** made of a motor that will start a gear and lead to the rotation of a cup, in witch we can place any obstacle (not to heavy). Unfortunatly, there was no cup that day at EURECOM.


We choose not to use **the compass** because even if it's values are quite constant (when the robot is oriented on a direction, the compass will allways give the same value), the angules arn't respected, so a 90° turn could lead to a 60° variation on the compass. For only two laps, there is no need to recalibrate that much. We used it on the first test to recalebrate to make the five laps.

### Pictures of our robot :

![Picture of the robot from side](./img/robot_side.jpeg)


![Picture of the robot from front](./img/robot_front.jpeg)

![Robot, thrashing its opponents](./img/robot_in_action.mp4)

## Algorithms :

### Go forward(go_forward)

This algorithm is meant to make sur that the robot allways goes forward without turning unknowlegly. On the begining of the function, the robot remembers the gyro value, and if if detects a turn of over 10°, it will slow down one wheel to go back to this position. Anyway, if the robot detect an obstacle (either a fixed plactic obstacle because sonar is over 2000, or a sonar less than 200) it will set the speed of the motors back to normal (in case it was recovering), stop, and go into the recover form stuck algorithm.

### Recover form stuck(recover_from_stuck)

This algorithm is meant to find the best path after finding a wall or an obstacle. It will "check" (see ```hit_and_check``` algorithm for more information) if the path is free. First, it will check if forward path is free, then if not, it will goes for a 90° turn left (trying to get in the next quarter of turn). If neither of those paths are free, it will tries to get into 45° turns, to avoid an obstacle. If the robot takes a 45° turn either left or right, it will go forward until it finds an obstacle and then go back straigh. This behaviours loose some time where the robot goes back in front and then makes a 90° turn left instead of just making a 45° turn, but it avoids a lot of going back situation. When the robot finds a free path, it will go back to the go forward algorithm.

### hit and check(hit_and_check)

This algorithm is meant to find out it what the sonar detected is a fixed obstacle, a wall or an obstacle placed by another team that we can move. To do so, the robot will go forward a little bit (try to push the obstacle if it is a ball or something we can move), and then get back the value of the sonar so verify if the obstacle been push away or if it's a fixed obstacle. It the sonar detect a plastic cone, the robot goes backard a little bit and trys to avoid it. This algorithm return a boolean to know if the path is free or not. It takes minimum sonar value as an input, that has to be checked for to figure out if there is enough path ahead of the robot.

### turn rel new(turn_rel_new)

This algorithm is used for making sure we turn correctly. It takes in an input gyro value, and the degree of rotation that is expected with respect to it, and the direction of the turn desired.
We change the tacho polarity to invert the rotation of the motor, on the side of rotation(one wheel goes forward, the other turns backward) to achieve turn in the same position. we start turning at maxspeed/4, then we keep reducing speed as we get closer to the desired angle to get an accurate angle.


## Algorithm changes/game play changes for different type of races:

Since our robot is pretty generic, it wouldn't need any algorithmic changes for races depending on the position where it starts, or if its starting right next to another robot.

However, in the "Race against time", we had to disable all the checks for obstacles, and make it take quick 90 degree left turns everytime it senses an obstacle(wood). Thus, we were able to cut down our race time from 52 seconds to 35 seconds, in just the second trial itself. we couldve done better if we had hurried up a bit and taken the third trial as well.

Race against time: //this is the best as, we dont need complex logic to slow us down, just turn blindly

```
while(1) {

go_forward() //with enough intelligence to maintain path, until an obstacle is sensed`

turn_90_degree_left() //Just turn 90 degree to left, very precisely
}
```

For other races: //We need complex logic to decide turns and avoid obstacles, and maintain/remember our path.

```
while(1) {
   go_forward()
   get_out_of_stuck() {
        if(robot_seems_to_be_deviated) {
            correct_robot_path()
        }
        if(on_cone) {
            go in reverse and reorient()
        }
        make_sure_obstacle_is_immovable()
        Turn_90/45 on left and check again()
        if(not enough space ahead) {
            turn 45 degrees on right and check again()
        } 

    }
}
```


## Evolution of the robot(generations of code and design changes):

### Gen1 : 
*Fully autonomous, no hardcoding, but, too slow, Lack of error correction.*

The Gen1 robot was designed to figure out the best possible way on its own. On every point of contention, it checks 45 and 90 degrees on either side of the robot to figure out the best possible way ahead(the max sonar value). If neither worked
Although this robot was totally autonamous and more intelligent, it takes a lot of time making decisions and would be at a disadvantage when its a race of time and with other parties involved in the fight. So, needed change.
We were using the wheel turns to relative sp, to achieve the approximate turn.

### Gen2 : 
*Use of compass for error correction after every 360 deg, use of mathematical formula to decide the turn angle, Use of gyro for error correction on deviation from straight line, introduction of hit_and_run.*
 
Gen2 was an advanced version of Gen1, with some changes. It has better overall physical design, with the front guards and deflectors.

Gen2 used a little bit of hardcoding to favour left side turns than the right side turns, this had a great performance improvement over Gen1. However, the turns were still a guess-estimate ones based on the wheel turn angles, and hence not reliable. Error correction was done only once every 360 degrees, hence it wasnt having a robust error correction in place, this needed a drastic change.

### Gen 3 : 
*Use of gyro for turns- Very accurate turns, better logic for get_out_of_stuck, improved and robust error correction.*

Gen3 is the final version of the robot presented in the class, it uses gyro values for accurate turns. ```get_out_of_stuck``` was improved with better decision making and angle tracking (we now track the number of total left turns taken since the initial start of race with counters), this helped in error correction on every turn.
Also, we enhanced error correction in most of the places to detect and rectify when ever the robot is deviating from planned track /if its in a stuck/bad place.

## Scope for improvement : 

As we kept improving the code logic and functionality very dynamically until the last day, even though we had an amazing logic and code at hand, we lacked time to test all corner cases.
We encountered corner cases on the day of test, while we have answers/counter logics to get out of that, we just lacked time, we could've done some private robot wars before the actual run to uncover these. Nevertheless, we believe our trojan horse is the best :) 

We have decided not to use any shared memory or threading, as the task at hand is very basic and can be achieved using just a single thread wihtout any performance degradation. We need very minimal number of variables to be stored during run time. We decided to keep it simple and bug free.

## Team work : THE BEST TEAMMATES EVER

Without a proper team communication/ understanding, we wouldnt have been able to make drastic design and functionality changes time to time. We were all in great sync, hence we were able to get the changes required very quickly in an agile fashion. It is this understanding and sync that helped us be dynamic.
Although we only see few functions in the final version, we had written so many functions during every phase of the robot, which we had to change and delete over time as we keep evolving.
And most of the time, we did parallel coding, multiple eyes and brains looking at the code and deciding and defining functions, so, the credit goes to all of us.

## Struggles: 

Initially we had struggled trying to setup a remote login to the robot, with some ipv4-ipv6 problems, which we had come over by starting an AWS-VM with dual interface support that bridges this ipv4-ipv6 issue. However, we still needed physical presence as its quite hard for people to work on robot remotely and play with it.

We also didn't made a great use of the git and the cross-compiling. Basically, we went from testing things on differents programs to a first version quite fluentely (that is why our main code is still named ```recover_from_stuck```). Since that, we never took the time to switch to a more organised working setup. Thanks to a good communication (and probably a little bit of luck), we never got any big problems related to that.


## Pins :

(the captors we dont care, it's auto-detected)

1 = touch

2 = compass

3 = gyro

4 = sonar

A(65) = left motor 

B(66) = top motor (to drop obstacles)

D(68) = right motor
# robotRaceEv3
# robotRaceEv3
