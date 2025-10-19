---
layout: default
title: Follow Line
nav_exclude: false
---

# Follow Line

The goal of this exercise is to perform a **PID reactive control** capable of following the line painted on the racing circuit.

⚠️ **Handicap**:
  - To carry out this exercise, only the use of **CAMERA** is permitted. No other sensor is allowed in terms of navigation. 🚫
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
>The previous image implies the existence of a **red line** in the circuit. Which is great, since it is a very bright and saturared color, making its detection easier.

___

## Standard approach

### Theoretical concepts

These are some thoughts that have been taken into account in the problem formulation phase, as well as the design of its solution.

#### PID control

The second step is to control the car. This is going to be done by applying a **PID control** to the line position. This is going to be done by computing the **error** between the line position and the center of the image. This error is going to be used to compute the control signal mainly for the steering angle. (We will be convering the issue with the throttle in the next section, since a PID won't apply here).

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

1. It is the first time on the whole degree that we are dealing with **raw image processing**. 🤯
2. I have played with image processing before as part of my final degree project from the previous degree I studied. Casually, it was also related to autonomous driving. 🤔 Also involving Python and OpenCV. 🤔🤔

That is why **I decided to put all of my effort on push the development of the image processing part of the exercise beyond the limits!** I am going to cover it in the following sections. Hope that this is enough to compensate the lack of effort on the PID part.

#### Throttle control

At first glance, a PID also sounds good for this situation too. However, there is a problem with this approach.

The linear speed of the vehicle does not directly affect to the line position, whereas a PID controller is supposed to be used for a system where the output is directly related to the input. In this case, the output is the steering angle, which is directly related to the line position, whereas the input is the linear speed, which is not directly related to the line position.

The easy solution would be to set a constant linear speed and try to tune a steering PID for such a speed, as my grandfather would say:

>_Y a cascarla._

But like I said, I would like to compensate the lack of effort on the PID part, so I decided to go for a more complex solution.

The throttle will be dynamically governed by a specific designed controller. This controller will be based on the line position. In a nutshell, **Three zones** will be defined:

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

#### ROI

The first step is to reduce the input image size. This is going to be done by cropping the image to the region of interest. All the information is going to be located in a certain part of the whole image.

With this, we are reducing the amount of information that the algorithm has to process, which is going to improve the performance of the algorithm. This is going to be done by applying a mask to the image.

>**Fun fact**
>This was actually implemented **after** the development of the rest of the algorithm. I was not sure if it was going to be necessary, but it turned out to be a good idea.
>That is why the images you are going to see in the following sections are not cropped. However, the final version of the algorithm is working with cropped images. The images are not cropped in the following sections because I did not want to re-record them, since the principle is the same! 😅

#### Color, gaussian and binary filters

This filters are going to be applied in order to isolate the line from the rest of the image. This is going to be done by filtering the image by color, smoothing the image and thresholding the image. The final result is going to be a binary image, where the line is going to be represented by white pixels and the rest of the image is going to be represented by black pixels.

This is a common approach for image processing, since the algorithms tend to work better with binary images. Also, there is a lot of information that is not relevant for the algorithm, so it is better to get rid of it.

Furthermore, this wrapps the fact that the color of the line is not relevant for the algorithm, which is a good thing. This means that the algorithm is going to be able to work with any color, as long as it is different from the rest of the image and you change the color filter accordingly.

In the image below, you can see the whole process. The first image is the original image, the second image is the image after applying the color filter, the third image is the image after applying the gaussian filter and the last image is the image after applying the binary filter.

<p align="center">
  <img src="https://github.com/dgarcu/mobile_robotics_blog/blob/master/follow_line/assets/img/segmentation_process.gif?raw=true" />
</p>

#### Contour filter

This actually is not a filter either, but a method for obtaining the line position. This is going to be done by finding the contours of the image. This is going to be done by applying the [findContours](https://docs.opencv.org/3.4/d4/d73/tutorial_py_contours_begin.html) method from OpenCV. However, this is going to be applied with some special considerations.

##### Image slices

In an effort to improve the performance of the algorithm, the image is going to be divided into slices. This is going to be done by applying a mask to the image. This mask is going to be a series of horizontal lines, which are going to be used to divide the image into slices. Below you can see an example of this.[^2]

<p align="center">
  <img src="https://github.com/dgarcu/mobile_robotics_blog/blob/master/follow_line/assets/img/slices.gif?raw=true" />
</p>

>**Fun fact _deja vu_**
>This was the moment when I realized that the ROI needed to be implemented. As you can see, the line never goes beyond the half of the image. So why bother processing the whole image? 😅

What I am trying to do here is to find the **moment** of each line slice. This is useful because now I have a series of points which represents the line. With the information of these points, I would be able to compute not only the line position, but also the shape of it. This is going to be interesting for detecting turns!

<p align="center">
  <img src="https://github.com/dgarcu/mobile_robotics_blog/blob/master/follow_line/assets/img/moments.gif?raw=true" />
</p>

##### Weighted average

Each of the moment you saw in the previous section is going to be used to compute the line position. This is going to be done by computing the **weighted average** of all the moments.

This means that the further points in the line are going to have more weight than the closer ones, trying to not overshoot the line position. The weight is distributed using a [sigmoid function](https://en.wikipedia.org/wiki/Sigmoid_function). Actually, this mathematical function is used a couple of times more in the development of the algorithm.

Said so, the final error will be the difference between the line position and the center of the image multiplied by the weight of the moment. In an image shown in later sections, you can see how this info is represented:

Moments are represented as a yellowed dot above the line. An arrowed line is drawed between the center axis of the img, to each one of the moments detected. The saturation of the color of the line is proportional to the weight of the moment. The more saturated the color is, the more weight the moment has.

Initially, the closer ones had more weight, thinking that it is more important to align the car with the near future. (You can actually see this in the following images) But as I have proven by smash-the-car-against-the-wall testing, this is not a good idea, since the car tends to ignore that time is inexorable, like most of us usually do, and sooner or later a wild turn will appear and despite of the car noticing it, not fixing its heading quickly enough.

In a stunning display of intelligence, I decided to **invert the weight of the moments**, so the further ones have more weight. This turned out to be a good idea, since at high speeds, the car tends to try to follow the end of the line, most of the time being more accurate with the future.

Since the speed was involved, I thought that maybe I can dynamically change the weights of the moments depending on the speed of the car. No advantage was found, so I decided to follow the **KISS principle**. (Keep It Simple, _Stupid!_)

The result of this is approach, for example, an acceptable input with a line position that is actually crossing the center of the image at a certain angle. This seemed to be helpful at higher speed turns, where is pretty easy to overshoot the line position. You will see this more clearly in the video at the end of the exercise.

<p align="center">
  <img src="https://github.com/dgarcu/mobile_robotics_blog/blob/master/follow_line/assets/img/gauges.gif?raw=true" />
</p>

##### Error threshold

The error is represented by a dot moving along the X axis of the img. Also, a vertical dashed line is drawed at the center of the img. This is the error threshold. The error is going to be computed as the difference between the line position and the center of the image. If the error is below the threshold, the car is in the safe zone. If the error is above the threshold, the car is in the warning zone. If the error is above the threshold, the car is in the danger zone.

This zones will be represented by colors. The safe zone will be represented by green, the warning zone will be represented by yellow and the danger zone will be represented by red. You will be able to see this in the video at the end of the exercise and in the images shown in the following sections.

#### LVC (Linear Vel Controller)

The linear vel controller is going to be used to govern the throttle. This is going to be done by computing the linear speed of the car based on the line position. This is going to be done by applying a series of thresholds to the line position.

When the car is in the safe zone, a watchdog is set. Once a certain amount of time has passed, the car will accelerate at an increasing rate, so the car will not only accelerate, but the acceleration itself will be increased the longer the car stays in this zone. This is represented as _Time on track_ label at the bottom left of the image you will see in the video.

When the car is in the warning zone, the acceleration rate is kept constant. This is going to be done in order to avoid sudden changes in the linear speed, so the steering PID has more time to react, trying to avoid braking to the minimum linear speed.

When the car is in the danger zone, the acceleration rate is reset. Along with this, the car will slow down, and the decceleration itself will be correlated to the current speed of the car. Again, a **sigmoid function** is used to distribute the decceleration rate. The higher the speed, the higher the decceleration rate.

This mimics the natural behaviour of a human driver, allowing the PID to recover the line safe zone from a slower speed, which is easier to achieve. Once this is done, the car can accelerate again. You will be able to see this in the gauges of the car in the video at the end of the exercise. (A gauge for the angular speed and another one for the linear speed)

#### Dynamic PID

Having a dynamic linear speed also introduces a problem with the steering PID. The steering PID is tuned for a certain linear speed, so it is not going to work properly if the linear speed changes. This is going to be done by applying a series of thresholds to the linear speed.

For the whole range of speed of the car, a manual tuning of the PID is going to be applied. However, for intermediate speeds, a **linear interpolation** between the two closest speeds is going to be applied. This is going to be done in order to avoid sudden changes in the steering angle, so the car will be able to follow the line more smoothly.

This values are going to be displayed in the final image from the video. The idea behind this was trying to reach a smoother behaviour with higher speeds.

### Results

All this effort was worth it. The car is able to complete the simple circuit in less than 90 seconds. (The video at the end of the exercise is a proof of this) Not only this, but is able to complete all the circuits in an apparently safe way and reasonably speed.

...

Actually not, because according to [this F1 fandom page](https://f1.fandom.com/wiki/Circuit_de_Barcelona-Catalunya), the record for Montmeló circuit is currently **1:16.330** by **Max Verstappen**.

<p align="center">
  <img src="https://e00-mx-marca.uecdn.es/mx/assets/multimedia/imagenes/2023/05/26/16851271280364.jpg" />
</p>

Whereas the record of mine overengineered algorithm is **2:19.000**. Not even close. Despite of this, I am still proud of the results, since I am pretty sure that I would end in the wall if I tried to drive a real F1 car that fast, not even thinking of achieving 2:19.000.

No more chit-chat, here is the video of the car completing the simple circuit (You might want to check [this link](https://open.spotify.com/track/5X7zViJE5FvOZ8vhcVk1NV?si=312c922d5f8a4a5b) just before clicking the play button):

<iframe width="560" height="315" src="https://www.youtube.com/embed/vawUmwpezh8?si=Zml-2CRwnc9VITIk" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

[full_lap.webm](https://github.com/dgarcu/mobile_robotics_blog/assets/92941081/331e7a2c-b786-4a3d-8dbd-c9e82e9ea2d4)

### Final thoughts

I really think that the time per lap could be improved by further tweaking of the parameters. But above all, I think that the overall solution can be improved by adding a simple concept:

**The shape of the line was not explored thoroughly**. What I mean is that the parameters of the steering PID are currently tied only to the linear speed of the car, but maybe, **if they are also correlated to the shape of the line**, this parameters could be more fine adjusted. Next life I guess.

[^1]: In the _simple circuit_, that usually means a lap time below 90 seconds.
[^2]: The green lines are the ones that are going to be used to divide the image into slices. They are only used for visualization purposes, and they are not part of the algorithm.
