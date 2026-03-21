<br><br><br>
# Click here to view the [Midterm Checkpoint](./report/) 
<br><br><br>
# ML_Masters_Proposal
**AerialVLN w Fuel Constraints**

**By Christine Yewon Kim, Mahika Raj, Devanshi Sood, Joseph C Miao, and Song Yue David Li**
- [Report](#report)
- [Gantt Chart](#gantt-chart)
- [Contribution Table](#contribution-table)

<br><br>

# Report:
<br>


# Introduction/Background

## Literature

In the midst of Vision and Language Navigation innovation, AerialVLN was proposed, a UAV-based VLN task for outdoor environments [1]. However, as energy awareness is not widely discussed in VLN, the typical path-length objective of existing approaches does not directly minimize energy consumption, nor allows constraining the energy of individual paths by battery capacity [2]. We will utilize a combination of Supervised Learning and Reinforcement Learning to better consider energy.

## Dataset Description
This dataset includes AerialVLN and AerialVLN-S annotated images for training and evaluation, a total of 32.4GB.

*Dataset Link*  
[AirVLN GitHub Repository](https://github.com/AirVLN/AirVLN)

## Problem
With the new constraints, specifically fuel capacity and limits, we focus on aerial navigation in the sky, UAV-based and aimed towards outdoor environments.

## Motivation
Many existing VLN tasks are built for agents that navigate on the ground, either indoors or outdoors. However, some tasks require intelligent agents to operate in the sky, such as UAV-based goods delivery, traffic/security patrol, and scenery tours [1]. Most importantly, aerial navigation is a field with much work to be done.

<br>

# Methods

## Data Preprocessing Methods

3 data preprocessing methods we are considering are fuel tokenization, state-space normalization, and trajectory-based windowing:

- **Fuel Tokenization:** Fuel capacity is a crucial constraint that is not quantified in the dataset. We can label trajectories with discrete tokens, such as low, middle, and high, or as a continuous function. The embedding of a fuel feature serves to steer towards both efficient and reliable path planning.  
  - PyTorch Tokenizer  
  - AutoTokenizer  

- **State-space Normalization:** Numerical data like position, cartesian and angular velocities, and fuel will be normalized into a standardized distribution so stable inputs get fed into the model. We want to balance them so that large-scale features like altitude do not dominate small-scale ones like fuel capacity.  
  - Scikit-learn StandardScaler  
  - Scikit-learn RobustScaler  

- **Trajectory-based Windowing:** Instead of optimizing fuel efficiency with entire simulations in AirVLN, we can segment each simulation into windows by training on sections between landmarks in the city-level environments. This should improve parallelization and make better use of the dataset, especially since AirVLN is not very extensive.  
  - PyTorch utils  
  - `torch.utils.data.DataLoader`  

## ML Algorithms and Models

We considered 2 supervised learning models: **Recurrent Neural Networks** and **Transformers**. These would function by embedding the instructions, visuals, and fuel as inputs, then outputting a low-cost trajectory.

Alternatively, and perhaps more intuitively, we can pursue **Proximal Policy Optimization (PPO)**, a reinforcement learning algorithm. Here, we would use fuel capacity as a Lagrange penalty.

<br>

## Learning Methods

### Supervised Training

- **Recurrent Neural Networks:** The goal of using an RNN is to predict the drone’s next action given embeddings for the state space. If we were to use an RNN, we would likely make the fuel parameter continuous so that regression can be performed.

- **Transformers:** The state space, fuel efficiency, and instructions are useful tokens for determining the most optimal path for saving fuel. A major challenge is balancing the flight instruction embedding with the energy-efficiency objective.  
  - Likely implemented in PyTorch or TensorFlow  

### RL Method

- **PPO with Lagrange Penalty:** Fuel for drones can span a number of things, but the most common is LiPo batteries for quadcopters. There is a 2016 paper that explores energy-efficient drones and the electrical “fuel” consumption for minimum-energy trajectories, so that will be a potentially useful reference [3]. The paper’s researchers leveraged ACADO, a MATLAB toolkit for KKT optimization, which serves as a proof of concept for PPO. The constraint for fuel could look something like:  
  - **L(x) = f(x) - λfuel g<sub>fuel</sub>(x)**, where g<sub>fuel</sub>(x) is an arbitrary constraint function for fuel remaining and x is the state space  
  - Stable Baselines3  

<br>

# Potential Results and Discussion

We will evaluate the model using **success rate, path efficiency, mean fuel consumption,** and **constraint violation rate**.

- **Success rate** measures how often the UAV reaches the target.
- **Path efficiency** reflects how smooth and direct the route is.
- **Mean fuel consumption** shows whether the model uses less energy than a baseline.
- **Constraint violation rate** tracks how often the model exceeds the fuel limit.

We aim to create a model that produces smoother paths by reducing sharp turns, lowering fuel use, and improving success rate under energy constraints.

Expected results are that the proposed supervised and reinforcement learning approaches will produce more efficient flight paths, lower average fuel consumption, and better respect energy limits, while still completing navigation tasks at a strong rate.

<br>

## Citations

**[1]**  
Liu et al. (2023): *AerialVLN: Vision-and-Language Navigation for UAVs* (The primary AirVLN paper).

**[2]**  
Pereira et al. (2025): *Energy-Aware Coverage Path Planner for Multirotor UAVs* (For the fuel modeling aspect).

**[3]**  
Fabio Morbidi, Roel Cano, David Lara. *Minimum-Energy Path Generation for a Quadrotor UAV.* IEEE International Conference on Robotics and Automation, May 2016, Stockholm, Sweden. Ffhal01276199v2f

## Other References

- Morbidi, F., Cano, R. & Lara, D. *Minimum-energy path generation for a quadrotor UAV.* In 2016 IEEE International Conference on Robotics and Automation (ICRA), 1492–1498 (2016).  
  [Link](https://hal.science/hal-01276199/document)

- Yacef, F., Rizoug, N., Degaa, L. & Hamerlain, M. *Energy-efficiency path planning for quadrotor UAV under wind conditions.* In 2020 7th International Conference on Control, Decision and Information Technologies (CoDIT), vol. 1, 1133–1138 (2020).  
  [Link](https://hal.science/hal-04504751/document)

<br><br><br>


# Gantt Chart

![Project Image](./images/Gantt.png)


<br><br><br>



# Contribution Table

![Project Image](./images/Table.png)


<br>
