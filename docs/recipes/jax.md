# JAX on AICR

JAX is a high-performance numerical computing library from Google that combines NumPy-like syntax with automatic differentiation and XLA (Accelerated Linear Algebra) compilation. It is widely used for scientific computing, physics-informed neural networks, molecular dynamics, and research that requires custom gradient computation.

## What Is JAX?

JAX provides three core capabilities:

- **NumPy API** -- `jax.numpy` is a drop-in replacement for NumPy that runs on GPUs
- **Automatic differentiation** -- `jax.grad` differentiates arbitrary Python functions, including higher-order derivatives
- **XLA compilation** -- `jax.jit` compiles functions to optimized GPU kernels via XLA, often outperforming hand-written CUDA

JAX is functional by design: functions are pure (no side effects), and arrays are immutable. This enables aggressive compiler optimizations and straightforward parallelism.

## Installing JAX

=== "Conda"

    Install JAX with CUDA support in a Conda environment:

    ```bash
    module load conda
    conda create -n jax python=3.12
    conda activate jax
    pip install "jax[cuda13]"
    ```

    !!! note
        JAX's CUDA wheels are installed via pip even within a Conda environment. The `jax[cuda13]` extra pulls in `jaxlib` with CUDA 13 support.

=== "Pip"

    Install JAX with CUDA support in a virtual environment:

    ```bash
    module load conda
    python3 -m venv jax-env
    source jax-env/bin/activate
    pip install "jax[cuda13]"
    ```

=== "Container"

    Use the NVIDIA JAX container from NGC for a pre-configured environment:

    ```bash
    apptainer pull jax.sif docker://nvcr.io/nvidia/jax:24.04-py3
    ```

    Run your script with GPU support:

    ```bash
    apptainer exec --nv jax.sif python my_script.py
    ```

## Verifying GPU Access

Confirm that JAX detects the [GPUs allocated to your job](../running-jobs/gpu-jobs.md) with:

```bash
python3 -c "
import jax
print('Devices:', jax.devices())
print('Device count:', jax.device_count())
print('Default backend:', jax.default_backend())
"
```

Expected output on a B200 node with 8 GPUs:

```
Devices: [CudaDevice(id=0), CudaDevice(id=1), ..., CudaDevice(id=7)]
Device count: 8
Default backend: gpu
```

!!! warning
    If `jax.devices()` returns CPU devices only, verify that you requested GPUs in your Slurm script and that JAX was installed with CUDA support (`jax[cuda13]`).

## Single-GPU Example

A simple JAX program that trains a linear regression model:

```python title="train_jax.py"
import jax
import jax.numpy as jnp
from jax import grad, jit

# Generate synthetic data
key = jax.random.PRNGKey(42)
X = jax.random.normal(key, (1000, 10))
true_w = jnp.ones(10)
y = X @ true_w + 0.1 * jax.random.normal(key, (1000,))

# Model: linear regression
def loss_fn(w, X, y):
    pred = X @ w
    return jnp.mean((pred - y) ** 2)

grad_fn = jit(grad(loss_fn))

# Training loop
w = jnp.zeros(10)
lr = 0.01

for step in range(500):
    g = grad_fn(w, X, y)
    w = w - lr * g
    if step % 100 == 0:
        print(f"Step {step}, loss: {loss_fn(w, X, y):.6f}")
```

Run this script on a single GPU with the following Slurm batch script:

```bash
#!/bin/bash
#SBATCH --job-name=jax-single
#SBATCH --partition=rtx-batch
#SBATCH --gpus=1
#SBATCH --cpus-per-task=8
#SBATCH --mem=32G
#SBATCH --time=00:10:00
#SBATCH --output=%x-%j.out

source jax-env/bin/activate

python train_jax.py
```

## Multi-GPU with JAX

JAX supports multi-GPU computation through its sharding API. Unlike PyTorch, JAX does not require launching separate processes per GPU. A single JAX process manages all GPUs on a node.

### Single-Node Multi-GPU (Automatic Sharding)

Use `jax.sharding` to distribute arrays and computation across GPUs:

```python title="train_jax_multigpu.py"
import jax
import jax.numpy as jnp
from jax.sharding import Mesh, PartitionSpec, NamedSharding

# Create a mesh of devices
devices = jax.devices()
mesh = Mesh(devices, axis_names=("batch",))

# Shard data across GPUs along the batch dimension
sharding = NamedSharding(mesh, PartitionSpec("batch"))

# Create sharded data
data = jax.random.normal(jax.random.PRNGKey(0), (8192, 512))
data = jax.device_put(data, sharding)

# JIT-compiled function runs in parallel across devices
@jax.jit
def process(x):
    return jnp.sin(x) @ jnp.ones((512, 256))

result = process(data)
print(f"Result shape: {result.shape}, sharding: {result.sharding}")
```

Run this script on a single node with multiple GPUs:

```bash
#!/bin/bash
#SBATCH --job-name=jax-multigpu
#SBATCH --partition=rtx-batch
#SBATCH --nodes=1
#SBATCH --gpus=4
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=32
#SBATCH --mem-per-cpu=4G
#SBATCH --time=00:30:00
#SBATCH --output=%x-%j.out

source jax-env/bin/activate

python train_jax_multigpu.py
```

### Multi-Node with jax.distributed

For multi-node JAX workloads, initialize `jax.distributed` and use `srun` to launch one process per node:

```bash
#!/bin/bash
#SBATCH --job-name=jax-multinode
#SBATCH --partition=rtx-batch
#SBATCH --nodes=2
#SBATCH --gpus-per-node=8
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=64
#SBATCH --mem=0
#SBATCH --time=1:00:00
#SBATCH --output=%x-%j.out

source jax-env/bin/activate

export MASTER_ADDR=$(scontrol show hostnames $SLURM_JOB_NODELIST | head -n 1)

srun --mpi=pmix python train_jax_distributed.py
```

!!! note
    `--mem=0` requests all available memory on the node, and `--ntasks-per-node=1` ensures one JAX process per node. Set `MASTER_ADDR` to the first node's hostname for inter-node communication.

In `train_jax_distributed.py`, initialize the distributed runtime, then build a mesh across every device on every node and shard the training data along it:

```python title="train_jax_distributed.py"
import jax
import jax.numpy as jnp
from jax import grad, jit
from jax.sharding import Mesh, PartitionSpec, NamedSharding

jax.distributed.initialize()

if jax.process_index() == 0:
    print(f"Global devices: {jax.device_count()}")
    print(f"Local devices: {jax.local_device_count()}")

# Mesh spanning every GPU on every node
devices = jax.devices()
mesh = Mesh(devices, axis_names=("batch",))
sharding = NamedSharding(mesh, PartitionSpec("batch"))

# Generate synthetic data and shard it across all nodes/GPUs
key = jax.random.PRNGKey(42)
X = jax.random.normal(key, (8192, 10))
true_w = jnp.ones(10)
y = X @ true_w + 0.1 * jax.random.normal(key, (8192,))

X = jax.device_put(X, sharding)
y = jax.device_put(y, sharding)

# Model: linear regression
def loss_fn(w, X, y):
    pred = X @ w
    return jnp.mean((pred - y) ** 2)

grad_fn = jit(grad(loss_fn))

# Training loop -- each process computes on its local shard,
# gradients are combined automatically across the mesh
w = jnp.zeros(10)
lr = 0.01

for step in range(500):
    g = grad_fn(w, X, y)
    w = w - lr * g
    if step % 100 == 0 and jax.process_index() == 0:
        print(f"Step {step}, loss: {loss_fn(w, X, y):.6f}")
```

!!! note
    `jax.distributed.initialize()` uses environment variables set by Slurm and MPI to discover all nodes. After initialization, `jax.devices()` returns GPUs from all nodes.

## Common JAX Libraries

JAX has a modular ecosystem. Common companion libraries:

| Library | Purpose | Install |
|---------|---------|---------|
| [Flax](https://flax.readthedocs.io/) | Neural network library (the `nn.Module` of JAX) | `pip install flax` |
| [Optax](https://optax.readthedocs.io/) | Gradient transformations and optimizers (Adam, SGD, etc.) | `pip install optax` |
| [Orbax](https://orbax.readthedocs.io/) | Checkpointing for JAX models | `pip install orbax-checkpoint` |
| [Equinox](https://docs.kidger.site/equinox/) | PyTorch-like neural networks in JAX | `pip install equinox` |
| [Diffrax](https://docs.kidger.site/diffrax/) | Differential equation solvers | `pip install diffrax` |

A typical Flax + Optax training pattern:

```python
import flax.linen as nn
import optax

class MLP(nn.Module):
    hidden_dim: int
    out_dim: int

    @nn.compact
    def __call__(self, x):
        x = nn.Dense(self.hidden_dim)(x)
        x = nn.relu(x)
        x = nn.Dense(self.out_dim)(x)
        return x

model = MLP(hidden_dim=256, out_dim=10)
params = model.init(jax.random.PRNGKey(0), jnp.ones((1, 784)))

optimizer = optax.adam(learning_rate=1e-3)
opt_state = optimizer.init(params)
```

## JAX vs. PyTorch

| Aspect | JAX | PyTorch |
|--------|-----|---------|
| Programming model | Functional (pure functions, immutable arrays) | Imperative (in-place ops, mutable tensors) |
| Compilation | XLA (`jax.jit`) -- compiles entire functions | `torch.compile` -- optional, graph capture |
| Autodiff | `jax.grad` on any Python function | `loss.backward()` on tensor graph |
| Multi-GPU (single node) | Single process, automatic sharding | One process per GPU (`torchrun`) |
| Ecosystem | Flax, Optax, Equinox (modular) | torchvision, torchaudio, HuggingFace (large) |
| Best for | Custom research, scientific computing, novel architectures | Production ML, large model ecosystem, community support |

!!! tip
    If you are starting a new project and your workflow is standard (image classification, NLP fine-tuning, etc.), PyTorch and its ecosystem will get you running faster. If you need custom differentiation, higher-order gradients, or XLA-level performance tuning, JAX is a strong choice.
