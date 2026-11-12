---
layout: paper
title: Tight PAC-Bayes Generalisation Guarantees for Large Language Model Judges
description: Certifying the reliability of LLM safety judges with compressed parameter-efficient adaptations.
permalink: /pac-bayes-judges/
nav: false
bibliography: pac-bayes-judges.bib
venue: NeurIPS 2026

authors:
  - name: Tom A. Lamb
    url: "https://tomalamb.github.io"
    affiliations: [1]
  - name: Philip H. S. Torr
    url: "https://www.robots.ox.ac.uk/~phst/"
    affiliations: [1]
  - name: Tim G. J. Rudner
    url: "https://timrudner.github.io"
    affiliations: [2, 3, 4]
affiliations:
  - University of Oxford
  - University of Toronto
  - Vector Institute
  - Vijil

links:
  - name: Paper
    icon: fa-solid fa-file-pdf
    url: "https://openreview.net/forum?id=megigqX2Rj"
  - name: Code
    icon: fa-brands fa-github
    url: "https://github.com/tomalamb/generalisation-guarantees-llm-judges"

toc:
  - name: Introduction
  - name: Preliminaries
    subsections:
      - name: Quantisation, compression, and description length
      - name: PAC-Bayes bounds
  - name: Certifying safety oversight models
    subsections:
      - name: The oversight model certificate
      - name: "LoRA-GT: global-scale quantised adaptations"
  - name: Empirical evaluation
    subsections:
      - name: Non-vacuous and tight guarantees
      - name: Adaptor compression yields the tightest guarantees
      - name: How far compression can go
  - name: Discussion
  - name: Conclusion
  - name: Citation
---

## Introduction

Large language models (LLMs) are increasingly deployed in generative workflows and agentic systems, including in safety-critical domains, yet remain vulnerable to adversarial attacks, misuse, and hallucinations and so require oversight. Since human supervision does not scale, _scalable oversight_ <d-cite key="bowman2022measuring"></d-cite>, in which LLMs act as automated judges, is becoming increasingly common for monitoring deployed models. Guardrail models such as Llama Guard <d-cite key="inan2023llama"></d-cite> and Qwen3Guard <d-cite key="zhao2025qwen3guard"></d-cite> already make safety decisions at scale, such as whether to block, escalate, or allow a model output.

For these decisions to be reliable, judges must generalise beyond their evaluation samples, while thresholded safety decisions additionally require well-calibrated uncertainty estimates. However, judges are increasingly used in safety-critical settings without formal guarantees on their performance or predictive uncertainty, while what determines their generalisation remains poorly understood.

Guardrail models are already capable safety judges, so adaptation to a new safety taxonomy or set of guidelines can be viewed as a simple refinement of their existing safety capabilities. PAC-Bayes theory provides a natural route to investigating whether simple adaptations can achieve both strong predictive performance and tight generalisation guarantees, by relating empirical performance to model complexity <d-cite key="alquier2024user,catoni2007pac"></d-cite>. Recent compression-based analyses have obtained non-vacuous guarantees at the scale of generative LLMs <d-cite key="lotfi2024non,lotfi2024unlocking"></d-cite>, but focus on a model's own generative risk. This motivates our central question:

<div class="takeaway">
  <i>Can we obtain practical, non-vacuous guarantees for safety judges using simple, standard adaptation pipelines and the same data for both training and certification?</i>
</div>

{% include figure.liquid path="assets/img/pac-bayes-judges/pull.png" class="img-fluid" zoomable=true %}

<div class="caption">
  <b>Figure 1. Compression yields tight, non-vacuous PAC-Bayes guarantees for safety oversight models.</b> <b>Left:</b> LoRA-T with per-tensor versus shared global scaling (LoRA-GT), which removes the per-layer quantisation-scale metadata that dominates PEFT adaptor description lengths under aggressive compression. <b>Right:</b> certified bounds and test risk on BeaverTails under 0–1 and Brier losses, where LoRA-GT achieves the tightest certificates among the shown adaptors while retaining competitive test risk.
</div>

Our contributions are the following:

1. We formalise PAC-Bayes certification for judges evaluating prompt–response pairs drawn from a fixed deployed generator, and certify both 0–1 classification risk and Brier risk, to obtain certificates for both predictive accuracy and calibration.
2. We introduce LoRA-GT, which shares a single quantisation scale across layers and yields the tightest certificates in all 40 model–dataset–loss cases. We also tighten the certification methodology with a generalised Catoni bound and Elias $$\delta$$ coding, and shorten adaptor descriptions using Brotli compression.
3. Across five guardrail models and four benchmarks, we show that tailored PEFT can achieve strong predictive performance and non-vacuous, often tight certificates, even in small-data regimes without requiring a separate held-out validation set. Certificate tightness depends more on adaptation complexity than on base model scale.

## Preliminaries

### Quantisation, compression, and description length

Quantisation maps real-valued parameters $$\theta \in \mathbb{R}^p$$ to a finite discrete representation $$(z, s)$$, where $$z$$ contains the quantised values and $$s$$ the reconstruction parameters, such as a scale. The resulting discrete representation can be further compressed losslessly, using a compressor such as Brotli, into a binary string $$c(h)$$. We define the description length of a hypothesis $$h$$ as $$\lvert h \rvert := \lvert c(h) \rvert$$. This reflects the minimum description length principle, an application of Occam's razor that favours simple hypotheses that fit the data. In our PAC-Bayes certificates, shorter descriptions contribute a smaller complexity term.

### PAC-Bayes bounds

Among the many bounds in the PAC-Bayes literature, we adopt a generalised Catoni-style bound <d-cite key="rodriguez2024more"></d-cite>, not explored in prior compression-based analyses of LLMs, which yields the tightest certificates in our setting. For a loss in $$[0,1]$$, a prior $$\mathbb{P}$$ independent of the sample $$S$$ and any $$\beta \in (0,1)$$, with probability at least $$1-\beta$$, simultaneously for all posteriors $$\mathbb{Q}$$,

$$
\mathcal{R}_{\mathcal{D}}(\mathbb{Q}) \leq \inf_{\lambda>0} \left\{ \frac{1}{1 - e^{-\lambda/n}} \left[ 1 - \exp\left(-\frac{\lambda \widehat{\mathcal{R}}_S(\mathbb{Q}) + \mathrm{KL}(\mathbb{Q}\|\mathbb{P}) + \log \frac{2+\sqrt{2n}}{\beta}}{n} \right) \right] \right\}.
$$

The infimum is _inside_ the bound, which holds simultaneously for all $$\lambda>0$$ on a single $$1-\beta$$ event. Minimising over $$\lambda$$ therefore evaluates the bound without a union-bound penalty or held-out data, unlike the fixed-$$\lambda$$ Catoni bound used in prior compression-based LLM analyses.

Following prior work <d-cite key="lotfi2024non,lotfi2024unlocking"></d-cite>, we adopt a Solomonoff prior $$\mathbb{P}_{\mathrm{Sol}}(h) \propto 2^{-K_p(h \mid M)}$$, where $$K_p$$ is the prefix-free Kolmogorov complexity of $$h$$ given a fixed, data-independent decoder $$M$$. Although $$K_p$$ is uncomputable, the length of any valid prefix-free encoding of $$h$$ upper-bounds it, yielding a computable certificate. We use the Elias $$\delta$$ code, which gives a tighter description-length bound than the Elias $$\gamma$$ code used in prior work.

## Certifying safety oversight models

Let a fixed generator $$G$$ define a distribution over responses $$y$$ to prompts $$x \sim \mathbb{P}_{\mathcal{D}}$$, inducing a joint distribution $$\mathbb{P}_{\mathcal{D},G}$$ over prompt–response pairs. Here _fixed_ refers to the fact that the generator weights, decoding procedure, system prompt, and prompt distribution remain unchanged between certification and deployment. Let $$z^*(x,y) \in \{0,1\}$$ indicate whether a pair satisfies the evaluation criterion, which the judge $$h$$ seeks to emulate. We define the judge's risk with respect to $$\mathbb{P}_{\mathcal{D},G}$$ as

$$
\mathcal{R}_{G}(h) := \mathbb{E}_{(x,y)\sim \mathbb{P}_{\mathcal{D}, G}}\big[\ell(h(x,y), z^*(x,y))\big].
$$

In line with prior compression-based analyses, we restrict attention to deterministic posteriors $$\mathbb{Q}_h = \delta_h$$. Since safety systems typically deploy a single fixed judge to monitor a given generator, this lets the certificate apply directly to the deployed judge without requiring Gibbs sampling from a stochastic posterior.

### The oversight model certificate

For a point-mass posterior under the Solomonoff prior, the KL divergence is upper-bounded by the Elias $$\delta$$ code length of $$h$$,

$$
\mathrm{KL}^{\mathrm{UB}}(\mathbb{Q}_h \,\|\, \mathbb{P}_{\mathrm{Sol}}) = \Bigl(|h| + \lfloor\log_2|h|\rfloor + 2\bigl\lfloor\log_2(\lfloor\log_2|h|\rfloor + 1)\bigr\rfloor + 1\Bigr)\log 2,
$$

and substituting this into the Catoni bound yields a computable certificate. With probability at least $$1-\beta$$ over $$S \sim (\mathbb{P}_{\mathcal{D}, G})^n$$, simultaneously for all judges $$h$$,

$$
\mathcal{R}_{G}(h) \leq \inf_{\lambda>0} \left\{ \frac{1}{1 - e^{-\lambda/n}} \left[ 1 - \exp\left(-\frac{\lambda \widehat{\mathcal{R}}_S(h) + \mathrm{KL}^{\mathrm{UB}}(\mathbb{Q}_h \| \mathbb{P}_{\mathrm{Sol}}) + \log \frac{2+\sqrt{2n}}{\beta}}{n} \right) \right] \right\}.
$$

The certificate thus bounds the risk of the deployed judge under $$\mathbb{P}_{\mathcal{D},G}$$. Its tightness reflects a trade-off between the empirical performance and simplicity of the learned judge. No assumptions are made on the learning algorithm itself: the judge is trained conventionally before its adaptor is quantised and compressed for certification.

### LoRA-GT: global-scale quantised adaptations

The Solomonoff prior assigns greater prior probability to hypotheses with shorter descriptions, formalising an Occam's razor-style preference for simpler hypotheses. Our hypothesis class is accordingly the set of PEFT adaptations of a frozen oversight model, namely LoRA <d-cite key="hu2022lora"></d-cite>, LoRA-XS <d-cite key="balazy2024lora"></d-cite>, and LoRA-T <d-cite key="morris2026learning"></d-cite>, of increasing parameter efficiency. In the weight-tied variant of LoRA-T, a single vector $$\mathbf{v}^{(\ell)} \in \mathbb{R}^u$$ is shared across all adapted modules within each layer $$\ell$$, and the adapted weights take the form

$$
W = W_0 + U \Sigma \Big( \textstyle\sum_{i=1}^u v_i P_i \Big) V^\top,
$$

where $$U, \Sigma, V$$ and $$\{P_i\}_{i=1}^u$$ are fixed.

**Scale overhead.** Under $$b$$-bit quantisation of weight-tied LoRA-T, the per-layer scales stored in bfloat16 cost $$16L$$ bits, dominating $$\lvert h \rvert$$ for small $$u$$ and $$b$$. At $$u{=}1$$, $$b{=}2$$ on Llama-Guard-3-8B ($$L{=}32$$), the weights require only 64 bits while the scales require 512. Sharing a single global scale across layers removes $$(L-1)\times16$$ bits of this metadata, taking the total from 576 to 80 bits, an 86% reduction, and leaves the complexity term set more by the weights themselves than by reconstruction overhead.

## Empirical evaluation

Our main results cover five safety oversight models from two guardrail families, Llama Guard 3 at 1B and 8B <d-cite key="grattafiori2024llama"></d-cite> and Qwen3Guard at 0.6B, 4B and 8B. We use four benchmarks: Egida <d-cite key="garcia2025efficient"></d-cite> ($$n{=}792$$), OpenAI Content Moderation <d-cite key="markov2023holistic"></d-cite> ($$n{=}1{,}344$$), Aegis <d-cite key="ghosh-etal-2025-aegis2"></d-cite> ($$n{=}28{,}984$$), and BeaverTails <d-cite key="ji2023beavertails"></d-cite> ($$n{=}300{,}567$$). Models are trained in a standard way, simply autoregressively via supervised fine-tuning, before adaptor parameters are quantised post-training at 2 and 4 bits.

### Non-vacuous and tight guarantees

{% include figure.liquid path="assets/img/pac-bayes-judges/main_results.png" class="img-fluid" zoomable=true %}

<div class="caption">
  <b>Figure 2. Tight non-vacuous guarantees for safety oversight models.</b> Certified bounds and test risk for 0–1 and Brier losses across models, datasets, and adaptor families. LoRA, whose bounds are vacuous, is shown at its lowest test risk. Highly compressed adaptors achieve tight non-vacuous guarantees at a small cost in test risk.
</div>

Figure 2 shows non-vacuous guarantees under both 0–1 and Brier loss across models, datasets, and adaptor families. Notably, this extends to Egida and OpenAI Moderation with only 792 and 1,344 certification samples, demonstrating certification in the common limited-data regime. The Brier results further provide guarantees on the quality of the predictive probabilities used for thresholded safety decisions.

The strongest certificates consistently come from the simplest adaptations. LoRA-GT is tightest in all 40 model–dataset–loss cases, while standard LoRA is vacuous in every case. This continues even on BeaverTails with $$n\approx300$$k, showing that large certification samples alone do not ensure tight guarantees.

<div class="takeaway">
  Compact safety adaptations yield non-vacuous, often tight guarantees for classification and Brier risk, even with only 792 certification samples.
</div>

### Adaptor compression yields the tightest guarantees

{% include figure.liquid path="assets/img/pac-bayes-judges/compression.png" class="img-fluid" zoomable=true %}

<div class="caption">
  <b>Figure 3. Compression strongly influences certifiability in safety oversight models.</b> (a) Bound vs. description length across models and datasets, showing that smaller, more compressed adaptations yield tighter PAC-Bayes guarantees. (b) LoRA-T variants, where tying and global-scale quantisation reduce description length and tighten bounds with only modest changes in test risk.
</div>

Certificate tightness closely tracks adaptor description length across models and datasets. The description-length ordering, LoRA, LoRA-XS, LoRA-TN, LoRA-TT, then LoRA-GT, matches the certificate ordering in 39 of 40 model–dataset–loss cases, despite the large variation in backbone scale. Adaptation complexity is therefore a stronger determinant of certificate tightness than backbone scale.

Greater compression can weaken predictive performance, although the effect is modest in many settings. On BeaverTails, for example, quantised LoRA-GT has 2.1 to 6.7 points higher 0–1 test risk than LoRA, while its certificate falls from 1.000 to between 0.169 and 0.226.

<div class="takeaway">
  Adaptor description length tracks certificate tightness more closely than backbone scale. LoRA-GT gives the tightest certificates, generally at a small cost in predictive performance.
</div>

### How far compression can go

Let $$h$$ and $$\tilde h$$ denote a fine-tuned judge and its quantised version, respectively. We define _functional distortion_ as the change in risk caused by quantisation:

$$
\Delta_G(h,\tilde h) = \mathbb{E}_{(x,y)\sim\mathbb{P}_{\mathcal{D},G}}\!\left[\ell(\tilde h(x,y),z^*(x,y))-\ell(h(x,y),z^*(x,y))\right].
$$

Compression tightens certificates when the complexity savings from shorter adaptor descriptions outweigh any increase in empirical risk stemming from functional distortion introduced through quantisation.

{% include figure.liquid path="assets/img/pac-bayes-judges/distortion.png" class="img-fluid" zoomable=true %}

<div class="caption">
  <b>Figure 4. Compression and distortion under 0–1 loss.</b> LoRA-GT across five models and \(u\in\{1,2,4\}\). (a) Mean Catoni certificates split into pre-quantisation empirical risk, signed distortion and complexity. (b) Absolute distortion against description length at \(b\in\{2,3,4,8\}\). Colour shows certificate minus test risk and marker size shows test risk. Dashed lines trace the Pareto front.
</div>

The trade-off between complexity savings and functional distortion depends on the certification sample size. Mean LoRA-GT certificates under 0–1 loss favour 2 bits on Egida and OpenAI Moderation, but 4 bits on Aegis and BeaverTails. The complexity penalty depends on $$\lvert\tilde h\rvert/n$$, so on these larger benchmarks further compression saves less, and the added functional distortion at 2 bits loosens the certificate.

<div class="takeaway">
  Aggressive quantisation helps most when certification data are scarce. With more data, quantisation error can outweigh the smaller complexity savings and loosen certificates.
</div>

## Discussion

Our results show that simple PEFT updates can adapt capable safety judges to new safety data while retaining strong predictive performance and admitting tight generalisation guarantees. These guarantees fit within a standard fine-tuning pipeline. The adaptor is trained conventionally and certified post hoc, without constraining the learning algorithm or requiring a separate validation split. The full labelled sample can therefore be used for adaptation and certification, which is particularly relevant when safety data are scarce.

**Limitations.** Our guarantees apply to the generator-induced deployment distribution from which the certification data are drawn. Changes to the generator, decoding configuration, system prompt, or prompt distribution constitute distribution shift <d-cite key="ben2010theory"></d-cite>, which remains beyond the scope of this work. We evaluate guardrail models only up to 8B parameters, a relevant scale for low-cost oversight of larger systems.

## Conclusion

We formalised PAC-Bayes certification for safety judges evaluating outputs from a fixed deployed generator. Across five guardrail models and four benchmarks, tailored PEFT yielded non-vacuous, often tight guarantees for both classification and Brier risk, including in small-data regimes and without a separate held-out validation set. Certificate tightness tracked adaptation complexity more closely than backbone scale, with LoRA-GT giving the tightest certificates in all 40 model–dataset–loss cases. These results show that simple, compressible adaptations can make formal certification practical when specialising safety judges to new taxonomies with scarce labelled data.

## Citation

```bibtex
@inproceedings{lamb2026tight,
  title     = {Tight {PAC}-{Bayes} Generalisation Guarantees for Large Language Model Judges},
  author    = {Lamb, Tom A and Torr, Philip H S and Rudner, Tim G J},
  booktitle = {Advances in Neural Information Processing Systems},
  year      = {2026}
}
```
