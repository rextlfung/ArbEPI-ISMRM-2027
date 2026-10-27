# Dynamic sampling for fast 3D-EPI fMRI: finger-tapping session

Sequences for four whole-brain 3D-EPI finger-tapping runs at R of about 10 that differ in their (ky, kz) sampling pattern.
All sequences are vendor-neutral [Pulseq](https://pulseq.github.io/) 1.5 files.

## Acquisitions

| Name | File prefix | Resolution | Matrix | Volume TR | Pattern |
|---|---|---|---|---|---|
| (a) Static CAIPI | `static_caipi_2.4mm` | 2.4 mm | 90x90x60 | 0.5004 s | one fixed CAIPI mask |
| (b) Dynamic CAIPI with random shifts | `dynamic_caipi_2.4mm` | 2.4 mm | 90x90x60 | 0.5004 s | ten shifted CAIPI masks, randomly permuted every ten frames |
| (c) Dynamic Poisson-Disc (PD) with fully sampled center | `dynamic_pd_2.4mm` | 2.4 mm | 90x90x60 | 0.5004 s | new variable-density PD mask every frame |
| (d) Dynamic PD, 1.6 mm, 0.8 s TR | `dynamic_pd_1.6mm` | 1.6 mm | 136x136x54 | 0.8 s | new PD mask every frame (R = 10.2) |

Common to all runs: TE 30 ms, echo train length 60, laminar echo-train ordering, water-selective excitation.
Each run has a 10 s instruction period (9.6 s at 1.6 mm), then eight cycles of 20 s finger tapping and 20 s rest.
The instruction frames are discarded.

## Contents

### `sequences/`

Name pattern: `<acquisition>_<scan>.seq`.

| Scan suffix | Purpose |
|---|---|
| `task_epi` | The 3D-EPI task run (330 s). |
| `epi_calibration` | Short EPI scan for the readout delay and odd/even phase. |
| `noise_scan` | Receiver noise only, for noise prewhitening. |

`gre_2.4mm.seq` and `gre_1.6mm.seq` are the dual-echo T1-weighted 3D GRE scans on each EPI grid.
Their data give the sensitivity maps, the B0 map and the anatomical underlay.
The 1.6 mm run was acquired a second time at the end of the session to check for drift.
The repeat is the same sequence, so it has no separate file.

### `sequence_design/`

One set per acquisition, `<acquisition>_<plot>.png`:

| Plot | Shows |
|---|---|
| `mask` | The (ky, kz) sampling mask of the first frame. |
| `trajectory` | The k-space trajectory of one frame. |
| `one_tr` | The gradient waveforms of one shot. |
| `psf` | The point-spread function of the first frame. |

## Reconstruction and analysis

The reconstruction code is [ArbEPI-python](https://github.com/rextlfung/ArbEPI-python).
Brief methods:

- Sensitivity maps: ESPIRiT from the GRE, with 32 coils compressed to 17.
- B0 map: ROMEO-initialized, estimated from the dual-echo GRE.
- Reconstruction: B0-informed SENSE with 24 time segments at 2.4 mm and 16 at 1.6 mm.
  - L1-wavelet + TV: lambda 0.005 each, 100 iterations, frame by frame.
  - LLR: non-overlapping 6^3 (2.4 mm) or 9^3 (1.6 mm) patches with cycle spinning, lambda = sqrt(R), 100 iterations, all frames jointly.
- Post-processing: rigid-body motion correction and a BET brain mask.
- Activation: a GLM with an HRF-convolved task regressor, drift terms and AR(1) prewhitening.
