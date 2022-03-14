---
layout: default
title: Access to IDIA Machines
nav_order: 2
---

# Accessing the ilifu cluster

You can request access to the ilifu cluster using [this form](https://docs.ilifu.ac.za/#/getting_started/request_access). You will need to specify which team/project you are a part of while signing up.

## Joining the IDIA Pipelines Team
Access to the IDIA-Pipeline resources can be arranged if you are part of the **Pipelines** Team.
Please contact Bradley Frank ([bradley@idia.ac.za][brad]) if you are interested in joining.

[Documentation][idia-pipelines] and [wiki][idia-pipelines-wiki] access are provided through a GitHub
project, so please sign-up to GitHub (if you haven't already) and join the [IDIA
Pipelines][idia-pipelines] repo as a contributor.

## Access

A summary of how to access the ilifu services are provided [here](https://docs.ilifu.ac.za/#/getting_started/access_ilifu).

## Storage
There are several storage areas available on the ilifu cluster, and access to data is managed via Unix groups.

There are several mounted volumes, intended for different use cases. These are summarized below, please take careful note of the following : 

* `/users/<username>/`, where `/users` is a shared CephFS volume.
* `/idia/users/<username>/`, where `/idia` is a shared CephFS volume. This is the preferred space
  for you to store your medium term data products.
* `/idia/projects/<Project>` is a shared directory on CephFS for a project. You can fetch raw data from here.
  Please steer away from dumping data into this directory.
* `/scratch/users/<username>/` is the shared working directory for <username>, where `/scratch` is a
  CephFS volume). This is the preferred space to use as a working directory, where final data products are moved into one of the more permanent locations listed above, and intermediate data products are purged.

## Setting up your own VM.

## GitHub
There are two important GitHub resources that project members can use:
* [IDIA-Pipelines Blog][idia-pipelines]: This is the repo for this website, and there is an
  associated [wiki][idia-pipelines-wiki] for project members to share information.
* [IDIA Containers][idia-containers]: This is very important repository for the
  [Singularity][singularity] containers used at IDIA.

[idia-pipelines]:  https://github.com/idia-pipelines/idia-pipelines.github.io
[idia-pipelines-wiki]:  https://github.com/idia-pipelines/idia-pipelines.github.io/wiki
[idia-containers]: https://github.com/AfricanResearchCloud/idia-containers
[singularity]: http://singularity.lbl.gov/
[sshkey]: https://confluence.atlassian.com/bitbucketserver/creating-ssh-keys-776639788.html
[helo]: https://helo.idia.ac.za
[request-vm]: https://docs.google.com/forms/d/e/1FAIpQLSc6GwFfqTUcKVmkcmn1VRIBADp-_JOSVt9aW1dfNmc12kxuvg/viewform
[access]: http://docs.ilifu.ac.za/#/getting_started/request_access
[brad]: mailto:bradley@idia.ac.za

