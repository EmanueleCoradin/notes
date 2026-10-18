---
title: "Research notes"
description: "Notes on physics, heterogeneous computing and machine learning inference."
---

# Research notes

A growing collection of personal research and technical study notes, organized for [Obsidian](https://obsidian.md/) and published using [Quartz](https://quartz.jzhao.xyz/).

## Heterogeneous ML inference in CMSSW

- [[GPU Inference in CMSSW]] — framework-level execution, concurrency, memory and portability
- [[PyTorch Alpaka]] — integrating PyTorch with Alpaka-managed GPU inference
- [[ONNXRuntimeAlpaka]] — ONNX Runtime execution providers, IOBinding and tensor layouts
- [[TensorRT Inference in CMSSW]] — deployment choices, lean runtime and native executor considerations


## TICL and HGCAL reconstruction

- [[TICL Reconstruction Pipeline]] — layer clustering, Tracksters and candidate reconstruction
- [[Trackster PID and Energy Regression]] — learned classification and energy estimation
- [[TICL Heterogeneous Inference Design]] — GPU feature extraction and configurable CNN/PFN/DNN execution
- [[TICL Candidate Inference and Superclustering]] — candidate-stage estimators and learned grouping

## Research logs

- [[Pixel Track Inference Backend Comparison]] — exploratory PyTorchAlpaka / ONNXRuntimeAlpaka timing comparison

## Background

- [[CMSSW]] — CMS software framework

These notes distinguish measured observations from design ideas and open questions. They are evolving research notes, not official CMS software documentation.
