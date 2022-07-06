---
layout: default
title: Example Use Cases
parent: processMeerKAT
nav_order: 4
---

# Calibration

Our algorithmic approach toward calibration in the pipeline can be found [here](/docs/processMeerKAT/cross-calibration-in-processmeerkat).

## Stokes I Calibration (Continuum)

Stokes I calibration is achieved in `xx_yy_solve.py`, and includes standard delay, bandpass and gain calibration. Within this pipeline, this is done to obtain better statistics for a second round of flagging.

Stokes I calibration is the default mode of the pipeline, using the `xx_yy_solve.py` and `xx_yy_apply.py` scripts. Since the 2nd round of flagging is different to the 1st, and much more effective, these two scripts are called twice, in a second round that should not be skipped. Furthermore, for good calibration, new solutions should always be derived after flagging.

<!-- Krishna to revise -->
<!-- Lastly, it is recommended the 'xy_yx' scripts be run anyway, since linear feeds tend to have non-negligible coupling between the feeds, and so even to get a good Stokes I image, full Stokes calibration may be required.

Therefore, for Stokes I calibration, the 'xy_yx' scripts may be replaced with the 'xx_yy' scripts, in which case only a minimal speedup will be gained. Therefore, this use case is generally discouraged. One reason for this use case may be where issues arise with the calibration, and to simplify the processing to what is well understood. -->

## Full Stokes Calibration (Polarisation)

Full Stokes calibration can be activated with `[-P --dopol]`, which, in addition to the calibration listed [above](#stokes-i-calibration-continuum), includes calibration of instrumental leakage and X-Y phase. While XY phase calibration typically results in accurate full Stokes spectra, absolute polarization angle calibration is not performed and any systematic offsets in the XY phase calibration will reflect as errors in the Stokes parameters of the source.


### Short-Track Observations and flux scale Issues

**New in V2.0**: The full Stokes calibration scripts (`xy_yx_solve` and `xy_yx_apply`) no longer rely on the parallactic angle coverage of the secondary, but default to using a dedicated scan on the polarization calibrator to solve for the XY phase. The secondary calibrator is used only in the event that there is no polarization calibrator present in the observation. Therefore full Stokes calibration can be performed on short-track observations, as long as a dedicated polarization calibrator is observed.

**New in v1.1**: Only `xx_yy` scripts are used by default, and if the parallactic angle coverage is < 30 degrees, a warning is output recommending disabling polarisation calibration.

Replacing the 'xy_yx' scripts with the 'xx_yy' scripts is recommended for short-track observations (e.g. 2 hours), because the 'xy_yx' scripts assume that the phase calibrator has sufficient parallactic angle coverage to solve for the leakage and Stokes Q & U. We also recommend this for cases where there are issues with fluxscale, which often occurs when the 'xy_yx' scripts are run for short-track observations.

## Calibrating a Small Sub-Band (Spectral Line)

A small sub-band of frequencies can be selected and calibrated by specifying a range of frequencies with argument `spw` in your config file. e.g. `spw = '0:1350~1450MHz'` means `partition` will extract only frequencies between 1350-1450 MHz for calibration. For very small bandwidth of several MHz, it may not be necessary to use MMS and MPI (see section [MS only](#ms-only-single-thread-processing) below).

This mode is useful for calibration of spectral line data.

# Data Format

### Default: MS -> MMS

By default, the pipeline will convert the input measurement set (MS) into a multi-measurement set (MMS), partitioning the data by scan, such that the number of sub-MSs making up the MMS is equal to the number of scans. The input MS does not need to be copied to your working directory, since `partition.py` needs only to read the input data (e.g. from `/idia/projects/`).

At the end of the pipeline, `split.py` will split each of the field IDs specified in your config file into MMS format. This is the default behaviour, since the default value of `keepmms` in the config file is `True`. This also ensures future tasks make use of MPI (and multiple CPUs), so that your imaging runs more quickly.

This mode is encouraged for all users, as it will run the pipeline as efficiently as possible without a change in calibration strategy. However, users wishing to perform post-processing using other software packages may be required to write back into MS format.

### MS -> MMS -> MS

When setting `keepmms=False` in your config file, the pipeline will convert your data to MMS as usual, but during running `split`, each of the field IDs specified in your config file will be written in MS format. Although any subsequest calibration tasks (e.g. `applycal` and `flagdata` run during [selfcal](/docs/processMeerKAT/self-calibration-in-processmeerkat)) will not be able to utilise multiple tasks / CPUs, imaging tasks performed with `tclean` will still be able to utilise multiple tasks / CPUs, resulting in only a small difference in performance. This general mode of processing is encouraged when the user wishes to do their own imaging that requires an MS.

### MS only (single thread processing)

The pipeline can be run on a MS, where the user would have to request 1 node and 1 task per node. In this case, the user would set `createmms=False` in their config.

This mode isn't generally encouraged, but may be useful for small datasets (tens of GB - e.g. small bandwidth of several MHz, or short-track observations), when using MMS has little to no advantage, or when the number of scans is very large (hundreds), in which case splitting into multiple SPWs and multiple sub-MSs is too extreme (e.g. running across tens of nodes).

# Field IDs

### Default: Primary and Secondary Calibrator

A standard observation will have a primary calibrator (e.g. J1939-6342 or J0408-6545), which is both the bandpass and total flux calibrator, and a secondary calibrator, which is the phase calibrator (and is also used for polarisation calibration).

### Single calibrator and target

In rare cases, a primary calibrator may be close enough to the target(s) that only one calibrator is used throughout the observation. This use case is supported, but caution must be used to ensure the correct flux scale is derived.

### Multiple calibrator field IDs

Multiple field IDs can be specified by writing a comma-separated list to your config file. However, this use case is not supported, since tasks such as `setjy` cannot handle this. Only the `targetfields` and `extrafields` parameters from the `[fields]` section of your config file allow for multiple comma-separated fields within the pipeline.
