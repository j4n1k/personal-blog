---
title: "Reinforcement Learning for AMR Charging Decisions: The Impact of Reward and Action Space Design"
date: 2025-05-26
draft: false
summary: "We propose a novel reinforcement learning (RL) design to optimize the charging strategy for autonomous mobile robots in large-scale block stacking warehouses. RL design involves a wide array of choices that can mostly only be evaluated through lengthy experimentation. Our study focuses on how different reward and action space configurations, ranging from flexible setups to more guided, domain-informed design configurations, affect the agent performance. Using heuristic charging strategies as a baseline, we demonstrate the superiority of flexible, RL-based approaches in terms of service times. Furthermore, our findings highlight a trade-off: While more open-ended designs are able to discover well-performing strategies on their own, they may require longer convergence times and are less stable, whereas guided configurations lead to a more stable learning process but display a more limited generalization potential. Our contributions are threefold. First, we extend SLAPStack, an open-source, RL-compatible simulation-framework to accommodate charging strategies. Second, we introduce a novel RL design for tackling the charging strategy problem. Finally, we introduce several novel adaptive baseline heuristics and reproducibly evaluate the design using a Proximal Policy Optimization agent and varying different design configurations, with a focus on reward. "
tags: ["optimization", "reinforcement learning", "logistics", "AMR", "AGV", "battery management"]
showViews: true
---

We published a paper on battery management with reinforcement learning on arxiv. 

Feel free to read it here:  
https://arxiv.org/abs/2505.11136

The accompanying code can be accessed here:
{{< github repo="j4n1k/slapstack-battery-management" >}}

## TLDR

A new RL-based charging strategy for autonomous robots in block-stacking warehouses is proposed. 
We explore how different reward and action space designs (ranging from flexible to domain-informed) impact agent performance. 
The approach outperforms heuristic baselines in terms of KPIs like service time but reveals a trade-off between flexibility (better performance, slower convergence) and guidance (faster learning, limited generalization).

## Abstract

We propose a novel reinforcement learning (RL) design to optimize the charging strategy for autonomous mobile robots in large-scale block stacking warehouses. 
RL design involves a wide array of choices that can mostly only be evaluated through lengthy experimentation. 
Our study focuses on how different reward and action space configurations, ranging from flexible setups to more guided, domain-informed design configurations, affect the agent performance. 
Using heuristic charging strategies as a baseline, we demonstrate the superiority of flexible, RL-based approaches in terms of service times.
Furthermore, our findings highlight a trade-off: While more open-ended designs are able to discover well-performing strategies on their own, they may require longer convergence times and are less stable, 
whereas guided configurations lead to a more stable learning process but display a more limited generalization potential. 
Our contributions are threefold. 
First, we extend SLAPStack, an open-source, RL-compatible simulation-framework to accommodate charging strategies. 
Second, we introduce a novel RL design for tackling the charging strategy problem. 
Finally, we introduce several novel adaptive baseline heuristics and reproducibly evaluate the design using a Proximal Policy Optimization agent and varying different design configurations, with a focus on reward. 
