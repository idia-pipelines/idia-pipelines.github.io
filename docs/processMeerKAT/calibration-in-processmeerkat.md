---
layout: default
title: Calibration in ProcessMeerKAT
parent: processMeerKAT
nav_order: 2
---
processMeerKAT implements a CASA based wide-band full Stokes calibration pipeline (in the linear
basis). Broadly, the pipeline aims to "do the right thing" and by keeping the steps as general as
possible we believe that there should be no need for fine tuning in order to obtain a well
calibrated data set. The pipeline is implemented as a series of SLURM `sbatch` scripts that in turn
call CASA scripts. The scripts are separated out to make optimal use of MPI, by splitting out
sections that can be run in parallel (via `mpicasa` and SLURM) and sections that do not take
advantage of parallel architectures.

The logical steps are :

**Input validation** : This script does a few basic data validity checks, that is to verify that the
input MS exists, and whether the reference antenna should be calculated internally. If reference
antenna calculation is specified, the script iteratively calls the `visstat` task on each antenna,
and checks for the antenna with the smallest standard deviation value. During this process, antennas
which are outliers to the distribution are identified and listed in the config file for future
flagging. If reference antenna calculation is not requested, a simple check is performed to verify
that the input reference antenna exists in the MS.

**Data partition** : The input measurement set (MS) is partitioned into a [multi-measurement set
(MMS)](https://casa.nrao.edu/casadocs/casa-5-1.2/uv-manipulation/data-partition) using the CASA task
`partition`. This task splits up the main MS into smaller SUBMSs that are individual units of a
larger logical MMS. The number of SUBMSs created are equal to the number of scans in the input MS.
Partitioning the data in this manner allows for more efficient use of computation while using MPI,
since each SUBMS can be independently operated on by different MPI workers.

**Flagging (round 1)** : The first of two rounds of pre-calibration flagging. If `badfreqranges` and
`badants` are specified in the config file, they are flagged. These lists are also allowed to be
empty. Following that the data are clipped at the level of 50 Jy to eliminate the strongest RFI and
the `tfcrop` algorithm is run independently on the primary and secondary calibrators and the
target(s).

**Flux scaling** : The `setjy` task is run on the specified primary calibrators - this step is run
once each before the first and second rounds of calibration.

**Parallel hand calibration** : Standard delay, bandpass and gain calibration is run on the data, in
order to obtain better statistics for a second round of flagging.

**Flagging (round 2)** : Similar to the first round, the `tfcrop` algorithm is run independently on
the primary and secondary calibrator and the target(s). The thresholds are lower than the first
round as the algorithm is now operating on calibrated data.

**Cross hand calibration** : The cross hand calibration recipe broadly follows the CASA guide for
[polarisation calibration in the linear
basis](https://casa.nrao.edu/casadocs/casa-5-1.2/synthesis-calibration/instrumental-polarization-calibration).
The previous round of calibration is cleared, and a new set of bandpass, delay and parallel hand
gains are computed. Using the CASA helper routine `qufromgain`, the Q and U values of the secondary
calibrator are computed, and are then used to solve for the frequency dependent leakages and XY
calibration (_i.e.,_ "Dflls" and "XYf" calibration in `polcal`).

**Splitting out calibrated data** : Finally the calibrated data are averaged down in time and
frequency by the amount specified in the config file, and the target(s) and secondary calibrator are
split out into separate MMSs for further imaging/processing.
