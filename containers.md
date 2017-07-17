---
layout: page
title: Software Containers 
permalink: /containers/
---

# Introduction
The installation of Singularity is easy and simple to install. The important bits to note is that
containers need to be built on a machine which have root access. This includes HPC interconnects,
resource managers, file systems, GPUs and/or accelerators, More information about this can be found
at - [Singularity][singularity]

[singularity]: http://singularity.lbl.gov/

# Quick Start

* Create a container: `singularity create -s 10240 casa-stable.img`
* Bootstrap the container: `singularity bootstrap casa-stable.img casa-stable.def`
* Run the container: `singularity run casa-stable.img`
* Spawn a shell to update or troubleshoot the container: `singularity shell -w casa-stable.img`

# Reasons to use Singularity
Singularity is a simple, portable and reproducible container technology to retain a specific
environment across a number of diverse systems. These diverse systems could range from a HPC center
in China to running workloads on Amazon Web Services. The only requirment is that Singularity is
installed. 

# Installation Procedure 
Linux installation -  There are no dependencies required besides `gcc` and `make`. You need to be
root on the system in order to complete the installation successfully. The `--prefix` could be changed
by using the default to ensure that singularity is installed into the `OS PATH`. The base cloud image
for IDIA will include singularity and also be added to the sudoers environment.

