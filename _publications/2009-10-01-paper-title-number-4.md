---
title: "Universal In-Context Approximation By Prompting Fully Recurrent Models"
collection: publications
permalink: /publication/2023-08-11-paper-title-number-2
excerpt: 'LLMs; steering; spurious correlations'
date: 24-10-10
venue: 'Arxiv'
paperurl: 'https://arxiv.org/pdf/2406.01424'
citation: 'Petrov, Aleksandar, et al. "Universal In-Context Approximation By Prompting Fully Recurrent Models." arXiv preprint arXiv:2406.01424 (2024).'
---
**Abstract:** Zero-shot and in-context learning enable solving tasks without model fine-tuning, making them essential for developing generative model solutions. Therefore, it is crucial to understand whether a pretrained model can be prompted to approximate any function, i.e., whether it is a universal in-context approximator. While it was recently shown that transformer models do possess this property, these results rely on their attention mechanism. Hence, these findings do not apply to fully recurrent architectures like RNNs, LSTMs, and the increasingly popular SSMs. We demonstrate that RNNs, LSTMs, GRUs, Linear RNNs, and linear gated architectures such as Mamba and Hawk/Griffin can also serve be universal in-context approximators. To streamline our argument, we introduce a programming language called LSRL that compiles to these fully recurrent architectures. LSRL may be of independent interest for further studies of fully recurrent models, such as constructing interpretability benchmarks.

**Recommended (bib) citation:**

@article{petrov2024universal,
  title={Universal In-Context Approximation By Prompting Fully Recurrent Models},
  author={Petrov, Aleksandar and Lamb, Tom A and Paren, Alasdair and Torr, Philip HS and Bibi, Adel},
  journal={arXiv preprint arXiv:2406.01424},
  year={2024}
}