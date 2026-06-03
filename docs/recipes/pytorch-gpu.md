# Running PyTorch on GPUs

PyTorch is a popular deep learning framework that can take advantage of GPUs to accelerate training and inference. In this recipe, we will show you how to set up and run PyTorch on GPUs in the AICR environment.

We have a few example scripts for this tutorial, which you can download and extract to your home directory:

```bash
wget <link>
unzip aicr_pytorch_gpu_examples.zip
```

<!-- TODO: Add the actual wget command and link -->

## Installing PyTorch with GPU Support

AICR provides a `conda` module, which allows you to create and manage Conda environments. You can use this module to create a Conda environment with PyTorch and GPU support. Alternatively, you can use Python's built-in `venv` module to create a virtual environment and install PyTorch using Pip. Below are instructions for both methods.

It's best to do this on a compute node with a GPU available so that you can ensure that the correct version of PyTorch with GPU support is installed. You can request an interactive job with a GPU using the following command:

```bash
salloc -n 1 --mem=4G -G 1 -p rtx-devel
```

Then, create a Conda environment or Python virtual environment and install PyTorch:

=== "Conda"

    ```bash
    module load conda
    conda create -n my_torch_env pytorch-gpu torchvision matplotlib
    ```

=== "Pip"

    ```bash
    module load conda
    python -m venv my_torch_env
    source my_torch_env/bin/activate
    pip install torch torchvision
    ```

!!! note
    If you don't have a GPU available when you install PyTorch, Conda or Pip may install the CPU-only version of PyTorch. You can check if PyTorch has GPU support by running the following command:

    ```bash
    conda list torch
    ```

    <!-- TODO: Add a Pip version of this instruction -->

    The build will say "cuda" if it has GPU support, and "cpu" if it does not.

## Single-GPU Training

Here we provide an example for training a convolutional neural network on the CIFAR-10 dataset using a single GPU, based on [an example from the PyTorch tutorial](https://docs.pytorch.org/tutorials/beginner/blitz/cifar10_tutorial.html). The script is called `cnn_cifar10_gpu.py` and is included in the example scripts you downloaded.

Next, we'll create a job script to run the training on a GPU. Below is an example of what the job script might look like:

```bash title="submit_single_gpu.sh"
#!/bin/bash

#SBATCH -p rtx-batch
#SBATCH -G 1
#SBATCH -t 30
#SBATCH -N 1
#SBATCH -n 1
#SBATCH -c 2
#SBATCH --mem=10G

module load conda
source activate my_torch_env

python cnn_cifar10_gpu.py
```

<!-- TODO: Move job scripts to zipped repo? -->

You can submit this job by running `sbatch submit_single_gpu.sh`. The `#SBATCH` flags `-N 1 -n 1 -c 2` request 1 node, 1 task, and 2 CPU cores per task. We requested 2 CPU cores because our training script uses 2 worker threads for data loading, and it's generally recommended to have at least as many CPU cores as worker threads to avoid bottlenecks. The `-G 1` flag requests 1 GPU for the job.

While your job is running, you can run `squeue --me` to check its status. You can also log in to the node via SSH by running `ssh <node_name>` and then running `nvtop` to monitor GPU usage and ensure that the GPU is being utilized during training.

## Multi-GPU Training

There are various parallelization strategies for training on multiple GPUs, such as data parallelism and model parallelism. We will cover data parallelism on this page.

Data parallelism allows you to train a model on multiple GPUs by splitting the input data across the GPUs and aggregating the gradients during backpropagation, while the model itself must fit within the memory of each GPU. PyTorch provides a convenient `DistributedDataParallel` module to facilitate this.

### Single-Node Multi-GPU Data Parallelism

On AICR, there are multiple nodes and multiple GPUs per node. Here, we will show how to use multiple GPUs on a single node.

We have prepared an example script called `multigpu.py` that demonstrates how to use `DistributedDataParallel` for training a model on multiple GPUs. You can run this script in a job with multiple GPUs using the following job script (submit by running `sbatch submit_multigpu.sh`):

```bash title="submit_multigpu.sh"
#!/bin/bash

#SBATCH -p rtx-batch
#SBATCH -G 2
#SBATCH -t 30
#SBATCH -N 1
#SBATCH -n 1
#SBATCH -c 2
#SBATCH --mem=20G

module load conda
source activate my_torch_env

# Run for 100 epochs and save checkpoints every 20 epochs
python multigpu.py --batch_size=1024 100 20
```

The above script uses `-N 1` to request a single node and `-G 2` to request 2 GPUs on that node. Note that it also uses `-n 1` and `-c 2` to request 1 task and 2 CPU cores per task. This is because `DistributedDataParallel` runs a single process that uses multiple threads to utilize the GPUs, so we only need 1 task. The number of CPU cores can be adjusted based on the workload and the number of GPUs. In general, it is recommended to have at least as many CPU cores as GPUs to avoid bottlenecks in data loading and processing.

The `multigpu.py` script will automatically detect the number of GPUs available and use them for training. The training happens on two batches of data simultaneously. As before, you can monitor GPU usage with `nvtop` to ensure that both GPUs are being utilized during training.

You can also submit PyTorch programs using the `torchrun` command, which is a PyTorch utility for launching distributed training jobs. You can use `torchrun` by editing the last line of the above job script as follows:

```bash
torchrun --nnodes=1 --nproc_per_node=2 \
         --rdzv_id=$SLURM_JOB_ID \
         --rdzv_endpoint="localhost:1234" \
         multigpu_torchrun.py --batch_size=1024 100 20
```

The flags with `rdzv` (meaning the Rendezvous protocol) are required by `torchrun` to coordinate multiple processes. The flag `--rdzv-id=$SLURM_JOB_ID` sets the rendezvous ID to the Slurm job ID, but it can be any random number. The flag `--rdzv-endpoint="localhost:1234"` is used to specify the host and port. Use `localhost` when there is only one node. The part can be any 4- or 5-digit number larger than 1024.

The `torchrun` command will be useful for running the program across multiple nodes in the next section.

<!-- TODO: Talk about using srun vs torchrun vs mpirun vs python-->

!!! note
    The NVIDIA Collective Communications Library (NCCL) is set as the backend in the PyTorch programs `multigpu.py` and `multigpu_torchrun.py`. As a result, the data communication between GPUs within one node benefits from the high bandwidth of NVLinks.

### Multi-Node Multi-GPU Data Parallelism

Data parallelism can also be used to train a model across multiple nodes, where each node has multiple GPUs. This is similar to single-node multi-GPU training, but requires additional coordination between the nodes.

Our approach here uses Slurm and `torchrun` together, with each tool handling a different layer of orchestration:

1. Use `srun` to launch `torchrun` on each node

2. `torchrun` coordinates worker processes on different nodes

Use the following job script to run the multi-node multi-GPU training:

```bash title="submit_multinode_multigpu.sh"
#!/bin/bash
#SBATCH -p rtx-batch
#SBATCH -N 2
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=2
#SBATCH --gpus-per-node=2 
#SBATCH --mem=20GB

module load conda
source activate my_torch_env

# Get IP address of the master node
nodes=( $( scontrol show hostnames $SLURM_JOB_NODELIST ) )
nodes_array=($nodes)
master_node=${nodes_array[0]}
master_node_ip=$(srun --nodes=1 --ntasks=1 -w "$master_node" hostname --ip-address)

echo "======== Run on multiple GPUs across multiple nodes ======"     
srun torchrun --nnodes=$SLURM_NNODES \
     --nproc-per-node=$SLURM_CPUS_PER_TASK \
     --rdzv-id=$SLURM_JOB_ID   \
     --rdzv-backend=c10d \
     --rdzv-endpoint=$master_node_ip:1234 \
     multinode.py --batch_size=1024 100 20
```

Submit using `sbatch submit_multinode_multigpu.sh`. The above script uses `-N 2` to request 2 nodes, `--ntasks-per-node=1` to request 1 task per node, `--cpus-per-task=2` to request 2 CPU cores per task, and `--gpus-per-node=2` to request 2 GPUs per node.

The `#SBATCH` flags `--cpus-per-task=2` and `--gpus-per-node=2` request 2 GPU cores and 2 GPUs on each node. Accordingly, the `torchrun` flags are set as `--nnodes=$SLURM_NNODES --nproc-per-node=$SLURM_CPUS_PER_TASK`, so that the `torchrun` command runs the program on 2 GPUs on each of the two nodes. Thus, the program runs on 4 GPUs, and the training process happens on 4 batches of data simultaneously.

The flags with `rdzv` are required by `torchrun` to coordinate processes across nodes. The `--rdzv-backend=c10d` is to use a C10d store (by default TCPStore) as the rendezvous backend, the advantage of which is that it requires no 3rd-party dependency. The `--rdzv-endpoint=$master_node_ip:1234` is to set up the IP address and the port of the master node. The IP address is obtained in a previous part of the job script.

<!-- TODO: Add a note about partitions -->

!!! note
    NCCL is set as the backend in the PyTorch program `multinode.py`. As a result, the data communication between GPUs within one node benefits from the high bandwidth of NVLinks, while the communication between GPUs across nodes benefits from the bandwidth of the InfiniBand network.
