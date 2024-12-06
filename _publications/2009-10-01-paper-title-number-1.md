---
title: "Faithful Knowledge Distillation [Preprint]"
collection: publications
permalink: /publication/2023-08-11-paper-title-number-1
excerpt: 'Knowledge distillation; verification; robustness'
date: 23-08-11
venue: 'Arxiv'
paperurl: 'https://arxiv.org/abs/2306.04431'
citation: 'Lamb, Tom A., et al. "Faithful Knowledge Distillation." arXiv preprint arXiv:2306.04431 (2023).'
---
**Abstract:** Knowledge distillation (KD) has received much attention due to its success in compressing networks to allow for their deployment in resource-constrained systems. While the problem of adversarial robustness has been studied before in the KD setting, previous works overlook what we term the relative calibration of the student network with respect to its teacher in terms of soft confidences. In particular, we focus on two crucial questions with regard to a teacher-student pair: (i) do the teacher and student disagree at points close to correctly classified dataset examples, and (ii) is the distilled student as confident as the teacher around dataset examples? These are critical questions when considering the deployment of a smaller student network trained from a robust teacher within a safety-critical setting. To address these questions, we introduce a faithful imitation framework to discuss the relative calibration of confidences and provide empirical and certified methods to evaluate the relative calibration of a student w.r.t. its teacher. Further, to verifiably align the relative calibration incentives of the student to those of its teacher, we introduce faithful distillation. Our experiments on the MNIST, Fashion-MNIST and CIFAR-10 datasets demonstrate the need for such an analysis and the advantages of the increased verifiability of faithful distillation over alternative adversarial distillation methods.

[Download paper here](https://arxiv.org/pdf/2306.04431.pdf)

**Recommended (bib) citation:* 

@article{lamb2023faithful,
  title={Faithful Knowledge Distillation},
  author={Lamb, Tom A and Brunel, Rudy and Kumar, M Pawan and Torr, Philip HS and Eiras, Francisco and others},
  journal={arXiv preprint arXiv:2306.04431},
  year={2023}
}