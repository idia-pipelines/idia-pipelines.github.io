---
layout: default
title: DEEP 2 Tutorial
parent: processMeerKAT
nav_order: 3
---


# DEEP 2 Tutorial

#### This tutorial walks you through running the various steps of the pipeline for a single DEEP 2 dataset, which is a 16 dish, 4k-mode MeerKAT observation of a random patch of sky, 11 GB in size.

To begin, ssh into the ilifu cluster (`slurm.ilifu.ac.za`), and create a working directory somewhere on the filesystem (e.g. `/scratch/users/your_username/tutorial/`).

##### 1. Source `setup.sh`, which will add to your PATH and PYTHONPATH

```source /data/exp_soft/pipelines/processMeerKAT/pipelines/setup.sh```

##### 2. Build a config file, using verbose mode, and pointing to the DEEP 2 dataset

```processMeerKAT.py -B -C tutorial_config.txt -M /data/projects/deep/1491550051.ms -v```

You should get the following output, with different timestamps

```
2019-01-16 14:28:52,854 INFO: Extracting field IDs from measurement set "/data/projects/deep/1491550051.ms" using CASA.
2019-01-16 14:28:52,854 DEBUG: Using the following command:
	 singularity exec /data/exp_soft/pipelines/casameer-5.4.1.xvfb.simg xvfb-run -d casa --nologger --nogui --nologfile -c /data/exp_soft/pipelines/processMeerKAT-jordan/pipelines/processMeerKAT/cal_scripts/get_fields.py -B -M /data/projects/deep/1491550051.ms --config tutorial_config.txt 1>/dev/null
```

This calls CASA via the default singularity container, without writing log files and suppressing standard out, and runs `get_fields.py`. The purpose of this call is to extract the field IDs corresponding to our different targets. The output statements with `DEBUG` correspond to those output during verbose mode.

##### 3. View the config file created, which has the following contents:

```
[crosscal]
minbaselines = 4        # Minimum number of baselines while calibrating
specave = 1             # Number of chans to avg after calibration
timeave = '8s'          # Time interval to avg after calibration
spw = '0:860~1700MHz'   # spw to use for calibration
calcrefant = True       # Calculate reference antenna in program (overwrites 'refant')
refant = 'm005'         # Reference antenna name/number
badfreqranges = ['944~947MHz', '1160~1310MHz', '1476~1611MHz', '1670~1700MHz']
standard = 'Perley-Butler 2010'  # Flux density standard for setjy
badants = []            # List of bad antenna numbers in the MS

[slurm]
nodes = 15
ntasks_per_node = 8
mem = 98304
plane = 4
submit = False
container = '/data/exp_soft/pipelines/casameer-5.4.1.xvfb.simg'
mpi_wrapper = '/data/exp_soft/pipelines/casa-prerelease-5.3.0-115.el7/bin/mpicasa'
verbose = True
scripts = [('validate_input.py', False, ''), ('partition.py', True, ''), ('flag_round_1.py', True, ''), ('run_setjy.py', True, ''), ('parallel_cal.py', False, ''), ('parallel_cal_apply.py', True, ''), ('flag_round_2.py', True, ''), ('run_setjy.py', True, ''), ('cross_cal.py', False, ''), ('cross_cal_apply.py', True, ''), ('split.py', True, ''), ('plot_solutions.py', False, '')]

[data]
vis = '/data/projects/deep/1491550051.ms'

[fields]
bpassfield = '0'
fluxfield = '0'
phasecalfield = '1,2'
targetfields = '3'
```

This config file contains four sections - crosscal, slurm, data, and fields. The fields IDs we just extracted, seen in section `[fields]`, correspond to field 0 for the bandpass calibrator, field 0 for the total flux calibrator, fields 1 and 2 for the phase calibrator, and field 3 for the science target (i.e. the DEEP 2 field). If multiple fields are present corresponding to the bandpass calibrator and total flux calibrator, only the first field is selected. If a field isn't found according to its intent, a warning is displayed, and the field for the total flux calibrator is selected. If the total flux calibrator isn't present, the program will display an error and terminate.

The SLURM parameters in section `[slurm]` correspond to those seen by running `processMeerKAT.py -h`. By default, for all threadsafe scripts (i.e. those with `True` in the `scripts` list), we use 15 nodes, 8 tasks per node, 98304 MB (96 GB) of memory per node, and `plane=4` (which distributes four tasks onto one node before moving onto next node). For script that aren't threadsafe (i.e. those with `False` in the `scripts` list), we use a single node, and a single task per node. We both scripts that are threadsafe and those that aren't, we use a single CPU per task, and explicitly export `OMP_NUM_THREADS=1`, since there is little evidence of a speedup with more than one CPU per task.

The cross-calibration parameters in section `[crosscal]` correspond to various CASA parameters passed into the calibration tasks that the pipeline used, each of which is documented here. By default all frequency ranges listed in `badfreqranges` will be flagged out entirely. The first script the pipeline runs (`validate_input.py`) will likely change the value of `refant`, and add a list of bad antennas to `badant` for flagging.

##### 4. Run the pipeline using your config file

```processMeerKAT.py -R -C tutorial_config.txt```

You should get the following output, with different timestamps

```
2019-01-16 15:05:17,110 DEBUG: Wrote sbatch file "validate_input.sbatch"
2019-01-16 15:05:17,112 DEBUG: Wrote sbatch file "partition.sbatch"
2019-01-16 15:05:17,115 DEBUG: Wrote sbatch file "flag_round_1.sbatch"
2019-01-16 15:05:17,118 DEBUG: Wrote sbatch file "run_setjy.sbatch"
2019-01-16 15:05:17,121 DEBUG: Wrote sbatch file "parallel_cal.sbatch"
2019-01-16 15:05:17,123 DEBUG: Wrote sbatch file "parallel_cal_apply.sbatch"
2019-01-16 15:05:17,126 DEBUG: Wrote sbatch file "flag_round_2.sbatch"
2019-01-16 15:05:17,128 DEBUG: Wrote sbatch file "run_setjy.sbatch"
2019-01-16 15:05:17,131 DEBUG: Wrote sbatch file "cross_cal.sbatch"
2019-01-16 15:05:17,134 DEBUG: Wrote sbatch file "cross_cal_apply.sbatch"
2019-01-16 15:05:17,136 DEBUG: Wrote sbatch file "split.sbatch"
2019-01-16 15:05:17,139 DEBUG: Wrote sbatch file "plot_solutions.sbatch"
2019-01-16 15:05:17,142 DEBUG: Copying 'tutorial_config.txt' to '.config.tmp', and using this to run pipeline
2019-01-16 15:05:17,145 INFO: Master script "submit_pipeline.sh" written, but will not run.
```

A number of sbatch files have now been written to your working directory, each of which corresponds to the python script in the list of scripts set by the `scripts` parameter in our config file. Our config file was copied to `.config.tmp`, which is the config file written and edited by the pipeline, which the user should not touch. A bash script called `submit_pipeline.sh` was written, which we will look at soon. However, this script was not run, since we set `submit = False` in our config file (you can change this in your config file, or by using option `[-s --submit]` when you build your config file with `processMeerKAT.py`). Lastly, a `logs` directory was created, which will store the log files from the SLURM output.

##### 5. View `validate_input.sbatch`, which has the following contents:

```
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=1
#SBATCH --mem=196608
#SBATCH --job-name=validate_input
#SBATCH --distribution=plane=1
#SBATCH --output=logs/validate_input-%j.out
#SBATCH --error=logs/validate_input-%j.err

export OMP_NUM_THREADS=1

srun singularity exec /data/exp_soft/pipelines/casameer-5.4.1.xvfb.simg xvfb-run -d casa --nologger --nogui --logfile logs/validate_input-${SLURM_JOB_ID}.casa -c /data/exp_soft/pipelines/processMeerKAT-jordan/pipelines/processMeerKAT/cal_scripts/validate_input.py --config .config.tmp
```

Since this script is not threadsafe, the job is called with `srun`, and is configured to run a single task on a single node, with double the memory per node. The last line shows the CASA call of the `validate_input.py` task, which will validate the parameters in the config file, and calculate a reference antenna (since we set parameter `calrefant=True`).

##### 6. Run the first sbatch job

```sbatch validate_input.sbatch```

You should see the following output, corresponding to your SLURM job ID

```Submitted batch job 10969```

##### 7. View your job in the SLURM queue

`squeue`

You will see something similar to the following

```
JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
10949      Main   averag   tremou PD       0:00      1 (Dependency)
10890      Main phase_ca   bright PD       0:00      1 (Dependency)
10891      Main  wsclean   bright PD       0:00      1 (Dependency)
10900      Main tclean-x  krishna  R   18:56:21     10 slwrk-[015-024]
10961      Main BFtclean    frank  R    1:08:16      8 slwrk-[025-032]
10947      Main     flag   tremou  R    4:33:18      1 slwrk-012
10964      Main  wsclean   bright  R      42:47      1 slwrk-010
10679      Main  wsclean   bright  R    8:14:31      1 slwrk-008
10889      Main     flag   bright  R   22:35:04      1 slwrk-006
10960      Main wsclean_ williams  R    1:11:34      1 slwrk-007
10925 JupyerSpa spawner-     russ  R   14:49:12      1 slwrk-001
10953 JupyerSpa spawner- jbochene  R    2:34:00      1 slwrk-001
10969      Main validate jcollier  R       0:01      1 slwrk-011
```



Above we can see the job with name `validate` was submitted to SLURM worker node 11, amongst a number of jobs in the main partition and the JupyterSpawner partition. Like some of those above, your job may list `(Priority)`, which means it is too low a priority to be submitted to the queue at this point, or `(Resources)`, which means it is awaiting resources to be made available.

*NOTE: You can view just your jobs with `squeue -u your_username`, an individual job with `squeue -j 10969`, and just the jobs in the main partition with `squeue -p Main`. You can view which nodes are allocated, which are idle, which are mixed - i.e. partially allocated - and which are down in the main partition with `sinfo -p Main`. Often it is good idea to check this before selecting your SLURM parameters.*

##### 8. Watch your submitted jobs

`watch sacct`

You will initially see something similar to the following

```
       JobID    JobName  Partition    Account  AllocCPUS      State ExitCode
------------ ---------- ---------- ---------- ---------- ---------- --------
10969        validate_+       Main b03-pipel+          1    RUNNING      0:0
10969.0      singulari+            b03-pipel+          1    RUNNING      0:0
```

`sacct` lists all recently submitted jobs and their status. If your job fails, it will list `FAILED` under `State`. However, please note jobs running `mpicasa` often state they have failed when they haven't. Similarly, when jobs do genuinely fail, the pipeline may continue to run. Both of these are issues we are working to overcome.

When your job completes, you will see something similar to the following

```
       JobID    JobName  Partition    Account  AllocCPUS      State ExitCode
------------ ---------- ---------- ---------- ---------- ---------- --------
10969        validate_+       Main b03-pipel+          1  COMPLETED      0:0
10969.batch       batch            b03-pipel+          1  COMPLETED      0:0
10969.0      singulari+            b03-pipel+          1  COMPLETED      0:0
```

##### 9. View the contents of your `logs` directory

```
ls logs
validate_input-10969.casa  validate_input-10969.err  validate_input-10969.out
```

As specified in our sbatch file, standard out is written to `logs/validate_input-10969.out`, standard error is written to `logs/validate_input-10969.err`, and the CASA logs are written to `validate_input-10969.casa`. Your standard error log should be empty, your CASA log will have little of interest, but your standard output log will contain the following output at the end:

```
This is version 1.0 of the pipeline
Flux field scan no: 1

 Antenna statistics on flux field
 ant    median    rms
Median:    8.457    283.475
best antenna:  7  amp =    7.68, rms =  208.30
1st good antenna:  0  amp =    8.36, rms =  314.52
setting reference antenna to: 7
```

Here we see `validate_input.py` has selected antenna 7 as the best reference antenna, which measures comparable amplitude for the total flux calibrator compared to antenna 0, but a lower RMS.

##### 10. View `ant_stats.txt`

You should see the following contents, corresponding to the amplitude and RMS each of the antennas measure

```
0    8.363  314.522
1    7.423  262.870
2    7.314  294.384
3    7.749  229.863
4    8.851  327.913
5    6.733  263.164
6    7.458  260.646
7    7.679  208.301
8    7.735  334.914
9    9.654  308.313
10    8.551  268.647
11   11.157  272.565
12   10.105  271.011
13   13.183  400.641
14   13.408  456.787
15  227.818  604.202
```

##### 11. View `.config.tmp`

You should now see `refant = 7` and `badants = [13, 14, 15]`.

##### 12. View `partition.sbatch`, which has the following contents:

```
#!/bin/bash
#SBATCH --nodes=15
#SBATCH --ntasks-per-node=8
#SBATCH --cpus-per-task=1
#SBATCH --mem=98304
#SBATCH --job-name=partition
#SBATCH --distribution=plane=4
#SBATCH --output=logs/partition-%j.out
#SBATCH --error=logs/partition-%j.err

export OMP_NUM_THREADS=1

/data/exp_soft/pipelines/casa-prerelease-5.3.0-115.el7/bin/mpicasa singularity exec /data/exp_soft/pipelines/casameer-5.4.1.xvfb.simg xvfb-run -d casa --nologger --nogui --logfile logs/partition-${SLURM_JOB_ID}.casa -c /data/exp_soft/pipelines/processMeerKAT-jordan/pipelines/processMeerKAT/cal_scripts/partition.py --config .config.tmp
```

Here we see the same default SLURM parameters for threadsafe tasks, as discussed in section 3. We now use mpicasa as the mpi wrapper, since we are calling a threadsafe script `partition.py`, which calls CASA task `partition`, which partitions the data into several sub measurement sets (sub-MSs - see section 14 below) and selects only frequencies specified by your spectral window with parameter `spw` in your config file.

##### 13. Watch your job in the queue

`squeue -j 10974`

You will see something similar to the following, showing that 15 nodes are now being used between worked nodes 13-24 and 33-35.

```
JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
10974      Main partitio jcollier  R       0:38     15 slwrk-[013-024,033-035]
```

Wait until the job completes, before step 13.

##### 14. View the contents of `1491550051.mms`.

You should see two new files - `1491550051.mms` and `1491550051.mms.flagversions`, which corresponds to the data and flag data of a multi-measurement set (MMS). From now on, the pipeline operates on these data, rather than the . The same path to the original MS will remain in your config file under section `[data]`, but each task will point to your MMS.

Inside this MMS, you will find the same tables and metadata as in a normal MS, but you will also see a `SUBMSS` directory, which should have the following contents.

```
1491550051.mms.0000.ms	1491550051.mms.0004.ms	1491550051.mms.0008.ms
1491550051.mms.0001.ms	1491550051.mms.0005.ms	1491550051.mms.0009.ms
1491550051.mms.0002.ms	1491550051.mms.0006.ms	1491550051.mms.0010.ms
1491550051.mms.0003.ms	1491550051.mms.0007.ms	1491550051.mms.0011.ms
```

These are the 12 sub-MSs, partitioned by this observation's 12 scans of the various targets.

If we now view the CASA log, you will find a bunch of junk output from mpicasa (often including nominal "errors", sometimes severe), and 13 calls of `partition`, corresponding to 12 workers for your 12 sub-MSs, and one master process. Similarly, your standard output logs will contains 120 sets of output from CASA launching, corresponding to the 120 threads (i.e. 15 node x 8 tasks per node) and some junk output from mpicasa. Again, your standard error log should be empty.

##### 15. Edit your config file to run the next steps

Edit `tutorial_config.txt` to remove the first two and last six tuples in the `scripts` parameter, so it looks like the following:

```
scripts = [('flag_round_1.py', True, ''), ('run_setjy.py', True, ''), ('parallel_cal.py', False, ''), ('parallel_cal_apply.py', True, '')]
```

Replace `refant` and `badants` with what was found by `validate_input.py`, and select the submit option, so it looks like the following:

```
[crosscal]
refant = 7
badants = [13, 14, 15]
	.
	.
[slurm]
submit = True
```

##### 16. Run the pipeline using your config file

```processMeerKAT.py -R -C tutorial_config.txt```

You should see the following output, with different timestamps

```
2019-01-16 17:21:14,002 DEBUG: Wrote sbatch file "flag_round_1.sbatch"
2019-01-16 17:21:14,005 DEBUG: Wrote sbatch file "run_setjy.sbatch"
2019-01-16 17:21:14,008 DEBUG: Wrote sbatch file "parallel_cal.sbatch"
2019-01-16 17:21:14,011 DEBUG: Wrote sbatch file "parallel_cal_apply.sbatch"
2019-01-16 17:21:14,014 INFO: Running master script "submit_pipeline.sh"
Copying tutorial_config.txt to .config.tmp, and using this to run pipeline.
Submitting flag_round_1.sbatch SLURM queue with following command
sbatch flag_round_1.sbatch
Submitting run_setjy.sbatch SLURM queue with following command
sbatch -d afterok:10983 run_setjy.sbatch
Submitting parallel_cal.sbatch SLURM queue with following command
sbatch -d afterok:10983,10984 parallel_cal.sbatch
Submitting parallel_cal_apply.sbatch SLURM queue with following command
sbatch -d afterok:10983,10984,10985 parallel_cal_apply.sbatch
Submitted sbatch jobs with following IDs: 10983,10984,10985,10986
Run killJobs.sh to kill all the jobs.
Run summary.sh to view the progress.
Run findErrors.sh to find errors (after pipeline has run).
Run displayTimes.sh to display start and end timestamps (after pipeline has run).
```

As before, we see the sbatch files being written to our working directory. Since we set `submit=True`, `submit_pipeline.sh` has been run, and all output after that (without the timestamps) comes from this bash script. After the first job is run (`sbatch flag_round_1.sbatch`), each other job is run with a dependency on all previous jobs (e.g. `sbatch -d afterok:10983,10984,10985 parallel_cal_apply.sbatch`). We can see this by calling `squeue -u your_username`, which shows those jobs `(Dependency)`. `submit_pipeline.sh` then writes four job scripts, all of which are explained in the output, written to `jobScripts` with a timestamp appended to the filename, and symlinked from your working directory. `findErrors.sh` finds errors after this pipeline run has completed, overlooking all nominal MPI errors.

These tasks follow the first step of a two step calibration process that is summarised on our wiki page [here](https://github.com/idia-astro/pipelines/wiki/Calibration-in-processMeerKAT).

##### 17. Run `./summary.sh`

This script simply calls `sacct` for all jobs submitted within this pipeline run. You should get output similar to the following.

```
       JobID    JobName  Partition    Account  AllocCPUS      State ExitCode
------------ ---------- ---------- ---------- ---------- ---------- --------
10983        flag_roun+       Main b03-pipel+        120    RUNNING      0:0
10983.0           orted            b03-pipel+         14    RUNNING      0:0
10984         run_setjy       Main b03-pipel+        120    PENDING      0:0
10985        parallel_+       Main b03-pipel+          1    PENDING      0:0
10986        parallel_+       Main b03-pipel+        120    PENDING      0:0
```

Those `PENDING` are the jobs with dependencies. Once this pipeline run has completed, `./summary.sh` should give output similar to the following.

```
       JobID    JobName  Partition    Account  AllocCPUS      State ExitCode
------------ ---------- ---------- ---------- ---------- ---------- --------
10983        flag_roun+       Main b03-pipel+        120  COMPLETED      0:0
10983.batch       batch            b03-pipel+          8  COMPLETED      0:0
10983.0           orted            b03-pipel+         14  COMPLETED      0:0
10984         run_setjy       Main b03-pipel+        120  COMPLETED      0:0
10984.batch       batch            b03-pipel+          8  COMPLETED      0:0
10984.0           orted            b03-pipel+         14  COMPLETED      0:0
10985        parallel_+       Main b03-pipel+          1  COMPLETED      0:0
10985.batch       batch            b03-pipel+          1  COMPLETED      0:0
10985.0      singulari+            b03-pipel+          1  COMPLETED      0:0
10986        parallel_+       Main b03-pipel+        120  COMPLETED      0:0
10986.batch       batch            b03-pipel+          8  COMPLETED      0:0
10986.0           orted            b03-pipel+         14  COMPLETED      0:0
```

##### 18. View `caltables` directory

The calibration solution tables have been written to `caltables/1491550051.mms.*`, including `bcal, gcal, fluxscale` and `kcal`, corresponding to the calibration solutions for bandpass, complex gains, flux-scaled complex gains, and delays, respectively.

##### 19. Run `./displayTimes.sh`

You should see output similar to the following, which shows this run took 22 minutes to complete, the longest of which was flagging for 8.5 minutes.

```
logs/flag_round_1-10983.casa
2019-01-16 15:21:55
2019-01-16 15:30:30
logs/run_setjy-10984.casa
2019-01-16 15:31:12
2019-01-16 15:36:07
logs/parallel_cal-10985.casa
2019-01-16 15:36:34
2019-01-16 15:39:51
logs/parallel_cal_apply-10986.casa
2019-01-16 15:40:28
2019-01-16 15:43:52
```

##### 20. Run `./findErrors.sh`

```
logs/flag_round_1-10983.out
logs/run_setjy-10984.out
logs/parallel_cal-10985.out
logs/parallel_cal_apply-10986.out
*** Error *** Error in data selection specification: MSSelectionNullSelection : The selected table has zero rows.
(The same error repeated another 23 times)
```

This error likely corresponds to empty sub-MS(s) with data completely flagged out, which give a worker node nothing to do for whichever CASA tasks are being called.

##### 21. Rebuild your config file without verbose mode

`processMeerKAT.py -B -C tutorial_config.txt -M /data/projects/deep/1491550051.ms`

This way we reset the list of scripts in our config file, and set `verbose=False` and `submit=False`. We will manually remove the scripts we already ran in step 23, so leave the `scripts` parameter as is.

##### 22. Edit your config file

Edit `tutorial_config.txt` to update the reference antenna to what `validate_input.py` found as the best reference antenna. If you've forgotten what that was, view it in `jobScripts/tutorial_config_*.txt` (antenna 7). We don't need to update `badants` as only `flag_round_1.py` uses this parameter, which we will not be running.

##### 23. Run the pipeline using your updated config file

`processMeerKAT.py -R -C tutorial_config.txt`

Since we have set `verbose=False`, you should now see simplified output like the following:

`2019-01-16 20:30:00,180 INFO: Master script "submit_pipeline.sh" written, but will not run.`

##### 24. Edit `submit_pipeline.sh`

You will see in `submit_pipeline.sh` that each sbatch job is submitted on its own line, and that the job ID is extracted. Remove everything from `#validate_input.sbatch` to the line before `#flag_round_2.sbatch`, so it looks like the following

```
#flag_round_2.sbatch
IDs=$(sbatch flag_round_2.sbatch | cut -d ' ' -f4)

#run_setjy.sbatch
IDs+=,$(sbatch -d afterok:$IDs run_setjy.sbatch | cut -d ' ' -f4)

#cross_cal.sbatch
IDs+=,$(sbatch -d afterok:$IDs cross_cal.sbatch | cut -d ' ' -f4)

#cross_cal_apply.sbatch
IDs+=,$(sbatch -d afterok:$IDs cross_cal_apply.sbatch | cut -d ' ' -f4)

#split.sbatch
IDs+=,$(sbatch -d afterok:$IDs split.sbatch | cut -d ' ' -f4)

#plot_solutions.sbatch
IDs+=,$(sbatch -d afterok:$IDs plot_solutions.sbatch | cut -d ' ' -f4)

#Output message and create jobScripts directory
echo Submitted sbatch jobs with following IDs: $IDs
mkdir -p jobScripts
	.
	.
	.
```

**Note the first line has been edited to replace `+=,` with `,` and remove `-d afterok:$IDs`, since it should not have any dependencies.**

##### 25. Run `./submit_pipeline.sh`

Again, we see simplified output

```
Copying tutorial_config.txt to .config.tmp, and using this to run pipeline.
Submitted sbatch jobs with following IDs: 11000,11001,11002,11003,11004,11005
```

These job IDs comprise the new pipeline run we've just launched. So now `./summary.sh` will display `sacct` for the new job IDs, similar to the following:

```
       JobID    JobName  Partition    Account  AllocCPUS      State ExitCode
------------ ---------- ---------- ---------- ---------- ---------- --------
11000        flag_roun+       Main b03-pipel+        120    RUNNING      0:0
11000.0           orted            b03-pipel+         14    RUNNING      0:0
11001         run_setjy       Main b03-pipel+        120    PENDING      0:0
11002         cross_cal       Main b03-pipel+          1    PENDING      0:0
11003        cross_cal+       Main b03-pipel+        120    PENDING      0:0
11004             split       Main b03-pipel+        120    PENDING      0:0
11005        plot_solu+       Main b03-pipel+          1    PENDING      0:0
```

The other 3 job script will also correspond to these 6 new job IDs. After this pipeline run has completed, viewing the output of `./displayTimes.sh` shows this run took XX minutes.

If you want to see the output from the job scripts referring to the old pipeline runs, don't worry, they're still in the `jobScripts` directory with an older timestamp in the filename. Only the symlink in your working directory has been updated.

These new tasks follow the second step of a two step calibration process that is summarised on our wiki page [here](https://github.com/idia-astro/pipelines/wiki/Calibration-in-processMeerKAT).

After `split.py` has run, you will see three new files

`1491550051.0252-712.mms 1491550051.0408-65.mms 1491550051.DEEP_2_off.mms`

This corresponds to the data split out from `1491550051.mms`, for the phase calibrators (field IDs 1 and 2), and the science target (field ID 3). You can now go ahead and image these with `tclean`.

##### 26. View the figures in `plots` directory

The last script that runs is `plot_solutions.py`, which calls CASA task `plotms` to plot the calibration solutions for the bandpass calibrator and the phase calibrator. This is the reason that `xvfb-run` is called, which runs a virtual X server to make use of the plotting libraries. Your plots should look like the following.

[[https://github.com/idia-astro/pipelines/blob/jordan-dev/plots/bpass_chan_amp.png|alt=octocat]]
[[https://github.com/idia-astro/pipelines/blob/jordan-dev/plots/bpass_chan_phase.png|alt=octocat]]
[[https://github.com/idia-astro/pipelines/blob/jordan-dev/plots/bpass_real_imag.png|alt=octocat]]
[[https://github.com/idia-astro/pipelines/blob/jordan-dev/plots/phasecal_time_amp.png|alt=octocat]]
[[https://github.com/idia-astro/pipelines/blob/jordan-dev/plots/phasecal_time_phase.png|alt=octocat]]

**That's it! You have completed the tutorial! Now go forth and do some phenomenal MeerKAT science!**

### Also see

- [Calibration in processMeerKAT](https://github.com/idia-astro/pipelines/wiki/Calibration-in-processMeerKAT)
- [SLURM and MPICASA](https://github.com/idia-astro/pipelines/wiki/SLURM-and-MPICASA)
- [Using the pipeline](https://github.com/idia-astro/pipelines/wiki/Using-the-pipeline)


