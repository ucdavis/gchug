# 1. Job scheduling with slurm

# 1.1 srun

Slurm schedules _interactive sessions_ with `srun`

When you request an _interactive session_, you are telling slurm (the job manager) to give you a terminal shell to a compute node that has whatever resources you want. Here's the syntax of an srun command

```sh
srun --partition=high --time=01:00:00 --mem=2G --account=genome-center-grp -c 1 -n 1 -N 1 --pty bash
```

1. `--partition=high`: specifies that we want to be in the "high priority partition"
2. `--time=01:00:00`: specifies that we want this interactive job to last for 2 hours (you can manually cancel it before the elapsed time as well)
3. `--mem=2g`: species that we want 2g of ram in total
4. `--account=genome-center-grp`: specifies that we want to access the resources in the genome-center-grp group
5. `-c 1`: specifies that we want 4 cores/nodes
6. `-n 1`: specifies that the number of tasks (processes per node) is 1
7. `-N 1`: specifies that we want 1 node
8. `--pty bash`: specifies that we want to allocate these resources and start a bash shell on the compute node

When you run this command, your request (job) is added to the queue and resources will be allocated when they are available

```sh
username@login2$ srun --partition=high --time=01:00:00 --mem=2G --account=genome-center-grp -c 1 -n 1 -N 1 --pty bash
# srun: job 13003912 queued and waiting for resources
# srun: job 13003912 has been allocated resources

username@hive-dc-7-6-38$ 
```

Now, you will be on a _compute node_. You can cancel the `srun` session before the time has elapsed with `exit`

```sh
username@hive-dc-7-6-38$ exit
# your srun session will be killed and you will be sent back to the login node
username$login2$ 
```

## 1.2 sbatch

It is more efficient to submit your job to the scheduler compared to requesting an interative session with `srun`. This way, you can submit a job script and have it run in the background without any additional interference. 

`sbatch` is built for this. Unlike interactive jobs, you write a slurm script that contains information about all the resources you need and the code you want to run, and then submit that to the cluster with `sbatch`.

Here's an example slurm script that I could submit with sbatch. The headers (start with `#SBATCH`) detail the resources you want.

Since a job submitted with sbatch won't source `~/.bashrc` (like srun does), you need to initialize your conda so that you can activate environments and use tools

```bash
#!/bin/bash
#SBATCH --account=genome-center-grp
#SBATCH --partition=high
#SBATCH --job-name=myjob
#SBATCH --nodes=1
#SBATCH --cpus-per-task=1
#SBATCH --mem=1G
#SBATCH --time=00:01:00
#SBATCH --output=/path/to/output.log
#SBATCH --error=/path/to/error.log

# initialize conda in the job shell
. "$HOME/miniforge3/etc/profile.d/conda.sh"
conda activate minimap2

echo "Hello World!"
```

Save all this into a slurm script, like a file called `hello_world.slurm`. You can then type

```sh
sbatch hello_world.slurm
```

To _submit_ this job to the scheduler and it will be added to the queue.

## 1.3 A note on resources

> [!CAUTION]
> When you request an `srun` session or submit an `sbatch` job with certain resources, you get access to all of them. This means that if you have requested 1000Gb RAM and 64 CPU for 48 hours, you get all of it. Even if the command you run only utilizes 2Gb memory on 1 CPU and it takes 10 minutes, all of the requested resources are still blocked off for your use. 
> 
> So be mindful of the resources you request! The slurm output logs will tell you the fraction of the requested resources that your job actually used. So read the logs and get a better idea of what resources certain commands actually need. 
> 
> Additionally, your `srun` session does not stop until the time elapses or you cancel it. So please end your srun sessions if you finish your work before the elapsed time so those resources are freed up for other people.

## 1.4 Monitoring job status

You can check on a job's status by typing in `sacct --allocations`

```sh
squeue --me
```

Jobs can be cancelled using `scancel JOBID`. from  `squeue --me`, you'll be able to see the ID associated with all your recent and actively running jobs.

```sh
scancel 3434685
```

# todo
- [ ] Add links to docs for best practices with slurm scheduling including
  - using screens/tmux
  - more advanced sbatch (arrays)
  - monitoring resources
