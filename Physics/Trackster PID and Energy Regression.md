---
title: "Trackster PID and energy regression"
description: "The distinct physical and computational roles of classification and energy estimation at Trackster level."
tags: [cms, hgcal, ticl, machine-learning]
date: 2026-10-18
draft: false
---

# Trackster PID and energy regression

## Two different tasks

**Particle identification (PID)** uses shower morphology and other inputs to estimate the compatibility of a Trackster with particle classes. Its outputs can be class scores or probabilities depending on the model and training objective.

**Energy regression** estimates a corrected or calibrated energy given a shower representation. The output semantics must be checked: a model may predict an absolute energy, multiplicative correction or residual. Applying the wrong interpretation can silently corrupt reconstruction.

Their relative importance for any particular physics workflow is an experimental and reconstruction choice, not a property of PyTorch or Alpaka. In our investigation, energy regression was highlighted as especially relevant, but this discussion alone does not establish which model or regression stage is deployed by default.

## Input representations

A Trackster may offer aggregate features (energy sums, layer occupancy, shower depth/shape, barycenter, etc.) or a structured collection of constituents. Different model families require different preprocessing contracts:

| Family | Illustrative representation | Integration concern |
| --- | --- | --- |
| CNN | Dense or image-like shower tensors | Grid construction, shape and padding |
| DNN / MLP | Fixed-length feature vector | Feature ordering, normalization |
| PFN | Set of per-constituent feature vectors | Masking, padding, permutation handling |

These are **model-family examples**, not verified shape declarations for any particular TICL `.pt` file. Actual tensor shapes, normalizations, class ordering and output names must come from model artifacts or configuration.

## Numerical and physics validation

A correct GPU integration needs more than a successful `forward()`. Check:
- agreement with reference preprocessing, including feature order and default values;
- consistency of padding and masks across different constituent counts;
- reproducibility within explicit floating-point tolerances;
- PID class order, score normalization and downstream label interpretation;
- calibration and bias of regressed energy over shower energy, pseudorapidity, pileup and particle type;
- behavior for empty, pathological or oversized Tracksters.

A unit test validating preprocessing without model weights is a useful **partial result**, not a validation of inference or physics equivalence.

## Stage boundaries

Trackster inference occurs on shower hypotheses before or during higher-level assembly. A candidate may combine additional objects or contextual information, so repeating an estimator after candidate reconstruction is not necessarily redundant. Distinguish task name (PID/energy regression) from object level (Trackster/candidate) and from model architecture (CNN/PFN/DNN).

## Open questions

- Which model artifacts and output conventions are currently deployed under each TICL modifier?
- Is a later-stage estimator intended to replace, supplement or refine earlier predictions?
- Can all features be constructed on GPU without altering reconstruction selection?
- How do preprocessing and inference costs scale with batch size and constituent multiplicity?

## Related notes

- [[TICL Reconstruction Pipeline]]
- [[TICL Heterogeneous Inference Design]]
- [[TICL Candidate Inference and Superclustering]]
- [[PyTorch Alpaka]]

## References

- [CMSSW TICL software](https://github.com/cms-sw/cmssw/tree/master/RecoHGCal/TICL)
- [Development branch: heterogeneous-trackster](https://github.com/EmanueleCoradin/cmssw/tree/heterogeneous-trackster)
