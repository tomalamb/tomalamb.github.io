---
title: "Focus on this, not that! Steering LLMs with Adaptive Feature Specification"
collection: publications
permalink: /publication/2023-08-11-paper-title-number-3
excerpt: 'LLMs; steering; spurious correlations'
date: 25-05-01
venue: 'ICML'
paperurl: 'https://openreview.net/pdf?id=rbI5mOUA8Z'
citation: 'Lamb, Tom A., et al. "Focus On This, Not That! Steering LLMs with Adaptive Feature Specification." Forty-second International Conference on Machine Learning.'
---
**Abstract:** Despite the success of Instruction Tuning (IT) in training large language models (LLMs), such models often leverage spurious or biased features learnt from their training data and can become misaligned, leading to undesired behaviours. While existing techniques can steer model behaviour at inference-time, they are often post-hoc and do not embed steering as an intrinsic model feature. In this work, we introduce Focus Instruction Tuning (FIT), which trains LLMs to condition their responses by focusing on specific features whilst ignoring others, leading to different behaviours based on what features are specified. Across diverse benchmarks, we demonstrate that FIT: (i) successfully steers behaviour at inference time; (ii) increases robustness by amplifying core task signals and down-weighting spurious cues; (iii) mitigates social bias by suppressing demographic attributes; and (iv) generalises under distribution shifts and to previously unseen focus features. FIT therefore offers a lightweight, intrinsic mechanism for building more robust, fair, and easily controllable LLMs.

**Recommended (bib) citation:**

@inproceedings{lambfocus,
  title={Focus On This, Not That! Steering LLMs with Adaptive Feature Specification},
  author={Lamb, Tom A and Davies, Adam and Paren, Alasdair and Torr, Philip and Pinto, Francesco},
  booktitle={Forty-second International Conference on Machine Learning},
  year={2025}
}