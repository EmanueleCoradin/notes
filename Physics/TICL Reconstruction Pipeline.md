---
title: "TICL reconstruction pipeline"
description: "Conceptual overview of HGCAL layer clustering, Tracksters, candidates and superclustering, with implementation-status caveats."
tags: [cms, hgcal, ticl, reconstruction]
date: 2026-10-18
draft: false
---

# TICL reconstruction pipeline

## Motivation

The CMS High Granularity Calorimeter (HGCAL) measures highly segmented showers in the endcaps. TICL (The Iterative CLustering framework) organizes calorimeter information across layers to reconstruct shower-like objects, combining geometric pattern recognition and learned estimators. Its output contributes to higher-level particle reconstruction.

This page deliberately separates **reconstruction concepts** from the **status of any particular CMSSW configuration**. The default algorithm, modules, and ML models can change between releases and workflow modifiers; they must be established from the actual configuration being studied.

## Data flow

A useful conceptual sketch is:

```text
HGCAL hits -> layer clusters -> cross-layer pattern recognition
             -> Tracksters -> optional linking / inference
             -> TICL candidates -> particle-flow integration
                                 \-> superclustering where configured
```

The sketch is not a literal schedule of EDProducers: inference and superclustering can appear at different stages depending on the algorithm/configuration. Tracksters represent shower candidates built from layer-cluster constituents; TICL candidates are higher-level objects that may incorporate track and calorimeter information.

### Layer clusters, CLUE and CLUE3D

Layer clustering identifies localized energy deposits in individual calorimeter layers. Three-dimensional linking/pattern recognition subsequently associates layer-level information across depth. Do **not** equate CLUEstering and CLUE3D solely by naming: check the dimensionality, neighborhood definition, seeds and exact implementation in the selected release. A generalized clustering framework does not necessarily preserve identical reconstruction behavior.

### Tracksters

A Trackster describes a connected calorimetric shower hypothesis formed from layer-cluster constituents and associated properties. Its geometrical and energy-related features are possible inputs to learned PID and energy correction/regression. Trackster-level inference should not be confused with candidate-level inference.

### Candidates and superclusters

TICL candidates may combine Trackster and charged-track information for downstream reconstruction. Candidate-level PID/energy regression is therefore a distinct inference point with a different semantic input contract.

Superclustering associates multiple reconstructed objects to better account for distributed energy, notably in electromagnetic showers. A DNN-based superclustering step should be documented separately from the CNN/DNN/PFN models proposed for Trackster inference; identical model-family names do not imply identical input features or outputs.

## What is validated and what is not

**Established architectural concepts:** layered calorimeter reconstruction, Trackster and candidate representations, and modular CMSSW reconstruction workflows.

**Under development in our investigation:** device-resident preprocessing and interchangeable PyTorchAlpaka inference for multiple Trackster-level model families; candidate-level and superclustering extension.

**Not established by the current evidence:** a production-ready fully GPU-resident TICL pipeline, identical physics output from CPU and GPU implementations, or the claim that a specific CNN/PFN/DNN is the default model in all current menus.

## Reproducibility checklist

Before declaring an implementation equivalent, record the CMSSW commit, era/modifiers, Python configuration, input data, model files and hashes, and reconstruction-quality metrics. Validate selected Tracksters/candidates and scores, not just whether an event processes successfully.

## Related notes

- [[Trackster PID and Energy Regression]]
- [[TICL Candidate Inference and Superclustering]]
- [[TICL Heterogeneous Inference Design]]
- [[GPU Inference in CMSSW]]

## References

- [CMSSW repository](https://github.com/cms-sw/cmssw)
- [CMSSW TICL configuration area](https://github.com/cms-sw/cmssw/tree/master/RecoHGCal/TICL)
- [Phase-2 workflow configuration including TICL superclustering variants](https://github.com/cms-sw/cmssw/blob/master/Configuration/PyReleaseValidation/python/upgradeWorkflowComponents.py)
- [Development branch: heterogeneous-trackster](https://github.com/EmanueleCoradin/cmssw/tree/heterogeneous-trackster)
