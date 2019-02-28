---
layout: default
title: Calibration in ProcessMeerKAT
parent: processMeerKAT
nav_order: 2
---
processMeerKAT implements a CASA based wide-band full Stokes calibration
pipeline (in the linear basis). Broadly, the pipeline aims to "do the right
thing" and by keeping the steps as general as possible we believe that there
should be no need for fine tuning in order to obtain a well calibrated data set.
The pipeline is implemented as a series of SLURM `sbatch` scripts that in turn
call CASA scripts. The scripts are separated out to make optimal use of MPI, by
splitting out sections that can be run in parallel (via `mpicasa` and SLURM) and
sections that do not take advantage of parallel architectures.

The logical steps are:

**Input validation** : This script performs a few basic validity checks, on the
default config file, and on the input MS. the existence of the input MS, and the
data types of the inputs specified in the config file are all verified before
the pipeline continues to the next steps. If reference antenna calculation is
not requested, a simple check is performed to verify that the input reference
antenna exists in the MS. Otherwise, the following paragraph describes the
details of reference antenna calculation.

**Reference antenna calculation** : If the `calcrefant` parameter in the config
file is set to `True`, then this script is executed. The algorithm works by
calculating the median and standard deviation over all the visibility amplitudes
for a given antenna, and iterates over every antenna in the array. Any outlier
antennas, in the top 2 and bottom 5 percentile of this distribution are then
flagged. The reference antenna is selected to be the un-flagged antenna with the
smallest visibility rms.

**Data partition** : The input measurement set (MS) is partitioned into
a [multi-measurement set
(MMS)](https://casa.nrao.edu/casadocs/casa-5-1.2/uv-manipulation/data-partition)
using the CASA task `partition`. This task splits up the main MS into smaller
SUBMSs that are individual units of a larger logical MMS. The number of SUBMSs
created are equal to the number of scans in the input MS. Partitioning the data
in this manner allows for more efficient use of computation while using MPI,
since each SUBMS can be independently operated on by different MPI workers.

**Flagging (round 1)** : The first of two rounds of pre-calibration flagging. If
`badfreqranges` and `badants` are specified in the config file, they are
flagged. These lists are also allowed to be empty. Following that the data are
clipped at the level of 50 Jy to eliminate the strongest RFI and the `tfcrop`
algorithm is run independently on the primary and secondary calibrators and the
target(s).

**setjy** : The `setjy` task is run on the specified primary calibrators - this
step is run once each before the first and second rounds of calibration.

**Parallel hand calibration** : Standard delay, bandpass and gain calibration is
run on the data, in order to obtain better statistics for a second round of
flagging.

**Flagging (round 2)** : Similar to the first round, the `tfcrop` algorithm is
run independently on the primary and secondary calibrator and the target(s). The
thresholds are lower than the first round as the algorithm is now operating on
calibrated data.

**Cross hand calibration** : The cross hand calibration recipe broadly follows
the CASA guide for [polarisation calibration in the linear
basis](https://casa.nrao.edu/casadocs/casa-5-1.2/synthesis-calibration/instrumental-polarization-calibration).
The previous round of calibration is cleared, and a new set of bandpass, delay
and parallel hand gains are computed. Using the CASA helper routine
`qufromgain`, the Q and U values of the secondary calibrator are computed, and
are then used to solve for the frequency dependent leakages and XY calibration
(_i.e.,_ "Dflls" and "XYf" calibration in `polcal`). The fluxes are then
bootstrapped from the primary calibrator to the other fields specified in the
config file. If the same source is specified as both the flux and secondary
calibrator, no bootstrapping is performed as the time-dependent gain solutions
should be correctly scaled to the fluxes specified in `setjy`.

**Splitting out calibrated data** : Finally the calibrated data are averaged
down in time and frequency by the amount specified in the config file, and the
target(s) and calibrators are split out into separate MMSs for further
imaging/processing.

------

### Detailed description

What follows is a more detailed description of each of the steps described
above, where applicable.

**Flagging (round 1)** : If a list of bad frequency ranges and bad antennas is
specified, those are flagged. Further, any autocorrelations are also flagged
using `mode='manual'` and `autocorr=True` in the `flagdata` parameters.
Subsequently, `flagdata` is called on the calibrators and target sources with
conservative limits to clip out the worst RFI. It also makes a single call to
`tfcrop` to flag data at a 6 $\sigma$ limit. `tfcrop` in this case is preferred,
since the as yet uncalibrated bandpass shape should be taken care of by fitting
a piecewise polynomial across the band.

**setjy** : By default, the 'Perley-Butler 2010' flux scale is used, since it is
the only one which contains the popular southern calibrator PKS B1934-638. In
case the calibrator J0408-6545 is present in the data, it is preferred.
A broadband Stokes I model for J0408-6545 is used, via the `manual` mode of
`setjy`.
