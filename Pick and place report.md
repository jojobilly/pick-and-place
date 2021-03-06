## Project: Kinematics Pick & Place

### Writeup / README

## Kinematic Analysis

### Forward Kinematics
The first step to understanding the pick and place project was to have the student learn about serial manipulators and how to perfrom forward and inverse kinematics using homegenous transformation matrices and the Denavit Hartenburg table.  The Modified DH paramter table was constructed with the instructions from the Udacity lectures. The steps presensented in the lecture follow as:

#### 1) label all joints from 0 to N
#### 2) label all links from 0 to N
#### 3) Draw lines through each joint defining the joint axes.
#### 4) Assign the z axis of each frame to point along joint axes.
#### 5) Idenfify the common normal between each frame z_i-1 to z_i
#### 6) Th endpoints of th intermediate links are associated with two joint axes i and i_+1
#### 7) For the base link, always choose frame 0 to be conicident with frame 1 when the first joint variable is = 0 this will ensure a_0 = alpha_0 and if joint 1 is a revolute d1 = 0, if joint 1 is prismatic, then theta_1 = 0
#### 8) For the end effector frame, if joint n is revolute, choose X-n to be in the direction of x_n-1 when theta_n=0 and the origin of frame n such tha d_n = 0

The a paramaters of the DH table consist of 4 unique properties of the model that describe the orienation of each link of the robot system with respect to the previous joint.  The distance between the z_i-1 to z_i.  The d paramters represent the distance between x_i-1 to x_i .  The alpha values or twist angles are the measured angles between z_i-1 and z_i measured about the x_i-1 axis.  The theta parameters represent the twist angles of the revolute joints.  Once all the parameteres have been obtained a generic homogenous transformtation matrix can be used to calculate all of the transformation matrices between each link.  When combiune these rotation matrices can compute and end effector postion if all the joint angles are supplied.  

### 6 dof schematic

![](./pics/DH_params_pickandplace.PNG)


### Generic DH transformation matrix

![](./pics/gen_DH_matrix.PNG)

### DH parameter table

Links | alpha(i-1) | a(i-1) | d(i-1) | theta(i)
--- | --- | --- | --- | ---
0->1 | 0 | 0 | .75 | q1
1->2 |-pi/2| .35 | 0 | q2-pi/2 
2->3 | 0 | 1.25 | 0 | q3
3->4 |-pi/2| -.054 | 1.50 | q4
4->5 | pi/2| 0 | 0 | q5
5->6 |-pi/2| 0 | 0 | q6
6->EE | 0 | 0 | .303 | q7


### Transformation matrix frame 0 to 2

![](./pics/transformation_matrix_T0_2.PNG)

### Transformation matrix frame 2 to 4

![](./pics/transformation_matrix_T2_4.PNG)

### Transformation matrix frame 4 to 6

![](./pics/transformation_matrix_T4_6.PNG)
### Transformation matrix frame 0 to EE

![](./pics/transformation_matrix_T0_EE.PNG)

### Inverse Kinematics
The approach used for finding the joint angles of the robotic arm as a function of the end effector postion consisted of breaking the robot into two seperate parts.  The wrist center comprised of 3 revolute joints and the first 3 joints of the robot leading up to the wrist center.  Although numerical methods exist to compute all 6 joint angles it can be difficult to arrive at the correct set of solutions without an accurate guess.  Computational time for numberical procedures can take signifigant periods of time depending on the complexity and hardware used. There will be multiple answers for each joint angle but the joints themselves impose their own constraints depending on the joint type ie. (does the joint move in that direction or past a certain degree limit).  Because of these limitations and ease of calculation an analytical or closed form solution was used.  Mapping the geometry of the first 3 joints or up to the wrist center, the joint angles can be found with simple geometry.  Angela Sodelman's videos on youtube were a big help in vizualization and methods to find the first 3 joint angles. Although I drew my own diagram to calculate the first joint angles I opted to utilize the schematic provided from Udacity because it was more simple.  Once the first three joint angles have been found, matrix multiplication will yield the last 3 joint angles.  This equation shown below takes the transpose of the rotation matrix from 0 to 3 muliplied with the Rotation matrix from 0 to 6.  This will output joint angles from 3 to 6.  

Finding the exaxt postion of the wrtist center required additional rotations to the transformation matrices derrived from the DH parameter table.  This was due to an offset between the gripper frame and frame 6.  An additional roation of 180 degrees around the z axis and -90 degree rotation about the y axis aligned these two frames.  

### Frame 0 to WC inverse kinematic diagram

![](./pics/0_3_inv_kin_diagram.PNG)

### Inverse kinematic frame 0 to WC

![](./pics/inv_kin_diag.PNG)



### Inverse kinematic frame 3 to EE

![](./pics/inv_kin_eq.PNG)




### Project Implementation
using the safe_spanwer script the gazebo, ros master, Rviz environments were brought up.  I had to leave the computer completely alone when allowing gazebo to start up.  It seemed like each time I tried to operate while gazebo was starting up it would crash.  After these environements were started the IK_server.py scripts were run.  This scripts purpose was to receive the individual end effector postions from Moveit motion planning and calculate the joint angles needed to place the end effector in the correct position.  The two programs worked together with the pick and place operation completing successfully, but the path that Moveit Planned would sometimes deviate wildly from what the shortest path.  I searched and found within the moveit::planning interface::MoveGroup class and found an option to set the minimum number of path planning attempts to find the shortest path.  I set this minimum number to 10 for both the move group and the end effector group.  After adding this piece of code to the trajectory_sampler.cpp file the ROS environment was refreshed using catkin_make to rebuild.  After this Moveit did a resonably good job with calculating the minimum distance between the start and end pose.  Running the simulation through 10 cycles of pick an place operations the robot failed to pick up the cylinder one time.  This was during a period when the continue button was pushed and the robot acted without user input.  A sleep period was added to the trajectory_sampler file to prevent the gripper arm from moving before the gripper had to time to fully contract.  This correction usually allowed the robot arm to operate without user input, but every know and then it would fail to pick up the cylinder.      

### trajectory_sampler code addition

![](./pics/trajectory_sampler_snip.PNG)

### Project Completion

![](./pics/pickandplace_result.PNG)

### Conclusion
Overall I found this project very interesting and useful information.  I found the most difficult information to process was the DH parameter configuration, especially the combination of the frames at the wrist center.  Watching tutorials on Youtube a different DH instruction set was used but I understand that combining the frames at the wrist center actually simplifies the transforms between the frames.  Going forward I would like to spend more time trying to understand ROS industrial software for path planning.  the IK_server script worked okay but in a tight spot or manufacturing environment the user would want the paths to be as optimized as possible.  This would decrease excessive wear on robot components and decrease the time it takes to perform a pick and place operation.


