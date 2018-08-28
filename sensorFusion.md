# Sensor Fusion for Autonomous Driving
Sensor fusion is combining of sensory data or data derived from disparate sources such that the resulting information has less uncertainty than would be possible when these sources were used individually. In autonomous driving, we mainly concern 5 schemes, which are: **ego car fusion, object fusion, lane fusion, traffic light fusion, and traffic sign fusion**. In the following section, we will discuss each scheme in details.

## Object Fusion
In order to let the vehicle drive by itself, we need to have knowledge of where the objects (obstacles) are and how they move, then we could plan the route accordingly. 

On the ego vehicle, two kinds of sensors are equiped, which are radars and cameras. Here we only consider the sensor fusion of the two.

The motion model (state transition function) should close mimic the true motion of the target. The commonly used motion models are a **free motion model (constant acceleration)** and a **bicycle model**. In our use case, the state for the  CA model is [x, vx, ax, y, vy, ay] and the state for the bicycle model is [x, y, &#968;, v, &#969;, a]. In the CA model, x, vx, and ax are the position, velocity, and acceleration along the x axis; y, vy, and ay are the position, velocity, and acceleration along the y axis. In the bicycle model, x, y, &#968;, v, &#969;, and a are the (x, y) coordinate, yaw angle, velocity, yaw rate, and acceleration. There could be other motion models for different scenarios which will not be listed here.

We devise two different observation models for each of the two different sensing modalities: radar and camera. The radar observation model aims at modeling about a point target. It is designed to process direct position and velocity measurements. The camera observation model aims at dealing with bounding box measurements in the image plane. Optionally, through some post processing, it may also provide position, velocity, obstacle width, height, length, and type (i.e., vehicle, truck, bike, pedestrian, etc.).

We will use **unscented kalman filter (UKF)** based **interactive multiple model (IMM)** algorithm to incorporate different motion models. E.g., a CA UKF and bicycle model UKF filter bank.

There are many track-to-detection assignment algorithms. They can be divided into two categories, one is **single-hypothesis tracking (SHT)**, the other is **multiple-hypothesis tracking (MHT)**. Based on different use cases, we can choose the algorithms accordingly. In general, SHT is easier to implement but MHT outperforms when measurements are noisy. 

Data are asyncrhonously collected from each sensor but is timestamped to a common system time. The sensors should be configured to maximize redundancy infront of the vehicle against false readings or failure of sensors while also providing redundant sensing around as much of the vehicle as possible.

![alt text](https://github.com/trooperli/sensorFusion/blob/master/objectFusion.jpg "Object Fusion")

In the figure above, we described how to use the sensor data to obtain object list. The inputs from the radar are range (r), azimuth angle (az), and Doppler frequency shift (fd); those from the camera are longitudinal position (x), lateral position (y), longitudinal velocity (vx), lateral velocity (vy), obstacle type, and obstacle dimension (w, h, l).

## Lane Fusion
Lane boundaries, especially the current lane boundaries, are important references to help the ego vehicle stay within the lane. There are many approaches to model the lane, the most commonly used one is the clothoid model, which can be described in the following equation:

<img src="https://latex.codecogs.com/svg.latex?\Large&space;y=ax^{3}+bx^{2}+cx+d \; (1)" /> 

In Eq. (1), a represents curvature derivative, b curvature, c heading angle, and d lateral offset.

