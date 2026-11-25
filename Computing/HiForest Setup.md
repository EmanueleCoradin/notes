---
title: "HiForest Setup"
description: "Reproducible CMSSW environments, reference and development workspaces, and an interactive launcher for heavy-ion analysis."
tags:
  - computing
  - cms
  - heavy-ion
  - cmssw
date: 2026-11-24
draft: false
---

# HiForest Setup

HiForest converts [[Computing/CMS MiniAOD|CMS MiniAOD]] into ROOT analysis trees using [[Computing/CMSSW|CMSSW]] producers and analyzers. See [[Computing/HiForest Output Format|HiForest Output Format]] for the structure of the original smoke-test output.

The working environment uses an EL8 CMS container on an EL9 host. The initial generic HiForest tests used CMSSW_13_2_4; the Trieste analysis framework now uses CMSSW_13_2_13. These are distinct configurations and their outputs must not be assumed to have identical schemas.

## Trieste analysis baseline

The [Trieste HiForest fork](https://github.com/cms-ts/cmssw/tree/ts_forest_CMSSW_13_2_X_v2) includes both dedicated foresting configurations and downstream ROOT analysis macros. Its [setup README](https://github.com/cms-ts/cmssw/blob/ts_forest_CMSSW_13_2_X_v2/HeavyIonsAnalysis/README) specifies CMSSW_13_2_13 and `el8_amd64_gcc11`.

The baseline selected on 2026-11-24 is commit [24818506602796f751e8665cd31aca601e06f1cb](https://github.com/cms-ts/cmssw/commit/24818506602796f751e8665cd31aca601e06f1cb) from `ts_forest_CMSSW_13_2_X_v2`. The local checkout initially contained no materialized packages despite having the correct Git HEAD. After the sparse checkout issue was addressed, the user reported successful compilation. This is a build check, not validation of forest production or reproduction of the physics results.

## Workspace organization

Use a persistent shared filesystem with a convenient home-directory symlink `~/HeavyIons`. Site-specific filesystem paths and bind mounts are configured locally.

```text
HeavyIons/
├── reference/
│   ├── CMSSW_13_2_13/
│   └── results/
├── development/
│   └── zjet-dielectron/
│       ├── CMSSW_13_2_13/
│       ├── notebooks/
│       └── results/
├── data/
│   ├── samples/
│   ├── forests/
│   └── test/
├── production/
│   ├── configs/
│   ├── jobs/
│   └── logs/
└── validation/
    ├── notebooks/
    ├── scripts/
    └── comparisons/
```

The reference workspace preserves the upstream dimuon implementation at a fixed revision (a detached HEAD is useful). Development workspaces have independent Git branches and SCRAM builds. The first planned development workspace, `zjet-dielectron`, is intended to extend the existing analysis to electrons; additional workspaces can be introduced as needed.

Share immutable input datasets and compatible forest productions, but keep results and modified forest productions separate. Each production should record its source revision, CMSSW release, input file list, configuration and correction versions.

The layout is the intended organization; the workspace directory creation was observed locally, but the final rename, relocated builds and multi-workspace launcher have not all been independently verified.

## Creating a CMSSW workspace

First configure the local shared-storage path and bind mount. Enter the EL8 container from the host, substituting the appropriate mount path:

```bash
/cvmfs/cms.cern.ch/common/cmssw-el8 -B /path/to/shared/storage
```

Inside the container, a fresh reference installation can be created as follows:

```bash
source /cvmfs/cms.cern.ch/cmsset_default.sh
export SCRAM_ARCH=el8_amd64_gcc11

mkdir -p "$HOME/HeavyIons/reference/results"
cd "$HOME/HeavyIons/reference"
cmsrel CMSSW_13_2_13
cd CMSSW_13_2_13/src
cmsenv

git cms-init
git remote add trieste-cmssw https://github.com/cms-ts/cmssw.git
git fetch trieste-cmssw ts_forest_CMSSW_13_2_X_v2
git checkout --detach 24818506602796f751e8665cd31aca601e06f1cb

git cms-sparse-checkout CMSSW_13_2_13 HEAD
git read-tree -mu HEAD

ls HeavyIonsAnalysis/Configuration/test/analyze/
git status --short
git rev-parse HEAD
scram b -j 8
```

This is a pinned-checkout alternative to the repository's merge-topic instructions. The exact fresh-install sequence is illustrative, not a second independently executed installation. Use appropriate compilation parallelism for the site.

### Sparse checkout

CMSSW uses a sparse working tree to avoid materializing the entire source repository. A successful Git checkout or an apparently successful SCRAM build does not prove that modified packages are present.

```bash
git config --get core.sparseCheckout
cat .git/info/sparse-checkout
git diff --name-only CMSSW_13_2_13 HEAD | cut -d/ -f1,2 | sort -u
git cms-sparse-checkout CMSSW_13_2_13 HEAD
git read-tree -mu HEAD
```

The [CMS sparse-checkout helper](https://github.com/cms-sw/cms-git-tools/blob/master/git-cms-sparse-checkout) adds packages differing between the release tag and the selected revision to the sparse-checkout rules; `git read-tree -mu HEAD` applies those rules to the working tree. Inspect the packages before building.

### Development and relocation

Create an independent SCRAM project in each development directory, initialize the same source revision and sparse package list as the reference, and create a development branch (e.g. `git switch -c zjet-dielectron` after checkout). A local Git clone can avoid downloading the repository again; ensure the destination source directory is empty, preserve the sparse-checkout rules and verify the base release tag and pinned commit are available. Keep development commits in a suitable personal fork rather than pushing to the reference branch.

When moving an already compiled CMSSW area, stop jobs and notebook kernels using it, check the Git working tree and use `scram b ProjectRename` from the relocated source directory in the compatible container. Reinitialize the runtime and rebuild as needed. Verify paths and Git state before removing the original area. Do not delete the legacy release until it is no longer needed.

## Interactive environment

The launcher selects the workspace rather than switching Git branches in a shared build. Configure `HIFOREST_BIND` in the local host shell configuration with the filesystem mount required by the container (a site-specific value is intentionally omitted).

Append to the host's `~/.bashrc`:

```bash
hiforest() {
    local workspace="${1:-zjet-dielectron}"

    if [[ ! "$workspace" =~ ^[a-zA-Z0-9_-]+$ ]]; then
        echo "Invalid workspace name"
        return 1
    fi

    if [[ -z "${HIFOREST_BIND:-}" ]]; then
        echo "Set HIFOREST_BIND to your shared-storage mount"
        return 1
    fi

    /cvmfs/cms.cern.ch/common/cmssw-el8 \
        -B "$HIFOREST_BIND" \
        --command-to-run "export HIFOREST_PROFILE=$workspace; exec bash --rcfile \"\$HOME/.hiforestrc\" -i"
}
```

Create `~/.hiforestrc`:

```bash
source ~/.bashrc

HIFOREST_WORK="$(cd "$HOME/HeavyIons" && pwd -P)" || exit 1
export HIFOREST_WORK
export SCRAM_ARCH=el8_amd64_gcc11
export HIFOREST_PROFILE="${HIFOREST_PROFILE:-zjet-dielectron}"

source /cvmfs/cms.cern.ch/cmsset_default.sh

if [[ "$HIFOREST_PROFILE" == reference ]]; then
    export HIFOREST_WORKSPACE="$HIFOREST_WORK/reference"
else
    export HIFOREST_WORKSPACE="$HIFOREST_WORK/development/$HIFOREST_PROFILE"
fi

export HIFOREST_DATA="$HIFOREST_WORK/data"
export HIFOREST_RESULTS="$HIFOREST_WORKSPACE/results"
export HIFOREST_VALIDATION="$HIFOREST_WORK/validation"

cd "$HIFOREST_WORKSPACE/CMSSW_13_2_13/src" || exit 1
runtime="$(scram runtime -sh)" || exit 1
eval "$runtime" || exit 1
unset runtime

export CFG_DIR="$CMSSW_BASE/src/HeavyIonsAnalysis/Configuration/test"
export HIFOREST_ANALYSIS="$CFG_DIR/analyze"

cd "$HIFOREST_WORKSPACE" || exit 1
echo "Workspace: $HIFOREST_PROFILE"
echo "CMSSW: $CMSSW_VERSION"
echo "Analysis: $HIFOREST_ANALYSIS"
echo "Results: $HIFOREST_RESULTS"
PS1='[HiForest:${HIFOREST_PROFILE} \W]\$ '
```

Reload and test from the host:

```bash
source ~/.bashrc
hiforest reference
# Verify inside the container:
echo "$CMSSW_BASE"
git -C "$CMSSW_BASE/src" rev-parse HEAD
exit

hiforest
# Verify the development area:
echo "$CMSSW_BASE"
git -C "$CMSSW_BASE/src" rev-parse HEAD
git -C "$CMSSW_BASE/src" branch --show-current
```

Initially the two revisions should match, while the local CMSSW paths differ. `hiforest` defaults to the dielectron development workspace; `hiforest reference` activates the baseline. The launcher requires no Git operations or recompilation during normal startup. This is the proposed revised launcher, based on the previously tested single-release launcher; verify it on the farm before relying on it.

## Analysis code and configurations

See [[Computing/Trieste Z+jet Analysis Workflow|Trieste Z+jet Analysis Workflow]] for the processing stages and script relationships.

The [analysis directory](https://github.com/cms-ts/cmssw/tree/24818506602796f751e8665cd31aca601e06f1cb/HeavyIonsAnalysis/Configuration/test/analyze) is accessible through `cd "$HIFOREST_ANALYSIS"`.

| Component | Purpose |
| --- | --- |
| `analyze_HI_TTreeReader_ZMM.C` | Main dimuon Z+jet analysis |
| `JetCorrector.h`, `JERProvider.h` | Jet corrections and resolution |
| `JetSelection_PbPb.h`, `JetSelection_pp.h` | Jet selections |
| `CorrectionSF.h`, `MC_samples.h` | Scale factors and sample definitions |
| `MixEvSub/`, `weights_MC/` | Background subtraction and MC reweighting |
| `pp_vs_PbPb/`, `syst_*/` | Comparisons and systematic studies |

The configuration directory contains dedicated ZMM DATA and MC forests, a generator-level ZMM configuration and pp reference configurations. Consult the [analysis README](https://github.com/cms-ts/cmssw/blob/24818506602796f751e8665cd31aca601e06f1cb/HeavyIonsAnalysis/Configuration/test/analyze/README) before running the macros. They use relative paths for corrections, sample lists and output files: setting `HIFOREST_RESULTS` alone does not redirect output. Verify input access and output locations first.

For a channel extension, the software goal is to preserve the common jet analysis while separating lepton selection, triggers, identification, isolation and scale factors. Validate a refactored dimuon implementation against the fixed baseline before adding electron-specific selections and corrections. This is a development plan, not an implemented or validated electron channel.

## Grid authentication

Renew the CMS VOMS proxy on the host when needed, rather than automatically in the launcher:

```bash
voms-proxy-init -voms cms -valid 24:00
```

Then enter `hiforest` and verify the credential inside the container with `voms-proxy-info -timeleft`. A valid proxy created on the host was visible in the container. The local VOMS wrapper was incompatible with the container client when generating a new proxy, so renewal is performed outside it.

Do not commit certificates containing private keys, proxy files, identity output or site credentials. A proxy is not needed for compilation or analysis of local ROOT files.

## Historical generic HiForest tests

The original CMSSW_13_2_4 installation used `CmsHI:forest_CMSSW_13_2_X` and the era-specific configurations under `HeavyIonsAnalysis/Configuration/test/Run3_2023_PbPb`. Both DATA and MC smoke tests produced 20-event forests on 2026-11-24 and were inspected with PyROOT. These tests establish the technical workflow, not the validity of the Trieste analysis or its schema.

For reproducing these historical tests, activate that old release explicitly (the new launcher selects CMSSW_13_2_13). The configurations require `phoEleReg_Run3_2023_PbPb.db` in the working directory due to a relative SQLite connection. Run each configuration in a separate directory, for example:

```bash
CFG="$CMSSW_BASE/src/HeavyIonsAnalysis/Configuration/test/Run3_2023_PbPb"
ln -sfn "$CFG/phoEleReg_Run3_2023_PbPb.db" .
set -o pipefail
cmsRun -n 1 "$CFG/forest_miniAOD_run3_DATA.py" 2>&1 | tee forest_DATA.log
rootls -t HiForestMiniAOD.root
```

Use `forest_miniAOD_run3_MC.py` for the MC test. The legacy top-level configurations could fail with empty lepton model/correction paths. The initial multithreaded DATA test failed with `ModulesSynchingOnLumis` for `skimanalysis`; single-thread execution succeeded. `cmsRun -n` sets thread count, not event count. The alternative of setting `process.options.numberOfConcurrentLuminosityBlocks = cms.untracked.uint32(1)` remains untested.

Keep the old sample files under `data/test/` and the reusable inspection notebook under `validation/notebooks/` when migrating; update the notebook input paths if the files move. Do not overwrite legacy forests with new productions.

## References

- [Trieste HiForest setup README](https://github.com/cms-ts/cmssw/blob/24818506602796f751e8665cd31aca601e06f1cb/HeavyIonsAnalysis/README)
- [Trieste analysis README](https://github.com/cms-ts/cmssw/blob/24818506602796f751e8665cd31aca601e06f1cb/HeavyIonsAnalysis/Configuration/test/analyze/README)
- [CMS Git tools: sparse checkout](https://github.com/cms-sw/cms-git-tools/blob/master/git-cms-sparse-checkout)
- [Original generic HiForest branch](https://github.com/CmsHI/cmssw/tree/forest_CMSSW_13_2_X/HeavyIonsAnalysis)
- [[Computing/INFN Trieste Computing Farm|INFN Trieste computing farm]]
