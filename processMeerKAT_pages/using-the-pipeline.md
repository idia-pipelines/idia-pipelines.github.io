---
layout: page
title: Using the Pipeline
permalink: /processMeerKAT_pages/using-the-pipeline
---
## Introduction

This repository contains scripts for pipeline processing of MeerKAT data. It is a work in progress, and so far, just runs the initial calibration steps.

### Requirements

This pipeline is designed to run on the IDIA/Ilifu data intensive research facility, making use of SLURM and MPICASA. For other use, please contact the authors.

## Usage

* The usage can be seen by running
```processMeerKAT.py -h```

which gives

```
usage: /data/exp_soft/pipelines/processMeerKAT/processMeerKAT.py
       [-h] [-M path] [-C path] [-N num] [-t num] [-p num] [-m num]
       [-S script threadsafe container] [--mpi_wrapper path]
       [--container path] [-c bogus] [-s] [-v] (-B | -R | -V)

Process MeerKAT data via CASA measurement set. Version: 1.0

optional arguments:
  -h, --help            show this help message and exit
  -M path, --MS path    Path to measurement set.
  -C path, --config path
                        Path to config file.
  -N num, --nodes num   Use this number of nodes [default: 15; max: 30].
  -t num, --ntasks-per-node num
                        Use this number of tasks (per node) [default: 8; max:
                        128].
  -p num, --plane num   Distribute tasks of this block size before moving onto
                        next node [default: 4; max: ntasks-per-node].
  -m num, --mem num     Use this many MB of memory (per node) [default: 98304;
                        max: 235520 MB (230 GB).
  -S script threadsafe container, --scripts script threadsafe container
                        Run pipeline with these scripts, in this order, using
                        this container (3rd tuple value - empty string to
                        default to [--container]). Is it threadsafe (2nd tuple
                        value)?
  --mpi_wrapper path    Use this mpi wrapper when calling scripts [default:
                        '/data/exp_soft/pipelines/casa-
                        prerelease-5.3.0-115.el7/bin/mpicasa'].
  --container path      Use this container when calling scripts [default:
                        '/data/exp_soft/pipelines/casameer-5.4.1.simg'].
  -c bogus, --CASA bogus
                        Bogus argument to swallow up CASA call.
  -s, --submit          Submit jobs immediately to SLURM queue [default:
                        False].
  -v, --verbose         Verbose output? [default: False].
  -B, --build           Build default config file using input MS.
  -R, --run             Run pipeline with input config file.
  -V, --version         Display the version.
```

### Simple usage

* To get things working, source setup.sh, which will add to your PATH and PYTHONPATH

```source /data/exp_soft/pipelines/processMeerKAT/pipelines/setup.sh```

* To print the version of the pipeline, run

```processMeerKAT.py -V```

* To build a config file, which the pipeline reads as input for how to process the data, run

```processMeerKAT.py -B -C myconfig.txt -M mydata.ms```

* To run the pipeline, run

```processMeerKAT.py -R -C myconfig.txt```

This will create `submit_pipeline.sh`, which you can then run to submit all pipeline jobs to a SLURM queue:

```./submit_pipeline.sh```

* Display a summary of the submitted jobs

```./summary.sh```

* Kill the submitted jobs

```./killJobs.sh```

* If the pipeline crashes, or reports an error, find the error(s) by running

```./findErrors.sh```

### Detailed usage

* Build config file using custom SLURM configuration

```processMeerKAT.py -B -C myconfig.txt -M mydata.ms -N 10 -t 32 -p 8 -m 131072```

* Build config file using different MPI wrapper and container

```processMeerKAT.py -B -C myconfig.txt -M mydata.ms --mpi_wrapper /path/to/another/mpi/wrapper --container /path/to/another/container```

* Build config file with different set of (python) scripts

```processMeerKAT.py -B -C myconfig.txt -S /absolute/path/to/my/script.py False /absolute/path/to/container.simg -S partition.py True '' -S relative/path/to/my/script.py True relative/path/to/container.simg -S flag_round_1.py True '' -S script_in_bash_PATH.py False container_in_bash_path.simg run_setjy.py True ''```

* Run the pipeline immediately in verbose mode

```processMeerKAT.py -R -v -s --config myconfig.txt```

**NOTE:** All other command-line arguments passed into `processMeerKAT.py` when using option `[-R --run]` will have no effect, since the arguments are read from the config file at this point. Only options `[-v --verbose]`, `[-s --submit]` and `[-C --config]` will have any effect at this point. If option `[-v --verbose]` is used, it will overwrite the value of `verbose` in your config file.

## Config files

The config file is where you set parameters affecting how you run the pipeline. The default config contains the following

```
[crosscal]
minbaselines = 4        # Minimum number of baseline while calibrating
specave = 1             # Number of chans to avg after calibration
timeave = '8s'          # Time interval to avg after calibration
spw = '0:860~1700MHz'   # spw to use for calibration
calcrefant = True       # Calculate reference antenna in program (overwrites 'refant')
refant = 'm005'         # Reference antenna name/number
badfreqranges = ['944~947MHz', '1160~1310MHz', '1476~1611MHz', '1670~1700MHz']
standard = 'Perley-Butler 2010'  # Flux density standard for setjy

[slurm]                 #See processMeerKAT.py -h for documentation
nodes = 15
ntasks_per_node = 8
cpus_per_task = 3
mem = 98304
plane = 4
submit = False
container = '/data/exp_soft/pipelines/casameer-5.4.1.simg'
mpi_wrapper = '/data/exp_soft/pipelines/casa-prerelease-5.3.0-115.el7/bin/mpicasa'
verbose = False
scripts = [ ('validate_input.py',False,''),
            ('partition.py',True,''),
            ('flag_round_1.py',True,''),
            ('run_setjy.py',True,''),
            ('parallel_cal.py',False,''),
            ('parallel_cal_apply.py',True,''),
            ('flag_round_2.py',True,''),
            ('run_setjy.py',True,''),
            ('cross_cal.py',False,''),
            ('cross_cal_apply.py',True,''),
            ('split.py',True,'')]
```

When the pipeline is run, the contents of your config file are copied to `.config.tmp` and each python script reads the parameters from this file as it is run. This way, the user cannot easily break the pipeline during the time it is running. This also means changing your config file will have no effect unless you once again run `processMeerKAT.py -R`.

## Selecting MS and fields IDs

As previously stated, to build a config file, run

```processMeerKAT.py -B -C myconfig.txt -M mydata.ms```

This calls CASA and adds a `[data]` section to your config file, which points to your MS, and a `[fields]` section, which points to the field IDs you want to process as bandpass, total flux and phase calibrators, and science target. Any of these may have multiple fields separated by a comma. 

The following is an example of what is appended to the bottom of your config file.

```
[data]
vis = '/data/projects/deep/1497056411.ms'

[fields]
bpassfield = '0'
fluxfield = '0'
phasecalfield = '1'
targetfields = '2'
```

You can edit your config file and change the field IDs, such as removing unwanted IDs where multiple exist.

## Inserting your own scripts

Our design allows the user to insert their own scripts into the pipeline, along with or instead of our own scripts. All scripts are assumed to be in python, with extension `.py`. To insert your own scripts, either build a config file and edit the `scripts` argument to contain your list of scripts, or pass your scripts via command line during building your config file. For each script that is added, three arguments are needed

1. The path to the script
2. Whether the script is threadsafe (for MPI - e.g. it uses mpicasa)
3. The path to the container with which to call the script - use ' ' for the default container

The path to the scripts (and containers) can be an absolute path, a relative path, or in your bash path. If none of these exist, the script (or container) is assumed to be in the calibration scripts directory (`/data/exp_soft/pipelines/processMeerKAT/pipelines/processMeerKAT/cal_scripts/`). Hence simply using `partition.py` will call the partition script in the calibration scripts directory.

#### Adding scripts to config file

Edit the `scripts` argument in your config file, which must be a list of lists/tuples.

#### Adding scripts via command line

Build a config file pointing to your scripts, each time appending the same three arguments (listed
above):

```processMeerKAT.py -B -C myconfig.txt -S /absolute/path/to/my/script.py False
/absolute/path/to/container.simg -S partition.py True '' -S relative/path/to/my/script.py True
relative/path/to/container.simg -S flag_round_1.py True '' -S script_in_bash_PATH.py False
container_in_bash_path.simg run_setjy.py True ''```

An error will be raised if any of the scripts or containers aren't found.
