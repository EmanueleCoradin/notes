---
title: "2026-11-25 — Reduced-sample Z+jet reproduction"
description: "Reproduce the nominal Trieste dimuon analysis on existing forests with fixed corrections in an independent CMSSW workspace."
tags:
  - research-log
  - cms
  - heavy-ion
  - root
  - reproducibility
date: 2026-11-25
draft: false
---

# Reduced-sample Z+jet reproduction — 2026-11-25

> [!WARNING] AI-generated content — review pending
> This log was prepared with ChatGPT from the actual terminal session and the reference source code. The author should review and validate the commands, observations and interpretation before relying on or publishing it. The commands below consolidate the session; the consolidated recipe has not been rerun from scratch.

## Objective and decisions

Reproduce Raffaele Delli Gatti's existing dimuon Z+jet analysis with minimal changes: first reproduce a plot from the committed histograms, then run the original nominal event loop on a small subset of the existing DATA and signal MC forests, and finally plot the new outputs.

Reuse all original calibration, scale-factor, reweighting, minimum-bias and MC normalization inputs. Do not rerun foresting, rederive corrections, run the systematic/background-MC wrappers, or unfold the reduced sample. The only intended modification before event processing is the input manifest. This is a technical smoke test, not a reproduction of the full physics measurement.

See [[Computing/Trieste Z+jet Analysis Workflow|Trieste Z+jet Analysis Workflow]] for the processing stages and script relationships, [[Computing/HiForest Output Format|HiForest Output Format]] for the tree interface, and [[Computing/HiForest Setup|HiForest Setup]] for the reference environment.

## 1. Baseline and isolation

- Reference commit: `24818506602796f751e8665cd31aca601e06f1cb`.
- CMSSW release: `CMSSW_13_2_13`; SCRAM architecture: `el8_amd64_gcc11`; EL8 Singularity environment.
- Personal branch: `zjet-reproduction` in the personal CMSSW fork. Start from the full original commit, **not** the code-only branch, which omits required ROOT and correction inputs.
- Separate SCRAM installation under the existing personal HeavyIons workspace.

~~~text
HeavyIons/
  reference/CMSSW_13_2_13/
  development/
  validation/
    comparisons/
    notebooks/
    scripts/
    2026-11-25_reduced-sample-fixed-corrections/
      CMSSW_13_2_13/src/
      results/
~~~

Read the existing forests remotely through XRootD. Do not upload to, rename or delete anything in the sample owner's EOS directories, submit production jobs or push to the upstream Trieste repository. The reference working tree is only used as a local Git clone source. The nominal macro writes relative output paths with ROOT's RECREATE mode, so always execute it inside the new workspace. This is sufficient isolation for the tested nominal workflow; auxiliary scripts should be audited separately before execution.

Private storage paths, full EOS URLs and event identifiers are deliberately omitted from this public-vault draft. The original tracked manifest and the locally saved reduced manifest supply the actual URLs.

## 2. Create the workspace

Start in the reference Singularity session (see the setup note; the tested launcher is invoked from the host as `hiforest reference`). If re-entering an existing reproduction workspace, skip creation and cloning. These commands assume the reference repository has the pinned commit and sparse-checkout rules. They stop on failure and do not intentionally overwrite an existing workspace.

~~~bash
export REPRO_NAME=2026-11-25_reduced-sample-fixed-corrections
export REPRO_DIR="$HIFOREST_WORK/validation/$REPRO_NAME"
export REF_SRC="$HIFOREST_WORK/reference/CMSSW_13_2_13/src"
export REV=24818506602796f751e8665cd31aca601e06f1cb
export REPRO_BRANCH=zjet-reproduction

git -C "$REF_SRC" log -1 --oneline "$REV"
git -C "$REF_SRC" status --short

(
    set -e
    mkdir "$REPRO_DIR"
    cd "$REPRO_DIR"
    mkdir results
    export SCRAM_ARCH=el8_amd64_gcc11
    scram project CMSSW CMSSW_13_2_13
)

(
    set -e
    cd "$REPRO_DIR/CMSSW_13_2_13"
    git clone --local --no-hardlinks --no-checkout "$REF_SRC" src
    cd src
    git config core.sparseCheckout true
    cp "$REF_SRC/.git/info/sparse-checkout" .git/info/sparse-checkout
    git switch -c "$REPRO_BRANCH" "$REV"
    git cms-sparse-checkout CMSSW_13_2_13 HEAD
    git read-tree -mu HEAD
    git remote remove origin
    git remote add my-cmssw git@github.com:EmanueleCoradin/cmssw.git
)
~~~

The actual session initially used HTTPS and then changed the remote to SSH with `git remote set-url my-cmssw git@github.com:EmanueleCoradin/cmssw.git`. The recipe above uses SSH directly. No Git hard links or analysis-directory symlinks are intended.

### Activate the environment

Run on every new session after initializing the compatible CMSSW environment (the existing launcher does not yet select this validation workspace directly).

~~~bash
export REPRO_DIR="$HIFOREST_WORK/validation/2026-11-25_reduced-sample-fixed-corrections"
cd "$REPRO_DIR/CMSSW_13_2_13/src"
cmsenv
export HIFOREST_WORKSPACE="$REPRO_DIR"
export HIFOREST_RESULTS="$REPRO_DIR/results"
export CFG_DIR="$CMSSW_BASE/src/HeavyIonsAnalysis/Configuration/test"
export HIFOREST_ANALYSIS="$CFG_DIR/analyze"

echo "$CMSSW_BASE"
echo "$HIFOREST_ANALYSIS"
echo "$HIFOREST_RESULTS"
git status --short --branch
git log -1 --oneline
git remote -v

cd "$HIFOREST_ANALYSIS"
du -sh .
find . -type l
~~~

Observed: the new CMSSW_BASE pointed into validation, HEAD was `248185066027` on `zjet-reproduction`, the working tree was clean, and the analysis directory occupied 91 MB and contained no symlinks. Only the personal fork was configured as a remote. No SCRAM compilation was required for the standalone ROOT macros.

Optional: publish the baseline branch to the personal fork after checking the push URL; this succeeded in the session. If the remote branch already exists, do not force-push.

~~~bash
git remote get-url --push my-cmssw
ssh -T git@github.com
git push -u my-cmssw zjet-reproduction
~~~

## 3. Inspect and plot the original results

The nominal selection used throughout the test was PbPb 2023, 0–30% centrality, Z transverse momentum above 40 GeV, no effective upper pT cut, and systematic flag 0.

~~~bash
cd "$HIFOREST_ANALYSIS/plot"
rootls -l -r output_HI_mu_data_Cen0_30_ptZ40_Inf.root
rootls -l -r output_HI_mu_MC_signal_Cen0_30_ptZ40_Inf.root

root -l -b -q 'compare_inclusive_leading.C("xZj")'
~~~

The original plotting macro reads the committed ROOT file in READ mode. For PbPb it uses `HI/Muons/h_xZj_subtracted`, `h_xZj_all_subtracted` and `h_mumu`; the latter is used for the per-Z normalization. It plots the leading and inclusive jet momentum balance distributions. The user confirmed that the plot was produced and could be viewed.

Preserve the original histogram files and the reproduced plot before processing events. The Git revision also retains the originals.

~~~bash
cd "$HIFOREST_ANALYSIS"
mkdir -p "$HIFOREST_RESULTS/reference" "$HIFOREST_RESULTS/logs" "$HIFOREST_RESULTS/reduced"
cp -n plot/output_HI_mu_data_Cen0_30_ptZ40_Inf.root \
      plot/output_HI_mu_MC_signal_Cen0_30_ptZ40_Inf.root \
      plot/compare_inclusive_vs_lead_xZj_PbPb23_data_Cen0_30_ptZ40_Inf.pdf \
      "$HIFOREST_RESULTS/reference/"
~~~

Only run this backup step when the files in plot/ are the original outputs (before their replacement). On a repeat run, retain the previous reference copies.

## 4. Establish the reduced sample

The original analysis reads `samples_root_files.txt` and matches its entries against sample-specific patterns, converting wildcards into expressions that do not cross directory separators. For 2023 DATA this selects merged ROOT files directly under a production timestamp directory; for signal MC the active pattern selects individual job outputs inside numbered subdirectories. This distinction was important: the first individual DATA file was empty and the first 20 individual DATA files contained very few entries.

The inspected signal file had consistent event-tree counts. Two of the four inspected merged DATA files had fewer HLT entries than entries in the other event trees. The other two had consistent counts. The DATA trigger branch is not explicitly read by the nominal event loop (the production applies a trigger selection), but the macro still attaches the HLT friend chain. The cause and implications of the discrepancies are not established. Use the two count-consistent merged DATA files for this smoke test; do not repair or replace the original workflow. Equal counts do not establish event-key alignment.

### Input selection

The exact three URLs are saved locally in `results/samples_reduced.txt`. To reconstruct the same selection from the original committed manifest without a new EOS crawl, use the following suffix filters. The production tags identify the selected sample entries without publishing full remote URLs.

~~~bash
cd "$HIFOREST_ANALYSIS"
# Read from the pinned revision, even if the working list was already reduced.
git show 24818506602796f751e8665cd31aca601e06f1cb:HeavyIonsAnalysis/Configuration/test/analyze/samples_root_files.txt \
    | grep -E 'HIPhysicsRawPrime0/CRAB3_Analysis_test24_ZMM_Prime0/260212_144356/000[23]\.root$' \
    > "$HIFOREST_RESULTS/samples_reduced.txt"
git show 24818506602796f751e8665cd31aca601e06f1cb:HeavyIonsAnalysis/Configuration/test/analyze/samples_root_files.txt \
    | grep -E 'CRAB3_Analysis_test24_ZMM_DYto2L2Jets/260210_215130/0000/HiForestMiniAOD_1\.root$' \
    >> "$HIFOREST_RESULTS/samples_reduced.txt"
wc -l "$HIFOREST_RESULTS/samples_reduced.txt"
cat "$HIFOREST_RESULTS/samples_reduced.txt"
~~~

Expected: two DATA URLs ending in `0002.root` and `0003.root`, one signal MC URL ending in `HiForestMiniAOD_1.root`, exactly three lines, no duplicate entries. The original full manifest is tracked in Git (approximately 39 MiB). Do not run the broad `list_samples.py` inventory again for this exercise.

Check the proxy with `voms-proxy-info -timeleft`. If necessary renew it on the host with `voms-proxy-init -voms cms -valid 24:00` and re-enter the appropriate environment. Use the storage server name required by the reference setup for `XrdSecGSISRVNAMES` (see the original README). Do not publish proxy files or their contents.

A read-only count check can be repeated for all three files:

~~~bash
python3 - "$HIFOREST_RESULTS/samples_reduced.txt" <<'PY'
import ROOT
import sys

paths = [
    "akCs2PFJetAnalyzer/t", "muonAnalyzer/MuonTree",
    "hiEvtAnalyzer/HiTree", "skimanalysis/HltTree",
    "hltanalysis/HltTree", "hiFJRhoAnalyzerFinerBins/t",
    "PbPbTracks/trackTree"
]
with open(sys.argv[1]) as manifest:
    for line in manifest:
        url = line.strip()
        if not url:
            continue
        f = ROOT.TFile.Open(url, "READ")
        if not f or f.IsZombie():
            raise RuntimeError("Cannot open input")
        counts = {}
        for path in paths:
            tree = f.Get(path)
            counts[path] = tree.GetEntries() if tree else None
        print(url.rsplit("/", 1)[-1], counts)
        n = counts["hiEvtAnalyzer/HiTree"]
        assert n is not None and all(v == n for v in counts.values())
        f.Close()
PY
~~~

Observed DATA event counts: 266 and 59 (325 total). Signal MC: 16,296. These are technical input-entry counts, not selected physics yields.

Activate the manifest only after verifying it:

~~~bash
cd "$HIFOREST_ANALYSIS"
test "$(wc -l < "$HIFOREST_RESULTS/samples_reduced.txt")" -eq 3 &&
    cp "$HIFOREST_RESULTS/samples_reduced.txt" samples_root_files.txt
git diff --stat
~~~

Do not commit the private URL manifest, generated ROOT files or raw logs.

## 5. Run the original nominal analysis

The argument order is `collision_type, sample_name, weight_phase, systFlag, cent_min, cent_max, ptZ_min, ptZ_max`. The values 1 (DATA) and 3 (signal MC) select the existing final workflows, not a full reweighting derivation.

~~~bash
cd "$HIFOREST_ANALYSIS"
mkdir -p "$HIFOREST_RESULTS/logs" "$HIFOREST_RESULTS/reduced"
set -o pipefail

time root -l -b -q \
    'analyze_HI_TTreeReader_ZMM.C("PbPb23","data",1,0,0,30,40,9999)' \
    2>&1 | tee "$HIFOREST_RESULTS/logs/data.log"

cp plot/output_HI_mu_data_Cen0_30_ptZ40_Inf.root "$HIFOREST_RESULTS/reduced/"
~~~

Inspect the log for the expected file count, successful initialization, completion of the event loop and analysis summary before moving to MC. The ROOT process exit code alone is not proof of a complete analysis. If rerunning, use a separate log filename or preserve previous logs first.

~~~bash
time root -l -b -q \
    'analyze_HI_TTreeReader_ZMM.C("PbPb23","signal",3,0,0,30,40,9999)' \
    2>&1 | tee "$HIFOREST_RESULTS/logs/signal.log"

cp plot/output_HI_mu_MC_signal_Cen0_30_ptZ40_Inf.root "$HIFOREST_RESULTS/reduced/"
~~~

### Existing inputs used

Both runs loaded the original muon scale-factor JSONs, JEC and JES uncertainties, veto map and the corresponding minimum-bias library. MC additionally loaded the rho and vz reweighting histograms, MC normalization and JER parameters. In particular:

- `MixEvSub/MinBias_inclusive_jets_data_VZ_Cen_Combined_Cen0_30.root` and the corresponding MC file.
- `weights_MC/rho_weights_1/weight_HI_rho_Cen0_30_ptZ40_Inf.root`.
- `weights_MC/vz_weights_2/weight_HI_vz_Cen0_30_ptZ40_Inf.root`.
- `weights_MC/MC_all_weights/output_HI_mu_MC_all.root`.
- `Prompt23HIPbPb_V1_MC_L2Relative_AK2PF.txt` and DATA residual `Prompt23HIPbPb_V1_DATA_L2Residual_AK2PF.txt`.
- `derive_JER_AK2/My_PbPb23_MC_PtResolution_AK2PF.txt` (MC); see the log for all other JER inputs.

Do not invoke `do_analyze_all.sh` or `do_analyze_weights.sh` in this reduced-sample exercise: they run additional stages and may regenerate weights or overwrite other local outputs.

### Observed technical results

| | DATA | Signal MC |
| --- | ---: | ---: |
| Files matched | 2 | 1 |
| Events processed | 325 | 16,296 |
| Event loop completion | 100% | 100% |
| Wall time | 14.843 s | 17.324 s |
| User CPU time | 9.084 s | 9.392 s |
| System CPU time | 1.674 s | 1.599 s |
| Minimum-bias entries cached | 9,000 | 9,000 |
| Centrality–vz bins cached | 90 | 90 |

Both executions completed and produced the nominal ROOT output files. Times are single shell measurements on the farm, including ROOT initialization and remote reads; they are not throughput benchmarks or an extrapolation to the full dataset. No event-level identifiers, selected physics yields or plots are included in this public draft; the local logs contain the full diagnostics.

## 6. Plot the reduced results

Use the same unchanged plotting macro on the newly generated DATA and MC histograms.

~~~bash
cd "$HIFOREST_ANALYSIS/plot"
root -l -b -q 'compare_inclusive_leading.C("xZj","PbPb23",true)'
root -l -b -q 'compare_inclusive_leading.C("xZj","PbPb23",false)'

cp compare_inclusive_vs_lead_xZj_PbPb23_*_Cen0_30_ptZ40_Inf.pdf \
   "$HIFOREST_RESULTS/reduced/"
~~~

Both plots were reported produced. Their quantitative agreement with the reference has not yet been evaluated. The first reference DATA plot and the two original histogram files were to be preserved in results/reference; the reduced outputs are stored separately.

~~~text
results/
  samples_reduced.txt
  reference/
    output_HI_mu_data_Cen0_30_ptZ40_Inf.root
    output_HI_mu_MC_signal_Cen0_30_ptZ40_Inf.root
    compare_inclusive_vs_lead_xZj_PbPb23_data_Cen0_30_ptZ40_Inf.pdf
  reduced/
    output_HI_mu_data_Cen0_30_ptZ40_Inf.root
    output_HI_mu_MC_signal_Cen0_30_ptZ40_Inf.root
    compare_inclusive_vs_lead_xZj_PbPb23_data_Cen0_30_ptZ40_Inf.pdf
    compare_inclusive_vs_lead_xZj_PbPb23_MC_signal_Cen0_30_ptZ40_Inf.pdf
  logs/
    data.log
    signal.log
~~~

## 7. Limitations and next study

- Technical reproduction only. The sample is tiny and not selected to be statistically representative.
- MC normalization refers to the original full generated sample and total weight, not the one file. Direct absolute-yield comparisons with DATA are not valid for this test.
- The original closure train/test split uses the event counter and full Ngen, not the reduced event count. The test portion will not be properly populated. Do not interpret closure results from this run.
- The progress interval is computed as total_events/100; a nonempty sample of fewer than 100 entries can cause modulo by zero. The tested samples avoid this issue without code changes.
- Matching tree counts do not establish event-key alignment; the merged DATA HLT discrepancy remains an open question.
- No systematic variations, electroweak/top background MC processing, full weight derivation or unfolding was performed.
- Next: study the original macro stage by stage, inspect the original and reduced histogram contents, understand the Z selection, jet selection, mixed-event background and response matrices before increasing statistics or modifying code.

The dedicated workspace and branch preserve a reproducible baseline for the future dielectron extension. Use explicit output snapshots in results/ when repeating the test. Keep changes to the private manifest and generated files local; review Git diffs before any push.

## References

- [Original analysis README](https://github.com/cms-ts/cmssw/blob/24818506602796f751e8665cd31aca601e06f1cb/HeavyIonsAnalysis/Configuration/test/analyze/README)
- [Main event-level ROOT macro](https://github.com/cms-ts/cmssw/blob/24818506602796f751e8665cd31aca601e06f1cb/HeavyIonsAnalysis/Configuration/test/analyze/analyze_HI_TTreeReader_ZMM.C)
- [MC sample definitions](https://github.com/cms-ts/cmssw/blob/24818506602796f751e8665cd31aca601e06f1cb/HeavyIonsAnalysis/Configuration/test/analyze/MC_samples.h)
- [Plotting README](https://github.com/cms-ts/cmssw/blob/24818506602796f751e8665cd31aca601e06f1cb/HeavyIonsAnalysis/Configuration/test/analyze/plot/README)
- [Inclusive/leading plotting macro](https://github.com/cms-ts/cmssw/blob/24818506602796f751e8665cd31aca601e06f1cb/HeavyIonsAnalysis/Configuration/test/analyze/plot/compare_inclusive_leading.C)
- Terminal session on 2026-11-25, including observed setup, DATA and signal MC logs and confirmation of generated plots (local records).
