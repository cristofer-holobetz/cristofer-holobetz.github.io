---
layout: post
title: "Notes on Dabney 2020 - \"A distributional code for value in dopamine-based reinforcement learning\""
---

# Distributional RL may be implemented by dopamine signals in Ventral Tegmental Area

This post is about distributional reinforcement learning. To fully understand distributional value learning, however, one first needs to understand classic methods for learning value functions; Monte Carlo and $TD(\lambda)$ are of particular note. A separate set of notes will be compiled on these algorithms but a summary is included here for $TD(\lambda)$.


Recall the update rule for $TD(0)$:

$$V(S_{t}) \longleftarrow V(S_{t}) + \alpha (R_{t+1} + \gamma V(S_{t+1}) - V(S_{t}))$$