# Drone Audio Perceptual Quality

This repository contains the full pipeline for a study on perceived annoyance of drone audio across different environments, heights, and flight paths. The pipeline is split into two tracks — **subjective** (listening test analysis) and **objective** (computational audio metrics) — which are combined in the final analysis notebook.

---

## Uncut Recordings

The original uncut field and garden recordings are too large to host on GitHub and are available for download from Google Drive:

**[Download uncut_recordings/ from Google Drive](https://drive.google.com/drive/folders/1EtsDp_Bm6DJP6lyP01zH0368OVxMY77A?usp=sharing)**

The folder contains two subfolders — `field/` and `garden/` — which are not used by any notebook and are provided for reference only, but contain the raw recordings from which the trimmed clips were taken from using Audacity.

---

## Repository Structure

```
project/
├── code/                   ← all notebooks live here
├── data/
│   ├── field/              ← original long field recordings, used for Audacity trimming only
│   ├── garden/             ← original long garden recordings, used for Audacity trimming only
│   ├── trimmed/
│   │   ├── raw/            ← original trimmed .wav files (input, do not delete)
│   │   ├── lufs_normalised/← created by 0011
│   │   ├── peak_normalised/← created by 0010
│   │   ├── split/          ← created by 0012
│   │   ├── lufs_normalised.zip ← created by 0011 (for Colab upload)
│   │   ├── peak_normalised.zip ← created by 0010
│   │   └── split.zip       ← created by 0012 (for Colab upload)
│   ├── refs/
│   │   ├── field_ref.wav   ← reference audio (input, do not delete)
│   │   └── garden_ref.wav  ← reference audio (input, do not delete)
│   └── results/
│       ├── subj_study_results.csv      ← listening test export (input, do not delete)
│       ├── scoreq_outputs_lufs.jsonl   ← created by 0020
│       ├── scoreq_outputs_split.jsonl  ← created by 0020
│       ├── versa_outputs_lufs.jsonl    ← downloaded from Colab (00210), place here manually
│       └── versa_outputs_split.jsonl   ← downloaded from Colab (00211), place here manually
├── graphs/
│   ├── wave/               ← created by 0012
│   ├── no5m/               ← created by 00300 (when EXCLUDE_5M = True)
│   └── full/               ← created by 00300 (when EXCLUDE_5M = False)
├── images/                 ← setup photos and project flowchart
└── misc/
    ├── audio_requirements.txt  ← pip requirements for audio_env
    └── scoreq_requirements.txt ← pip requirements for scoreq_env
```

---

## Environments Required

Two separate Conda environments are used due to dependency conflicts between SCOREQ and the other libraries:

- **`audio_env`** — used for notebooks `0010`, `0011`, `0012`, `00300`
- **`scoreq_env`** — used for notebook `0020` (SCOREQ)
- **Google Colab** — used for notebooks `00210` and `00211` (VERSA), requires GPU runtime

Pip requirements for both environments are provided in `misc/audio_requirements.txt` and `misc/scoreq_requirements.txt`.

---

## Notebooks

### `0010_peak_normalisation` — kernel: `audio_env`

Peak-normalises each `.wav` file in `data/trimmed/raw/` to the range [-1, 1] and saves the results to `data/trimmed/peak_normalised/`. Also plots the waveform of each file and displays audio widgets for listening. Finally, zips the output folder to `data/trimmed/peak_normalised.zip`.

> **Note:** This notebook is for inspection only. The LUFS-normalised files (`0011`) are what the rest of the pipeline uses.

**Reads:** `data/trimmed/raw/`  
**Writes:** `data/trimmed/peak_normalised/`, `data/trimmed/peak_normalised.zip`

---

### `0011_lufs_normalisation` — kernel: `audio_env`

Normalises each `.wav` file in `data/trimmed/raw/` to -24 LUFS using `pyloudnorm`, which is the standard loudness level used throughout the pipeline. Saves results to `data/trimmed/lufs_normalised/`. Also displays a file inventory and audio widgets. Finally, zips the output folder to `data/trimmed/lufs_normalised.zip`.

**Reads:** `data/trimmed/raw/`  
**Writes:** `data/trimmed/lufs_normalised/`, `data/trimmed/lufs_normalised.zip`

---

### `0012_objective_audio_split` — kernel: `audio_env`

Several flyby recordings contain two distinct drone passes in a single file. This notebook uses hardcoded sample indices to segment those files into two separate clips each. It plots the waveform of every flyby file with the split point marked in red for visual verification. Saves split files to `data/trimmed/split/` and zips to `data/trimmed/split.zip`.

**Reads:** `data/trimmed/lufs_normalised/`  
**Writes:** `data/trimmed/split/`, `data/trimmed/split.zip`, `graphs/wave/segmented_audio.png`

---

### `0020_scoreq_metrics` — kernel: `scoreq_env`

Runs the SCOREQ objective audio quality model on all stimuli in two modes:

- **NR (no-reference):** scores each file independently
- **NMR (non-matching reference):** scores each file against its corresponding environment background reference (`field_ref.wav` or `garden_ref.wav`)

Runs on both the full LUFS-normalised files and the split files separately. Results are written as JSONL files.

**Reads:** `data/trimmed/lufs_normalised/`, `data/trimmed/split/`, `data/refs/`  
**Writes:** `data/results/scoreq_outputs_lufs.jsonl`, `data/results/scoreq_outputs_split.jsonl`

---

### `00210_versa_metrics_lufs` — Google Colab (GPU required)

Runs the VERSA toolkit on the full LUFS-normalised audio files, computing DNSMOS, UTMOS, SRMR, and NISQA scores. Must be run on Google Colab with a GPU runtime enabled.

**Steps to run:**
1. Open in Google Colab and set runtime to GPU (T4 is sufficient)
2. Run all cells in order
3. When prompted, upload `data/trimmed/lufs_normalised.zip` from this repository
4. The output file `versa_outputs_lufs.jsonl` will download automatically when complete
5. Move the downloaded file into `data/results/` in this repository

**Writes (after manual move):** `data/results/versa_outputs_lufs.jsonl`

---

### `00211_versa_metrics_split` — Google Colab (GPU required)

Identical to `00210` but runs on the split audio files instead. Must also be run on Google Colab with a GPU runtime.

**Steps to run:**
1. Open in Google Colab and set runtime to GPU (T4 is sufficient)
2. Run all cells in order
3. When prompted, upload `data/trimmed/split.zip` from this repository
4. The output file `versa_outputs_split.jsonl` will download automatically when complete
5. Move the downloaded file into `data/results/` in this repository

**Writes (after manual move):** `data/results/versa_outputs_split.jsonl`

---

### `00300_data_analysis` — kernel: `audio_env`

The final analysis notebook. Loads all objective model outputs and the subjective listening test results, then performs:

- One-way ANOVA and Tukey HSD post-hoc tests for environment, height, and flight path
- Repeated measures ANOVA for interaction effects
- Spearman correlation between each objective metric and mean subjective MOS
- Scatter plots of each metric vs MOS, faceted by environment
- A summary bar chart of all model correlations

There is a flag at the top of the notebook:

```python
EXCLUDE_5M = True
```

Set this to `True` to exclude 5m recordings from the analysis (recommended — 5m recordings were excluded from the listening test due to recording inconsistencies). Set to `False` to include all heights. Graphs are saved to `graphs/no5m/` or `graphs/full/` accordingly.

> To reproduce all graphs used in the report, the notebook should be run **twice** — once with `EXCLUDE_5M = True` and once with `EXCLUDE_5M = False`.

> Faceted scatter plots (objective metric vs MOS, broken down by environment) are produced for all 11 objective models. Only 3 of these are discussed in the report; the rest are included for completeness.

**Reads:** `data/results/scoreq_outputs_lufs.jsonl`, `data/results/scoreq_outputs_split.jsonl`, `data/results/versa_outputs_lufs.jsonl`, `data/results/versa_outputs_split.jsonl`, `data/results/subj_study_results.csv`  
**Writes:** `graphs/no5m/` or `graphs/full/` (plots and CSV tables)

---

## Full Run Order

```
0010  →  optional, inspection only
0011  →  required before 0012 and 0020
0012  →  required before 0020
0020  →  required before 00300
00210 →  required before 00300  (Colab, move output to data/results/ manually)
00211 →  required before 00300  (Colab, move output to data/results/ manually)
00300 →  final analysis, run last
```

All folders are created automatically by the notebooks where needed. The only manual step in the pipeline is moving the two VERSA output files from your Downloads folder into `data/results/` after running the Colab notebooks.
