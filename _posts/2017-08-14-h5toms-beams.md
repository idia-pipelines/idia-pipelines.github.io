---
layout: post
title: Updated h5toms.py, incoming MeerKAT beams, Tsys
data: 2017-08-14 11:19:00 +02000
---

Hi Folks! 

Here's an update of a few important issues relating to the processing of MeerKAT data.

### KATDAL and H5-to-MS Conversions.

**Summary:** A new `h5toms.py` script is being used to remedy the _upside-down_ UVW coordinate issue
in MeerKAT AR1.5 measurement sets.

We currently have all the available MeerKAT AR1.5 data stored and accessible at IDIA. The MSs were
converted from the original H5 data using the `h5toms.py` script from SKA-SA's KAT Data Access Layer
Software, [`KATDAL`][katdal]. We had initially built [`KATDAL`][katdal] using the [KERN-2][kern2]
repository, but this provided an _old_ version of [`KATDAL`][katdal] (0.7) which produced MSs with a
reversal in the UVW coordinates that CASAPY expected, which lead to _upside-down_ images being
produced.

 [`KATDAL`][katdal] version 0.8 is meant to fix this -- and the details can be found in Bruce
 Merry's [commit][bmerry] aiming to fix this issue. However, this version is not available in repo,
 and we've thus had to build it separately.

 Incidentally, **this issue can be fixed** after the fact, in CASAPY using the task `fixvis`.

 So -- I've started converting the DEEP-2 data into MS again using the most recent version of
 `h5toms.py`, and we will test the MSs to make sure that the coordinates have been fixed before I
 convert the rest of the data. The temporary location for the newly created MSs are in the following
 location on the IDIA systems: `/data/share/AR1.5_June_MKAT_Files/UVFIX/`.

### MeerKAT and Tsys

I'd also like to note this important point, that MeerKAT visibility data is "as raw as possible",
and has **not** been scaled by Tsys (sytem temperature) of the antenna pairs.  

### KATDAL and H5-to-MS Conversions.

I've contacted the MeerKAT office and have requested full-polarization, full-spectral resolution
wide-band beam models. If I understand correctly, the beams were measured by Mattieu de Villiers
using holography, and are being modelled using Zernicke polynomials by Khan bin Assad. I am hoping
that these beams will be transferred and available on our system this week. 

Please contact me if you have any questions regarding this information.

-brad

[katdal]: https://github.com/ska-sa/katdal
[kern2]: https://launchpad.net/~kernsuite/+archive/ubuntu/kern-2
[bmerry]: https://github.com/ska-sa/katdal/commit/de7706c3b884b09c3e6c276b089a54e7fee1eadd
