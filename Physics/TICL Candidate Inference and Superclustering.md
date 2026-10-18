---
title: "TICL candidate inference and superclustering"
description: "Separating candidate-level learned estimators from Trackster inference and DNN-based superclustering."
tags: [cms, hgcal, ticl, machine-learning, superclustering]
date: 2026-10-18
draft: false
---

# TICL candidate inference and superclustering

## Why this is a distinct stage

A Trackster describes a calorimeter shower hypothesis. A TICL candidate is a higher-level reconstructed object that can incorporate the Trackster and additional contextual information, including charged tracking. Accordingly, candidate-stage PID or energy regression should be treated as a **new inference contract**, even if it reuses the same PyTorchAlpaka executor.

Superclustering is a separate association task: deciding which reconstructed showers or objects should be grouped to form a more complete energy deposit or electromagnetic candidate. Its DNN is not automatically interchangeable with a candidate PID classifier or energy regressor.

## Candidate-stage PID and regression

Before implementing this stage, identify from the configuration and actual model:
- candidate constituents and their device/host availability;
- which stage owns the authoritative PID and energy estimates;
- target outputs and downstream consumers;
- whether candidate-to-Trackster associations can stay device-resident;
- how empty associations and track-linked candidates are handled.

A generic `doPID`/`doRegression` interface can be reused, but the feature schema and post-processing need not be the same as at Trackster stage.

## Superclustering DNN

A learned grouping stage may consume pairs, sets, or aggregate features of Tracksters/candidates, depending on the trained network. Its actual feature definition, score semantics, thresholding/assignment strategy and supported multiplicities **must not be inferred from the term 'DNN'**.

Potential issues include dynamic object counts, candidate pair enumeration, duplicate group assignments and maintaining consistent index mappings. A fixed-size padded tensor may simplify deployment but changes memory requirements as multiplicity grows.

## What has and has not been demonstrated

**Grounded in earlier development discussions:** the intention to extend the generalized CNN/PFN/DNN infrastructure to candidate reconstruction and superclustering, with minimal changes to the established abstractions.

**Not confirmed here:** final C++ implementation, availability of trained weights, successful end-to-end inference, equivalence to a CPU reference, and actual GPU throughput. Earlier Work changes may have existed only locally rather than on the remote branch; inspecting the remote alone is not conclusive evidence of completion.

**Evidence of a configurable superclustering workflow:** CMSSW contains Phase-2 workflow variants for TICL superclustering. The existence of a workflow modifier does not demonstrate that the proposed PyTorchAlpaka GPU path is active.

## Validation plan

1. Trace the selected candidate and superclustering producers in the CMSSW release/configuration.
2. Document precise model contracts from actual artifacts and configuration.
3. Reuse the common executor; specialize feature builders and output adapters as needed.
4. Validate preprocessing without weights if necessary, clearly marking it partial.
5. Compare scores, grouped objects, reconstructed energies and edge cases against reference workflows.
6. Benchmark host/device movement, batching cost, memory and full event throughput.

## Related notes

- [[TICL Reconstruction Pipeline]]
- [[Trackster PID and Energy Regression]]
- [[TICL Heterogeneous Inference Design]]
- [[PyTorch Alpaka]]

## References

- [CMSSW TICL area](https://github.com/cms-sw/cmssw/tree/master/RecoHGCal/TICL)
- [TICL superclustering workflow variants](https://github.com/cms-sw/cmssw/blob/master/Configuration/PyReleaseValidation/python/upgradeWorkflowComponents.py)
- [heterogeneous-trackster development branch](https://github.com/EmanueleCoradin/cmssw/tree/heterogeneous-trackster)
