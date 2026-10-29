# CUDA-Aware MPI

CUDA-aware MPI is an MPI implementation that accepts GPU memory pointers directly in MPI calls such as `MPI_Send` and `MPI_Recv`. Data can then move between GPUs on different MPI tasks without first being copied to host (CPU) memory, which simplifies code and can improve performance.

This page shows how to build and run a CUDA-aware MPI program using the MPI that ships with the [NVHPC](https://developer.nvidia.com/hpc-sdk) software stack, NVIDIA's suite of compilers, libraries, and tools for NVIDIA GPUs.

## Basic Usage Example

This example compiles and runs a small C program in which two MPI tasks, each using its own GPU, exchange a buffer held in GPU memory.

The steps below can be run in an interactive Slurm session. A complete Slurm batch script that performs all of the steps is given [at the end](#example-slurm-job-script).

!!! note
    MPI transfers between GPUs may be staged through the CPU. For faster direct GPU-to-GPU communication, consider using [NCCL](https://developer.nvidia.com/nccl).

### 1. Load the NVHPC module

The CUDA-aware MPI used here is part of the NVHPC environment module. Load it with:

```bash
module load nvhpc
```

This adds the latest NVHPC version to your shell or batch script. For this example, the relevant parts of the module are:

- the `mpicc` and `mpiexec` MPI compiler wrapper and launcher, built on OpenMPI with CUDA support
- the CUDA runtime library and headers

### 2. Set the paths needed for compilation

The compiler needs to know where the CUDA header files and libraries are in order to build and link the program. The `nvhpc` module does not add these to your environment, but they are inside the NVHPC installation directory, which you can find from the location of the NVHPC C compiler, `nvc`:

```bash
nvhpc_root=$(dirname $(dirname $(dirname $(which nvc))))
culibdir=$nvhpc_root/cuda/lib64
cuincdir=$nvhpc_root/cuda/include
```

The first line strips `/compilers/bin/nvc` from the path to `nvc`, which leaves the installation directory.

### 3. Create the test program

Next, create a C program in which two MPI tasks, each using its own GPU, send data from one GPU to the other. Paste the code below into a file named `test.c`.

```c title="test.c"
#include <stdio.h>
#include <stdlib.h>
#include <mpi.h>
#include <cuda_runtime.h>
int main(int argc, char *argv[])
{
  int myrank, mpi_nranks;
  int LBUF=1000000;
  float *sBuf_h, *rBuf_h;
  float *sBuf_d, *rBuf_d;
  int bSize;
  int i;

  MPI_Init(&argc, &argv);
  MPI_Comm_rank(MPI_COMM_WORLD, &myrank);                  // my MPI rank
  MPI_Comm_size(MPI_COMM_WORLD, &mpi_nranks);

  if ( mpi_nranks != 2 ) { printf("Program requires exactly 2 ranks\n");exit(-1); }

  int deviceCount;
  cudaGetDeviceCount(&deviceCount);               // How many GPUs?
  printf("Number of GPUs found = %d\n",deviceCount);
  int device_id = myrank % deviceCount;
  cudaSetDevice(device_id);                       // Map MPI-process to a GPU
  printf("Assigned GPU %d to MPI rank %d of %d.\n",device_id, myrank, mpi_nranks);

  // Allocate buffers on each host and device
  bSize = sizeof(float)*LBUF;
  sBuf_h = malloc(bSize);
  rBuf_h = malloc(bSize);
  for (i=0;i<LBUF;++i){
    sBuf_h[i]=(float)myrank;
    rBuf_h[i]=-1.;
  }
  if ( myrank == 0 ) {
   printf("rBuf_h[0] = %f\n",rBuf_h[0]);
  }

  cudaMalloc((void **)&sBuf_d,bSize);
  cudaMalloc((void **)&rBuf_d,bSize);

  cudaMemcpy(sBuf_d,sBuf_h,bSize,cudaMemcpyHostToDevice);

  if ( myrank == 0 ) {
   MPI_Recv(rBuf_d,LBUF,MPI_FLOAT,1,0,MPI_COMM_WORLD,MPI_STATUS_IGNORE);
  }
  else if ( myrank == 1 ) {
   MPI_Send(sBuf_d,LBUF,MPI_FLOAT,0,0,MPI_COMM_WORLD);
  }
  else
  {
   printf("Unexpected myrank value %d\n",myrank);
   exit(-1);
  }

  cudaMemcpy(rBuf_h,rBuf_d,bSize,cudaMemcpyDeviceToHost);
  if ( myrank == 0 ) {
   printf("rBuf_h[0] = %f\n",rBuf_h[0]);
  }

  MPI_Barrier(MPI_COMM_WORLD);
  MPI_Finalize();
}
```

In this program, rank 1 sends a buffer of ones from its GPU to rank 0, which initially holds `-1`. Rank 0 prints the first element of its receive buffer before and after the transfer, so a successful run shows it change from `-1` to `1`.

### 4. Compile the program

Compile with the NVHPC MPI compiler wrapper, `mpicc`:

```bash
mpicc test.c -I${cuincdir} -L${culibdir} -Wl,-rpath,${culibdir} -lcudart
```

The options do the following:

- `-I` and `-L` point the compiler to the CUDA header and library directories set in step 2.
- `-lcudart` links the CUDA runtime library, which provides the CUDA functions used by the program.
- `-Wl,-rpath` records the CUDA library directory in the executable so the CUDA runtime can be found when the program runs.

This produces an executable named `a.out`.

### 5. Run the program

Use the `mpiexec` command provided by the `nvhpc` module to run the program. By default, the module uses its built-in version of OpenMPI. The `--bind-to none` option stops OpenMPI from pinning each task to specific CPU cores. Without it, `mpiexec` fails with a binding error.

```bash
salloc -n 2 --gpus-per-task=1 -p rtx-devel
mpiexec --bind-to none -n 2 ./a.out
```

The `salloc` command requests an interactive allocation and is only needed when working from a login node. If you are already in an interactive job on a compute node, or are running in a batch job, skip it.

The program should produce output like the following. The lines may appear in a different order, because the two tasks run asynchronously.

```
Number of GPUs found = 2
Number of GPUs found = 2
Assigned GPU 1 to MPI rank 1 of 2.
Assigned GPU 0 to MPI rank 0 of 2.
rBuf_h[0] = -1.000000
rBuf_h[0] = 1.000000
```

## Example Slurm job script

The script below performs all of the steps above in a single batch job. First, create `test.c` as described in [step 3](#3-create-the-test-program). Then save the script as `test_cuda_and_mpi.sbatch` in the same directory and submit it with:

```bash
sbatch test_cuda_and_mpi.sbatch
```

Output is written to `myjob.log-<jobID>`, where `<jobID>` is the ID Slurm assigns to the job.

```bash title="test_cuda_and_mpi.sbatch"
#!/bin/bash
#SBATCH -n 2
#SBATCH -o myjob.log-%A
#SBATCH --gpus-per-task=1
#SBATCH -p rtx-devel
#
# Basic Slurm job that tests CUDA-aware MPI in the NVHPC tool stack.
#
# Submit with:
#
#   $ sbatch test_cuda_and_mpi.sbatch

# Write some log information
echo "## Start time \""`date`"\""
echo "## Slurm job running on nodes \"${SLURM_JOB_NODELIST}\""
echo "## Slurm submit directory \"${SLURM_SUBMIT_DIR}\""
echo "## Slurm submit host \"${SLURM_SUBMIT_HOST}\""
echo " "

module load nvhpc
nvhpc_root=$(dirname $(dirname $(dirname $(which nvc))))
culibdir=$nvhpc_root/cuda/lib64
cuincdir=$nvhpc_root/cuda/include

mpicc test.c -I${cuincdir} -L${culibdir} -Wl,-rpath,${culibdir} -lcudart

mpiexec --bind-to none -n 2 ./a.out
```
