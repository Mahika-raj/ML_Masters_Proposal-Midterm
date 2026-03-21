---
layout: page
title: "Report"
permalink: /report/
---
**By Christine Yewon Kim, Mahika Raj, Devanshi Sood, Joseph C Miao, and Song Yue David Li**

Link to GitHub code: 
https://github.gatech.edu/mraj39/ML_Masters


# Literature/Introduction/Background
In the midst of Vision and Language Navigation innovation, AerialVLN was proposed, a UAV-based VLN task for outdoor environments specifically taking into consideration aerial actions as aerial navigation is significantly more complicated than ground-based VLN [1]. However, as energy awareness is not widely discussed in VLN, the typical path-length objective of existing approaches does not directly minimize energy consumption, nor allows constraining the energy of individual paths by battery capacity [2]. We will utilize a combination of Supervised Learning and Reinforcement Learning to better consider energy, building on top of their implementation.

Dataset Description
This Dataset includes AerialVLN and AerialVLN-S annotated images for training and evaluation, a total of 32.4GB.

Why These Dataset(s)

<ol type="A">
  <li>AerialVLN was selected because it is the only large scale outdoor aerial VLN dataset available. Its long flight paths and complex city level environments make fuel constraints a realistic and meaningful addition to the task. Without a dataset of this scale and complexity, testing energy aware navigation would not be possible.</li>
  <li>Why AerialVLN-S For most of our experiments we use AerialVLN-S since the shorter average path length makes training more computationally feasible while still capturing the core challenges of the task. It also has more evenly distributed path lengths which makes evaluation more consistent. </li>
</ol>


**Overview**
- Total size: 32.4GB
- Built using AirSim simulator on Unreal Engine 4
- 25 city level environments (cities, factories, parks, villages)
- 870+ different object types
- Supports dynamic weather (sun, rain, snow, fog) and lighting (morning, noon, night)


<img src="{{ '/images/a.png' | relative_url }}" width="650">
* recorded by AOPA licensed human UAV pilots to ensure realistic and high quality flight trajectories. Each path is paired with 3 instructions written by Amazon Mechanical Turk workers, giving the dataset a wide variety of ways to describe the same flight.


**Each Flight Data Set Contains**
- Natural language instructions describing landmarks and directional cues
- Flight trajectory as a sequence of 9 discrete actions: Move Forward, Turn Left, Turn Right, Ascend, Descend, Move Left, Move Right, Stop, and the initial starting pose
- RGB and depth images from a front facing camera at every step (depth range up to 100m)
- Semantic segmentation data available for future use

<img src="{{ '/images/b.png' | relative_url }}" width="650">

**Our Addition:**
Fuel Constraints - On top of the standard dataset, our preprocessing pipeline adds a fuel fraction at every step of every flight:
- Ranges from 1.0 (full tank) to 0.0 (empty)
- Computed dynamically during preprocessing
- Fed into the FuelGateModule during training
- Forces the model to adjust its behavior based on remaining battery level
- This addition is unique to our work and is not part of the original AerialVLN dataset

**Dataset Linkzx:**
https://github.com/AirVLN/AirVLN 

**Problem:**
With the new constraints, specifically fuel capacity and limits, we focus on aerial navigation in the sky, UAV-based and aimed towards outdoor environments.

First step of our goal is to implement fuel constraints on our first method, Teacher Forcing, and to compare the performance of simulation without fuel constraints. This method is primarily going to be used as a **baseline**.  


**Motivation:**
Many existing VLN tasks are built for agents that navigate on the ground, either indoors or outdoors. However, some tasks require intelligent agents to operate in the sky, such as UAV-based goods delivery, traffic/security patrol, and scenery tours [1]. Most importantly, aerial navigation is a field with much work to be done.

<br><br>
# Methods
**Data Preprocessing Methods:**
The preprocessing converts the raw simulation data from the AirSim environment into a high-dimensional feature space. We distinguish between the baseline architectural encoding provided by AirVLN and our novel Fuel-Aware state implementation. 

**Baseline preprocessing:**
To manage the computational load of the AerialVLN dataset, the baseline processes the raw sensor data into optimized feature vectors using a two-stream encoding process:
- Visual Feature Extraction: The raw RGB and Depth maps are passed through a pretrailed convolutional neural network. This is done by stripping away the actual images' spatial data and only saving the final feature vectors, representing the semantic content of the UAV images. 
- Language Tokenization: The raw textual navigation instructions are processed using a tokenizer to convert natural language into numerical tokens, mapping the human readable commands into the model’s embedding space.
- Serialized packaging: These features are synchronized and packaged into high-performance binary database files to maximize the throughput during the Teacher Forcing (TF) training loop.
  
**Dynamic Fuel Constraining:**
To introduce energy awareness into the navigation task, we implemented a dynamic preprocessing layer that injects space dependent constraints on the fly:
- Fuel fraction: For every discrete step within the flight trajectory, we calculate a normalized fuel fraction representing the battery life
- Mathematical formulation: This fraction is derived as the quotient of remaining mission steps over the total allocated steps: Fuel fraction[0.0, 1.0] = (Remaining steps)/(Maximum steps)
- State injection: This scalar value is appended to the visual and linguistic features at every timestep, allowing the model to perceive the remaining “survival time” relative to the goal state.

## ML Algorithms and Models
The primary model utilized up to this point is Teacher Forcing. Teacher forcing is a supervised learning algorithm that is useful in sequential models, such as navigation problems. It trains the model by taking the ground-truth as the input to the next step instead of using the predicted action. This allows the model to maintain stability, avoiding accumulation of error. 

In the original implementation of AerialVLN, a dataset of pre-planned trajectories represents the ground-truth, including expected camera data, navigation instructions, and actions. It attempts up to a maximum action limit. At each step, it passes the RGB and depth observations through the Seq2Seq policy, along with the previous ground-truth action and current hidden state of the RNN, to output a probability distribution of actions. The primary training objective is an imitation loss, where the trainer computes the primary loss by comparing the network’s predicted actions against the ground-truth action. Thus, Teacher Forcing encourages the model to imitate the expert trajectories. 

**Fuel Constraints**

Our extension of the model involves the incorporation of fuel constraints. We introduced fuel awareness as an additional input to guide the decision making process. As discussed in the pre-processing section, the fuel constraint is defined as the ratio of steps remaining to the maximum steps: it is a continuous variable from 0.0 to 1.0, where 0.0 represents no fuel remaining. 

Remaining Steps / Max Steps = Fuel Fraction [0.0 - 1.0]

The fuel parameter is fed into the present model and the gate module, which determines the importance of the fuel fraction. In addition to the original standard action loss, we also calculate an auxiliary loss using MSE, which trains the Fuel Budget Predictor for future steps. This auxiliary loss is in as a specialized component called the FuelGateModule, which essentially keeps the gate high or low depending on if the fuel is high or empty. The gate includes features that are important for exploration and goal-oriented navigation, so if fuel is limited, the drone will shift towards more direct trajectories instead of exploring the environment. 

Overall, the objective of this fuel awareness constraint, in addition to the original imitation loss, is to adapt the model’s predictions to be aware of physical constraints. In the beginning, when fuel is abundant, the model will prioritize exploration and improve familiarity with new environments. When fuel is low, the model will shift towards efficient paths to maximize likelihood of reaching the end within budget. 

<br><br>
# Results and Discussion

**1. TF Baseline without Fuel Constraints**

<img src="{{ '/images/c.png' | relative_url }}" width="850">

We take these baseline results from AerialVLN and use the CMA model as our teacher-forcing baseline without fuel constraints. It is evaluated using Success Rate (SR), Oracle Success Rate (OSR), Navigation Error (NE), and SDTW. On the test unseen split, CMA gets 1.6% SR, 4.1% OSR, 358.6 m NE, and 0.5 SDTW, while human performance is much higher. This shows the task is hard, especially in unseen environments. Another useful point is that the baseline often gets close to the goal but does not stop correctly, since OSR is higher than SR. Longer paths are also much harder, with only 1.8% success on long paths compared to 7.4% on shorter ones. Since this baseline does not include fuel or energy limits, our project adds path efficiency, mean fuel consumption, and constraint violation rate to better evaluate energy-aware navigation. We expect our model to keep a good success rate while producing smoother paths, using less fuel, and breaking the fuel constraint less often.


**2. TF with Fuel Constraints**
Since our current model is still based on Teacher Forcing, the main comparison we use is the Seq2Seq baseline from the AerialVLN paper. We use this because it is the closest baseline to what we have implemented so far, so it makes the comparison more fair. We compare it with our TF + Fuel model.

<img src="{{ '/images/d.png' | relative_url }}" width="850">


**3. Discussion + Analysis**

From the table, our TF + Fuel model is slightly better than the paper’s Seq2Seq baseline on both splits. On Val Seen, it improves NE from 146.0 to 144.5, SR from 4.8 to 5.2, OSR from 19.8 to 20.4, and SDTW from 1.6 to 1.7. On Val Unseen, it improves NE from 218.9 to 216.0, SR from 2.3 to 2.7, OSR from 11.7 to 12.2, and SDTW from 0.7 to 0.8.

The improvement is small, but that makes sense for this project. Aerial VLN is already a hard task, and the original paper also showed that teacher-forcing methods perform pretty badly, especially on unseen environments. So even though our fuel-aware version does a little better, the overall metrics are still low.
This suggests that adding fuel awareness helps the model make slightly better decisions, probably because it has some idea of how much budget is left and can avoid wasting movement. At the same time, it is still clear that Teacher Forcing is not a strong final solution for this task. That is also why the original paper moved to stronger methods like DAGGER.

Overall, our TF + Fuel model works as a good starting baseline for the project. It shows that fuel constraints can be added into the pipeline and can give a small improvement, even if the task is still difficult. This gives us something to compare against later when we try stronger methods like RL DAGGER-style training.

**4. Visualisations:**

<img src="{{ '/images/e.png' | relative_url }}" width="650">


Visualisation of successful navigation: 

<img src="{{ '/images/f.png' | relative_url }}" width="650">

Green arrows show horizontal movement, such as moving forward or side to side. Blue arrows show vertical movement and turning actions, including moving up or down and turning left or right. The red circle marks the stop action. To show which landmarks match the instruction, the same colors are used for both the bounding boxes in the images and the corresponding words in the text.

More images:

<img src="{{ '/images/g.png' | relative_url }}" width="550">

<img src="{{ '/images/h.png' | relative_url }}" width="550">

<img src="{{ '/images/i.png' | relative_url }}" width="550">



<br><br>

# Next Steps
## Goal: Implement M2 RL with Fuel Constraints and compare against TF baseline
**1: Design the RL Reward Function**
- Reward navigation success
- Penalize running out of fuel before reaching the goal
- Reward arriving at the destination with fuel remaining
- Invest some time into using aerialVLN instead of aerialvln-s

**2: Train and Compare**
- Train RL agent under the same fuel-constrained setup used for Teacher Forcing
- Evaluate both using the 4 standard AerialVLN metrics: SR, OSR, NE, and SDTW

**3: Ablation Studies**
- Toggle fuel fraction input on and off
- Measure how much fuel awareness actually contributes to performance

**4: Test Generalization**
- Evaluate on Val Unseen and Test Unseen splits
- Check if fuel-aware strategies learned in training hold up in new environments
- Benchmarks we could consider :
- Best ML baseline from the paper: ~4.5% SR
- Human performance: ~80% SR
- Our goal is to **meaningfully** improve on the ML baseline using fuel-aware RL



# Citations/References

[1] Liu et al. (2023). *AerialVLN: Vision-and-Language Navigation for UAVs*.  
The primary AirVLN paper.

[2] Pereira et al. (2025). *Energy-Aware Coverage Path Planner for Multirotor UAVs*.  
Used for the fuel modeling aspect.

[3] Morbidi, F., Cano, R., & Lara, D. (2016). *Minimum-Energy Path Generation for a Quadrotor UAV*.  
IEEE International Conference on Robotics and Automation (ICRA), Stockholm, Sweden.

## Other References

Morbidi, F., Cano, R., & Lara, D. (2016). *Minimum-energy path generation for a quadrotor UAV*. In **2016 IEEE International Conference on Robotics and Automation (ICRA)**, 1492–1498.  
[https://hal.science/hal-01276199/document](https://hal.science/hal-01276199/document)

Yacef, F., Rizoug, N., Degaa, L., & Hamerlain, M. (2020). *Energy-efficiency path planning for quadrotor UAV under wind conditions*. In **2020 7th International Conference on Control, Decision and Information Technologies (CoDIT)**, Vol. 1, 1133–1138.  
[https://hal.science/hal-04504751/document](https://hal.science/hal-04504751/document)

