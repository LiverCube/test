# PsyTools — Psychoacoustic Toolbox

*A modular Python toolbox for psychoacoustic metrics (Zwicker, Aures, Sottek, ECMA-418) with automatic CSV export and plotting.*

## Features

- ✅ Zwicker loudness (ISO 532-1:2017), sharpness (DIN 45692), fluctuation & roughness  
- ✅ ECMA-418-1 (TNR/PR) and ECMA-418-2 (Sottek hearing model)  
- ✅ Stationary and time-varying analysis (2 ms standard resolution)  
- ✅ Automatic export to CSV and plots (PNG/PDF) in a clean folder layout  
- ✅ Works on audio time-signals or 1/3-octave spectra  
- ✅ Simple API, optional plotting, and comparison utilities

---

## Installation

```bash
pip install psytools  # or: pip install -e .
```

---

## Requirements

- Python ≥ 3.8  
- numpy, scipy, matplotlib  
- librosa (optional, for .mp3 decoding)

---

## Inputs & Outputs

**Input formats**  
- Waveform audio (`.wav`, `.mp3`, etc.) as NumPy array with sampling rate `fs`  
- 1/3-octave spectra (`spectrum_db`, shape (28,) or (N, 28)), in dB SPL (Z-weighted)

**ISO 532-1:2017 conventions**  
- Stationary spectra: 28-band SPL values (25 Hz – 12.5 kHz)  
- Time-varying spectra: sampled every 2 ms

**Output**  
All results are exported automatically as `.csv` files and plots into the `/output/` directory.

---

## Quick Start

```python
from scipy.io import wavfile
import psytools as pt

fs, signal = wavfile.read("example.wav")
L_total, L_spec, percentiles = pt.loudness_zwicker(signal=signal, fs=fs, time_varying=True, plot=True)
print(L_total)
```

---

## Modules Overview

### analysis.py
Main module for automated psychoacoustic analysis of audio or spectrum data. Runs all core metrics and saves results/plots in structured directories.

**Main function:** `psyacoustic_analysis()`  
Computes full psychoacoustic analysis pipeline.

**Outputs:**  
- Loudness (Zwicker & Sottek)  
- Sharpness  
- Fluctuation & roughness  
- Tonality (Aures & Sottek)  
- TNR & PR metrics  

---

### compare_metrics.py
Compare psychoacoustic metrics across multiple result folders.

**Main function:** `compare_metrics()`  
Generates comparison plots and saves PNG/PDF in structured folders.

---

### loudness.py
Implements Zwicker loudness (ISO 532‑1) for stationary and time‑varying signals.

**Main function:** `loudness_zwicker()`  
Computes total and specific loudness based on ISO 532‑1.

**References:**  
- ISO 532‑1:2017 — *Acoustics – Methods for calculating loudness – Part 1: Zwicker method*  
- E. Zwicker & H. Fastl, *Psychoacoustics: Facts and Models*, 3rd ed., 2017

---

### sharpness.py
Computes perceived acoustic sharpness (acum) using DIN 45692, Aures, or von Bismarck weighting models.

**Main function:** `acoustic_sharpness()`  
Computes total or time‑varying sharpness from an audio signal or specific loudness.

**References:**  
- DIN 45692:2009 — *Acoustics – Determination of perceived sound quality – Sharpness*  
- Fastl & Zwicker, *Psychoacoustics: Facts and Models*, 2007

---

### fluctuation.py & roughness.py
Compute fluctuation strength (vacil) and roughness (asper) following Zwicker models (ISO 532‑1).

**References:**  
- ISO 532‑1:2017 — *Acoustics – Methods for calculating loudness – Part 1: Zwicker method*

---

### tonality.py
Implements tonal perception metrics (ECMA‑418‑1) and Aures Tonality Index.

**References:**  
- Aures (1985) *Acustica 59*  
- Terhardt et al. (1982) *JASA*  
- ECMA‑418‑1:2025

---

### sottek_hm.py
Implements ECMA‑418‑2 (Sottek model) for loudness and tonality.

**References:**  
- ECMA‑418‑2:2025 — *Methods for describing human perception based on the Sottek Hearing Model*

---

### modulation.py
Helper functions for modulation‑based psychoacoustic analysis.

**References:**  
- ISO 532‑1:2017  
- Fastl & Zwicker (2007)

---

## Citation

If you use **PsyTools** in academic work, please cite:

> *PsyTools: A Python Toolbox for Psychoacoustic Metrics (Zwicker, Aures, Sottek, ECMA‑418)*

---

## License

Distributed under the **MIT License**.  
See the [LICENSE](LICENSE) file for details.
