---
title: "Semantic-Level Confidence Calibration of Language Models via Temperature Scaling"
collection: publications
permalink: /publication/2025-05-20-paper-title-number-5
excerpt: 'LLMs; Uncertiainty Quantification; Semantic Calibration'
date: 25-05-20
venue: 'ICLR: QUESTION Workshop'
paperurl: 'https://openreview.net/pdf?id=22UNXHzBgt'
citation: 'Lamb, T. A., Ivanova, D. R., Torr, P., & Rudner, T. G. (2025). Semantic-Level Confidence Calibration of Language Models via Temperature Scaling. In ICLR Workshop: Quantify Uncertainty and Hallucination in Foundation Models: The Next Frontier in Reliable AI.'
---
**Abstract:** Calibration of language models is typically studied at the token level, with scalar temperature scaling serving as the primary approach for recalibrating models. Recent multi-sampling techniques allow us to elicit semantic uncertainty measures from language models. However, these techniques focus on summary statistics of the limited existing semantic confidence distributions rather than on how well-calibrated these distributions are, a crucial factor in ensuring that the resulting semantic likelihoods are both meaningful and reliable. In this paper, we investigate whether and how temperature scaling, which directly influences generative diversity and token-level calibration, affects semantic calibration. We address these question by investigating semantic-level calibration in both pre-trained and fine-tuned models. In particular, we introduce a framework for assessing semantic confidence that incorporates both existing and novel confidence measures, comparing them to a single-generation confidence measure. Furthermore, we investigate various temperature scaling methods and their effect on semantic calibration. Our experiments span both open-book and closed-book question answering datasets. Our empirical findings demonstrate that scalar temperature scaling, when appropriately applied, provides a simple, widely applicable, and effective method for improving semantic calibration in language models.

**Recommended (bib) citation:**

@inproceedings{lamb2025semantic,
  title={Semantic-Level Confidence Calibration of Language Models via Temperature Scaling},
  author={Lamb, Tom A and Ivanova, Desi R and Torr, Philip and Rudner, Tim GJ},
  booktitle={ICLR Workshop: Quantify Uncertainty and Hallucination in Foundation Models: The Next Frontier in Reliable AI}
}