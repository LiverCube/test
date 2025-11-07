PsyTools - Psychoacoustic Toolbox
====================

A modular Python toolbox for computing psychoacoustic metrics (Zwicker, Aures, Sottek, ECMA-418) with automatic data export and plotting.

Requirements
------------
- Python ≥ 3.8
- numpy, scipy, matplotlib
- librosa (optional, for .mp3 decoding)

Inputs & Outputs
----------------
Input formats:
- Waveform audio (.wav, .mp3, etc.) as NumPy array with sampling rate `fs`
- 1/3-octave spectra (`spectrum_db`, shape (28,) or (N, 28)), in dB SPL (Z-weighted)

ISO 532-1:2017 conventions:
- Stationary spectra: 28-band SPL values (25 Hz – 12.5 kHz)
- Time-varying spectra: sampled every 2 ms

Output:
All results are automatically exported as `.csv` files and plots into the `/output/` directory.

Quick Start
----------------
```python
from scipy.io import wavfile
import psytools as pt

fs, signal = wavfile.read(file_path)
L_total, L_spec, percentiles = pt.loudness_zwicker(signal=x, fs=fs, time_varying=True, plot=True)
print(L_total)
```

Modules Overview
----------------

analysis.py
~~~~~~~~~~~~~~~~~~~~
Main module for automated psychoacoustic analysis of audio or spectrum data. Runs all core metrics of the PsyAcoustic Toolbox and automatically saves results and plots in a structured output directory.

Main function:
- psyacoustic_analysis(): Runs the full psychoacoustic analysis pipeline.
  - Inputs:
      - input_path (str, optional): Path to input file (WAV, audio, or AVL CSV).
      - input_mode (str): Input type: 'wave', 'spectrum', 'AVL', or'audio'.
      - sound_field (str): Sound field compensation, 'free' or 'diffuse'.
      - time_varying (bool): Compute time-varying metrics if True.
      - time_resolution (str): 'standard' (2 ms) or 'high' (0.5 ms) for time-resolved calculations.
      - show_plots (bool): Display plots if True.
      - safe_csv (bool): Save CSV files automatically if True.
      - spectrum_db (np.ndarray, optional): 1/3-octave spectrum (only for 'spectrum' mode).
      - avl_path (str, optional): Path to AVL CSV file (only for 'AVL' mode).
      - params (list[str], optional): Metrics to compute. Options:
        "plot_input", "loudness_zwicker", "sharpness", "loudness_sottek", "fluctuation", "roughness",
        "tonality_sottek", "tonality_aures", "tnr", "pr".
        Default: all metrics allowed for the selected input mode.
      - output_base (str, optional): Base directory for all outputs. Default: relative to `input_path`.
- Outputs:
      - results (dict): Computed values for each metric:
        - 'loudness_stationary': Total Zwicker loudness (sones)
        - 'loudness_timevarying': Time-varying Zwicker loudness (if time_varying=True)
        - 'sottek': Output dictionary from sottek_hm.loudness_sottek()
        - 'sharpness': Total sharpness (acum)
        - 'fluctuation': Time-varying fluctuation strength (vacil)
        - 'fmod_fluc': Detected fluctuation modulation frequency (Hz)
        - 'roughness': Time-varying roughness (asper)
        - 'tonality_sottek': Average Sottek tonality (tu_HMS)
        - 'tonality_aures': Average Aures tonality index (t.u.)
        - 'tnr': Tone-to-Noise Ratio analysis results
        - 'prominence_ratio': Prominence Ratio analysis results

Features:
- Automatic handling of different input types:
  - 'wave' / 'audio': Direct signal loading
  - 'spectrum': Uses provided 1/3-octave spectrum
  - 'AVL': Synthesizes time-domain signal from AVL measurement CSVs
- Logging of all console output into run_YYYYMMDD_HHMMSS.txt
- Structured output directories:
  output/
  └─ <soundfile-name>/
     ├─ run_YYYYMMDD_HHMMSS.txt  …console log…
     ├─ csv/                     …all CSV files…
     └─ plots/
        ├─ pdf/                  …PDF figures…
        └─ png/                  …PNG figures…
- Computes both time-varying and stationary metrics for loudness, sharpness, fluctuation strength, roughness, and tonality.
- Norm-compliant with ISO 532-1, DIN 45692, ECMA-418-1/-2.
- Optional saving of all plots as PNG/PDF.

Helper utilities:
- ALLOWED_PARAMS: Defines allowed metrics per input mode.
- is_allowed(input_mode, param): Checks if a metric is allowed for the selected mode.
- signal_processing.build_output_structure(): Automatic creation of output folder structure.
- signal_processing.Tee for consolidated logging.


compare_metrics.py
~~~~~~~~~~~~~~~~~~~~
Module for visualizing and comparing psychoacoustic metrics across multiple analysis outputs.  

Main function:  
- compare_metrics(): Generates comparison plots for psychoacoustic metrics across multiple result folders.  
  - Inputs:  
      - output_dirs (list of str): List of folders containing psychoacoustic analysis CSV files.  
      - font_size (int, optional) Font size for the plots (default is 14).
      - save_dir (str, optional): Base folder for saving generated plots (default "output/comparisons").
      - metrics (list[str] or None, optional): Optional list of metric filenames (e.g., ["loudness_sottek.csv"]).
        If None, all available metrics defined in PLOT_FUNCTIONS will be used.
      - show (bool, optional): If True, show all generated plots interactively at the end. 
  - Behavior/Features:  
      - Automatically detects which metrics are available in the provided folders.  
      - Calls the corresponding plotting functions for each metric.  
      - Aligns time axes across folders for time-varying metrics.  
      - Saves plots as PNG and PDF in structured folders (PNG and PDF subfolders).  
      - Skips metrics that are missing in any folder and prints a warning.  
      - Facilitates easy extension by adding new plotting functions and registering them in `PLOT_FUNCTIONS`.  

Helper/utility functions:  
- plot_loudness_stationary_compare(): Compare stationary Zwicker loudness (total + specific) across multiple folders.  
- plot_loudness_time_varying_compare(): Compare time-varying Zwicker loudness including specific loudness spectrograms.  
- plot_sharpness_time_varying_compare(): Compare time-varying sharpness (DIN 45692).  
- plot_fluctuation_strength_compare(): Compare total and specific fluctuation strength over time.  
- plot_loudness_sottek_compare(): Compare ECMA-418-2 Sottek time-varying loudness.  
- plot_tonality_sottek_compare(): Compare ECMA-418-2 Sottek tonality over time.  
- plot_tonality_aures_compare(): Compare Aures/Terhardt tonality (time-dependent only).  
- single_color_cmap(): Generates white-to-color gradient colormaps for spectrogram visualizations.  


signal_processing.py
~~~~~~~~~~~~~~~~~~~~
General-purpose tools for audio loading, spectrum analysis and signal synthesis.

- synthesize_signal_from_spectrum(): Create signals from sinusoidal components.
- process_mic_data(): Load mic CSVs, convert to Pa, compute 1/3-octave spectra.
- load_audio(), load_audio_librosa(): Audio loading and normalization.
- plot_input(): Visualize time signals or spectra.
- build_output_structure(): Create output folders (/csv, /plots/pdf, /plots/png).
- save_all_figures(): Save all open matplotlib figures to PNG and PDF.


loudness.py
~~~~~~~~~~~
Implements Zwicker loudness (ISO 532-1) for stationary and time-varying inputs.

Main function:
- loudness_zwicker(): Computes total and specific loudness based on ISO 532-1.
  - Inputs:
      - signal (np.ndarray, optional):  Audio waveform, 1D array.
      - fs (int, optional):  Sampling frequency in Hz (required if signal is provided).
      - cal (float, optional):  Calibration factor (default sqrt(8), ~100 dB SPL full-scale 1 kHz sine).
      - spectrum_db (np.ndarray, optional):  1/3-octave SPL spectrum (28 bands).
      - output_in_phon (bool, optional):  Convert sones to phon if True.
      - remove (bool, optional):  Apply warm-up correction for stationary signals.
      - sound_field (str, optional): 'free' or 'diffuse' field correction.
      - time_varying (bool, optional):  Compute time-varying loudness.
      - time_resolution (str, optional):  'standard' (2 ms) or 'high' (0.5 ms).
      - plot (bool, optional):  Plot results.
      - font_size (int, optional) Font size for the plots (default is 14).
      - main_color (str, optional): Main color used for plotting total loudness (default is 'blue').
      - cmap (str, optional): Colormap used for plotting specific loudness (default is 'viridis').
      - save_csv (bool, optional):  Save results as CSV in output_dir.
      - output_dir (str, optional):  Folder for CSV output.
  - Outputs:
      - total_loudness:  Total loudness (scalar for stationary, time series if time_varying).
      - specific_loudness:  Specific loudness across Bark scale (240 bins).
      - percentiles:  Dictionary with Nmax and N5 for time-varying signals, else None.

Helper functions:
- smooth_signal_biquad(): Applies temporal smoothing using cascaded biquad filters (ISO 532-1, Sec. 6.3).
- apply_nonlinear_decay_iso(): Diode-like nonlinear temporal integration for time-varying core loudness (ISO 532-1, Annex A.10.2).
- iso_5321_frequency_spreading(): Maps core loudness to 0.1 Bark resolution and applies frequency spreading (Annex A.9).

References:
- ISO 532-1:2017 — Acoustics – Methods for calculating loudness – Part 1: Zwicker method
- E. Zwicker & H. Fastl, "Psychoacoustics: Facts and Models", 2017


sharpness.py
~~~~~~~~~~~~
Computes perceived acoustic sharpness (acum) using DIN 45692, Aures, or von Bismarck weighting models.

Main function:
- acoustic_sharpness(): Computes total or time-varying sharpness from an audio signal or precomputed specific loudness.
  - Inputs:
      - signal (np.ndarray, optional): Mono audio waveform. Required if specific_loudness is not provided.
      - sample_rate (int, optional): Sampling rate in Hz (default 48000 Hz).
      - weighting (str, optional): 'DIN 45692' (default), 'Aures', or 'von Bismarck'.
      - time_varying (bool, optional): Compute frame-based sharpness.
      - plot (bool, optional): Display time-varying sharpness plot.
      - font_size (int, optional) Font size for the plots (default is 14).
      - main_color (str, optional): Main color used for plotting total loudness (default is 'blue').
      - specific_loudness (np.ndarray, optional): Precomputed specific loudness array (frames x 240 Bark bands).
      - total_loudness (float or ndarray, optional): Total loudness.
      - output_dir (str, optional): Directory for CSV export (required if safe_csv=True).
      - safe_csv (bool, optional): Save results as CSV files.
      - **kwargs: Additional arguments passed to `loudness_zwicker()`.

  - Outputs:
      - sharpness: Sharpness in acum (scalar for stationary or array for time-varying).

Helper functions:
- get_weighting_g(): Computes the frequency weighting function g(z) according to the selected model.
- get_normalization_constant(): Returns the normalization factor to match the reference sharpness for a 1 kHz tone at 100 dB SPL.

References:
- DIN 45692:2009 — Acoustics — Determination of perceived sound quality — Sharpness
- Fastl, H., & Zwicker, E. (2007). Psychoacoustics: Facts and Models, 3rd ed.


fluctuation.py
~~~~~~~~~~~~~~
Computes acoustic fluctuation strength (vacil) based on the Zwicker model (ISO 532-1).

Main function:
- acoustic_fluctuation(): Computes total and specific fluctuation strength from an audio signal or precomputed specific loudness.
  - Inputs:
      - signal (np.ndarray, optional):  Mono audio waveform. Required if specific_loudness is not provided.
      - fs (int, optional):  Sampling rate of input signal (default 48000 Hz).
      - fmod (float or str, optional):  Modulation frequency in Hz or 'auto-detect'.
      - specific_loudness (np.ndarray, optional):  Precomputed specific loudness array (time x bark bands).
      - spec_fs (int, optional):  Sampling rate of specific loudness (default 500 Hz).
      - plot (bool, optional):  Plot total and specific fluctuation strength.
      - plot (bool, optional):  Plot results.
      - font_size (int, optional) Font size for the plots (default is 14).
      - main_color (str, optional): Main color used for plotting total loudness (default is 'blue').
      - cmap (str, optional): Colormap used for plotting specific loudness (default is 'viridis').
      - save_csv (bool, optional):  Save results as CSV files.
      - output_dir (str, optional):  Directory for CSV export (required if save_csv=True).
      - **kwargs: Additional arguments passed to `loudness_zwicker()`.

  - Outputs:
      - fluctuation: Time series of total fluctuation strength (vacil).
      - specific:  Specific fluctuation strength per Bark band (vacil/Bark).
      - fmod:  Modulation frequency used or detected (Hz).

Helper functions:
- compute_fluctuation(): Calculates total and specific fluctuation strength (vacil) from specific loudness based on modulation detection and envelope processing.

References:
- ISO 532-1:2017 — Acoustics – Methods for calculating loudness – Part 1: Zwicker method
- Fastl, H., & Zwicker, E. (2007). Psychoacoustics: Facts and Models, 3rd ed.


roughness.py
~~~~~~~~~~~~
Computes acoustic roughness (asper) based on the Zwicker model (ISO 532-1).

Main function:
- acoustic_roughness(): Computes total and specific roughness from an audio signal or precomputed specific loudness.
  - Inputs:
      - signal (np.ndarray, optional):  Mono audio waveform. Required if specific_loudness is not provided.
      - fs (int, optional): Sampling rate of input signal (default 48000 Hz).
      - fmod (float or str, optional): Modulation frequency in Hz or 'auto-detect'.
      - specific_loudness (np.ndarray, optional): Precomputed specific loudness array (time x bark bands).
      - plot (bool, optional): Plot total and specific roughness.
      - font_size (int, optional) Font size for the plots (default is 14).
      - main_color (str, optional): Main color used for plotting total loudness (default is 'blue').
      - cmap (str, optional): Colormap used for plotting specific loudness (default is 'viridis').
      - save_csv (bool, optional): Save results as CSV files.
      - output_dir (str, optional): Directory for CSV export (required if save_csv=True).
      - **kwargs: Additional arguments passed to `loudness_zwicker()`.

  - Outputs:
      - asper: Time-varying total roughness (asper).
      - spec: Specific roughness per Bark band (asper/Bark).
      - fmod: Modulation frequency used or detected (Hz).

Helper functions:
- compute_roughness(): Calculates total and specific roughness (asper) from specific loudness using bandpass filtering, envelope extraction, modulation compensation, and Bark-band aggregation.

References:
- ISO 532-1:2017 — Acoustics – Methods for calculating loudness – Part 1: Zwicker method
- Fastl, H., & Zwicker, E. (2007). Psychoacoustics: Facts and Models, 3rd ed.


tonality.py
~~~~~~~~~~~~
Implements tonal perception metrics based on ECMA-418-1 and the Aures Tonality Index.

Main Functions:
- tonality_aures(): Computes the Aures tonality index for a monophonic signal, either time-varying or stationary.
 -Inputs:
      - signal_in (np.ndarray): Input monophonic audio signal.
      - sample_rate (int): Sampling frequency in Hz.
      - time_varying (bool, optional): If True, compute frame-wise tonality (default True).
      - plot (bool, optional): If True, visualize tonality results (default False).
      - font_size (int, optional) Font size for the plots (default is 14).
      - main_color (str, optional): Main color used for plotting total loudness (default is 'blue').
      - safe_csv (bool, optional): If True, export results to CSV (default False).
      - output_dir (str, optional): Directory for CSV export (required if safe_csv=True).
-Output:
  -tonalities (np.ndarray or float): Tonality index per frame (time-varying) or scalar (stationary).
  -times (np.ndarray or None): Time vector corresponding to tonalities (None for stationary).

- tone_to_noise_ratio(): Computes Tone-to-Noise Ratio (TNR) according to ECMA-418-1.
 -Inputs:
   -signal_in (np.ndarray): Input audio signal (1D, mono).
   -sample_rate (int): Sampling rate in Hz.
   -frame_size (int, optional): FFT frame size (default 16384).
   -hop_size (int, optional): Hop size between frames (default 8192, 50% overlap).
   -time_varying (bool, optional): Compute time-varying TNR if True (default False).
   -plot (bool, optional): Show plots of spectrum and TNR (default False).
   - font_size (int, optional) Font size for the plots (default is 14).
   - main_color (str, optional): Main color used for plotting total loudness (default is 'blue').
   - cmap (str, optional): Colormap used for plotting specific loudness (default is 'viridis').
   -prominence_only (bool, optional): Return only prominent tones (default False).
   -safe_csv (bool, optional): Export CSV files if True (default False).
   -output_dir (str, optional): Directory to save CSV files (required if safe_csv=True).
-Output:
   - results (dict): Dictionary containing:
       -'tnr' : List/array of TNR values (per tone or per frame if time-varying).
       -'frequencies' : Array of detected tone frequencies (Hz).
       -'prominent' : Array of boolean flags for prominent tones.
       -'times' : Time stamps (s) for frames (None if stationary).
       -'peak_limits' : Frequency bin limits of detected peaks.
       -'f_axis' : Frequency axis used for PSD calculation.

- prominence_ratio(): Computes Prominence Ratio (PR) per ECMA-418-1.
  -Inputs:
   -signal_in (np.ndarray): Input audio signal (mono, 1D array).
   -sample_rate (int): Sampling rate in Hz.
   -frame_size (int, optional): FFT frame size (default 16384).
   -hop_size (int, optional): Hop size between frames (default 8192).
   -time_varying (bool, optional): Compute time-resolved PR if True (default False).
   -plot (bool, optional): Show plots of PR and spectrum (default False).
   - font_size (int, optional) Font size for the plots (default is 14).
   - main_color (str, optional): Main color used for plotting total loudness (default is 'blue').
   - cmap (str, optional): Colormap used for plotting specific loudness (default is 'viridis').
   -window_type (str, optional): Window function for FFT (default 'hann').
   -frequency_range (tuple, optional): Frequency range for analysis in Hz (default [89.1, 11200]).
   -output_dir (str, optional): Directory for saving CSV files (required if safe_csv=True).
   -safe_csv (bool, optional): Export PR data to CSV if True (default False).
 -Output:
   -results (dict): Dictionary containing:
       -'PR' : PR values (vector if stationary, matrix if time-varying).
       -'freqs' : Frequency bins corresponding to PR values.
       -'is_prominent' : Boolean mask of prominent tones.
       -'times' : Time stamps for frames (only for time-varying analysis).

Helper Functions:
- critical_band_limits(fpk)
- octave_24th_bands(fmin, fmax)
- smooth_in_db_domain(f, psd_db, octave24b, cf)
- Gaussian smoothing, peak detection, audibility check, tonal width estimation routines
- Plotting functions for TNR and PR
- Internal calculations for tonal weighting, loudness masking, and Aures tonal index aggregation

References:
-Aures, W. (1985). 'Berechnungsverfahren für den sensorischen Wohlklang'. Acustica 59
-Terhardt, E. et al. (1982). 'Algorithm for extraction of pitch and pitch salience'. JASA
-ECMA-418-2:2025 - prominent discrete tones

sottek_hm.py
~~~~~~~~~~~~
Implements ECMA-418-2 (Sottek model) for loudness and tonality.

Main Functions:
-loudness_sottek(): Computes total and specific loudness according to ECMA-418-2:2025.
 - Inputs:
   - insig (np.ndarray): Mono input signal (time x channels if multi-channel, mono preferred).
   - fs (int or float): Sampling frequency in Hz.
   - plot (bool, optional): If True, plots total and specific loudness.
   - font_size (int, optional) Font size for the plots (default is 14).
   - main_color (str, optional): Main color used for plotting total loudness (default is 'blue').
   - cmap (str, optional): Colormap used for plotting specific loudness (default is 'viridis').
   - fieldtype (str, optional): Sound field type, either 'free' or 'diffuse' (default 'free').
   - time_skip (float, optional): Initial skip duration in seconds for averaging (default 0.304 s).
   - calibration_factor (float, optional): Scaling factor applied to the input signal prior to processing.
   - save_csv (bool, optional): If True, automatically exports results as CSV files.
   - output_dir (str, optional): Directory for saving CSV files (required if save_csv=True).
   - **kwargs: Additional optional arguments for internal processing (e.g., fade-in, block length).
 -Output:
   - OUT (dict): Dictionary containing:
       - 'specLoudness' (np.ndarray): Specific loudness vs. time and Bark band (sone/Bark).
       - 'loudnessTDep' (np.ndarray): Time-dependent total loudness (sone).
       - 'loudnessPowAvg' (float): Power-averaged overall loudness (sone).
       - 'specLoudnessPowAvg' (np.ndarray): Power-averaged specific loudness per Bark band (sone/Bark).
       - 'bandCentreFreqs' (np.ndarray): Bark band center frequencies (Hz).
       - 'timeOut' (np.ndarray): Output time vector (s).
       - 'soundField' (str): Sound field type used ('free' or 'diffuse').

-tonality_sottek(): Computes tonality metrics according to ECMA-418-2:2025 using the Sottek hearing model.
 - Inputs:
   - insig (np.ndarray): Input time-domain signal [samples x channels] in Pascal (mono or stereo).
   - fs (int): Sampling frequency in Hz. Will be resampled to 48 kHz if different.
   - plot (bool, optional): If True, plots tonality results.
   - font_size (int, optional) Font size for the plots (default is 14).
   - main_color (str, optional): Main color used for plotting total loudness (default is 'blue').
   - cmap (str, optional): Colormap used for plotting specific loudness (default is 'viridis').
   - fieldtype (str, optional): Either 'free' or 'diffuse'. Affects outer/middle ear filtering.
   - time_skip (float, optional): Initial time in seconds to skip for time-averaging (default 0.304 s).
   - calibration_factor (float, optional): Scaling factor applied to the input signal prior to processing.
   - safe_csv (bool, optional): If True, CSV files will be saved to output_dir.
   - output_dir (str, optional): Directory to save CSV files.
   - **kwargs: Additional optional arguments for internal processing.
 -Output:
     - OUT (dict): Dictionary containing:
       - 'specTonality' (np.ndarray): Time-dependent specific tonality per Bark band.
       - 'specTonalityAvg' (np.ndarray): Time-averaged specific tonality.
       - 'specTonalityFreqs' (np.ndarray): Frequency-resolved specific tonality.
       - 'specTonalityAvgFreqs' (np.ndarray): Power-averaged frequency-resolved specific tonality.
       - 'specTonalLoudness' (np.ndarray): Tonal loudness per band.
       - 'specNoiseLoudness' (np.ndarray): Noise loudness per band.
       - 'tonalityTDep' (np.ndarray): Time-dependent overall tonality.
       - 'tonalityAvg' (float): Time-averaged overall tonality.
       - 'tonalityTDepFreqs' (np.ndarray): Frequency-resolved time-dependent tonality.
       - 'bandCentreFreqs' (np.ndarray): Bark band center frequencies (Hz).
       - 'timeOut' (np.ndarray): Output time vector (s).
       - 'timeInsig' (np.ndarray): Input time vector (s).
       - 'soundField' (str): Sound field type used ('free' or 'diffuse').

Helper Functions:
-shm_resample(): Resamples a signal to 48 kHz using polyphase resampling.
-shm_preproc(): Applies fade-in and zero-padding according to ECMA-418-2.
-shm_out_mid_ear_filter(): Implements outer & middle ear filter for 'free-frontal' or 'diffuse' sound fields.
-shm_auditory_filt_bank(): Computes 53 half-Bark bandpass filtered output of an input signal.
-shm_signal_segment(): Segments a signal into overlapped blocks, matching ECMA-418-2 behavior.

References:
-ECMA-418-2:2025 - methods for describing human perception based on the Sottek Hearing Model


modulation.py
~~~~~~~~~~~~~
Helper functions for modulation-based psychoacoustic analysis, used in roughness and fluctuation computation.

Helper Functions:
- sone2phon(): Convert specific loudness from sone to phon scale.
- assemble_bark_bands(): Aggregate 240 frequency channels into 24 or 47 Bark bands.
- filter_dc_and_transient(): Remove DC offset and suppress initial transients.
- design_octave_filter(): Design a Butterworth bandpass filter centered at a given frequency.
- apply_octave_filter(): Apply an octave-based bandpass filter to a signal.
- envelope_filter(): Extract the positive envelope using zero-phase low-pass filtering.
- detect_fmod(): Estimate dominant modulation frequency from Bark-band loudness.

References:
-----------
- ISO 532-1:2017 Acoustics - Methods for calculating loudness - Zwicker method
- Fastl, H., & Zwicker, E. (2007). Psychoacoustics: Facts and Models, 3rd ed.
