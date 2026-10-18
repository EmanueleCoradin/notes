---
title: "GPU inference in CMSSW"
description: "Architecture, portability, concurrency and performance considerations for heterogeneous ML inference in the CMS framework."
tags: [cmssw, gpu, ml-inference, alpaka]
date: 2026-10-18
draft: false
---

# GPU inference in CMSSW

## Motivation

CMS reconstruction and trigger workflows increasingly use neural networks. Efficient inference is not just a question of choosing an inference engine: models execute inside the CMSSW event-processing framework, alongside heterogeneous reconstruction algorithms and with many concurrent events or streams. A useful integration must control synchronization, host–device transfers, memory footprint, model initialization, and runtime portability.

This note describes engineering principles and links to the implementations examined in the related notes: [[PyTorch Alpaka]], [[ONNXRuntimeAlpaka]], and [[TensorRT Inference in CMSSW]].

## The execution boundary

CMSSW can execute work concurrently in multiple framework streams. GPU algorithms implemented through Alpaka operate on device queues associated with framework execution. A model runner must therefore answer:

1. **Data residency:** Are model inputs already in device memory, or must they be copied from the host?
2. **Layout:** Does the model require contiguous sample-major tensors, or can it consume the reconstruction SoA layout directly?
3. **Queue integration:** Does the engine enqueue inference on the framework-owned queue, or does it execute or synchronize on a separate stream?
4. **Lifetime:** Who owns inputs, outputs, temporary tensors, executable models and workspace allocations, and for how long?
5. **Concurrency:** Can sessions/modules be shared safely between framework streams? Are contexts or model instances replicated?
6. **Fallback:** What happens when a GPU backend, model representation or accelerator capability is unavailable?

A useful idealized pipeline is:

```text
Alpaka reconstruction -> device-resident features -> inference engine
                      -> device-resident predictions -> reconstruction
```

Copying to host for every inference or synchronizing between steps can erase otherwise promising kernel-time improvements.

## Inference engines and the portability problem

| Engine | Role | Main integration question |
| --- | --- | --- |
| PyTorchAlpaka | Execute PyTorch models with CMSSW/Alpaka integration | Queue ownership, allocator, thread safety and model memory |
| ONNXRuntimeAlpaka | Execute exported ONNX models through ONNX Runtime providers | IOBinding, device-memory compatibility, tensor packing and synchronization |
| TensorRT | NVIDIA-specific optimized execution of engines | Engine portability, runtime dependencies, CUDA streams and fallback |

These are *execution choices*, not interchangeable guarantees of numerical equivalence or identical model coverage. Export support, precision, operator availability and provider behavior must be validated per model.

## Memory and tensor layout

A typical reconstruction SoA stores each feature in a separate column. Many model interfaces expect contiguous tensors of shape `[batch, features]`; transforming between them adds an SoA-to-AoS packing kernel and potentially an output unpacking kernel. A feature-major model representation could sometimes avoid the transformation, but only when the exported model and operator stack actually support it.

A caching allocator can reduce repeated device-allocation overhead, but does not by itself eliminate model weights, workspaces, per-stream execution contexts or transient activations. Compare both **allocated** and **reserved** memory, and measure per framework stream and per inference engine.

## Synchronization and concurrency

A framework-aware inference producer should aim to enqueue work on the queue associated with its event and avoid a blocking device-wide synchronization. GPU kernels being asynchronous does not establish end-to-end asynchronous behavior: API calls, provider copies, engine setup and output access can still synchronize.

An engine's thread safety must be checked at the relevant granularity: session, execution context, model object, allocator and queue. Avoid assuming that one global model object is safe simply because an inference API appears read-only.

## Benchmark methodology

For meaningful comparisons, fix:
- the model graph, weights, preprocessing and numerical precision;
- representative batch-size distributions and padding;
- device type, driver/runtime, available GPU count and CPU environment;
- CMSSW jobs, framework streams and threads;
- warm-up, measurement duration, statistical treatment and synchronization boundaries.

Record **end-to-end producer latency**, GPU kernel activity, host time, transfer volume, peak/reserved memory and throughput. Isolated kernel timings alone do not establish a framework-level speedup.

For an exploratory comparison with measured numbers, see [[Pixel Track Inference Backend Comparison]].

## Questions for continued investigation

- Can feature extraction, inference and consumption remain fully device-resident in TICL?
- Can execution providers reuse CMSSW queues and allocation policies without hidden synchronization?
- When is per-stream model/context replication necessary?
- Which inference representations remain portable across CUDA, ROCm and CPU backends?

## References

- [CMSSW source code](https://github.com/cms-sw/cmssw)
- [Alpaka project](https://github.com/alpaka-group/alpaka)
- [PyTorchAlpaka allocator integration PR #52115](https://github.com/cms-sw/cmssw/pull/52115)
- [ONNXRuntimeAlpaka PR #51958](https://github.com/cms-sw/cmssw/pull/51958)
