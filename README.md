# Robot Localization
An and Madie

## Goal:

The goal of the project was to localize a robot in known space using particle
filtering, a localization algorithm that given a map, estimates the robot’s
position and pose on the map as it traverses through its environment.

The image below shows our robot localizing from the Miller Academic Center first
floor as it moves around.

![image](/images/particle_filter.gif)

## Solution:

Our solution implemented a particle filter with the following steps:

1. Initializing a particle cloud
2. Updating particles using new odometry pose
3. Updating particles’ weight based on laser scan data
4. Updating the robot’s pose
5. Resampling the particles in the cloud based on the point’s weights

#### Summary of how a Particle Filter functions

NOTE: This is a rough, abstract explanation of a particle filter.

The particle filter works by first sampling a random set of particles of varying
positions and poses on the map. As the robot traverses through its environment,
each particle copies the robot’s exact movements as if it were its own. The
laser scan collected by the moving robot is projected onto each particle, and
the laser scan data with where the particle is in relation to the obstacles/map.
How far off the particle’s sensed surroundings are from the laser scan data will
determine the particle’s weights, which in turn, determines how the particle
cloud gets resampled. A high particle weight indicates that the particle is
close to where the robot actually is on the map. Therefore, particles of higher
weight get particles resampled around them. Additionally, the particle with the
highest weight is assumed for the time being that it is the robot’s pose. The
robot’s pose gets updated to where this particle is. This process gets looped
multiple times, and each time, the localization gets increasingly accurate since
the particle cloud begins to congregate onto a single particle that is the
closest to the robot’s actual position and pose.

#### 1. Initializing a Particle Cloud

We first initialized a particle cloud by creating normal distributions of x, y,
and theta values centered around an initial. given estimate of the robot's pose for our particles to assume, and randomly sampling values from the distributions. Each 
distribution’s output was stored as an array of values, and each initialized
particle was assigned an x, y, and theta value corresponding to each index
of the array. We then filtered out particles that
were initialized outside of the map by simply not adding them to the particle cloud; This is because x and y values outside of the map do not exist in the stored map. Finally, each particle was
normalized. 

#### 2. Updating particles using the robot’s new odometry pose

As the robot traverses through the map, its pose in the robot coordinate frame
is updated and tracked. As it is necessary for the particle filter to work, each
particle’s pose and position must be updated to mimic the way the robot is
moving. We calculate this using matrix multiplication.

First, we must express the robot’s previous pose (T1) and its current pose that
it moved to (T2) in the odometry coordinate frame. We can represent these two
using homogeneous transformation matrices as follows:
![image](/images/image3.jpg)
However, we are interested in understanding the
robot’s current pose relative to its previous pose. We can do this by inverting
our previous pose T1 matrix, then performing matrix multiplication with the T2
matrix, which will give us our relative pose. 
![image](/images/image1.jpg) 
Now
that we have our relative pose, we want to make the particles move in the same
way the robot is moving. Since we know what the particle’s current pose is, and
we want to know the pose of the where we want the particle to be, we can find
this by representing our particle’s current pose as a homogeneous matrix as
well, and again performing matrix multiplication of the particle pose and
relative pose to find the pose of the particle after moving.
![image](/images/image4.jpg)

#### 3. Updating particles’ weight using the robot’s laser scan data

Now that the particles have moved as the robot has, we can start to determine
whether or not the particle is alike in any way to the way the robot is
oriented, and update its weight based on this. In order to first do this, we
represent the given laser scan data as a matrix, which is as of right now, in
polar coordinates. We calculate the Cartesian coordinates and represent these in
a matrix (with its last column being all 1s to make the math work out).

Each particle in the cloud is represented as a homogeneous transformation
matrix. As each particle is currently defined in the map frame, and the laser
scan is in the odometry frame, we need to project the laser scan onto the
particle in the map frame. We do this using matrix multiplication, where the
homogeneous matrix manipulates the laser scan to be in the map frame.

![image](/images/image2.jpg) 

From there, we find the closest object distance of the projected laser scan and
adjust our weights based off of it.

#### 4. Updating the robot’s pose

We update the robot’s pose by looping through the particle cloud and finding the
particle with the highest weight. Whichever particle has the highest weight, the
robot’s pose gets updated to that particle’s pose.

#### 5. Resampling the particles in the point cloud based on their weights

To resample the particles, we used a distribution based on the weights of
the particles currently in the particle cloud. To each new particle, we added
noise to their pose. Intuitively, we knew that adding too much noise would cause
an ineffective filter, as would adding too little noise. We determined our
values using guess-and-check method.

## Design Decision:

#### Implementation of update_particles_with_odom()

We chose to update the Odom with linear algebra to convert between coordinate
frames. We wanted to learn more about a generalized solution for transforming
poses into different coordinate frames. We chose to use Numpy because it has
fast computation time which is important for large particle filters and has
multiple built-in functions like transpose and matrix multiplication. We chose
to replace a for loop by integrating the particles into a single matrix 361 x 3
and multiplying it by the pose. This allowed us to do a single Numpy matrix
multiplication as opposed to a for loop through 361 particles.

## Challenges:

We faced multiple challenges with our robot. We initially ran into multiple
problems visualizing the robot which led us to implement the majority of the
code functions. Our implementation was extremely difficult to debug since we
tried debugging at the final step as opposed to testing smaller test cases
before the final. We focused mainly on the logic of each problem as opposed
smaller implementation details like noise, filtering data, and matrix
multiplication and parsing.

## Improvements:

If we had more time, we would make numerous improvements.

Our first improvement would be optimizing the code for speed. The code we
implemented included numerous loops that could be replaced with Numpy operations
like matrix multiplication, saving certain data points, or trying libraries like
Numba which has tags like @jit that optimize Python code. After optimizing code,
we would also try to adjust our parameters for the particle numbers and noise
and quantitatively test which filter works better.

Moreover, given more time, we would love to explore different implementations of
where to add noise or adjustments to make the particles converge more precisely
onto a single point. Our final particle filter had some noise around the
convergence as shown below.

![image](/images/particle_filter.gif)

## Lessons:

We learned multiple lessons including transforming coordinates between reference
frames, optimizing code, and effective debugging skills.

During the process of creating a particle filter and especially during
debugging, we were able to cement our understanding of the main steps of
implementing a particle filter and switching between coordinate frames. The
transformation matrix can be applicable to numerous other situations.

We learned the importance of time-blocking debugging before asking for help from
the course assistant or the Professor. We also learned the importance of
debugging with small test cases and discrete functions as opposed to debugging
the whole particle filter. With small test cases and walking through the logic,
we could narrow down the problems to the implementation of a solution. A
function that does not flag errors and has correct logic does not necessarily
mean that it is working as it should be. We also realized the importance and
impact of noise values. We initially chose them as arbitrary values that were
decided without justifications or testing at the start of coding. After talking
with the professor, we learned that our values were set significantly too high.
In the future, we will also use a reference point for our initial constants or
chosen values and then do further testing or research to find the most optimal
value.
