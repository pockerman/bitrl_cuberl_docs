# bitsim

_bitsim_ is a C++20 library to model, simulate, estimate and control robots: from the maths of frames,
rotations and kinematics, through trajectories, motion planning, sensors and actuators, to a robot that
drives in a <a href="https://projectchrono.org/">Project Chrono</a> world, or on the floor with real motors
and a real camera. The source is on <a href="https://github.com/pockerman/bitsim">GitHub</a>.

Reinforcement learning is in \ref bitrl_main "bitrl", which builds on bitsim and is documented here too.

| ![A simulated robot following a line](line_following_infrared_sonar_lap.gif) | ![An extended Kalman filter fusing the robot's sensors](ekf_sensor_fusion_lap.gif) |
|:--:|:--:|
| A differential drive robot in Chrono follows a line with its infrared elements, and drives around the boxes its ultrasonic sensor hears. \ref bitsim_example_line_following_infrared_sonar "Example" | An extended Kalman filter fuses the encoders, the IMU, the infrared elements, the camera and two sonars: odometry (magenta) drifts, the filter (cyan) stays on the true path (yellow). \ref bitsim_example_ekf_sensor_fusion "Example" |
| ![Line following with a camera on a Raspberry Pi](pi_camera_line_following_lap.gif) | ![A simulated quadrotor flown with RC sticks](quadrotor_rc_simulation_flight.gif) |
| Hardware in the loop: the simulated robot follows the line with the frames of a real camera on a Raspberry Pi, streamed over HTTP. \ref bitsim_example_pi_camera_line_following "Example" | A quadrotor flown from the laptop's keyboard with RC stick commands, through an angle mode flight controller. \ref bitsim_example_quadrotor_rc_simulation "Example" |
| ![An arm following a straight line with inverse kinematics](inverse_kinematics_line.gif) | ![A rapidly exploring random tree planning a path](rrt_planning.gif) |
| A three joint arm in Chrono moves its tip along a straight line, with the joint angles from inverse kinematics at every step. \ref bitsim_example_inverse_kinematics "Example" | A rapidly exploring random tree plans across a warehouse; its path is shortened and compared with A* on an occupancy grid. \ref bitsim_example_rrt "Example" |

| ![A fixed wing aircraft taking off and landing under its autopilot](fixed_wing_flight_takeoff_landing.gif) |
|:--:|
| A fixed wing aircraft with a 6-DOF aerodynamic model takes off, flies a circuit of waypoints under an autopilot designed from its linearised dynamics, and lands with a flare and braking. \ref bitsim_example_fixed_wing_flight "Example" |

## What is in it

| Area | What bitsim has |
| ---- | --------------- |
| Geometry and maths | points, lines, frames, rotation matrices and Euler angles, probability distributions |
| Kinematics | planar and spatial arms with Denavit-Hartenberg parameters, forward and inverse kinematics, the kinematics of a differential drive |
| Trajectories | point to point motion with time scaling, waypoint trajectories |
| Motion planning | A* on an occupancy grid, rapidly exploring random trees |
| Dynamics | differential drive and bicycle models, a quadrotor, a fixed wing aircraft |
| Sensors | ultrasonic, infrared and IMU sensors, a downward camera, a camera stream; simulated, or real over MQTT. See \ref bitsim_sensors_guide |
| Actuators | DC gear motors with encoders, brushless DC motors; simulated, or real on a board. See \ref bitsim_actuators_guide |
| Robots | a differential drive robot in Chrono, built from a robot file, and the vehicle models. See \ref bitsim_robots_guide |
| Estimation | Kalman and extended Kalman filters, wheel odometry, sensor fusion |
| Control | PID controllers, line following, a quadrotor flight controller, a fixed wing autopilot |
| Simulation | rigid bodies and robots in Project Chrono, drawn with Irrlicht |
| Hardware in the loop | Arduino boards over USB and Wi-Fi, MQTT sensors and motors, a Raspberry Pi camera |

## Where to start

- \ref bitsim_install lists the dependencies, and explains how to build and install the library.
- \ref bitsim_examples "The examples" go through the library in order, from its basic types to robots on
  real hardware, and each has a page that explains it. The first is \ref bitsim_example_basic_types.
- \ref bitsim_sensors_guide, \ref bitsim_actuators_guide and \ref bitsim_robots_guide describe the parts a
  robot is made of.
- \ref build_documentation builds this site on your machine.

## bitrl

\ref bitrl_main "bitrl" is a library for reinforcement learning, built on bitsim: environments, from grid
worlds to Gymnasium's and robots in Chrono, and the algorithms that learn in them, from dynamic programming
to proximal policy optimization, with <a href="https://pytorch.org/cppdocs/">libtorch</a> for the deep
methods. Its pages are under _bitrl_ in the menu: \ref bitrl_main "what it is",
\ref bitrl_install "how to install it" and \ref bitrl_examples "its examples".
