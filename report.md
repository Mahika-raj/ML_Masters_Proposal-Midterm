---
layout: page
title: "Report"
permalink: /report/
---

# Literature 
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







