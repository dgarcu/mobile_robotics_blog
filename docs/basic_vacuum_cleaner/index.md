---
layout: default
title: Basic Vacuum Cleaner
nav_exclude: false
---

# Basic Vacuum Cleaner

The objective of this practice is to implement the logic of a navigation algorithm for an autonomous vacuum. The main objective will be to cover the largest area of ​​a house using the programmed algorithm.

⚠️ **Handicap**:
  - To carry out this exercise, only the use of LIDAR is permitted. No other sensor is allowed in terms of navigation. 🚫

    That basically means two things:
      1. The algorithm will assume that the localization of the robot is not available for the whole process. This also implies that the robot will not be able to know its progress on the task.
      2. Some other sensors, like the bumper, may only be included to avoid risky situations, but never for navigation purposes.

<p align="center">
  <img src="https://jderobot.github.io/RoboticsAcademy/assets/images/exercises/vacuum_cleaner/vacuum_cleaner.png" />
</p>

>**PRO HINT**
>
>The previous image suggest the usage of several things:
>  - Wall follower algorithm.
>  - Spiral pattern for surface coverage.
>  - Random redirections when an obstacle is detected.

___

## Standard approach

### Theoretical concepts

These are some thoughts that have been taken into account in the problem formulation phase, as well as the design of its solution.

#### Wall follower

In an effort to cover the surface of a challenging part of the room like the wall's base, a **Wall-Follower Algorithm** could be implemented. In order to tackle this, a safe distance to the wall should be determined, lets call it **D**.

Once the robot detects an upfront obstacle, the robot can orientate its closest flank to it. Once this is achieved, the robot should maintain a constant distance (**D**) to the wall while moving forward. From this state, two main situations may ocurr:

  1. The robot detects another obstacle while following a wall (More likely a corner). The robot must avoid it trying to maintain the distance **D** to the wall constant.
  2. The robot moves away from the setpoint distance. In turn, it can be divided into two more:
     1. The robot is moving **towards** the wall. The robot must **turn away from the wall**.
     2. The robot is moving **away from** the wall. The robot must **turn towards the wall**.

This whole process shall be controlled by a _controller_... [_Genius, huh_](https://www.youtube.com/watch?v=3ILscTrJrY4)? Smells like a [**PID** Controller](https://en.wikipedia.org/wiki/Proportional%E2%80%93integral%E2%80%93derivative_controller)!

Since the room is filled with furnitures, there is a chance for the robot to start to follow the shape of one of them. Despite this, technically there would be no difference, so for simplicity we will continue to refer to the algorithm as **Wall-Follower**.

#### Obstacle avoidance

The only sensor we can count on is the LIDAR. That being said, that should be enough!

Since the main goal is to cover as much surface as possible, but we can not access to the odometry or localization of the robot, the best chance we get is... **send it to randomly navigate around the room**. That being said, turn to any direction a few degrees after detecting and obstacle should do the job.

To keep things even more simple (And apparently improving the performance of the algorithm...) the direction and degrees of rotation would be random too. However, since we cannot access to the odometry of the robot, this rotation will be time-restricted.

With this we get a robot that walks randomly around the room... But maybe there is a chance of complementing this with a little bit of intelligence...

Do you remember that DVD screen saver...? Seems like it is following a pattern whenever an obstacle is detected! 

<p align="center">
  <img src="https://repository-images.githubusercontent.com/344610266/246c3a80-7cf0-11eb-92d0-fe1d20e11982" />
</p>

Although for now we will leave it for the [advanced approach](#advanced-approach)...

#### Surface coverage

The robot should endeavor to maximize its coverage of the available surface area. However, no feedback about the progression or the location of the robot would be provided. The best chance we get is to perform a blind movement intented to cover the surface around the robot, aborting it if any obstacle is detected.

At first glance, I thought about the 3D Printers, since they must cover all the surface area with their nozzle:

<p align="center">
  <img src="https://content.invisioncic.com/ultimake/monthly_2022_07/ezgif.com-gif-maker.gif.e8adf9b28a6d78fa157482576ed91e28.gif" />
</p>

Browsing the [Prusa Knowledge Base](https://help.prusa3d.com/article/infill-patterns_177130) I ran into the [**Hilbert Curve**](https://en.wikipedia.org/wiki/Hilbert_curve), introducing me to the concept of [**Space-filling curves**](https://en.wikipedia.org/wiki/Space-filling_curve). According to the definition seen on that page:

>_A space-filling curve is a curve whose range reaches every point in a higher dimensional region, typically the unit square._

Which sounds perfect for this application... **If only we know the space to fill in advance**! But then I remembered that the environment will remain unknown for the whole task, so this approach was discarded much to my regret, since it looked promising, funny and eye-catching!

<p align="center">
  <img src="https://upload.wikimedia.org/wikipedia/commons/b/b1/Hilbert_curve%21.gif" />
</p>

In the end, I decided to go for a **spiral** pattern for this main reasons:

  1. It is simple to implement in terms of code.
  2. Could be fine-tuned with very few parameters. Easy to control.
  3. Assuming that the surface to be covered is unknown, this pattern would ensure that at least the surroundings of the robot will be covered.
  4. Looks cool enough too!

This procedure may be triggered either randomly or when the LIDAR sensor ensures that there is enough clearance in front of the robot to begin.

#### Control flow

The task is simple enough to be managed by a **FSM**[^1]. They can be summarized in three simple steps:

  1. **Move forward**. The robot will advance in its current orientation. Two events could take the robot out of this state:
     - The robot detects an obstacle in the trajectory. That will trigger randomly one of this states:
        - The **second** state, **redirect**.
        - The **fourth** state, **Follow wall**.
     - The robot determines that there is enough clearance. That will trigger the **third** state, **exploration**.
  2. **Redirect**. The robot will face a random direction. From there, the **first** state, **move forward**, will be triggered.
  3. **Exploration**. The robot will start the exploration pattern (_A.K.A._ **spiral** pattern, as stated before). If an obstacle is detected in the trajectory of the robot, one of the following states will be triggered randomly:
      - The **second** state, **redirect**.
      - The **fourth** state, **Follow wall**.
  5. **Follow wall**. The robot will follow the shape of the obstacle detected until a given event, for example, detecting another obstacle upfront. After this, the **second** state, **redirect**, will be triggered.

That being said, a provisional **flowchart** would look like this:

<p align="center">
  <img src="https://raw.githubusercontent.com/dgarcu/mobile_robotics_blog/main/docs/basic_vacuum_cleaner/assets/img/flowchart.png" />
</p>

However, as the application is being developed, this flowchart will suffer severe changes, I am sure of it!

___

## Advanced approach

> _To do..._

___

[^1]: [Finite-State Machine.](https://en.wikipedia.org/wiki/Finite-state_machine)
