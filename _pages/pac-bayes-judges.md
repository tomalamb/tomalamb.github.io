---
layout: distill
title: Certifying LLM Judges
description: Tight PAC-Bayes Generalisation Guarantees for Large Language Model Judges
permalink: /pac-bayes-judges/
nav: false
date: 2026-11-12
bibliography: pac-bayes-judges.bib

authors:
  - name: Tom A. Lamb
    url: "https://tomalamb.github.io"
    affiliations:
      name: University of Oxford
  - name: Philip H. S. Torr
    url: "https://www.robots.ox.ac.uk/~phst/"
    affiliations:
      name: University of Oxford
  - name: Tim G. J. Rudner
    url: "https://timrudner.github.io"
    affiliations:
      name: University of Toronto, Vector Institute, Vijil

toc:
  - name: Introduction
  - name: Why Certify Safety Judges?
  - name: PAC-Bayes Certificates for Judges
    subsections:
      - name: Compression-Based Bounds
      - name: The Oversight Model Certificate
  - name: "LoRA-GT: Global-Scale Quantised Adaptations"
  - name: Empirical Evaluation
    subsections:
      - name: Non-Vacuous and Tight Guarantees
      - name: Compression Yields the Tightest Guarantees
      - name: How Far Compression Can Go
  - name: Takeaways
  - name: Code and Paper
---

## Introduction

Large language model (LLM) judges are increasingly used to monitor the outputs of LLM-based AI agents. Guardrail models such as Llama Guard <d-cite key="inan2023llama"></d-cite> and Qwen3Guard <d-cite key="zhao2025qwen3guard"></d-cite> already make safety decisions at scale, such as whether to block, escalate, or allow a model output. Yet formal guarantees on the reliability of these judges remain largely absent, limiting confidence in their use for safety-critical oversight.

This motivates our central question: **can we obtain practical, non-vacuous guarantees for safety judges using simple, standard adaptation pipelines and the same data for both training and certification?**

Our contributions are the following:

- We formalise PAC-Bayes certification for judges evaluating prompt–response pairs drawn from a fixed deployed generator, and certify both 0–1 classification risk and Brier risk, to obtain certificates for both predictive accuracy and calibration.
- We introduce **LoRA-GT**, which shares a single quantisation scale across layers and yields the tightest certificates in all 40 model–dataset–loss cases.
- Across five guardrail models and four benchmarks, we show that certificate tightness depends more on adaptation complexity than on base model scale.

{% include figure.liquid path="assets/img/pac-bayes-judges/pull.png" class="img-fluid" zoomable=true %}

<div class="caption">
  <b>Figure 1.</b> Compression yields tight, non-vacuous PAC-Bayes guarantees for safety oversight models. <b>Left:</b> LoRA-T with per-tensor versus shared global scaling (LoRA-GT). LoRA-GT removes the per-layer quantisation-scale metadata that dominates adaptor description length under aggressive compression. <b>Right:</b> certified bounds and test risk on BeaverTails under 0–1 and Brier losses.
</div>

## Why Certify Safety Judges?

For safety decisions to be reliable, judges must generalise beyond their evaluation samples, while thresholded safety decisions additionally require well-calibrated uncertainty estimates. Validation-based bounds can provide guarantees for a fixed predictor using data held out from training, but can be costly when labelled safety data are limited, and depend only on held-out performance and sample size. They therefore provide little insight into which properties of the learned model and learning process are associated with strong generalisation.

Guardrail models are already capable safety judges, so adaptation to a new safety taxonomy or set of guidelines can be viewed as a simple refinement of their existing safety capabilities. PAC-Bayes theory <d-cite key="catoni2007pac"></d-cite> provides a natural route to investigating whether such simple adaptations can achieve both strong predictive performance and tight generalisation guarantees, by relating empirical performance to model complexity.

## PAC-Bayes Certificates for Judges

Let a fixed generator $$G$$ produce responses $$y$$ to prompts $$x \sim \mathbb{P}_{\mathcal{D}}$$, inducing a joint distribution $$\mathbb{P}_{\mathcal{D},G}$$ over prompt–response pairs. A judge $$h$$ tries to emulate a ground-truth safety label $$z^*(x,y) \in \{0,1\}$$. We certify the judge's risk on the deployment distribution,

$$
\mathcal{R}_G(h) = \mathbb{E}_{(x,y)\sim\mathbb{P}_{\mathcal{D},G}}\big[\ell(h(x,y), z^*(x,y))\big],
$$

from $$n$$ i.i.d. labelled pairs. Here _fixed_ refers to the fact that the generator weights, decoding procedure, system prompt, and prompt distribution remain unchanged between certification and deployment.

### Compression-Based Bounds

Following prior compression-based analyses of LLMs <d-cite key="lotfi2024non,lotfi2024unlocking"></d-cite>, we use a Solomonoff prior $$\mathbb{P}_{\mathrm{Sol}}(h) \propto 2^{-K(h)}$$. This prior assigns greater probability to hypotheses with shorter descriptions, formalising an Occam's razor-style preference for simpler hypotheses. The Kolmogorov complexity $$K(h)$$ is uncomputable, but the length $$\lvert h \rvert$$ of any valid encoding of the quantised and compressed adaptor upper-bounds it. We refine the existing methodology in three ways:

- a **generalised Catoni bound** <d-cite key="rodriguez2024more"></d-cite>, whose infimum over $$\lambda$$ holds on a single high-probability event, so no union bound or held-out data is needed;
- **Elias $$\delta$$ coding**, which is tighter than the Elias $$\gamma$$ code used in prior work;
- **Brotli compression** of the quantised adaptor, which shortens its description further.

### The Oversight Model Certificate

With a point-mass posterior on the deployed judge, the KL term reduces to the description length (plus a small coding overhead $$\mathrm{KL}^{\mathrm{UB}}$$). With probability at least $$1-\beta$$, for all judges $$h$$ simultaneously,

$$
\mathcal{R}_{G}(h) \leq \inf_{\lambda>0} \left\{ \frac{1}{1 - e^{-\lambda/n}} \left[ 1 - \exp\left(-\frac{\lambda \widehat{\mathcal{R}}_S(h) + \mathrm{KL}^{\mathrm{UB}}(h) + \log \frac{2+\sqrt{2n}}{\beta}}{n} \right) \right] \right\}.
$$

The certificate applies directly to the single deployed judge, so no Gibbs sampling is needed. It makes no assumptions about the learning algorithm. The adaptor is trained conventionally and certified post hoc on the same data.

## LoRA-GT: Global-Scale Quantised Adaptations

The complexity term is governed by adaptor description length. Increasingly compact parameter-efficient adaptations, from LoRA <d-cite key="hu2022lora"></d-cite> to LoRA-XS to LoRA-T <d-cite key="morris2026learning"></d-cite>, can therefore tighten certificates, as long as they keep predictive performance. The weight-tied LoRA-T variant learns a single small vector $$\mathbf{v}^{(\ell)} \in \mathbb{R}^u$$ per layer:

$$
W = W_0 + U \Sigma \Big( \textstyle\sum_{i=1}^u v_i P_i \Big) V^\top .
$$

**The scale overhead.** Under aggressive quantisation, the per-layer bfloat16 scales dominate the description length. At $$u{=}1$$, $$b{=}2$$ on Llama-Guard-3-8B, the weights require only 64 bits while the scales require 512. **LoRA-GT** shares a single global scale across layers, cutting the total from 576 to 80 bits, an 86% reduction. The scale is chosen by a fixed search over representable bfloat16 values, starting from the smallest clipping-free scale.

## Empirical Evaluation

We evaluate five guardrail models: Llama Guard 3 (1B, 8B) and Qwen3Guard (0.6B, 4B, 8B). We use four benchmarks: Egida ($$n{=}792$$), OpenAI Moderation ($$n{=}1{,}344$$), Aegis ($$n{=}28{,}984$$) and BeaverTails ($$n{=}300{,}567$$). Each adaptor is trained with standard supervised fine-tuning, then quantised and compressed for certification.

### Non-Vacuous and Tight Guarantees

{% include figure.liquid path="assets/img/pac-bayes-judges/main_results.png" class="img-fluid" zoomable=true %}

<div class="caption">
  <b>Figure 2.</b> Certified bounds and test risk for 0–1 and Brier losses across models, datasets and adaptor families. LoRA, whose bounds are vacuous, is shown at its lowest test risk. Highly compressed adaptors achieve tight, non-vacuous guarantees at a small cost in test risk.
</div>

We obtain non-vacuous guarantees under both losses across models, datasets and adaptor families. This holds even on Egida and OpenAI Moderation, which have only 792 and 1,344 certification samples. Standard LoRA is vacuous in every case, even on BeaverTails with $$n \approx 300$$k: the complexity term depends on $$\lvert h \rvert / n$$, and LoRA descriptions there take 0.8M to 28M bits.

### Compression Yields the Tightest Guarantees

{% include figure.liquid path="assets/img/pac-bayes-judges/compression.png" class="img-fluid" zoomable=true %}

<div class="caption">
  <b>Figure 3.</b> (a) Bound vs. description length across models and datasets: more compressed adaptations yield tighter guarantees. (b) LoRA-T variants: tying and global-scale quantisation shorten descriptions and tighten bounds, with only modest changes in test risk.
</div>

Certificate tightness closely tracks adaptor description length. The description-length ordering is LoRA, LoRA-XS, LoRA-TN, LoRA-TT, then LoRA-GT. It matches the certificate ordering in 39 of 40 cases, despite large differences in backbone scale. On BeaverTails, quantised LoRA-GT has 2.1 to 6.7 points higher test risk than LoRA, while its certificate falls from 1.000 to between 0.169 and 0.226.

### How Far Compression Can Go

Quantisation shortens descriptions but can also change the judge's predictions. We measure this as _functional distortion_, the change in risk caused by quantisation:

$$
\Delta_G(h,\tilde h) = \mathbb{E}_{(x,y)\sim\mathbb{P}_{\mathcal{D},G}}\big[\ell(\tilde h(x,y),z^*) - \ell(h(x,y),z^*)\big].
$$

{% include figure.liquid path="assets/img/pac-bayes-judges/distortion.png" class="img-fluid" zoomable=true %}

<div class="caption">
  <b>Figure 4.</b> Compression and distortion under 0–1 loss for LoRA-GT. (a) Mean certificates split into pre-quantisation empirical risk, signed distortion and complexity. (b) Absolute distortion against description length at 2, 3, 4 and 8 bits.
</div>

On the smaller benchmarks, 2-bit quantisation gives the tightest certificates. On Aegis and BeaverTails, the complexity savings are already small relative to $$n$$, so the extra distortion at 2 bits loosens the certificate and 4 bits wins.

## Takeaways

1. **Compact safety adaptations yield non-vacuous, often tight guarantees** for both classification and Brier risk, even with only 792 certification samples and no held-out validation set.
2. **Adaptation complexity matters more than backbone scale.** Adaptor description length tracks certificate tightness more closely than model size, and LoRA-GT is tightest in all 40 cases.
3. **Aggressive quantisation helps most when data are scarce.** With more data, quantisation error can outweigh the smaller complexity savings.

## Code and Paper

- Paper: [OpenReview](https://openreview.net/forum?id=megigqX2Rj)
- Code: [GitHub](https://github.com/tomalamb/generalisation-guarantees-llm-judges)

```bibtex
@inproceedings{lamb2026tight,
  title     = {Tight {PAC}-{Bayes} Generalisation Guarantees for Large Language Model Judges},
  author    = {Lamb, Tom A and Torr, Philip H S and Rudner, Tim G J},
  booktitle = {Advances in Neural Information Processing Systems},
  year      = {2026}
}
```
