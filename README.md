# MIT-BIH Malignant Ventricular Ectopy Database (VFDB)

| Field | Value |
|-------|-------|
| Id | `vfdb` |
| Category | `physio` |
| Access | `public` |
| Homepage | https://physionet.org/content/vfdb/1.0.0/ |
| PhysioNet slug | `vfdb` |
| WFDB name | `vfdb` |
| Paper alias | VFDB (Abdeldayem & Bourlai, TBIOM 2019 spectral-correlation ECG ID) |

Half-hour ECG with VT/VFL/VF episodes @ 250 Hz. **22** subjects.

## Download

```bash
# from repo root (Git Bash / WSL / Linux / macOS)
bash projects/datasets/scripts/download_vfdb.sh
```

Or:

```bash
cd projects/datasets/scripts
./download_vfdb.sh
```

Preferred (after `pip install wfdb`):

```bash
python -c "import wfdb; wfdb.dl_database('vfdb', r'projects/datasets/vfdb')"
```

Place PhysioNet files under `projects/datasets/vfdb/`.
