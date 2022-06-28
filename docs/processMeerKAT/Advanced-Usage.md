---
layout: default
title: Advanced Usage
parent: processMeerKAT
nav_order: 11
---

# Inserting your own scripts

Our design allows the user to insert their own scripts into the pipeline, along with or instead of our own scripts. All scripts are assumed to be written in python, with extension `.py`. They must either have hard-coded values for input such as the MS name, or be able to read the config file and extract the values (e.g. as in the main() function of most of our scripts).

To insert your own scripts, either build a config file and edit the `scripts`, `precal_scripts` or `postcal_scripts` parameters (see [SPW splitting](/docs/processMeerKAT/config-files#spw-splitting)) argument to contain your list of scripts, or pass your scripts via command line during building your config file. For each script that is added, three arguments are needed

1. The path to the script
2. Whether the script is threadsafe (for MPI - i.e. it can use `casampi`)
3. The path to the container with which to call the script - use `''` for the default container

The path to the scripts (and containers) can be an absolute path, a relative path, or in your bash path. If none of these exist, the script is assumed to be in one of the pipeline scripts directories (e.g. `/idia/software/pipelines/master/processMeerKAT` and below). Hence simply using `partition.py` will call the partition script in the cross-calibration scripts directory.

## Adding scripts to config file

Edit the `scripts`, `precal_scripts` or `postcal_scripts` parameters (see [SPW splitting](/docs/processMeerKAT/config-files#spw-splitting)) in your config file, which must be a list of lists/tuples.

## Adding scripts via command line

Build a config file pointing to your scripts, each time appending the same three arguments (listed above):

```processMeerKAT.py -B -C myconfig.txt -S /absolute/path/to/my/script.py False /absolute/path/to/container.simg -S partition.py True '' -S relative/path/to/my/script.py True relative/path/to/container.simg -S flag_round_1.py True '' -S script_in_bash_PATH.py False container_in_bash_path.simg -S setjy.py True ''```

An error will be raised if any of the scripts or containers aren't found.

Similarly, the `[-b --precal_scripts]` and `[-a --postcal_scripts]` parameters can be used to add precal and postcal scripts (see [SPW splitting](/docs/processMeerKAT/config-files#spw-splitting)) via the command line, with the same syntax.

# Editing pipeline scripts

Users may wish to edit some of the pipeline scripts to do their own custom progressing by changing CASA task parameters not exposed via the config file. To do so, first copy the script (e.g. from `/idia/software/pipelines/master/processMeerKAT/crosscal_scripts/`) to your working directory, and then edit it from there. The pipeline will first look for a local copy of the pipeline scripts (in your working directory and the one above) and run this instead of the normal version of the script.

<!-- If you can't find the script, type `which processMeerKAT.py` into your terminal, and then search in the same directory (e.g. `which `) -->


# Restarting or Resuming the Pipeline

The pipeline can be killed (via `./killJobs.sh`) and re-run from the start at any point. However, this is not always necessary. The pipeline is generally not designed to run twice within the same directory, since CASA will complain the files already exist. However, the pipeline can be resumed by skipping certain steps, achieved by editing the `scripts` parameter in your config file and removing the steps that have already been run.

As the hidden config (`.config.tmp`) is updated during your pipeline run, you may first wish to check the differences (e.g. with `diff myconfig.txt .config.tmp`) and update your config accordingly. For example, the `vis` or reference antennas (and bad antennas) may have been updated. If you remove the `partition` step, and you have `nspw` > 1, the pipeline will attempt to update the config files in your SPW directories by updating the `vis` to the partitioned (M)MS.

Particular care should also be taken in cases where a job crashed during writing to a column of your (M)MS (e.g. during flagging, applying, or `setjy`), to ensure the data are not corrupted. Checking the logs should reveal which step was being run, and if the job crashed while writing to the column, it is best to remove that (M)MS and re-run the relevant steps (or if necessary, wipe everything in your working directory and restart the pipeline). Similar care must be taken during steps where a new product was being written (e.g. `partition`, `split`, `concat`, or any of the imaging jobs). In such cases, it is best to remove the incomplete product(s) and re-run that step (and the subsequent ones). For example, if an imaging step crashed, remove the image products, patch any bugs that may have caused the crash (by correcting your config, or [editing the script for that step](#editing-pipeline-scripts)), and re-run that step (and subsequent ones). Or if a split step crashed, remove the incomplete (M)MS, patch any bugs, and re-run.

There are a few steps within the pipeline that only need to be run once for a given dataset. For datasets that have already been through the default pipeline, the solutions from `xx_yy_solve` (or `xy_yx_solve`) have been written to the `caltables` directory (renamed to `caltables_round1` if a 2nd round is written, which is then written to `caltables`), which will automatically be applied during running `xx_yy_apply` (or `xy_yx_apply`). Simlarly, if `calcrefant=True` and the `calc_refant.py` script was included, the reference antenna and list of bad antennas would have been calculated, and can be found in `ant_stats.txt` (and `logs/calc_refant-*.err`). These can be written in your new config file (e.g. check `.config.tmp`), where you can now set `calcrefant=False`.

Other than editing the `scripts` parameter, users can run any selected parts of the pipeline by passing scripts in via the command-line arguments (see [above](#inserting-your-own-scripts)). Users can also manually submit sbatch jobs corresponding to certain steps within the pipeline, although this will not be captured by the pipeline infrastructure, such as the bash `jobScripts`: `summary.sh`, `killJobs.sh`, etc.

# Spectral line pre-processing

One advanced use case is to process continuum data at lower spectral resolution (e.g. 1k / 1024 channels), and apply that calibration to higher spectral-resolution data (e.g. 32k / 32,768 channels). There are several ways to do this, and an optimal method is to partition and fully calibrate a 1k copy of your data, and then partition a separate 32k copy. You can then use your fresh 32k copy to re-run bandpass calibration, apply your (cross-calibration and self-calibration) solutions, predict to the model column (based on your 1k images), and flag on the `RESIDUAL` column. This is achievable by "tricking" the pipeline into doing so by renaming or moving your previous (M)MSs, and partitioning out a fresh 32k copy with the same default names (and then replacing your bandpass caltable with a 32k bandpass solution). However, there are a few quirky steps involved in doing this, so please get in touch with us at [ilifu Support](mailto:support@ilifu.ac.za) to ask about doing this.

Once your 32k data are calibrated, and have valid model and corrected data colums, a simple `uvsub` call can be made to subtract the continuum model from your data, and then a `uvcontsub` call with your preferred parameters can be run in parallel mode with `casampi`.
