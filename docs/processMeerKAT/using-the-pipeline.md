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
usage: /users/jcollier/Scripts/pipelines_casa6_dev/processMeerKAT/processMeerKAT.py [-h] [-M path] [-C path] [-N num] [-t num] [-D num] [-m num] [-p name] [-T time] [-S script threadsafe container] [-b script threadsafe container]
                                                                                    [-a script threadsafe container] [--modules [module [module ...]]] [-w path] [-c path] [-n unique] [-d list] [-e nodes] [-A group] [-r name] [-l] [-s]
                                                                                    [-v] [-q] [-P] [-2] [-I] [-x] [-j] (-B | -R | -V | -L)

Process MeerKAT data via CASA MeasurementSet. Version: 1.1

optional arguments:
  -h, --help            show this help message and exit
  -M path, --MS path    Path to MeasurementSet.
  -C path, --config path
                        Relative (not absolute) path to config file.
  -N num, --nodes num   Use this number of nodes [default: 1; max: 79].
  -t num, --ntasks-per-node num
                        Use this number of tasks (per node) [default: 16; max: 32].
  -D num, --plane num   Distribute tasks of this block size before moving onto next node [default: 1; max: ntasks-per-node].
  -m num, --mem num     Use this many GB of memory (per node) for threadsafe scripts [default: 232; max: 232].
  -p name, --partition name
                        SLURM partition to use [default: 'Main'].
  -T time, --time time  Time limit to use for all jobs, in the form d-hh:mm:ss [default: '12:00:00'].
  -S script threadsafe container, --scripts script threadsafe container
                        Run pipeline with these scripts, in this order, using these containers (3rd value - empty string to default to [-c --container]). Is it threadsafe (2nd value)?
  -b script threadsafe container, --precal_scripts script threadsafe container
                        Same as [-S --scripts], but run before calibration.
  -a script threadsafe container, --postcal_scripts script threadsafe container
                        Same as [-S --scripts], but run after calibration.
  --modules [module [module ...]]
                        Load these modules within each sbatch script.
  -w path, --mpi_wrapper path
                        Use this mpi wrapper when calling threadsafe scripts [default: 'mpirun'].
  -c path, --container path
                        Use this container when calling scripts [default: '/idia/software/containers/casa-6.4.4-modular.simg'].
  -n unique, --name unique
                        Unique name to give this pipeline run (e.g. 'run1_'), appended to the start of all job names. [default: ''].
  -d list, --dependencies list
                        Comma-separated list (without spaces) of SLURM job dependencies (only used when nspw=1). [default: ''].
  -e nodes, --exclude nodes
                        SLURM worker nodes to exclude [default: ''].
  -A group, --account group
                        SLURM accounting group to use (e.g. 'b05-pipelines-ag' - check 'sacctmgr show user $USER cluster=ilifu-slurm20 -s format=account%30') [default: 'b03-idia-ag'].
  -r name, --reservation name
                        SLURM reservation to use. [default: ''].
  -l, --local           Build config file locally (i.e. without calling srun) [default: False].
  -s, --submit          Submit jobs immediately to SLURM queue [default: False].
  -v, --verbose         Verbose output? [default: False].
  -q, --quiet           Activate quiet mode, with suppressed output [default: False].
  -P, --dopol           Perform polarization calibration in the pipeline [default: False].
  -2, --do2GC           Perform (2GC) self-calibration in the pipeline [default: False].
  -I, --science_image   Create a science image [default: False].
  -x, --nofields        Do not read the input MS to extract field IDs [default: False].
  -j, --justrun         Just run the pipeline, don't rebuild each job script if it exists [default: False].
  -B, --build           Build config file using input MS.
  -R, --run             Run pipeline with input config file.
  -V, --version         Display the version of this pipeline and quit.
  -L, --license         Display this program's license and quit.
```

## Selecting MS and fields IDs

As previously stated, to build a config file, run

```processMeerKAT.py -B -C myconfig.txt -M mydata.ms```

This calls CASA and adds a `[data]` section to your config file, which points to your MS, and a `[fields]` section, which points to the field IDs you want to process as bandpass, total flux and phase calibrators, and science target(s), as extracted by your input MS via their `INTENT` labels. Only targets and extra fields may have multiple fields separated by a comma, and all extra calibrator fields are appended as "targets", to allow for solutions to be applied to them, and quick-look images to be made of them (see [v1.0 release notes](/docs/processMeerKAT/Release-Notes/)).

The following is an example of what is appended to the bottom of your config file.

```
[data]
vis = '/idia/software/pipelines/test_data/mightee_cdfs_1350_1400mhz.ms'

[fields]
bpassfield = 'J1939-6342'
fluxfield = 'J1939-6342'
phasecalfield = 'J0240-2309'
targetfields = 'CDFS16'
extrafields = 'J0521+1638'
```

You can edit your config file and change the field IDs, as discussed in [config files](/docs/processMeerKAT/config-files#manually-selecting-field-ids).
