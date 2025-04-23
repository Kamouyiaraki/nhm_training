# Using the job scheduler (Slurm)

Now you have all the files and programs you need, but there's a problem: lots of people are running huge data analyses simultaneously. That's why job scheduling systems have been developed: to ensure a fair distribution of the computing resources across all users and to allow the cluster administrators to manage such resources. HPC uses a standard open-source job scheduling system called Slurm. A [complete user guide](https://slurm.schedmd.com/tutorials.html) can be found on their website and ideally you should become familiar with it. 

## Partitions/queues explained

The cluster has four **partitions** (also known as **queues**) where you can submit your analyses (from here onwards sometimes defined as jobs) depending on their length and computational requirements: hour, day, week and month. As a general rule, the shorter the job and the less computationally expensive, the faster it gets assigned. Long analyses have lower priority.


| Partition | Use case                                                                                                           |
|-----------|--------------------------------------------------------------------------------------------------------------------|
| hour	    | Very short jobs, or things that can be sped up to less than an hour by multithreading                              |       
| day	    | Average jobs, or things that even with multithreading can take up to 24 hours                                      |
| week	    | Long jobs that take up to seven days, either because they cannot be parallelised or because they have low priority |
| month	    | Non-important jobs that can run for a long time and have low priority                                              |

Additionally, there is an **interactive** partition, that can be used to test computationally intensive tasks (almost any job really) to see how they perform, with a strict control of the used resources. There are many ways of using interactive, but it should not be abused as it can reduce the priority assigned to your future analyses. This is because the scheduling system calculates the priority of your jobs depending on how much you're using the cluster, and usually when you use an interactive session you're requesting a lot of resources (most of which you aren't even using).

## How to submit a job

- [Example 1](#example-1---running-a-job-using-software-installed-with-conda) - Running a job using software installed with conda
- [Example 2](#example-2---running-a-job-using-software-from-the-shared-software-area) - Running a job using software from the shared software area
- [Example 3](using-the-job-scheduler.md/#example-3---running-a-job-using-a-singularity-container) - Running a job using a Singularity container
- [Example 4](#example-4---running-a-job-using-the-gpu) - Running a job using the GPU

### Example 1 - Running a job using software installed with conda

The first step is to create a file that contains all the instructions for the job scheduler to run the analysis. Here is an example file which I've named slurm_script.sh:

```
#!/bin/bash

#SBATCH -J rob-job
#SBATCH -p day
#SBATCH --mem=2GB
#SBATCH -c 2
#SBATCH -e /home/robtest1234/demo/test2/job.%J.err
#SBATCH -o /home/robtest1234/demo/test2/job.%J.out
#SBATCH -w hpc-gpu-002
#SBATCH --mail-user=robtest1234@nhm.ac.uk
#SBATCH --mail-type=ALL

source ~/miniconda3/etc/profile.d/conda.sh
conda init bash
conda activate base

echo "Starting at `date`"
echo "Running on hosts: $SLURM_NODELIST"
echo "Running on $SLURM_NNODES nodes"
echo "Current workig directory is `pwd`"

python python_script.py
```

The `#SBATCH` lines allow you to define various options for your job, as described below. Most of these options are not required, and there are many more that are not listed here, but they are important for optimization when the cluster gets busy and lots of people are submitting jobs to the same queue/partition:

```
-J			    Job name
-p			    Partition (queue)
--mem			Memory
-c			    Number of CPUs
-e			    Output file for standard error
-o 			    Output file for standard output
--mail-user		Who to send email notification for job state changes
--mail-type		Notify on state change: BEGIN, END, FAIL or ALL
```

Further options are described [here](https://slurm.schedmd.com/sbatch.html). 

These lines are required if you are using conda from your home directory:
```
source ~/miniconda3/etc/profile.d/conda.sh
conda init bash
```

This line allows you to activate a conda environment called `base`:
```
conda activate base
```

The `echo` commands are not required, but they demonstrate how you can add additional output for your job:
```
echo "Starting at `date`"
echo "Running on hosts: $SLURM_NODELIST"
echo "Running on $SLURM_NNODES nodes."
echo "Current working directory is `pwd`"
```
Finally, this is the actual command that you're running. In this example I'm using Python to run a script called `python_script.py`, which is located in the same directory as my `slurm_script.sh`:
```
python python_script.py
```

Use `sbatch` to submit your job:
```
sbatch <script_name>
```

For example:
```
robtest1234@hpc-head-002:~/demo/test2$ sbatch slurm_script.sh
Submitted batch job 965216
```

> If your analysis produces intermediate files between steps, you should set up a scratch folder for temporary files in `/mbl/share/scratch`. Ideally, clean up after your job. Scratch is set to auto-delete files after 21 days.

### Example 2 - Running a job using software from the shared software area

Here is another sample job:

![terminal-slurm-example-2](images/terminal-slurm-example-2.png)

This time, instead of using Conda, it is using the `ml` command to load the `AdapterRemoval` software from the shared software area:
```
ml adapterremoval
```
Notice that instead of having a fixed sample name/ID, you can add a variable to your `.sh` file. The variable `NAME` allows you to make the shell script more flexible, as you do not have to modify it any time you have a different sample name/ID:
```
NAME=$1
```
Once the Slurm file is ready, you just need to submit the job:

```
sbatch slurm_script.sh SRR001
```

Or:
```
sbatch slurm_script.sh SRR002
```

If you have a text file listing all your sample names/IDs (in this case data/runids.txt), you can even use a **for** loop to submit several jobs simultaneously (there is also a way to do it internally in the script using an array job, but that is a more complex topic):
```
for i in $(cat data/runids.txt); do sbatch slurm_script.sh $i; done
```

If there are enough computational resources the jobs will run immediately (or at least some of them), otherwise they will be placed in a queue. 

### Example 3 - Running a job using a Singularity container

Here is another sample job, this time using a Singularity container. Note the use of backslashes at the end of the lines, as the `singularity` command is split across multiple lines for readability:
```
#!/bin/bash

#SBATCH --partition=hour
#SBATCH -c 2
#SBATCH -e /home/robtest1234/demo/test3/job.%J.err
#SBATCH -o /home/robtest1234/demo/test3/job.%J.out

singularity run \
--bind /workspaces/groups/rob-project-2:/mnt \
/workspaces/groups/singularity-images/audiowaveform/1.7.0/audiowaveform.sif \
audiowaveform --version
```

This line means that we're going to run a Singularity container:
```
singularity run
```

By default, Singularity can only access data in your home folder. However, you can use the `--bind` option to mount additional folders inside the container, like this:
```
--bind /path/on/host:/path/in/container 
```

This would be useful if, for example, the job needed to read or write data in the `/workspaces/groups/rob-project-2` folder:

```
--bind /workspaces/groups/rob-project-2:/mnt
```
This line refers to the location of the singularity image:
```
/workspaces/groups/singularity-images/audiowaveform/1.7.0/audiowaveform.sif
```

This is the application or command that we want to execute inside the container:
```
audiowaveform --version
```

### Example 4 - Running a job using the GPU

Here is a sample job that makes use of the GPU. Note that only one GPU job can run at a time &ndash; if someone else's job is using the GPU, your job will wait in the queue until theirs is finished.

```
#!/bin/bash

#SBATCH -J rob-biomedisa
#SBATCH -p hour
#SBATCH --mem=2GB
#SBATCH -c 2
#SBATCH --gres=gpu
#SBATCH -e /home/robtest1234/demo/test4/job.%J.err
#SBATCH -o /home/robtest1234/demo/test4/job.%J.out
#SBATCH --mail-user=robtest1234@nhm.ac.uk
#SBATCH --mail-type=ALL

singularity run --nv \
/workspaces/groups/singularity-images/biomedisa/23.01.1/biomedisa.sif \
python3 /biomedisa/biomedisa_features/pycuda_test.py
```

This line is required to allow Slurm to access the GPU:
```
#SBATCH --gres=gpu
```

Like [Example 3](using-the-job-scheduler.md/#example-3---running-a-job-using-a-singularity-container), this job runs in a Singularity container. The `--nv` option is necessary to allow Singularity to access the GPU.

## Viewing and managing your jobs

You can use `squeue` to view the status of jobs in the queue:
```
squeue -l
```

For example:
```
robtest1234@hpc-head-002:~$ squeue -l
Tue Apr 22 13:10:03 2025
             JOBID PARTITION     NAME     USER    STATE       TIME TIME_LIMI  NODES NODELIST(REASON)
            965037       day 11-ancIB   alexs4  RUNNING    1:31:05 1-00:00:00      1 hpc-gpu-002
            965226       day snakejob   danip3  RUNNING      29:19 1-00:00:00      1 hpc-cpu-001
            965227       day snakejob   danip3  RUNNING      29:07 1-00:00:00      1 hpc-cpu-001
            965230       day snakejob   danip3  RUNNING      27:05 1-00:00:00      1 hpc-cpu-001
            965274       day snakejob   danip3  RUNNING       0:02 1-00:00:00      1 hpc-cpu-001
            965275       day snakejob   danip3  RUNNING       0:02 1-00:00:00      1 hpc-cpu-001
          915974_4     month 2-aDNA_P   alexs4  RUNNING 7-07:54:29 30-00:00:00      1 hpc-cpu-001
            954429     month raxml_ng    beatl  RUNNING 8-16:32:45 30-00:00:00      1 hpc-cpu-001
          957954_4     month EAGenome   amelr1  RUNNING 7-19:08:15 30-00:00:00      1 hpc-gpu-002
            963590     month ALLSAMPL   amelr1  RUNNING 4-19:48:48 30-00:00:00      1 hpc-gpu-002
            963605     month ONLY3XCo   amelr1  RUNNING 4-19:46:39 30-00:00:00      1 hpc-gpu-002
            964163     month 3-parall   alexs4  RUNNING 3-04:31:30 30-00:00:00      1 hpc-cpu-006
            964249     month 4-parall   alexs4  RUNNING 2-07:02:27 30-00:00:00      1 hpc-cpu-006
            964294      week unicycle    larav  RUNNING 1-13:06:46 6-21:00:00      1 hpc-gpu-002
            964295      week unicycle    larav  RUNNING 1-13:03:04 6-21:00:00      1 hpc-gpu-002
            964296      week unicycle    larav  RUNNING 1-13:01:30 6-21:00:00      1 hpc-cpu-001
            964738      week mge_fast   danip3  RUNNING    3:14:39 7-00:00:00      1 hpc-cpu-006
            964739      week solver.s    zekuw  RUNNING    3:12:22 7-00:00:00      1 hpc-cpu-006
```

While your job is running, you can use `scontrol` to check its status:
```
scontrol show jobid <job_id>
```

For example:
```
robtest1234@hpc-head-002:~/demo/test4$ scontrol show jobid 965279
JobId=965279 JobName=rob-biomedisa
   UserId=robtest1234(1399642667) GroupId=domain users(1399600513) MCS_label=N/A
   Priority=4294715614 Nice=0 Account=core_research_labs QOS=normal
   JobState=COMPLETED Reason=None Dependency=(null)
   Requeue=1 Restarts=0 BatchFlag=1 Reboot=0 ExitCode=0:0
   RunTime=00:00:04 TimeLimit=01:00:00 TimeMin=N/A
   SubmitTime=2025-04-22T13:11:19 EligibleTime=2025-04-22T13:11:19
   AccrueTime=2025-04-22T13:11:19
   StartTime=2025-04-22T13:11:19 EndTime=2025-04-22T13:11:23 Deadline=N/A
   SuspendTime=None SecsPreSuspend=0 LastSchedEval=2025-04-22T13:11:19 Scheduler=Main
   Partition=hour AllocNode:Sid=hpc-head-002:1996908
   ReqNodeList=(null) ExcNodeList=(null)
   NodeList=hpc-gpu-002
   BatchHost=hpc-gpu-002
   NumNodes=1 NumCPUs=2 NumTasks=1 CPUs/Task=2 ReqB:S:C:T=0:0:*:*
   TRES=cpu=2,node=1,billing=2
   Socks/Node=* NtasksPerN:B:S:C=0:0:*:* CoreSpec=*
   MinCPUsNode=2 MinMemoryNode=2G MinTmpDiskNode=0
   Features=(null) DelayBoot=00:00:00
   OverSubscribe=OK Contiguous=0 Licenses=(null) Network=(null)
   Command=/gpfs/nhmfsa/bulk/share/data/mbl/share/workspaces/users/robtest1234/demo/test4/slurm_script.sh
   WorkDir=/gpfs/nhmfsa/bulk/share/data/mbl/share/workspaces/users/robtest1234/demo/test4
   StdErr=/home/robtest1234/demo/test4/job.%J.err
   StdIn=/dev/null
   StdOut=/home/robtest1234/demo/test4/job.%J.out
   Power=
   TresPerNode=gres:gpu
   MailUser=robtest1234@nhm.ac.uk MailType=INVALID_DEPEND,BEGIN,END,FAIL,REQUEUE,STAGE_OUT
```

Once your job is finished, you can use `sacct` to view details of your job. This is very useful information, because it will allow you to define the exact requirements of your jobs and make good use of the computational resources:
```
sacct --format=User,JobID,Jobname,partition,state,time,start,end,elapsed,MaxRss,MaxVMSize,nnodes,ncpus,nodelist -j <job_id>
```

For example:
```
robtest1234@hpc-head-002:~$ sacct --format=User,JobID,Jobname,partition,state,time,start,end,elapsed,MaxRss,MaxVMSize,nnodes,ncpus,nodelist -j 965279
     User JobID           JobName  Partition      State  Timelimit               Start                 End    Elapsed     MaxRSS  MaxVMSize   NNodes      NCPUS        NodeList
--------- ------------ ---------- ---------- ---------- ---------- ------------------- ------------------- ---------- ---------- ---------- -------- ---------- ---------------
robtest1+ 965279       rob-biome+       hour  COMPLETED   01:00:00 2025-04-22T13:11:19 2025-04-22T13:11:23   00:00:04                              1          2     hpc-gpu-002
          965279.batch      batch             COMPLETED            2025-04-22T13:11:19 2025-04-22T13:11:23   00:00:04      2484K    143144K        1          2     hpc-gpu-002
```

In this example, we see the job completed successfully, it lasted five seconds, it used a maximum of 200MB of RAM, it ran on hpc-gpu-001, and so on - CHANGE THIS TO NEW EXAMPLE.

You can use `sinfo -l` to check that status of the cluster nodes. For example:
```
robtest1234@hpc-head-002:~$ sinfo -l
Tue Apr 22 13:14:46 2025
PARTITION   AVAIL  TIMELIMIT   JOB_SIZE ROOT OVERSUBS     GROUPS  NODES       STATE NODELIST
month          up 30-00:00:0 1-infinite   no       NO        all      2       mixed hpc-cpu-001,hpc-gpu-002
month          up 30-00:00:0 1-infinite   no       NO        all      1   allocated hpc-cpu-006
week           up 7-00:00:00 1-infinite   no       NO        all      2       mixed hpc-cpu-001,hpc-gpu-002
week           up 7-00:00:00 1-infinite   no       NO        all      1   allocated hpc-cpu-006
day*           up 1-00:00:00 1-infinite   no       NO        all      2       mixed hpc-cpu-001,hpc-gpu-002
day*           up 1-00:00:00 1-infinite   no       NO        all      1   allocated hpc-cpu-006
hour           up    1:00:00 1-infinite   no       NO        all      2       mixed hpc-cpu-001,hpc-gpu-002
hour           up    1:00:00 1-infinite   no       NO        all      1   allocated hpc-cpu-006
sip            up 30-00:00:0 1-infinite   no       NO        all      2       mixed hpc-cpu-001,hpc-gpu-002
sip            up 30-00:00:0 1-infinite   no       NO        all      1   allocated hpc-cpu-006
sip            up 30-00:00:0 1-infinite   no       NO        all      1        idle hpc-cpu-005
gpu            up 30-00:00:0 1-infinite   no       NO        all      1    drained* hpc-gpu-001
galaxy         up 30-00:00:0 1-infinite   no       NO        all      2       mixed hpc-cpu-001,hpc-gpu-002
galaxy         up 30-00:00:0 1-infinite   no       NO        all      1   allocated hpc-cpu-006
interactive    up 1-00:00:00 1-infinite   no       NO        all      2       mixed hpc-cpu-001,hpc-gpu-002
interactive    up 1-00:00:00 1-infinite   no       NO        all      1   allocated hpc-cpu-006
ts             up 1-00:00:00 1-infinite   no       NO        all      1    drained* hpc-gpu-001
ts             up 1-00:00:00 1-infinite   no       NO        all      2       mixed hpc-cpu-001,hpc-gpu-002
ts             up 1-00:00:00 1-infinite   no       NO        all      1   allocated hpc-cpu-006
ts             up 1-00:00:00 1-infinite   no       NO        all      1        idle hpc-cpu-005
```

In the example below we can see that `hpc-gpu-001` is in a **mixed** state, meaning it is running jobs, but has free capacity to run more jobs. And we can see that `hpc-gpu-002` is in an **idle** state, meaning it is not currently running any jobs, and is ready to receive new jobs - CHANGE TO NEW EXAMPLE.

Others common states you may see:
- **allocated** &ndash; this means the node is fully utilized by running jobs, so new jobs must wait in the queue.
- **drained** &ndash; this means the node is unable to to receive new jobs, either due to an error or due to scheduled maintenance.

You can cancel your running job with `scancel <job_id>`. For example:
```
scancel 965310
```

Look at your own jobs in the queue with `squeue -u <username>`. For example: 
```
robtest1234@hpc-head-002:~/demo/test4$ squeue -u robtest1234
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
            965321      hour rob-biom robtest1 PD       0:00      1 (Resources)
            965322      hour rob-biom robtest1 PD       0:00      1 (Priority)
            965320      hour rob-biom robtest1  R       0:02      1 hpc-gpu-002
```
