---
title: "Focus on this, not that! Steering LLMs with Adaptive Feature Specification [Preprint]"
collection: publications
permalink: /publication/2023-08-11-paper-title-number-2
excerpt: 'LLMs; steering; spurious correlations'
date: 24-08-30
venue: 'Arxiv'
paperurl: 'https://arxiv.org/pdf/2410.22944'
citation: 'Lamb, Tom A., et al. "Focus On This, Not That! Steering LLMs With Adaptive Feature Specification." arXiv preprint arXiv:2410.22944 (2024).'
---
**Abstract:** Despite the success of Instruction Tuning (IT) in training large language models (LLMs) to perform arbitrary user-specified tasks, these models often still leverage spurious or biased features learned from their training data, leading to undesired behaviours when deploying them in new contexts. In this work, we introduce Focus Instruction Tuning (FIT), which trains LLMs to condition their responses
by “focusing on” specific features whilst ignoring others, leading to different behaviours based on what features are specified. Across several experimental settings, we show that focus-tuned models can be adaptively steered by focusing on different features at inference-time: for instance, robustness can be improved by focusing on task-causal features and ignoring spurious features, and social bias can be mitigated by ignoring demographic categories. Furthermore, FIT can steer behaviour in new contexts, generalising under distribution shift and to new unseen features at inference time, and thereby facilitating more robust, fair, and controllable LLM applications in real-world environments.

[Download paper here](https://arxiv.org/pdf/2410.22944)

**Recommended (bib) citation:**

@article{lamb2024focus,
  title={Focus On This, Not That! Steering LLMs With Adaptive Feature Specification},
  author={Lamb, Tom A and Davies, Adam and Paren, Alasdair and Torr, Philip HS and Pinto, Francesco},
  journal={arXiv preprint arXiv:2410.22944},
  year={2024}
}