# bitrl & cuberl documentation

`bitrl` is an effort to provide implementations and wrappers of environments suitable for training reinforcement learning agents
using  C++. You can find a very basic example here \ref bitrl_example_1. 

`cuberl` is an extension of bitrl that provides implementations of various reinforcement learning algorithms. 

Gymnasium environments exposed over a REST like API can be found at: <a href="https://github.com/pockerman/bitrl-rest-api">bitrl-rest-api</a>.
Various RL algorithms using the environments can be found at <a href="https://github.com/pockerman/cuberl/tree/master">cuberl</a>.

## Dependencies

Below are the dependencies for the common dependencies for both libraries

- CMake
- Boost
- Eigen3
- Blas

`bitrl` has the following dependencies: 

- OpenCV (optional)
- PahoMqttCpp (optional)
- Chrono (optional)

In addition the library uses 

- <a href="https://github.com/nlohmann/json">json</a> (extern/nlhmann)
- <a href="https://github.com/elnormous/HTTPRequest">HTTPRequest</a> (extern/)

`cuberl` has the following dependencies:

- OpenCV (optional)
- PahoMqttCpp (optional)
- Pytorch (optional)
- Chrono (optional)

## Installation

The usual cmake installation/build can be used:

@code
mkdir build && cd build
cmake -DCMAKE_INSTALL_PREFIX=/path/where/bitrl/should/be/installed/to ..
make install -j4
@endcode

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

