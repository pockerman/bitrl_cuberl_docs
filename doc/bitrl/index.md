\page bitrl_main bitrl

_bitrl_ is a C++20 library for reinforcement learning, built on \ref index "bitsim". It has environments,
from grid worlds, Connect2 and multi-armed bandits to bitsim's robots in Project Chrono and Gymnasium's
environments, and the algorithms that learn in them, from dynamic programming to proximal policy
optimization. Its deep reinforcement learning uses <a href="https://pytorch.org/cppdocs/">libtorch</a>, the
C++ API of PyTorch. The source is on <a href="https://github.com/pockerman/bitrl">GitHub</a>.

| ![PPO landing the lander](ppo_lunar_lander.gif) | ![Monte Carlo tree search driving a robot around a track](mcts_line_following_lap.gif) |
|:--:|:--:|
| PPO learns to land Gymnasium's LunarLander, with continuous throttles. \ref bitrl_example_ppo_lunar_lander "Example" | Monte Carlo tree search drives bitsim's Chrono robot around a track and around boxes, planning every 0.2 s. \ref bitrl_example_mcts_line_following "Example" |
| ![Q-learning, double Q-learning, expected SARSA and SARSA on CliffWalking](cliff_walking_compared.gif) | ![Value iteration on FrozenLake](value_iteration.gif) |
| The paths four temporal difference methods learn on CliffWalking: Q-learning walks along the edge of the cliff, SARSA keeps away from it. \ref bitrl_example_q_learning "Example" | Value iteration on FrozenLake: the values spread back from the goal, sweep by sweep. \ref bitrl_example_value_iteration "Example" |

## What is in it

| Area | What bitrl has |
| ---- | -------------- |
| Environments | grid worlds, Connect2, multi-armed bandits, GymWalk, bitsim robots in Chrono, and Gymnasium's FrozenLake, CliffWalking, Taxi, Blackjack, CartPole, MountainCar, Pendulum, Acrobot and LunarLander |
| Dynamic programming | iterative policy evaluation, policy iteration, value iteration |
| Monte Carlo | first visit Monte Carlo, Monte Carlo tree search |
| Temporal difference | SARSA, Q-learning, expected SARSA, double Q-learning |
| Policy gradients | REINFORCE, with and without a baseline, A2C, PPO |
| Policies | epsilon-greedy with decay, soft-max, greedy and uniform tabular policies |
| Tools | experience replay buffers, trainers, policies saved to and loaded from files, Tensorboard logging, PyTorch distributions and optimizers |

## Gymnasium environments

The Gymnasium environments run in Python, on the
<a href="https://github.com/pockerman/bitrl-rest-api">bitrl-rest-api</a> server, and bitrl talks to them over
HTTP, at `http://0.0.0.0:8001/api` by default. Start the server before you run the code that uses them. The
other environments, the grid worlds, Connect2, the bandits and the Chrono robots, run in bitrl itself.

## Where to start

- \ref bitrl_install lists what bitrl needs, and explains how to build and install it.
- \ref bitrl_examples "The examples" build on each other: the basic ideas of reinforcement learning, the
  PyTorch C++ API, running and logging experiments, the environments, and then the algorithms, from bandits
  to deep reinforcement learning. Each has a page that explains it, and every algorithm's ends by showing
  what it learned.
