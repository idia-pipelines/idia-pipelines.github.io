---
layout: default
title: Self-calibration in processMeerKAT
parent: processMeerKAT
nav_order: 8
---

# Self-calibration

*Note:*
1. *Our selfcal and science imaging workflow generally assumes a fixed image size, and may perform sub-optimally (e.g. sources missing from the CLEAN mask) if the image size changes at any point*
2. *Imaging at higher spectral resolution (i.e. with more frequency channels) requires higher computational power / runtime, and may require increasing the time limit of your imaging jobs. In general we recommend self-calbrating at 1k (1024 channels). If your raw data is at higher spectral resolution, consider first averaging in frequency with the `width` or `chanbin` config parameters (also see spectral-line pre-processing [here](/docs/processMeerKAT/advanced-usage#spectral-line-pre-processing))*

processMeerKAT implements the imaging & self-calibration loop using a
combination of [CASA](https://casadocs.readthedocs.io/en/stable/) for imaging
and calibration and [PyBDSF](https://www.astron.nl/citt/PyBDSF/) for
source-finding and masking.

Briefly, the self-calibration implementation in processMeerKAT works as
follows : The initial iteration of the loop performs a blind deconvolution of
the sky, which is followed by running PyBDSF to identify and create a mask
around the point sources detected in the image. The subsequent iterations use
the source mask generated from the previous iteration, allowing for deeper
imaging. Each iteration runs PyBDSF after the imaging stage in order to further
refine the mask.

By default, no gain calibration is performed after the first blind deconvolution
(although this is configurable), and the desired type of gain calibration can be
specified (phase-only, amplitude & phase, etc.) per loop.

The configuration options are intended to be as flexible as possible, with
sensible defaults. Some configuration options can be specified per loop, and
others only accept a single value for the entire self-calibration cycle. These
options are summarized below :

* **Single options -** These config variables accept only a single argument, and will be
  applied identically to all iterations of the self-cal cycle: `nloops`, `loop`,
  `discard_nloops`, `outlier_threshold`, `outlier_radius`
* **Gaincal options -** These config variables accept either a single argument, or a list of length `nloops`: `solint`, `calmode`, `gaintype`, `flag`
* **List of list options -** These config variables accept either a single argument, or a list of lists of length `nloop`: `imsize`

Any configuration options not listed above can contain either a single value
(applied to all loops) or a list of length `nloops + 1`. This is explained
further in the following section.

## Selfcal Config Options

The default self-cal section in the config file is shown below, and the details of each option are explained further down.

```
[selfcal]
nloops = 2                        # Number of clean + bdsf loops.
loop = 0                          # If nonzero, adds this number to nloops to name images or continue previous run
cell = '1.5arcsec'
robust = -0.5
imsize = [6144, 6144]
wprojplanes = 512
niter = [10000, 50000, 50000]
threshold = ['0.5mJy', 10, 10]    # After loop 0, S/N values if >= 1.0, otherwise Jy
nterms = 2                        # Number of taylor terms
gridder = 'wproject'
deconvolver = 'mtmfs'
calmode = ['','p']                # '' to skip solving (will also exclude mask for this loop), 'p' for phase-only and 'ap' for amplitude and phase
solint = ['','1min']
uvrange = ''                      # uv range cutoff for gaincal
flag = True                       # Flag residual column after selfcal?
gaintype = 'G'                    # Use 'T' for polarisation on linear feeds (e.g. MeerKAT)
discard_nloops = 0                # Discard this many selfcal solutions (e.g. from quick and dirty image) during subsequent loops (only considers when calmode !='')
outlier_threshold = 0.0           # S/N values if >= 1.0, otherwise Jy
outlier_radius = 0.0              # Radius in degrees for identifying outliers in RACS
```


* **nloops** : The number of self calibration loops to perform. This determines
  the number of times gain calibration will be performed. The total number of
  images generated will be `nloops+1`.
  Each loop in general comprises imaging (with `tclean`), calibration (with
  `gaincal`) and source finding/masking (with `PyBDSF`).  The final loop does
  not perform the `gaincal` stage since the gain solutions will not be applied
  for any further loops.

* **loop** : Set this to be a non-zero number if restarting from a previous
  selfcal run. Can be left to zero for all other cases.

* **cell**: The cell size of the pixels in the image plane, specified as a CASA
  compatible string (with units). This can be a single parameter, which will be
  applied identically to all loops, or specified as a list of strings if
  a different cell size per loop is desired. The length of the list must be
  `nloops+1` to account for the final image after all the self-calibration loops
  are complete.

* **robust**: The Briggs' robust parameter used during gridding. Specified
  identically to the `cell` parameter.

* **imsize**: The size of the image in pixels. The image dimensions must be
  specified in both dimensions, hence a list of [6144,6144] is treated as a
  single specification and will be applied identically to all loops. To specify
  a different image size per iteration, a list-of-lists syntax must be used,
  such as - `[[6144, 6144], [8192,8192]]`.

* **wprojplanes** : The number of W-projection planes to use (in case the
  `wproject` gridder is specified). Specified identically to the `cell`
  parameter.

* **niter**: The number of iterations to perform per loop. Imaging will continue
  until either the stopping threshold (described below) is reached, or the
  iteration limit is reached. Specified identically to the `cell` parameter.

* **threshold**: The stopping threshold per loop (a total of `nloop+1`). The
  stopping threshold for the initial loop (i.e. index 0 if setting `threshold` as a list) must be specified as a string with
  units, or a decimal < 1 in units of Jy. For subsequent loops, if a decimal is specified, values > 1 are considered as
  multipliers on the image RMS (as determined by the minimum value of the PyBDSF RMS map), while values < 1 remain considered as a threshold
  value in Jy.

* **nterms** : The number of Taylor terms to use for broadband imaging, assuming `deconvolver=mtmfs`.
  Specified identically to the `cell` parameter.

* **gridder** : The gridder to use for imaging, typically one of either
  `standard` or `wproject`. Specified identically to the `cell` parameter.

* **deconvolver** : The deconvolver to use during imaging. Use `clark` or `hogbom` when the fractional bandwidth (bandwidth divided by band centre) is < ~15%. Specified identically to the `cell` parameter.

* **calmode** : The calibration type per loop - one of either `'p'` (phase only)
  or `'ap'` (amplitude and phase). A blank string (`''`) specifies that no calibration will be
  performed for that loop. Must be of length `nloop`, and we recommend skipping gaincal during the first blind imaging step.

* **solint** : The solution interval per loop, specified similar to `calmode`. We recommend `'inf'` for loops with `calmode='ap'`, which will solve per scan. Specified identically to the `calmode` parameter.

* **uvrange**: The UV range limit to consider for gaincal. In the presence of
  strong RFI, it can be helpful to discard the shortest baselines while
  performing the gain calibration solve. This does not affect the imaging steps.
  Specified identically to the `calmode` parameter.

* **flag**: True/False - Determines whether flagging is performed on the
  residuals (data-model) between gain calibration solves. In cases where residual RFI remains, this can
  improve the image quality during subsequent loops. Setting it to `False` can
  provide a speedup to the imaging process. Specified identically to the `calmode` parameter.

* **gaintype**: The gaintype specified in the `gaincal` task. Typically one of
  either `G` or `T`. Use `T` if preserving the polarization in instruments with
  linear feeds is important (e.g., MeerKAT polarization processing). Specified identically to the `calmode` parameter.

* **discard_nloops**: Discard the first N loops. This is useful
  if the initial images are intentionally "quick-and-dirty" (e.g. large and poor resolution) and intended to generate more
  accurate source masks, for example. It will only count and discard the gain solutions
  from loops where `calmode` is not blank.

* **outlier_threshold**: The threshold value to identify an outlier. Considered
  to be an SNR threshold if > 1 and an absolute value in Jy if < 1. We recommend 0.5 as a good default, although using `''` or 0.0 (the default) disables outlier imaging.

* **outlier_radius**: The radius in degrees used to identify outliers. When unset (`''` or 0.0), the pipeline will calculate an appropriate radius based on the FWHM of the primary beam. The algorithm for determining and imaging outliers is explained in much more
    detail in the next section.

<!-- `rmsmap` and `mask` will be copied. -->

## Outlier Imaging

An optional part of the self-calibration routine within processMeerKAT is the outlier imaging. Within this mode, we utilise the [`outlierfile` parameter](https://casa.nrao.edu/Release3.3.0/docs/UserMan/UserMansu280.html) within CASA's `tclean` task, which allows the positions of strong off-axis outliers to be specified, along with a number of other imaging parameters, as outlined below. Within this routine, CASA phase-rotates to the specified position, enabling a small image to be created that is centred on the source, without the need for W-projection.

The motivation for including outlier imaging within processMeerKAT is to produce images equivalent in quality to much larger images (e.g. 10x10k pixels), with the strong sources far off axis being deconvolved, without the need for computational-heavy and long-running imaging over such a large area. However, an additional benefit is being able to specify precise positions of the outliers, centred within a pixel, with higher sampling of the beam over many smaller pixels, giving improved deconvolution that is not restricted by the pixel grid of the main image. For instance, without outliers, one may image 10,240 x 10,240 pixels with 1024 W-projection planes, deconvolving all the sources within the field of view, taking 10s of hours and lots of RAM to create an image. With outlier imaging, one could image 6144 x 6144 pixels with 512 W-projection planes within a few to several hours and with less RAM, and the few outliers above your threshold and beyond your main imaging area are deconvolved and included in your model for self-calibration.

<!-- It may even be possible to image sources within the main imaging area, but this mode isn't currently enabled, and needs to be verified. -->
 <!-- An additional benefit is the ability to increase taylor terms to more accurately model spectral curvature of bright outliers, although this 'mixing of taylor terms' between the main image and outliers needs to be verified and is therefore disabled by default. -->

In order to construct a local sky model, from which outliers can be selected according to the specified threshold, we make use of the [Rapid ASKAP Continuum Survey (RACS)](https://research.csiro.au/racs/home/survey/). Durning the `[-R --run]` step, we query the RACS catalogue for all sources (in and away from the galactic plane) within `outlier_radius` degrees (e.g. calculated at ~2 degrees) of the image's phase centre, written to `RACS_local.fits`. We then select the sources above your threshold, written as an outlierfile to `outliers.txt`. We then select sources from this list that are outside the inner 99% of your main imaging region (i.e. include as outliers any sources within the outer 1% and beyond) for each selfcal loop, written to `outliers_loop0.txt`, `outliers_loop1.txt`, etc. This is performed separately for each loop since the image size may change between loops. Each outlierfile is passed into the `tclean` call for that loop, including `selfcal_part2`, in which the image (including outliers) is used to predict to the `MODEL_DATA` column, used for subsequest self-calibration.

During each selfcal loop, the mask and position are updated based on the output from running PyBDSF. A mask is contructed corresponding to the island boundaries, the same as is done for the main image. The position is taken from the resulting PyBDSF catalogue, corresponding to the brightest Gaussian component fit to the source(s), which is often the only source found within the small image. If no source is found, or if the total integrated flux over the catalogue for that outlier is < 1 mJy, the outlier is discarded.

We recommend having fewer than ~10 outliers, since the overhead in run-time caused by imaging outliers outweighs any improvement in run-time caused by creating a smaller main image with fewer W-projection planes. When the `[-R --run]` step is performed, a warning will be displayed when the number of outliers exceeds 10 sources, in which case, a good compromose may be a larger image with fewer outliers, or a higher outlier threshold.

Outlier imaging is disabled by default, but can be enabled by setting a value for the `outlier_threshold` parameter in the `[selfcal]` section of your config file. Similar to the `threshold` parameter, this is considered a S/N value if it's greater than 1, or otherwise in Jy units. A good default is 0.5, which selects all sources above 0.5 Jy that are outside the your main image area. When a S/N value is used, the threshold corresponds to the value given by the integrated flux density divided by its uncertainty, taken from the RACS catalogue.

For each outlier within your `outlierfile`, by default, we set a 128x128 imsize, 1 arcsec cell sizes, and the standard gridder, leaving all other options identical to what is specified for the main image (some of which is set in your config file). However, as specified in the CASA docs, the following parameters are fully configurable within your `outlierfile`:

<!-- a number of taylor terms matched to your config (2 by default), -->

```
imagename, imsize, cell, phasecenter, startmodel, mask, specmode, nchan, start, width, nterms, reffreq, gridder, deconvolver, wprojplanes
```

If you wish to use non-default values, you can do this by [editing the selfcal_part2 script](/docs/processMeerKAT/advanced-usage#editing-pipeline-scripts).

Similarly, outlier imaging can be performed during the science imaging step of processMeerKAT, by passing an outlier file into the `outlierfile` parameter from the `[imaging]` section of your config file. If this section exists (enabled during the build step with `processMeerKAT.py -B -I`), the outlier file from your last self-calibration loop will be copied to the `outlierfile` parameter, in the same way that `rmsmap` and `mask` from your last selfcal loop are copied to this section, enabling S/N-based thresholding and masked deconvolution, respectively.

### Known Issues

During outlier imaging, the resolution of the output data products, from both the outliers and main image, including the PSF, is observed to be broader, from an apparent difference in weighting. This may be due to the local PSF being worse at the far-off-axis position of the outlier, where it is more severely affected by bandwidth and time smearing, compared to the centre of the image. At its extreme, this effect significantly worsens the quality of the output images, which we speculate is due to CASA attempting to match the PSF of the main image to that of the worst outlier.
<!-- as the weighting moves closer to natural weighting. -->
<!-- (high positive Briggs robust weightings) -->

CASA contains a bug such that `Stokes='IQUV'` cannot be specified during outlier imaging, but only `Stokes='I'` can be specified.
