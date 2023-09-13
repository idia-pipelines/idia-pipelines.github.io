---
layout: default
title: SLURM and MPICASA
parent: processMeerKAT
nav_order: 12
---

# Parallel CASA Using SLURM at IDIA

**Note: For details on how to setup SLURM batch and interactive jobs on the ilifu system, please look at the [ilifu documentation](https://docs.ilifu.ac.za/#/getting_started/submit_job_slurm).**

This page deals with the specifics of using CASA in conjuction with the SLURM environment on the ilifu computing system.

SLURM is a resource and job management system that is available on many clusters. Jobs/tasks are typically submitted to the job management system, and are inserted into a job queue; the job is executed when the requested resources become available. SLURM is the job management and scheduling software used on the ilifu cluster.

While SLURM Clusters provide the option to request and reserve resources to work in an interactive mode, its preferred method is to submit jobs to the queue to be run in a non-interactive way.

To run a CASA script in a non-interactive way in the SLURM cluster, you would use the following steps.

1. Write your CASA script.
2. Write an associated SBATCH script for your job.
3. Submit the script (i.e., your job) to the queue using `sbatch`.

<!---

_**The image below illustrates these different steps.**_

![Basic step-by-step guide on to use SLURM and MPICASA to run a CASA Script.](/assets/slurm-and-mpicasa.png)

--->

## Write your CASA Script

CASA scripts are written in Python. An entire pipeline can be written in such a script, that includes flagging, initial calibration and imaging.

It is important to note that different CASA tasks use different schemes for parallelism, when writing your script. For example, `flagdata` parallelises by scan and is thus RAM intensive; `tclean` splits up the input data to occupy as many processes as are specified, and is this CPU & RAM intensive.

Therefore, a single script that includes flagging and imaging could have sub-optimal usage of a cluster resources for some tasks, and optimal usage for others. Keep this in mind when writing your script. Here's an example of a python script that calls tclean in CASA 6:

```
import os,sys
import casampi
from casatasks import tclean,casalog
logfile=casalog.logfile()
casalog.setlogfile('logs/{SLURM_JOB_NAME}-{SLURM_JOB_ID}.casa'.format(**os.environ))

vis=sys.argv[-1]
imagename=vis + '.image'

tclean(vis=vis, imagename=imagename, imsize=[6144,6144], cell='1.5arcsec', gridder='wproject', wprojplanes=512, deconvolver='mfmfs', weighting='briggs',robust=-0.5, niter=50000, threshold=10e-6, nterms=2, pblimit=-1, parallel = True)

if os.path.exists(logfile):
  os.rename(logfile,'logs/{SLURM_JOB_NAME}-{SLURM_JOB_ID}.mpi'.format(**os.environ))
```

## Write your SBATCH Script

An SBATCH script is a bash script that wraps the relevant SLURM parameters needed for your script. Consult the following website for more details on how to use SBATCH:
https://slurm.schedmd.com/sbatch.html

Here’s an example of an SBATCH script that submits a `tclean` job for CASA 6:

```
#!/bin/bash
#SBATCH --account=b05-pipelines-ag
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=32
#SBATCH --cpus-per-task=1
#SBATCH --mem=232GB
#SBATCH --job-name=tclean
#SBATCH --output=logs/%x-%j.out
#SBATCH --error=logs/%x-%j.err
#SBATCH --partition=Main
#SBATCH --time=03:00:00

#Run the application:
module load openmpi/4.0.3
mpirun singularity exec /idia/software/containers/casa-6.simg python tclean.py /path/to/data.ms
```

More details can be found in the ilifu documentation about [using Singularity](https://docs.ilifu.ac.za/#/getting_started/container_environments) and [parallel processing](https://docs.ilifu.ac.za/#/getting_started/submit_job_slurm?id=parallel-computing-on-the-cluster). Some other helpful resources include the [resource allocation guide](https://docs.ilifu.ac.za/#/tech_docs/resource_allocation) and the [ilifu training](https://www.ilifu.ac.za/latest-training).

There are a few important SBATCH parameters to define:

- `--nodes` or `-N` specifies the node count, i.e., the nodes requested for the job.
- `--tasks-per-node` specifies the number of parallel tasks to execute on each node.
- `--distribution` or `-m` specifies the mode in which the tasks are distributed to each node.
  - This parameter is useful for scripts that include flagging. Since flagging is parallelised by scan, the first node(s) could run out of RAM for a particular flagging job. This would lead to SLURM killing the offending task(s), hence killing the main job.
  - There are two distribution modes that can be used to solve this problem. `plane=X` distributes X jobs at a time, in a round-robin fashion across nodes. The `cyclic` mode distributes single tasks at a time in a round-robin fashion across nodes.
  - `plane=X` or `cyclic` modes are useful for jobs that are RAM limited, i.e., when you need to use the aggregated RAM that’s in the total pool requested.
<!-- - The `-J`, `-o` and `-e` parameters -->

### More about SLURM
[SLURM](https://slurm.schedmd.com/overview.html) is a workload manager that will distribute jobs across a specified cluster environment. It understands how to control message passing interface (MPI) jobs across multiple tasks (or nodes) via `mpirun` and for CASA 5, `mpicasa`. In principle this means that SLURM should be able to schedule and manage a CASA job that is running across a cluster.

Following is a "practical" definition of some SLURM keywords that should help clarify how to best to allocate resources.

__task__ : A "task" by SLURM's definition is what one would usually call a "process" on a regular computer. Similar to a process, a task has its own memory allocation that it does not share with other tasks. Each task is then operated on independently via MPI. This also means a more _fine-grained_ parallelism can be employed per task, by using multiple threads (_e.g._ via OpenMP) to work on a single task.

__--cpus-per-task__ : Defines the number of CPUs to dedicate to a single task. If each task can take advantage of multiple threads, setting this value to more than one can speed things up further (_e.g.,_ `tclean` in CASA is parallelised across OpenMP and MPI)

__--mem-per-cpu__ : The RAM dedicated to each CPU in the node. Currently, the ilifu cluster is set to 3096 MB per CPU. If a job is running out of memory, setting this to a larger value can help. Alternatively, as mentioned above, if the task can take advantage of more threads, it may be preferable to set `--cpus-per-task` instead.

<!-- __-m/--distribution__ : This controls how the tasks are allocated across the requested nodes. The sbatch `man` page has a very good explanation on the various modes available. -->



------

###  Notes on CASA Tasks and Parallelism

Running CASA through SLURM requires calling CASA via `mpirun`, or for CASA 5, `mpicasa`. CASA understands how to use `mpi` on tasks that are optimised for `mpi` (such as `flagdata`, `tclean`, `setjy`, and `applycal`) while operating as per usual on tasks that are not `mpi` aware (like `gaincal`). Ideally, the only change to an existing script would be to add a call to `partition` (or `mstransform(createmms=True)`) at the top. Below are some notes on tasks.

__partition__ (basic): In order to run tasks (except tclean) across a cluster, the `partition` (or `mstransform`) task needs to be called prior to running any other tasks. `partition`  [creates a multi-measurement set](https://casa.nrao.edu/casadocs/casa-5.4.1/uv-manipulation/data-partition) (MMS) that is a collection of multiple sub-MSs, each of which will be operated upon as a task in SLURM. By default, CASA will split the MS across scans, and a spectral window (`spw`) axis, given by the default `separationaxis='auto'` parameter. However, this partitioning of data gives a data format that does not appear to perform well during processing. A better partitioning of the data is given by simply partitioning by scan, given by `separationaxis='scan', numsubms=msmd.nscans()`

__mstransform__ (advanced): the `mstransform` task (called under the hood by `partition`) is better suited to partition an MS into an MMS, as it allows for more control via several additional parameters, such as averaging in time and frequency. For example, the following call averages by 8 frequency channels (from selected frequency range 880-1680 MHz) and 16 second integrations, as well as selecting only two (parallel-hand) correlations (controlled by one core each, given by `nthreads=2`), and no autocorrelations:

```
mstransform(vis=visname, outputvis=mvis, spw='*:880~1680MHz', createmms=True, datacolumn='DATA', chanaverage=True, chanbin=8, timeaverage=True, timebin='16s', separationaxis='scan', numsubms=msmd.nscans(), nthreads=2, antenna='*&', correlation='XX,YY')
```

<!--
The number of SUBMSes created can be specified in `partition`, however it seems that specifying a number larger than what CASA would decide leads to some strangeness with the metadata (and a failure of tasks that operate on the MMS).
-->

__tclean__: In order to run across a cluster, `parallel=True` should be specified in `tclean`. However, if `savemodel='modelcolumn'` is also specified, it triggers some kind of a race condition between the different nodes where they are competing for write access, and the task crashes. So setting `savemodel='virtual'` or `savemodel='none'` are the only options that work when running `tclean` in parallel. Both the `makePSF` step and the major cycles of deconvolution are MPI and OpenMP aware, and can exploit additional resources specified via `--cpus-per-task` in the SLURM `sbatch` file.
