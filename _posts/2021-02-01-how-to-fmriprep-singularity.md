---
title: "How to run fmriprep on an HPC cluster (Singularity)"
date: 2021-02-01
permalink: /blog/how-to-fmriprep-singularity/
tags:
  - fMRI
  - fMRIPrep
  - Singularity
  - HPC
  - tutorial
---

This is a very shot tutorial on how to run fMRIPrep on a shared cluster where you do not have access to Docker. This usually happens because docker requieres sudo access to run so in shared machines administrators usually would rather install Singularity which also uses containers but without sudo.

## Why?

If you are here, you have probably tried to run fmriprep through Docker and have realized that you do not have admin rights in the system that you want to use. **If you have not tried that, I suggest you do it** (see [here how to]({{ '/blog/how-to-fmriprep/' | relative_url }})) and only use Singularity as a plan B.

Since I cannot find a reason why you might want to run Singularity locally (other than debugging your scripts), I will describe here how to run it on a **High-Performance Cluster (HPC)**. The first thing to do is to take a deep breath, get a cup of your favorite drink, and equip yourself with a lot of patience: this can get messy.

Every HPC can be different and you will probably have to tweak things here and there to get it running the first time. The examples I give here are for an HPC using SLURM and with a rather closed setup, so you can think about this procedure as a brute-force approach: the bright side is that if you try all this, you will probably succeed on every system.

Also check the [Tips for Singularity]({{ '/blog/tips-for-singularity/' | relative_url }}) post, gathered after too many trial-and-error attempts of my own.

## How?

Make sure Singularity is (or can be) installed on your HPC. This part might require some time and *mano izquierda* to talk your support people into doing it.

Once Singularity is up and running, you need to make sure that you have enough writing space for your user on the HPC. Usually some temporary storage drive is allowed for this — make sure you know where it is and how to access it. NOTE: storage limits can come in size or number of files (inodes), or both; try to discover your limits.

The next step is to download fmriprep's image. While this is a rather trivial step in Docker, in Singularity it can be tricky. First, identify which version of fmriprep you would like to use; I recommend you always start with the latest one. Singularity does not like the "latest" tag when building, so you need to specify the exact version (e.g., `poldracklab/fmriprep:20.1.1`). You might want to build the image locally and transfer it to the HPC:

```bash
singularity build /my_images/fmriprep-<version>.simg docker://poldracklab/fmriprep:<version>

scp /my_images/fmriprep-<version> user@address:/path/to/writable/location/
```

Replacing `<version>` with the exact version name as in the example above, and `user` and `address` with your HPC login details.

1. Some HPC nodes do not have **internet access**, and fmriprep will try to go online to pull brain templates from [*TemplateFlow*](https://www.templateflow.org/). My recommendation is that you download the templates you are going to need (or all of them, if you want to go with the *sledgehammer* approach) and transfer them to the HPC. Templates can be downloaded with the Python API using the syntax below (choose the templates that you need):

   ```python
   from templateflow import api as tflow

   tflow.get('MNI152NLin2009cAsym')
   tflow.get('OASIS30ANTs')
   tflow.get('MNI152NLin6Asym')
   tflow.get('fsaverage')
   tflow.get('NKI')
   tflow.get('MNI152NLin2009cSym')
   tflow.get('WHS')
   tflow.get('fsLR')
   tflow.get('MNIPediatricAsym')
   tflow.get('MNI152Lin')
   tflow.get('MNIInfant')
   tflow.get('MNI152NLin6Sym')
   ```

2. Rather obvious step: you need to put your BIDS dataset on the HPC (duh!).

3. Once you have all the files needed on the HPC, you can start... preparing your call script. Singularity and fmriprep will try to write a bunch of temporary files in your home directory, so if you have limited space in home, you need to redirect them to your larger writable locations. See the lines after *"Re-direct some (...)"* in the script below.

4. You also need to overwrite fmriprep's `TEMPLATEFLOW_HOME` with the path inside the container where you are going to mount your templates. See the lines after *"Pass some variables into the container"*.

5. Now you are ready to call fmriprep through Singularity. After the `run` command, you need to mount the BIDS folder, the output folder, the templates folder, and a writable folder for the home and working directory. Use the `-B` flag for that (see script below). Do not forget to also mount the path to your FreeSurfer license.

6. Remember about the nodes not having internet access? You need to tell fmriprep not to attempt it, since it will fail anyway. That is what the `--notrack` flag does.

7. Finally, it is important to pass the `--home` flag to Singularity so that it overwrites the default `home` directory. This one needs to be followed by `/home/fmriprep`.

8. And you are set!

Just a few more notes on the script below:

- I am passing a variable into the container named `subject` so I can pass it to the `--participant-label` flag for fmriprep. This is pretty useful if you want to create different temporary folders for different subjects, or if you want to use your job ID to index several subjects.
- Sometimes, after crashing, FreeSurfer does not delete all of its intermediate files, and that might cause problems when trying to re-run fmriprep on the same subjects. You might want to include a line at the beginning to search for those corrupted files and delete them if found; something like this will do:

  ```bash
  find ${FREESURFER_HOST_CACHE}/$subject/ -name "*IsRunning*" -type f -delete
  ```

- Everything prior to the line *"Which subject do you want to run?"* is set for our HPC at Goethe University in Frankfurt. Yours might be different in terms of available resources or requirements; adjust it accordingly.

```bash
#!/bin/bash
#SBATCH --array 01
#SBATCH --partition=general1
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=80
#SBATCH --mem-per-cpu=1024
#SBATCH --time=02:30:00
#SBATCH --no-requeue
#SBATCH --mail-type=ALL
# ------------------------------------------

##########################################################################
# fmriprep
##########################################################################
# This script runs a singularity image of fmriPrep on files on BIDS format.
# This script is optimized for Frankfurt's HLR cluster.
#
# Javier Ortiz-Tudela (ortiz-tudela@psych.uni-frankfurt.de)
##########################################################################

echo "Loading singularity..."
spack load singularity@3.5.2

# Which subject do you want to run?
subject=01

# Re-direct some environmental variables to writable locations
export SINGULARITY_TMPDIR=/path/to/writable/location/fmriprep_temp/$subject
export SINGULARITY_CACHEDIR=/path/to/writable/location/.cache
export TEMP_DIR=/path/to/writable/location/fmriprep_temp/$subject
echo "creating temp directory in"
echo $TEMP_DIR
mkdir -p $TEMP_DIR

# Pass some variables into the container
export SINGULARITYENV_subject=$subject
export SINGULARITYENV_TEMPLATEFLOW_HOME=/templateflow

# Setup done, print a message and run the fmriprep call
echo -e "\n"
echo "Starting fmriprep."
echo "subject: $subject"
echo -e "\n"

singularity run \
    -B /path/to/writable/location:/home/fmriprep \
    -B /path/to/data/BIDS:/data:ro \
    -B /path/to/templates/templateflow:/templateflow \
    -B /path/to/output/folder:/output \
    -B /path/to/license:/lic \
    -B $TEMP_DIR:/working_dir \
    --home /home/fmriprep --cleanenv \
    /path/to/image/fmriprep-20.1.0.simg \
    /data /output participant \
    --notrack \
    --fs-license-file /lic/license.txt \
    --participant-label $subject \
    --work-dir /working_dir
```
