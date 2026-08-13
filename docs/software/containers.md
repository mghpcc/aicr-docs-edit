# Containers

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
apptainer exec --nv pytorch_26.07-py3.sif python train.py
```

Above, the `--nv` option is used to enable GPU support inside the container. This is necessary if you want to use PyTorch with GPUs.

### Multinode Jobs

Running a container across multiple nodes requires some way for the processes on each node to find each other. How you do this depends on what your application uses for communication: MPI applications use Slurm's PMIx support, while PyTorch distributed applications use `torchrun`'s rendezvous mechanism.

#### MPI Applications

If your application uses MPI, you can use `srun` with the `--mpi=pmix` option and the `apptainer exec` command. For example, to run an MPI program called `mpi_program` using 2 nodes and 8 tasks per node:

```bash
srun -N 2 --ntasks-per-node=8 --mpi=pmix apptainer exec --nv pytorch_26.07-py3.sif ./mpi_program
```

In this approach, `srun` launches a separate `apptainer exec` process for each of the 16 tasks, and the `--mpi=pmix` flag tells Slurm to use the PMIx plugin to set up the wire-up environment and coordinate the parallel launch across the 2 nodes. The MPI library used for communication is the one installed *inside* the container, not the host's.

!!! warning
    Because the container supplies its own MPI, that MPI must be built against a version of PMIx that is compatible with the one on the host. If it only supports the older PMI2 interface, the launch will fail in a confusing way: instead of one 16-rank job, you will get 16 independent single-rank jobs, each reporting itself as rank 0. The container also needs the InfiniBand and UCX libraries in order to use AICR's high-speed [InfiniBand fabric](../system-description.md); without them, MPI will silently fall back to slower TCP communication.

#### PyTorch Distributed Applications

PyTorch distributed applications typically use NCCL rather than MPI, so they do not need `--mpi=pmix`. Instead, `srun` launches one `torchrun` per node, and `torchrun` coordinates the worker processes. This is the same approach described in [Multi-Node Multi-GPU Data Parallelism](../recipes/pytorch-gpu.md#multi-node-multi-gpu-data-parallelism), but with the `torchrun` command wrapped in `apptainer exec` instead of run from a conda environment:

```bash title="submit_multinode_container.sh"
#!/bin/bash
#SBATCH -p rtx-batch
#SBATCH -N 2
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=2
#SBATCH --gpus-per-node=2
#SBATCH --mem=20GB

# Get IP address of the master node
nodes=( $( scontrol show hostnames $SLURM_JOB_NODELIST ) )
nodes_array=($nodes)
master_node=${nodes_array[0]}
master_node_ip=$(srun --nodes=1 --ntasks=1 -w "$master_node" hostname --ip-address)

srun apptainer exec --nv pytorch_26.07-py3.sif \
     torchrun --nnodes=$SLURM_NNODES \
     --nproc-per-node=$SLURM_CPUS_PER_TASK \
     --rdzv-id=$SLURM_JOB_ID \
     --rdzv-backend=c10d \
     --rdzv-endpoint=$master_node_ip:1234 \
     multinode.py --batch_size=1024 100 20
```

Note that there is no `module load` step here: PyTorch, CUDA, and `torchrun` all come from inside the container.

<!-- TODO: Test whether this runs -->

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
    conda install pytorch torchvision torchaudio -c pytorch
```

There are many more options you can include in a definition file, such as environment variables, files to copy into the image, and scripts to run during the build process. You can find more information about writing Apptainer definition files in the [Apptainer documentation](https://apptainer.org/docs/user/main/definition_files.html).

!!! tip
    On AICR, we have `fakeroot` enabled, which allows you to run root-level commands inside a container without needing root privileges on the host. This means you can build an image using commands like `apt-get` or `yum` to install software packages.

### Building an Image in Sandbox Mode

If you want to build an image interactively, you can use the `--sandbox` option to create a writable container image. This allows you to make changes to the container and test them before building a final image. Here's an example of how to build a new image based on a base Ubuntu image in sandbox mode:

```bash
apptainer pull --sandbox docker://ubuntu:22.04
```

This will create a writable container image in a directory called `ubuntu_22.04.sif`. You can then enter the container using the `apptainer shell` command:

```bash
apptainer shell --writable --fakeroot ubuntu_22.04.sif
```

Then, within the container, you can install software packages, copy files, and make any other changes you need, e.g.:

```bash
apt update
apt install -y python3 python3-pip
pip3 install numpy pandas
```

Once you are done making changes, you can exit the container and build a final image using the `apptainer build` command:

```bash
apptainer build python_custom.sif ubuntu_22.04.sif
```
