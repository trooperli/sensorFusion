## Practical Consideration of Sensor Fusion
In the most natural multi-sensor tracking scenario all sensors are synchronized and the complete state vector is updated using all available information simultaneously. However, in practice sensor data is delayed due to internal signal processing algorithms or limited communication bandwidth and will arrive to the fusion in an asynchronous manner. If this is not considered, performance will suffer.

The arrival order of measurements is generally unknown in advance, and in a system with delays it may happen that we receive a so called out-of-sequence measurement. It is possible for a measurement to arrive after the state vector has been updated with information from a newer measurement, i.e., out-of-sequence. In practice, the sensor should record the timestamp when it captures the scene and pass it to fusion modules. 

## Object Fusion
In order to let the vehicle drive by itself, we need to have knowledge of where the objects (obstacles) are and how they move, then the ego vehicle could avoid them and plan the route accordingly. 

On the ego vehicle, two kinds of sensors are equiped, which are radars and cameras. Here we only consider the sensor fusion of the two.

The motion model (state transition function) should close mimic the true motion of the target. The commonly used motion models are a **free motion model (constant acceleration)** and a **bicycle model**. In our use case, the state for the  CA model is [x, vx, ax, y, vy, ay] and the state for the bicycle model is [x, y, &#968;, v, &#969;, a]. In the CA model, x, vx, and ax are the position, velocity, and acceleration along the x axis; y, vy, and ay are the position, velocity, and acceleration along the y axis. In the bicycle model, x, y, &#968;, v, &#969;, and a are the (x, y) coordinate, yaw angle, velocity, yaw rate, and acceleration. There could be other motion models for different scenarios which will not be listed here.

There are two different observation models in accordance to two sensing modalities: radar and camera. The radar measures direct position and velocity measurements. The camera observation model aims at dealing with bounding box measurements in the image plane. Through some po, it may also provide position, velocity, obstacle width, height, length, and type (i.e., vehicle, truck, bike, pedestrian, etc.).

We will use **unscented kalman filter (UKF)** based **interactive multiple model (IMM)** algorithm to incorporate different motion models. E.g., a CA UKF and bicycle model UKF filter bank.

There are many track-to-detection assignment algorithms. They can be divided into two categories, one is **single-hypothesis tracking (SHT)**, the other is **multiple-hypothesis tracking (MHT)**. Based on different use cases, we can choose the algorithms accordingly. In general, SHT is easier to implement but MHT outperforms when measurements are noisy. 

Data are asyncrhonously collected from each sensor but is timestamped to a common system time. The sensors should be configured to maximize redundancy in front of the vehicle against false readings or failure of sensors while also providing redundant sensing around as much of the vehicle as possible.
