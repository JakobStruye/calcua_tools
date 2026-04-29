# Apptainer (==Singularity) container definitions

This directory contains definitions for Apptainer containers for use on the CalcUA cluster GPUs. They are subdivided into containers for NVIDIA (Ampere) and for AMD (Arcturus). Note that multi-node is not taken into account currently (there is only 1 Ampere node and 2 Arcturus).  

## Base images
Containers usually take a base image from Docker that matches our requirements as close as possible and are then extended with any further needs. This includes the following requirements, in no particular order:

- **Operating System**: Ideally RockyLinux 9, but not strictly necessary
- **CUDA version**: At time of writing, Ampere runs CUDA 12.8.0. Ideally, the container would match this version. An **older** version in the container should also work.
- **ROCm version**: At time of writing, Arcturus runs ROCm 7.0.3. Ideally, the container would match this version. Historically, there was a suppoort window of 2 minor versions. As of ROCm 7, driver and userspace versions are decoupled.
- **Installed software/packages**: Ideally, as many things as possible are preinstalled on the image.

## Building
Support for building on the cluster is limited, so images are ideally built locally and then pushed to the cluster. This requires **apptainer** to be installed. The build command is:
`apptainer build --fakeroot imagename.sif definitionname.def` This may take several tens of minutes. 

Ideally, the build is run as non-root (`--fakeroot`). On **Ubuntu**, this requires `sudo apt install uidmap` and `sudo sysctl -w kernel.apparmor_restrict_unprivileged_userns=0` ([source](https://github.com/apptainer/apptainer/issues/2360#issuecomment-2553136933)). 
You can also build as root (`sudo apptainer build` instead of `apptainer build --fakeroot`). If environment variables need to be passed (e.g. `APPTAINER_CACHEDIR`) you will need `sudo -E apptainer build`. This flag currently does NOT work on xUbuntu 25.10/26.04!

By default, apptainer will use a cache and temporary storage, defaulting to the user home folder and `/tmp` respectively. This can be overridden with `APPTAINER_CACHEDIR` and `APPTAINER_TMPDIR`. With big containers, these can run out of storage easily (home folder on shared filesystems, `/tmp` on local machines). It may be necessary to change these.

## Running
Each container can be run with the command `apptainer exec containername.sif command` (for interactive use, instead of `exec` use `shell`, without command). The following flags, to be placed after `exec`, may be of use:
- **`-B /path/on/host:/path/in/container`**: makes a directory from outside the container available inside of it, writable
- **`-B /path/to/file.sqsh:/path/in/container:image-src=/`**: mounts a SquashFS file as directory into the container. Read-only. The `:image-src=/` means that the entire content of the `.sqsh` file is mounted, and should not be modified unless that is not the required behavior.
- **`--nv`**: Ensures that the necessary bindings and environment variables are configured to properly access the host's **NVIDIA** GPUs from within the container
- **`--rocm`**: Ensures that the necessary bindings and environment variables are configured to properly access the host's **AMD** GPUs from within the container

