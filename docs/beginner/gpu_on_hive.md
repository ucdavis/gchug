# Using GPUs on HIVE

This guide is for basic HIVE users who want to run software on GPUs. It explains when a GPU helps, how to request one with SLURM, and how to choose between the `gpu-a100` and `low` partitions.

## What a GPU is good for

A GPU is a processor built to do many simple calculations at the same time. This is different from a CPU, which is better for general-purpose work and complex branching logic.

GPUs are useful for:

- Deep learning training and inference
- Protein structure prediction tools that use neural networks
- Molecular simulation or image processing programs with GPU support
- Running many similar numerical operations in parallel

GPUs are usually not useful for:

- File copying, compression, or text processing
- Small scripts that finish in seconds
- Software that was not written to use CUDA or GPUs
- Jobs where the bottleneck is disk I/O, downloading files, or waiting on a database

If a program does not explicitly support GPUs, requesting a GPU will not automatically make it faster. It will just reserve a scarce resource that your job may not use.

## The basic choice

For most users, the first decision is whether you need a specific reliable GPU resource or whether any available GPU is acceptable.

| Use case | Recommended partition | Why |
|----------|-----------------------|-----|
| You need a stable A100 job that should not be interrupted | `gpu-a100` | Higher-priority GPU partition, no preemption |
| Your job needs lots of GPU memory | `gpu-a100` | A100 GPUs have 80 GB of GPU memory |
| You are running many independent jobs and can restart them | `low` | More total GPUs are available and jobs can spread out more easily |
| Your code works on A100, A6000, and Blackwell GPUs | `low` with `--gres=gpu:1` | Lets SLURM give you any free GPU, including a random mixed GPU type |
| Your conda environment may not support Blackwell GPUs | `low` with a constraint or exclude list, or use `gpu-a100` | Avoids newer GPUs that may need a newer CUDA/software stack |

The `gpu-a100` partition is the safer choice for a single important long-running A100 job. The `low` partition is the throughput choice for restartable work, but it has lower priority, a shorter time limit, and jobs can be preempted and requeued.

## GPU hardware on HIVE

The GPU pool includes:

- 16 NVIDIA A100 GPUs
- 16 NVIDIA A6000 GPUs
- 4 NVIDIA RTX 6000 Blackwell GPUs, with 12 expected in the future

The advantage of `low` is that your job can use the broader mixed GPU pool instead of waiting for one specific GPU partition. The cost is that your job may land on any available GPU type unless you add a constraint.

The exact nodes and availability can change. To inspect the current GPU nodes from HIVE, use:

```bash
sinfo -o "%n %P %G %f" | grep gpu
```

This shows node names, partitions, GPU resources, and node features. The feature names matter if you want to request or avoid a specific GPU type.

## Requesting an A100 GPU

Use `gpu-a100` when you specifically want an A100, when your job is long-running, or when you do not want the job to be preempted.

Example `sbatch` script:

```bash
#!/bin/bash
#SBATCH --job-name=a100_job
#SBATCH --partition=gpu-a100
#SBATCH --account=your_account
#SBATCH --time=1-00:00:00
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=8
#SBATCH --mem=32G
#SBATCH --gres=gpu:a100:1
#SBATCH --output=%x_%j.out

module load conda/latest
conda activate myenv

python my_gpu_script.py
```

Submit it with:

```bash
sbatch run_a100_job.sh
```

Replace `your_account` with the account you should charge, such as your lab or department account.

## Requesting any GPU on `low`

The `low` partition is useful when you care more about throughput than about getting a specific GPU. If you request any GPU, SLURM can place your job on whatever compatible GPU is free. This can be helpful because there are many more GPUs reachable through the low-priority pool than through a single named GPU partition.

Example `sbatch` script:

```bash
#!/bin/bash
#SBATCH --job-name=low_gpu_job
#SBATCH --partition=low
#SBATCH --account=your_account
#SBATCH --time=12:00:00
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=8
#SBATCH --mem=32G
#SBATCH --gres=gpu:1
#SBATCH --requeue
#SBATCH --output=%x_%j.out

module load conda/latest
conda activate myenv

python my_gpu_script.py
```

The important line is:

```bash
#SBATCH --gres=gpu:1
```

This asks for one GPU without saying which kind. That gives the scheduler more freedom, so your jobs may start sooner and may run across more nodes.

## Why `low` can be good for parallel work

The `low` partition can be a good fit for workflows where you have many independent tasks. For example:

- Predict structures for 200 different proteins
- Score many input files independently
- Run many short model inference jobs
- Test many parameter combinations

Instead of running one giant job, use a SLURM array job so each task can run independently:

```bash
#!/bin/bash
#SBATCH --job-name=low_gpu_array
#SBATCH --partition=low
#SBATCH --account=your_account
#SBATCH --time=4:00:00
#SBATCH --array=1-100%20
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=4
#SBATCH --mem=16G
#SBATCH --gres=gpu:1
#SBATCH --requeue
#SBATCH --output=logs/%x_%A_%a.out

module load conda/latest
conda activate myenv

python run_one_input.py --task-id "$SLURM_ARRAY_TASK_ID"
```

In `#SBATCH --array=1-100%20`, the `1-100` creates 100 tasks and `%20` limits the job to 20 running tasks at a time. This is safer for the cluster and easier to restart if something fails.

## The tradeoff: `low` can be preempted

Jobs on `low` are lower priority. They can be preempted, which means SLURM may stop them when higher-priority jobs need the resources. On HIVE, `low` jobs are configured to requeue, so a preempted job can be put back in the queue and started again later.

Use `low` only when at least one of these is true:

- Your job is short enough that restarting is not a major problem
- Your program writes checkpoints and can resume from them
- Each task is independent, so losing one task does not ruin the whole workflow
- You are using an array job and can rerun failed tasks

Avoid `low` when:

- The job runs for days without checkpointing
- Restarting would waste a lot of expensive computation
- You need predictable completion for a deadline
- The software does not handle interruption cleanly

For preemptible work, include:

```bash
#SBATCH --requeue
```

Also write outputs in a way that is safe to restart. For array jobs, give each task its own output file instead of having many tasks write to the same file.

## Blackwell GPU warning

The RTX 6000 Blackwell GPUs are very new. Some programs installed through typical conda commands may not work on them yet, even if they work on A100 or A6000 GPUs.

This happens because conda GPU packages often include specific CUDA, PyTorch, TensorFlow, JAX, or compiled library versions. Older builds may not know how to run kernels on Blackwell GPUs.

Symptoms can include errors like:

- `no kernel image is available for execution on the device`
- CUDA runtime or driver compatibility errors
- PyTorch, TensorFlow, or JAX reporting that no usable GPU is available
- A program starting correctly on A100 but failing on Blackwell

Fixing this usually requires updating the software environment, using a newer CUDA-compatible build, or switching to a container built for the newer GPU architecture. That is an advanced software problem.

For beginner workflows, the simpler option is to avoid Blackwell GPUs until you know your program supports them.

## Avoiding Blackwell GPUs

If you want the flexibility of `low` but do not want a Blackwell GPU, use a constraint or exclude specific nodes.

First, inspect the available GPU features:

```bash
sinfo -o "%n %P %G %f" | grep gpu
```

If HIVE reports GPU features like `gpu:a100` and `gpu:a6000`, you can request only those older GPU types:

```bash
#SBATCH --partition=low
#SBATCH --gres=gpu:1
#SBATCH --constraint="gpu:a100|gpu:a6000"
```

If the feature names are different, adjust the constraint to match what `sinfo` shows.

Another option is to exclude known Blackwell nodes:

```bash
#SBATCH --partition=low
#SBATCH --gres=gpu:1
#SBATCH --exclude=node1,node2
```

Replace `node1,node2` with the actual Blackwell node names from `sinfo`. This is more fragile than using a constraint because node names can change or new Blackwell nodes can be added.

## Checking that your job got a GPU

Once your job starts, run:

```bash
hostname
nvidia-smi
echo "$CUDA_VISIBLE_DEVICES"
```

For PyTorch, you can also test:

```bash
python - <<'PY'
import torch

print("CUDA available:", torch.cuda.is_available())
if torch.cuda.is_available():
    print("GPU:", torch.cuda.get_device_name(0))
PY
```

If `nvidia-smi` works but your Python package does not see the GPU, the problem is probably your software environment rather than the SLURM request.

## Rule of thumb

Use `gpu-a100` when you need a reliable A100 job and do not want preemption. Use `low` when you have many restartable GPU tasks, your software can run on different GPU types, and you want access to the larger pool of available GPUs.
