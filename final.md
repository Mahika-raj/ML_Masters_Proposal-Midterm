---
layout: page
title: "Final Report"
permalink: /final/
---

**Final Project Report**

**By Christine Yewon Kim, Mahika Raj, Devanshi Sood, Joseph C Miao, and Song Yue David Li**

Link back to [←Proposal]({{ '/' | relative_url }})

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
