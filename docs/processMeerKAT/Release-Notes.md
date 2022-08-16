---
layout: default
title: Release Notes
parent: processMeerKAT
nav_order: 2
---

# Version 2.0

This is the third release of the IDIA Pipelines `processMeerKAT` package, intended for use on the [ilifu](https://docs.ilifu.ac.za/#/) SLURM cluster. The software uses a parallelised implementation of [CASA 6](https://casadocs.readthedocs.io/en/stable/) to calibrate and image interferometric (imaging) data from the MeerKAT telescope.

The current release adds the following functionality:

* **Self-calibration and science imaging**: this allows for configuration of multiple self-calibration loops, with customisable parameters per loop, as well as an additional final imaging stage to generate science-ready images, which includes primary beam correction using [katbeam](https://github.com/ska-sa/katbeam).
* **Support for outlier fields (experimental)**: it is now possible to specify an outlier threshold to identify and image bright sources outside the main field of view, which improves the run-time of imaging, and can improve image fidelity in some cases. This is an experimental feature, so please report any bugs by logging an issue on our [GitHub repo](https://github.com/idia-astro/pipelines/issues/).
* **Changes in SPW selection syntax**: The SPW selection syntax has changed in the pipeline. SPWs are now specified as `'*:1280~1320MHz'` rather than `0:1280~1320MHz` (as was the case in the previous release). This allows for more flexible SPW selection in the configuration file. Please note that using the old syntax (such as `0:1280~1320MHz` or `1:1430~1450MHz` etc.) will result in malformed directory names and is discouraged.
* Bugfixes and improvements to polarisation calibration
* Support for loading modules on the ilifu SLURM cluster
* Updated to use CASA 6.5+, Python 3.8, OpenMPI 4.0.3, and Singularity 3.9.1

## Known Issues (minor)

Beyond the unresolved issues listed below, there are two minor CASA issues associated with outlier imaging, namely that `Stokes='IQUV'` cannot be specified during outlier imaging (but only `Stokes='I'`), and there is an observed loss in resolution whenever outlier imaging is invoked. For more information, see [here](/docs/processMeerKAT/self-calibration-in-processmeerkat#known-issues).

# Version 1.1

This is the second release of the IDIA Pipelines `processsMeerKAT` package, to be used on the ilifu SLURM cluster. The software uses a parallelised implementation of CASA to calibrate interferometric (imaging) data from the MeerKAT telescope.

The current release adds the following functionality:

* Spectral Window (SPW) splitting (see docs [here](/docs/processMeerKAT/config-files#spw-splitting)), where each separate SPW is processed independently and concurrently, providing a speed-up for large (TB) datasets, better polarisation calibration, and better flux scaling
* Quick-look continuum cube, across all SPWs
* Pre-processing during initial partition of MS, including pre-averaging of frequency channels, removal of cross-hand correlations up front for Stokes I processing, and removal of autocorrelations
* If running in full Stokes mode, `setjy` now includes the polarisation models for 3C286 and 3C138 if they are present in the data.
<!-- * Improved performance, including ... -->
* Improved default parameters, including a smaller RFI mask that removes the persistent RFI in the ranges `933~960, 1163~1299`, and `1524~1630` MHz
* Improved interaction with SLURM, including `exclude, dependencies, account` and `reservation` parameters, and graceful termination of pipeline after errors
<!-- * Improved calculation of antenna statistics, based on flags within raw data, used to select reference antenna and flag any bad antennas -->
* Uses CASA 5.6.2, Python 2.7, and Singularity 3.5.2.

## Known Issues

### Moderate:

* **Discontinuities in the Stokes Q and U spectra [Resolved in V2.0]**:
In the event that full Stokes calibration is requested (by passing the `--dopol` parameter during the build stage) we have noticed that the Stokes Q and U spectra of the calibrated data show discontinuities between the spectral windows (i.e. when `nspw` > 1 in your config). While the overall shape of the Q and U spectra seem to be right, the discontinuities will affect the inferences made during rotation measure synthesis. We are in the process of debugging this and will issue a patch once we have fixed it.

### Minor:

* **Resource allocation**:
   * There is an upper limit to the number of CPUs that can be utilised by MPICASA, which is given by the number of scans, plus the master MPIClient (nscans + 1). Relating this to the processing footprint is complex, since each task uses a different selection of those scans. We estimate the optimal number of MPI tasks as int(1.1*(nscans/2 + 1)), where nscans is the number of scans, typically 30-40. The motivation for this is that at most, if you have only target and phase calibrator, nscans / 2 will be selected by a task. So the master MPI client is added (+1) to this, and then 10%, in case some nodes time out.
    * For tasks reading only sub-MSs corresponding to other calibrators (e.g. bandpass), some CPUs will not be used
    * Similarly, we use a single memory value for all threadsafe tasks.

<!-- * **SLURM reports setjy jobs as FAILED**: Every time the `setjy` pipeline job is run, SLURM reports that this job failed, even though it has successfully completed. A quick glance at the last few lines of the logs will determine whether this step has legitimately failed or not. -->

* **Discontinuities in the phase solutions**: We have noticed a discontinuity in the phase of the bandpass solutions between spectral windows (i.e. when `nspw` > 1 in your config). However, this does not seem to have a significant effect on the calibration, as the spectrum of sources within the target field matches between data calibrated with `nspw` > 1, and data calibrated with `nspw=1`.

* **Slow plotting**: Generating plots of the calibrated visibilities is very time consuming, often running to a few hours. However, as this is the last step of the pipeline, the calibrated, split measurement sets and images should be ready for further analysis while the plots are being generated. The speed of plotting is limited by how quickly `plotms` can generate the plots.

* **Field IDs**: The pipeline does not currently support specifying multiple fields for anything other than the targets.

# Version 1.0

This is the first release of the IDIA Pipelines `processsMeerKAT` package, to be used on the ilifu SLURM cluster. The software uses a parallelized implementation of CASA to calibrate interferometric (imaging) data from the MeerKAT telescope.

The version 1.0 release includes the following functionality:

* The processMeerKAT.py script builds a config based on an input measurement set (MS).
* The pipeline currently only does cross-calibration, or a’priori (1GC) calibration.
* This includes parallelised flagging using FLAGDATA (tfcrop and rflag).
* Flux bootstrapping, gain and bandpass calibration.
* Full Stokes calibration.
* Quick-look imaging (i.e. without selfcal, w-projection, etc) of the calibrators and science targets.
* Diagnostic plots of the calibration tables and corrected data.
* Uses CASA 5.4.1, Python 2.7, and Singularity 2.6.1.

Please consult the documentation on [GitHub](https://idia-pipelines.github.io/) for more information. Three talks about version 1.0 of the pipeline, presented from the IDIA pipelines team during the [2019 South African MIGHTEE Early Science Workshop](https://www.idia.ac.za/mightee-uwc-2019/), can be found here: [1](/assets/Talk1.pdf), [2](/assets/Talk2.pdf), [3](/assets/Talk3.pdf).

## Known Issues

### Moderate:

* **Flux scale (Resolved in V1.1)**: Although the fluxes of the calibrated targets and calibrator sources are typically accurate to within a few percent, we find that there are certain datasets that result in a flux scale that is down by a factor of a few, particularly for short-track (e.g. 2 hour) observations (see [example use case](/docs/processMeerKAT/Example-Use-Cases#short-track-observations-and-fluxscale-issues)). We are in the process of tracking down the root cause of these issues, and expect to issue a fix soon. However, if you do happen to notice that the fluxes of one or more of the sources/targets are off (either higher/lower), please report it by creating a [Github issue](https://github.com/idia-astro/pipelines/issues).

* **Broadband polarisation (resolved in V1.1)**: CASA does not natively support solving broad-band polarisations, _i.e.,_ it is not sensitive to rotation measure (RM). The assumption is that the RM within a single spectral window (SPW) is constant, however MeerKAT has only a single SPW that spans the entire bandwidth. We have identified future workarounds (which is to split up the band into several SPWs), however presently the broadband polarisation models do contain systematic errors.

* **Calculation of antenna statistics (resolved in V1.1)**: The amplitude and RMS per antenna computed in `calc_refant.py` does not match what is found by CASA task `visstat`, and decreases as a function of antenna number. We expect to issue a fix soon. For now, we recommend users have `calcrefant=False` in their config files, which also disables antenna flagging (i.e. entirely flagging out bad antennas).

* **Resource allocation**:
   * We set the number of threads to half the number of scans + 1 (master) + 10%, so that for tasks reading the target or phasecal (since we partition by scans - i.e. the number of sub-MSs = the number of scans), the number of threads is approximately the number of sub-MSs being read. For tasks reading only sub-MSs corresponding to other calibrators (e.g. bandpass), many threads will not be used
   * Similarly, we use a single memory value for all threadsafe tasks, and hardcode 100 GB for single thread tasks

### Minor:

* **Empty rows in sub-MSs**: Some tasks might complain that no valid data were found in a sub-MS, but generally this seems to be a "harmless" error, and doesn’t seem to affect the progress of the calibration/pipeline.

* **Exit codes (resolved in V1.1)**
   * Some jobs fail in the queue that shouldn’t have, and others don’t fail when they should
   * Generally the pipeline continues even when dependencies legitimately fail

* **Slow plotting**: Generating plots of the calibrated visibilities is very time consuming, often running to a few hours. However, as this is the last step of the pipeline, the calibrated, split measurement sets and images should be ready for further analysis while the plots are being generated. The speed of plotting is limited by how quickly `plotms` can generate the plots.

* **Field IDs**: The pipeline does not currently support specifying multiple fields for anything other than the targets.
