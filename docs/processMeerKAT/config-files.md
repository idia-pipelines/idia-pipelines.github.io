---
layout: default
title: Configuration Files
parent: processMeerKAT
nav_order: 6
---

# Configuration files

The config file is where you set parameters affecting how you run the pipeline. The default config contains the following:

```
[data]
vis = ''

[fields]
bpassfield = ''
fluxfield = ''
phasecalfield = ''
targetfields = ''
extrafields = ''

[slurm]                           # See processMeerKAT.py -h for documentation
nodes = 1
ntasks_per_node = 8
plane = 1
mem = 232                         # Use this many GB of memory (per node)
partition = 'Main'                # SLURM partition to use
exclude = ''                      # SLURM nodes to exclude
time = '12:00:00'
submit = False
container = '/idia/software/containers/casa-6.5.0-modular.sif'
mpi_wrapper = 'mpirun'
name = ''
dependencies = ''
account = 'b03-idia-ag'
reservation = ''
modules = ['openmpi/4.0.3']
verbose = False
precal_scripts = [('calc_refant.py',False,''), ('partition.py',True,'')]
postcal_scripts = [('concat.py',False,''), ('plotcal_spw.py', False, ''), ('selfcal_part1.py',True,''), ('selfcal_part2.py',False,''), ('science_image.py', True, '')]
scripts = [ ('validate_input.py',False,''),
            ('flag_round_1.py',True,''),
            ('calc_refant.py',False,''),
            ('setjy.py',True,''),
            ('xx_yy_solve.py',False,''),
            ('xx_yy_apply.py',True,''),
            ('flag_round_2.py',True,''),
            ('xx_yy_solve.py',False,''),
            ('xx_yy_apply.py',True,''),
            ('split.py',True,''),
            ('quick_tclean.py',True,'')]

[crosscal]
minbaselines = 4                  # Minimum number of baselines to use while calibrating
chanbin = 1                       # Number of channels to average before calibration (during partition)
width = 1                         # Number of channels to (further) average after calibration (during split)
timeavg = '8s'                    # Time interval to average after calibration (during split)
createmms = True                  # Create MMS (True) or MS (False) for cross-calibration during partition
keepmms = True                    # Output MMS (True) or MS (False) during split
spw = '*:880~933MHz,*:960~1010MHz,*:1010~1060MHz,*:1060~1110MHz,*:1110~1163MHz,*:1299~1350MHz,*:1350~1400MHz,*:1400~1450MHz,*:1450~1500MHz,*:1500~1524MHz,*:1630~1680MHz' # Spectral window / frequencies to extract for MMS
nspw = 11                         # Number of spectral windows to split into
calcrefant = False                # Calculate reference antenna in program (overwrites 'refant')
refant = 'm059'                   # Reference antenna name / number
standard = 'Stevens-Reynolds 2016'# Flux density standard for setjy
badants = []                      # List of bad antenna numbers (to flag)
badfreqranges = [ '933~960MHz',   # List of bad frequency ranges (to flag)
                  '1163~1299MHz',
                  '1524~1630MHz']

[run]                             # Internal variables for pipeline execution
continue = True
dopol = False
```

If you're also performing self-calibration (option ``[-2 --do2GC]`` - see [here](/docs/processMeerKAT/self-calibration-in-processmeerkat)), the default config will also contain the `[selfcal]` section:

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

If you're also performing science imaging (option `[-I --science_image]` - see [here](/docs/processMeerKAT/science-imaging-in-processmeerkat)), the default config will also conatin the `[image]` section:

```
[image]
cell = '1.5arcsec'
robust = -0.5
imsize = [6144, 6144]
wprojplanes = 512
niter = 50000
threshold = 10                    # S/N value if >= 1.0 and rmsmap != '', otherwise Jy
multiscale = [0, 5, 10, 15]
nterms = 2                        # Number of taylor terms
gridder = 'wproject'
deconvolver = 'mtmfs'
restoringbeam = ''
stokes = 'I'
pbthreshold = 0.1                 # Threshold below which to mask the PB for PB correction
mask = ''
rmsmap = ''
outlierfile = ''
```

If you do not perform either self-calibration or science imaging, the script related to those steps will be stripped from your config. Similarly `calc_refant` will be stripped from your config if `calcrefant=False` (the default), in which case `m059` will be used as a good default reference antenna, or the pipeline will output a warning if this antenna doesn't exist in your input MS, and will reommend other good reference antennas.

When the pipeline is run, the contents of your config file are copied to the hidden file `.config.tmp` and each python script reads the parameters from this file as it is run. This way, the user cannot easily break the pipeline during the time it is running. This means changing the `[slurm]` section in your config file will have no effect unless you once again run `processMeerKAT.py -R`.

# Polarisation config

If you select `[-P --dopol]` during the `[-B --build]` step of the pipeline, your config file be similar to the above, except `xy_yx_solve` and `xy_yx_apply` replace the 2nd call of the `xx_yy_solve` and `xx_yy_apply` scripts. Furthermore, in the `[run]` section, `dopol=True` will be set, which will cause all four correlations to be included during the initial `partition` step (or the pipeline will output a warning during the `[-B --build]` step if only two correlations exist). Additionally, we recommend setting `gaintype = 'T'` in the `[selfcal]` section, if performing self-calibration for polarisation processing.

# Manually selecting field IDs

As discussed [here](/docs/processMeerKAT/using-the-pipeline#selecting-ms-and-field-ids), the pipeline selects field IDs for you by default, during the `[-B --build]` step. However, these can be overwritten after this step by editing the `[fields]` section of your config file and manually selecting field IDs. Both field names (i.e. strings) and field IDs (i.e. strings of integers) are supported, although field names are preferable.

Example use cases for manually selecting field IDs include when your input MS has mislabelled (or missing) `INTENT`, and when multiple flux/bandpass calibrators were used (or labelled as such via their `INTENT`), but where the default is less preferred (as the pipeline chooses the one with the most scans by default). For example, you may have two scans on field `J0408-6545`, but only one on `J1939-6342`, but the latter is still preferred as its model is more well constrained. Lastly, you may wish to remove `extrafields` to reduce processing time, if you have no interest in calibrating these fields and producing quick-look images of them.

# SPW Splitting

[Version 1.1](/docs/processMeerKAT/Release-Notes#version-11) introduced spectral window (SPW) splitting, where each separate SPW is processed concurrently. Here we discuss all that is relevant to this functionality.

This mode is the default mode, invoked by selecting a value greater than one with the config parameter `nspw`. This mode is recommended for most use cases, when the input dataset is TB in size, and the number of scans is typical (tens of scans). When the input data (after pre-processing, including selection of `spw`) is small (tens to hundreds of GB), or when the number of scans is very large (hundreds), the single MS mode (see [MS only](/docs/processMeerKAT/example-use-cases#ms-only-single-thread-processing)) is recommended. Alternatively, for such a use case, the user could set `nspw=1`, resulting in the previous behaviour from version 1.0, but with potential polarisation calibration and flux scale issues.

When selecting `nspw` > 1 and passing in a single SPW range via the `spw` config parameter (e.g. `spw=*:880~1680MHz`), the data will split into `nspw` SPWs during the `[-R --run]` step, which will be equally-sized frequency ranges encompassing the frequency range provided. The [calibration algorithm](/docs/processMeerKAT/cross-calibration-in-processmeerkat) within each SPW remains the same. However, the resulting Stokes I calibration when `nspw` > 1 is improved overall, as it accounts for the intrinsic spectrum of the phase calibrator, which would otherwise be treated as flat across the entire `spw`. Furthermore, the [polarisation calibration](/docs/processMeerKAT/cross-calibration-in-processmeerkat) is improved, since each SPW, within which the phase calibrator's leakages and Stokes Q and U are assumed to be constant, is solved separately, which accounts for the wideband polarisation structure.

## Setting `spw` and `nspw`

The SPWs into which the pipeline splits and independently (and concurrently) processes, is determined by the final values stored in the config parameter `spw`. Multiple SPWs are listed as comma-seperated values, but will only be considered as separate SPWs when `nspw` is equal to the number of comma-separated SPWs given by the `spw` parameter. If `nspw=1` and `spw` contains comma-separated values, only a single SPW will be processed, and the comma-separated values will be passed into the CASA tasks that set the `spw` parameter. If `nspw` is not equal to the number of comma-separated SPWs, the pipeline will output a warning and set `nspw` in your config to be equal to the number of comma-separated SPWs.

Therefore, the SPWs can be manually set to ranges that will be separately (and concurrently) processed, such as manual frequency ranges that avoid regions of persistent RFI. We recommend the following default for this purpose:

```
nspw=11
spw = '*:880~933MHz,*:960~1010MHz,*:1010~1060MHz,*:1060~1110MHz,*:1110~1163MHz,*:1299~1350MHz,*:1350~1400MHz,*:1400~1450MHz,*:1450~1500MHz,*:1500~1524MHz,*:1630~1680MHz'
```

This also enables you to set `badfreqranges = []`, as the SPWs have already been constructed to avoid these regions of persistent RFI.

Alternatively, if `spw` is equal to a single value (i.e. not a comma-separated list), the pipeline can split the frequency range given by `spw` into `nspw` equal ranges, which is done during the `[-R --run]` step. This is the default behaviour when `spw` is not manually set. By default, `nspw` is set to 16 during the `[-B --build]` step, and then updated to 12 during the `[-R --run]` step, since 4 SPWs are completely encompassed by the default flagging mask given by the config parameter `badfreqranges`, which has a default value of `['933~960MHz', '1163~1299MHz','1524~1630MHz']`. So after the `[-B --build]` step, the default `spw` is `*:860~1680MHz`, and during the `[-R --run]` step, the pipeline will remove `*:1167.5~1218.75MHz, *:1218.75~1270.0MHz, *:1526.25~1577.5MHz` and `*:1577.5~1628.75MHz`, so that the final default `spw` is

```
*:860.0~911.25MHz,*:911.25~962.5MHz,*:962.5~1013.75MHz,*:1013.75~1065.0MHz,*:1065.0~1116.25MHz,*:1116.25~1167.5MHz,*:1270.0~1321.25MHz,*:1321.25~1372.5MHz,*:1372.5~1423.75MHz,*:1423.75~1475.0MHz,*:1475.0~1526.25MHz,*:1628.75~1680.0MHz
```

Any frequency unit can be used for each comma-separated `spw`, such as GHz, kHz, or no unit, which selects channel indices. However, the removal of SPWs that are encompassed by the RFI mask will only be removed when using `MHz`.

After updating the config's SPWs, the `[-R --run]` step creates directories for each of the SPWs, named as `880~933MHz, 960~1010MHz`, etc. It then copies your config file into each of them but overwrites `spw` to be the single SPW that will be processed, and `nspw=1`. Furthermore, if you have not requested all 32 tasks per node (i.e. each SPW will be processed by an entire node) with the config parameter `ntasks_per_node=32`, it will overwrite the config parameter `mem` with the integet part of the previous value divided by `nspw/2`. Lastly, it will set `precal_script` and `postcal_scripts` to empty lists (see below).

Each SPW directory created during the `[-R --run]` step will be processed independently and concurrently, after the initial partition job that runs over the single input MS (see below). This is achieved by running an instance of the pipeline within each SPW directory, with `nspw=1`, following the same calibration recipe within each separate SPW as within version 1.0.

## Pre-cal and post-cal scripts

When `nspw` > 1, the config parameter `scripts` refers to the separate scripts that will be run as single jobs within each SPW directory. Any scripts that should be run within the top-level working directory (i.e. above the SPW directories) are stored in `precal_scripts`, and `postcal_scripts`, respectively run before and after running the scripts within each SPW directory. By default, `precal_scripts = [('calc_refant.py',False,''), ('partition.py',True,'')]`, and `postcal_scripts = [('concat.py',False,''), ('plotcal_spw.py', False, ''), ('selfcal_part1.py',True,''), ('selfcal_part2.py',False,''), ('science_image.py', True, '')]`, although `calc_refant.py` will be stripped if `calcrefant=False`, and the selfcal and imaging scripts will be stripped if you do not select these routines during the `[-B --build]` step with `[-2 --do2GC]` and `[-I --science_image]`, respectively. The `partition.py` and `concat.py` steps are discussed below.

The steps following `concat` are run over the concatenated dataset that spans all of the SPWs. Alternatively, these scripts (e.g. for self-carlibation and science imaging) can be appended to the end of `scripts`, which will then be run on the targets split from each of the SPWs. Similar to the behaviour during the cross-calibration steps, self-calibrating separately over SPWs will partially solve for frequency dependence of your gain, albeit with a loss of signal-to-noise (compared to the full-band self-calibration run after concatenating). After doing this, concat can be run, and then one final science imaging step over the whole (concatenated) band. Alternatively, you could perform a custom combination of your SPW images, such as a weighted average image, assuming you've matched the resolution with a common `restoringbeam` and/or `uvtaper`.

<!-- `calc_refant.py` calculates the antenna statistics by considering the percentage of flagged data...-->

During the `[-R --run]` step, if `nspw=1` and `precal_scripts` or `postcal_scripts` are not empty, a warning will be output, and these lists will be respectively prepended and appended to `scripts`, and then set to empty lists.

## Partition

The initial partition, given by the `partiton.py` script (by default the last script in `precal_scripts`), which reads your input MS and partitions out your selected `spw` into an MMS, is run as a SLURM array job when `nspw` > 1. This step allows multiple concurrently-running partitions over several SPWs, but limited to < 200 cores, meaning that sometimes not every SPW is partitioned concurrently. After the first SPW is partitioned, the first SPW launches an instance of the pipeline, and so on until the last SPW is processing. This behaviour is a special case that only works when `partition.py` is the last script within `precal_scripts`, otherwise the processing of every SPW waits until the last script within `precal_scripts` has been run.

## Bash `jobScripts`

When using `nspw` > 1, the behaviour of the bash scripts within the `jobScripts` directory (symlinked from the working directory) is different. Each bash script will iterate through the SPW directories and display the output for that SPW, running each SPW directory's instance of that bash script, which remains the same as above. Since there are more than 100 jobs by default, `./summary.sh` will display only the running or failed jobs, and will not display completed or pending jobs, whereas `./fullSummary.sh` will display all of the jobs. Additionally, when `precal_scripts` and `postcal_scripts` are not empty lists, there will be a version of each of these `jobScripts` starting with `allSPW_`. These correspond to the pipeline jobs that are run at top-level directory over all SPWs, which is also displayed when calling the other `jobScripts`. For example, when running `./summary.sh -X` (where `-X` outputs ones line per job) during the middle of your processing with `nspw=2`, you will see something similar to the following:

```
jcollier@slurm-login:/scratch/users/jcollier/MIGHTEE/nspw2$ ./summary.sh -X
SPW #1: /scratch/users/jcollier/MIGHTEE/nspw2/1350~1375MHz
          JobID         JobName  Partition    Elapsed NNodes NTasks NCPUS  MaxDiskRead MaxDiskWrite             NodeList   TotalCPU    CPUTime     MaxRSS      State ExitCode
--------------- --------------- ---------- ---------- ------ ------ ----- ------------ ------------ -------------------- ---------- ---------- ---------- ---------- --------
1838779         setjy                 Main   00:02:31      1            4                                    compute-039   00:00:00   00:10:04             RUNNING        0:0
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------
SPW #2: /scratch/users/jcollier/MIGHTEE/nspw2/1375~1400MHz
          JobID         JobName  Partition    Elapsed NNodes NTasks NCPUS  MaxDiskRead MaxDiskWrite             NodeList   TotalCPU    CPUTime     MaxRSS      State ExitCode
--------------- --------------- ---------- ---------- ------ ------ ----- ------------ ------------ -------------------- ---------- ---------- ---------- ---------- --------
1838788         flag_round_1          Main   00:06:22      1            4                                    compute-059   00:00:00   00:25:28             RUNNING        0:0
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------
All SPWs: /scratch/users/jcollier/MIGHTEE/nspw2
          JobID         JobName  Partition    Elapsed NNodes NTasks NCPUS  MaxDiskRead MaxDiskWrite             NodeList   TotalCPU    CPUTime     MaxRSS      State ExitCode
--------------- --------------- ---------- ---------- ------ ------ ----- ------------ ------------ -------------------- ---------- ---------- ---------- ---------- --------
1838776_0       partition             Main   00:03:49      1            8                                    compute-058   00:00:00   00:30:32             COMPLETED      0:0
1838776_1       partition             Main   00:03:19      1            8                                    compute-018   00:00:00   00:26:32             COMPLETED      0:0
1838797         concat                Main   00:02:13      1            1                                    compute-020   00:00:00   00:00:00             PENDING        0:0
1838798         plotcal_spw           Main   00:00:19      1            1                                    compute-020   00:00:00   00:00:00             PENDING        0:0
```

## Concatenation and further imaging

After all of the SPWs have completed, irrespective of their completion status, the scripts within `postcal_scripts` are run at the top level directory (i.e. above the SPW directories). By default, the first of these is `concat.py`, which concatenates any output MMSs/MSs and quick-look images. By default these are the split calibrator and target fields, and their quick-look images.

If the config parameter `keepmms=True`, `virtualconcat` will be run for each field that has been split into its own MMS, resulting in an MMS with `nspw` x `nscans` sub-MSs. If `keepmms=False`, `concat` is run over each field that has been split into its own MS, resulting in a single concatenated MS. In both cases, the resulting MMS/MS will now contain multiple SPWs, given by `nspw` (assuming that all SPWs successfully completed processing). These can then be imaged over the whole concatenated frequency range via further image processing, such as [self-calibration](/docs/processMeerKAT/self-calibration-in-processmeerkat) and [science imaging](/docs/processMeerKAT/science-imaging-in-processmeerkat).

Furthermore, each split field's quick-look image is concatenated together into a quick-look continuum cube. If any SPWs failed to split out a field or create an image for that field, they will be excluded from the (image and MMS/MS) concatenation.
