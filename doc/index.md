# bitsim & bitrl documentation

Two libraries are documented here.

- <a href="https://github.com/pockerman/bitsim">bitsim</a> is a C++20 library to model, simulate,
  estimate and control robots, from the maths of frames and kinematics to a simulated robot that drives
  with real motors and a real camera. Its CMake package and C++ namespace are still called `bitrl`.
  You can find a very basic example in \ref bitrl_example_basic_types, and all the examples, in order,
  in \ref bitrl_examples.
- <a href="https://github.com/pockerman/bitrl">bitrl</a> builds on bitsim for reinforcement learning.
  It has the environments, among them grid worlds, Connect2, multi armed bandits, Chrono robots and
  Gymnasium environments, and implementations of reinforcement learning algorithms. Its C++ namespace is
  still `cuberl`. The Gymnasium environments run on the
  <a href="https://github.com/pockerman/bitrl-rest-api">bitrl-rest-api</a> server, and bitrl talks to
  them over HTTP.

## Installation

- bitsim: \ref bitrl_install lists the dependencies and explains how to build and install the library.
- bitrl needs bitsim, libtorch and TBB. See the
  <a href="https://github.com/pockerman/bitrl">bitrl README</a>.

## Sensors

\ref bitrl_sensors_guide lists the sensors bitsim supports: what each reads, its simulated and real
backends, how the simulated ones are modelled and what the models leave out, and the MQTT messages
of the real ones.

## Build the documentation

The documentation is maintained on a different repository. However, you can build the documentation locally.
You will need <a href="https://www.doxygen.nl/">Doxygen</a> and <a href="https://www.graphviz.org/">graphviz</a> installed:

@code
sudo apt install doxygen graphviz
@endcode

Clone the repository that hosts the documentation:

@code
git clone https://github.com/pockerman/bitrl_cuberl_docs
@endcode

Update the submodules for the two libraries:

@code
git submodule init
git submodule update
@endcode

Execute doxygen

@code
doxygen Doxyfile
@endcode


The documentation will be installed under `./docs/html` under the project's source directory


## Examples

➡ \ref examples

