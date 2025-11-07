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

Results can optionally be saved as CSV and plots (PNG/PDF) if `save_csv=True` (or module-dependent). Files are written to the `/output/` directory.  
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
    params=["loudness_zwicker","sharpness"],
    time_varying=True,
    show_plots=True,
)
print(results["sharpness"])
```
---

## Modules

## `loudness.py`

Implements **Zwicker loudness** according to **ISO 532-1:2017** for stationary and time-varying inputs.

### Main Function — `loudness_zwicker()`

| **Parameter** | **Type** | **Description** |
|----------------|-----------|-----------------|
| `signal` | `np.ndarray`, optional | Audio waveform (mono). |
| `fs` | `int`, optional | Sampling frequency in Hz. |
| `cal` | `float`, optional | Calibration factor (default √8). |
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
| `specific_loudness` | Specific loudness (240 Bark bins, shape = (time × 240)). |
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

## `sharpness.py`

Computes **acoustic sharpness** (acum) using **DIN 45692**, **Aures**, or **von Bismarck** weighting.

### Main Function — `acoustic_sharpness()`

| **Parameter** | **Type** | **Description** |
|----------------|-----------|-----------------|
| `signal` | `np.ndarray`, optional | Mono waveform. |
| `fs` | `int`, optional | Sampling rate (default 48 kHz). |
| `weighting` | `str`, optional | `'DIN 45692'`, `'Aures'`, or `'von Bismarck'`. |
| `time_varying` | `bool`, optional | Compute per frame. |
| `plot` | `bool`, optional | Show sharpness plot. |
| `font_size` | `int`, optional | Font size for plots. |
| `main_color` | `str`, optional | Plot color for total sharpness. |
| `specific_loudness` | `np.ndarray`, optional | Precomputed specific loudness array. |
| `total_loudness` | `float or ndarray`, optional | Total loudness input. |
| `output_dir` | `str`, optional | Directory for CSV export. |
| `save_csv` | `bool`, optional | Save results as CSV. |

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
- E. Zwicker & H. Fastl (2017) *Psychoacoustics: Facts and Models*

---

## `fluctuation.py`

Computes **fluctuation strength (vacil)** based on the **Zwicker model (ISO 532‑1)**.

### Main Function — `acoustic_fluctuation()`

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
| `cmap` | `str`, optional | Colormap for specific fluctuation strength (default `'viridis'`). |
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
- E. Zwicker & H. Fastl (2017) *Psychoacoustics: Facts and Models*

---

## `roughness.py`

Computes **roughness (asper)** following the **Zwicker model (ISO 532‑1)**.

### Main Function — `acoustic_roughness()`

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

- ISO 532‑1:2017 — *Zwicker model for loudness and modulation perception*  
- E. Zwicker & H. Fastl (2017) *Psychoacoustics: Facts and Models*

---

## `modulation.py`

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
- E. Zwicker & H. Fastl (2017) *Psychoacoustics: Facts and Models*

---

## `tonality.py`

Implements **tonal perception metrics** according to **ECMA‑418‑1** and **Aures (1985)** tonality model.

## `tonality_aures()`

Computes the **Aures tonality index** for stationary or time‑varying input signals.

### Parameters

| **Parameter** | **Type** | **Description** |
|----------------|-----------|-----------------|
| `signal` | `np.ndarray` | Input monophonic audio signal. |
| `fs` | `int` | Sampling frequency in Hz. |
| `time_varying` | `bool`, optional | If True, compute frame-wise tonality (default True). |
| `plot` | `bool`, optional | Display time-varying tonality results (default False). |
| `font_size` | `int`, optional | Font size for plots (default 14). |
| `main_color` | `str`, optional | Main color for plot (default `'blue'`). |
| `save_csv` | `bool`, optional | Export results to CSV if True (default False). |
| `output_dir` | `str`, optional | Directory for CSV export (required if `save_csv=True`). |

### Returns

| **Name** | **Type** | **Description** |
|-----------|-----------|----------------|
| `tonalities` | `np.ndarray or float` | Tonality index per frame (time-varying) or scalar (stationary). |
| `times` | `np.ndarray or None` | Time vector corresponding to tonalities (None for stationary). |

## `tone_to_noise_ratio()`

Computes the **Tone‑to‑Noise Ratio (TNR)** according to **ECMA‑418‑1**.

### Parameters

| **Parameter** | **Type** | **Description** |
|----------------|-----------|-----------------|
| `signal` | `np.ndarray` | Input audio signal (1D, mono). |
| `fs` | `int` | Sampling frequency in Hz. |
| `frame_size` | `int`, optional | FFT frame size (default 16384). |
| `hop_size` | `int`, optional | Hop size between frames (default 8192, 50% overlap). |
| `time_varying` | `bool`, optional | Compute time-varying TNR if True (default False). |
| `plot` | `bool`, optional | Show plots of spectrum and TNR (default False). |
| `font_size` | `int`, optional | Font size for the plots (default 14). |
| `main_color` | `str`, optional | Main color used for plot (default `'blue'`). |
| `cmap` | `str`, optional | Colormap used for plotting heatmaps (default `'viridis'`). |
| `prominence_only` | `bool`, optional | Return only prominent tones (default False). |
| `save_csv` | `bool`, optional | Export CSV files if True (default False). |
| `output_dir` | `str`, optional | Directory to save CSV files (required if `save_csv=True`). |

### Returns

| **Name** | **Type** | **Description** |
|-----------|-----------|----------------|
| `results` | `dict` | Dictionary containing: |
| || `'tnr'` : list or array — TNR values (dB). |
| || `'freqs'` : np.ndarray — Detected tone frequencies (Hz). |
| || `'is_prominent'` : np.ndarray — Boolean flags for prominent tones. |
| || `'times'` : np.ndarray or None — Frame timestamps (s) for time-varying analysis. |
| || `'peak_limits'` : list[tuple[int, int]] — Frequency bin limits of detected peaks. |
| || `'freq_axis'` : np.ndarray — Frequency axis used for PSD calculation. |

## `prominence_ratio()`

Computes the **Prominence Ratio (PR)** per **ECMA‑418‑1** standard.

### Parameters

| **Parameter** | **Type** | **Description** |
|----------------|-----------|-----------------|
| `signal` | `np.ndarray` | Input audio signal (mono). |
| `fs` | `int` | Sampling rate in Hz. |
| `frame_size` | `int`, optional | FFT frame size (default 16384). |
| `hop_size` | `int`, optional | Hop size between frames (default 8192). |
| `time_varying` | `bool`, optional | Compute time-resolved PR if True (default False). |
| `plot` | `bool`, optional | Show plots of PR and spectrum (default False). |
| `font_size` | `int`, optional | Font size for the plots (default 14). |
| `main_color` | `str`, optional | Main color for plot (default `'blue'`). |
| `cmap` | `str`, optional | Colormap used for plotting heatmaps (default `'viridis'`). |
| `window_type` | `str`, optional | Window function for FFT (default `'hann'`). |
| `frequency_range` | `tuple`, optional | Frequency range for analysis (Hz) (default (89.1, 11200)). |
| `output_dir` | `str`, optional | Directory for saving CSV files (required if `save_csv=True`). |
| `save_csv` | `bool`, optional | Export PR data to CSV if True (default False). |

### Returns

| **Name** | **Type** | **Description** |
|-----------|-----------|----------------|
| `results` | `dict` | Dictionary containing: |
| || `'pr'` : np.ndarray — PR values (vector or matrix). |
| || `'freqs'` : np.ndarray — Frequency bins corresponding to PR values. |
| || `'is_prominent'` : np.ndarray — Boolean mask of prominent tones. |
| || `'times'` : np.ndarray — Timestamps for frames (only for time-varying analysis). |

**Helper Functions**

| Function | Description |
|-----------|-------------|
| `critical_band_limits()` | Calculates critical-band frequency limits and bandwidths according to ECMA‑418‑1. |
| `critical_bandwidth_fft()` | Maps critical-band limits (Hz) to FFT-bin spans and computes total frequency width. |
| `band_level()` | Integrates linear PSD values within each specified frequency bin span. |
| `check_prominence()` | Applies ECMA‑418‑1 prominence criteria to TNR or PR values. |
| `octave_24th_bands()` | Generates 1/24‑octave frequency bands for tone detection. |
| `smooth_in_db_domain()` | Smooths PSD in the decibel domain to reduce noise. |
| *(+ internal Gaussian smoothing, tonal width estimation, audibility check, and plotting utilities)* |  |

**References**

- Aures, W. (1985). *Berechnungsverfahren für den sensorischen Wohlklang*. **Acustica**, 59.  
- Terhardt, E. et al. (1982). *Algorithm for extraction of pitch and pitch salience*. **JASA**.  
- ECMA‑418‑1:2025 — *Methods for the assessment of prominent discrete tones*.

---

## `sottek_hearing_model.py`

Implements **ECMA‑418‑2:2025 (Sottek hearing model)** for **loudness** and **tonality**.

## `loudness_sottek()`

Computes total and specific **loudness** according to the **Sottek hearing model** (ECMA‑418‑2:2025).

### Parameters

| **Parameter** | **Type** | **Description** |
|----------------|-----------|-----------------|
| `signal` | `np.ndarray` | Input signal (mono or stereo). |
| `fs` | `int or float` | Sampling frequency in Hz. |
| `plot` | `bool`, optional | Plot total and specific loudness results. |
| `font_size` | `int`, optional | Font size for plots (default = 14). |
| `main_color` | `str`, optional | Color for total loudness plot. |
| `cmap` | `str`, optional | Colormap for specific loudness (default `'viridis'`). |
| `sound_field` | `str`, optional | Sound field type — `'free'` or `'diffuse'` (default `'free'`). |
| `time_skip` | `float`, optional | Initial skip duration (s) for averaging (default 0.304 s). |
| `calibration_factor` | `float`, optional | Scaling factor applied to the input prior to processing. |
| `save_csv` | `bool`, optional | If True, automatically export results to CSV. |
| `output_dir` | `str`, optional | Directory for CSV output (required if `save_csv=True`). |
| `**kwargs` | `dict`, optional | Additional parameters (fade‑in time, block length, etc.). |

### Returns

| **Name** | **Type** | **Description** |
|-----------|-----------|----------------|
| `OUT` | `dict` | Dictionary containing: |
|  || `'specLoudness'` — Specific loudness vs. time and Bark band (sone/Bark). |
|  || `'loudnessTDep'` — Time‑dependent total loudness (sone). |
|  || `'loudnessPowAvg'` — Power‑averaged total loudness (sone). |
|  || `'specLoudnessPowAvg'` — Power‑averaged specific loudness (sone/Bark). |
|  || `'bandCentreFreqs'` — Bark‑band center frequencies (Hz). |
|  || `'timeOut'` — Output time vector (s). |
|  || `'soundField'` — Sound field setting (`'free'` or `'diffuse'`). |

## `tonality_sottek()`

Computes **tonality metrics** using the **Sottek model** (ECMA‑418‑2:2025).

### Parameters

| **Parameter** | **Type** | **Description** |
|----------------|-----------|-----------------|
| `signal` | `np.ndarray` | Input signal [samples × channels] in Pascal. |
| `fs` | `int` | Sampling frequency in Hz (resampled to 48 kHz if different). |
| `plot` | `bool`, optional | Plot tonality results. |
| `font_size` | `int`, optional | Font size for plots (default 14). |
| `main_color` | `str`, optional | Main color for total tonality. |
| `cmap` | `str`, optional | Colormap for specific tonality (default `'viridis'`). |
| `sound_field` | `str`, optional | `'free'` or `'diffuse'` (affects ear filtering). |
| `time_skip` | `float`, optional | Initial time (s) skipped for averaging (default 0.304 s). |
| `calibration_factor` | `float`, optional | Scaling factor applied before processing. |
| `save_csv` | `bool`, optional | Save CSV outputs if True. |
| `output_dir` | `str`, optional | Target directory for CSV output. |
| `**kwargs` | `dict`, optional | Additional parameters for internal processing. |

### Returns

| **Name** | **Type** | **Description** |
|-----------|-----------|----------------|
| `OUT` | `dict` | Dictionary containing: |
|  || `'specTonality'` — Time‑dependent specific tonality per Bark band. |
|  || `'specTonalityAvg'` — Time‑averaged specific tonality. |
|  || `'specTonalityFreqs'` — Frequency‑resolved specific tonality. |
|  || `'specTonalityAvgFreqs'` — Power‑averaged frequency‑resolved tonality. |
|  || `'specTonalLoudness'` — Tonal loudness per Bark band. |
|  || `'specNoiseLoudness'` — Noise loudness per Bark band. |
|  || `'tonalityTDep'` — Time‑dependent overall tonality. |
|  || `'tonalityAvg'` — Time‑averaged overall tonality. |
|  || `'tonalityTDepFreqs'` — Frequency‑resolved time‑dependent tonality. |
|  || `'bandCentreFreqs'` — Bark‑band center frequencies (Hz). |
|  || `'timeOut'` — Output time vector (s). |
|  || `'timesignal'` — Input time vector (s). |
|  || `'soundField'` — Sound field used (`'free'` or `'diffuse'`). |

**Helper Functions**

| Function | Description |
|-----------|-------------|
| `shm_resample()` | Resample to 48 kHz using polyphase filtering. |
| `shm_preproc()` | Apply fade‑in and zero‑padding. |
| `shm_out_mid_ear_filter()` | Apply outer/middle‑ear filtering. |
| `shm_auditory_filt_bank()` | Compute auditory band outputs (53 half‑Bark). |
| `shm_signal_segment()` | Segment signal into overlapped blocks. |

**References**

- ECMA‑418‑2:2025 — *Methods for describing human perception based on the Sottek Hearing Model*

---

###  `analysis.py`
Central entry point for automated psychoacoustic analysis.

### Main Function — `psyacoustic_analysis()`

### Parameters

| **Name** | **Type** | **Description** |
|-----------|-----------|-----------------|
| `input_path` | `str` | Path to the input file (audio, spectrum, or AVL). |
| `input_mode` | `str` | Defines the input mode: <br> • **`'wave'`** – For standard **WAV** files (PCM), loaded via `scipy.io.wavfile`. <br> • **`'audio'`** – Uses **librosa** for general audio decoding (e.g., **MP3**, AAC, FLAC). <br> • **`'spectrum'`** – Accepts spectral input according to standard conventions (e.g., 1/3-octave SPL values per ISO 532-1). <br> • **`'AVL'`** – Special mode that reads data from **Excel files** in the AVL input format. |
| `sound_field` | `str` | `'free'` or `'diffuse'` field for loudness calculations. |
| `time_varying` | `bool` | Enables time-varying calculations for supported metrics. |
| `time_resolution` | `str` | `'standard'` (2 ms) or `'high'` (0.5 ms). |
| `show_plots` | `bool` | Display result figures interactively. |
| `save_csv` | `bool` | Automatically export results to CSV files. |
| `params` | `list[str]` | Select psychoacoustic metrics to compute. Supported values: <br>  • `"plot_input"` — Plot waveform or spectrum.<br>  • `"loudness_zwicker"` — Loudness (ISO 532-1).<br>  • `"loudness_sottek"` — Sottek loudness (ECMA 418-2).<br>  • `"sharpness"` — Sharpness (DIN 45692).<br>  • `"fluctuation"` — Fluctuation strength (vacil).<br>  • `"roughness"` — Roughness (asper).<br>  • `"tonality_aures"` — Aures tonality (1985).<br>  • `"tonality_sottek"` — Sottek tonality (ECMA 418-2).<br>  • `"tnr"` — Tone-to-Noise Ratio (ECMA 418-1).<br>  • `"pr"` — Prominence Ratio (ECMA 418-1). |
| `output_base` | `str` | Base directory for saving plots and data. |

---

### Returns

Returns a **dictionary** with computed psychoacoustic results depending on the requested metrics in `params`.

| **Key in `results` dict** | **Value / Description** |
|-----------------------------|--------------------------|
| `'loudness_stationary'` | Total **Zwicker loudness** (sones) for stationary input. |
| `'loudness_timevarying'` | Time-varying loudness (array, sones vs. time). |
| `'sottek'` | Output dictionary from **`loudness_sottek()`** containing: `'loudnessTDep'`, `'loudnessPowAvg'`, `'specLoudness'`, `'specLoudnessPowAvg'`, `'bandCentreFreqs'`, `'timeOut'`. |
| `'sharpness'` | Sharpness values (**acum**) — scalar (stationary) or vector (time-varying). |
| `'fluctuation'` | Time-varying **fluctuation strength** (**vacil**). |
| `'fmod_fluc'` | Detected modulation frequency (Hz) from fluctuation analysis. |
| `'roughness'` | Time-varying **roughness** (**asper**). |
| `'tonality_sottek'` | Time-averaged **Sottek tonality index** (**tu_HMS**). |
| `'tonality_aures'` | Average **Aures tonality index** (**t.u.**). |
| `'tnr'` | Dictionary from `tone_to_noise_ratio()` with keys: `'tnr'`, `'freqs'`, `'is_prominent'`, `'times'`, `'peak_limits'`, `'freq_axis'`. |
| `'prominence_ratio'` | Dictionary from `prominence_ratio()` with keys: `'pr'`, `'freqs'`, `'is_prominent'`, `'times'`. |
| `'plot_input'` | (optional) Returns a plotted waveform or spectrum if `"plot_input"` is included in `params`. |

---

### Example Usage

```python
from psytools.analysis import psyacoustic_analysis

results = psyacoustic_analysis(
    input_path="example.wav",
    input_mode="audio",
    sound_field="free",
    time_varying=True,
    time_resolution="standard",
    show_plots=False,
    save_csv=True,
    params=[
        "plot_input",
        "loudness_zwicker",
        "sharpness",
        "fluctuation",
        "roughness",
        "tonality_aures",
        "tonality_sottek",
        "tnr",
        "pr"
    ],
    output_base="output/session_01"
)

# Access results
print(results["loudness_stationary"])
print(results["sharpness"])
print(results["tnr"]["tnr"])          # TNR values (dB)
print(results["prominence_ratio"]["pr"])
```

---

## `compare_metrics.py`

Visualizes and compares psychoacoustic results across different analysis runs.

## Main Function — `compare_metrics()`

| Parameter | Type | Description |
|------------|------|--------------|
| `output_dirs` | list[str] | Folders with psychoacoustic CSV results. |
| `font_size` | int, optional | Font size for plots (default 14). |
| `save_dir` | str, optional | Output for plots (default `output/comparisons`). |
| `metrics` | list[str], optional | List of metric filenames to compare (default: all). |
| `show` | bool, optional | Show plots interactively (default False). |

**Returns:**
Comparison plots saved as PNG/PDF.

**Features:**  
- Auto-detects metrics in each folder.  
- Aligns time axes for fair comparison.  
- Saves in `/png` and `/pdf` subfolders.  
- Extendable via `PLOT_FUNCTIONS` registry.

---

### Example Usage

```python
from psytools.compare_metrics import compare_metrics

output_dirs = [
    "output/Railway",
    "output/Airport",
    "output/electric_axle_drive"
]

compare_metrics(
    output_dirs=output_dirs,
    font_size=14,
    save_dir="output/comparisons",
    show=False
)
```

---

## `signal_processing.py`

Provides general‑purpose utilities for **audio loading**, **spectrum synthesis**, and **plotting**.

### Functions

| **Function** | **Description** |
|---------------|----------------|
| `synthesize_signal_from_spectrum()` | Creates synthetic test signals from sinusoidal spectral components. |
| `process_mic_data()` | Loads microphone CSV data, converts to Pa, and computes 1/3‑octave spectra. |
| `load_audio()` / `load_audio_librosa()` | Load audio signals from WAV or compressed formats, normalize. |
| `plot_input()` | Visualizes time signals or 1/3‑octave spectra. |
| `build_output_structure()` | Creates `/output/csv`, `/output/plots/pdf`, and `/output/plots/png` directories automatically. |
| `save_all_figures()` | Saves all open matplotlib figures as PNG and PDF. |

**Notes**

- Handles both time‑domain and frequency‑domain inputs.  
- Normalizes and calibrates audio before analysis.  
- Ensures consistent directory structure across analyses.  

---

## Validation

...

---

## License

Released under the **MIT License**.  
See the [LICENSE](LICENSE) file for details.
