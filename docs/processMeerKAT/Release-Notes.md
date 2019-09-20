---
layout: default
title: Release Notes
parent: processMeerKAT
nav_order: 2
---

# Version 1.0

This is the first release of the IDIA Pipelines’ processsMeerKAT package, to be used on the ilifu SLURM cluster. The software uses a parallelized implementation of CASA to calibrate interferometric (imaging) data from the MeerKAT telescope.

The current release includes the following functionality:

* The processMeerKAT.py script builds a config based on an input measurement set (MS).
* The pipeline currently only does cross-calibration, or a’priori (1GC) calibration.
* This includes parallelised flagging using FLAGDATA (tfcrop and rflag).
* Flux bootstrapping, gain and bandpass calibration.
* Full Stokes calibration.
* Quick-look imaging (i.e. without selfcal, w-projection, etc) of the calibrators and science targets.
* Diagnostic plots of the calibration tables and corrected data.
* Uses CASA 5.4.1, Python 2.7, and Singularity 2.6.1.

Please consult the documentation on [GitHub](https://idia-pipelines.github.io/) for more information. Three talks about version 1.0 of the pipeline, presented from the IDIA pipelines team during the [2019 South African MIGHTEE Early Science Workshop](https://www.idia.ac.za/mightee-uwc-2019/), can be found here: [1](https://idia-pipelines.github.io/assets/Talk1.pdf), [2](https://idia-pipelines.github.io/assets/Talk2.pdf), [3](https://idia-pipelines.github.io/assets/Talk3.pdf).

Known Issues
==

* **Broadband polarization**: CASA does not natively support solving broad-band polarizations, _i.e.,_ it is not sensitive to rotation measure (RM). The assumption is that the RM within a single spectral window (SPW) is constant, however MeerKAT has only a single SPW that spans the entire bandwidth. We have identified future workarounds (which is to split up the band into several SPWs), however presently the broadband polarization models do contain systematic errors.

* **Slow plotting**: Generating plots of the calibrated visibilities is very time consuming, often running to a few hours. However, as this is the last step of the pipeline, the calibrated, split measurement sets and images should be ready for further analysis while the plots are being generated. The speed of plotting is limited by how quickly `plotms` can generate the plots.

* **Field IDs**: The pipeline does not currently support specifying multiple fields for anything other than the targets.

* **Flux scale**: Although the fluxes of the calibrated targets and calibrator sources are typically accurate to within a few percent, we find that there are certain datasets that result in a flux scale that is down by a factor of a few, particularly for short-track (e.g. 2 hour) observations (see [example use case](https://idia-pipelines.github.io/docs/processMeerKAT/Example-Use-Cases#short-track-observations-and-fluxscale-issues)). We are in the process of tracking down the root cause of these issues, and expect to issue a fix soon. However, if you do happen to notice that the fluxes of one or more of the sources/targets are off (either higher/lower), please report it by creating a [Github issue](https://github.com/idia-astro/pipelines/issues).

* **Calculation of antenna statistics**: The amplitude and RMS per antenna computed in `calc_refant.py` does not match what is found by CASA task `visstat`, and decreases as a function of antenna number. We expect to issue a fix soon. For now, we recommend users have `calcrefant=False` in their config files, which also disables antenna flagging (i.e. entirely flagging out bad antennas).

* **Resource allocation**:
   * We set the number of threads to half the number of scans + 1 (master) + 10%, so that for tasks reading the target or phasecal (since we partition by scans - i.e. the number of sub-MSs = the number of scans), the number of threads is approximately the number of sub-MSs being read. For tasks reading only sub-MSs corresponding to other calibrators (e.g. bandpass), many threads will not be used
   * Similarly, we use a single memory value for all threadsafe tasks, and hardcode 100 GB for single thread tasks

* **Empty rows in sub-MSs**: Some tasks might complain that no valid data were found in a sub-MS, but generally this seems to be a "harmless" error, and doesn’t seem to affect the progress of the calibration/pipeline.

* **Exit codes**
   * Some jobs fail in the queue that shouldn’t have, and others don’t fail when they should
   * Generally the pipeline continues even when dependencies legitimately fail





