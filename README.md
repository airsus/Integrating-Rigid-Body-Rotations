# Integrating Rigid Body Rotations

I've been working on angular velocity estimation of an object from noisy pose measurements, which is a common problem in augmented reality and surgical applications. To test my algorithms, I needed to do the opposite and generate simulated noisy rotation measurements from a known angular velocity profile. That turned into a project as well. I came across to this beautifully and clearly written paper below, which made me question numerous papers I've read that restricted their formulation to unit quaternions and stated 
the algebraic unit norm constraint as an inconvenience that comes at a price of having the computational efficiencies of using quaternions.

Rucker, Caleb. "Integrating rotations using nonunit quaternions." IEEE Robotics and Automation Letters 3.4 (2018): 2979-2986.

Below is a summary of the implementation of the paper and some test results.

**Properties of quaternions** 
The properties of nonunit and unit quaternions are as follows:

![Nonunit quaternions](./figs/unit_vs_nonunit_quaternion.png)

The paper shows that the ODE that relates the unit quaternion derivative to angular velocity in body frame is the  minimum norm solution to the general form of the equation for nonunit quaternion. The parameter c is defined as such to minimize the drift of the quaternion norm. Note that this is only necessary if norm gets close to zero or floating point limit. Thee equations for converting quaternion to rotation matrix is different for nonunit quaternions. Therefore, as long as you're consistent with this notation and you do the normalization during conversion, the drift in integration does not have an affect on the rotation matrix. The paper also gives a simple example where angular velocity depends on rotation calculation and shows that the momentum conservation of nonunit quaternion approach is similar.

**Integration of angular velocity using exponential update and non-unit quaternions**

The table below shows the equations for different implementations presented in the paper. The exponential methods are very common in robotics and aerospace applications to preserve the structure of SO(3).Indeed, the algorithm that I mentioned above and will share later  uses the exponential update rule in the process model of a kalman filter. The paper shows that you can use nonunit quaternions with a standard ODE solver like Runge-Kutta and get better results that exponential update method which assumes constant velocity or constant acceleration between measurements.

![Methods](./figs/methods.png)


**Test Results**
Here's a test data generated by integrating a given angular velocity profile using RK4 and normalizing the quaternion at the end of each iteration.

![Angular velocity profile](./figs/test_data.png)

Here's the drift in quaternion norm for different implementations. You can see the c parameter limiting the norm as time progresses.

![Quaternion norm drift](./figs/quat_drift.png)

Here's the determinant of the rotation matrix obtained using exponential update with unit quaternions, exponential update using rotation matrices, integration of nonunit quaternions with RK4 (c=0) and integration of nonunit quaternions with RK4, where the drift limit factor c is nonzero. As you can see the drift doesn't affect the rotation matrix since the normalization is captured during conversion. Overall, the approach in the paper gives better results than the exponenial update method here. On the right, you also see the integration of the rotation matrix elements using RK4, which clearly shows that you need an [orthogonalization](https://en.wikipedia.org/wiki/Gram–Schmidt_process) step after that (hopefully this approach fell out of practice completely).

![Rotation matrix determinant drift](./figs/determinant_drift.png)

Enjoy!









