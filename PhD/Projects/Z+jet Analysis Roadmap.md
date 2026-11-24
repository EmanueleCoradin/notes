---
title: "Z+jet analysis roadmap"
description: "Reproducibility, channel extension and methodological questions for Z-tagged jet studies."
tags:
  - cms
  - heavy-ion
  - jet-quenching
date: 2026-11-24
draft: false
---

# Z+jet analysis roadmap

Z-tagged jets in pp and PbPb collisions provide a way to investigate jet modification in the quark-gluon plasma. The colorless Z boson constrains the recoil kinematics, but does not uniquely determine the energy or angular structure of the original parton shower. Radiation, recoil topology and selection effects must be accounted for.

## Analysis development

The technical baseline and workspace organization are documented in [[Computing/HiForest Setup|HiForest Setup]]. Generic forest smoke tests and compilation are technical checks, not a reproduction of a physics measurement.

The proposed sequence is:

1. Understand the existing dimuon analysis, including the forest schema, event and object selection, corrections, weights, background treatment and uncertainty propagation.
2. Establish a reproducible dimuon reference with fixed inputs, configurations, code revision and reference outputs.
3. Identify the genuinely shared Z+jet logic and isolate channel-specific components only where needed. Compare the refactored dimuon implementation against the reference before introducing new physics selections.
4. Investigate the dielectron channel, including reconstruction, identification, isolation, triggers, energy calibration and resolution, efficiency corrections, backgrounds and systematic uncertainties.
5. Compare the channels in a compatible fiducial phase space before considering a combination, accounting for correlated and uncorrelated uncertainties.

No physics reproduction, refactoring or electron-channel validation is claimed here.

## Validation strategy

For controlled comparisons, retain the same input events and correction versions, compare event and object selections, cutflows, weighted yields, and histogram contents (including uncertainties). Where practical, compare event-level quantities to localize discrepancies. Define numerical tolerances before accepting changes and investigate differences rather than relying only on agreement in aggregate plots.

Maintain a fixed reference workspace and independent development workspaces, and record the provenance of each output. See [[Computing/HiForest Setup|HiForest Setup]] for the technical organization.

## Methodological questions

These are open questions rather than established results or a finalized measurement proposal.

- How can tracking efficiency and fake rates in dense jet environments be characterized as functions of track momentum, angular distance from the jet axis and local particle density?
- How does reconstructed jet response depend on fragmentation and angular structure?
- How can reconstruction-induced migration and selection effects be distinguished from changes in the selected jet population due to physical energy loss?
- Which observables retain information about the initial angular structure of a jet, and how can their correlations and limitations be tested in simulation?

Start from published CMS reconstruction and performance studies, then establish whether their phase space and definitions are appropriate for a given study. Distinguish generator-level definitions, detector-level observables and model-dependent interpretations. A reconstructed jet radius is not a direct measurement of the unquenched shower radius.

Keep unpublished collaboration material, internal results, datasets and analysis-specific decisions in appropriate private documentation. This public page records general methodology only.

## Related notes

- [[Computing/HiForest Setup|HiForest Setup]]
- [[Computing/HiForest Output Format|HiForest Output Format]]
- [[Physics/Heavy-Ion/index|Heavy-ion physics]]
- [[PhD/Literature/index|Literature]]
