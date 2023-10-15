---
layout: default
title: Follow Line
nav_exclude: false
---

# Follow Line

The goal of this exercise is to perform a PID reactive control capable of following the line painted on the racing circuit.

⚠️ **Handicap**:
  - To carry out this exercise, only the use of CAMERA is permitted. No other sensor is allowed in terms of navigation. 🚫
  - The car should complete the circuit as fast as possible.[^1] ⏱️

    That basically means two things:
      1. The full algorithm should relay on the camera feed.
      2. The image of the camera might be manipulated in order to improve the performance of the algorithm.
      3. The algorithm should be robust since the car is going to be moving at high speeds. This also implies that the algorithm should be able to work in real-time, with low latency.

<p align="center">
  <img src="https://jderobot.github.io/RoboticsAcademy/assets/images/exercises/follow_line/formula1_2.png" />
</p>

>**PRO HINT**
>
>The previous image implies the existence of a red line in the circuit. Which is great, since it is a very bright and saturared color, making its detection easier.

___

## Standard approach

### Theoretical concepts

These are some thoughts that have been taken into account in the problem formulation phase, as well as the design of its solution.

#### Line follower

[^1]: In the _simple circuit_, that usually means a lap time below 90 seconds.
