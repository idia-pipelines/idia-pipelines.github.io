---
layout: default
title: Quick Start
parent: processMeerKAT
nav_order: 1
---

# Quick Start

**Note: It is not necessary to copy the raw data (i.e. the MS) to your working directory. The first step of the pipeline does this for you by creating an MMS or MS, and does not attempt to manipulate the raw data (e.g. stored in `/idia/projects` - see [data format](/docs/processMeerKAT/Example-Use-Cases/#data-format)).**

### 1. In order to use the `processMeerKAT.py` script, source the `setup.sh` file:

        source /idia/software/pipelines/master/setup.sh

which will add the correct paths to your `$PATH` and `$PYTHONPATH` in order to correctly use the pipeline. We recommend you add this to your `~/.profile`, for future use.

### 2. Build a config file:

        processMeerKAT.py -B -C myconfig.txt -M mydata.ms


This defines several variables that are read by the pipeline while calibrating the data, as well as requesting resources on the cluster. The config file parameters are described by in-line comments in the config file itself wherever possible.

### 3. To run the pipeline:

        processMeerKAT.py -R -C myconfig.txt

This will create `submit_pipeline.sh`, which you can then run with `./submit_pipeline.sh` to submit all pipeline jobs to the SLURM queue.

Other convenient scripts are also created that allow you to monitor and (if necessary) kill the jobs. `summary.sh` provides a brief overview of the status of the jobs in the pipeline, `findErrors.sh` checks the log files for commonly reported errors, and `killJobs.sh` kills all the jobs from the current run of the pipeline, ignoring any other jobs you might have running.

For help, run `processMeerKAT.py -h`, which provides a brief description of all the command line arguments.

## Using multiple spectral windows (new in v1.1)

Starting with v1.1 of the processMeerKAT pipeline, the default behaviour is to split up the MeerKAT band into 16 spectral windows (SPWs), and process each concurrently. This results in a few major usability changes as outlined below:

1. **Calibration output** : Since the calibration is performed independently per SPW, all the output specific to that SPW is within it's own directory. Output such as the calibration tables, logs, plots etc. per SPW can be found within each SPW directory.

2. **Logs in the top level directory** : Logs in the top level directory (*i.e.,* the directory where the pipeline was launched) correspond to the scripts in the `precal_scripts` and `postcal_scripts` variables in the config file. These scripts are run from the top level before and after calibration respectively. By default these correspond to the scripts to calculate the reference antenna (if enabled), partition the data into SPWs, and concat the individual SPWs back into a single MS/MMS.

More detailed information about SPW splitting is found [here](/docs/processMeerKAT/using-the-pipeline#spw-splitting).

<!-- The documentation can be accessed on the [pipelines website](/docs/processMeerKAT), or on the [Github wiki](https://github.com/idia-astro/pipelines/wiki). -->

----------------------------------------------------------------------------------------------------------