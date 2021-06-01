---
layout: post
title: "Notes on Dabney 2020 - \"A distributional code for value in dopamine-based reinforcement learning\""
---

# Distributional RL may be implemented by dopamine signals in Ventral Tegmental Area

This post is about distributional reinforcement learning. To fully understand distributional value learning, however, one first needs to understand classic methods for learning value functions; Monte Carlo and $TD(\lambda)$ are of particular note. A separate set of notes will be compiled on these algorithms but a summary is included here for $TD(0)$.

\section{Temporal Difference Learning - TD(0)}
Recall the update rule for $TD(0)$:

$$V_{k+1}(S_{t}) \longleftarrow V_{k}(S_{t}) + \alpha (R_{t+1} + \gamma V_{k}(S_{t+1}) - V_{k}(S_{t}))$$

The name of this method should give some hint to its method of action. At each timestep, an error signal is computed by subtracting the previous estimate from an estimate composed of one step of true reward, and the discounted value function of the successor state. These pieces are termed the TD target:

$$R_{t+1} + \gamma V_{k}(S_{t+1})$$

and anticipated reward:

$$V_{k}(S_{t})$$

The error signal is given by their difference:

$$\delta_{t} = R_{t+1} + \gamma V(S_{t+1}) - V(S_{t})$$

It may make intuitive sense to increase the estimated value function when new rewards are larger than expected, and decrease it when the environment is surprisingly stingy compared to $V_{k}(S_{t})$. This is exactly what the TD learning rule does. When the true reward - $R_{t+1}$ - is smaller than expected, $sgn(\delta_{t}) = -1$. This will update the value function to be:

$$V_{k}(S_{t}) \longleftarrow V_{k}(S_{t}) - \alpha (|\delta_{t}|))$$

nota bene the subtraction.


This is easy to see formally upon decomposition of $$V(S_{t}) = E[R_{t+1}] + \gamma V(S_{t+1})$$

Then we have $$\delta_{t} = (R_{t+1} + \gamma V_{k}(S_{t+1})) - (E[R_{t+1}] + \gamma V_{k}(S_{t+1}))$$

The anticipated reward/value terms $\gamma V_{k}(S_{t+1})$ cancel out, leaving us with:

$$\delta_{t} = R_{t+1} - E[R_{t+1}]$$

At this point it should be evident that the update rule diminishes $V_{k}(S_{t})$ specifically when actual reward is smaller than expected reward. And it increases it when true reward is greater than expected reward.

It is useful at this to take a step back and ask what we've been learning

\end{document}
