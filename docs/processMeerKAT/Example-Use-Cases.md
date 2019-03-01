---
layout: default
title: Example Use Cases
parent: processMeerKAT
nav_order: 4
---

# Calibration

Our algorithmic approach toward calibration in the pipeline can be found in [[Calibration-in-processMeerKAT]].

#### Stokes I Calibration (Continuum)

Stokes I calibration is achieved in `xx_yy_solve.py`, and includes standard delay, bandpass and gain calibration. Within this pipeline, this is done to obtain better statistics for a second round of flagging.

To calibrate for only Stokes I, the `xy_yx_solve.py` and `xy_yx_apply.py` scripts may be replaced with `xx_yy_solve.py` and `xx_yy_apply.py`, or removed completely. However, since the 2nd round of flagging is different to the 1st, and much more effective, it should not be skipped. Furthermore, for good calibration, new solutions should always be derived after flagging. Lastly, it is recommended the 'xy_yx' scripts be run anyway, since linear feeds tend to have non-negligible coupling between the feeds, and so even to get a good Stokes I image, full Stokes calibration may be required.

Therefore, for Stokes I calibration, the 'xx_yy' scripts may be replaced with the 'xx_yy' scripts, in which case only a minimal speedup will be gained. Therefore, this use case is generally discouraged. One reason for this use case may be where issues arise with the calibration, and to simplify the processing to what is well understood.

#### Full Stokes Calibration (Polarisation)

Full Stokes calibration is the default mode of the pipeline, which, in addition to the calibration listed [above](#Stokes-I-Calibration-(Continuum)), includes calibration of instrumental leakage, X-Y phase, source polarisation, and Stokes Q and U from gain variations, using the CASA helper task `GainFromQU`. This is achieved by using the phase calibrator to solve as a function of parallactic angle.

Currently the pipeline does not solve for absolute polarisation angle, since the `CALIBRATE_POLARIZATION` intent is missing from MeerKAT datasets. Additionally, CASA currently has no functionality to solve for wide-band leakage and QU, but assumes a constant value across single spectral windows. Both of these issues will be addressed in future releases.

#### Calibrating a Small Sub-Band (Spectral Line)

A small sub-band of frequencies can be selected and calibrated by specifying a range of frequencies with argument `spw` in your config file. e.g. `spw = '0:1350~1450MHz'` means `partition` will extract only frequencies between 1350-1400 MHz for calibration. For very small bandwidth of several MHz, it may not be necessary to use MMS and MPI (see section [MS only](#MS-only) below).

This mode is useful for calibration of spectral line data.

# Data Format

#### Default: MS -> MMS

By default, the pipeline will convert the input measurement set (MS) into a multi-measurement set (MMS), partitioning the data by scan, such that the number of sub-MSs making up the MMS is equal to the number of scans. The input MS does not need to be copied to your working directory, since `partition.py` needs only to read the input data (e.g. from `/data/projects/`).

At the end of the pipeline, `split.py` will split each of the field IDs specified in your config file into MMS format. This is the default behaviour, since the default value of `keepmms` in the config file is `True`. This also ensures `tclean` makes use of MPI (and multiple CPUs) during `quick_tclean.py`, so that your imaging runs much more quickly.

This mode is encouraged for users who only want to have quicklook images, or who have written a `tclean` script that will make use of MPI, that they can insert at the end of the pipeline (see [[Using-the-pipeline#Inserting-your-own-scripts]].

#### MS -> MMS -> MS

When setting `keepmms=False` in your config file, the pipeline will convert your data to MMS as usual, but during running `split`, each of the field IDs specified in your config file will be written in MS format. By default, `quick_tclean.py` will still run, and will make use of multiple CPUs, but not MPI, meaning your imaging will run slower. This mode is encouraged when the user wishes to do their own imaging that requires an MS.

#### MS only (single thread processing)

The pipeline can be run on a MS, where the user would have to request 1 node and 1 task per node. In the case, the user would need to skip the `partition.py` script, by removing it from the `scripts` list in the config file, copy the MS do their working directory, and use this as the input dataset.

This mode in generally not encouraged, but may be useful for small datasets (tens of GB - e.g. small bandwidth of several MHz), when using MMS has little to no advantage. In such a case, the user may need to manually run `partition` (e.g. to select a `spw`), and set `createmms=False`. Alternatively, the user can run `split` or `mstransform` to select an `spw`.

# Field IDs

#### Default: Primary and Secondary Calibrator

A standard observation will have a primary calibrator (e.g. J1939-6342 or J0408-6545), which is both the bandpass and total flux calibrator, and a secondary calibrator, which is the phase calibrator (and is also used for polarisation calibration).

#### Single calibrator and target

In rare cases, a primary calibrator may be close enough to the target(s) that only one calibrator is used throughout the observation. This use case is supported, but caution must be used to ensure the correct flux scale.

#### Multiple calibrator field IDs

Multiple field IDs can be specified by writing a comma-separated list to your config file. However, this use case is not supported, since tasks such as `setjy` and `qufromgain` cannot handle this.

# Restarting or Using Part of the Pipeline

There are a few steps within the pipeline that only need to be run once for a given dataset. For datasets that have already been through the default pipeline, the reference antenna and list of bad antennas would have been calculated, and can be found in `logs/calc_refant-*.out`. These can be written your new config file, where you also set `calcrefant=False`.

More generally, users can run any selected parts of the pipeline by editing the `scripts` argument or passing scripts in via the command-line arguments (see [[Using-the-pipeline#Inserting-your-own-scripts]]).

The pipeline is not designed to run twice within the same directory, since CASA will complain the files already exist. The pipeline can be killed and re-run from the start at any point, but care should also be taken to ensure the data are not corrupted (e.g. during flagging or solving).