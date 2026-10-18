---
title: "TICL heterogeneous inference design"
description: "A device-resident feature-extraction and configurable PyTorchAlpaka inference design for CNN, PFN and DNN models."
tags: [cmssw, alpaka, ticl, gpu, ml-inference]
date: 2026-10-18
draft: false
---

# TICL heterogeneous inference design

## Objective

Investigate a reusable PyTorchAlpaka execution path for TICL, beginning with Trackster PID and energy regression, then assess whether its abstractions generalize to candidate-level inference and superclustering. The goal is to preserve framework queue ownership and GPU-resident data while making model-family selection a configuration decision.

**Status:** experimental architecture discussed and developed on the `heterogeneous-trackster` branch; this note is not a certification of merged or fully validated functionality. The latest branch contents and Work-generated local patches should be reconciled when assessing exact implementation status.

## Producer separation

The proposed split separates feature preparation from inference:

```text
Trackster SoA (GPU)
     |
     v
TracksterInferenceFeaturesProducer
  - on-device feature extraction
  - selection / batching metadata
     |
     v
device-resident feature event product
     |
     v
TracksterInferenceByCNNTorch
  - model execution
  - post-processing / output propagation
```

The historical name `TracksterInferenceByCNNTorch` does not mean the long-term interface should be restricted to a CNN.

A `stream::SynchronizingEDProducer` was considered for feature preparation. The exact synchronization semantics and data-product ownership must be checked against the concrete CMSSW base class and Alpaka integration: adding a synchronizing producer is not itself proof of an asynchronous end-to-end pipeline.

## Interchangeable model selection

The intended Python interface was:

```python
ticlTracksterInference = makeTracksterInference(
    model="PFN",
    doPID=True,
    doRegression=True,
    batchSize=64,
)
```

This expresses a **desired configuration contract**. The C++ code can generalize model loading, input/output binding, batching and regression/PID switching; it cannot infer a model's feature schema solely from the labels CNN/PFN/DNN. Each selected model still requires its proper preprocessing metadata and model artifact.

## SoA and tensor representation

A device-side feature SoA can reduce copies and permit preprocessing to overlap with upstream reconstruction, provided the consuming model can accept the resulting layout or a controlled on-device conversion. A multi-view SoA may help share buffers between producers and consumers. Alignment, stride, selection index mapping and logical-versus-padded size must be documented explicitly.

In earlier refactoring, counting selected elements using the SoA size was questioned: a collection's allocated/logical size does not necessarily equal the count surviving a selection. Preserve an explicit selected count or deterministic mapping where required.

## Model-family adapter boundary

Define explicit contracts for:
- input topology and shape (including variable-size sets);
- normalization, masks and padding;
- max batch size and splitting;
- output names, class ordering and regression meaning;
- shape and memory lifetime of device buffers;
- handling of zero-element batches and per-stream contexts.

A common execution service can be shared; preprocessing and post-processing may remain model-specific.

## Validation ladder

1. **Compile and configuration tests:** all targeted Alpaka backends and model choices build/configure.
2. **Preprocessing validation:** compare device-extracted features with a trusted reference, including padding and selection.
3. **Inference validation:** use actual model files and compare outputs numerically.
4. **Reconstruction validation:** demonstrate equivalent downstream Trackster/candidate properties and physics quality.
5. **Performance validation:** measure copies, host blocking, GPU kernels, memory, streams and event throughput.

Completing a lower step does **not** imply the higher steps passed.

## Related notes

- [[TICL Reconstruction Pipeline]]
- [[Trackster PID and Energy Regression]]
- [[TICL Candidate Inference and Superclustering]]
- [[GPU Inference in CMSSW]]
- [[PyTorch Alpaka]]

## References

- [heterogeneous-trackster branch](https://github.com/EmanueleCoradin/cmssw/tree/heterogeneous-trackster)
- [CMSSW SoA multi-view related PR #51458](https://github.com/cms-sw/cmssw/pull/51458)
- [CMSSW SoA multi-view related PR #51937](https://github.com/cms-sw/cmssw/pull/51937)
- [PyTorchAlpaka allocator PR #52115](https://github.com/cms-sw/cmssw/pull/52115)
