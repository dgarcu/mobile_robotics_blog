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

    That basically means three things:
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

#### PID control

The second step is to control the car. This is going to be done by applying a PID control to the line position. This is going to be done by computing the error between the line position and the center of the image. This error is going to be used to compute the control signal mainly for the steering angle. (We will be convering the issue with the throttle in the next section, since a PID won't apply here).

[PID's](https://en.wikipedia.org/wiki/Proportional%E2%80%93integral%E2%80%93derivative_controller) are widely cover in the literature, so we are not going to cover them in this exercise. In fact, it was widely covered in the previous years of the degree I am currently studying... Probably you have already studied them too, so I am not discovering anything new here.

<p align="center">
  <img src="https://media3.giphy.com/media/8vIFoKU8s4m4CBqCao/giphy.gif" />
</p>

Neither I am covering the very implementation of the PID because as life have taught me, there is always someone who has already done it. In this case, the [PID library](https://pypi.org/project/simple-pid/) from Python has been used, however it was not packed whithin the resources from the Robotics Academy. Thankfully, nerds like you and I tend to share their blood, sweat and tears with the rest of the world, so you can find the source code of the library [here](https://github.com/m-lundberg/simple-pid), which I humbly borrowed for the developing of this exercise.

>**DISCLAIMER**
>
>If you are a teacher (Specially the teacher that is going to review this exercise), please don't take this as a lack of effort from my side.
>You should know that I am already familiar with said implementation, which was originally developed for [Arduino](https://playground.arduino.cc/Code/PIDLibrary/). In fact, the developer from the Python port mentioned that this is not their work either, but [**Brett Beauregards**](http://brettbeauregard.com/blog/2011/04/improving-the-beginners-pid-introduction/)'s work, which is the author from the original Arduino library. So, if you are going to blame someone, blame him. (Not Brett, the dude who ported it to Python) 😅
>Also, I have already played with it in my own port as a ROS 2 package, which you can check it out [here](https://github.com/dgarcu/ros2_pid). It's currently a work in progress, but that piece of code was worth a tiny part of the 4th position on the RoboCup 2023 Competition held in Bourdeaux! 🏆

All that being said, I kinda feel like this was an important part of the exercise, so leaving it to just a <kbd>Ctrl</kbd> +<kbd>C</kbd>//<kbd>Ctrl</kbd> + <kbd>V</kbd> feels like cheating. However, I find the other main part very much more interesting, for two main reasons:

1. It is the first time on the whole degree that we are dealing with raw image processing. 🤯
2. I have played with image processing before as part of my final degree project from the previous degree I studied. Casually, it was also related to autonomous driving. 🤔 Also involving Python and OpenCV. 🤔🤔

That is why **I decided to put all of my effort on push the development of the image processing part of the exercise beyond the limits!** I am going to cover it in the following sections. Hope that this is enough to compensate the lack of effort on the PID part.

#### Throttle control

At first glance, a PID also sounds good for this situation too. However, there is a problem with this approach.

The linear speed of the vehicle does not directly affect to the line position, whereas a PID controller is supposed to be used for a system where the output is directly related to the input. In this case, the output is the steering angle, which is directly related to the line position, whereas the input is the linear speed, which is not directly related to the line position.

The easy solution would be to set a constant linear speed and try to tune a steering PID for such a speed, as my grandfather would say:

>_Y a cascarla._

But like I said, I would like to compensate the lack of effort on the PID part, so I decided to go for a more complex solution.

The throttle will be dynamically governed by a specific designed controller. This controller will be based on the line position. In a nutshell, Three zones will be defined:

1. **Safe zone**: The car is in the middle of the line, so it can go as fast as possible.
2. **Warning zone**: The car is close to the edge of the line, so it should slow down. (Or, at least not going any faster)
3. **Danger zone**: The car is about to leave the line, so it should decrease its speed trying to get back to the safe zone.

This three zones will be delimited by two thresholds, which will be defined by the user. The throttle will behave in each zone as follows:

1. **Safe zone**: _Pedal to the metal!_ Not only the car will accelerate, but the acceleration itself will be increased the longer the car stays in this zone.
2. **Warning zone**: The car will **NOT** slow down, but the acceleration will remain constant.
3. **Danger zone**: The car will slow down, and the acceleration will be reset to its initial value. The decceleration itself will be correlated to the current speed of the car.

With this I am aiming to achieve a more natural behaviour of the car, which will be more similar to the one of a human driver. I am not sure if this type of controller has a name, but I am going to call it **_Linear Vel Controller_**. Maybe it is a [fuzzy controller](https://en.wikipedia.org/wiki/Fuzzy_control_system)?

#### Image processing

The first step is to obtain the image from the camera. This image is going to be the input of the algorithm. The image is going to be processed in order to obtain the line position. This is going to be done by applying a series of filters to the image.

We will be covering them more deeply in the next section, but for now, we will just enumerate them:

1. **ROI**: This actually is not a filter, but an approach for reducing the input image size. This is going to be done by cropping the image to the region of interest.
2. **Color filter**: This filter is going to be used to isolate the line from the rest of the image. This is going to be done by filtering the image by color.
3. **Gaussian filter**: This filter is going to be used to smooth the image. This is going to be done by applying a Gaussian kernel to the image.
4. **Binary filter**: This filter is going to be used to obtain a binary image. This is going to be done by thresholding the image.
5. **Contour filter**: This filter is going to be used to obtain the line position. This is going to be done by finding the contours of the image.

This are basically the main steps to obtain the line position. However, there are some other steps that are going to be performed in order to improve the performance of the algorithm, which could add complexity for some of the mentioned steps, specially for the **contour filter**.

### Implementation

[^1]: In the _simple circuit_, that usually means a lap time below 90 seconds.
