---
layout: default
title: Using the Pipeline
parent: processMeerKAT
nav_order: 3
---

## Usage
The usage can be seen by running
```processMeerKAT.py -h``` the output of which is documented below.


### Simple usage

* To get things working, source `setup.sh`, which will add to your `$PATH` and `$PYTHONPATH` (add this to your `~/.profile`, for future use)

```source /idia/software/pipelines/master/setup.sh```

* To print the version of the pipeline, run

```processMeerKAT.py -V```

* To build a config file, which the pipeline reads as input for how to process the data, run

```processMeerKAT.py -B -C myconfig.txt -M mydata.ms```

* To run the pipeline, run

```processMeerKAT.py -R -C myconfig.txt```

This will create `submit_pipeline.sh`, which you can then run to submit all pipeline jobs to a SLURM queue:

`./submit_pipeline.sh`

* Display a summary of the submitted jobs

`./summary.sh`

* Kill the submitted jobs

`./killJobs.sh`

* If the pipeline crashes, or reports an error, find the error(s) by running (after the pipeline has run)

`./findErrors.sh`

* Once the pipeline has completed, display the start and end times of each job by running

`./displayTimes.sh`

### Detailed usage

* Build config file locally (e.g. on a fat node) using a custom SLURM configuration (nodes and tasks per node may be overwritten in your config file with something more appropriate by the end of the build step)

```processMeerKAT.py -l -B -C myconfig.txt -M mydata.ms -p Test02 -N 10 -t 8 -P 4 -m 100 -T 06:00:00 -n mydata_```

* Build config file using different MPI wrapper and container

```processMeerKAT.py -B -C myconfig.txt -M mydata.ms --mpi_wrapper /path/to/another/mpi/wrapper --container /path/to/another/container```

* Build config file with different set of (python) scripts

```processMeerKAT.py -B -C myconfig.txt -S /absolute/path/to/my/script.py False /absolute/path/to/container.simg -S partition.py True '' -S relative/path/to/my/script.py True relative/path/to/container.simg -S flag_round_1.py True '' -S script_in_bash_PATH.py False container_in_bash_PATH.simg setjy.py True ''```

* Run the pipeline immediately in verbose mode

```processMeerKAT.py -R -v -s -C myconfig.txt```

**NOTE:** All other command-line arguments passed into `processMeerKAT.py` when using option `[-R --run]` will have no effect, since the arguments are read from the config file at this point. Only options `[-s --submit], [-v --verbose]` and `[-C --config]` will have any effect at this point. Similarly, changing the `[slurm]` section in your config file after using option `[-R --run]` will have no effect unless you `[-R --run]` again.

The command line help text is :
```
usage: /idia/software/pipelines/master/processMeerKAT/processMeerKAT.py
       [-h] [-M path] [-C path] [-N num] [-t num] [-D num] [-m num] [-p name]
       [-T time] [-S script threadsafe container]
       [-b script threadsafe container] [-a script threadsafe container]
       [-w path] [-c path] [-n unique] [-d list] [-e nodes] [-A group]
       [-r name] [-l] [-s] [-v] [-q] [-P] [-2] [-x] (-B | -R | -V | -L)

Process MeerKAT data via CASA measurement set. Version: 1.1

optional arguments:
  -h, --help            show this help message and exit
  -M path, --MS path    Path to measurement set.
  -C path, --config path
                        Relative (not absolute) path to config file.
  -N num, --nodes num   Use this number of nodes [default: 1; max: 79].
  -t num, --ntasks-per-node num
                        Use this number of tasks (per node) [default: 16; max:
                        32].
  -D num, --plane num   Distribute tasks of this block size before moving onto
                        next node [default: 1; max: ntasks-per-node].
  -m num, --mem num     Use this many GB of memory (per node) for threadsafe
                        scripts [default: 232; max: 232].
  -p name, --partition name
                        SLURM partition to use [default: 'Main'].
  -T time, --time time  Time limit to use for all jobs, in the form d-hh:mm:ss
                        [default: '12:00:00'].
  -S script threadsafe container, --scripts script threadsafe container
                        Run pipeline with these scripts, in this order, using
                        these containers (3rd value - empty string to default
                        to [-c --container]). Is it threadsafe (2nd value)?
  -b script threadsafe container, --precal_scripts script threadsafe container
                        Same as [-S --scripts], but run before calibration.
  -a script threadsafe container, --postcal_scripts script threadsafe container
                        Same as [-S --scripts], but run after calibration.
  -w path, --mpi_wrapper path
                        Use this mpi wrapper when calling threadsafe scripts
                        [default: '/idia/software/pipelines/casa-pipeline-
                        release-5.6.1-8.el7/bin/mpicasa'].
  -c path, --container path
                        Use this container when calling scripts [default:
                        '/idia/software/containers/casa-stable-5.6.2-2.simg'].
  -n unique, --name unique
                        Unique name to give this pipeline run (e.g. 'run1_'),
                        appended to the start of all job names. [default: ''].
  -d list, --dependencies list
                        Comma-separated list (without spaces) of SLURM job
                        dependencies (only used when nspw=1). [default: ''].
  -e nodes, --exclude nodes
                        SLURM worker nodes to exclude [default: ''].
  -A group, --account group
                        SLURM accounting group to use (e.g. 'b05-pipelines-ag'
                        - check 'sacctmgr show user $(whoami) -s
                        format=account%30') [default: 'b03-idia-ag'].
  -r name, --reservation name
                        SLURM reservation to use. [default: ''].
  -l, --local           Build config file locally (i.e. without calling srun)
                        [default: False].
  -s, --submit          Submit jobs immediately to SLURM queue [default:
                        False].
  -v, --verbose         Verbose output? [default: False].
  -q, --quiet           Activate quiet mode, with suppressed output [default:
                        False].
  -P, --dopol           Perform polarization calibration in the pipeline
                        [default: False].
  -2, --do2GC           Perform (2GC) self-calibration in the pipeline
                        [default: False].
  -x, --nofields        Do not read the input MS to extract field IDs
                        [default: False].
  -B, --build           Build config file using input MS.
  -R, --run             Run pipeline with input config file.
  -V, --version         Display the version of this pipeline and quit.
  -L, --license         Display this program's license and quit.
```


## Config files

The config file is where you set parameters affecting how you run the pipeline. The default config contains the following

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
container = '/idia/software/containers/casa-stable-5.6.2-2.simg'
mpi_wrapper = '/idia/software/pipelines/casa-pipeline-release-5.6.1-8.el7/bin/mpicasa'
name = ''
dependencies = ''
account = 'b03-idia-ag'
reservation = ''
verbose = False
precal_scripts = [('calc_refant.py',False,''), ('partition.py',True,'')]
postcal_scripts = [('concat.py',False,''), ('plotcal_spw.py', False, ''), ('selfcal_part1.py',True,''), ('selfcal_part2.py',False,''), ('run_bdsf.py', False, ''), ('make_pixmask.py', False, '')]
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
            ('quick_tclean.py',True,''),
            ('plot_solutions.py',False,'')]

[crosscal]
minbaselines = 4                  # Minimum number of baselines to use while calibrating
chanbin = 1                       # Number of channels to average before calibration (during partition)
width = 1                         # Number of channels to (further) average after calibration (during split)
timeavg = '8s'                    # Time interval to average after calibration (during split)
createmms = True                  # Create MMS (True) or MS (False) for cross-calibration during partition
keepmms = True                    # Output MMS (True) or MS (False) during split
spw = '0:880~1680MHz'             # Spectral window / frequencies to extract for MMS
nspw = 16                         # Number of spectral windows to split into
calcrefant = False                # Calculate reference antenna in program (overwrites 'refant')
refant = 'm059'                   # Reference antenna name / number
standard = 'Stevens-Reynolds 2016'# Flux density standard for setjy
badants = []                      # List of bad antenna numbers (to flag)
badfreqranges = [ '933~960MHz',   # List of bad frequency ranges (to flag)
                  '1163~1299MHz',
                  '1524~1630MHz']

[run]                             # Internal variables for pipeline execution
continue = True
```

If you're also performing self-calibration (option [-2 --do2GC], experimental at this point), the default config will also contain the `[selfcal]` section:

```
[selfcal]
nloops = 4                        # Number of clean + bdsf + self-cal loops.
restart_no = 0                    # If nonzero, adds this number to nloops to name images
cell = '2.0arcsec'
robust = -0.5
imsize = [6827, 6827]
wprojplanes = 128
niter = [8000, 11000, 14000, 15000, 200000]
threshold = [100e-6, 50e-6, 20e-6, 10e-6, 4e-6] # In units of Jy
multiscale = []
nterms = 2                        # Number of taylor terms
gridder = 'wproject'
deconvolver = 'mtmfs'
solint = ['10min','5min','2min','1min']
calmode = 'p'
atrous = True                     # Source find for diffuse emission (see PyBDSF docs)
```

When the pipeline is run, the contents of your config file are copied to `.config.tmp` and each python script reads the parameters from this file as it is run. This way, the user cannot easily break the pipeline during the time it is running. This means changing the [slurm] section in your config file will have no effect unless you once again run `processMeerKAT.py -R`.

## Selecting MS and fields IDs

As previously stated, to build a config file, run

```processMeerKAT.py -B -C myconfig.txt -M mydata.ms```

This calls CASA and adds a `[data]` section to your config file, which points to your MS, and a `[fields]` section, which points to the field IDs you want to process as bandpass, total flux and phase calibrators, and science target(s). Only targets may have multiple fields separated by a comma, and all extra calibrator fields are appended as "targets", to allow for solutions to be applied to them, and images to be made of them (see [v1.0 release notes](/docs/processMeerKAT/Release-Notes/)).

The following is an example of what is appended to the bottom of your config file.

```
[data]
vis = '/idia/projects/deep/1497056411.ms'

[fields]
bpassfield = '0'
fluxfield = '0'
phasecalfield = '1'
targetfields = '2'
```

You can edit your config file and change the field IDs.

## SPW Splitting

[Version 1.1](/docs/processMeerKAT/Release-Notes) introduced spectral window (SPW) splitting, where each separate SPW is processed concurrently. Here we discuss all that is relevant to this functionality.

This mode is the default mode, invoked by selecting a value greater than one with the config parameter `nspw`. This mode is recommended for most use cases, when the input dataset is TB in size, and the number of scans is typical (tens of scans). When the input data (after pre-processing, including selection of `spw`) is small (tens to hundreds of GB), or when the number of scans is very large (hundreds), the single MS mode (see [MS only](/docs/processMeerKAT/example-use-cases#ms-only-single-thread-processing)) is recommended. Alternatively, for such a use case, the user could set `nspw=1`, resulting in the previous behaviour from version 1.0, but with potential polarisation calibration and flux scale issues.

When selecting `nspw` > 1, the data is split into `nspw` SPWs, which are equally-sized frequency ranges encompassing the frequency range given by the config parameter `spw` (set to `0:860~1680MHz` by default). For Stokes I, the [calibration algorithm](/docs/processMeerKAT/cross-calibration-in-processmeerkat) within each SPW remains the same. However, the resulting Stokes I calibration when `nspw` > 1 is improved overall, as it accounts for the intrinsic spectrum of the phase calibrator, which was previous treated as flat across the entire `spw`. Furthermore, the [polarisation calibration](/docs/processMeerKAT/cross-calibration-in-processmeerkat) is improved, since each SPW, within which the phase calibrator's leakages and Stokes Q and U are assumed to be constant, is solved separately, which accounts for the wideband polarisation structure.

### Setting `spw` and `nspw`

The SPWs into which the pipeline splits and independently (and concurrently) processes, is determined by the final values stored in the config parameter `spw`. Multiple SPWs are listed as comma-seperated values, but will only be considered as separate SPWs when `nspw` is equal to the number of comma-separated SPWs given by the `spw` parameter. If `nspw=1` and `spw` contains comma-separated values, only a single SPW will be processed, and the comma-separated values will be passed into the CASA tasks that set the `spw` parameter. If `nspw` is not equal to the number of comma-separated SPWs, the pipeline will output a warning and set `nspw` in your config to be equal to this number.

Therefore, the SPWs can be manually set to ranges that will be separately (and concurrently) processed, such as manual frequency ranges that avoid regions of persistent RFI. Or alternatively, if `spw` is equal to a single value (i.e. not a comma-separated list), the pipeline can split the frequency range given by `spw` into `nspw` equal ranges, which is done during the `[-R --run]` step. This is the default behaviour when `spw` is not manually set. By default, `nspw` is set to 16 during the `[-B --build]` step, and then updated to 12 during the `[-R --run]` step, since 4 SPWs are completely encompassed by the default flagging mask given by the config parameter `badfreqranges`, which has a default value of `['933~960MHz', '1163~1299MHz','1524~1630MHz']`. So after the `[-B --build]` step, the default `spw` is `0:860~1680MHz`, and during the `[-R --run]` step, the pipeline will remove `0:1167.5~1218.75MHz, 0:1218.75~1270.0MHz, 0:1526.25~1577.5MHz` and `0:1577.5~1628.75MHz`, so that the final default `spw` is

```
0:860.0~911.25MHz,0:911.25~962.5MHz,0:962.5~1013.75MHz,0:1013.75~1065.0MHz,0:1065.0~1116.25MHz,0:1116.25~1167.5MHz,0:1270.0~1321.25MHz,0:1321.25~1372.5MHz,0:1372.5~1423.75MHz,0:1423.75~1475.0MHz,0:1475.0~1526.25MHz,0:1628.75~1680.0MHz
```

Any frequency unit can be used for each comma-separated `spw`, such as GHz, kHz, or no unit, which selects channel indices. However, the removal of SPWs that are encompassed by the RFI mask will only be removed when using `MHz`.

After updating the config's SPWs, the `[-R --run]` step creates directories for each of the SPWs, named as `860.0~911.25MHz, 911.25~962.5MHz`, etc. It then copies your config file into each of them but overwrites `spw` to be the single SPW that will be processed, and `nspw=1`. Furthermore, if you have not requested all 32 tasks per node (i.e. each SPW will be processed by an entire node) with the config parameter `ntasks_per_node=32`, it will over the config parameter `mem` by the previous value divided by `nspw`. Lastly, it will set `precal_script` and `postcal_scripts` to empty lists (see below).

Each SPW directory created during the `[-R --run]` step will be processed independently and concurrently, after the initial partition job that runs over the single input MS (see below). This is achieved by running an instance of the pipeline within each SPW directory, with `nspw=1`, following the same calibration recipe within each separate SPW as within version 1.0.

### Pre-cal and post-cal scripts

When `nspw` > 1, the config parameter `scripts` refers to the separate scripts that will be run as single jobs within each SPW directory. Any scripts that should be run within the top-level working directory (i.e. above the SPW directories) are stored in `precal_scripts`, and `postcal_scripts`, respectively run before and after running the scripts within each SPW directory. By default, `precal_scripts = [('calc_refant.py',False,''), ('partition.py',True,'')]`, and `postcal_scripts = [('concat.py',False,''), ('quick_tclean.py',True,'')]`. `partition.py` and `concat.py` as discussed below. The final `quick_tclean.py` is run over a concatenated dataset that spans all of the SPWs.

<!-- `calc_refant.py` calculates the antenna statistics by considering the percentage of flagged data...-->

During the `[-R --run]` step, if `nspw=1` and `precal_scripts` or `postcal_scripts` are not empty, a warning will be output, and these lists will be respectively prepended and appended to `scripts`, and then set to empty lists.

### Partition

The initial partition, given by the `partiton.py` script (by default the last script in `precal_scripts`), which reads your input MS and partitions out your selected `spw` into an MMS, is now run as a SLURM array job. By default, this step concurrently partitions two SPWs. After the first SPW is partitioned, the first SPW launches an instance of the pipeline, and so on until the last SPW is processing. This behaviour only works when `partition.py` is the last script within `precal_scripts`, otherwise the processing of every SPW waits until the last script within `precal_scripts` has been run.

### Bash `jobScripts`

When using `nspw` > 1, the behaviour of the bash scripts within the `jobScripts` directory (symlinked from the working directory) is different. Each bash script will iterate through the SPW directories and display the output for that SPW, running each SPW directory's instance of that bash script, which remains the same as above. Since there are more than 100 jobs by default, `./summary.sh` will display only the running or failed jobs, and will not display completed or pending jobs, whereas `./fullSummary.sh` will display all of the jobs. Additionally, when `precal_scripts` and `postcal_scripts` are not empty lists, there will be a version of each of these `jobScripts` starting with `allSPW_`. These correspond to the pipeline jobs that are run at top-level directory over all SPWs, which is also displayed when calling the other `jobScripts`. For example, when running `./summary.sh` during the middle of your processing with `nspw=4`, you will see something similar to the following:

```
jcollier@slurm-login:/scratch/users/jcollier/MIGHTEE/32k_nspw4$ ./summary.sh
SPW #1: /scratch/users/jcollier/MIGHTEE/32k_nspw4/1350.0~1375.0MHz
          JobID         JobName  Partition    Elapsed NNodes NTasks NCPUS  MaxDiskRead MaxDiskWrite             NodeList   TotalCPU    CPUTime     MaxRSS      State ExitCode
--------------- --------------- ---------- ---------- ------ ------ ----- ------------ ------------ -------------------- ---------- ---------- ---------- ---------- --------
1362974         setjy                 Main   01:22:05      2           16                                slwrk-[102-103]   00:00:00   21:53:20               RUNNING      0:0
1362974.0       orted                        01:22:05      1      1     1                                      slwrk-103   00:00:00   01:22:05               RUNNING      0:0
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------
SPW #2: /scratch/users/jcollier/MIGHTEE/32k_nspw4/1375.0~1400.0MHz
          JobID         JobName  Partition    Elapsed NNodes NTasks NCPUS  MaxDiskRead MaxDiskWrite             NodeList   TotalCPU    CPUTime     MaxRSS      State ExitCode
--------------- --------------- ---------- ---------- ------ ------ ----- ------------ ------------ -------------------- ---------- ---------- ---------- ---------- --------
1362983         flag_round_1          Main   01:14:44      2           16                                slwrk-[160-161]   00:00:00   19:55:44               RUNNING      0:0
1362983.0       orted                        01:14:44      1      1     1                                      slwrk-161   00:00:00   01:14:44               RUNNING      0:0
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------
SPW #3: /scratch/users/jcollier/MIGHTEE/32k_nspw4/1400.0~1425.0MHz
          JobID         JobName  Partition    Elapsed NNodes NTasks NCPUS  MaxDiskRead MaxDiskWrite             NodeList   TotalCPU    CPUTime     MaxRSS      State ExitCode
--------------- --------------- ---------- ---------- ------ ------ ----- ------------ ------------ -------------------- ---------- ---------- ---------- ---------- --------
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------
SPW #4: /scratch/users/jcollier/MIGHTEE/32k_nspw4/1425.0~1450.0MHz
          JobID         JobName  Partition    Elapsed NNodes NTasks NCPUS  MaxDiskRead MaxDiskWrite             NodeList   TotalCPU    CPUTime     MaxRSS      State ExitCode
--------------- --------------- ---------- ---------- ------ ------ ----- ------------ ------------ -------------------- ---------- ---------- ---------- ---------- --------
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------
All SPWs: /scratch/users/jcollier/MIGHTEE/32k_nspw4
          JobID         JobName  Partition    Elapsed NNodes NTasks NCPUS  MaxDiskRead MaxDiskWrite             NodeList   TotalCPU    CPUTime     MaxRSS      State ExitCode
--------------- --------------- ---------- ---------- ------ ------ ----- ------------ ------------ -------------------- ---------- ---------- ---------- ---------- --------
1362970         calc_refant           Main   00:03:27      1            1                                      slwrk-105  02:54.393   00:03:27             COMPLETED      0:0
1362970.batch   batch                        00:03:27      1      1     1        0.06M        0.00M            slwrk-105  00:00.034   00:03:27      0.01G  COMPLETED      0:0
1362970.0       singularity                  00:03:27      1      1     1       69.58G        0.02M            slwrk-105  02:54.358   00:03:27      1.01G  COMPLETED      0:0
1362971_[3%1]   partition             Main   00:00:00      2           64                                  None assigned   00:00:00   00:00:00               PENDING      0:0
1363012         concat                Main   00:00:00      1            1                                  None assigned   00:00:00   00:00:00               PENDING      0:0
1362971_0       partition             Main   01:22:50      2           64                                slwrk-[130-131]  58:39.694 3-16:21:20             COMPLETED      0:0
1362971_0.batch batch                        01:22:50      1      1    32      552.96G       46.36G            slwrk-130  19:17.852 1-20:10:40      4.15G  COMPLETED      0:0
1362971_0.0     orted                        01:22:49      1      1     4     1671.24G      143.22G            slwrk-131  39:21.841   05:31:16      4.34G  COMPLETED      0:0
1362971_1       partition             Main   00:36:07      2           64                                slwrk-[130-131]  54:48.316 1-14:31:28             COMPLETED      0:0
1362971_1.batch batch                        00:36:07      1      1    32      950.31G       80.41G            slwrk-130  26:23.615   19:15:44      4.39G  COMPLETED      0:0
1362971_1.0     orted                        00:36:06      1      1     4     1273.81G      109.17G            slwrk-131  28:24.700   02:24:24      4.36G  COMPLETED      0:0
1362971_2       partition             Main   01:15:22      2           64                                slwrk-[130-131]   00:00:00 3-08:23:28               RUNNING      0:0
1362971_2.0     orted                        01:15:21      1      1     4                                      slwrk-131   00:00:00   05:01:24               RUNNING      0:0
```

### Concatenation and further imaging

After all of the SPWs have completed, irrespective of their completion status, the scripts within `postcal_scripts` are run at the top level directory (i.e. above the SPW directories). By default, the first of these is `concat.py` which concatenates any output MMSs/MSs and images. By default these are the split calibrator and target fields, and their quick-look images. If the config parameter `keepmms=True`, `virtualconcat` will be run for each field that has been split into its own MMS, resulting in an MMS with `nspw` x `nscans` sub-MSs. If `keepmms=False`, `concat` is run over each field that has been split into its own MS, resulting in a single concatenated MS. In both cases, the resulting MMS/MS will now contain multiple SPWs, given by `nspw` (assuming all successfully completed). These can then be imaged over the whole concatenated frequency range with `quick_tclean.py`, or further image processing, such as the self-calibration routine that will be released in the next version of this pipeline.

Furthermore, each split field's quick-look image is concatenated together into a continuum cube. If any SPWs failed to split out a field or create an image for that field, they will be excluded from the (image and MMS/MS) concatenation.

## Inserting your own scripts

Our design allows the user to insert their own scripts into the pipeline, along with or instead of our own scripts. All scripts are assumed to be written in python, with extension `.py`. They must either have hard-coded values for input such as the MS name, or be able to read the config file and extract the values (e.g. as in the main() function of most of our scripts).

To insert your own scripts, either build a config file and edit the `scripts`, `precal_scripts` or `postcal_scripts` parameters (see [SPW splitting](/docs/processMeerKAT/using-the-pipeline#spw-splitting)) argument to contain your list of scripts, or pass your scripts via command line during building your config file. For each script that is added, three arguments are needed

1. The path to the script
2. Whether the script is threadsafe (for MPI - i.e. it can use mpicasa)
3. The path to the container with which to call the script - use ' ' for the default container

The path to the scripts (and containers) can be an absolute path, a relative path, or in your bash path. If none of these exist, the script (or container) is assumed to be in the calibration scripts directory (`/idia/software/pipelines/master/processMeerKAT/cal_scripts/`). Hence simply using `partition.py` will call the partition script in the calibration scripts directory.

### Adding scripts to config file

Edit the `scripts`, `precal_scripts` or `postcal_scripts` parameters (see [SPW splitting](/docs/processMeerKAT/using-the-pipeline#spw-splitting)) in your config file, which must be a list of lists/tuples.

### Adding scripts via command line

Build a config file pointing to your scripts, each time appending the same three arguments (listed above):

```processMeerKAT.py -B -C myconfig.txt -S /absolute/path/to/my/script.py False /absolute/path/to/container.simg -S partition.py True '' -S relative/path/to/my/script.py True relative/path/to/container.simg -S flag_round_1.py True '' -S script_in_bash_PATH.py False container_in_bash_path.simg setjy.py True ''```

An error will be raised if any of the scripts or containers aren't found.

Similarly, the `[-b --precal_scripts]` and `[-a --postcal_scripts]` parameters can be used to add precal and postcal scripts (see [SPW splitting](/docs/processMeerKAT/using-the-pipeline#spw-splitting)) via the command line, with the same syntax.

## Editing pipeline scripts

Users may wish to edit some of the pipeline scripts to do their own custom progressing by changing CASA task parameters not exposed via the config file. To do so, first copy the script (e.g. from `/idia/software/pipelines/master/processMeerKAT/cal_scripts/`) to your working directory, and then edit it from there. The pipeline will first look for a local copy of the pipeline scripts and run this instead of the normal version of the script.

<!-- If you can't find the script, type `which processMeerKAT.py` into your terminal, and then search in the same directory (e.g. `which `) -->
