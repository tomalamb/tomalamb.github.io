---
title: "Universal In-Context Approximation By Prompting Fully Recurrent Models"
collection: publications
permalink: /publication/2023-08-11-paper-title-number-4
excerpt: 'LLMs; steering; spurious correlations'
date: 24-10-10
venue: 'NeurIPS'
paperurl: 'https://proceedings.neurips.cc/paper_files/paper/2024/file/848784373188ddf641079524e89e0ac9-Paper-Conference.pdf'
citation: 'Petrov, A., Lamb, T., Paren, A., Torr, P. and Bibi, A., 2024. Universal in-context approximation by prompting fully recurrent models. Advances in Neural Information Processing Systems, 37, pp.72061-72093.'
---
**Abstract:** Zero-shot and in-context learning enable solving tasks without model fine-tuning, making them essential for developing generative model solutions. Therefore, it is crucial to understand whether a pretrained model can be prompted to approximate any function, i.e., whether it is a universal in-context approximator. While it was recently shown that transformer models do possess this property, these results rely on their attention mechanism. Hence, these findings do not apply to fully recurrent architectures like RNNs, LSTMs, and the increasingly popular SSMs. We demonstrate that RNNs, LSTMs, GRUs, Linear RNNs, and linear gated architectures such as Mamba and Hawk/Griffin can also serve be universal in-context approximators. To streamline our argument, we introduce a programming language called LSRL that compiles to these fully recurrent architectures. LSRL may be of independent interest for further studies of fully recurrent models, such as constructing interpretability benchmarks.

**Recommended (bib) citation:**

@article{petrov2024universal,
  title={Universal in-context approximation by prompting fully recurrent models},
  author={Petrov, Aleksandar and Lamb, Tom and Paren, Alasdair and Torr, Philip and Bibi, Adel},
  journal={Advances in Neural Information Processing Systems},
  volume={37},
  pages={72061--72093},
  year={2024}
}