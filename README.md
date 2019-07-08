# CARLA PPO agent

THIS DOCUMENT IS A WORK-IN-PROGRESS

## About the Project
This project concerns how we may design environments in order to facilitate the training of
deep reinforcement learning based autonomous driving agents. The goal of the project is to
provide a working deep reinforcement learning framework that can learn to drive in visually
complex environments, with a focus on providing a solution that:

1. Works out-of-the-box.
2. Learns in the shortest time posible, to make it easier to quickly iterate on and test our hypoteses.
3. Provides the nessecary metrics to compare agents between runs.

We have used the urban driving simulator [CALRA](http://carla.org/) (version 0.9.5) as our environment.

Find a detailed project write-up [here](doc/Accelerating_Training_of_DeepRL_Based_AV_Agents_Through_Env_Designs.pdf).

Video of results:

<p align="center">
<a href="http://www.youtube.com/watch?feature=player_embedded&v=iF502iJKTIY" target="_blank"><img src="http://img.youtube.com/vi/iF502iJKTIY/0.jpg" alt="Proximal Policy Gradient in CARLA 0.9.5" width="480" height="360" border="0" /></a>
</p>

Use the timestaps in the description to navigate to the experiments of your interest.

## Contributions

<p align="center">
  <img width="500" src="doc/town07_lap.jpg">
</p>

- We provide two gym-like environments for CARLA*:
    1. Lap environment: This environment is focused on training an agent to follow a predetermined lap (see [CarlaEnv/carla_lap_env.py](CarlaEnv/carla_lap_env.py))
    2. Route environment: This environment is focused on training agents that can navigate from point A to point B (see [CarlaEnv/carla_route_env.py](CarlaEnv/carla_route_env.py). TODO: Lap env figure
- We provide analysis of optimal PPO parameters, environment designs, reward functions, etc. with the aim of finding the optimal setup to train reinforcement learning based autonomous driving agents (see Chapter 4 of [the project write-up](doc/Accelerating_Training_of_DeepRL_Based_AV_Agents_Through_Env_Designs.pdf) for further details.)
- We have shown that how we train and use a VAE can be consequential to the performance of a deep reinforcement learning agent, and we have found that major improvements can be made by training the VAE to reconstruct semantic segmentation maps instead of reconstructing the RGB input itself.
- We have devised a model that can reliably solve the lap environment in ~8 hours on average on a Nvidia GTX 970.
- We have provided an example of how sub-policies can be used to navigate with PPO, and we found it to have moderately success in the route environment (TODO add code for this).

\* While there are existing examples of gym-like environments for CARLA, there is no implementation that is officially endorsed by CARLA. Furthermore, most of the third-party environments do not provide an example of an agent that works out-of-the-box.

## Related Work

1. [https://arxiv.org/abs/1807.00412](Learning to Drive in a Day) by Kendall _et. al._
This paper by researchers at Wayve describes a method that showed how state representation learning through a variational autoencoder can be used to train a car to follow a straight country road in approximately 15 minutes.
2. [https://towardsdatascience.com/learning-to-drive-smoothly-in-minutes-450a7cdb35f4](Learning to Drive Smoothly in Minutes) by Raffin _et. al._ This medium articles lays out the details of a method that was able to train an agent in a Donkey Car simulator in only 5 minutes, using a similar approach as (1). They further provide some solutions to the unstable steering we may observe when we train with the straight forward speed-as-reward reward formulation of Kendall.
3. [https://arxiv.org/abs/1710.02410](End-to-end Driving via Conditional Imitation Learning) by Codevilla _et. al._ This paper outlines an imitation learning model that is able to learn to navigate arbitrary routes by using multiple actor networks, conditioned on what the current manouver the vehicle should take is. We have used a similar approach in our route environment agent.

# Method Overview

This is a high-level overview of the method.

1. Collect 10k 160x80x3 images by driving around manually.
2. Train a VAE to reconstruct the images.
3. Train the agent using the encoded state representations generated by the trained VAE and append a vector of measurements (steering, throttle, speed.) This is the input of the PPO-based agent.

<p align="center">
  <img width="500" src="doc/ppo_trianing_pipeline.jpg">
</p>

# How to Run

## Prerequisites

- Python 3.6
- [CARLA 0.9.5](https://github.com/carla-simulator/carla/tree/0.9.5) (may also work with later versions)
    - Our code expects the CARLA python API to be installed and available through `import carla`. TODO: instructions on installing the .egg file
    - Note that the map we use, Town07, may not be include by default when running `make package`. Add `+MapsToCook=(FilePath="/Game/Carla/Maps/Town07")` to `Unreal/CarlaUE4/Config/DefaultGame.ini` before running `make package` to solve this.
- [TensorFlow for GPU](https://www.tensorflow.org/) (we have used version 1.13, may work with later versions)
- [OpenAI gym](https://github.com/openai/gym)
- [OpenCV for Python](https://pypi.org/project/opencv-python/) (TODO insert our version)
- A GPU with at least 4 GB VRAM (we used a GeForce GTX 970)

## Running a Trained Agent

With the project, we provide one pretrained PPO agent for the lap environment.
The checkpoint files for this model is located in the `models` folder.

To run the trained agent, we first have to start an instance of CARLA.
Assuming that CARLA was build as a stand alone executeable (with `make package`,)
navigate to the folder containing CarlaUE4.sh (typically `CARLA_ROOT/Dist/LinuxTODO/`) and
start CARLA with the following command:

```
./CarlaUE4.sh Town07 -benchmark -fps=30
```

The parameters `-benchmark -fps=30` here are used to start a synchrnounous instance of CARLA,
where the delta time between each tick will be `dt=1/fps=1/30s`.
We need to use a synchounous environment with `fps=30` when running the `models/model_name` pretrained agent,
because this is the same environment setup as the agent were trained in.

Note that our environment has only been designed to work with `Town07` since this map is the one that closest
resembles the environments of Kendall _et. al._ and Raffin _et. al._
(see TODO for more information on environment design.)

Once the CARLA environment is up and running, you can use the following command to run
the trained agent in evaluation mode:

```
python run_eval.py --model_name TODO
```

If everything was setup correctly, we should now see an agent that is able to drive
smootly along the designated lap that is shown in Figure TODO.

## Training a New Agent

Start CARLA as is described in [Running a Trained Agent](#running-a-trained-agent).

Once the CARLA environment is up and running, use the following command to train a new agent:

```
python train.py --model_name name_of_your_model
```

To see and compare trained agents with TensorBoard, use the following command:

```
tensorboard --logdir logs/
```

## Training the Variational Autoencoder

If you wish to collect data to train the variational autoencoder yourself, please use the
following command:

```
python CarlaEnv/collect_data.py TODO
```

Press SPACE to begin recording frames.

After you have collected data to train the VAE with, use the following command to train the VAE:

```
python vae/train_vae.py TODO
```

To see and compare trained VAEs with TensorBoard, use the following command:

```
tensorboard --logdir vae/logs/
```

## Inspecting VAE Reconstructions

Once we have a trained VAE, we can use the following commad to inspect how its reconstructions look:

```
python vae/inspect_vae.py TODO
```

Use the TODO button to seed your VAE with a latent z that is generated when the image is passed through the encoder (useful for comparing VAE reconstructions across models, as there is no guarantee that the various features of the input will be encoded in the same indices of Z.)

## Inspecting the Agent's Decision Making

We may also use the following command to see how a trained agent will behave to changes in latent space vector z by running:

```
python inspect_agent.py TODO
```

# File Overview

train.py
Script for training a PPO agent in the lap environment

run_eval.py
Script for running a trained model in eval mode

utils.py
Various mathematical, tensorflow, DRL utility functions

ppo.py
Script that defines the PPO model

common.py
File containing all reward fomulations and other functions directly related to
reward functions and state spaces.

inspect_agent.py
Script that can be used to inspect the behaviour of the agent as
the VAE's latent space vector z is annealed.

vae/
This folder contains all variational autoencoder related code.

vae/train_vae.py
Script that trains a variational autoencoder.

vae/models.py
Python file containing code for constructing MLP and CNN-based VAE models.

vae/inspect_vae.py
Script used to inspect how latent space vector z affects the reconstructions
of a trained VAE.

vae/data/
Folder containing the data (images) that were used when training the VAE model
that is bundled with the repo.

vae/models/
Folder containing trained models and tensorboard logs.

doc/
Folder containing figures that are used in this readme,
in addition to a PDF version of the accompanying thesis
for this repo.

CarlaEnv/
Folder containing code that is related to the CARLA environments.

CarlaEnv/carla_lap_env.py
File containing code for the CarlaLapEnv class

CarlaEnv/carla_route_env.py
File containing code for the CarlaRouteEnv class

CarlaEnv/collect_data.py
TODO Move to root?
Script that is used to manually drive a car in the environment to
collect images that can be used to train a VAE.

CarlaEnv/hud.py
Code for the HUD that is displayed on the left-hand-side of the spectating window.

CarlaEnv/keyboard_control.py
TODO wasn't this file removed?

CarlaEnv/planner.py
Global route planner used to find routes from A to B.
TODO insert link to source

CarlaEnv/wrappers.py
File containing wrapper classes for several CARLA classes.

CarlaEnv/agents/
Contains code that is used by the route planner (TODO verify that these files are necessary).
TODO insert link to source

models/
Folder containing trained model weights and tensorboard log files

# Summary of Environment and Agent Design

Here we have summarized the main findings and reasoning behind various design decisions.

## Reward functions

TODO insert reward figure

TODO insert reward functions

<p align="center">
  <img width="500" src="doc/reward5_formula.png">
</p>

In our experiments, we have found that the reward function to give the best results
in terms of creating agents that drive in the center of the lane, and at a constant
target speed of 20 kmh, was reward 5.

The idea behind this reward function is that...

## Fail Faster Through Checkpoints

One thing that we quickly observed when we tried to train an agent in the lap environment,
was that the agent's learning stagnated after approximately 3 hours. Beyond the 3 hour mark,
the agent would spend about 2 minutes driving a well-known path until it would hit a particularly
difficult turn and would immediately fail.

In order to overcome this obstacle, the agent needs to either
(1) repeatedly attempt this stretch of road, and, by the help of lucky values
sampled from exploration noise, sample actions that lead to better rewards, or
(2) experience similar stretches of roads to eventually generalize to this road as well.

Since the agent did not learn even after 20 hours, we can conclude that the repeated
stretch of the lap does not aid the agent in generalizing (2). That means that the
only way the agent will overcome the obstacle is by getting lucky.

In order to expediate this process we have introduced checkpoints in the training phase of the agent.
The checkpoints work by simply keeping track of the agents progress, saving the center lane waypoint
every 50m of the track, and resetting the agent to that checkpoint when the environment is reset.

## VAE Trained on Semantic Maps

We found great improvements when we trained the VAE to reconstruct the
semantic segmentation maps rather than the RGB input itself.
The improvements suggests that more informative state representations are important in
accelerating the agent's learning, and that a semantic encoding of the environment
is more useful to the agent for this particular task.

As a result, training on semantic maps is enabled by default when calling
train_vae.py (disable by setting `--use_segmentations False`).

## Exploration Noise



## Environment Synchroncity



# Future Work


# Cite this Project

Citation will be provided as soon as the write-up is officially published (Expected mid-August.)

TODO: Paste in gramarly
