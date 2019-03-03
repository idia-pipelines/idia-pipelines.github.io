---
layout: default
title: DEEP 2 Tutorial
parent: processMeerKAT
nav_order: 6
---

# DEEP 2 Tutorial

### This tutorial walks you through running the various steps of the pipeline for a single DEEP 2 dataset, which is a snapshot (~20 minutes on source), 16-dish MeerKAT observation of a random patch of sky using the old ROACH-2 correlator, 11 GB in size.

To begin, ssh into the ilifu cluster (`slurm.ilifu.ac.za`), and create a working directory somewhere on the filesystem (e.g. `/scratch/users/your_username/tutorial/`).

##### 1. Source `setup.sh`, which will add to your PATH and PYTHONPATH

```source /data/exp_soft/pipelines/master/setup.sh```

##### 2. Build a config file, using verbose mode, and pointing to the DEEP 2 dataset

```processMeerKAT.py -B -C tutorial_config.txt -M /data/projects/deep/1491550051.ms -v```

After some initial debug output, you should get the following output, with different timestamps

```
2019-02-28 02:36:14,421 INFO: Extracting field IDs from measurement set "/data/projects/deep/1491550051.ms" using CASA.
2019-02-28 02:36:14,422 DEBUG: Using the following command:
   srun --nodes=1 --ntasks=1 --time=10 --mem=4GB --partition=Main singularity exec /data/exp_soft/pipelines/casameer-5.4.1.xvfb.simg  casa --nologger --nogui --nologfile -c /data/exp_soft/pipelines/master/processMeerKAT/cal_scripts/get_fields.py -B -M /data/projects/deep/1491550051.ms -C tutorial_config.txt -N 8 -t 4
      .
      .
      .
2019-02-28 02:37:39,788 WARNING: The number of threads (8 node(s) x 4 task(s) = 32) is not ideal compared to the number of scans (12) for "/data/projects/deep/1491550051.ms".
2019-02-28 02:37:39,788 WARNING: Config file has been updated to use 2 node(s) and 4 task(s) per node.
2019-02-28 02:37:39,788 INFO: For the best results, update your config file so that nodes x tasks per node = 7.
2019-02-28 02:37:40,045 INFO: Multiple fields found with intent "CALIBRATE_FLUX" in dataset "/data/projects/deep/1491550051.ms" - [0 1].
2019-02-28 02:37:40,110 WARNING: Only using field "0" for "fluxfield", which has the most scans (1).
2019-02-28 02:37:40,110 WARNING: Putting extra fields with intent "CALIBRATE_FLUX" in "targetfields" - [1]
2019-02-28 02:37:40,111 INFO: Multiple fields found with intent "CALIBRATE_BANDPASS" in dataset "/data/projects/deep/1491550051.ms" - [0 1].
2019-02-28 02:37:40,111 WARNING: Only using field "0" for "bpassfield", which has the most scans (1).
2019-02-28 02:37:40,112 INFO: Multiple fields found with intent "CALIBRATE_PHASE" in dataset "/data/projects/deep/1491550051.ms" - [1 2].
2019-02-28 02:37:40,112 WARNING: Only using field "2" for "phasecalfield", which has the most scans (5).
2019-02-28 02:37:40,123 INFO: [fields] section written to "tutorial_config.txt". Edit this section to change field IDs (comma-seperated string for multiple IDs).
2019-02-28 02:37:41,990 INFO: Config "tutorial_config.txt" generated.
```

This calls CASA via the default singularity container without writing log files, and runs `get_fields.py`. It calls `srun`, requesting only 1 node, 1 task, 4 GB of memory, and a 10 minute time limit, to increase the likelihood of jumping to the top of the queue. The purpose of this call is to extract the field IDs corresponding to our different targets, and check the nodes and tasks per node against the number of scans, each of which is handled by a thread (see section 3). The output statements with `DEBUG` correspond to those output during verbose mode. The warnings display when multiple fields are present with the same intent, but only one is extracted, corresponding to the field with the most scans. In this case the extras are moved to `targetfields` (i.e. for applying calibration and imaging).

##### 3. View the config file created, which has the following contents:

```
[crosscal]
minbaselines = 4                  # Minimum number of baselines to use while calibrating
specavg = 1                       # Number of channels to average after calibration (during split)
timeavg = '8s'                    # Time interval to average after calibration (during split)
keepmms = True                    # Output MMS (True) or MS (False) during split
spw = '0:860~1700MHz'             # Spectral window / frequencies to extract for MMS
calcrefant = True                 # Calculate reference antenna in program (overwrites 'refant')
refant = 'm005'                   # Reference antenna name / number
standard = 'Perley-Butler 2010'   # Flux density standard for setjy
badants = []                      # List of bad antenna numbers (to flag)
badfreqranges = [ '944~947MHz',   # List of bad frequency ranges (to flag)
        '1160~1310MHz',
        '1476~1611MHz',
        '1670~1700MHz']

[slurm]
nodes = 2
ntasks_per_node = 4
plane = 2
mem = 236
partition = 'Main'
time = '12:00:00'
submit = False
container = '/data/exp_soft/pipelines/casameer-5.4.1.xvfb.simg'
mpi_wrapper = '/data/exp_soft/pipelines/casa-prerelease-5.3.0-115.el7/bin/mpicasa'
name = ''
verbose = True
scripts = [('validate_input.py', False, ''), ('partition.py', True, ''), ('calc_refant.py', False, ''), ('flag_round_1.py', True, ''), ('setjy.py', True, ''), ('xx_yy_solve.py', False, ''), ('xx_yy_apply.py', True, ''), ('flag_round_2.py', True, ''), ('setjy.py', True, ''), ('xy_yx_solve.py', False, ''), ('xy_yx_apply.py', True, ''), ('split.py', True, ''), ('quick_tclean.py', True, ''), ('plot_solutions.py', False, '')]

[data]
vis = '/data/projects/deep/1491550051.ms'

[fields]
bpassfield = '0'
fluxfield = '0'
phasecalfield = '2'
targetfields = '3,1'
```

This config file contains four sections - crosscal, slurm, data, and fields. The fields IDs we just extracted, seen in section `[fields]`, correspond to field 0 for the bandpass calibrator, field 0 for the total flux calibrator, field 2 for the phase calibrator, and fields 3 and 1 for the science targets (i.e. the DEEP 2 field + another calibrator). Only the target may have multiple fields. If a field isn't found according to its intent, a warning is displayed, and the field for the total flux calibrator is selected. If the total flux calibrator isn't present, the program will display an error and terminate.

The SLURM parameters in section `[slurm]` correspond to those seen by running `processMeerKAT.py -h`. By default, for all threadsafe scripts (i.e. those with `True` in the `scripts` list), we use 8 nodes, 4 tasks per node (=32 threads), 236 GB of memory (per node), and `plane=2` (which distributes four tasks onto one node before moving onto next node). During step 2, only 12 scans were found, and since `partition.py` partitions the data into one sub-measurement set (sub-MS) per scan, only 12 sub-MSs will exist in the multi-measurement set (see [section 20](#20-run-displaytimessh)). Assuming each observation has a phase calibrator bracketing each target scan, at most, 6 sub-MSs will be operated on at any given time, each handled by one thread, and a master thread. So we aim to have a limit of nscans+1+10% threads, with the 10% to account for the occasional thread that hangs. For this dataset, this limit is 7 threads, so `get_fields.py` attempts to match this number by using the specified number of tasks per node and increasing the node count from 1 until the number of threads is more than the limit, terminating at 2 nodes x 4 tasks per node = 8 threads.

For script that aren't threadsafe (i.e. those with `False` in the `scripts` list), we use a single node, and a single task per node. For both scripts that are threadsafe and those that aren't, we use a single CPU per task, and explicitly export `OMP_NUM_THREADS=1`, since there is little evidence of a speedup with more than one CPU per task. However, for `quick_tclean.py` we use 4 CPUs per task.

The cross-calibration parameters in section `[crosscal]` correspond to various CASA parameters passed into the calibration tasks that the pipeline used, each of which is documented [here](../Calibration-in-processMeerKAT). By default all frequency ranges listed in `badfreqranges`, and all antenna numbers listed in `badants`, will be flagged out entirely. The third script the pipeline runs (`calc_refant.py`) will likely change the value of `refant`, and add a list of bad antennas to `badant`.

##### 4. Run the pipeline using your config file

```processMeerKAT.py -R -C tutorial_config.txt```

You should get the following output, with different timestamps

```
2019-02-28 02:44:06,078 DEBUG: Copying 'tutorial_config.txt' to '.config.tmp', and using this to run pipeline.
2019-02-28 02:44:06,096 WARNING: Changing [slurm] section in your config will have no effect unless you [-R --run] again
2019-02-28 02:44:06,131 DEBUG: Wrote sbatch file "validate_input.sbatch"
2019-02-28 02:44:06,138 DEBUG: Wrote sbatch file "partition.sbatch"
2019-02-28 02:44:06,144 DEBUG: Wrote sbatch file "calc_refant.sbatch"
2019-02-28 02:44:06,150 DEBUG: Wrote sbatch file "flag_round_1.sbatch"
2019-02-28 02:44:06,156 DEBUG: Wrote sbatch file "setjy.sbatch"
2019-02-28 02:44:06,172 DEBUG: Wrote sbatch file "xx_yy_solve.sbatch"
2019-02-28 02:44:06,247 DEBUG: Wrote sbatch file "xx_yy_apply.sbatch"
2019-02-28 02:44:06,253 DEBUG: Wrote sbatch file "flag_round_2.sbatch"
2019-02-28 02:44:06,260 DEBUG: Wrote sbatch file "setjy.sbatch"
2019-02-28 02:44:06,267 DEBUG: Wrote sbatch file "xy_yx_solve.sbatch"
2019-02-28 02:44:06,273 DEBUG: Wrote sbatch file "xy_yx_apply.sbatch"
2019-02-28 02:44:06,279 DEBUG: Wrote sbatch file "split.sbatch"
2019-02-28 02:44:06,291 DEBUG: Wrote sbatch file "quick_tclean.sbatch"
2019-02-28 02:44:06,331 DEBUG: Wrote sbatch file "plot_solutions.sbatch"
2019-02-28 02:44:06,338 INFO: Master script "submit_pipeline.sh" written, but will not run.
```

A number of sbatch files have now been written to your working directory, each of which corresponds to the python script in the list of scripts set by the `scripts` parameter in our config file. Our config file was copied to `.config.tmp`, which is the config file written and edited by the pipeline, which the user should not touch. A bash script called `submit_pipeline.sh` was written, which we will look at soon. However, this script was not run, since we set `submit = False` in our config file (you can change this in your config file, or by using option `[-s --submit]` when you build your config file with `processMeerKAT.py`). Lastly, a `logs` directory was created, which will store the log files from the SLURM output.

##### 5. View `validate_input.sbatch`, which has the following contents:

```
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=1
#SBATCH --mem=100GB
#SBATCH --job-name=validate_input
#SBATCH --distribution=plane=1
#SBATCH --output=logs/validate_input-%j.out
#SBATCH --error=logs/validate_input-%j.err
#SBATCH --partition=Main
#SBATCH --time=12:00:00

export OMP_NUM_THREADS=1

srun singularity exec /data/exp_soft/pipelines/casameer-5.4.1.xvfb.simg  casa --nologger --nogui --logfile logs/validate_input-${SLURM_JOB_ID}.casa -c /data/exp_soft/pipelines/master/processMeerKAT/cal_scripts/validate_input.py --config .config.tmp
```

Since this script is not threadsafe, the job is called with `srun`, and is configured to run a single task on a single node, with 100 GB of memory. The last line shows the CASA call of the `validate_input.py` task, which will validate the parameters in the config file.

##### 6. Run the first sbatch job

```sbatch validate_input.sbatch```

You should see the following output, corresponding to your SLURM job ID

```Submitted batch job 1097914```

##### 7. View your job in the SLURM queue

`squeue`

You will see something similar to the following, with other people's jobs mixed into the queue.

```
  JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
1097914      Main validate jcollier  R       0:13      1 slwrk-008
```

We can see the job with name `validate` was submitted to SLURM worker node 8, amongst a number of jobs in the main partition, the JupyterSpawner partition, and possible other partitions. Your job may list `(Priority)`, which means it is too low a priority to be run at this point, or `(Resources)`, which means it is waiting for resources to be made available.

*NOTE: You can view just your jobs with `squeue -u your_username`, an individual job with `squeue -j 1097914`, and just the jobs in the main partition with `squeue -p Main`. You can view which nodes are allocated, which are idle, which are mixed (i.e. partially allocated), and which are down in the main partition with `sinfo -p Main`. Often it is good idea to check this before selecting your SLURM parameters.*


##### 8. View `partition.sbatch`, which has the following contents:

```
#!/bin/bash
#SBATCH --nodes=2
#SBATCH --ntasks-per-node=4
#SBATCH --cpus-per-task=1
#SBATCH --mem=236GB
#SBATCH --job-name=partition
#SBATCH --distribution=plane=2
#SBATCH --output=logs/partition-%j.out
#SBATCH --error=logs/partition-%j.err
#SBATCH --partition=Main
#SBATCH --time=12:00:00

export OMP_NUM_THREADS=1

/data/exp_soft/pipelines/casa-prerelease-5.3.0-115.el7/bin/mpicasa singularity exec /data/exp_soft/pipelines/casameer-5.4.1.xvfb.simg  casa --nologger --nogui --logfile logs/partition-${SLURM_JOB_ID}.casa -c /data/exp_soft/pipelines/master/processMeerKAT/cal_scripts/partition.py --config .config.tmp
```

Here we see the same default SLURM parameters for threadsafe tasks, as discussed in section 3. We now use mpicasa as the mpi wrapper, since we are calling a threadsafe script `partition.py`, which calls CASA task `partition`, which partitions the data into several sub measurement sets (sub-MSs - see [section 14 below](#10-view-the-contents-of-1491550051mms)) and selects only frequencies specified by your spectral window with parameter `spw` in your config file.

##### 9. Submit your job and watch it in the queue

```
sbatch partition.sbatch
Submitted batch job 1097917
squeue -j 1097917
```

You will see something similar to the following, showing that 2 nodes are now being used (worker nodes 60 & 61).

```
  JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
1097917      Main partitio jcollier  R       0:22      2 slwrk-[060-061]
```

Wait until the job completes, before step 10.

##### 10. View the contents of `1491550051.mms`.

You should see two new files - `1491550051.mms` and `1491550051.mms.flagversions`, which corresponds to the data and flag data of a multi-measurement set (MMS). From now on, the pipeline operates on these data, rather than the . The same path to the original MS will remain in your config file under section `[data]`, but each task will point to your MMS.

Inside this MMS, you will find the same tables and metadata as in a normal MS, but you will also see a `SUBMSS` directory, which should have the following contents.

```
1491550051.mms.0000.ms  1491550051.mms.0004.ms  1491550051.mms.0008.ms
1491550051.mms.0001.ms  1491550051.mms.0005.ms  1491550051.mms.0009.ms
1491550051.mms.0002.ms  1491550051.mms.0006.ms  1491550051.mms.0010.ms
1491550051.mms.0003.ms  1491550051.mms.0007.ms  1491550051.mms.0011.ms
```

These are the 12 sub-MSs, partitioned by this observation's 12 scans of the various targets.

If we now view the CASA log, you will find a bunch of junk output from mpicasa (often including nominal "errors", sometimes severe), and 13 calls of `partition`, corresponding to 12 workers for your 12 sub-MSs, and one master process. Similarly, your standard output logs will contains 8 sets of output from CASA launching, corresponding to the 8 threads (i.e. 2 nodes x 4 tasks per node) and some junk output from mpicasa. Again, your standard error log should be empty.


##### 11. Run `calc_refant.sbatch` and watch your submitted job

```
sbatch calc_refant.sbatch
watch sacct
```

You will initially see something similar to the following

```
       JobID    JobName  Partition    Account  AllocCPUS      State ExitCode
------------ ---------- ---------- ---------- ---------- ---------- --------
1097917       partition       Main b03-pipel+          8  COMPLETED      0:0
1097917.bat+      batch            b03-pipel+          4  COMPLETED      0:0
1097917.0         orted            b03-pipel+          1  COMPLETED      0:0
1097918      calc_refa+       Main b03-pipel+          1    RUNNING      0:0
1097918.0    singulari+            b03-pipel+          1    RUNNING      0:0
```

`sacct` lists all recently submitted jobs and their status. If your job fails, it will list `FAILED` under `State`. However, please note jobs running `mpicasa` often state they have failed when they haven't. Similarly, when jobs do genuinely fail, the pipeline may continue to run. Both of these are issues we are working to overcome.

When your job completes, you will see something similar to the following

```
       JobID    JobName  Partition    Account  AllocCPUS      State ExitCode
------------ ---------- ---------- ---------- ---------- ---------- --------
1097917       partition       Main b03-pipel+          8  COMPLETED      0:0
1097917.bat+      batch            b03-pipel+          4  COMPLETED      0:0
1097917.0         orted            b03-pipel+          1  COMPLETED      0:0
1097918      calc_refa+       Main b03-pipel+          1  COMPLETED      0:0
1097918.bat+      batch            b03-pipel+          1  COMPLETED      0:0
1097918.0    singulari+            b03-pipel+          1  COMPLETED      0:0
```

Control-C to exit out of `watch`.

##### 12. View the contents of your `logs` directory

```
ls logs
calc_refant-1097918.casa  calc_refant-1097918.err  calc_refant-1097918.out
partition-1097917.casa  partition-1097917.err  partition-1097917.out
validate_input-1097914.casa  validate_input-1097914.err  validate_input-1097914.out
```

As specified in our sbatch file, standard out is written to `logs/calc_refant-1097918.out`, standard error is written to `logs/calc_refant-1097918.err`, and the CASA logs are written to `logs/calc_refant-1097918.casa`. Your standard output log and CASA log will have little of interest, but your standard error log will contain the following output:

```
2019-02-28 03:02:33,870 INFO: Flux field scan no: 1
2019-02-28 03:02:34,034 INFO: Antenna statistics on total flux calibrator
2019-02-28 03:02:34,035 INFO: (flux in Jy averaged over scans & channels, and over all of each antenna's baselines)
2019-02-28 03:02:34,035 INFO: ant median rms
2019-02-28 03:03:10,900 INFO: All 8.92  275.02
2019-02-28 03:03:10,900 INFO: 7   8.10  199.94 (best antenna)
2019-02-28 03:03:10,900 INFO: 0   8.84  305.28 (1st good antenna)
2019-02-28 03:03:10,900 INFO: setting reference antenna to: 7
2019-02-28 03:03:10,900 INFO: Bad antennas: [5, 15]
```

Here we see `calc_refant.py` has selected antenna 7 as the best reference antenna, which measures comparable amplitude for the total flux calibrator compared to antenna 0, but a lower RMS. It has also found antennas 5 and 15 to be bad enough to flag out.

##### 13. View `ant_stats.txt` and `.config.tmp`

You should see the following contents in `ant_stats.txt`, corresponding to the amplitude and RMS each of the antennas measure

```
ant median rms
0   8.84   305.28
1   7.80   251.12
2   7.69   279.64
3   8.13   219.86
4   9.37   322.45
5   7.10   253.62
6   7.95   254.35
7   8.10   199.94
8   8.21   326.82
9   10.15   306.20
10  9.00   258.92
11  11.57   270.17
12  10.55   270.40
13  13.38   396.25
14  13.66   434.22
15  243.71   569.51
```

You should now see `refant = 7` and `badants = [5, 15]` in `.config.tmp`.

##### 14. Edit your config file to run the next steps

Edit `tutorial_config.txt` to remove the first three and last seven tuples in the `scripts` parameter, so it looks like the following:

```
scripts = [('flag_round_1.py', True, ''), ('setjy.py', True, ''), ('xx_yy_solve.py', False, ''), ('xx_yy_apply.py', True, '')]
```

Replace `refant` and `badants` with what was found by `validate_input.py`, select the submit option, and update `vis` to the MMS, so it looks like the following:

```
[crosscal]
refant = 7
badants = [5, 15]
  .
  .
[slurm]
submit = True
  .
  .
[data]
vis = 1491550051.mms
```

##### 15. Run the pipeline using your config file

```processMeerKAT.py -R -C tutorial_config.txt```

You should see the following output, with different timestamps

```
2019-02-28 03:18:26,585 DEBUG: Copying 'tutorial_config.txt' to '.config.tmp', and using this to run pipeline.
2019-02-28 03:18:26,605 WARNING: Changing [slurm] section in your config will have no effect unless you [-R --run] again
2019-02-28 03:18:26,615 DEBUG: Wrote sbatch file "flag_round_1.sbatch"
2019-02-28 03:18:26,624 DEBUG: Wrote sbatch file "setjy.sbatch"
2019-02-28 03:18:26,631 DEBUG: Wrote sbatch file "xx_yy_solve.sbatch"
2019-02-28 03:18:26,639 DEBUG: Wrote sbatch file "xx_yy_apply.sbatch"
2019-02-28 03:18:26,647 INFO: Running master script "submit_pipeline.sh"
Copying tutorial_config.txt to .config.tmp, and using this to run pipeline.
Submitting flag_round_1.sbatch SLURM queue with following command:
sbatch flag_round_1.sbatch
Submitting setjy.sbatch SLURM queue with following command
sbatch -d afterok:1097919 --kill-on-invalid-dep=yes setjy.sbatch
Submitting xx_yy_solve.sbatch SLURM queue with following command
sbatch -d afterok:1097919,1097920 --kill-on-invalid-dep=yes xx_yy_solve.sbatch
Submitting xx_yy_apply.sbatch SLURM queue with following command
sbatch -d afterok:1097919,1097920,1097921 --kill-on-invalid-dep=yes xx_yy_apply.sbatch
Submitted sbatch jobs with following IDs: 1097919,1097920,1097921,1097922
Run killJobs.sh to kill all the jobs.
Run summary.sh to view the progress.
Run findErrors.sh to find errors (after pipeline has run).
Run displayTimes.sh to display start and end timestamps (after pipeline has run).
```

As before, we see the sbatch files being written to our working directory. Since we set `submit=True`, `submit_pipeline.sh` has been run, and all output after that (without the timestamps) comes from this bash script. After the first job is run (`sbatch flag_round_1.sbatch`), each other job is run with a dependency on all previous jobs (e.g. `sbatch -d afterok:1097919,1097920,1097921 --kill-on-invalid-dep=yes xx_yy_apply.sbatch`). We can see this by calling `squeue -u your_username`, which shows those jobs `(Dependency)`. `submit_pipeline.sh` then writes four job scripts, all of which are explained in the output, written to `jobScripts` with a timestamp appended to the filename, and symlinked from your working directory. `findErrors.sh` finds errors after this pipeline run has completed, overlooking all nominal MPI errors.

These tasks follow the first step of a two-step calibration process that is summarised [here](../Calibration-in-processMeerKAT).

##### 16. Run `./summary.sh`

This script simply calls `sacct` for all jobs submitted within this pipeline run. You should get output similar to the following.

```
          JobID         JobName  Partition    Elapsed NNodes NTasks NCPUS  MaxDiskRead MaxDiskWrite             NodeList   TotalCPU      State ExitCode
--------------- --------------- ---------- ---------- ------ ------ ----- ------------ ------------ -------------------- ---------- ---------- --------
1097927         flag_round_1          Main   00:00:04      2            8                                slwrk-[060-061]   00:00:00    RUNNING      0:0
1097927.0       orted                        00:00:04      1      1     1                                      slwrk-061   00:00:00    RUNNING      0:0
1097928         setjy                 Main   00:00:00      2            8                                  None assigned   00:00:00    PENDING      0:0
1097929         xx_yy_solve           Main   00:00:00      1            1                                  None assigned   00:00:00    PENDING      0:0
1097930         xx_yy_apply           Main   00:00:00      2            8                                  None assigned   00:00:00    PENDING      0:0
```

Those `PENDING` are the jobs with dependencies. Once this pipeline run has completed, `./summary.sh` should give output similar to the following.


```
          JobID         JobName  Partition    Elapsed NNodes NTasks NCPUS  MaxDiskRead MaxDiskWrite             NodeList   TotalCPU      State ExitCode
--------------- --------------- ---------- ---------- ------ ------ ----- ------------ ------------ -------------------- ---------- ---------- --------
1097927         flag_round_1          Main   00:08:25      2            8                                slwrk-[060-061]  11:04.530  COMPLETED      0:0
1097927.batch   batch                        00:08:25      1      1     4       11759M        2971M            slwrk-060  04:46.830  COMPLETED      0:0
1097927.0       orted                        00:08:25      1      1     1       14493M        1084M            slwrk-061  06:17.699  COMPLETED      0:0
1097928         setjy                 Main   00:06:18      2            8                                slwrk-[060-061]  03:21.451  COMPLETED      0:0
1097928.batch   batch                        00:06:18      1      1     4       24722M       24575M            slwrk-060  02:23.206  COMPLETED      0:0
1097928.0       orted                        00:06:17      1      1     1        5779M        5655M            slwrk-061  00:58.245  COMPLETED      0:0
1097929         xx_yy_solve           Main   00:03:28      1            1                                      slwrk-066  01:59.767  COMPLETED      0:0
1097929.batch   batch                        00:03:28      1      1     1        0.22M        0.18M            slwrk-066  00:00.068  COMPLETED      0:0
1097929.0       singularity                  00:03:28      1      1     1       10275M           5M            slwrk-066  01:59.699  COMPLETED      0:0
1097930         xx_yy_apply           Main   00:06:30      2            8                                slwrk-[060-061]  03:32.520  COMPLETED      0:0
1097930.batch   batch                        00:06:30      1      1     4        8237M        4228M            slwrk-060  01:29.291  COMPLETED      0:0
1097930.0       orted                        00:06:29      1      1     1       13386M        6554M            slwrk-061  02:03.228  COMPLETED      0:0
```

##### 17. View `caltables` directory

The calibration solution tables have been written to `caltables/1491550051.*`, including `bcal, gcal, fluxscale` and `kcal`, corresponding to the calibration solutions for bandpass, complex gains, flux-scaled complex gains, and delays, respectively.

##### 18. Run `./displayTimes.sh`

You should see output similar to the following, which shows this run took ~24 minutes to complete, the longest of which was flagging for ~8 minutes.

```
logs/flag_round_1-1097927.casa
2019-02-28 01:32:37
2019-02-28 01:40:32
logs/setjy-1097928.casa
2019-02-28 01:41:02
2019-02-28 01:46:49
logs/xx_yy_solve-1097929.casa
2019-02-28 01:47:15
2019-02-28 01:50:22
logs/xx_yy_apply-1097930.casa
2019-02-28 01:50:47
2019-02-28 01:56:49
```

##### 19. Run `./findErrors.sh`

```
logs/flag_round_1-1097927.out
logs/setjy-1097928.out
logs/xx_yy_solve-1097929.out
logs/xx_yy_apply-1097930.out
*** Error *** Error in data selection specification: MSSelectionNullSelection : The selected table has zero rows.
(The same error repeated another 23 times)
```

This error likely corresponds to empty sub-MS(s) with data completely flagged out, which give a worker node nothing to do for whichever CASA tasks are being called.

##### 20. Rebuild your config file without verbose mode

`processMeerKAT.py -B -C tutorial_config.txt -M 1491550051.mms`

This way we reset the list of scripts in our config file, and set `verbose=False` and `submit=False`. We will manually remove the scripts we already ran in [step 24](#24-run-the-pipeline-using-your-updated-config-file), so leave the `scripts` parameter as is.

##### 21. Edit your config file

Edit `tutorial_config.txt` to update the reference antenna to what `calc_refant.py` found as the best reference antenna. If you've forgotten that was, view it in `jobScripts/tutorial_config_*.txt` (antenna 7). We don't need to update `badants` as only `flag_round_1.py` uses this parameter, which we will not be running.

##### 22. Run the pipeline using your updated config file

`processMeerKAT.py -R -C tutorial_config.txt`

Since we have set `verbose=False` and `submit=False`, the pipeline will not yet run, and you should see simplified output like the following:

```
2019-02-28 11:47:20,011 WARNING: Changing [slurm] section in your config will have no effect unless you [-R --run] again.
2019-01-16 20:30:00,180 INFO: Master script "submit_pipeline.sh" written, but will not run.
```

##### 23. Edit `submit_pipeline.sh`

You will see in `submit_pipeline.sh` that each sbatch job is submitted on its own line, and that the job ID is extracted. Remove everything from `#validate_input.sbatch` to one line before `#flag_round_2.sbatch`, so it looks like the following

```
#!/bin/bash
cp tutorial_config.txt .config.tmp

#flag_round_2.sbatch
IDs=$(sbatch flag_round_2.sbatch | cut -d ' ' -f4)

#setjy.sbatch
IDs+=,$(sbatch -d afterok:$IDs --kill-on-invalid-dep=yes setjy.sbatch | cut -d ' ' -f4)

#xy_yx_solve.sbatch
IDs+=,$(sbatch -d afterok:$IDs --kill-on-invalid-dep=yes xy_yx_solve.sbatch | cut -d ' ' -f4)

#xy_yx_apply.sbatch
IDs+=,$(sbatch -d afterok:$IDs --kill-on-invalid-dep=yes xy_yx_apply.sbatch | cut -d ' ' -f4)

#split.sbatch
IDs+=,$(sbatch -d afterok:$IDs --kill-on-invalid-dep=yes split.sbatch | cut -d ' ' -f4)

#quick_tclean.sbatch
IDs+=,$(sbatch -d afterok:$IDs --kill-on-invalid-dep=yes quick_tclean.sbatch | cut -d ' ' -f4)

#plot_solutions.sbatch
IDs+=,$(sbatch -d afterok:$IDs --kill-on-invalid-dep=yes plot_solutions.sbatch | cut -d ' ' -f4)

#Output message and create jobScripts directory
echo Submitted sbatch jobs with following IDs: $IDs
mkdir -p jobScripts
   .
   .
   .
```

**Note the first line has been edited to replace `+=,` with `=` and remove `-d afterok:$IDs --kill-on-invalid-dep=yes`, since it does not have any dependencies.**

##### 24. Run `./submit_pipeline.sh`

Again, we see simplified output

```
Submitted sbatch jobs with following IDs: 1097948,1097949,1097950,1097951,1097952,1097953,1097954
Run killJobs.sh to kill all the jobs.
Run summary.sh to view the progress.
Run findErrors.sh to find errors (after pipeline has run).
Run displayTimes.sh to display start and end timestamps (after pipeline has run).
```

These job IDs comprise the new pipeline run we've just launched. So now `./summary.sh` will display `sacct` for the new job IDs, similar to the following:

```
          JobID         JobName  Partition    Elapsed NNodes NTasks NCPUS  MaxDiskRead MaxDiskWrite             NodeList   TotalCPU      State ExitCode
--------------- --------------- ---------- ---------- ------ ------ ----- ------------ ------------ -------------------- ---------- ---------- --------
1097948         flag_round_2          Main   00:01:49      2            8                                slwrk-[013,020]   00:00:00    RUNNING      0:0
1097948.0       orted                        00:01:45      1      1     1                                      slwrk-020   00:00:00    RUNNING      0:0
1097949         setjy                 Main   00:00:00      2            8                                  None assigned   00:00:00    PENDING      0:0
1097950         xy_yx_solve           Main   00:00:00      1            1                                  None assigned   00:00:00    PENDING      0:0
1097951         xy_yx_apply           Main   00:00:00      2            8                                  None assigned   00:00:00    PENDING      0:0
1097952         split                 Main   00:00:00      2            8                                  None assigned   00:00:00    PENDING      0:0
1097953         quick_tclean          Main   00:00:00      2           64                                  None assigned   00:00:00    PENDING      0:0
1097954         plot_solutions        Main   00:00:00      1            1                                  None assigned   00:00:00    PENDING      0:0
```

The 4 new ancillary (bash) jobScripts will also correspond to these 7 new job IDs. If you want to see the output from the job scripts referring to the old pipeline runs, don't worry, they're still in the `jobScripts` directory with an older timestamp in the filename. Only the symlink in your working directory has been updated.

Wait until the run finishes before step 25. You may want to come back later, as it takes ~1.5 hours.

##### 25. View the pipeline output

After this pipeline run has completed, viewing the output of `./displayTimes.sh` shows this run took ~1.5 hours, including ~25 minutes for quick-look imaging all fields, and ~40 minutes for plotting (a [known issue](../Release-Notes#known-issues)).

These new tasks follow the second step of a two step calibration process that is summarised on [this page](../Calibration-in-processMeerKAT).

After `split.py` has run, you will see three new files

`1491550051.0252-712.mms 1491550051.0408-65.mms 1491550051.DEEP_2_off.mms`

This corresponds to the data split out from `1491550051.mms`, for the bandpass/flux calibrator (`0408-65`), the phase calibrator (`0252-712`), and the science target (`DEEP_2_off`).

##### 26. View the images in the `images` directory

`quick_tclean.py` creates quick-look images (i.e. with no selfcal, w-projection, threadholding, no-multiscale, etc) with robust weighting 0, for all fields specified in the config file, creating 512x512 images of the calibrator fields, and 2048x2048 images of the target field(s), both with 2 arcsec pixel sizes. For data with > 100 MHz bandwidth, two taylor terms are used, otherwise the 'clark' deconvolver is used.

You can view the images by connecting to a fat node (e.g. `racetrack.idia.ac.za` - also ensure X-forwarding is enabled) and launching ds9 or CASA viewer, respectively with the syntax (replace `/scratch/users/your_username/tutorial` below):

```
singularity exec /data/exp_soft/containers/sourcefinding_py3.simg ds9 /scratch/users/your_username/tutorial/images/*fits
singularity exec /data/exp_soft/pipelines/casameer-5.4.1.xvfb.simg casa --nologger --log2term -c "viewer(infile='/scratch/users/your_username/tutorial/images/1491550051_DEEP_2_off.im.image.tt0/'); raw_input()"
```

Here's what your images of the flux calibrator (`1934-638`) and target (`DEEP_2_off`) should look like.

![DEEP2_image](https://idia-pipelines.github.io/assets/DEEP2_image.png)

Since we imaged a snapshot 16-dish MeerKAT observation using the old ROACH-2 correlator, with an on source time of ~20 minutes, we do not get very good image quality. Below is a more typical image produced by `quick_tclean.py` for a 64-dish observation using the SKARAB correlator, spanning ~8 hours, and only 5 MHz bandwidth.

![64-dish-image](https://idia-pipelines.github.io/assets/64-dish-image.png)

##### 27. View the figures in `plots` directory

The last script that runs is `plot_solutions.py`, which calls CASA task `plotms` to plot the calibration solutions for the bandpass calibrator and the phase calibrator, as well as plots of the corrected data to eyeball for RFI. Below are a few selected plots.

![bpass_freq_amp](https://idia-pipelines.github.io/assets/bpass_freq_amp.png)
![DEEP_2_off_freq_amp](https://idia-pipelines.github.io/assets/DEEP_2_off_freq_amp.png)
![DEEP_2_off_real_imag](https://idia-pipelines.github.io/assets/DEEP_2_off_real_imag.png)

**That's it! You have completed the tutorial! Now go forth and do some phenomenal MeerKAT science!**

### Also see

- [Calibration in processMeerKAT](../Calibration-in-processMeerKAT)
- [Diagnosing Errors](../Diagnosing-Errors)
- [Using the pipeline](../Using-the-pipeline)

