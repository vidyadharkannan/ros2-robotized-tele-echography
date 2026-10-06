# ROS 2-Based Robotised Tele-Echography

**M1 Advanced Robotics Project — École Centrale de Nantes**  
**2025**

<p align="justify">
This project was carried out as part of my **M1 Advanced Robotics programme (CORO-IMARO) at École Centrale de Nantes**. The project focused on developing a ROS 2-based bilateral teleoperation framework using a **Haption Virtuose 6D haptic interface** and a **Franka Emika Panda collaborative robot** for robotised tele-echography.
</p>

## Why This Project?

<p align="justify">
Ultrasound examinations normally require a trained medical practitioner to physically manipulate the ultrasound probe while controlling the contact with the patient. This can limit access to specialised examinations in locations where experienced practitioners are not available.
</p>

<p align="justify">
Robotised tele-echography provides an alternative in which the practitioner remotely controls a robotic manipulator carrying the ultrasound probe. For this interaction to be useful, the system must not only reproduce the operator's hand motion but also provide force feedback so that the operator can perceive the interaction occurring at the remote site.
</p>


## Project Objective

The objective was to develop a **ROS 2-based bilateral teleoperation framework** in which:

1. The **Haption Virtuose 6D** acts as the master device and captures the operator's position and orientation commands.
2. The **Franka Emika Panda** acts as the slave robot and follows the commanded Cartesian motion.
3. A **Cartesian impedance controller** provides compliant robot motion.
4. Force feedback is transmitted back to the haptic device, closing the bilateral teleoperation loop.




## System Architecture

The teleoperation system was developed in **ROS 2 Humble** and consists of two main sides:

- **Master side:** The Haption Virtuose 6D captures the operator's motion and provides haptic force feedback.
- **Slave side:** The Franka Emika Panda follows the commanded motion using Cartesian impedance control.

The operator's 6-DoF pose is transmitted from the Virtuose to the Franka controller through ROS 2. In the opposite direction, force feedback is sent from the robot side to the Virtuose, forming a bilateral communication loop.

<p align="center">
  <img src="images/bilateral_teleoperation1.png" width="650">
</p>

<p align="center">
  <em>ROS 2 bilateral communication architecture between the Haption Virtuose 6D and Franka Panda.</em>
</p>



## Haption-to-Franka Teleoperation

The **Haption Virtuose 6D** is used to command the desired position and orientation of the Franka end-effector.

The Virtuose publishes its pose through a device-specific ROS 2 message. A **Python pose relay node** was implemented to convert this data into the standard `geometry_msgs/msg/PoseStamped` format used by the robot controller.

The communication pipeline is:

`Virtuose 6D → Raw Pose → Python Pose Relay → PoseStamped → Cartesian Impedance Controller → Franka Panda`

The pose relay subscribes to `/out_virtuose_pose` and republishes the converted command on `/out_virtuose_pose_clean`. The Cartesian impedance controller then uses this pose as the desired end-effector command.

## Cartesian Impedance Control

The Franka Panda was controlled using a **Cartesian impedance controller implemented in C++** to provide compliant end-effector motion.

The controller computes the Cartesian pose error between the commanded and current end-effector pose. A virtual spring-damper model generates the Cartesian wrench.

The Cartesian wrench is converted into joint torques using the Jacobian transpose:

**τ = JᵀF**

where `J` is the robot Jacobian, `F` is the Cartesian wrench, and `τ` is the commanded joint torque.

Forward kinematics and the robot Jacobian were computed using the **Kinematics and Dynamics Library (KDL)**, a library for robot kinematics and dynamics calculations.




## Haptic Force Feedback

To close the bilateral teleoperation loop, a feedback wrench is sent from the robot side to the **Haption Virtuose 6D**, allowing the operator to perceive the interaction at the robot end-effector.

The system supports **6-DoF force and torque feedback**, while the experimental testing focused mainly on **Z-axis force feedback** to represent contact with a surface.




