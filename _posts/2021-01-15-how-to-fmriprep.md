---
title: "How to run fmriprep (and not die trying)"
date: 2021-01-15
permalink: /blog/how-to-fmriprep/
tags:
  - fMRI
  - fMRIPrep
  - Docker
  - tutorial
---

If you are here you probably already know about [fmriprep](https://fmriprep.org/en/stable/usage.html) and [BIDS](https://bids.neuroimaging.io/), so no need to get into that. Let's get started.

The easiest way to use fmriprep is through a Docker container (learn more about them [here](https://opensource.com/resources/what-docker)). You need to install it in your system in case it is not there yet (click [here](https://docs.docker.com/get-docker/) to learn how to). Bear in mind that, in order to use Docker, you need admin rights in your system. In case you cannot get those, [you can use Singularity instead]({{ '/blog/how-to-fmriprep-singularity/' | relative_url }}).

Below you will find an example shell script that I use to make the fmriprep call using Docker (original source [here](https://gitlab.com/ortizTud/neuroim-methods/-/blob/master/resources/example-fmriprep_local.sh)).

```bash
##########################################################################
# fmriprep
##########################################################################
# This script runs a docker image of fmriPrep on files on BIDS format.
#
# Javier Ortiz-Tudela (ortiz-tudela@psych.uni-frankfurt.de)
##########################################################################

### Input in the study info here #####

# Which subject are you preprocessing?
which_sub=999

# Where is your project located?
project_path=~/path/to/project/

### Now I will build some names for you ####

# Build BIDS and output folders paths
bids_path=$project_path/BIDS
out_path=$project_path/BIDS/derivatives

# Build license folder path
lic_path=$project_path/

#----START PREPROCESSING ------#
sudo docker run -ti --rm \
  -e $which_sub \
  -v $bids_path/:/data:ro \
  -v $out_path/:/output \
  -v $lic_path/:/lic \
  nipreps/fmriprep:latest \
  /data /output \
  participant \
  --participant_label $which_sub \
  --fs-license-file /lic/license.txt
```

Just place this snippet of code on your machine, adapt the paths to match yours, and select which options you want to turn on or off for fmriprep (check the full list [here](https://fmriprep.org/en/stable/usage.html)). Most of the things can be set to default, but there are a few options that you will probably want to fiddle with:

- **Options for handling performance**: fmriprep runs some heavy stuff on your images, so you might want to restrict how much of your machine's power it can harvest. Find [here the recommendations from their site](https://fmriprep.org/en/stable/faq.html#how-much-cpu-time-and-ram-should-i-allocate-for-a-typical-fmriprep-run). I would not recommend less than 10-12 GB of RAM for one subject. Use the number-of-threads argument to restrict the number of subprocesses run in parallel (too many things in parallel will max out your RAM and crash the entire thing!).
- **`--output-spaces`**: your preprocessed data will be resampled into whichever space you put here. You can specify the space and the resolution (this last option is only available for standard spaces). See [here](https://fmriprep.org/en/stable/spaces.html) a more detailed guide on spaces. NOTE: If you want to use AROMA in fmriprep to clean your data, you can only use `MNI152NLin6Asym`; if you want to preprocess your data with fmriprep and then run AROMA manually, that is also possible.

Getting the whole thing running for the first time might sound daunting, but after you have done it once for one dataset (and computer), it will be super easy to run more people and to transfer that knowledge to new projects.
