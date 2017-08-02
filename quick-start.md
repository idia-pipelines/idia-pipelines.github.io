---
layout: page
title: Quick Start 
permalink: /quick-start/
---

This usage manual has been developed by IDIA and is currently implemented on the **helo**
system. **helo** can be accessed at `helo.idia.ac.za`. The process will be the same for
any other Pipelines' Virtual Machine on the IDIA system.

## Jupyter-Hub
* Go to [https://helo.idia.ac.za][helo] in your preferred browser. 
* Login with your LDAP username/password.
* Simply choose the container that you want to use from your Jupyter Hub dashboard. A Jupyter
  Notebook corresponding to the container will open up.

## Terminal

* SSH as follows: `ssh -Y -i /path/to/your/key helo.idia.ac.za -l <your-username>`.
* The containers can be found in the following path: `/data/exp_soft/containers/`
* To jump into a container shell, you can use the command: `singularity shell
  /path/to/container.img`.

[helo]: https://helo.idia.ac.za
