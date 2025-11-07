# PsyTools — Psychoacoustic Toolbox

*A comprehensive Python toolbox for psychoacoustic analysis based on standardized hearing models (Zwicker, Aures, Sottek, ECMA‑418).*

---

## Requirements

- Python ≥ 3.8  
- numpy, scipy, matplotlib  
- librosa (optional, for .mp3 decoding)

---

## Input / Output Formats

### Inputs

- **Audio signals:** `.wav`, `.mp3`, or NumPy array with sampling rate `fs`  
- **1/3‑octave spectra:** 28 bands (25 Hz–12.5 kHz), dB SPL (Z‑weighted)

**ISO 532-1:2017 conventions**  

- Stationary spectra: 28-band SPL values (25 Hz – 12.5 kHz)  
- Time-varying spectra: sampled every 2 ms or 0.5 ms

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

## Quick Start

```python
from scipy.io import wavfile
import psytools as pt

fs, signal = wavfile.read("example.wav")
L_total, L_spec, percentiles = pt.loudness_zwicker(signal=signal, fs=fs, time_varying=False, plot=True)
print(L_total)
```
or
```python
import psytools as pt

results = pt.psyacoustic_analysis(
    input_path="example.wav",
    input_mode="wave",
    params=["loudness_zwicker","sharpness"]
    time_varying=True,
    show_plots=True,
)
print(results["sharpness"])
```
---

## Module Reference

## 🔊 `loudness.py`

Implements **Zwicker loudness** according to **ISO 532-1:2017** for stationary and time-varying inputs.

### 🧩 Main Function — `loudness_zwicker()`

| **Parameter** | **Type** | **Description** |
|----------------|-----------|-----------------|
| `signal` | `np.ndarray`, optional | Audio waveform (mono). |
| `fs` | `int`, optional | Sampling frequency in Hz. |
| `cal` | `float`, optional | Calibration factor (default √8, ≈ 100 dB SPL FS). |
| `spectrum_db` | `np.ndarray`, optional | 1/3-octave SPL spectrum (28 bands). |
| `output_in_phon` | `bool`, optional | Convert sones → phon if True. |
| `remove` | `bool`, optional | Apply warm-up correction (stationary). |
| `sound_field` | `str`, optional | `'free'` or `'diffuse'` field correction. |
| `time_varying` | `bool`, optional | Compute time-varying loudness. |
| `time_resolution` | `str`, optional | `'standard'` (2 ms) or `'high'` (0.5 ms). |
| `plot` | `bool`, optional | Generate plots. |
| `font_size` | `int`, optional | Font size for plots (default 14). |
| `main_color` | `str`, optional | Color for total loudness line. |
| `cmap` | `str`, optional | Colormap for specific loudness (default `'viridis'`). |
| `save_csv` | `bool`, optional | Save results as CSV. |
| `output_dir` | `str`, optional | Output directory for CSV. |

**Returns**

| **Name** | **Description** |
|-----------|----------------|
| `total_loudness` | Total loudness (scalar / time series). |
| `specific_loudness` | Specific loudness (240 Bark bins). |
| `percentiles` | Dictionary with `Nmax`, `N5` for time-varying mode. |

**Helper Functions**

| Function | Description |
|-----------|-------------|
| `smooth_signal_biquad()` | Temporal smoothing using cascaded biquad filters (ISO 532-1 §6.3). |
| `apply_nonlinear_decay_iso()` | Diode-like nonlinear integration (Annex A.10.2). |
| `iso_5321_frequency_spreading()` | Maps core loudness → 0.1 Bark resolution and applies spreading. |

**References**

- ISO 532-1:2017 — *Methods for calculating loudness – Part 1: Zwicker method*  
- E. Zwicker & H. Fastl (2017) *Psychoacoustics: Facts and Models*

---

## ✨ `sharpness.py`

Computes **acoustic sharpness** (acum) using **DIN 45692**, **Aures**, or **von Bismarck** weighting.

### 🧩 Main Function — `acoustic_sharpness()`

| **Parameter** | **Type** | **Description** |
|----------------|-----------|-----------------|
| `signal` | `np.ndarray`, optional | Mono waveform. |
| `sample_rate` | `int`, optional | Sampling rate (default 48 kHz). |
| `weighting` | `str`, optional | `'DIN 45692'`, `'Aures'`, or `'von Bismarck'`. |
| `time_varying` | `bool`, optional | Compute per frame. |
| `plot` | `bool`, optional | Show sharpness plot. |
| `font_size` | `int`, optional | Font size for plots. |
| `main_color` | `str`, optional | Plot color for total sharpness. |
| `specific_loudness` | `np.ndarray`, optional | Precomputed specific loudness array. |
| `total_loudness` | `float or ndarray`, optional | Total loudness input. |
| `output_dir` | `str`, optional | Directory for CSV export. |
| `safe_csv` | `bool`, optional | Save results as CSV. |

**Returns**

| **Name** | **Description** |
|-----------|----------------|
| `sharpness` | Sharpness value (acum, scalar or array). |

**Helper Functions**

| Function | Description |
|-----------|-------------|
| `get_weighting_g()` | Frequency weighting function g(z). |
| `get_normalization_constant()` | Returns normalization factor. |

**References**

- DIN 45692:2009 — *Acoustics – Sharpness determination*  
- Fastl & Zwicker (2007)

---

## 🌊 `fluctuation.py`

Computes **fluctuation strength (vacil)** based on the **Zwicker model (ISO 532‑1)**.

### 🧩 Main Function — `acoustic_fluctuation()`

| **Parameter** | **Type** | **Description** |
|----------------|-----------|-----------------|
| `signal` | `np.ndarray`, optional | Audio waveform. |
| `fs` | `int`, optional | Sampling frequency in Hz. |
| `fmod` | `float or str`, optional | Modulation frequency in Hz or `'auto-detect'`. |
| `specific_loudness` | `np.ndarray`, optional | Precomputed specific loudness array. |
| `spec_fs` | `int`, optional | Sampling rate of specific loudness (default 500 Hz). |
| `plot` | `bool`, optional | Generate fluctuation strength plots. |
| `font_size` | `int`, optional | Font size for plots. |
| `main_color` | `str`, optional | Plot color. |
| `cmap` | `str`, optional | Colormap (default `'viridis'`). |
| `save_csv` | `bool`, optional | Save results as CSV. |
| `output_dir` | `str`, optional | Output directory. |

**Returns**

| **Name** | **Description** |
|-----------|----------------|
| `fluctuation` | Total fluctuation strength (vacil). |
| `specific` | Specific fluctuation per Bark band (vacil/Bark). |
| `fmod` | Modulation frequency used (Hz). |

**Helper Functions**

| Function | Description |
|-----------|-------------|
| `compute_fluctuation()` | Calculates total/specific fluctuation using envelope modulation analysis. |

**References**

- ISO 532‑1:2017 — *Zwicker model for loudness and modulation perception*  
- Fastl & Zwicker (2007)

---

## ⚡ `roughness.py`

Computes **roughness (asper)** following the **Zwicker model (ISO 532‑1)**.

### 🧩 Main Function — `acoustic_roughness()`

| **Parameter** | **Type** | **Description** |
|----------------|-----------|-----------------|
| `signal` | `np.ndarray`, optional | Audio waveform. |
| `fs` | `int`, optional | Sampling frequency in Hz. |
| `fmod` | `float or str`, optional | Modulation frequency or `'auto-detect'`. |
| `specific_loudness` | `np.ndarray`, optional | Precomputed specific loudness. |
| `plot` | `bool`, optional | Display roughness plot. |
| `font_size` | `int`, optional | Font size for plots. |
| `main_color` | `str`, optional | Color for total roughness plot. |
| `cmap` | `str`, optional | Colormap for specific roughness. |
| `save_csv` | `bool`, optional | Save CSV output. |
| `output_dir` | `str`, optional | Output directory. |

**Returns**

| **Name** | **Description** |
|-----------|----------------|
| `asper` | Total roughness (asper). |
| `spec` | Specific roughness (asper/Bark). |
| `fmod` | Modulation frequency used. |

**Helper Functions**

| Function | Description |
|-----------|-------------|
| `compute_roughness()` | Calculates total and specific roughness using modulation depth and band aggregation. |

**References**

- ISO 532‑1:2017  
- Fastl & Zwicker (2007)

---

## 🎵 `tonality.py`

Implements **tonal perception metrics** (ECMA‑418‑1) and **Aures tonality**.

### 🧩 Main Functions

#### `tonality_aures()`
Computes Aures tonality index for stationary or time‑varying input.

#### `tone_to_noise_ratio()`
Computes Tone‑to‑Noise Ratio (TNR) according to ECMA‑418‑1.

#### `prominence_ratio()`
Computes Prominence Ratio (PR) per ECMA‑418‑1.

**Common Parameters**

| **Parameter** | **Type** | **Description** |
|----------------|-----------|-----------------|
| `signal_in` | `np.ndarray` | Input audio signal (mono). |
| `sample_rate` | `int` | Sampling frequency (Hz). |
| `frame_size` | `int`, optional | FFT frame size (default 16384). |
| `hop_size` | `int`, optional | Hop size between frames (default 8192). |
| `time_varying` | `bool`, optional | Enable time‑varying mode. |
| `plot` | `bool`, optional | Display spectrum and TNR/PR plots. |
| `font_size` | `int`, optional | Font size for plots. |
| `main_color` | `str`, optional | Main color for plots. |
| `cmap` | `str`, optional | Colormap for visualizations. |
| `prominence_only` | `bool`, optional | Return only prominent tones (TNR≥3 dB). |
| `safe_csv` | `bool`, optional | Export CSV if True. |
| `output_dir` | `str`, optional | CSV output directory. |

**Outputs**

| **Function** | **Return Type** | **Description** |
|---------------|----------------|-----------------|
| `tonality_aures()` | `tuple(np.ndarray, np.ndarray)` | Tonality index (scalar/array) and time vector. |
| `tone_to_noise_ratio()` | `dict` | Contains `tnr`, `frequencies`, `prominent`, `times`, `peak_limits`, `f_axis`. |
| `prominence_ratio()` | `dict` | Contains `PR`, `freqs`, `is_prominent`, `times`. |

**Helper Functions**

| Function | Description |
|-----------|-------------|
| `critical_band_limits()` | Calculates critical band boundaries. |
| `octave_24th_bands()` | Generates 1/24-octave bands for tone detection. |
| `smooth_in_db_domain()` | Smooths PSD in the decibel domain. |
| *(+ internal Gaussian smoothing, tonal width estimation, and plotting utilities)* |  |

**References**

- Aures (1985) *Acustica 59*  
- Terhardt et al. (1982) *JASA*  
- ECMA‑418‑1:2025

---

## 🧠 `sottek_hm.py`

Implements **ECMA‑418‑2:2025 (Sottek hearing model)** for **loudness** and **tonality**.

### 🧩 Main Functions

| Function | Description |
|-----------|-------------|
| `loudness_sottek()` | Computes total/specific loudness using Sottek model. |
| `tonality_sottek()` | Computes time‑ and frequency‑dependent tonality from auditory filter outputs. |

**Shared Parameters**

| **Parameter** | **Type** | **Description** |
|----------------|-----------|-----------------|
| `insig` | `np.ndarray` | Input signal (mono/stereo). |
| `fs` | `int` | Sampling frequency (Hz). |
| `plot` | `bool`, optional | Show results. |
| `fieldtype` | `str`, optional | `'free'` or `'diffuse'` field. |
| `calibration_factor` | `float`, optional | Calibration scaling factor. |
| `save_csv` | `bool`, optional | Save CSV outputs. |
| `output_dir` | `str`, optional | Export directory. |

**Outputs**

| **Name** | **Description** |
|-----------|----------------|
| `specLoudness` | Specific loudness vs. time and Bark band (sone/Bark). |
| `loudnessTDep` | Time‑dependent total loudness (sone). |
| `loudnessPowAvg` | Power‑averaged loudness. |
| `specLoudnessPowAvg` | Power‑averaged specific loudness. |
| `specTonality` | Specific tonality per Bark band. |
| `tonalityTDep` | Time‑varying tonality index. |
| `soundField` | Applied sound field setting. |

**Helper Functions**

| Function | Description |
|-----------|-------------|
| `shm_resample()` | Resample to 48 kHz using polyphase filtering. |
| `shm_preproc()` | Apply fade‑in and zero‑padding. |
| `shm_out_mid_ear_filter()` | Apply outer/middle‑ear filtering. |
| `shm_auditory_filt_bank()` | Compute auditory band outputs (53 half‑Bark). |
| `shm_signal_segment()` | Segment signal into overlapped blocks. |

**References**

- ECMA‑418‑2:2025 — *Sottek Hearing Model*

---

###  `analysis.py`
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

## 📈 `compare_metrics.py`

Visualizes and compares psychoacoustic results across different analysis runs.

### 🧩 Main Function — `compare_metrics()`

| **Parameter** | **Type** | **Description** |
|----------------|-----------|-----------------|
| `output_dirs` | `list[str]` | List of folders containing psychoacoustic CSV results to compare. |
| `font_size` | `int`, optional | Font size for plots (default = 14). |
| `save_dir` | `str`, optional | Output directory for plots (`output/comparisons`). |
| `metrics` | `list[str]`, optional | List of metric filenames to compare (default = all). |
| `show` | `bool`, optional | Show plots interactively (default False). |

**Returns**

| **Name** | **Description** |
|-----------|----------------|
| *(None)* | Generates and saves all comparison plots (PNG/PDF). |

**Helper Functions**

| Function | Description |
|-----------|-------------|
| `plot_loudness_stationary_compare()` | Compare stationary loudness results. |
| `plot_loudness_time_varying_compare()` | Compare time‑varying loudness (Zwicker). |
| `plot_sharpness_time_varying_compare()` | Compare sharpness time‑series. |
| `plot_fluctuation_strength_compare()` | Compare fluctuation strength curves. |
| `plot_loudness_sottek_compare()` | Compare ECMA‑418‑2 Sottek loudness results. |
| `plot_tonality_sottek_compare()` | Compare Sottek tonality across runs. |
| `plot_tonality_aures_compare()` | Compare Aures tonality results. |

**Features**

- Automatically detects which metrics are available in each folder.  
- Aligns time axes across analyses for fair comparison.  
- Saves plots in structured folders (`/plots/png` and `/plots/pdf`).  
- Prints warnings for missing or mismatched data.  
- Easily extendable via new plot registration in `PLOT_FUNCTIONS`.

---

## 🎛️ `signal_processing.py`

Provides general‑purpose utilities for **audio loading**, **spectrum synthesis**, and **plotting**.

### 🧩 Functions

| **Function** | **Description** |
|---------------|----------------|
| `synthesize_signal_from_spectrum()` | Creates synthetic test signals from sinusoidal spectral components. |
| `process_mic_data()` | Loads microphone CSV data, converts to Pa, and computes 1/3‑octave spectra. |
| `load_audio()` / `load_audio_librosa()` | Load audio signals from WAV or compressed formats, normalize to RMS = 1. |
| `plot_input()` | Visualizes time signals or 1/3‑octave spectra. |
| `build_output_structure()` | Creates `/output/csv`, `/output/plots/pdf`, and `/output/plots/png` directories automatically. |
| `save_all_figures()` | Saves all open matplotlib figures as PNG and PDF. |

**Notes**

- Handles both time‑domain and frequency‑domain inputs.  
- Normalizes and calibrates audio before analysis.  
- Ensures consistent directory structure across analyses.  

---

## 🌀 `modulation.py`

Helper functions for modulation‑based psychoacoustic analysis.

| Function | Description |
|-----------|-------------|
| `sone2phon()` | Convert specific loudness from sone → phon scale. |
| `assemble_bark_bands()` | Aggregate fine‑resolution channels into Bark bands. |
| `filter_dc_and_transient()` | Remove DC offset and transient effects. |
| `design_octave_filter()` | Design a Butterworth bandpass filter centered at f₀. |
| `apply_octave_filter()` | Apply octave‑based bandpass to a signal. |
| `envelope_filter()` | Extract signal envelope using zero‑phase LP filter. |
| `detect_fmod()` | Detect dominant modulation frequency. |

**References**

- ISO 532‑1:2017 — *Methods for calculating loudness – Zwicker method*  
- Fastl & Zwicker (2007)

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
