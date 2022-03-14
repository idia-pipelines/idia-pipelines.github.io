---
layout: default
title: Singularity Containers
nav_order: 4
---
# Singularity Containers
At IDIA, astronomical software packages are provided and managed using [Singularity][singularity] containers. Containers
are managed and built by our developers. If you have any questions/issues relating to containers,
please send an email to [support@ilifu.ac.za][support]. You can find more documentation about containers on Ilifu [here](http://docs.ilifu.ac.za/#/getting_started/container_environments).


# Available Containers
The most recent, stable versions of containers are available in `/idia/software/containers/`. Some
of these containers can be accessed via JupyterHub, but all of them can be accessed via the
terminal (i.e., once you've ssh'd into an IDIA machine).

Here are a few examples of the latest stable Singularity container builds that are available in
`/idia/software/containers/`:
* `jupyter-casa-latest.simg`
    * Allows you to use CASA tasks via the Jupyter notebook. Note that visualisation tools like
      `plotms` or `viewer` will not work on the notebook. This currently runs CASA v5.5.
* `casa-6.4.simg`
    * Contains the modular CASA 6.4 install, which can be accessed via Python using `import casatools` etc.
* `kern7.simg`
    * Contains _all_ the software packages provided by the [Kern][kern] repository.
* `sourcefinding_py3.simg`
    * Builds of commonly used source finding packages, e.g., PyBDSF.

# Reporting a bug / Requesting Software
Please send an email to [support@ilifu.ac.za][support] to report issues, or to request new
containers, or for existing containers to be updated.

# Using Containers
#### Shell
You can access software in Singularity containers using two methods.

Firstly, you can _shell_ into the container. You will enter a _shell_ which provides access to the
software provided in that container:

```bash
$ singularity shell /idia/software/containers/casa-stable-5.3.0.simg
Singularity: Invoking an interactive shell within container...

Singularity casa-stable-5.3.0.simg:~> casa --nologger --log2term --nogui

=========================================
The start-up time of CASA may vary
depending on whether the shared libraries
are cached or not.
=========================================

IPython 5.1.0 -- An enhanced Interactive Python.

CASA 5.3.0-143   -- Common Astronomy Software Applications

2018-08-07 12:37:28	INFO	::casa	CASA Version 5.3.0-143
--> CrashReporter initialized.
Enter doc('start') for help getting started with CASA...
Using matplotlib backend: TkAgg

CASA <1>: print "Hello World"
Hello World
```

#### Exec
You can also pass a command to a container (using the `exec` argument), which then gets executed via
the shell in that container.

For example, here's an illustration of how to use the `exec` argument to jump into an interactive
CASA session:

```bash
$ singularity exec /idia/software/containers/casa-stable-5.3.0.simg casa --nologger --log2term --nogui

=========================================
The start-up time of CASA may vary
depending on whether the shared libraries
are cached or not.
=========================================

IPython 5.1.0 -- An enhanced Interactive Python.

CASA 5.3.0-143   -- Common Astronomy Software Applications

2018-08-07 12:41:19	INFO	::casa	CASA Version 5.3.0-143
--> CrashReporter initialized.
Enter doc('start') for help getting started with CASA...
Using matplotlib backend: TkAgg
CASA <1>: inp listobs
--------> inp(listobs)
#  listobs :: List the summary of a data set in the logger or in a file
vis                 =         ''        #  Name of input visibility file (MS)
selectdata          =       True        #  Data selection parameters
     field          =         ''        #  Selection based on field names or field index numbers. Default is all.
     spw            =         ''        #  Selection based on spectral-window/frequency/channel.
     antenna        =         ''        #  Selection based on antenna/baselines. Default is all.
     timerange      =         ''        #  Selection based on time range. Default is entire range.
     correlation    =         ''        #  Selection based on correlation. Default is all.
     scan           =         ''        #  Selection based on scan numbers. Default is all.
     intent         =         ''        #  Selection based on observation intent. Default is all.
     feed           =         ''        #  Selection based on multi-feed numbers: Not yet implemented
     array          =         ''        #  Selection based on (sub)array numbers. Default is all.
     uvrange        =         ''        #  Selection based on uv range. Default: entire range. Default units: meters.
     observation    =         ''        #  Selection based on observation ID. Default is all.

verbose             =       True        #  Controls level of information detail reported. True reports more than False.
listfile            =         ''        #  Name of disk file to write output. Default is none (output is written to logger only).
listunfl            =      False        #  List unflagged row counts? If true, it can have significant negative performance impact.
cachesize           =         50        #  EXPERIMENTAL. Maximum size in megabytes of cache in which data structures can be held.
CASA <2>:
```

The true utility of the `exec` argument is to execute commands non-interactively:
```bash
$ singularity exec /idia/software/containers/casa-stable-5.3.0.simg casa --nologger --log2term --nogui -c "print 'Hello World'"

=========================================
The start-up time of CASA may vary
depending on whether the shared libraries
are cached or not.
=========================================

IPython 5.1.0 -- An enhanced Interactive Python.

CASA 5.3.0-143   -- Common Astronomy Software Applications

2018-08-07 12:45:43	INFO	::casa	CASA Version 5.3.0-143
--> CrashReporter initialized.
Hello World
$
```

While the command may seem cumbersome, it is very useful when trying to build scripts that utilise
several containers.

[singularity]: http://singularity.lbl.gov/
[github-containers]:https://github.com/AfricanResearchCloud/idia-containers
[kern]: http://kernsuite.info/
[sfissues]: https://github.com/AfricanResearchCloud/idia-containers/issues/4
[support]: mailto:support@ilifu.ac.za
