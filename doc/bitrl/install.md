\page bitrl_install Installing bitrl

bitrl is built on bitsim, so bitsim comes first. Ubuntu 24.04 is the reference platform: the CI builds bitrl
there, with the scripts in bitrl's `ci/` directory that this page mentions.

## Dependencies

| Dependency | Needed for |
| ---------- | ---------- |
| A C++20 compiler, such as g++ 11, and CMake 3.20 or newer | everything |
| <a href="https://github.com/pockerman/bitsim">bitsim</a> 5, and what it needs: Boost, Eigen, spdlog | everything; see \ref bitsim_install |
| <a href="https://pytorch.org/get-started/locally/">libtorch</a>, built with the C++11 ABI | everything; the deep reinforcement learning |
| TBB, `sudo apt-get install libtbb-dev` | everything |
| <a href="https://github.com/google/googletest">GoogleTest</a> | `ENABLE_TESTS` |
| The <a href="https://github.com/pockerman/bitrl-rest-api">bitrl-rest-api</a> server | running the Gymnasium environments, not building |

HTTPRequest and nlohmann/json come with bitsim. bitsim installs as the CMake package `bitsim`;
`ci/install_bitsim.sh <prefix>` builds and installs it, and `ci/install_libtorch.sh <prefix>` downloads
libtorch, the way the CI does.

## Build

Tell CMake where bitsim and libtorch are:

@code{.sh}
cmake -B build -DCMAKE_PREFIX_PATH="/path/to/bitsim/prefix;/path/to/libtorch"
cmake --build build --parallel 3
@endcode

Give `--parallel` a number. The files that include libtorch take a lot of memory to compile, and an
unlimited build runs out of memory on a 16 GB machine. If bitsim was built with Chrono, and Chrono is not in
a system prefix, add its prefix to `CMAKE_PREFIX_PATH` too.

| Option | Default | Description |
| ------ | ------- | ----------- |
| `ENABLE_EXAMPLES` | `ON` | Build the examples, see \ref bitrl_examples |
| `ENABLE_TESTS` | `OFF` | Build the tests |
| `USE_CUDA` | `OFF` | Use CUDA; libtorch must be built with CUDA too |
| `BITRL_LOG` | `ON` | Library logging |
| `BITRL_WERROR` | `OFF` | Treat warnings as errors |

With the tests:

@code{.sh}
cmake -B build -DCMAKE_PREFIX_PATH="..." -DENABLE_TESTS=ON
cmake --build build --parallel 3
ctest --test-dir build --output-on-failure
@endcode

## Install

@code{.sh}
cmake -B build -DCMAKE_PREFIX_PATH="..." -DCMAKE_INSTALL_PREFIX=$HOME/opt/bitrl
cmake --build build --parallel 3
cmake --install build
@endcode

installs `lib/libbitrllib.so`, the headers in `include/bitrl/` and the CMake package in `lib/cmake/bitrl/`.
Another project then uses bitrl with

@code{.cmake}
find_package(bitrl 5 REQUIRED)
target_link_libraries(my_target PRIVATE bitrl::bitrl)
@endcode

and `-DCMAKE_PREFIX_PATH` with the prefixes of bitrl, bitsim and libtorch. The package finds bitsim, libtorch
and TBB itself. `ci/check_install.sh` checks that an installed bitrl works this way.

## The environment server

Gymnasium's environments run on the <a href="https://github.com/pockerman/bitrl-rest-api">bitrl-rest-api</a>
server, which bitrl reaches at `http://0.0.0.0:8001/api` by default. Start it before you run the examples
that use them; its README says how to install and run it.

## Logging

bitrl logs with bitsim's logger, which writes with <a href="https://github.com/gabime/spdlog">spdlog</a> to
standard error. bitrl's messages go to the logger named `bitrl`:

@code{.cpp}
#include "bitrl/base/bitrl_log.h"

BITRL_LOG_INFO << "Episode " << episode << " reward " << reward;

bitrl::log::set_level("bitrl", bitrl::log::Level::warn); // only warnings and above from bitrl
bitrl::log::add_file_sink("training.log");               // to a file as well
@endcode

The `SPDLOG_LEVEL` environment variable sets the levels without changing the code:
`SPDLOG_LEVEL=bitrl=warn ./ppo_cartpole`. The logging section of \ref bitsim_install has the details.

## The documentation

This site is built with Doxygen from its own repository; \ref build_documentation says how.
