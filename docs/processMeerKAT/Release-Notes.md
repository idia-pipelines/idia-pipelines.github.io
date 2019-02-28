---
layout: default
title: Release Notes
parent: processMeerKAT
nav_order: 2
---

# Version 1.0

This is the first release of the IDIA Pipelines’ processsMeerKAT package, to be used on the ILIFU SLURM cluster. The software uses a parallelized implementation of CASA to calibrate interferometric (imaging) data from the MeerKAT telescope.

The current release includes the following functionality:

* The processMeerKAT.py script builds a config based on an input measurement set (MS).
* The pipeline currently only does cross-calibration, or a’priori (1GC) calibration.
* This includes parallelised flagging using FLAGDATA (tfcrop and rflag).
* Flux bootstrapping, gain and bandpass calibration.
* Full stokes calibration.
* Quick imaging of the calibrators and science targets.
* Diagnostic plots of the calibration tables.

Please consult the documentation on [https://idia-pipelines.github.io/](https://idia-pipelines.github.io/) for more information.


Known Issues
==

* **Broadband polarization :** CASA does not natively support solving broad-band polarizations, _i.e.,_ it is not RM aware. The assumption is that the RM within a single spectral window (SPW) is constant, however the MeerKAT has only a single SPW that spans the entire bandwidth. We have identified workarounds (which is to split up the band into several SPWs), however presently the broadband polarization models do contain systematic errors.

* We recommend using the Test02 partition for the moment (set via the --partition option in processMeerKAT.py) due to ongoing issues in the Main SLURM partition.

* Generating plots of the calibrated visibilities is very time consuming, often running to a few hours, However as this is the last step of the pipeline, the calibrated, split measurement sets should be ready for further analysis (imaging etc) while the plots are being generated. The speed of plotting is limited by how quickly `plotms` can generate the plots.

* **Field IDs**: The pipeline does not currently support specifying multiple fields for anything other than the targets.

* **Threads**:
   * We set the number of threads to half the number of scans + 1 (master) + 10%, so that for tasks reading the target or phasecal (since we partition by scans - i.e. the number of sub-MSs = the number of scans), the number of threads is approx the number of sub-MSs being read. For tasks reading only sub-MSs corresponding to other calibrators (e.g. bandpass), many threads will not be used
   * Similarly, we use a single memory value for all threadsafe tasks, and hardcode 100 GB for single thread tasks

* **Empty rows in sub-MSs**: Some tasks complain that no valid data were found in a sub-MS, but generally this doesn’t seem to affect the progress of the calibration/pipeline.

* **Exit codes**
   * Issue where some jobs fail in the queue that shouldn’t have, and others don’t fail when they should
   * Generally the pipeline continues even when dependencies legitimately fail





