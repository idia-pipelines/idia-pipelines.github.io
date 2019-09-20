---
layout: default
title: Using the Pipeline
parent: processMeerKAT
nav_order: 3
---

## Usage

* The usage can be seen by running
```processMeerKAT.py -h```

which gives

```
usage: /idia/software/pipelines/master/processMeerKAT/processMeerKAT.py
       [-h] [-M path] [-C path] [-N num] [-t num] [-P num] [-m num] [-p name]
       [-T time] [-S script threadsafe container] [-w path] [-c path]
       [-n unique] [-l] [-s] [-v] (-B | -R | -V)

Process MeerKAT data via CASA measurement set. Version: 1.0

optional arguments:
  -h, --help            show this help message and exit
  -M path, --MS path    Path to measurement set.
  -C path, --config path
                        Path to config file.
  -N num, --nodes num   Use this number of nodes [default: 8; max: 35].
  -t num, --ntasks-per-node num
                        Use this number of tasks (per node) [default: 4; max:
                        128].
  -P num, --plane num   Distribute tasks of this block size before moving onto
                        next node [default: 2; max: ntasks-per-node].
  -m num, --mem num     Use this many GB of memory (per node) for threadsafe
                        scripts [default: 236; max: 236].
  -p name, --partition name
                        SLURM partition to use [default: 'Main'].
  -T time, --time time  Time limit to use for all jobs, in the form d-hh:mm:ss
                        [default: '12:00:00'].
  -S script threadsafe container, --scripts script threadsafe container
                        Run pipeline with these scripts, in this order, using
                        this container (3rd value - empty string to default to
                        [-c --container]). Is it threadsafe (2nd value)?
  -w path, --mpi_wrapper path
                        Use this mpi wrapper when calling threadsafe scripts
                        [default: '/idia/software/pipelines/casa-
                        prerelease-5.3.0-115.el7/bin/mpicasa'].
  -c path, --container path
                        Use this container when calling scripts [default:
                        '/idia/software/pipelines/casameer-5.4.1.xvfb.simg'].
  -n unique, --name unique
                        Unique name to give this pipeline run (e.g. 'run1_'),
                        appended to the start of all job names. [default: ''].
  -l, --local           Build config file locally (i.e. without calling srun)
                        [default: False].
  -s, --submit          Submit jobs immediately to SLURM queue [default:
                        False].
  -v, --verbose         Verbose output? [default: False].
  -B, --build           Build config file using input MS.
  -R, --run             Run pipeline with input config file.
  -V, --version         Display the version of this pipeline and quit.
  -L, --license         Display this program's license and quit.
```

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

## Config files

The config file is where you set parameters affecting how you run the pipeline. The default config contains the following

```
[crosscal]
minbaselines = 4                  # Minimum number of baselines to use while calibrating
specavg = 1                       # Number of channels to average after calibration (during split)
timeavg = '8s'                    # Time interval to average after calibration (during split)
spw = '0:860~1700MHz'             # Spectral window / frequencies to extract for MMS
calcrefant = False                # Calculate reference antenna in program (overwrites 'refant')
refant = 'm005'                   # Reference antenna name / number
standard = 'Perley-Butler 2010'   # Flux density standard for setjy
badants = []                      # List of bad antenna numbers (to flag)
badfreqranges = [ '935~947MHz',   # List of bad frequency ranges (to flag)
                  '1160~1310MHz',
                  '1476~1611MHz',
                  '1670~1700MHz']

[slurm]                           # See processMeerKAT.py -h for documentation
nodes = 8
ntasks_per_node = 4
plane = 2
mem = 236                         # Use this many GB of memory (per node)
partition = 'Main'                # SLURM partition to use
time = '12:00:00'
submit = False
container = '/idia/software/pipelines/casameer-5.4.1.xvfb.simg'
mpi_wrapper = '/idia/software/pipelines/casa-prerelease-5.3.0-115.el7/bin/mpicasa'
name = ''
verbose = False
scripts = [ ('validate_input.py',False,''),
            ('partition.py',True,''),
            ('calc_refant.py',False,''),
            ('flag_round_1.py',True,''),
            ('setjy.py',True,''),
            ('xx_yy_solve.py',False,''),
            ('xx_yy_apply.py',True,''),
            ('flag_round_2.py',True,''),
            ('setjy.py',True,''),
            ('xy_yx_solve.py',False,''),
            ('xy_yx_apply.py',True,''),
            ('split.py',True,''),
            ('quick_tclean.py',True,''),
            ('plot_solutions.py',False,'')]
```

When the pipeline is run, the contents of your config file are copied to `.config.tmp` and each python script reads the parameters from this file as it is run. This way, the user cannot easily break the pipeline during the time it is running. This means changing the [slurm] section in your config file will have no effect unless you once again run `processMeerKAT.py -R`.

## Selecting MS and fields IDs

As previously stated, to build a config file, run

```processMeerKAT.py -B -C myconfig.txt -M mydata.ms```

This calls CASA and adds a `[data]` section to your config file, which points to your MS, and a `[fields]` section, which points to the field IDs you want to process as bandpass, total flux and phase calibrators, and science target(s). Only targets may have multiple fields separated by a comma, and all extra calibrator fields are appended as "targets", to allow for solutions to be applied to them, and images to be made of them (see [v1.0 release notes](https://idia-pipelines.github.io/docs/processMeerKAT/Release-Notes/)).

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


## Inserting your own scripts

Our design allows the user to insert their own scripts into the pipeline, along with or instead of our own scripts. All scripts are assumed to be written in python, with extension `.py`. They must either have hard-coded values for input such as the MS name, or be able to read the config file and extract the values (e.g. as in the main() function of most of our scripts).

To insert your own scripts, either build a config file and edit the `scripts` argument to contain your list of scripts, or pass your scripts via command line during building your config file. For each script that is added, three arguments are needed

1. The path to the script
2. Whether the script is threadsafe (for MPI - i.e. it can use mpicasa)
3. The path to the container with which to call the script - use ' ' for the default container

The path to the scripts (and containers) can be an absolute path, a relative path, or in your bash path. If none of these exist, the script (or container) is assumed to be in the calibration scripts directory (`/idia/software/pipelines/master/processMeerKAT/cal_scripts/`). Hence simply using `partition.py` will call the partition script in the calibration scripts directory.

### Adding scripts to config file

Edit the `scripts` argument in your config file, which must be a list of lists/tuples.

### Adding scripts via command line

Build a config file pointing to your scripts, each time appending the same three arguments (listed above):

```processMeerKAT.py -B -C myconfig.txt -S /absolute/path/to/my/script.py False /absolute/path/to/container.simg -S partition.py True '' -S relative/path/to/my/script.py True relative/path/to/container.simg -S flag_round_1.py True '' -S script_in_bash_PATH.py False container_in_bash_path.simg setjy.py True ''```

An error will be raised if any of the scripts or containers aren't found.
