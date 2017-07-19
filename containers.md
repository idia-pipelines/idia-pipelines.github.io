---
layout: page
title: Software Containers 
permalink: /containers/
---
# Overview
At IDIA, software is available and managed using [Singularity][singularity] containers. You can find
out more about what's available and how to use them on our dedicated [Github][github-containers]
repository.

### Available Containers
We have the following containers available:
* `casa-stable`
** Access: Terminal (SSH).
** The most recent, stable build of **CASA**, and can only be used on
  the terminal (e.g., by SSH'ing into the machine). Includes *MPI* support.
* `idia-jupyter-casa`
** Access: Jupyter (https).
** A working development version of CASA which is controlled through the Jupyter Notebook.
* `idia-python-2.7`
** Access: Terminal (SSH) and Jupyter (https).
** Basic Python 2.7 stack.
* `idia-python-3.6`
** Access: Terminal (SSH) and Jupyter (https).
** Basic Python 3.6 stack.
* `kern2`
** Access: Terminal (SSH) and Jupyter (https).
** A comprehensive radio astronomy software environment that has **all** the packages in the
[KERN2][kern2] repository, which has been painstakingly created by Gijs Molenaar. Please visit
[KERN2][kern2] to find out more about the available software. This includes source finding finding
software (e.g., PYBDSF), CASA and MeqTrees-based tools, for example.

### Reporting a bug / Requesting Software
Visit our [Github][github-containers] repo and open an **Issue**. 
#### Reporting a bug
Please provide a sensible report, which a relevant excerpt of an error message. Please provide
details on the expected behaviour, and possible ideas on how to fix the bug. 
#### Requesting Software
Provide the **name** of the package, the **location** or **link** to the repository or website, and
the name of the *existing* or *new* container in which to install the software package. 

# A Primer of [Singularity][singularity]
### Introduction
The installation of [Singularity][singularity] is easy and simple to install. The important bits to
note is that containers need to be built on a machine which have root access. This includes HPC
interconnects, resource managers, file systems, GPUs and/or accelerators.

The basic model of software deployment and management using [Singularity][singularity] containers
is depicted in the following image.

![singularity-flow]({{ site.url }}/assets/singularity-2.3-flow.png)

### Quick Start

* Create a container: `singularity create -s 10240 casa-stable.img`
* Bootstrap the container: `singularity bootstrap casa-stable.img casa-stable.def`
* Run the container: `singularity run casa-stable.img`
* Spawn a shell to update or troubleshoot the container: `singularity shell -w casa-stable.img`

### Reasons to use [Singularity][singularity]
[Singularity][singularity] is a simple, portable and reproducible container technology to retain a
specific environment across a number of diverse systems. These diverse systems could range from a
HPC center in China to running workloads on Amazon Web Services. The only requirment is that
[Singularity][singularity]  is installed. 

### Installation Procedure 
Linux installation -  There are no dependencies required besides `gcc` and `make`. You need to be
root on the system in order to complete the installation successfully. The `--prefix` could be changed
by using the default to ensure that singularity is installed into the `OS PATH`. The base cloud image
for IDIA will include singularity and also be added to the sudoers environment.

Here's an example of how how to install [Singularity][singularity] on in Linux:

````bash
VERSION=2.3.1
wget https://github.com/singularityware/singularity/releases/download/$VERSION/singularity-$VERSION.tar.gz
tar xvf singularity-$VERSION.tar.gz
cd singularity-$VERSION
./configure --prefix=/usr/local
make
sudo make install
````

OSX installation is slightly more complicated with the use of Vagrant and VirtualBox. The
installation instructions can be found here -- [http://singularity.lbl.gov/install-mac].


# Building our first container: 
Now that we have [Singularity][singularity] installed, we need to create an empty container file or
image file.  Docker, which makes use of the union filesystem, creates the filesystem in a layered
approach which reduces duplication. [Singularity][singularity] however retains everything in a
single container image file which can be easily distributed between environments. 

#### Creating the image file requires the following command: 
````
singularity create -s 10240 casa-stable.img
````
This command will create the image file with a size of 10GB and initialize the filesystem for use.

#### Create a singularity bootstrap file. 
Let us refer to this file as casa-stable.def. This bootstrap file is a plain text file with
instructions on how to build the container and which configuration options to apply. Below is an
excerpt of a bootstrap file we built for an astronomy package called Casa. The following options are
important: 
* **Bootstrap**: Yum as its preferred package management tool. 
* **OSVersion**: Version of the OS.
* **MirrorURL**: Location from where to fetch the OS
* **Include**: Installation of a package on a minimal OS installation

````bash
fontconfig-dev* libSM-dev* libX11-dev* libXext-dev* libXi-dev* libXrender-dev*
mkdir -p /opt/casa 
cd /opt/casa
wget https://casa.nrao.edu/download/distro/linux/release/el7/casa-release-4.7.2-el7.tar.gz
tar xfvz casa-release-4.7.2-el7.tar.gz

%runscript
    exec /opt/casa/casa-release-4.7.2-el7/bin/casa "$@"BootStrap: yum
OSVersion: 7
MirrorURL: http://mirror.centos.org/centos-%{OSVERSION}/%{OSVERSION}/os/$basearch/
Include: yum

# inside the container during the bootstrap instead of the General Availability
# point release (7.x) then uncomment the following line
UpdateURL: http://mirror.centos.org/centos-%{OSVERSION}/%{OSVERSION}/updates/$basearch/

%post
yum -y install centos-release-scl
yum -y install wget vim svn git gcc cmake python libXrandr-dev* libXcursor-dev* libXinerama-dev* libfontconfig-dev* libfontconfig fontconfig-dev libGL* libGL-dev*

# If you want the updates (available at the bootstrap date) to be installed

````
Scriptlets, defined as %post, %setup and %runscript, are used to process various sections of the
bootstrap process. These scriptlets execute and provision the container. 

#### Bootstrap the container by executing the follow command. 
````bash
$ singularity bootstrap casa-stable.img casa-stable.def 
````

The bootstrap process starts by fetching the operating system, installing the dependencies,
downloading/uncompressing the Casa stable package and configuring a few bits and pieces into the
container. It then executes the %runscript scriptlet which places everything listed underneath the
scriptlet into the “/singularity” file in the root filesystem of the container. 

#### Executing the container is as simple as : 
````bash
$ singularity run casa-stable.img
````
or 
````bash
./casa-stable.img
````

#### Interactive use of a container 
The major differences between Docker and Singularity are networking components which are completely
stripped from the environment and makes direct use of the server networking. The user namespace is
well defined and therefore a user on the physical host is the same user in the container. This
improves security several fold because users who need to execute the container no longer need to
have escalated privileges ( ie: root access ) in the container to mount volumes.

Updating a singularity container with additional software or just security updates is as simple as
initiating a shell terminal and executing the commands you would traditionally do in a typical Linux
environment. The container image is updated immediately upon exit of the shell and is ready for
distribution. The bootstrap definition file can be updated as well for future deployments if needed
but is not necessary to rebuild the container.

As root, execute the command which gives the user write access to the container. By default, only
read-only access is granted to the environment. 

````bash
singularity shell -w casa-stable.img 
````


[singularity]: http://singularity.lbl.gov/
[github-containers]:https://github.com/AfricanResearchCloud/idia-containers
[kern2]: http://kernsuite.info/
