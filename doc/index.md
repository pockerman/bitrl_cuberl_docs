# bitrl & cuberl documentation

`bitrl` is an effort to provide implementations and wrappers of environments suitable for training reinforcement learning agents
using  C++. You can find a very basic example here \ref bitrl_example_1. 

`cuberl` is an extension of bitrl that provides implementations of various reinforcement learning algorithms. 

Gymnasium environments exposed over a REST like API can be found at: <a href="https://github.com/pockerman/bitrl-rest-api">bitrl-rest-api</a>.
Various RL algorithms using the environments can be found at <a href="https://github.com/pockerman/cuberl/tree/master">cuberl</a>.

## Installation

- bitrl: \ref bitrl_install lists the dependencies and explains how to build and install the library.
- cuberl needs bitrl and, optionally, OpenCV, PahoMqttCpp, PyTorch and Chrono. See the
  <a href="https://github.com/pockerman/cuberl/tree/master">cuberl README</a>.

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

