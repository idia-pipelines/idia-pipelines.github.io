---
layout: default
title: Release Notes
parent: processMeerKAT
nav_order: 1
---

This is the first release of the IDIA calibration pipeline. It currently does only cross-calibration (i.e., *a priori* calibration) and there are plans to include imaging and self-calibration in a future release.

By default, the pipeline executes a full band, full Stokes calibration using a multi-measurement set (MMS). The [[Use Cases|Example-Use-Cases]] web page describes other ways
to use the pipeline.

Known Issues
==

   * *Broadband polarization :* CASA does not natively support solving broad-band polarizations, _i.e.,_ it is not RM aware. The assumption is that the RM within a single spectral window (SPW) is constant, however the MeerKAT has only a single SPW that spans the entire bandwidth. We have identified workarounds (which is to split up the band into several SPWs), however presently the broadband polarization models do contain systematic errors.
