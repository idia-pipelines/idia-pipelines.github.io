---
layout: page
title: Access to IDIA 
permalink: /access/
---

# Accounts
Access to the IDIA-Pipeline resources can be arranged if you are part of the **Pipelines** Team.
Please contact Bradley Frank ([bradley@idia.ac.za][brad-email]) if you are interested in joining. 

Members of the **Pipelines** team can request access as follows:
* Send your preferred username, first-name and last-name to admin@arc.ac.za
* You will be mailed with your username and temporary password. You can then login to the cloud dashboard
  ([https://dashboard.arc.ac.za][dashboard]).
* Change your password.
* You can then upload your SSH key, or use the dashboard to generate a new one. You will be able to
  SSH into IDIA machines using this key.

Your username, password and SSH public key will be added to LDAP (default username will be created
from pre-@ part of email)

This will enable you to log into all IDIA machines that connect to LDAP, allowing you to log into
VMs using ssh using their SSH key without needing to use your password.

# Access
You can access IDIA-Pipelines machines via a Jupyter-Hub (preferred) or SSH (traditional). IP
addresses or URLs for IDIA-Pipeline machines will be provided. 

The URL usually corresponds to the name of the VM, and is served on the IDIA domain. For example,
our **racetrack** node can be accessed via https://racetrack.idia.c.za.

### Jupyter-Hub
A Virtual Machine (VM) can be accessed online using your browser (Chrome/Firefox preferred). Simply
type in the URL provided.

You will be presented with a login window for the Jupyter-Hub. Use your previously generated (LDAP)
username and password to access your account.

### SSH
To `SSH` into an IDIA-Pipelines VM, you will need your SSH key. 

This is how you would `SSH` into **racetrack** with X-forwarding:
````bash
$ ssh  -XY -i /path/to/your/key.pem racetrack.idia.ac.za -l <username>
````

# Storage
There are several storage areas available. Most (all) Pipelines machines will have the IDIA storage
attached. This means that you will have access to your work and your data on any system that you're
logged into. 

Access to data is managed with Unix groups. 

Please take careful note of the following.

* `/users/<username>/`, where `/users` is a shared BeeGFS volume.
* `/data/users/<username>/`, where `/data` is a shared BeeGFS volume. This is the preferred space
  for you to store your longish term data products.
* `/data/<Project>` is a shared directory on BeeGFS for a project. You can fetch raw data from here.
  Please steer away from dumping data into this directory.
* `/scratch/users/<username>/` is the shared working directory for <username>, where `/scratch` is a
  BeeGFS volume). This is the preferred space for intermediate data products, e.g., you can use this
  space to do imaging.

# Setting up your own VM. 
Smaller VMs can be spun-up for (sub)-projects. Additionally, users will have to follow the
aforementioned process to setup their accounts.

Please use the Google form to design and [Request a VM][request-vm].

[brad-email]: bradley@idia.ac.za
[dashboard]: https://dashboard.arc.ac.za
[request-vm]: https://goo.gl/forms/qNAhKW0drThtVwAC2
