# AlphaFold3

[AlphaFold](https://deepmind.google/technologies/alphafold/) is an AI system developed by Google that is used for predicting protein structures. Here we provide a brief description of how to run this model on AICR.

Your home institution may already have a module for running AlphaFold. You can check this by running `module avail`. Please reach out to your [Institutional Support Team](../getting-help.md#institutional-support-teams) for more information.

## Getting Started

For simplicity, in this example, we will create a folder in our home directory to use as our working directory:

```bash
mkdir ~/af3
export WORKDIR=~/af3
```

### Obtaining Model Weights

Due to licensing restrictions, each user needs to obtain the model weights individually, and ensure their use is in accordance with the [terms of use](https://github.com/google-deepmind/alphafold3/blob/main/WEIGHTS_TERMS_OF_USE.md).

To download the weights, run the following command:

```bash
cd $WORKDIR
wget https://storage.googleapis.com/alphafold3/af3.bin.zst
```

Then, decompress the weights into a `models` directory:

```bash
mkdir models
zstd -d af3.bin.zst -o models/af3.bin
rm af3.bin.zst
```

## Building the Container Image

Google DeepMind recommends running AlphaFold with Docker. Docker is not compatible with most HPC environments, so we need to use [Apptainer](../software/containers.md) instead. We have put together this [Apptainer definition file](../scripts/alphafold3/af3.def) that you can use to build the image, which is based on the [Dockerfile from the AlphaFold 3 repository](https://github.com/google-deepmind/alphafold3/blob/main/docker/Dockerfile).

Building container images can require significant computational resources. As such, you should build your image in a job on a compute node. You can use this batch script to do so:

```bash title="build_af3_image.sbatch"
#!/bin/bash

#SBATCH -N 1
#SBATCH -n 4
#SBATCH --mem-per-cpu=4G
#SBATCH -p cpu
#SBATCH -t 30
#SBATCH -o build_af3_image_%j.out

cd ~/af3

export APPTAINER_CACHEDIR=/scratch/$USER/.apptainer
export APPTAINER_TMPDIR=/scratch/$USER/.apptainer/tmp
mkdir -p $APPTAINER_TMPDIR

# wget https://raw.githubusercontent.com/mghpcc/aicr-docs-edit/main/docs/scripts/alphafold3/af3.def
wget https://raw.githubusercontent.com/mghpcc/aicr-docs-edit/sec/alphafold3/docs/scripts/alphafold3/af3.def
apptainer build af3.sif af3.def
```

<!-- TODO: Fix URL for def file -->

Submit this script by running `sbatch build_af3_image.sbatch`.

## Running AlphaFold3

The last thing you will need to run AlphaFold 3 is the AlphaFold dataset. This dataset is quite large, so your institution may have saved it in a shared directory to be available for use.

<!-- TODO: Confirm shared datasets directory existence -->

Now you should have everything you need to run AlphaFold 3. To test your setup, you can follow the below test case adapted from the [AlphaFold 3 GitHub Repository](https://github.com/google-deepmind/alphafold3).

From the working directory (`~/af3`), create an output directory and a test input file:

```bash
mkdir af_output
mkdir af_input
touch af_input/fold_input.json
```

Copy the following into `af_input/fold_input.json` (using `vim`, `emacs`, or `nano`):

```title="fold_input.json"
{
  "name": "2PV7",
  "sequences": [
    {
      "protein": {
        "id": ["A", "B"],
        "sequence": "GMRESYANENQFGFKTINSDIHKIVIVGGYGKLGGLFARYLRASGYPISILDREDWAVAESILANADVVIVSVPINLTLETIERLKPYLTENMLLADLTSVKREPLAKMLEVHTGAVLGLHPMFGADIASMAKQVVVRCDGRFPERYEWLLEQIQIWGAKIYQTNATEHDHNMTYIQALRHFSTFANGLHLSKQPINLANLLALSSPIYRLELAMIGRLFAQDAELYADIIMDKSENLAVIETLKQTYDEALTFFENNDRQGFIDAFHKVRDWFGDYSEQFLKESRQLLQQANDLKQG"
      }
    }
  ],
  "modelSeeds": [1],
  "dialect": "alphafold3",
  "version": 1
}
```

To run inference on this input using the container we built, you can use the `apptainer run` command along with the `--nv` flag to make the GPU available in the container.

```bash title="run_af3.sbatch"
#!/bin/bash

#SBATCH -N 1
#SBATCH -n 16
#SBATCH -p rtx-batch
#SBATCH -G 1
#SBATCH -J af3
#SBATCH -o run_af3_%j.out

# Enter the directory of the AF3 material:
WORKDIR=$HOME/af3

# Enter the path to the AF3 dataset and container image:
DATABASES_DIR=/path/to/dataset
IMAGE_PATH=$WORKDIR/af3.sif

apptainer run \
    --bind $DATABASES_DIR:/root/public_databases \
    --nv \
    $IMAGE_PATH \
    --json_path=$WORKDIR/af_input/fold_input.json \
    --model_dir=$WORKDIR/models \
    --output_dir=$WORKDIR/af_output \
    --db_dir=/root/public_databases
```

<!-- TODO: Fill in path to dataset -->

