# Containers

<!-- TODO: 
- Include a note about running multinode containerized jobs -->

Containers provide an isolated environment to run applications. You may want to use a container if you need to run an application that requires a specific version of a library or software package that is not available or difficult to install on AICR. Containers can also be used to package and distribute applications, making it easier to share your work with others.

The most well-known container software is Docker, which is designed for laptops/desktops and cloud platforms. On AICR, however, we use Apptainer instead, which is particularly designed for HPC.

!!! note
    Apptainer was formerly known as Singularity. You may still see references to Singularity in some documentation or online resources. Most of the time, the `apptainer` command is a drop-in replacement for `singularity`.

## Downloading an Image

A container "image" is a file that contains an application and all of its dependencies. Apptainer lets you download a container image from a public registry, such as [Docker Hub](https://hub.docker.com/), and convert it into an Apptainer image.

Let's walk through an example of downloading a PyTorch Docker image and converting it into an Apptainer image. It would be best to do this in a [job on a compute node](../running-jobs/slurm-basics.md#interactive-jobs), as downloading and converting an image can take up lots of time and resources. First, let's start an interactive job using the `cpu` partition:

```bash
salloc -N 1 -n 8 --mem=32G -p cpu -t 120
```

[NVIDIA's NGC catalog](https://catalog.ngc.nvidia.com/) is a good source for container images that are optimized for GPU workloads. Once you find an image you want to use, you can download it using the image's "tag" (e.g., `nvcr.io/nvidia/pytorch:26.07-py3`). You can use the `apptainer pull` command to download and convert the image into an Apptainer image:

```bash
apptainer pull docker://nvcr.io/nvidia/pytorch:26.07-py3
```

!!! tip
    Set the `APPTAINER_CACHEDIR` environment variable to `/scratch/$USER/.apptainer` to avoid filling up your home directory.

Once this command completes, you will have a file called `pytorch_26.07-py3.sif` in your current working directory. This is the Apptainer image that you can use to run PyTorch.

## Using an Image

There are a few ways to use an Apptainer image. The most common way is to run a command inside the container using the `apptainer exec` command. For example, to run a Python script called `train.py` inside the PyTorch container, you can use the following command:

```bash
apptainer exec pytorch_26.07-py3.sif python train.py
```

### Bind Mounting Directories

By default, Apptainer only gives the container access to your home directory. If you need to access files outside of your home directory (e.g. in `/scratch` or `/work`), you can use the `--bind` option to bind mount a directory from the host into the container. For example, to bind mount the `/scratch` directory into the container, you can use the following command:

```bash
apptainer exec --bind /scratch pytorch_26.07-py3.sif python train.py
```

You can also set the `APPTAINER_BIND` environment variable to bind mount directories by default. For example, to bind mount both `/scratch` and `/work`, you can use the following command:

```bash
export APPTAINER_BIND="/scratch,/work"
```

## Building an Image

If a public container registry does not have the image you need, you can build your own. Most of the time, you will build an image based on an existing image. You can do this either using an Apptainer definition file or by building the image in sandbox mode.

### Building an Image from a Definition File

An Apptainer definition file is a text file that contains instructions for building a container image. In the file, you can specify the base image to use, the software packages to install, and any other configuration options. You can then use the `apptainer build` command to build the image from the definition file. Here's an example of a simple definition file that installs PyTorch and its dependencies using a base Miniconda image:

```bash
Bootstrap: docker
From: anaconda/miniconda

%post
    conda install pytorch torchvision torchaudio cpuonly -c pytorch
```

!!! tip
    On AICR, we have `fakeroot` enabled, which allows you to run root-level commands inside a container without needing root privileges on the host. This means you can build an image using commands like `apt-get` or `yum` to install software packages.

### Building an Image in Sandbox Mode


