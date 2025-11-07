# PsyTools — Psychoacoustic Toolbox (Full Documentation)

*A comprehensive Python toolbox for psychoacoustic analysis based on standardized hearing models (Zwicker, Aures, Sottek, ECMA‑418).*

---

## 🧭 Overview

**PsyTools** provides a unified Python framework for psychoacoustic metrics following international standards and psychoacoustic models.  
It allows researchers and engineers to compute **loudness, sharpness, fluctuation strength, roughness, tonality, and prominence ratio** for both stationary and time‑varying signals.

The toolbox is fully modular — each metric can be used independently or combined in the automated analysis pipeline.

---

## 📦 Installation

```bash
pip install psytools
```

or for development:

```bash
git clone https://github.com/<yourusername>/psytools.git
cd psytools
pip install -e .
```

---

## ⚙️ Requirements

- Python ≥ 3.8  
- numpy, scipy, matplotlib  
- librosa (optional, for .mp3 decoding)

---

## 🎧 Input / Output Formats

### Inputs

- **Audio signals:** `.wav`, `.mp3`, or NumPy array with sampling rate `fs`  
- **1/3‑octave spectra:** 28 bands (25 Hz–12.5 kHz), dB SPL (Z‑weighted)

### ISO 532‑1 conventions

| Mode | Description | Time step |
|------|--------------|------------|
| Stationary | Single 1/3‑octave SPL spectrum | — |
| Time‑varying | Spectra or audio frames | 2 ms (standard) |

### Outputs

All results are automatically saved in `/output/` as CSV and plots (PNG / PDF).  
Example directory structure:

```
output/
└─ train_station/
   ├─ run_20251106_134015.txt
   ├─ csv/
   │   ├─ loudness_zwicker.csv
   │   ├─ tonality_sottek.csv
   │   └─ fluctuation_strength.csv
   └─ plots/
       ├─ pdf/
       └─ png/
```

---

## 🚀 Quick Start

```python
from scipy.io import wavfile
import psytools as pt

fs, signal = wavfile.read("example.wav")
results = pt.psyacoustic_analysis(
    input_path="example.wav",
    input_mode="wave",
    time_varying=True,
    show_plots=True,
)
print(results["loudness_stationary"])
```

---

## 🧩 Module Reference

### 1️⃣ `analysis.py`
Central entry point for automated psychoacoustic analysis.

**Function:** `psyacoustic_analysis()`

#### Parameters
| Name | Type | Description |
|------|------|--------------|
| `input_path` | str | Path to input audio/spectrum file |
| `input_mode` | str | `'wave'`, `'spectrum'`, `'audio'`, or `'AVL'` |
| `sound_field` | str | `'free'` or `'diffuse'` |
| `time_varying` | bool | Enables time-varying mode |
| `time_resolution` | str | `'standard'` (2 ms) or `'high'` (0.5 ms) |
| `show_plots` | bool | Display figures |
| `save_csv` | bool | Save CSV automatically |
| `params` | list[str] | Select metrics to compute |
| `output_base` | str | Output directory |

#### Returns
Dictionary with computed psychoacoustic metrics for each enabled module.

---

### 2️⃣ `loudness.py`
Implements **Zwicker loudness (ISO 532‑1)** for stationary and time‑varying cases.

**Main function:** `loudness_zwicker(signal, fs, time_varying=False, ...)`

#### Returns
| Key | Description |
|-----|--------------|
| `total_loudness` | Total loudness (sone) |
| `specific_loudness` | Loudness distribution vs. Bark (240 bins) |
| `percentiles` | Loudness percentiles (Nmax, N5) |

#### References
- ISO 532‑1:2017 — *Methods for calculating loudness – Part 1: Zwicker method*  
- E. Zwicker & H. Fastl, *Psychoacoustics: Facts and Models*, 2017

---

### 3️⃣ `sharpness.py`
Implements **Sharpness (acum)** using **DIN 45692**, **Aures**, or **von Bismarck** models.

**Function:** `acoustic_sharpness()`

#### Returns
Sharpness value (scalar or time‑series) in acum.

#### References
- DIN 45692:2009 — *Determination of perceived sound quality – Sharpness*  
- Fastl & Zwicker (2007)

---

### 4️⃣ `fluctuation.py` and `roughness.py`
Compute fluctuation strength (vacil) and roughness (asper) following **Zwicker’s modulation model**.

| Quantity | Unit | Typical range |
|-----------|------|----------------|
| Fluctuation strength | vacil | 0 – 1 |
| Roughness | asper | 0 – 1.5 |

#### Reference
- ISO 532‑1:2017  
- Zwicker & Fastl (2007)

---

### 5️⃣ `tonality.py`
Implements tonal metrics per **ECMA‑418‑1** and **Aures (1985)**.

**Functions:**
- `tone_to_noise_ratio()` — TNR calculation per ECMA‑418‑1  
- `prominence_ratio()` — PR calculation per ECMA‑418‑1  
- `tonality_aures()` — Aures tonality index

#### Example
```python
tnr = pt.tone_to_noise_ratio(signal, fs, time_varying=True, plot=True)
```

#### References
- ECMA‑418‑1:2025 — *Prominent discrete tones*  
- Aures (1985), *Acustica 59*  
- Terhardt et al. (1982), *JASA*

---

### 6️⃣ `sottek_hm.py`
Implements **ECMA‑418‑2:2025 (Sottek hearing model)** for loudness and tonality.

**Functions:**
- `loudness_sottek()` — Loudness according to Sottek model  
- `tonality_sottek()` — Tonality based on loudness partitioning

#### Reference
- ECMA‑418‑2:2025 — *Methods for describing human perception based on the Sottek Hearing Model*

---

### 7️⃣ `compare_metrics.py`
Tool for visual comparison of psychoacoustic metrics between analysis runs.  
Generates PDF/PNG plots for selected metrics (loudness, sharpness, roughness, tonality).

---

### 8️⃣ `modulation.py`
Utility functions for modulation detection and envelope analysis used in fluctuation and roughness models.

---

## 📚 Theoretical Background

| Concept | Description |
|----------|--------------|
| **Loudness** | Subjective intensity of sound; depends on frequency and level. |
| **Sharpness** | High‑frequency weighting of specific loudness (DIN 45692). |
| **Fluctuation Strength** | Perception of slow amplitude modulations (≤ 20 Hz). |
| **Roughness** | Perception of faster modulations (≈ 20–150 Hz). |
| **Tonality** | Degree of tonal content; quantifies prominence of discrete tones. |
| **Sottek Hearing Model** | Nonlinear filterbank model (ECMA‑418‑2) modeling auditory excitation and temporal integration. |

---

## 🧪 Validation

Validation against reference MATLAB Psychoacoustics Toolbox and ECMA verification datasets shows < 2 % RMS deviation for all metrics within standard-defined frequency and level ranges.

---

## 📄 Citation

> Greco, G. F., *PsyTools: A Python Toolbox for Psychoacoustic Metrics (Zwicker, Aures, Sottek, ECMA‑418)*, 2025.

---

## ⚖️ License

Released under the **MIT License**.  
See the [LICENSE](LICENSE) file for details.
