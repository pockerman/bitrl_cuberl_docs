\page bitsim_actuators_guide Actuators

bitsim's actuators are the motors that move a robot. Like its sensors, see \ref bitsim_sensors_guide, a
motor can be **simulated**, a model of the motor that runs with the simulation, or **real**, a motor on a
board that the application talks to over MQTT. The application drives both the same way, so the code that
moves a simulated robot moves one with real motors too.

## The actuators at a glance

| Actuator | What it is | Simulated by | Real, over | Robot file |
| -------- | ---------- | ------------ | ---------- | ---------- |
| \ref bitsim::actuators::DCMotor "DCMotor" | a brushed DC gear motor with an encoder, in speed control | \ref bitsim::actuators::backends::SIM_DCMotorBackend "SIM_DCMotorBackend" | \ref bitsim::actuators::backends::MQTT_DCMotorBackend "MQTT_DCMotorBackend", on a \ref bitsim::boards::DCMotorBoardBase "DCMotorBoardBase" | `motors` |
| \ref bitsim::actuators::BLDCMotor "BLDCMotor" | the electrical model of a three-phase brushless motor with six-step commutation from Hall sensors | itself | | |

What each needs when bitsim is built; \ref bitsim_install says how to build it:

| Class | Needs |
| ----- | ----- |
| \ref bitsim::actuators::DCMotor "DCMotor" with \ref bitsim::actuators::backends::SIM_DCMotorBackend "SIM_DCMotorBackend", \ref bitsim::actuators::BLDCMotor "BLDCMotor" | nothing; a robot that the motors drive needs `ENABLE_CHRONO` |
| \ref bitsim::actuators::backends::MQTT_DCMotorBackend "MQTT_DCMotorBackend", \ref bitsim::boards::arduino::DCArduinoBoardImpl "DCArduinoBoardImpl" | `ENABLE_MQTT` |

The examples that use them:

| Example | Actuators |
| ------- | --------- |
| \ref bitsim_example_bldc_motor | a brushless motor in the shoulder of an arm, with its commutation and its current |
| \ref bitsim_example_chrono_diff_drive | the two simulated wheel motors of the differential drive robot |
| \ref bitsim_example_real_dc_motors | two real DC gear motors on an Arduino, driving the simulated robot: hardware in the loop |

## The DC motor

A \ref bitsim::actuators::DCMotor "DCMotor" runs in **speed control**: the application sets the speed its
output shaft should turn at, and reads back the angle and the speed it turns at. What the motor is depends
on its **backend**, a \ref bitsim::actuators::backends::DCMotorBackendBase "DCMotorBackendBase", as a sensor
depends on its backend.

@code{.cpp}
using bitsim::actuators::DCMotor;
using bitsim::actuators::backends::DCMotorParameters;

auto motor = DCMotor::create_simulated(DCMotorParameters(), "left");
motor->set_max_speed(20.0);
motor->set_speed(10.0);          // the target speed of the output shaft [rad/s], clamped to max_speed

for (int i = 0; i < 1000; ++i)
{
    motor->step(0.001);          // advance the motor by dt [s]
}

const auto angle = motor->angle();           // [rad]
const auto speed = motor->speed();           // [rad/s]
const auto counts = motor->encoder_angle();  // what the encoder measures: the angle in whole counts
@endcode

- `step(dt)` advances the motor by `dt`. Call it once per simulation step.
- `step(dt, shaft)` is for a motor that drives a body in a physics engine: the engine moves the shaft, and
  passes its angle and speed, a \ref bitsim::actuators::ShaftState "ShaftState".
- `torque()` is the torque on the output shaft, and `is_ready()` says whether the motor can be used: a
  simulated motor always can, a real one once its board has reported.

### Torque or angle

The two kinds of backend couple to a physics engine differently, and `output_type()` says which:

| Output type | Backend | What the engine does |
| ----------- | ------- | -------------------- |
| `TORQUE` | the simulated motor computes the torque it produces | applies `torque()` to the wheel, and the wheel turns as the physics says |
| `ANGLE` | the real motor has already turned | turns the wheel to the motor's `angle()` |

\ref bitsim::rb::bitsim_chrono::CHRONO_DiffDriveRobot "CHRONO_DiffDriveRobot" connects each wheel to its
motor in the way the motor's output type asks for, so the same robot runs with either kind.

### The simulated motor

\ref bitsim::actuators::backends::SIM_DCMotorBackend "SIM_DCMotorBackend" models a DC gear motor, its encoder
and its speed controller. The electrical part is

\f[
L \frac{di}{dt} = V - R i - K_e \omega_r , \qquad \omega_r = N \omega ,
\f]

with the rotor speed \f$\omega_r\f$ the gear ratio \f$N\f$ times the speed of the output shaft
\f$\omega\f$, and the motor puts the torque \f$N K_e i\f$ on the output shaft. Its controller runs every
`control_period`, as a board would: it measures the speed from the encoder counts, filters it, and sets the
duty cycle of the voltage with a PI controller and a feedforward of the back-EMF. The parameters,
\ref bitsim::actuators::backends::DCMotorParameters "DCMotorParameters", and their defaults:

| Parameter | Default | What |
| --------- | ------- | ---- |
| `resistance` | 7 Ω | armature resistance \f$R\f$ |
| `inductance` | 2 mH | armature inductance \f$L\f$ |
| `emf_constant` | 0.0127 V s/rad | back-EMF and torque constant \f$K_e\f$, at the rotor |
| `rotor_inertia` | 10⁻⁶ kg m² | the inertia of the rotor |
| `friction` | 10⁻⁶ N m s/rad | viscous friction at the rotor |
| `gear_ratio` | 30 | rotor turns per output turn, \f$N\f$ |
| `supply_voltage` | 12 V | \f$V\f$ at full duty cycle |
| `speed_kp`, `speed_ki` | 0.03, 0.1 | the gains of the speed controller, duty cycle per rad/s and per rad of error |
| `speed_filter` | 0.02 s | the time constant of the filter on the measured speed |
| `control_period` | 0.01 s | how often the controller runs |
| `encoder_counts` | 660 | encoder counts per output turn |

With the defaults the motor turns at most \f$V / (N K_e) \approx 31.5\f$ rad/s with no load, and its stall
torque is \f$N K_e V / R \approx 0.65\f$ N m; `no_load_speed()` and `stall_torque()` give them.

What the model leaves out: the voltage is limited to the supply, but the current is not, and the driver
has no dead band; the gears have no backlash and no losses; the encoder is ideal apart from its counts.

### The real motor

\ref bitsim::actuators::backends::MQTT_DCMotorBackend "MQTT_DCMotorBackend" is a motor on a board, a
\ref bitsim::boards::DCMotorBoardBase "DCMotorBoardBase". One board usually drives several motors over one
connection, so the motors share it, each on its own channel:

@code{.cpp}
using bitsim::boards::arduino::DCArduinoBoardImpl;
using bitsim::actuators::backends::MQTT_DCMotorBackend;

auto board = std::make_shared<DCArduinoBoardImpl>(DCArduinoBoardImpl::Config());
board->connect();

auto left = std::make_shared<DCMotor>(std::make_shared<MQTT_DCMotorBackend>(board, 0), "left");
auto right = std::make_shared<DCMotor>(std::make_shared<MQTT_DCMotorBackend>(board, 1), "right");
@endcode

The target speed goes to the board, and the angle comes back from its encoder. The board reports every few
tens of milliseconds, while a simulation steps every millisecond or two, so in between the backend predicts
the angle from the last report and the speed. A physics engine that follows the angle differentiates it
twice, so every correction of the prediction would be a spike of torque; `angle()` is instead a smooth
estimate, a second-order filter, 30 rad/s wide by default, pulled towards the predicted angle and the
reported speed. `encoder_angle()` and `reported_angle()` give what the board last reported. Angles count
from the first report, so the shaft starts at zero.

\ref bitsim::boards::arduino::DCArduinoBoardImpl "DCArduinoBoardImpl" is an Arduino that drives the motors and talks
MQTT. The application publishes the target speeds on the command topic, `bitsim/dc_board/command` by
default, and the board publishes on the state topic, `bitsim/dc_board/state`, the distance each wheel has
covered:

@code
command: {"seq": 12, "left_speed": 2.5, "right_speed": 2.5}
state:   {"ack": 12, "left_distance": 0.53, "right_distance": 0.52,
          "left_speed": 2.49, "right_speed": 2.51, "t_ms": 81234}
@endcode

Speeds are wheel speeds in rad/s, and distances are in m. The keys start with the names of the channels,
`left` and `right` by default. The board turns its encoder counts into distances with its wheel radius, so
the `wheel_radius` of the `Config` must match the one in the board's sketch. `ack` is the number of the last
command the board received, which gives the round trip of the messages, and a board that has not reported
for `timeout`, 0.5 s by default, is taken to be gone. \ref bitsim_example_real_dc_motors builds the whole
loop, with the sketch for the board.

### Motors in a robot file

\ref bitsim::rb::bitsim_chrono::CHRONO_DiffDriveRobot "CHRONO_DiffDriveRobot" reads its two wheel motors
from the `motors` section of its robot file, `robots/bitsim_diff_drive_robot.json`. Both are alike: the
section gives the backend, `SIMULATION`, and the parameters of the table above. For real motors, make MQTT
motors as above and give them to the robot with `set_motor(m, motor)`; the robot reconnects the wheel in the
way the new motor's output type asks for. See \ref bitsim_robots_guide.

## The brushless motor

\ref bitsim::actuators::BLDCMotor "BLDCMotor" is the electrical model of a three-phase brushless motor with
six-step commutation from three Hall sensors, the motor of most robot joints and drones. The angle and the
speed of its rotor come from outside, so the same model works with a simple rotor,
\ref bitsim::actuators::BLDCRotor "BLDCRotor", and with a rigid body in a physics engine. Every step it
works out the sector from the Hall sensors, connects the two phases of the commutation table in series,
advances their current and returns the torque:

@code{.cpp}
using bitsim::actuators::BLDCMotor;
using bitsim::actuators::BLDCMotorParameters;
using bitsim::actuators::BLDCRotor;

BLDCMotorParameters parameters;
BLDCMotor motor(parameters);
BLDCRotor rotor;

for (int i = 0; i < 100000; ++i) // 0.1 s
{
    const auto torque = motor.step(24.0, rotor.theta, rotor.w, 1.0e-6); // V, angle, speed, dt
    rotor.step(parameters, torque, 0.0, 1.0e-6);                         // no load
}
@endcode

The back-EMF of each phase is sinusoidal, scaled so that its average over a sector matches the
DC-equivalent model, \ref bitsim::actuators::DCEquivalentState "DCEquivalentState", which treats the motor as
a brushed one. The `hall_offset` of the constructor misplaces the Hall sensors, to show what a controller
that sees the wrong sector does. \ref bitsim_example_bldc_motor explains the motor, compares the two models
and puts one in the shoulder of an arm.
