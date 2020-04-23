# SuperSuit

SuperSuit introduces a collection of small functions which can wrap reinforcement learning environments to do preprocessing ('microwrappers').
We support Gym for single agent environments and PettingZoo for multi-agent environments. Using it to convert space invaders to have a grey scale observation space and stack the last 4 frames looks like:

```
import gym
from supersuit import color_reduction, frame_stacking

env = gym.make('SpaceInvaders-v0')

env = frame_stacking(color_reduction(env, 'full'), 4)
```

You can install it via `pip install supersuit`

## Full list of functions:

`color_reduction(env, mode='full')` simplifies color information in graphical ((x,y,3) shaped) environments. `mode='full'` fully greyscales of the observation. This can be computationally intensive. Arguments of 'R', 'G' or 'B' just take the corresponding R, G or B color channel from observation. This is much faster and is generally sufficient.

`continuous_actions(env, seed=None)` discrete action spaces are converted to a 1d Box action space of size *n*. This space is treated as a vector of logits, and the multinomial distribution defined by those input logits is sampled to get a discrete value. Currently supports Discrete action spaces. It passes Box action spaces through without any alteration. Setting the `seed` parameter allows this module to act deterministically.

`down_scale(env, x_scale=1, y_scale=1)` uses mean pooling to reduce the observations output by each game by the given x and y scales. The dimension of an environment must be an integer multiple of it's scale. This is only available for 2D or 3D observations.

`dtype(env, dtype)` recasts your observation as a certain dtype. Many graphical games return `uint8` observations, while neural networks generally want `float16` or `float32`.

`flatten(env)` flattens observations into a 1D array.

`frame_stack(env, num_frames=4)` stacks the most recent frames. For vector games observed via plain vectors (1D arrays), the output is just concatenated to a longer 1D array. For games via observed via graphical outputs (a 2D or 3D array), the arrays are stacked to be taller 3D arrays. At the start of the game, frames that don't yet exist are filled with 0s. `num_frames=1` is analogous to not using this function.

`normalize_obs(env, env_min=0, env_max=1)` linearly scales observations to be 0 to 1, given known minimum and maximum observation values. Only works on Box observations with finite bounds.

`reshape(env, shape)` reshapes observations into given shape.

`homogenize_observations(env)` pads observations to be of the shape of the largest observaion of any agent, per the algorithm posed in *Parameter Sharing is Surprisingly Useful for Deep Reinforcement Learning*. This enables MARL methods that require the observations of all agents to work in environments with heterogenous agents. This currently supports on Discrete and Box observation spaces.

`homogenize_actions(env)` actions will be of same shape and belong to the same action_space. Discrete actions will be set to zero if they overshoot their original action space, and Box action spaces will be cropped.

`observation_lambda_wrapper(change_observation_fn, change_obs_space_fn=None)`
allows you to define arbitrary changes to the observations by specifying the observation transformation function  `change_observation_fn(observation) : observation`, and the observation space transformation `change_obs_space_fn(obs_space) : obs_space`. For Box-Box transformations the space transformation will be inferred from `change_observation_fn` if `change_obs_space_fn=None`.

`action_lambda_wrapper(change_action_fn, change_space_fn)` Allows you to define arbitrary changes to the actions with the function parameter `change_action_fn(action) : action` and to the action spaces with `change_space_fn(action_space) : action_space`


We hope to support Gym in all wrappers that are not explicitly multiplayer.
