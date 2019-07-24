---
layout: default
title: Quick Start
parent: processMeerKAT
nav_order: 1
---

### 1. In order to use the `processMeerKAT.py` script, source the `setup.sh` file:

        source /idia/software/pipelines/master/setup.sh

which will add the correct paths to your `$PATH` and `$PYTHONPATH` in order to correctly use the pipeline. We recommend you add this to your `~/.profile`, for future use.

### 2. Build a config file:

        processMeerKAT.py -B -C myconfig.txt -M mydata.ms


This defines several variables that are read by the pipeline while calibrating the data, as well as requesting resources on the cluster. The config file parameters are described by in-line comments in the config file itself wherever possible.

### 3. To run the pipeline:

        processMeerKAT.py -R -C myconfig.txt

This will create `submit_pipeline.sh`, which you can then run like `./submit_pipeline.sh` to submit all pipeline jobs to the SLURM queue.

Other convenience scripts are also created that allow you to monitor and (if necessary) kill the jobs. `summary.sh` provides a brief overview of the status of the jobs in the pipeline, `findErrors.sh` checks the log files for commonly reported errors, and `killJobs.sh` kills all the jobs from the current run of the pipeline, ignoring any other jobs you might have running.

For help, run `processMeerKAT.py -h`, which provides a brief description of all the command line arguments.

The documentation can be accessed on the [pipelines website](https://idia-pipelines.github.io/docs/processMeerKAT), or on the [Github wiki](https://github.com/idia-astro/pipelines/wiki).