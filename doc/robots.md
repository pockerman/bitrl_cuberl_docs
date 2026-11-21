\page bitsim_robots_guide Robots

bitsim has robots of two kinds:

- a **robot in Project Chrono**, a rigid body model that drives on a floor, bumps into boxes and carries
  sensors and motors: \ref bitsim::rb::bitsim_chrono::CHRONO_DiffDriveRobot "CHRONO_DiffDriveRobot", a
  differential drive robot built from a robot file;
- **vehicle models**, the equations of motion of a vehicle, which step without a physics engine: a
  differential drive, a bicycle model, a quadrotor and a fixed wing aircraft.

The Chrono robot is the one the simulation, estimation and hardware in the loop examples drive; the
models are for planning, estimation and control, where the equations themselves are what matters.

## The robots at a glance

| Robot | What it is | Needs | Examples |
| ----- | ---------- | ----- | -------- |
| \ref bitsim::rb::bitsim_chrono::CHRONO_DiffDriveRobot "CHRONO_DiffDriveRobot" | a chassis, two driven wheels with DC motors and a ball caster, in Chrono, with its sensors | `ENABLE_CHRONO` | \ref bitsim_example_chrono_diff_drive, \ref bitsim_example_line_following_infrared_sonar, \ref bitsim_example_ekf_sensor_fusion, \ref bitsim_example_real_dc_motors |
| \ref bitsim::dynamics::DiffDriveDynamics "DiffDriveDynamics" | the motion of a differential drive in the plane, with its Jacobians for a Kalman filter | nothing | \ref bitsim_example_diff_drive_kinematics, \ref bitsim_example_extended_kalman_filter |
| \ref bitsim::dynamics::BicycleVehicleModelDynamics "BicycleVehicleModelDynamics" | the kinematic bicycle model of a car-like vehicle | nothing | |
| \ref bitsim::dynamics::QuadrotorDynamics "QuadrotorDynamics" | the 6-DOF rigid body dynamics of a quadrotor, driven by the speeds of its four rotors | nothing | \ref bitsim_example_quadrotor_dynamics, \ref bitsim_example_quadrotor_rc_simulation |
| \ref bitsim::dynamics::FixedWingDynamics "FixedWingDynamics" | the 6-DOF flight dynamics of a fixed wing aircraft with a propeller and a landing gear | nothing | \ref bitsim_example_fixed_wing_flight |

## The differential drive robot in Chrono

\ref bitsim::rb::bitsim_chrono::CHRONO_DiffDriveRobot "CHRONO_DiffDriveRobot" is a chassis, two driven
wheels and a ball caster. Each driven wheel has a \ref bitsim::actuators::DCMotor "DCMotor", see
\ref bitsim_actuators_guide, and the robot carries the sensors of \ref bitsim_sensors_guide. It is loaded
from a robot file, added to a Chrono system, and then driven step by step:

@code{.cpp}
using bitsim::rb::bitsim_chrono::CHRONO_DiffDriveRobot;

chrono::ChSystemNSC system;
// ... gravity, the collision system, the floor

CHRONO_DiffDriveRobot robot;
robot.load(std::filesystem::path(bitsim::consts::ROBOTS_DIR + "/bitsim_diff_drive_robot.json"));
robot.set_world_pose(0.0, 0.0, 0.0);   // the axle centre at (x, y), facing yaw [rad]
robot.add_to_sys(system);

const double dt = 0.002;
for (int i = 0; i < 2000; ++i)
{
    robot.set_velocity(0.3, 0.2);      // forward speed [m/s] and turning rate [rad/s], positive left
    robot.update(dt);                  // run the motors and pass their output to the wheels
    system.DoStepDynamics(dt);
}

const auto centre = robot.axle_center();
const auto heading = robot.heading();
@endcode

- `set_velocity(v, omega)` turns a forward speed and a turning rate into a target speed for each wheel's
  motor; `set_speed(v)` drives straight, and `set_motor_speed(m, speed)` sets one wheel's motor directly.
- `update(dt)` runs the motors and passes their output to the wheels. Call it before every
  `DoStepDynamics(dt)`. A simulated motor puts its torque on its wheel, and a real one turns its wheel to
  the angle it reports, see \ref bitsim_actuators_guide; either way the contact with the floor moves the
  robot.
- `get_pose()`, `axle_center()` and `heading()` give where the robot is; `get_chassis()` and
  `get_wheel(m)` give its Chrono bodies.
- `get_sonar()`, `get_rear_sonar()`, `get_line_sensor()`, `get_imu()` and `get_camera()` give its sensors,
  and `set_floor()` the floor that the line sensor looks at.
- `set_motor(m, motor)` replaces a wheel's motor, for example with a real one.

### The robot file

The robot file, `robots/bitsim_diff_drive_robot.json`, describes the robot. Its sections:

| Section | What |
| ------- | ---- |
| `name` | the name of the robot |
| `robot-data` | `max_speed`, the largest speed of a wheel on the ground [m/s], `wheel_radius`, `wheel_width`, `axle_length`, the distance between the wheels, and `caster_wheel_radius`, all in m, and the `density` of the parts [kg/m³] |
| `material` | the `friction` and the `restitution` of the contacts |
| `chassis` | the size of the chassis along x, y and z, `chassis_x` to `chassis_z`, and the position of its centre of mass |
| `left-wheel`, `right-wheel`, `caster-wheel` | where each wheel is, relative to the chassis |
| `motors` | the backend of the wheel motors, `SIMULATION`, and their parameters, see \ref bitsim_actuators_guide |
| `sensors` | the sensors and their backends, see \ref bitsim_sensors_guide |

The `chassis`, `motors` and `sensors` sections have a `description` that says what their values mean. A
changed copy of the file makes a different robot. The examples load the one in `robots/` with
`bitsim::consts::ROBOTS_DIR`, or `bitsim::consts::build_robot_file_path("bitsim_diff_drive_robot.json")`;
both are in \ref bitsim::consts::data_dir "data_dir()": the directory in the `BITSIM_DATA_DIR`
environment variable if it is set, otherwise the source tree bitsim was built from if it is still there, and
otherwise the files an installed bitsim put in `share/bitsim`.

## The vehicle models

The vehicle models are the equations of motion of a vehicle, with no physics engine: each takes the state
and the inputs and steps them forward.

**The differential drive.** \ref bitsim::dynamics::DiffDriveDynamics "DiffDriveDynamics" moves the centre
of the axle at \f$(x, y)\f$, facing \f$\theta\f$. In its default version, V1, the input is the speed
\f$v\f$ and the turning rate \f$\omega\f$, and over a step the robot drives an exact arc; with no errors
that is exact for a constant \f$v\f$ and \f$\omega\f$. It also gives the Jacobians of the step with respect
to the state and to the errors, which is what an extended Kalman filter needs, as in
\ref bitsim_example_extended_kalman_filter. Its other versions take a forward Euler step, V2 with \f$v\f$
and \f$\omega\f$ and V3 with the speeds of the two wheels.

**The bicycle model.** \ref bitsim::dynamics::BicycleVehicleModelDynamics "BicycleVehicleModelDynamics" is
the kinematic bicycle model of a car-like vehicle: its position and heading, a wheelbase \f$L\f$ between the
front and the rear wheels, and the speed and the steering angle as inputs.

**The quadrotor.** \ref bitsim::dynamics::QuadrotorDynamics "QuadrotorDynamics" is the rigid body dynamics
of a quadrotor in a plus layout, driven by the speeds of its four rotors: the thrust \f$k_1 \omega_i^2\f$
of each rotor, the torques of their differences about the three axes, and the drag torque
\f$k_2 \omega_i^2\f$ of each rotor about its axis. \ref bitsim::control::QuadrotorFlightController
"QuadrotorFlightController" flies it in angle mode, from the sticks of a radio transmitter, which
\ref bitsim_example_quadrotor_rc_simulation takes from the laptop's keyboard.

**The fixed wing aircraft.** \ref bitsim::dynamics::FixedWingDynamics "FixedWingDynamics" is the 6-DOF
flight dynamics of a small fixed wing aircraft, after Beard and McLain's _Small Unmanned Aircraft_: air data
with wind, lift that stalls, drag, the side force and the moments, a propeller driven by a DC motor, and a
landing gear on the ground. \ref bitsim_example_fixed_wing_flight trims it, designs an autopilot from its
linearised dynamics, and flies it from takeoff to landing.
