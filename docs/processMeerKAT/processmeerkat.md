---
layout: default
title: processMeerKAT
nav_order: 3
has_children: true
permalink: /docs/processMeerKAT
---

The `processMeerKAT` software has been written to do calibration and imaging of MeerKAT
interferometric data.

Typical calibration and imaging of radio data comprises three steps:
* Initial or *a priori* calibration, which involves bootstrapping phase and flux gains from observations of
  reference calibrators.
* Self-calibration or *a posteriori'* calibration, where the residual on-target errors are solved for
  by (iteratively) building a good representation of the field.
* 3rd Generation Calibration (3GC), aka post-selfcal, which deals with higher-order effects in the
  pursuit of high dynamic range imaging. This includes compensating for the primary beam and
  direction dependent effects.

The `processMeerKAT` currently does full-polarisation *a priori* calibration on MeerKAT data, and includes automated
flagging. `processMeerKAT` is written solely for the processing of data on the Ilifu SLURM cluster, but
future revisions will allow you to run the software on any HPC platform.

The main features of `processMeerKAT` are as follows:
* Is written in Python 2.7 (for CASA 5.4.X).
* Calibration algorithms only use CASA 5.4.X tasks and helper functions.
* Uses a purpose-built CASA Singularity container for parallel processing at IDIA, i.e, is fully
thread-safe.
* Uses `MPICASA` to run parallel jobs over the cluster.
* Generates `SBATCH` files and ancillary helper scripts for processing.

Please read the documentation to learn how to use the pipeline for your imaging data at IDIA.

{: .fs-6 .fw-300 }

