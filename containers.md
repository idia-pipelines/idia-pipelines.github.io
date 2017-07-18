---
layout: page
title: Software Containers 
permalink: /containers/
---
# TL;DR
At IDIA, software is available and managed using [Singularity][singularity] containers. You can find
out more about what's available and how to use them on our dedicated [Github][github-containers]
repository.

# Introduction
The installation of Singularity is easy and simple to install. The important bits to note is that
containers need to be built on a machine which have root access. This includes HPC interconnects,
resource managers, file systems, GPUs and/or accelerators, More information about this can be found
at - [Singularity][singularity]


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

```bash
VERSION=2.3.1
wget https://github.com/singularityware/singularity/releases/download/$VERSION/singularity-$VERSION.tar.gz
tar xvf singularity-$VERSION.tar.gz
cd singularity-$VERSION
./configure --prefix=/usr/local
make
sudo make install
```

[singularity]: http://singularity.lbl.gov/
[github-containers]:https://github.com/AfricanResearchCloud/idia-containers
