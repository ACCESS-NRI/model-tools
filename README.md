# Model Tools Deployment

## Overview

This repository is for deploying general purpose software that is used in conjunction with ACCESS models. e.g. pre and post processing tools.

Deployment is on to HPC targets, in this case gadi@NCI, utilising the [build-cd](https://github.com/ACCESS-NRI/build-cd) infrastructure.

## Supported tools

### fre-nctools
[FRE-NCtools](https://github.com/NOAA-GFDL/FRE-NCtools) is a collection of tools for creating grids and mosaics commonly used in climate and weather models, the remapping of data among grids, and the creation and manipulation of netCDF files.

### mppnccombine-fast
[mppnccombine-fast](https://github.com/ACCESS-NRI/mppnccombine-fast) is an accelerated version of the mppnccombine post-processing tool for MOM.

### babeltrace2
[babeltrace2](https://github.com/efficios/babeltrace) is a CTF (Common Trace Format) trace processing library that provides both a CLI and Python bindings (bt2). It is a required dependency for [esmf-trace](https://github.com/ACCESS-NRI/esmf-trace), which is used to extract and visualise runtime profiling data from ESMF/NUOPC-based configurations.

## How to use

**Requirements**: you must be a member of [`vk83`](https://my.nci.org.au/mancini/project/vk83).

The software is accessible through the environment module system on `gadi`, e.g.
```
module use /g/data/vk83/modules
```

To discover what tools and versions are available:
```
module avail model-tools
```
or a specific tool
```
module avail model-tools/fre-nctools
```
It also works without the `model-tools` namespace:
```
module avail fre-nctools
```
To load the most recent version of `fre-nctools`:
```
module load model-tools/fre-nctools
```

## Support

This repository and the software deployed from it are supported by ACCESS-NRI.

If you encounter any issues, or would like other tools added to this repository either [make an issue](https://github.com/ACCESS-NRI/system-tools/issues), or request support through the [ACCESS-Hive Forum](https://forum.access-hive.org.au/t/access-help-and-support/908).
