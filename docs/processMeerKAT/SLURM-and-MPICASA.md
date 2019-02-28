---
layout: default
title: SLURM and MPICASA
parent: processMeerKAT
nav_order: 8
---

# Parallel CASA Using SLURM at IDIA

SLURM is a resource and job management system that is available on many clusters. Jobs/tasks are typically submitted to the job management system, and are inserted into a job queue; the job is executed when the requested resources become available. SLURM is currently used with the IDIA cluster. Please consult the Appendix at the bottom of this page on the computing environments available at IDIA, and how to use Singularity containers.

While SLURM Clusters provide the option to request and reserve resources to work in an interactive mode, its preferred to submit jobs to the queue to be run in a non-interactive way.

To run a CASA script in a non-interactive way in the SLURM cluster, you would use the following steps.

1. Write your CASA script.
2. Write an associated SBATCH script for your job.
3. Submit the script (i.e., your job) to the queue using `sbatch`.

_**The image below illustrates these different steps.**_

![Basic step-by-step guide on to use SLURM and MPICASA to run a CASA Script.](https://d2mxuefqeaa7sj.cloudfront.net/s_EA37BF1003DCBCF5DA97C07EE87BCC9C3B03F3B2E4F25A26C4DD1191504B01C3_1530182750818_Screen+Shot+2018-06-28+at+12.45.22.png)

## Write your CASA Script
CASA scripts are written in Python. An entire pipeline can be written in such a script, that includes flagging, initial calibration and imaging.

It is important to note that different CASA tasks use different schemes for parallelism, when writing your script. For example, `flagdata` parallelises by scan and is thus RAM intensive; `tclean` parallelises by frequency and is thus CPU intensive.

Therefore, a single script that includes flagging and imaging could have sub-optimal usage of a cluster resources for some tasks, and optimal usage for others. Keep this in mind when writing your script.


    vis = 'blah'
    var1 = 'something'
    var2 = 1e-3
    casatask(vis=vis,
      var1=var1,
      var2=var2)

    casatask2(vis=vis)

## Write your SBATCH Script
An SBATCH script is a bash script that wraps the relevant SLURM parameters needed for your script. Consult the following website for more details on how to use SBATCH:
https://slurm.schedmd.com/sbatch.html

Here’s an example of an SBATCH script that submits a TCLEAN job:


    #!/bin/bash
    #SBATCH -N 4-4
    #SBATCH --tasks-per-node 48
    #SBATCH -J tclean
    #SBATCH -m plane=8
    #SBATCH -o casameer-batch-1530187312-tclean.sh.stdout.txt
    #SBATCH -e casameer-batch-1530187312-tclean.sh.stderr.txt
    #Run the application:
    /data/users/frank/casa-cluster/casa-prerelease-5.3.0-115.el7/bin/mpicasa /usr/bin/singularity exec ~/casameer.simg  "casa" --nologger --log2term --nogui -c tclean.py

Please consult our documentation/wiki pages for more details on how software and containers are used on the IDIA Cloud.

There are a few important SBATCH parameters to define:

- `--nodes` or `-N` specifies the node count, i.e., the nodes requested for the job.
- `--tasks-per-node` specifies the number of parallel tasks to execute on each node.
- `--mode` or `-m` specifies the mode in which the tasks are distributed to each node.
  - This parameter is useful for scripts that include flagging. Since flagging is parallelised by scan, the first node(s) could run out of RAM for a particular flagging job. This would lead to SLURM killing the offending task(s), hence killing the main job.
  - There are two distribution modes that can be used to solve this problem. `plane=X` distributes X jobs at a time, in a round-robin fashion across nodes. The `cyclic` mode distributes single tasks at a time in a round-robin fashion across nodes.
  - `plane=X` or `cyclic` modes are useful for jobs that are RAM limited, i.e., when you need to use the aggregated RAM that’s in the total pool requested.
- The `-J`, `-o` and `-e` parameters

### More about SLURM
[SLURM](https://slurm.schedmd.com/overview.html) is a workload manager that will distribute jobs across a specified cluster environment. It understands how to control an MPI-aware job via `mpirun` and hence also via `mpicasa`. In principle this means that SLURM should be able to schedule and manage a CASA job that is running across a cluster.

Following is a "practical" definition of some SLURM keywords that should help clarify how to best to allocate resources.

__task__ : A "task" by SLURM's definition is what one would usually call a "process" on a regular computer. Similar to a process, a task has its own memory allocation that it does not share with other tasks. Each task is then operated on independently via MPI. This also means a more _fine-grained_ parallelism can be employed per task, by using multiple threads (_e.g._ via openMP) to work on a single task.

__--cpus-per-task__ : Defines the number of CPUs to dedicate to a single task. If each task can take advantage of multiple threads, setting this value to more than one can speed things up further (_e.g.,_ `tclean` in CASA is parallelised across openMP and MPI)

__--mem-per-cpu__ : The RAM dedicated to each CPU in the node. At the moment, the IDIA cluster is set to 4096MB per CPU. If a job is running out of memory, setting this to a larger value can help. Alternatively, as mentioned above if the task can take advantage of more threads, it may be preferable to set `--cpus-per-task` instead.

__-m/--distribution__ : This controls how the tasks are allocated across the requested nodes. The sbatch `man` page has a very good explanation on the various modes available.



------

###  Notes on CASA Tasks and Parallelism

Running CASA through SLURM requires calling CASA via `mpicasa`. CASA understands how to use `mpi` on tasks that are optimised for `mpi` (such as `flagdata`, `tclean`, `setjy`, and `applycal`) while operating as per usual on tasks that are not `mpi` aware (like `gaincal`). Ideally, the only change to an existing script would be to add a call to `partition` at the top. Below are some notes on tasks.

__partition__: In order to run across a cluster the `partition` task needs to be called prior to running any other tasks. `partition`  [creates a multi-measurement set](https://casa.nrao.edu/docs/TaskRef/partition-task.html) (MMS) that is a collection of multiple SUBMS's, each of which will be operated upon as a task in SLURM. By default CASA will split the MS along the spectral window (`spw`) axis, and across scans. The number of SUBMSes created can be specified in `partition`, however it seems that specifying a number larger than what CASA would decide leads to some strangeness with the metadata (and a failure of tasks that operate on the MMS).


__tclean__: In order to run across a cluster, `parallel=True` should be specified in `tclean`. However, if `savemodel='modelcolumn'` is also specified, it triggers some kind of a race condition between the different nodes where they are competing for write access, and the task crashes. So setting `savemodel='virtual'` or `savemodel='none'` are the only options that work. Both the `makePSF` step and the minor cycles of deconvolution are openMP aware, and can exploit additional resources specified via `--cpus-per-task` in the SLURM `sbatch` file.