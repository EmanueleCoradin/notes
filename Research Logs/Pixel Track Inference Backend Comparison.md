---
title: "Pixel Track inference backend comparison"
description: "Exploratory GPU kernel-time comparison of PyTorchAlpaka and ONNXRuntimeAlpaka for Pixel Track high-purity selection."
tags: [research-log, cmssw, benchmarking, gpu, ml-inference]
date: 2026-10-18
draft: false
---

# Pixel Track inference backend comparison

## Research question

How does the GPU inference activity of PyTorchAlpaka compare with ONNXRuntimeAlpaka on the same Pixel Track high-purity selection model under a multi-stream CMSSW workload?

## Experimental configuration

The original exploratory tests were reported using:
- node with **two NVIDIA H100 GPUs**;
- **four jobs, 16 framework streams and 16 threads** (as reported in the test configuration);
- matching model, minibatches and numerical precision;
- GPU activity inspected using **Perfetto**, with additional GPU monitoring.

The available record does not fully specify driver/library versions, the distributions of actual batch sizes, warm-up procedure, number of repetitions, clock variance or error bars. It should not be treated as a reproducible published benchmark without these details.

## Observations

| Metric | PyTorchAlpaka | ONNXRuntimeAlpaka |
| --- | ---: | ---: |
| Measured GPU kernel time per batch | approximately **300 μs** | approximately **450 μs** |
| Observed kernels per batch | approximately **46** | approximately **88** |
| Mean GPU memory per stream | similar in reported test | similar in reported test |

For this run, the ONNXRuntimeAlpaka reported batch kernel time was about 1.5 times the PyTorchAlpaka figure and showed nearly twice as many kernel launches. This is an **observation for one implementation and workload**, not a general ordering of the backends.

The PyTorchAlpaka baseline did not include subsequent improvements involving a batched tensor abstraction and the CMSSW caching allocator.

## Interpretation and limitations

Differences could stem from graph representation, provider/operator implementation, fusion, tensor packing/unpacking and launch overhead. The reported kernel counts do not isolate any one of these factors. Similar mean device memory per stream does not imply equivalent peak memory or allocation overhead.

The timing values are **GPU kernel activity**, not necessarily the complete CMSSW producer latency or end-to-end event throughput. No statistical uncertainty was recorded here.

## Reproduction and follow-up

- [ ] Record exact CMSSW/ONNX Runtime/PyTorch versions and model hashes.
- [ ] Measure warm and cold execution separately.
- [ ] Profile pack, inference and unpack stages independently.
- [ ] Capture throughput, host blocking and end-to-end latency.
- [ ] Compare with the same allocator policy and corresponding tensor collection design.
- [ ] Check numerical agreement and effect of FP16 or alternative execution providers.

## Related notes

- [[PyTorch Alpaka]]
- [[ONNXRuntimeAlpaka]]
- [[GPU Inference in CMSSW]]

## Sources

- [ONNXRuntimeAlpaka implementation PR #51958](https://github.com/cms-sw/cmssw/pull/51958)
- [PyTorchAlpaka caching allocator PR #52115](https://github.com/cms-sw/cmssw/pull/52115)
- Experimental measurements discussed during the implementation investigation; raw Perfetto traces are not attached to this note.
