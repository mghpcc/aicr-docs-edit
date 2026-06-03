# Running PyTorch on GPUs - Advanced

This recipe covers advanced topics in using PyTorch on GPUs in the AICR environment. If you haven't already, we recommend reading the [previous recipe on running PyTorch](pytorch-gpu.md) before proceeding with this one. In this recipe, we will cover some more advanced topics related to using PyTorch on GPUs, including [Fully Sharded Data Parallelism (FSDP)](https://pytorch.org/blog/introducing-pytorch-fully-sharded-data-parallel-api/) and model parallelism.

## Getting Started

We have a few example scripts for this tutorial, which you can download and extract to your home directory:

```bash
wget <link>
unzip aicr_pytorch_gpu_examples.zip
```
<!-- TODO: Add the actual wget command and link -->

Ensure you have a Python enviornment with PyTorch installed with GPU support. You can follow [these instructions](pytorch-gpu.md#installing-pytorch-with-gpu-support) to set up your environment.

## Fully Sharded Data Parallelism (FSDP)

FSDP is a method for training large models that do not fit into the memory of a single GPU. It works by sharding the model across multiple GPUs and synchronizing the gradients during training. This allows you to train larger models than would be possible with data parallelism alone.

We will use an [example from the PyTorch examples repository](https://github.com/pytorch/examples/tree/main/mnist) to train a convolutional neural network (CNN) with the MNIST dataset. First, we will train using a single GPU, and then we will show how to use FSDP to train the same model on multiple GPUs.

### Single GPU

We will be using the `mnist_gpu.py` script included in the examples. To train the model on a single GPU, create the following job script:

```bash title="submit_single_gpu_mnist.sh"
#!/bin/bash

#SBATCH -p rtx-batch
#SBATCH --job-name=mnist-gpu
#SBATCH -N 1
#SBATCH -n 1
#SBATCH --mem=20GB
#SBATCH -G 1  

module load conda
source activate my_torch_env

python mnist_gpu.py
```

You can submit this job by running `sbatch submit_single_gpu_mnist.sh` and monitor progress using `squeue --me`.

### Single-Node Multi-GPU FSDP

Now let's modify the job script to use FSDP for training on multiple GPUs, using the `FSDP_mnist.py` script. We will use 2 GPUs in this example:

```bash title="submit_fsdp_mnist.sh"
#!/bin/bash

#SBATCH -p rtx-batch
#SBATCH --job-name=fsdp
#SBATCH -N 1
#SBATCH -n 1
#SBATCH -c 2
#SBATCH --mem=20GB
#SBATCH -G 2

module load conda
source activate my_torch_env

python FSDP_mnist.py
```

You can submit this job with `sbatch submit_fsdp_mnist.sh` and monitor it with `squeue --me`.

In this case, we are requesting 2 GPUs and 2 tasks, which will allow us to use FSDP to shard the model across the two GPUs. The model is split into 2 shards, each stored on a different GPU, and the training happens on 2 batches of data simultaneously.

## Hybrid Fully Sharded Data Parallel and Tensor Parallel

FSDP does not gain additional speedup beyond the data parallel framework. To further speed up training, some approaches are based on model parallelism, which not only splits the model onto multiple GPUs but also accelerates the training process with parallel sharded computations.

There are various schemes of model parallelism, such as pipeline parallel (PP) and tensor parallel (TP). Often, model parallelism can be combined with data parallelism to further speed up training. In this section, we will cover such hybrid approaches.

We will use an example that implements FSDP + TP on LLAMA2 (Large Language Model Meta AI 2). Refer to the description of this example [here](https://pytorch.org/tutorials/intermediate/TP_tutorial.html). We will be using the following scripts for this section: `fsdp_tp_example.py`, `llama2_model.py`, and `log_utils.py`.

### Single-node multi-GPU FSDP + TP

First, we will run the example on multiple GPUs on a single node.

In the script `fsdp_tp_example.py`, the TP size is set to 2. The total number of GPUs should be equal to a multiple of the TP size, and then the FSDP size is equal to the number of GPUs divided by the TP size.

Prepare the following job script to run the example on 4 GPUs:

```bash title="submit_fsdp_tp.sh"
#!/bin/bash

#SBATCH -p rtx-batch
#SBATCH --job-name=fsdp-tp
#SBATCH -t 60
#SBATCH -N 1
#SBATCH -n 1
#SBATCH -c 4
#SBATCH --mem=30GB
#SBATCH -G 4

module load conda
source activate my_torch_env

torchrun --nnodes=1 --nproc_per_node=4 \
     --rdzv_id=$SLURM_JOB_ID \
     --rdzv_endpoint="localhost:1234" \
     fsdp_tp_example.py
```

Submit this job with `sbatch submit_fsdp_tp.sh` and monitor it with `squeue --me`.

With the flags `-N 1` and `-n 4`, the `torchrun` command will run the program on 4 GPUs within one node. The training process happens on 2 batches of data with FSDP, and the model is trained with TP sharded computation on 2 GPUs for each batch of data.

The flags with `rdzv` (rendezvous protocol) are required by `torchrun` to coordinate multiple processes. The flag `--rdzv-id=$SLURM_JOB_ID` sets the rendezvous ID to the Slurm job ID, but it can be any random number. The flag `--rdzv-endpoint="localhost:1234"` is used to specify the host and port. Use `localhost` when there is only one node. The part can be any 4- or 5-digit number larger than 1024.

### Multi-node multi-GPU FSDP + TP

Now let's run the same example across multiple nodes. We will use 2 nodes with 4 GPUs each, for a total of 8 GPUs.

Prepare the following job script to run the example on 2 nodes with 4 GPUs each:

```bash title="submit_fsdp_tp_multinode.sh"
#!/bin/bash

#SBATCH -p rtx-batch
#SBATCH --job-name=fsdp-tp-multinode
#SBATCH -N 2
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=4
#SBATCH --gpus-per-node=4 
#SBATCH --mem=30GB

module load conda
source activate my_torch_env

# Get IP address of the master node
nodes=( $( scontrol show hostnames $SLURM_JOB_NODELIST ) )
nodes_array=($nodes)
master_node=${nodes_array[0]}
master_node_ip=$(srun --nodes=1 --ntasks=1 -w "$master_node" hostname --ip-address)

srun torchrun --nnodes=$SLURM_NNODES \
          --nproc-per-node=$SLURM_CPUS_PER_TASK \
          --rdzv-id=$SLURM_JOB_ID   \
          --rdzv-backend=c10d \
          --rdzv-endpoint=$master_node_ip:1234 \
          fsdp_tp_example.py
```

You can submit this job with `sbatch submit_fsdp_tp_multinode.sh` and monitor it with `squeue --me`.

The training process happens on 4 batches of data with FSDP, and the model is trained with TP sharded computation on 2 GPUs for each batch of data.

!!! note
    The NVIDIA Collective Communications Library (NCCL) is set as the backend in all of the PyTorch programs in this tutorial so that the communication between GPUs within one node benefits from the high bandwidth of NVLinks, and the communication between GPUs across nodes benefits from the bandwidth of the Infiniband network.

    The intra-node GPU-GPU communication speed is much faster than inter-node. The communication overhead of TP is much larger than that of FSDP. The topology of GPU communication is set up (in the code `fsdp_tp_example.py`) in a way that TP communication is intra-node and FSDP communication is inter-node, so that the usage of the network bandwidth is optimized.
