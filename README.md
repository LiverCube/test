# PsyTools - Psychoacoustic Toolbox

A modular Python toolbox for computing psychoacoustic metrics (Zwicker, Aures, Sottek, ECMA‑418) 
with automatic data export and plotting.

---

## Requirements

- Python ≥ 3.8  
- `numpy`, `scipy`, `matplotlib`  
- `librosa` (optional, for `.mp3` decoding)

---

## Inputs & Outputs

**Input formats**
- Waveform audio (`.wav`, `.mp3`, etc.) as NumPy array with sampling rate `fs`
- 1/3‑octave spectra (`spectrum_db`, shape `(28,)` or `(N, 28)`), in dB SPL (*Z‑weighted*)

**ISO 532‑1:2017 conventions**
- Stationary spectra: 28‑band SPL values (25 Hz – 12.5 kHz)
- Time‑varying spectra: sampled every 2 ms

**Output**
- All results are automatically exported as `.csv` files and plots into the `/output/` directory.

---

## Quick Start

```python
from scipy.io import wavfile
import psytools as pt

fs, signal = wavfile.read("example.wav")
L_total, L_spec, percentiles = pt.loudness_zwicker(signal, fs, time_varying=True, plot=True)
print(L_total)
```

---

## Modules Overview

### analysis.py

Main module for automated psychoacoustic analysis of audio or spectrum data.  
Runs all core metrics of the PsyTools Toolbox and automatically saves results and plots.

**Main function:** `psyacoustic_analysis()`  
Runs the full psychoacoustic analysis pipeline.

**Inputs**
- `input_path` (str, optional): Path to input file (WAV, audio, or AVL CSV).
- `input_mode` (str): Input type: `'wave'`, `'spectrum'`, `'AVL'`, or `'audio'`.
- `sound_field` (str): Sound field compensation, `'free'` or `'diffuse'`.
- `time_varying` (bool): Compute time‑varying metrics if True.
- `time_resolution` (str): `'standard'` (2 ms) or `'high'` (0.5 ms).
- `show_plots` (bool): Display plots if True.
- `save_csv` (bool): Save CSV files automatically if True.
- `spectrum_db` (np.ndarray, optional): 1/3‑octave spectrum (only for `'spectrum'` mode).
- `avl_path` (str, optional): Path to AVL CSV file (only for `'AVL'` mode).
- `params` (list[str], optional): Metrics to compute.  
  Options: `"plot_input"`, `"loudness_zwicker"`, `"sharpness"`, `"loudness_sottek"`, 
  `"fluctuation"`, `"roughness"`, `"tonality_sottek"`, `"tonality_aures"`, `"tnr"`, `"pr"`.
- `output_base` (str, optional): Base directory for all outputs. Default: relative to `input_path`.

**Outputs**
- `results` (dict): Computed values for each metric, e.g.:
  - `loudness_stationary`: Total Zwicker loudness (sones)
  - `sharpness`: Sharpness (acum)
  - `roughness`: Roughness (asper)
  - `tonality_sottek`: Tonality (tu_HMS)

**Features**
- Handles multiple input types (`wave`, `spectrum`, `AVL`, `audio`)
- Logs all console output (`run_YYYYMMDD_HHMMSS.txt`)
- Structured output directories:  
  ```
  output/
  └─ <soundfile-name>/
     ├─ run_YYYYMMDD_HHMMSS.txt
     ├─ csv/
     └─ plots/
        ├─ pdf/
        └─ png/
  ```
- Norm‑compliant with ISO 532‑1, DIN 45692, ECMA‑418‑1/‑2  
- Optional saving of plots as PNG/PDF

---

### compare_metrics.py

Visualize and compare psychoacoustic metrics across multiple analysis outputs.

**Main function:** `compare_metrics()`  
Generates comparison plots for psychoacoustic metrics across result folders.

**Inputs**
- `output_dirs` (list of str): Folders containing psychoacoustic analysis CSV files.  
- `font_size` (int, optional): Font size for plots (default 14).  
- `save_dir` (str, optional): Directory for saving generated plots (default `"output/comparisons"`).  
- `metrics` (list[str] or None, optional): List of metric filenames; if None, use all found.  
- `show` (bool, optional): Show all generated plots interactively.

**Features**
- Auto‑detects available metrics  
- Aligns time axes across folders for time‑varying metrics  
- Saves plots as PNG/PDF in structured subfolders  
- Skips missing metrics gracefully  
- Easily extendable via new plotting functions

---

### signal_processing.py

General‑purpose tools for audio loading, spectrum analysis, and signal synthesis.

**Functions**
- `synthesize_signal_from_spectrum()` – Create signals from sinusoidal components  
- `process_mic_data()` – Load mic CSVs, convert to Pa, compute 1/3‑octave spectra  
- `load_audio()` / `load_audio_librosa()` – Audio loading and normalization  
- `plot_input()` – Visualize time signals or spectra  
- `build_output_structure()` – Create output folders (/csv, /plots/pdf, /plots/png)  
- `save_all_figures()` – Save all open figures to PNG/PDF

---

### loudness.py

Implements Zwicker loudness (ISO 532‑1) for stationary and time‑varying inputs.

**Main function:** `loudness_zwicker()`  
Computes total and specific loudness based on ISO 532‑1.

**References**
- ISO 532‑1:2017 — *Acoustics – Methods for calculating loudness – Part 1: Zwicker method*  
- E. Zwicker & H. Fastl, *Psychoacoustics: Facts and Models*, 3rd ed., 2017

---

### sharpness.py

Computes perceived acoustic sharpness (acum) using DIN 45692, Aures, or von Bismarck weighting models.

**Main function:** `acoustic_sharpness()`  
Computes total or time‑varying sharpness from an audio signal or precomputed specific loudness.

**References**
- DIN 45692:2009 — *Acoustics – Determination of perceived sound quality – Sharpness*  
- Fastl & Zwicker (2007) *Psychoacoustics: Facts and Models*, 3rd ed.

---

### fluctuation.py

Computes acoustic fluctuation strength (vacil) based on the Zwicker model (ISO 532‑1).

**References**
- ISO 532‑1:2017 — *Acoustics – Methods for calculating loudness – Part 1: Zwicker method*  
- Fastl & Zwicker (2007) *Psychoacoustics: Facts and Models*, 3rd ed.

---

### roughness.py

Computes acoustic roughness (asper) based on the Zwicker model (ISO 532‑1).

**References**
- ISO 532‑1:2017 — *Acoustics – Methods for calculating loudness – Part 1: Zwicker method*  
- Fastl & Zwicker (2007) *Psychoacoustics: Facts and Models*, 3rd ed.

---

### tonality.py

Implements tonal perception metrics based on ECMA‑418‑1 and the Aures Tonality Index.

**References**
- Aures (1985), *Berechnungsverfahren für den sensorischen Wohlklang*, Acustica 59  
- Terhardt et al. (1982), *Algorithm for extraction of pitch and pitch salience*, JASA  
- ECMA‑418‑2:2025 — Prominent discrete tones

---

### sottek_hm.py

Implements ECMA‑418‑2 (Sottek model) for loudness and tonality.

**References**
- ECMA‑418‑2:2025 — *Methods for describing human perception based on the Sottek Hearing Model*

---

### modulation.py

Helper functions for modulation‑based psychoacoustic analysis, used in roughness and fluctuation computation.

**References**
- ISO 532‑1:2017 — *Acoustics – Methods for calculating loudness – Zwicker method*  
- Fastl & Zwicker (2007) *Psychoacoustics: Facts and Models*, 3rd ed.

---

## License & Citation

This software is released under the **MIT License** (see LICENSE file).

If you use *PsyTools* in scientific work, please cite it as:

> Your Name (2025). *PsyTools: A modular Python toolbox for psychoacoustic metrics.*  
> Zenodo. https://doi.org/10.xxxx/zenodo.xxxxx
