---
title: "HiForest Setup"
description: "Reproducible CMSSW environment and interactive launcher for 2023 PbPb HiForest production."
tags:
  - computing
  - cms
  - heavy-ion
  - cmssw
date: 2026-11-24
draft: false
---

# HiForest Setup

HiForest converts [[Computing/CMS MiniAOD|CMS MiniAOD]] into ROOT analysis trees using [[Computing/CMSSW|CMSSW]] producers and analyzers. This setup was tested on the [[Computing/INFN Trieste Computing Farm|INFN Trieste computing farm]] with the 2023 PbPb HiForest branch, an EL9 host and an EL8 CMS container.

## One-time installation

On the host, create a persistent workspace and a shortcut from the home directory:

```bash
WORK="/gfsvol01/cms/users/$USER/work/HeavyIons"
mkdir -p "$WORK"/{tests/DATA,tests/MC,production,ntuples,analysis}
ln -sT "$WORK" "$HOME/HeavyIons"
cd "$HOME/HeavyIons"
```

The symlink requires that the destination name does not already exist. The files remain on GFS, not in the home filesystem. Adjust the storage path and container bind mount for other sites.

Enter the compatible container:

```bash
/cvmfs/cms.cern.ch/common/cmssw-el8 -B /gfsvol01
```

Inside the container, install and build the release once:

```bash
source /cvmfs/cms.cern.ch/cmsset_default.sh
export SCRAM_ARCH=el8_amd64_gcc11
cd "$HOME/HeavyIons"
cmsrel CMSSW_13_2_4
cd CMSSW_13_2_4/src
cmsenv
git cms-init
git cms-merge-topic -u CmsHI:forest_CMSSW_13_2_X
scram b -j 4
git rev-parse HEAD
```

Choose the build parallelism according to local resource policies. The compiled area is persistent; subsequent sessions need only activate its runtime. Record the actual Git revision locally for reproducibility, as the upstream branch can change.

## Interactive environment

Create `~/.hiforestrc` on the host:

```bash
source ~/.bashrc

export HIFOREST_WORK="/gfsvol01/cms/users/$USER/work/HeavyIons"
export SCRAM_ARCH=el8_amd64_gcc11

source /cvmfs/cms.cern.ch/cmsset_default.sh

cd "$HIFOREST_WORK/CMSSW_13_2_4/src" || exit 1
eval "$(scram runtime -sh)"

export CFG_DIR="$CMSSW_BASE/src/HeavyIonsAnalysis/Configuration/test/Run3_2023_PbPb"

cd "$HIFOREST_WORK" || exit 1

echo "HiForest environment ready"
echo "CMSSW: $CMSSW_VERSION"
echo "Working directory: $HIFOREST_WORK"

PS1='[HiForest \W]\$ '
```

Append the launcher to the host's `~/.bashrc`:

```bash
hiforest() {
    /cvmfs/cms.cern.ch/common/cmssw-el8 \
        -B /gfsvol01 \
        --command-to-run 'bash --rcfile "$HOME/.hiforestrc" -i'
}
```

Reload the shell configuration:

```bash
source ~/.bashrc
hiforest
```

The launcher enters the EL8 container, activates the existing CMSSW environment, sets `CFG_DIR` and starts in the HeavyIons workspace. The setup was tested successfully on 2026-11-24. The container may display its default Singularity prompt rather than the custom prompt.

Verify the active environment with:

```bash
echo "$CMSSW_VERSION"
echo "$CMSSW_BASE"
echo "$CFG_DIR"
pwd
```

Use `exit` to return to the host. No additional navigation aliases are required.

## Grid authentication

Renew the CMS VOMS proxy on the host when needed, rather than automatically in the launcher:

```bash
voms-proxy-init -voms cms -valid 24:00
```

Then enter `hiforest` and verify the credential inside the container with `voms-proxy-info -timeleft`. A valid proxy created on the host was visible in the container. The local VOMS wrapper was incompatible with the container client when generating a new proxy, so renewal is performed outside it.

Do not commit certificates containing private keys, proxy files, identity output or site credentials. A proxy is not needed for compilation or analysis of local ROOT files.

## First DATA and MC tests

Use the era-specific configurations under `Configuration/test/Run3_2023_PbPb`. The older top-level examples omit required lepton model and correction paths and can fail with `FileInPathError: Relative path must not be empty`.

Inside the initialized container:

```bash
cd "$HIFOREST_WORK/tests/DATA"
ln -sfn "$CFG_DIR/phoEleReg_Run3_2023_PbPb.db" .
set -o pipefail
cmsRun -n 1 "$CFG_DIR/forest_miniAOD_run3_DATA.py" 2>&1 | tee forest_DATA.log
rootls -t HiForestMiniAOD.root
```

The SQLite database is required in the job's working directory because the regression configuration uses a relative connection path.

The DATA test processed 20 events and produced `HiForestMiniAOD.root` on 2026-11-24. Remote MiniAOD access was successful. The output structure was subsequently inspected with PyROOT; see [[Computing/HiForest Output Format|HiForest Output Format]].

For MC, use a separate working directory:

```bash
cd "$HIFOREST_WORK/tests/MC"
ln -sfn "$CFG_DIR/phoEleReg_Run3_2023_PbPb.db" .
set -o pipefail
cmsRun -n 1 "$CFG_DIR/forest_miniAOD_run3_MC.py" 2>&1 | tee forest_MC.log
```

The MC test also produced `HiForestMiniAODMC.root` with 20 entries in the event trees on 2026-11-24. Both example files were inspected with PyROOT; this establishes the technical workflow, not physics validation.

### Concurrency

`cmsRun -n` sets the thread count, not the event limit. The reference configurations already request 20 events through `process.maxEvents.input`.

The multithreaded DATA test failed with `ModulesSynchingOnLumis` because `FilterAnalyzer` (`skimanalysis`) requires synchronization at luminosity block boundaries. Single-thread execution succeeded. For future multithreaded runs, the framework suggests setting `process.options.numberOfConcurrentLuminosityBlocks = cms.untracked.uint32(1)` in a separate configuration; that alternative has not yet been tested.

## Reusing the environment

- Reuse the same compiled CMSSW area for different Python configurations and input samples.
- Run from separate working directories to keep outputs and logs independent.
- Rebuild incrementally only when compiled code changes; do not clean the build unnecessarily.
- Verify shared filesystem visibility and runtime compatibility before reusing the area on batch worker nodes.

## References

- [HiForest source branch](https://github.com/CmsHI/cmssw/tree/forest_CMSSW_13_2_X/HeavyIonsAnalysis)
- [2023 PbPb DATA and MC configurations](https://github.com/CmsHI/cmssw/tree/forest_CMSSW_13_2_X/HeavyIonsAnalysis/Configuration/test/Run3_2023_PbPb)
