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

___

## Standard approach

### Theoretical concepts

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

> A space-filling curve is a curve whose range reaches every point in a higher dimensional region, typically the unit square.

Which sounds perfect for this application... If only we know the space to fill! But then I remembered that the environment will remain unknown for the whole task, so this approach was discarded much to my regret, since it looked promising, funny and eye-catching!

<p align="center">
  <img src="https://upload.wikimedia.org/wikipedia/commons/b/b1/Hilbert_curve%21.gif" />
</p>

In the end, I decided to go for a **spiral** pattern for this main reasons:

  1. It is simple to implement in terms of code.
  2. Could be fine-tuned with very few parameters. Easy to control.
  3. Assuming that the surface to be covered is unknown, this pattern would ensure that at least the surroundings of the robot will be covered.
  4. Looks cool enough too!

This procedure may be triggered either randomly or when the LIDAR sensor ensures that there is enough clearance in front of the robot to begin.

___

## Advanced approach

> _To do..._

___

