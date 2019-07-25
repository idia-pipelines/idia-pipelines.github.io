---
layout: default
title: Diagnosing Errors
parent: processMeerKAT
nav_order: 5
---

# Diagnosing Errors

This is a list of _potential_ issues that could arise while running the pipeline, which are typically associated with runtime errors, which could be due to issues with the cluster (e.g., worker-node failure during your job) or job parameterisation (e.g., underestimating RAM required).

This page is, by no means, exhaustive, and will be updated as more issues are reported.

We note here that although the pipeline is constructed to have job dependencies, _i.e.,_ if one job fails, all the subsequent jobs are meant to be cancelled, this does not always work. We find that there are several cases where CASA will crash, and SLURM does not recognise that the job has terminated and will continue to execute the other jobs in the pipeline. We suspect that this is due to a difference in error handling between SLURM and CASA. Please keep a watch on the status of your jobs using the `./summary.sh` script, as well as the log files created to make sure that the pipeline is proceeding correctly.

### Fluxscale Issues

If any fluxscale issues are discovered within your dataset (see [known issues](https://idia-pipelines.github.io/docs/processMeerKAT/Release-Notes/#known-issues)), particularly for short-track observations, we recommend you replace the 'xy_yx' scripts in your config file with the 'xx_yy' scripts (see [example use case](https://idia-pipelines.github.io/docs/processMeerKAT/Example-Use-Cases#short-track-observations-and-fluxscale-issues)).

### ORTE error

An error of the form

      --------------------------------------------------------------------------

      ORTE has lost communication with its daemon located on node:


       hostname:  slwrk-067


      This is usually due to either a failure of the TCP network

      connection to the node, or possibly an internal failure of

      the daemon itself. We cannot recover from this failure, and

      therefore will terminate the job.


      --------------------------------------------------------------------------

      --------------------------------------------------------------------------

      An ORTE daemon has unexpectedly failed after launch and before

      communicating back to mpirun. This could be caused by a number

      of factors, including an inability to create a connection back

      to mpirun due to a lack of common network interfaces and/or no

      route found between them. Please check network connectivity

      (including firewalls and network routing requirements).

      --------------------------------------------------------------------------

where `slwrk-067` can be any of the SLURM worker nodes, typically means that there was a communication/network error between the SLURM head node and the worker node. Please report this issue to the ILIFU support team at support@ilifu.ac.za. This is indicative of an underlying hardware issue, rather than a software problem.


### Unable to launch jobs in SLURM

If the status of your job is `(launch failed requeued held)`, please file a ticket with support@ilifu.ac.za.

### Memory error

If you see the phrase `MemoryError` in the `stderr` logs (this can be located by `grep -i MemoryError logs/*.err`) this is typically indicative that CASA did not have enough memory to complete the task. This often happens while running `flagdata` and does not always halt execution of the pipeline. Reduce the number of tasks per node and increase the nodes in the config file (e.g. halve tasks and double nodes) before re-launching the pipeline, as that will allocate more memory per task.



