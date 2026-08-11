---
title: "Tips for Singularity"
excerpt: "Tips gathered after too-many trial-and-error attempts."
date: 2021-02-01
permalink: /blog/tips-for-singularity/
tags:
  - Singularity
  - HPC
  - tutorial
---

Tips gathered after too-many trial-and-error attempts.

## Failed to set effective UID to 0

From [this GitHub issue](https://github.com/hpcng/singularity/issues/1258) I got that the easiest way is to look for a `singularity.conf` file and change `allow setuid=yes` to `allow setuid=no`. That file should be located in your Singularity installation directory. If you used spack for the installation, the path would look something like this:

```
/path/to/spack/opt/spack/linux-scientific7-haswell/gcc-4.8.5/singularity-3.5.2-k2cgya4v4mjzvfay2n4jfmy7wbda5ucq/etc/singularity/singularity.conf
```

See also: [how to run fmriprep on an HPC cluster with Singularity]({{ '/blog/how-to-fmriprep-singularity/' | relative_url }}).
