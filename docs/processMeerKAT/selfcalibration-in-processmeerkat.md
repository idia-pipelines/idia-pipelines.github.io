---
layout: default
title: Self-calibration in processMeerKAT
parent: processMeerKAT
nav_order: 8
-

processMeerKAT implements the imaging & self-calibration loop using a
combination of [CASA](https://casadocs.readthedocs.io/en/stable/) for imaging
and calibration and [pyBDSF](https://www.astron.nl/citt/pybdsf/) for
source-finding and masking.

Briefly, the self-calibration implementation in processMeerKAT works as
follows : The initial iteration of the loop performs a blind deconvolution of
the sky, which is followed by running pyBDSF to identify and create a mask
around the point sources detected in the image. The subsequent iterations use
the source mask generated from the previous iteration, allowing for deeper
imaging. Each iteration runs pyBDSF after the imaging stage in order to further
refine the mask.

By default, no gain calibration is performed after the first blind deconvolution
(although this is configurable), and the desired type of gain calibration can be
specified (phase-only, amplitude & phase etc.) per loop.

The configuration options are intended to be as flexible as possible, with
sensible defaults. Some configuration options can be specified per loop, and
others only accept a single value for the entire self-calibration cycle. These
options are summarized below :

* **Single options:** These config variables accept only a single argument, and will be
  applied identically to all iterations of the self-cal cycle; `nloops`, `loop`,
  `discard_nloops`, `outlier_threshold`
* **Gaincal options:** These config variables accept either a single argument, or a list of length `nloops`; `solint`, `calmode`, `gaintype`, `flag`
* **List of list options:** These config variables accept either a single argument, or a list of lists of length `nloop`; `imsize`

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
```


* **nloops** : The number of self calibration loops to perform. This determines
  the number of times gain calibration will be performed. The total number of
  images generated will be `nloops+1`.
  Each loop in general comprises imaging (with `tclean`), calibration (with
  `gaincal`) and source finding/masking (with `pyBDSF`).  The final loop does
  not perform the `gaincal` stage since the gain solutions will not be applied
  for any further loops.
  
* **loop** : Set this to be a non-zero number if restarting from a previous
  selfcal run. Can be left to zero for all other cases.
  
* **cell**: The cell size of the pixels in the image plane, specified as a CASA
  compatible string (with units). This can be a single parameter, which will be
  applied identically to all loops, or specified as a list of strings if
  different cell sizes per loop is desired. The length of the list must be
  `nloops+1` to account for the final image after all the self-calibration loops
  are complete.
  
* **robust**: The Briggs' robust parameter used during gridding. Specified
  identically to the `cell` parameter.
  
* **imsize**: The size of the image in pixels. The image dimensions must be
  specified in both dimensions, hence a list of [6144,6144] is treated as a
  single specification and will be applied identically to all loops. To specify
  a different image size per iteration, a list-of-lists syntax must be used,
  such as - ``[[6144, 6144], [8192,8192]]`.

* **wprojplanes** : The number of W-projection planes to use (in case the
  `wproject` gridder is specified). Specified identically to the `cell`
  parameter.
  
* **niter**: The number of iterations to perform per loop. Imaging will continue
  until either the stopping threshold (described below) is reached, or the
  iteration limit is reached. Specified identically to the `cell` parameter.
  
* **threshold**: The stopping threshold per loop (a total of `nloop+1`). The
  stopping threshold for the initial loop must be specified as a string with
  units. Subsequent loops may either be specified as a decimal, or as a string
  with units. If a decimal is specified, values > 1 are considered as
  multipliers on the image RMS and values < 1 are considered to be the threshold
  value in Jy.
  
* **nterms** : The number of Taylor terms to use for broadband imaging.
  Specified identically to the `cell` parameter.
  
* **gridder** : The gridder to use for imaging, typically one of either
  `standard` or `wproject`. Specified identically to the `cell` parameter.
  
* **deconvolver** : The deconvolver to use during imaging. Specified identically
  to the `cell` parameter.
  
* **calmode** : The calibration type per loop - one of either `'p'` (phase only)
  or `'ap'` (amplitude and phase). A blank string implies no calibration will be
  performed on that loop. Must be of length `nloop`. 
  
* **solint** : The solution interval per loop, specified similar to `calmode`. 

* **uvrange**: The UV range limit to consider for gaincal. In the presence of
  strong RFI, it can be helpful to discard the shortest baselines while
  performing the gain calibration solve. This does not affect the imaging steps.
  Specified similar to the `calmode` parameter.
  
* **flag**: True/False - Determines whether flagging is performed on the
  residuals prior to the gain calibration solve. In situations with RFI this can
  improve the image quality on subsequent loops. Setting it to `False` can
  provide a speedup to the imaging process.
  
* **gaintype**: The gaintype specified in the `gaincal` task. Typically one of
  either `G` or `T`. Use `T` if preserving the polarization in instruments with
  linear feeds is important (e.g., MeerKAT).
  
* **discard_nloops**: Discard the first `discard_nloops` loops. This is useful
  if the initial images are intentionally smaller and intended to generate more
  accurate source masks for example. It will only discard the gain solutions
  from loops where `calmode` is not blank.
  
* **outlier_threshold**: The threshold value to identify an outlier. Considered
  to be an SNR threshold if > 1 and an absolute value in Jy if < 1. The
  algorithm for determining and imaging outliers is explained in much more
  detail in the next section. 
