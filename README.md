# MIT-BIH Malignant Ventricular Ectopy Database (VFDB)

[![License: ODC-By 1.0](https://img.shields.io/badge/License-ODC--By%201.0-green)](https://opendatacommons.org/licenses/by/1-0/)
[![Access: public](https://img.shields.io/badge/access-public-0e8a16.svg)](https://physionet.org/content/vfdb/1.0.0/)

**MIT-BIH Malignant Ventricular Ectopy Database (VFDB)** — two-channel ECGs with sustained VT / VFL / VF episodes and rhythm-change annotations; PhysioNet open access.

- **Dataset repository**: https://github.com/biometric-community/MIT-BIH-Malignant-Ventricular-Ectopy-Database
- **Upstream source**: https://physionet.org/content/vfdb/1.0.0/
- **DOI**: https://doi.org/10.13026/C22P44
- **Original format**: PhysioNet WFDB (`.dat` / `.hea` / `.atr`) under `data/`
- **License (data)**: [Open Data Commons Attribution License v1.0](https://opendatacommons.org/licenses/by/1-0/) (PhysioNet)
- **License (helpers / docs)**: CC BY 4.0 (see [`LICENSE`](LICENSE))

| Field | Value |
|-------|-------|
| Catalog id (tbiom) | `vfdb` |
| Category | `physio` |
| Access | `public` |
| PhysioNet / WFDB slug | `vfdb` |
| Upstream homepage | https://physionet.org/content/vfdb/1.0.0/ |
| Paper alias | VFDB (e.g. Abdeldayem & Bourlai, TBIOM 2019 spectral-correlation ECG ID) |

## TL;DR

- **Task**: malignant ventricular arrhythmia / VF–VT detection and rhythm analysis
- **Modality**: two-channel ECG (WFDB `.dat` / `.hea` / `.atr`)
- **Platform**: MIT-BIH malignant ventricular ectopy recordings
- **Real/Synthetic**: real
- **Subjects / records**: **22** half-hour-scale excerpts (one record per subject in this release)
- **Sampling**: 250 Hz, format 212, ~525 000 samples per record (~35 min)
- **Annotations**: rhythm-change labels only (no beat labels) in `.atr`
- **Size**: ~33.1 MiB uncompressed (PhysioNet); ~33 MiB under `data/` locally
- **Citation**: Greenwald 1986 M.S. thesis (+ PhysioNet citation)

## Table of contents

- [Download](#download)
- [Dataset structure](#dataset-structure)
- [Annotation schema](#annotation-schema)
- [Stats and splits](#stats-and-splits)
- [Quick start](#quick-start)
- [Evaluation and baselines](#evaluation-and-baselines)
- [Datasheet (data card)](#datasheet-data-card)
- [Known issues and caveats](#known-issues-and-caveats)
- [License](#license)
- [Citation](#citation)
- [Contact](#contact)

## Download

- **This repository**: WFDB records under [`data/`](data/) (22 × `.hea` / `.dat` / `.atr`).
- **Upstream**: https://physionet.org/content/vfdb/1.0.0/ (ZIP ≈ 33.1 MiB uncompressed).
- **Helper script** (from tbiom monorepo root):

```bash
bash projects/datasets/scripts/download_vfdb.sh
```

Preferred (after `pip install wfdb`):

```bash
python -c "import wfdb; wfdb.dl_database('vfdb', r'projects/datasets/vfdb/data')"
```

Or with wget:

```bash
wget -r -N -c -np https://physionet.org/files/vfdb/1.0.0/ -P projects/datasets/vfdb/data
```

## Dataset structure

```text
vfdb/
├── README.md
├── LICENSE
└── data/
    ├── 418.hea / 418.dat / 418.atr
    ├── 419.hea / 419.dat / 419.atr
    ├── ...
    ├── 430.hea / 430.dat / 430.atr
    ├── 602.hea / 602.dat / 602.atr
    ├── 605.hea / 605.dat / 605.atr
    ├── ...
    └── 615.hea / 615.dat / 615.atr
```

**Record IDs** (22): `418`–`430`, `602`, `605`, `607`, `609`–`612`, `614`, `615`.

- **Splits**: no official ML train/val/test partition.
- **Layout notes**: flat WFDB naming — each record `NNN` has header, signal, and annotation files.

## Annotation schema

### WFDB records (`data/*.hea`, `.dat`, `.atr`)

- **`.hea`**: channels (2), sampling rate (250 Hz), sample count, ADC/gain fields
- **`.dat`**: binary two-channel ECG samples (format 212 in headers checked locally)
- **`.atr`**: rhythm-change annotations only (no beat labels)
- **Time base**: sample index at 250 Hz
- **Example** (record `418` header):

```text
418 2 250 525000
418.dat 212 200 12 0 -128 1830 0 ECG
418.dat 212 200 12 0 -14 -5967 0 ECG
```

Use [WFDB](https://physionet.org/content/wfdb/) / `wfdb` (Python) to read signals and annotations.

### Rhythm labels (`.atr` aux strings)

Upstream interpretation of rhythm-change aux fields:

| Aux | Meaning |
|-----|---------|
| `AFIB` | atrial fibrillation |
| `ASYS` | asystole |
| `B` | ventricular bigeminy |
| `BI` | first-degree heart block |
| `HGEA` | high-grade ventricular ectopic activity |
| `N` / `NSR` | normal sinus rhythm |
| `NOD` | nodal (AV junctional) rhythm |
| `NOISE` | noise (previous rhythm continues until next annotation) |
| `PM` | paced rhythm |
| `SBR` | sinus bradycardia |
| `SVTA` | supraventricular tachyarrhythmia |
| `VER` | ventricular escape rhythm |
| `VF` / `VFIB` | ventricular fibrillation |
| `VFL` | ventricular flutter |
| `VT` | ventricular tachycardia |

Rhythm-change markers are placed at the **start** of each episode.

## Stats and splits

Counts verified from local `data/`:

| Measure | Count |
|---------|------:|
| Records (`.hea` / `.dat` / `.atr`) | 22 |
| Channels per record | 2 |
| Sample rate | 250 Hz |
| Samples per record | 525 000 |
| Approx. duration per record | ~35 min |
| Local `data/` size | ~33 MiB |

No official subject-disjoint train/test split — define folds carefully if using for identity or episode-level ML.

## Quick start

```bash
cd projects/datasets/vfdb
pip install wfdb
```

```python
from pathlib import Path
import wfdb

root = Path("data")
records = sorted(p.stem for p in root.glob("*.hea"))
print(len(records), "records:", records[:5], "...")

rec = root / "418"
sig, fields = wfdb.rdsamp(str(rec))
ann = wfdb.rdann(str(rec), "atr")
print(sig.shape, fields["fs"], "Hz;", len(ann.sample), "rhythm annotations")
print("aux sample:", [a.strip() for a in ann.aux_note[:8]])
```

**Dependencies**: optional `wfdb` for loading; bash + network for re-download helpers.

## Evaluation and baselines

- **Primary metrics**: VF / VT / VFL episode detection sensitivity and specificity; arrhythmia-detector confidence limits (see Greenwald et al., CinC 1985)
- **Suggested baselines**: classical VF detectors evaluated on this corpus; modern ECG deep-learning papers citing VFDB
- **Baseline numbers**: not reproduced here — see citing literature

## Datasheet (data card)

### Motivation

Provide annotated ECG recordings that include sustained malignant ventricular arrhythmias (VT, ventricular flutter, VF) for detector development and evaluation.

### Composition

22 two-channel ECG records from subjects who experienced sustained VT / VFL / VF. Annotations mark rhythm changes (and occasional noise / asystole), not individual beats.

### Collection process

Prepared as part of MIT-BIH work on ventricular fibrillation detection (Greenwald, 1986); distributed via PhysioNet as `vfdb` 1.0.0.

### Preprocessing

Distributed in PhysioNet WFDB format at 250 Hz (format 212 in local headers).

### Distribution

- **Signal / annotation files**: ODC-By 1.0 via PhysioNet
- **Helpers / docs in this folder**: CC BY 4.0 (`LICENSE`)
- **GitHub mirror**: https://github.com/biometric-community/MIT-BIH-Malignant-Ventricular-Ectopy-Database

### Maintenance

Re-download with `wfdb.dl_database('vfdb', ...)` or `download_vfdb.sh` if needed. Keep record IDs and WFDB triples intact.

## Known issues and caveats

- Record IDs are **not contiguous** in the 600s range (no `603`, `604`, `606`, `608`, `613` in this release)
- Annotations are **rhythm-only** — do not expect beat-level labels as in MIT-BIH Arrhythmia (`mitdb`)
- Upstream describes “half-hour” recordings; local headers use **525 000** samples at 250 Hz (~35 min)
- No official ML split — avoid leakage when segmenting episodes from the same record
- Catalog id `vfdb` is the PhysioNet / WFDB database name

## License

**Data files** are licensed under **[ODC-By 1.0](https://opendatacommons.org/licenses/by/1-0/)** as published by PhysioNet.

**Packaging helpers / docs** in this folder are **[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/)**. See [`LICENSE`](LICENSE).

## Citation

When using this resource, cite the original publication and PhysioNet:

```bibtex
@mastersthesis{Greenwald1986VF,
  title  = {Development and analysis of a ventricular fibrillation detector},
  author = {Greenwald, Scott D.},
  school = {MIT Dept. of Electrical Engineering and Computer Science},
  year   = {1986}
}

@misc{vfdb100,
  title        = {{MIT-BIH} Malignant Ventricular Ectopy Database},
  author       = {{MIT-BIH} and {PhysioNet}},
  howpublished = {PhysioNet},
  year         = {1999},
  note         = {Version 1.0.0},
  doi          = {10.13026/C22P44},
  url          = {https://physionet.org/content/vfdb/1.0.0/}
}
```

Related methods paper often cited with this corpus:

```bibtex
@inproceedings{Greenwald1985CinC,
  title     = {Estimating confidence limits for arrhythmia detector performance},
  author    = {Greenwald, S. D. and Albrecht, P. and Moody, G. B. and Mark, R. G.},
  booktitle = {Computers in Cardiology},
  volume    = {12},
  pages     = {383--386},
  year      = {1985}
}
```

Also include the current PhysioNet platform citation required on the project page.

## Contact

- **Upstream**: https://physionet.org/content/vfdb/1.0.0/
- **Dataset repository**: https://github.com/biometric-community/MIT-BIH-Malignant-Ventricular-Ectopy-Database
- **tbiom catalog**: `projects/datasets/vfdb/`
