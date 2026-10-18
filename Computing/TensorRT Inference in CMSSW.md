---
title: "TensorRT inference in CMSSW"
description: "Trade-offs among TensorRT via ONNX Runtime, lean runtimes and native Alpaka-integrated execution."
tags: [cmssw, tensorrt, gpu, inference, cuda]
date: 2026-10-18
draft: false
---

# TensorRT inference in CMSSW

## Motivation and scope

NVIDIA TensorRT optimizes neural-network inference for NVIDIA GPUs. Using it in CMSSW requires more than converting an ONNX model: deployability depends on packaged libraries, engine compatibility, framework CUDA stream integration, memory ownership and a fallback path on unsupported hardware.

See [[GPU Inference in CMSSW]] and [[ONNXRuntimeAlpaka]] for the wider execution context.

## Three integration architectures

### 1. ONNX Runtime TensorRT execution provider

Use an ONNX Runtime session with the TensorRT EP, potentially deploying a precompiled engine through EPContext. Benefits include reuse of ONNX Runtime APIs and the prospect of fallback to another provider. Costs include ONNX Runtime integration complexity, potential packaging overhead and provider-specific behavior around stream management and engine initialization.

**Important:** fallback requires explicit provider selection and validation. A precompiled engine does not automatically run on a different GPU or execution provider.

### 2. TensorRT EP with a lean runtime

Experiment with an EPContext-only workflow that uses the smaller TensorRT lean runtime rather than the full inference/build stack. In the local packaging investigation, a reduced runtime around 732 MB and an experimental lean dependency around 102 MB were considered. These are **configuration-specific library footprint observations**, not guaranteed installed sizes or proof that inference works.

The experimental lean-runtime patch compiled/linked, but GPU execution had not been validated at the time of the investigation. API availability and engine serialization/runtime compatibility remain gating constraints.

### 3. Native TensorRTAlpaka executor

Execute precompiled TensorRT engines directly using CUDA streams managed by Alpaka, potentially by passing the underlying native stream to TensorRT's `enqueueV3()`. This could reduce indirection and make stream ownership explicit, while introducing a new backend responsible for engine loading, tensor bindings, execution contexts, shape handling, allocations and error propagation.

A native executor is not portable to AMD GPUs. A separate model representation and fallback engine (e.g. ONNX Runtime) would still be necessary.

## Engine portability and deployment

Precompiled TensorRT plans are sensitive to the build/runtime environment and GPU compatibility rules. Compatibility modes can broaden support under specific constraints, but engine portability must not be assumed. A sound deployment procedure records TensorRT/CUDA versions, supported hardware, network input profiles and precision.

Dynamic shapes require consistent optimization profiles and runtime binding shapes. An engine compiled for one tensor-shape envelope cannot safely be treated as supporting arbitrary shapes.

## Precision and quantization

FP16, INT8 and newer formats can improve throughput or footprint under suitable hardware and model support. Quantization needs numerical validation against physics requirements. For INT8, representative calibration or quantization-aware scales typically matter: random Gaussian calibration data may be useful for a smoke test of an API path, but not for trustworthy physics-performance claims.

Distinguish changes in *kernel speed* from changes in prediction quality, memory footprint and engine startup time.

## Concurrency and known open issues

A multi-stream ONNX Runtime TensorRT EP experiment hit duplicate registration of the `trt.plugins` domain; details are recorded in [[ONNXRuntimeAlpaka]]. Its actual cause requires investigation. For a native backend, execution contexts and buffer bindings must be managed per concurrent invocation as required by the TensorRT API.

## Decision criteria

| Criterion | ORT TensorRT EP | Lean-runtime EP | Native TensorRTAlpaka |
| --- | --- | --- | --- |
| Integration effort | Reuses ONNX infrastructure | Additional EP packaging and compatibility work | Requires a new executor |
| CUDA stream control | Provider-specific | Provider-specific | Potentially direct |
| Runtime dependencies | Potentially large | Potentially smaller | Potentially small with lean runtime |
| GPU portability | TensorRT path is NVIDIA-only; other ORT EPs can provide alternatives | Same | NVIDIA-only plus external fallback |
| Current evidence | In experimental use | Build/link explored; inference unvalidated | Design proposal, not an implemented benchmark |

No choice is universally preferable without deployment tests, reproducible end-to-end measurements and compatibility checks.

## Related notes

- [[GPU Inference in CMSSW]]
- [[ONNXRuntimeAlpaka]]
- [[PyTorch Alpaka]]

## References

- [TensorRT documentation](https://docs.nvidia.com/deeplearning/tensorrt/latest/)
- [ONNX Runtime TensorRT execution provider](https://onnxruntime.ai/docs/execution-providers/TensorRT-ExecutionProvider.html)
- [Experimental CMSSW TensorRT branch](https://github.com/EmanueleCoradin/cmssw/tree/tensorrt-onnx-HP)
- [Experimental cmsdist packaging branch](https://github.com/EmanueleCoradin/cmsdist/tree/tensorRT)
