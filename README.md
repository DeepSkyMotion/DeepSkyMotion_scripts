<p align="center">
  <img src=".github/assets/dsm-banner.png" alt="Deep Sky Motion — PixInsight Scripts" width="100%">
</p>

<p align="center">
  <a href="#install"><img src="https://img.shields.io/badge/PixInsight-1.9.4%2B-1B2540?style=flat-square&labelColor=0B1121" alt="PixInsight 1.9.4+"></a>
  <img src="https://img.shields.io/badge/engine-PJSR%20%C2%B7%20V8-1B2540?style=flat-square&labelColor=0B1121" alt="PJSR V8">
  <img src="https://img.shields.io/badge/scripts-3-1B2540?style=flat-square&labelColor=0B1121" alt="3 scripts">
  <img src="https://img.shields.io/badge/platform-macOS%20%C2%B7%20Windows%20%C2%B7%20Linux-1B2540?style=flat-square&labelColor=0B1121" alt="macOS, Windows, Linux">
</p>

<p align="center">
  <b>Custom PixInsight scripts by Deep Sky Motion</b><br>
  <sub>Installed and updated straight from PixInsight — no manual copying.</sub>
</p>

---

## Install

Add this repository to PixInsight once. Every new script and update then arrives through PixInsight's own updater.

1. Open **Resources → Updates → Manage Repositories**.
2. Click **Add** and paste:

   ```
   https://raw.githubusercontent.com/DeepSkyMotion/DeepSkyMotion_scripts/main/
   ```

3. Run **Resources → Updates → Check for Updates**, apply, and restart PixInsight.

The scripts appear under **Script → DeepSkyMotion**.

## Scripts

<table>
  <tr>
    <td width="120" align="center" valign="top">
      <img src=".github/assets/synthetic-light-icon.png" width="96" alt="Synthetic Light icon">
    </td>
    <td valign="top">
      <h3>Synthetic Light</h3>
      <code>Script → DeepSkyMotion → Synthetic Light</code>
      <p>Builds a synthetic monochrome luminance from the R, G and B data of a linear stack, so every colour channel contributes to one deep luminance frame. Made for OSC/CFA data where no real L exists, and works just as well with mono R, G, B masters.</p>
    </td>
  </tr>
  <tr>
    <td width="120" align="center" valign="top">
      <img src=".github/assets/smart-frame-selection-icon.png" width="96" alt="Smart Frame Selection icon">
    </td>
    <td valign="top">
      <h3>Smart Frame Selection</h3>
      <code>Script → DeepSkyMotion → Smart Frame Selection</code>
      <p>Picks the best subframes for preprocessing without endless SubframeSelector tuning. Measures only the centre of each frame, so edge aberrations don't skew the result, debayers CFA frames to measure one or all colour channels at full resolution, and selects frames automatically, by a rule table, or in an interactive review window.</p>
    </td>
  </tr>
  <tr>
    <td width="120" align="center" valign="top">
      <img src=".github/assets/flat-strength-calibration-icon.png" width="96" alt="Flat Strength Calibration icon">
    </td>
    <td valign="top">
      <h3>Flat Strength Calibration</h3>
      <code>Script → DeepSkyMotion → Flat Strength Calibration</code>
      <p>Calibrates light frames with per-session flats, master darks and dark flats, without WBPP, and lets you set how strongly the flat is applied. On CFA/OSC data the strength can be set separately for the red, green and blue pixels of the Bayer mosaic, without debayering. A live before/after preview shows the result.</p>
    </td>
  </tr>
</table>

### Synthetic Light

**Input**
- one RGB image, **or**
- three registered mono masters (R, G, B) of the same size.

**Channel weights**

| Camera / source type | R | G | B |
|---|---|---|---|
| Colour CFA / OSC camera (Bayer 1:2:1) | 0.25 | 0.50 | 0.25 |
| Monochrome camera with RGB filters (Rec.709) | 0.2126 | 0.7152 | 0.0722 |
| Custom | your values | your values | your values |

Weights are always normalized to sum 1.

**Normalization**
- **None** — plain weighted average.
- **Match backgrounds** — subtracts each channel's median and adds back the weighted mean background.
- **Match backgrounds and scale** — also brings each channel's noise (MAD) to the G level before weighting.

**Output** — a new linear 32-bit float mono image (`SyntheticLight`, `SyntheticLight_1`, …) with the source FITS keywords and astrometric solution copied. Your source images are never modified.

**Preview** — a live, auto-stretched preview updates as you change weights and normalization.

> [!TIP]
> Feed Synthetic Light **linear** (unstretched) data. The script warns you if the input looks stretched.

### Smart Frame Selection

**Input** — light frames (FITS or XISF, mono or CFA/OSC) and an output folder. Subframe scale and camera gain are filled in from the FITS header when present.

**Measurement**
- Native SubframeSelector, run only on a centred region of interest (50 % of width and height by default), in small batches to stay light on RAM.
- **CFA frames** — measure normally, or debayer the central region (VNG, Bilinear or fast SuperPixel) and measure one channel (R, G or B) or **all three** at full resolution.

**Selection modes**
- **Auto** — robust session statistics (median ± MAD-sigma) with absolute safety caps; never rejects more than 40 % of the frames.
- **Rule set** — a table with one row per metric (FWHM, Eccentricity, PSF Signal Weight, SNR, Stars, Median, …). Each limit is an absolute value, a sigma distance from the median, or a percentage of the median, on the low side, the high side, or both.
- **Review** — a table and a graph of every frame; adjust the rule table live or click frames to keep or reject them by hand.

With all channels measured, a frame passes only if it passes in every channel.

**Output** — approved frames are copied to the output folder, either unchanged or as XISF with their measurements stored as keywords and properties (raw CFA data is kept lossless). A CSV report of every frame is written alongside. Your source files are never modified, and temporary files are always removed.

### Flat Strength Calibration

**Input** — one or more sessions, each with light frames, flat frames, a master dark and a master dark flat (FITS or XISF, mono or CFA/OSC). Shared master dark and dark flat files can be set once for all sessions; a session's own file overrides them.

**Flat strength** — a slider from −1 to +1 per session, with optional per-frame overrides:
- **0** — standard calibration, identical to a plain ImageCalibration run;
- **below 0** — weaker flat correction (−1 = no flat);
- **above 0** — stronger flat correction.

**Channel-wise (CFA)** — separate strengths for the R, G and B pixels of the Bayer mosaic. Each colour is calibrated with its own strength in a single ImageCalibration run; the output stays a CFA mosaic, no debayering.

**Preview** — auto-stretched before/after view of any light (linked or unlinked stretch, debayered or as a mosaic). In channel-wise mode you can switch between All, R, G and B; the colour views show a bin2 image made only from the pixels of that colour.

**Output** — 32-bit float XISF calibrated lights in `calibrated/<session>/`, master flats in `masters/`, and a `strength_log.csv` per session. One shared output folder for all sessions, or a separate folder per session. Your source files are never modified, and temporary files are removed.

## Updating

PixInsight checks this repository via **Resources → Updates → Check for Updates**. When a new release is available, apply it and restart PixInsight.

## Repository layout

| Path | Purpose |
|---|---|
| `updates.xri` | Repository index read by PixInsight |
| `*.zip` | Script packages; their contents mirror the PixInsight install folder (`src/scripts/DeepSkyMotion/<Script>/…`) |
| `.github/assets/` | Images for this page |

---

<p align="center">
  <sub>© 2026 Deep Sky Motion · All rights reserved</sub>
</p>
