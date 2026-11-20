\page bitsim_sensors_guide Sensors

# Sensors

bitsim models the sensors of a small mobile robot. Each sensor can be **simulated**, so that a
robot in a Chrono world senses that world, or **real**, a sensor on a board that sends its
readings over MQTT. The application reads both the same way, so the code that drives a simulated
robot drives a real one too.

This page lists the sensors bitsim supports, what each of them reads, how the simulated ones are
modelled and what that model leaves out, and the messages the real ones send.

## The sensors at a glance

| Sensor | Measures | Values | Simulated by | Real, over | Robot file |
| ------ | -------- | ------ | ------------ | ---------- | ---------- |
| \ref bitsim::sensors::UltrasonicSensor "UltrasonicSensor" | distance to the nearest obstacle along its beam | 1, [m] | \ref bitsim::sensors::backends::CHRONO_UltrasonicBackend "CHRONO_UltrasonicBackend" | \ref bitsim::sensors::backends::MQTT_UltrasonicBackend "MQTT_UltrasonicBackend" | `front-sonar`, `rear-sonar` |
| \ref bitsim::sensors::InfraredSensor "InfraredSensor" | reflectance of the floor under each element | one per element, in [0, 1] | \ref bitsim::sensors::backends::CHRONO_InfraredBackend "CHRONO_InfraredBackend" | \ref bitsim::sensors::backends::MQTT_InfraredBackend "MQTT_InfraredBackend" | `line-sensor` |
| \ref bitsim::sensors::IMUSensor "IMUSensor" | specific force and angular velocity | 6, [m/s²] and [rad/s] | \ref bitsim::sensors::backends::CHRONO_IMUBackend "CHRONO_IMUBackend" | \ref bitsim::sensors::backends::MQTT_IMUBackend "MQTT_IMUBackend" | `imu` |
| \ref bitsim::sensors::DownwardCamera "DownwardCamera" | an image of the floor ahead | an image | itself, from a \ref bitsim::sensors::FloorImage "FloorImage" | | `front-camera` |
| camera stream | images from a real camera | an image | | \ref bitsim::sensors::backends::MJPEG_HTTPCameraBackend "MJPEG_HTTPCameraBackend", MJPEG over HTTP | |
| wheel encoders | the angle each wheel has turned | one per motor, [rad] | \ref bitsim::actuators::backends::SIM_DCMotorBackend "SIM_DCMotorBackend" | \ref bitsim::actuators::backends::MQTT_DCMotorBackend "MQTT_DCMotorBackend" | `motors` |

The wheel encoders are part of the motors, not sensors of their own, and with
\ref bitsim::estimation::DiffDriveOdometry "DiffDriveOdometry" they give the robot's pose; see
\ref bitsim_sensors_odometry. \ref bitsim::estimation::DiffDriveEKF "DiffDriveEKF" fuses all of
them; see \ref bitsim_sensors_fusion.

What each needs when bitsim is built; \ref bitsim_install says how to build it:

| Backend | Needs |
| ------- | ----- |
| the simulated ultrasonic, infrared and IMU backends | `ENABLE_CHRONO` |
| the MQTT backends | `ENABLE_MQTT` |
| \ref bitsim::sensors::DownwardCamera "DownwardCamera", \ref bitsim::sensors::FloorImage "FloorImage" | `ENABLE_OPENCV` |
| \ref bitsim::sensors::backends::MJPEG_HTTPCameraBackend "MJPEG_HTTPCameraBackend" | `ENABLE_OPENCV`, and an OpenCV built with FFmpeg |
| \ref bitsim::actuators::backends::SIM_DCMotorBackend "SIM_DCMotorBackend", \ref bitsim::estimation::DiffDriveOdometry "DiffDriveOdometry" | nothing; a robot that the motors drive needs `ENABLE_CHRONO` |
| \ref bitsim::actuators::backends::MQTT_DCMotorBackend "MQTT_DCMotorBackend" | `ENABLE_MQTT` |
| \ref bitsim::estimation::DiffDriveEKF "DiffDriveEKF" | nothing |

The examples that use them:

| Example | Sensors |
| ------- | ------- |
| \ref bitsim_example_chrono_ultrasonic_sensor | a simulated ultrasonic sensor on its own |
| \ref bitsim_example_mqtt_ultrasonic_sensor | a real ultrasonic sensor that publishes over MQTT |
| \ref bitsim_example_real_dc_motors | real motors on a board, whose encoders report over MQTT |
| \ref bitsim_example_pi_camera_line_following | the camera stream from a Raspberry Pi, and the robot's DownwardCamera |
| \ref bitsim_example_line_following_infrared_sonar | the infrared elements and the ultrasonic sensor |
| \ref bitsim_example_imu_heading | the IMU, with the sensors of the line following example |
| \ref bitsim_example_wheel_odometry | the wheel encoders for odometry, with the sensors of the IMU example |
| \ref bitsim_example_ekf_sensor_fusion | all of them, the camera, the infrared elements, both ultrasonic sensors, the IMU and the encoders, fused with an extended Kalman filter |

## How a sensor works

A sensor is two objects:

- the **sensor**, a \ref bitsim::sensors::SensorBase "SensorBase": what the application talks to.
  It knows what its values mean, for example
  \ref bitsim::sensors::InfraredSensor::is_dark "InfraredSensor::is_dark" or
  \ref bitsim::sensors::IMUSensor::angular_velocity "IMUSensor::angular_velocity";
- its **backend**, a \ref bitsim::sensors::backends::SensorBackendBase "SensorBackendBase": where
  the values come from, a simulation or a board.

@code{.cpp}
auto sonar = bitsim::sensors::UltrasonicSensor::create("SIMULATION", "front");
sonar->range_backend()->set_max_distance(4.0);
sonar->backend_ptr()->set_chrono_body(chassis);
sonar->init();                                  // enable it

const auto &values = sonar->read_values();      // read the backend
const auto distance = sonar->distance();        // the last values, without reading
@endcode

- `init()` enables the sensor and sizes its values, which hold a placeholder until the first read:
  -1 for the ultrasonic sensor, white for the infrared elements and zeros for the IMU. A disabled
  sensor does not read its backend: `read_values()` returns its last values.
- `read_values()` reads the backend once, now. A sensor does not keep time. The backend's
  \ref bitsim::sensors::backends::SensorBackendBase::sampling_period "sampling_period", the inverse
  of the `update_rate` in a robot file, says how often a real sensor of its kind reads, and the
  application calls `read_values()` that often. The examples keep a `next_read` time per sensor.
- `last_read_values()` gives the values of the last read.

Every sensor type has a `create(backend_type, name)` that makes the backend from its name:

| Name | Backend |
| ---- | ------- |
| `SIMULATION` | the simulated backend. `CHRONO`, its old name, works too |
| `MQTT` | the backend for a real sensor on a board |

### Sensors in a robot file

\ref bitsim::rb::bitsim_chrono::CHRONO_DiffDriveRobot "CHRONO_DiffDriveRobot" reads its sensors
from the `sensors` section of its robot file, `robots/bitsim_diff_drive_robot.json`. It attaches
each one to the chassis, draws it in the Irrlicht window, enables it, and hands it back:

| Section | `type` | Read back with |
| ------- | ------ | -------------- |
| `front-sonar` | `ultrasonic-range` | \ref bitsim::rb::bitsim_chrono::CHRONO_DiffDriveRobot::get_sonar "get_sonar" |
| `rear-sonar` | `ultrasonic-range` | \ref bitsim::rb::bitsim_chrono::CHRONO_DiffDriveRobot::get_rear_sonar "get_rear_sonar" |
| `line-sensor` | `infrared-array` | \ref bitsim::rb::bitsim_chrono::CHRONO_DiffDriveRobot::get_line_sensor "get_line_sensor" |
| `imu` | `imu` | \ref bitsim::rb::bitsim_chrono::CHRONO_DiffDriveRobot::get_imu "get_imu" |
| `front-camera` | `downward-camera` | \ref bitsim::rb::bitsim_chrono::CHRONO_DiffDriveRobot::get_camera "get_camera" |

A section is skipped if it is missing or has `"enable": false`. Each has a `backend`, a `name`,
an `update_rate` in readings per second, and `dimensions` and `color` for the box that shows it.
Positions are in metres from the centre of the chassis, x ahead, y to the left and z up. The
settings of each sensor are below.

## Ultrasonic sensor

An ultrasonic sensor, such as an HC-SR04, sends a burst of sound and times its echo. It reads the
distance to the nearest thing that reflects the sound in a cone in front of it.

**Values:** one, the distance [m], between `min_distance` and `max_distance`. Closer than
`min_distance` it reads `min_distance`; with nothing in range, `max_distance`.

### The simulated sensor

\ref bitsim::sensors::backends::CHRONO_UltrasonicBackend "CHRONO_UltrasonicBackend" casts rays
into the Chrono world with its collision system. The sensor sits at `frame_pos` on its body and
looks along the body's x axis turned by `yaw` about z: 0 ahead, 180 behind. It casts `n_rays` rays fanned evenly across `beam_angle` in the
body's x-y plane, and reads the distance to the nearest hit:

\f[
d = \mathrm{clamp}\left(\min_{k} d_k,\ d_{\min},\ d_{\max}\right),
\qquad \theta_k = \beta \left(\frac{k}{n - 1} - \frac{1}{2}\right),\ k = 0 \dots n-1
\f]

where \f$d_k\f$ is the distance along ray \f$k\f$, at angle \f$\theta_k\f$ from where it looks, to the
first body it hits, and \f$\beta\f$ is the beam angle. With one ray, the default, it looks
straight ahead only.

| Setting | Robot file | Meaning |
| ------- | ---------- | ------- |
| position | `frame_pos` (or `mounted_position`) | where it is on the body [m] |
| direction | `yaw` | where it looks, about the body's z axis from x [deg]; 0 if missing |
| range | `min_distance`, `max_distance` | what it can measure [m]; 0.02 and 4 for an HC-SR04 |
| beam | `beam_angle`, `n_rays` | the width of the cone [deg], and the rays cast across it |
| rate | `update_rate` | readings per second |

The robot file has two: `front-sonar` on the front of the chassis, and `rear-sonar` at
`[-0.27, 0, 0.05]` with `"yaw": 180`, looking behind. Both cast 5 rays across 15 degrees, up to
4 m, 20 times a second.

What the model leaves out:

- **Only bodies with collision enabled** reflect the rays, and never the body the sensor is on. A
  body added to a system that has already stepped is invisible to the rays until it joins the
  collision system: `system.GetCollisionSystem()->BindItem(body)`.
- **The cone is flat**: the rays fan out sideways only, not up or down, so the sensor misses
  anything above or below its height.
- **Echoes are perfect.** A real sensor misses a wall it meets at a steep angle, because the sound
  bounces away, and hears soft or thin things badly. Here every surface a ray touches reflects.
- **No noise, no time of flight, no cross talk.** The reading is exact and instantaneous. A real
  HC-SR04 takes up to about 25 ms a reading, and two of them can hear each other.

### The real sensor

\ref bitsim::sensors::backends::MQTT_UltrasonicBackend "MQTT_UltrasonicBackend" reads an
\ref bitsim::sensors::UltrasoundMessage "UltrasoundMessage":

@code
{"distance": 62.0, "unit": "cm", "timestamp": "2026-11-18 10:15:02"}
@endcode

`unit` is m, cm or mm, and m if it is missing; the backend reads metres either way. `timestamp`,
in UTC, is optional. Until the first message it reads `max_distance`. See \ref bitsim_sensors_mqtt.

## Infrared reflectance elements

An infrared reflectance element, such as a TCRT5000, has an infrared LED and a photo transistor
side by side, a few millimetres above the floor. The transistor measures how much of the LED's
light the floor reflects: most of it for white paper, little for a black line. Line following
robots have a row of them under their front.

**Values:** one per element, the reflectance, from 0 for black to 1 for white. An element whose
value is below the sensor's `threshold`, 0.5 by default, is over something dark:
\ref bitsim::sensors::InfraredSensor::is_dark "is_dark".

### The simulated elements

\ref bitsim::sensors::backends::CHRONO_InfraredBackend "CHRONO_InfraredBackend" carries the
elements on a Chrono body. The floor is a function that gives the reflectance at each point of
the world, and each element reads it at the point straight below it:

\f[
v_i = \mathrm{clamp}\left(F(x_i, y_i),\ 0,\ 1\right), \qquad
(x_i, y_i, z_i) = T_{\text{body}}\, p_i
\f]

where \f$p_i\f$ is where element \f$i\f$ is on the body and \f$T_{\text{body}}\f$ the pose of the
body. Set the floor with
\ref bitsim::sensors::backends::CHRONO_InfraredBackend::set_floor "set_floor", or for a robot with
\ref bitsim::rb::bitsim_chrono::CHRONO_DiffDriveRobot::set_floor "CHRONO_DiffDriveRobot::set_floor".
For a floor that is a picture, \ref bitsim::sensors::FloorImage::reflectance "FloorImage::reflectance"
gives the brightness of the pixel at a point, its luma divided by 255, and white outside the
picture:

@code{.cpp}
robot.set_floor([&floor](real_t x, real_t y) { return floor.reflectance(x, y); });
@endcode

| Setting | Robot file | Meaning |
| ------- | ---------- | ------- |
| elements | `elements` | [[x, y, z], ...], where they are on the body, in the order of the values [m] |
| threshold | `threshold` | below it an element is over something dark |
| rate | `update_rate` | readings per second |

What the model leaves out:

- **The spot is a point.** A real element sees a spot a few millimetres wide, so its value changes
  gradually across the edge of a line; here it jumps from one pixel to the next.
- **No height or tilt.** A real element reads less the higher it is above the floor; here the
  height does not matter, and neither does the tilt of the body.
- **No ambient light, no noise, no differences between elements.** Real elements are lit by the
  room too, read with noise, and each needs calibrating against white and black.

### The real elements

\ref bitsim::sensors::backends::MQTT_InfraredBackend "MQTT_InfraredBackend" reads an
\ref bitsim::sensors::InfraredMessage "InfraredMessage" with one value per element, in the order of
`elements`:

@code
{"values": [0.91, 0.12, 0.88]}
@endcode

The board divides each ADC reading by the reading over white paper. Values outside [0, 1] are
clamped, and a message with a different number of values than the backend has elements is
ignored. Until the first message every element reads 1, white. See \ref bitsim_sensors_mqtt.

## IMU

An inertial measurement unit, such as an MPU-6050, has a three axis accelerometer and a three axis
gyroscope, along the axes of its board.

**Values:** six, along the axes of the body the IMU is on.

| Index | Value | Unit |
| ----- | ----- | ---- |
| 0, 1, 2 | the specific force: the acceleration minus gravity. At rest it is 9.81 up | m/s² |
| 3, 4, 5 | the angular velocity | rad/s |

\ref bitsim::sensors::IMUSensor::acceleration "acceleration" and
\ref bitsim::sensors::IMUSensor::angular_velocity "angular_velocity" pick them out of the last read.

### The simulated IMU

\ref bitsim::sensors::backends::CHRONO_IMUBackend "CHRONO_IMUBackend" takes the motion of its body
from Chrono. The IMU sits at \f$r\f$ from the centre of the body, so it accelerates by

\f[
a_P = a + \alpha \times r + \omega \times (\omega \times r)
\f]

where \f$a\f$ is the acceleration of the centre of the body and \f$\omega\f$, \f$\alpha\f$ its
angular velocity and acceleration. With \f$R\f$ the rotation from the body to the world and
\f$g\f$ the gravity of the system, it reads

\f[
f = R^T (a_P - g) + n_a, \qquad \omega_m = R^T \omega + b_g + n_g
\f]

where \f$b_g\f$ is a fixed gyroscope bias and \f$n_a\f$, \f$n_g\f$ are normal noise, new for every
reading, with standard deviations \f$\sigma_a\f$ and \f$\sigma_g\f$. The noise comes from a
generator with a fixed seed, so a simulation repeats.

| Setting | Robot file | Meaning |
| ------- | ---------- | ------- |
| position | `frame_pos` | where it is on the body [m]; its axes are the body's |
| \f$b_g\f$ | `gyro_bias` | what the gyroscope reads when still [rad/s] |
| \f$\sigma_g\f$ | `gyro_noise` | [rad/s] |
| \f$\sigma_a\f$ | `accel_noise` | [m/s²] |
| | `seed` | of the noise generator |
| rate | `update_rate` | readings per second |

The robot file has values like an MPU-6050's after its factory calibration: a bias of a few
mrad/s, which is a fraction of a degree a second, and a few mrad/s and 0.04 m/s² of noise at 100
readings a second. Integrated, the bias makes a heading drift by \f$b_g\,t\f$, and the noise adds a
random walk that grows with \f$\sigma_g \sqrt{\Delta t\, t}\f$; \ref bitsim_example_imu_heading measures the
bias and takes it off.

What the model leaves out:

- **The bias is fixed.** A real bias wanders slowly, with temperature and over time, so a
  calibration at the start does not hold for ever.
- **No scale factor, misalignment, cross coupling or saturation.** A real gyroscope reads a little
  more or less than the true rate, its axes are not quite square to each other or to the body, and
  an MPU-6050 saturates at its range, ±250 deg/s and ±2 g by default.
- **The accelerometer feels the contacts.** \f$a\f$ is the acceleration that Chrono's contact
  solver gave the body in its last step. Wheels on the ground make it jitter: with no noise added,
  the robot's IMU reads about 0.1 m/s² of jitter standing still and 0.2 m/s² driving straight,
  with single readings up to about 1.3 m/s² off. Its mean is right: in a turn at 0.3 m/s and
  1 rad/s, the y axis reads 0.30 m/s² on average, the \f$v\,\omega\f$ of the turn.

### The real IMU

\ref bitsim::sensors::backends::MQTT_IMUBackend "MQTT_IMUBackend" reads an
\ref bitsim::sensors::IMUMessage "IMUMessage":

@code
{"accel": [0.12, -0.03, 9.79], "gyro": [0.001, -0.002, 0.35]}
@endcode

in m/s² and rad/s, along the axes of the IMU, which the board should line up with the robot's.
Many IMUs report in g and deg/s, so the board converts. Until the first message the backend reads
zeros. See \ref bitsim_sensors_mqtt.

\anchor bitsim_sensors_odometry
## Wheel encoders and odometry

An encoder on a wheel's motor counts how far the wheel turns: a disc with slots, or magnets, on the
shaft, and a sensor that counts them going by. The DC gear motors of the robot have one with 660
counts a turn of the wheel, so it measures the wheel's angle in steps of \f$2\pi/660\f$, about
half a degree. On a real robot the encoder is part of the motor and its board, so in bitsim it is
part of the motor too: it is not a \ref bitsim::sensors::SensorBase "SensorBase".

**Values:** \ref bitsim::actuators::DCMotor::encoder_angle "DCMotor::encoder_angle", the angle of
the wheel as the encoder measures it [rad], counted from where it started. Read it for each wheel:

@code{.cpp}
const real_t left = robot.get_motor(CHRONO_DiffDriveRobot::LEFT)->encoder_angle();
const real_t right = robot.get_motor(CHRONO_DiffDriveRobot::RIGHT)->encoder_angle();
@endcode

### The simulated encoder

\ref bitsim::actuators::backends::SIM_DCMotorBackend "SIM_DCMotorBackend" takes the angle of the
wheel from Chrono, the angle of the joint between the wheel and the chassis, and rounds it down to
whole counts:

\f[
\theta_{enc} = \frac{2\pi}{N} \left\lfloor \frac{N \theta}{2\pi} \right\rfloor
\f]

with \f$N\f$ the counts a turn, `encoder_counts` in the `motors` section of the robot file. Its
speed controller measures the speed from the same counts.

What the model leaves out: it counts perfectly. A real encoder can miss counts at high speed or
pick up noise as extra counts, and a cheap one has slots that are not evenly spaced.

### The real encoder

\ref bitsim::actuators::backends::MQTT_DCMotorBackend "MQTT_DCMotorBackend" takes the angle from
the reports of the board that drives the motors,
\ref bitsim::boards::arduino::DCArduinoBoardImpl "DCArduinoBoardImpl":

@code
{"ack": 12, "left_distance": 0.53, "right_distance": 0.52, "left_speed": 2.49, "right_speed": 2.51, "t_ms": 81234}
@endcode

The board turns its counts into the distance each wheel has rolled [m], and the backend divides by
the wheel radius. The encoder angle is the angle the board last reported, counted from its first
report; unlike `angle()`, which the backend smooths and predicts between reports for Chrono, it is
not filtered. See \ref bitsim_example_real_dc_motors.

### Odometry

\ref bitsim::estimation::DiffDriveOdometry "DiffDriveOdometry" works out the pose of a
differential drive robot from its encoders, by dead reckoning. Between two readings the left and
right wheels turn by \f$\Delta\theta_L\f$ and \f$\Delta\theta_R\f$. With wheel radius \f$r\f$ and
the wheels \f$L\f$ apart, the robot drives and turns by

\f[
\Delta s = \frac{r\,(\Delta\theta_L + \Delta\theta_R)}{2}, \qquad
\Delta\psi = \frac{r\,(\Delta\theta_R - \Delta\theta_L)}{L}
\f]

and its pose moves along the heading halfway through the step:

\f[
x \mathrel{+}= \Delta s \cos\bar\psi, \qquad y \mathrel{+}= \Delta s \sin\bar\psi, \qquad
\bar\psi = \psi + \tfrac{1}{2}\Delta\psi
\f]

The heading comes from the wheels, `update(left, right)`, or from another sensor,
`update(left, right, heading)`, such as the gyroscope of the IMU above; then
\f$\bar\psi\f$ is the mean of the last two headings. Start it with `reset(pose, left, right)` from
where the robot is placed. `distance()` gives the length of the path driven.

@code{.cpp}
DiffDriveOdometry odometry(data.wheel_radius, data.axle_length);
odometry.reset({x0, y0, yaw0}, left->encoder_angle(), right->encoder_angle());
// every reading
odometry.update(left->encoder_angle(), right->encoder_angle(), gyro_heading);
@endcode

Odometry adds up small steps, so its errors never go away:

| Error | What it does |
| ----- | ------------ |
| counting | each reading is up to a count short. That does not add up: the next reading counts from the true angle again |
| slip | a wheel that slips turns without moving the robot as far. Chrono makes the simulated wheels slip as real ones would; on the robot's floor, with friction 0.8 and at 0.3 m/s, they hardly do |
| wheel radius | a radius off by a percent makes every distance off by a percent, whichever the heading comes from |
| axle length | an axle length off by a percent makes every turn off by a percent, so the heading from the wheels drifts: after a full turn by \f$2\pi\f$ times that. It is hard to know better than a percent or two, because a tyre touches the floor over its width |

A heading error is the worst, as every step after it goes a little the wrong way and the position
error grows with the distance driven. A gyroscope's heading has none of the wheels' errors, so
robots take their distance from the wheels and their heading from a gyroscope.
\ref bitsim_example_wheel_odometry runs both over a lap of 14 m:

| Odometry | Exact geometry | Axle length 2 % off |
| -------- | -------------- | ------------------- |
| heading from the gyroscope | 3.0 cm off | 3.0 cm off |
| heading from the wheels | 0.7 cm off, heading 0.08 deg off | 24 cm off, heading 7.1 deg off |

With exact geometry and wheels that hardly slip, the simulation favours the wheels; with an axle
length that is a little off, as on a real robot, the gyroscope wins.

What `DiffDriveOdometry` leaves out: it gives a pose, not how uncertain the pose is. A filter
that combines odometry with other sensors needs that too; see \ref bitsim_sensors_fusion.

## Downward camera

\ref bitsim::sensors::DownwardCamera "DownwardCamera" is a camera on a robot that looks straight
down at the floor ahead of it. It is simulated only. The floor is a
\ref bitsim::sensors::FloorImage "FloorImage": a picture centred on the world origin that covers a
given width, its columns along x and its top row the side with the largest y.

**Values:** an image of `image_width` × `image_height` pixels of the rectangle of floor,
`view_length` × `view_width` metres, centred `offset` metres ahead of the robot's axle. Its top row
is the far edge and its first column the robot's left, as a camera facing forward and tilted down
would show them. \ref bitsim::sensors::DownwardCamera::to_robot "to_robot" turns a pixel into metres
ahead of and to the left of the robot.

The model: the pose of the robot gives an affine map from the camera's pixels to the floor's
pixels, and the camera samples the picture with it. Floor outside the picture is white. So the
camera has no perspective, lens distortion, exposure or motion blur, and it sees only the floor:
bodies in the Chrono world do not show in it.

| Setting | Robot file | Meaning |
| ------- | ---------- | ------- |
| | `offset` | how far ahead of the axle centre the middle of the view is [m] |
| | `view_length`, `view_width` | the floor it sees, along and across the robot [m] |
| | `image_width`, `image_height` | the size of its image [pixels] |
| rate | `update_rate` | images per second |

It is not a \ref bitsim::sensors::SensorBase "SensorBase": call
\ref bitsim::rb::bitsim_chrono::CHRONO_DiffDriveRobot::camera_view "camera_view" for an image. See
\ref bitsim_example_pi_camera_line_following.

## Camera stream

\ref bitsim::sensors::backends::MJPEG_HTTPCameraBackend "MJPEG_HTTPCameraBackend" receives the
frames of a real camera as an MJPEG stream over HTTP, the `multipart/x-mixed-replace` response
that many cameras and camera servers give, such as the Raspberry Pi server of
\ref bitsim_example_pi_camera_line_following. A thread of its own reads the stream and keeps only the latest frame, so a
slow reader never falls behind the camera. If the stream breaks, it opens it again.

| Setting | `Config` | Meaning |
| ------- | -------- | ------- |
| | `url` | the stream, for example `http://raspberrypi.local:8000/odysseus/camera` |
| | `timeout` | the stream is alive while frames arrive at most this far apart [s] |
| | `reconnect_delay` | how long to wait before opening it again [s] |
| | `io_timeout` | give up opening it, or reading a frame, after this long [s] |

It reads the stream with OpenCV's FFmpeg backend, and an OpenCV built without FFmpeg gets no
frames; \ref bitsim::sensors::backends::MJPEG_HTTPCameraBackend::can_read_streams "can_read_streams"
tells. Ubuntu's `libopencv-dev` has FFmpeg.

\anchor bitsim_sensors_fusion
## Fusing the sensors

Each sensor alone has its faults: odometry drifts, the line says nothing about where the robot is
along it, and a sonar hears whatever is nearest. Together they can do better.
\ref bitsim::estimation::DiffDriveEKF "DiffDriveEKF" is an extended Kalman filter for the
differential drive robot that uses them all. Its state is the centre of the axle, the heading and
the bias of the gyroscope, \f$(x, y, \psi, b)\f$, with its covariance.

| Sensor | What the filter does with it | Call |
| ------ | ---------------------------- | ---- |
| encoders and gyroscope | **predict**: the distance from the wheels and the turn from the gyroscope less its bias, \f$\Delta\psi = (g - b)\,\Delta t\f$ | `predict(distance, gyro_z, dt)` |
| infrared elements | **correct**: where the line is under them, \ref bitsim::sensors::InfraredSensor::line_position "line_position()", should be on the track | `update_line(points, sigma)` |
| downward camera | **correct**: where the line crosses bands of its image, \ref bitsim::sensors::DownwardCamera::line_points "line_points()", should be on the track | `update_line(points, sigma)` |
| ultrasonic sensors | **correct**: each should read the distance to the nearest wall or box of the map along its beam | `update_range(sensor, distance)` |

The filter knows the floor and the room from a \ref bitsim::estimation::DiffDriveMap "DiffDriveMap":
the track, a \ref bitsim::trajectory::RoundedRectangle "RoundedRectangle", and the walls and boxes,
as rectangles. A line point at the robot's estimated pose has a signed distance to the track, which
the filter takes the sensor to measure as 0. A range is the shortest of rays cast through the map
across the beam, as the simulated sensor casts them. A reading whose innovation is too large for
its covariance, \f$\nu^T S^{-1} \nu\f$ over 9 per value, is kept out by a gate, and a sonar that
hears nothing, or whose map says there is nothing in range, is not used.

@code{.cpp}
using bitsim::estimation::DiffDriveEKF;
DiffDriveEKF ekf(map, {x0, y0, yaw0}, DiffDriveEKF::Parameters{});
// where it is from the centre of the axle and where it looks, its beam, rays and range
const DiffDriveEKF::RangeSensor front{0.37, 0.0, 0.0, 15.0 * PI / 180.0, 5, 0.02, 4.0};

ekf.predict(distance, gyro_z, dt);                    // every reading of the IMU
ekf.update_line(camera.line_points(view), 0.01);      // every image
const auto result = ekf.update_range(front, sonar.distance());
if (!result.accepted) { /* the gate kept it out, or there was nothing to use */ }
@endcode

The filter is built on \ref bitsim::estimation::ExtendedKalmanFilter "ExtendedKalmanFilter", which
predicts with a motion model and updates with any number of observation models, each with its own
noise and gate; \ref bitsim_example_ekf_sensor_fusion_mqtt runs it on readings that come over
MQTT. \ref bitsim_example_ekf_sensor_fusion drives a lap of the track with odometry and
the filter side by side. Off the true position, at most over the lap:

| Run | Odometry | The filter |
| --- | -------- | ---------- |
| as it is | 3.5 cm | 2.6 cm |
| wheel radius taken 3 % too big | 13 cm | 3.1 cm |
| the same, without the sonars | 13 cm | 10 cm |
| the same, with the sonars only | 13 cm | 2.8 cm |
| the gyroscope's bias not measured first | 40 cm | 2.7 cm |

The line fixes the position across the track and the heading, the sonars the position along it,
and the filter learns the gyroscope's bias as it drives, 0.0032 rad/s against the true 0.0030.

\anchor bitsim_sensors_mqtt
## Real sensors over MQTT

A real sensor sits on a board, an Arduino or a Raspberry Pi, that publishes each reading as JSON
on a topic of its own. Each MQTT backend reads its topic through an
\ref bitsim::sensors::backends::MQTT_SensorChannel "MQTT_SensorChannel", so they all behave alike:

- **Reading never waits.** `read_values()` takes every message that has arrived since the last
  read and keeps the latest valid one, so an application that reads often keeps up with a board
  that publishes often. Invalid messages are ignored and counted in `n_rejected()`.
- **Until the first message** a backend reads a neutral value: `max_distance`, white, or zeros.
  So check **`is_alive()`**, which is true while valid messages arrive at most `timeout` apart,
  0.5 s by default, before trusting a reading. A board that goes quiet leaves its last reading.
- **Connect before reading.** A backend makes its own subscriber from its `broker` and `topic`
  when `connect()` is called, or reads a \ref bitsim::network::MqttSubscriber "MqttSubscriber" you
  connected and pass to its constructor. Reading a backend that is not connected throws, so a
  forgotten `connect()` does not look like a clear path.
- **`receive(text)`** hands a backend a message without a broker, for tests.
- **Timestamps** are optional: `"timestamp": "2026-11-18 10:15:02"`, in UTC. Without one, the
  message counts from when it arrived.

In a robot file, give the sensor `"backend": "MQTT"` and its `broker` and `topic`, then connect it:

@code{.json}
"front-sonar": {"backend": "MQTT", "broker": "tcp://localhost:1883", "topic": "odysseus/sonar", ...}
@endcode

@code{.cpp}
using bitsim::sensors::backends::MQTT_UltrasonicBackend;
auto sonar = std::dynamic_pointer_cast<MQTT_UltrasonicBackend>(robot.get_sonar()->range_backend());
sonar->connect();
@endcode

With a broker on your computer you can stand in for the board:

@code{.sh}
mosquitto_pub -t odysseus/sonar -m '{"distance": 35, "unit": "cm"}'
@endcode

Every MQTT client after the first in a program takes about 2 s to connect; that is Paho's, not
bitsim's. A program with a few sensors spends some seconds starting.

## Other pieces

| Piece | What it is |
| ----- | ---------- |
| \ref bitsim::sensors::SensorManager "SensorManager" | keeps a list of sensors; `update()` reads all the enabled ones |
| \ref bitsim::sensors::OdometryMessage "OdometryMessage" | `{"x": ..., "y": ..., "yaw": ...}`, a pose estimate from a board, with an optional UTC `timestamp` |
| \ref bitsim::sensors::CameraMessage "CameraMessage" | a base64 JPEG image, for a camera that publishes over MQTT |
| \ref bitsim::sensors::EigenVectorMessage "EigenVectorMessage" | a vector of numbers as `[1.0, 2.0, ...]` |

## Adding a sensor

To add a kind of sensor, follow the three that are there:

1. A message in `sensors/messages` with a static `parse` that returns `std::nullopt`, not an
   exception, for a message it cannot read.
2. A backend base in `sensors/backends`, derived from
   \ref bitsim::sensors::backends::SensorBackendBase "SensorBackendBase", with the settings common
   to every backend, a `load(json)` that reads them, and a `create(backend_type)`.
3. A `CHRONO_` backend that models the sensor from its body, and an `MQTT_` backend that reads the
   message through an \ref bitsim::sensors::backends::MQTT_SensorChannel "MQTT_SensorChannel".
4. The sensor, derived from \ref bitsim::sensors::SensorBase "SensorBase", with accessors that say
   what its values mean.
5. A section in the robot file, read in `CHRONO_DiffDriveRobot::RobotSensorManager::read`, and a
   getter on the robot.
6. Tests, and a section on this page: what it reads, its model and what the model leaves out, and
   its message.
