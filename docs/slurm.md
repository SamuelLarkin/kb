# SLURM

## Access Internet from a Node

### GPSC-C

```sh
export https_proxy=http://webproxy.collab.science.gc.ca:8888
export http_proxy=http://webproxy.collab.science.gc.ca:8888
```

### GPSC

```sh
export https_proxy=http://webproxy.science.gc.ca:8888
export http_proxy=http://webproxy.science.gc.ca:8888
```

## Job Command and Information

### Update a job

Add a dependency to run a job after another job.

```sh
scontrol update jobid=<JOBID> dependency=afterok:<PREVIOUS_JOBID>
```

Add dependencies to multiple jobs

```sh
scontrol \
  update \
    jobid=$(echo {15882..15893} | tr ' ' ',')
    dependency=afterok:15754:15756:15757:15758:15765:15766:15767:15740:15743
```

Change the time limit.

```sh
scontrol update jobid=<JOBID> TimeLimit=<NEW_TIMELIMIT>
```

Change the number of maximum concurrent tasks.

```sh
scontrol update jobid=<JOBID> ArrayTaskThrottle=0
```

### A Scriptless Job

[Running a binary without a top level script in SLURM](https://stackoverflow.com/a/33402070)

```sh
 sbatch \
   --time=00:20:00 \
   --partition=TrixieMain,JobTesting \
   --account=dt-mtp \
   --nodes=1 \
   --ntasks-per-node=1 \
   --cpus-per-task=40 \
   --mem=40G \
   --wrap='time python -m sockeye.translate --output-type json --batch-size 32 --models base_model --input corpora/validation.en --use-cpu > model/decode.output.0.00000.json'
```

### Job's Information

List detailed information for a job (useful for troubleshooting):

```sh
scontrol show jobid --details <JOBID>
```

```
JobId=403641 JobName=HoC-CL.training
   UserId=larkins(171967808) GroupId=larkins(171967808) MCS_label=N/A
   Priority=68528 Nice=0 Account=dt-mtp QOS=normal
   JobState=PENDING Reason=ReqNodeNotAvail,_Reserved_for_maintenance Dependency=(null)
   Requeue=1 Restarts=1 BatchFlag=1 Reboot=0 ExitCode=0:0
   DerivedExitCode=0:0
   RunTime=00:00:00 TimeLimit=12:00:00 TimeMin=N/A
   SubmitTime=2024-01-19T04:39:27 EligibleTime=2024-01-19T04:41:28
   AccrueTime=2024-01-19T04:41:28
   StartTime=2024-01-21T14:30:00 EndTime=2024-01-22T02:30:00 Deadline=N/A
   SuspendTime=None SecsPreSuspend=0 LastSchedEval=2024-01-19T10:13:15
   Partition=TrixieMain AllocNode:Sid=hn2:20782
   ReqNodeList=(null) ExcNodeList=cn119
   NodeList=(null)
   BatchHost=cn120
   NumNodes=1 NumCPUs=24 NumTasks=4 CPUs/Task=6 ReqB:S:C:T=0:0:*:*
   TRES=cpu=24,mem=96G,node=1,billing=24,gres/gpu=4
   Socks/Node=* NtasksPerN:B:S:C=4:0:*:* CoreSpec=*
   MinCPUsNode=24 MinMemoryNode=96G MinTmpDiskNode=0
   Features=(null) DelayBoot=00:00:00
   OverSubscribe=OK Contiguous=0 Licenses=(null) Network=(null)
   Command=/gpfs/projects/DT/mtp/models/HoC-ContinualLearning/nmt/tools/train.sh
   WorkDir=/gpfs/projects/DT/mtp/models/HoC-ContinualLearning/nmt/en2fr/baseline_initial
   Comment=House of Commons Continual Learning NMT training
   StdErr=/gpfs/projects/DT/mtp/models/HoC-ContinualLearning/nmt/en2fr/baseline_initial/HoC-CL.training-403641.out
   StdIn=/dev/null
   StdOut=/gpfs/projects/DT/mtp/models/HoC-ContinualLearning/nmt/en2fr/baseline_initial/HoC-CL.training-403641.out
   Power=
   TresPerNode=gpu:4
```

Find out where a job will run

```sh
scontrol show job <JOBID>
```

### Tabulate the Information of Queued jobs

If you are running a lot of jobs and need to inspect some of them for some feature, example, what command they will run, use `mlr`.

```
squeue --Format='JobID' --user=$USER --noheader \
| \parallel 'scontrol --oneliner show job {}' \
| sed 's/ /,/g' \
| mlr --opprint cat \
| less
```

### Stats of a job

Get some stats about a job that ran on `Slurm`.
Get stats about a job that has finished.

```sh
sacct --long --jobs=<JOBID>
sacct -l -j <JOBID>
```

```sh
seff 112610
```

```
Job ID: 112610
Cluster: trixie-rhel9
User/Group: larkins/larkins
State: COMPLETED (exit code 0)
Nodes: 1
Cores per node: 32
CPU Utilized: 00:02:17
CPU Efficiency: 3.79% of 01:00:16 core-walltime
Job Wall-clock time: 00:01:53
Memory Utilized: 2.78 GB
Memory Efficiency: 6.96% of 40.00 GB
```

## Cluster information

See what the nodes really offer.

```sh
scontrol show nodes
```

```
NodeName=cn136 Arch=x86_64 CoresPerSocket=16
   CPUAlloc=0 CPUTot=64 CPULoad=0.01
   AvailableFeatures=(null)
   ActiveFeatures=(null)
   Gres=gpu:4
   NodeAddr=cn136 NodeHostName=cn136
   OS=Linux 3.10.0-1160.62.1.el7.x86_64 #1 SMP Tue Apr 5 16:57:59 UTC 2022
   RealMemory=192777 AllocMem=0 FreeMem=176763 Sockets=2 Boards=1
   State=IDLE ThreadsPerCore=2 TmpDisk=0 Weight=1 Owner=N/A MCS_label=N/A
   Partitions=JobTesting
   BootTime=2023-09-21T07:56:52 SlurmdStartTime=2023-09-21T07:57:17
   CfgTRES=cpu=64,mem=192777M,billing=64,gres/gpu=4
   AllocTRES=
   CapWatts=n/a
   CurrentWatts=0 AveWatts=0
   ExtSensorsJoules=n/s ExtSensorsWatts=0 ExtSensorsTemp=n/s
```

See the information of an upcoming maintenance.

```sh
scontrol show reservation
```

```
ReservationName=maintenance2 StartTime=2024-01-19T14:30:00 EndTime=2024-01-21T14:30:00 Duration=2-00:00:00
   Nodes=cn[104-136] NodeCnt=33 CoreCnt=1056 Features=(null) PartitionName=(null) Flags=MAINT,IGNORE_JOBS,SPEC_NODES,ALL_NODES
   TRES=cpu=2112
   Users=root Accounts=(null) Licenses=(null) State=INACTIVE BurstBuffer=(null) Watts=n/a

ReservationName=maintenance3 StartTime=2024-02-03T14:30:00 EndTime=2024-02-05T14:30:00 Duration=2-00:00:00
   Nodes=cn[104-136] NodeCnt=33 CoreCnt=1056 Features=(null) PartitionName=(null) Flags=MAINT,IGNORE_JOBS,SPEC_NODES,ALL_NODES
   TRES=cpu=2112
   Users=root Accounts=(null) Licenses=(null) State=INACTIVE BurstBuffer=(null) Watts=n/a
```

### Node's Specs

Get node's specs.

```sh
sinfo --Node --responding --long
sinfo -N -r -l
```

| NODELIST | NODES | PARTITION    | STATE   | CPUS | S:C:T  | MEMORY | TMP_DISK | WEIGHT | AVAIL_FE | REASON |
| -------- | ----- | ------------ | ------- | ---- | ------ | ------ | -------- | ------ | -------- | ------ |
| cn101    | 1     | TrixieMain\* | drained | 64   | 2:16:2 | 192777 | 0        | 1      | (null)   | update |
| cn102    | 1     | TrixieMain\* | mixed   | 64   | 2:16:2 | 192777 | 0        | 1      | (null)   | none   |
| cn110    | 1     | TrixieMain\* | mixed   | 64   | 2:16:2 | 192777 | 0        | 1      | (null)   | none   |
| cn118    | 1     | TrixieMain\* | idle    | 64   | 2:16:2 | 192777 | 0        | 1      | (null)   | none   |
| cn119    | 1     | TrixieMain\* | idle    | 64   | 2:16:2 | 192777 | 0        | 1      | (null)   | none   |
| cn125    | 1     | TrixieMain\* | idle    | 64   | 2:16:2 | 192777 | 0        | 1      | (null)   | none   |

### Cluster Usage

Find the cluster usage per user.

```sh
sreport user top start=2020-06-01 end=2020-06-30 -t percent
```

```
--------------------------------------------------------------------------------
Top 10 Users 2020-06-01T00:00:00 - 2020-06-29T23:59:59 (2505600 secs)
Usage reported in Percentage of Total
--------------------------------------------------------------------------------
  Cluster     Login     Proper Name         Account      Used   Energy
--------- --------- --------------- --------------- --------- --------
   trixie  stewartd         Stewart            ai4d    24.23%    0.00%
   trixie   larkins          Larkin            ai4d    23.75%    0.00%
   trixie   ryczkok          Ryczko             sdt     3.07%    0.00%
   trixie      guoh             Guo    ai4d-core-06     0.16%    0.00%
   trixie       loc              Lo            ai4d     0.13%    0.00%
   trixie   valdesj          Valdes              dt     0.00%    0.00%
   trixie       xip              Xi              dt     0.00%    0.00%
   trixie     asadh                        covid-02     0.00%    0.00%
   trixie     paulp            Paul            ai4d     0.00%    0.00%
   trixie    ebadia           Ebadi              dt     0.00%    0.00%
```

### Allocated Hostnames of a Multinode Job

Get a list of hostnames allocated to a multi node job.

```sh
scontrol show hostnames $SLURM_NODELIST
```

```
cn101
cn102
cn103
cn104
```

## GPSC5

### SSH to a Worker Node

This is an example command to connect to a worker node on GPSC5

```sh
srun --overlap --pty --jobid=<JOBID> bash -l
```

Example of connecting to a GPU running job on GPSC5.

```sh
srun --overlap --gres=gpu:1 --nodes=1 --ntasks=1 --mem-per-cpu=0 --pty --oversubscribe --jobid=<SLURM_JOBID> /bin/bash -l
```

```sh
srun \
  --overlap \
  --oversubscribe \
  --pty \
  --gres=gpu:0 \
  --nodes=1 \
  --ntasks=1 \
  --mem-per-cpu=0 \
  --jobid=<JOBID> \
  /bin/bash -l
```

## GPSCC

### Script Example

The following example uses multiple nodes.

```sbatch
#!/bin/bash

#SBATCH --job-name=train
#SBATCH --partition=gpu_a100
#SBATCH --account=nrc_ict__gpu_a100

#SBATCH --time=2880

##IMPORTANT: `#SBATCH --ntasks-per-node=2`  MUST match  `#SBATCH --gres=gpu:2`
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=2
#SBATCH --gres=gpu:2

#SBATCH --output=%x-%j.out
#SBATCH --error=%x-%j.err
#SBATCH --open-mode=append
#SBATCH --mail-user==tes001
#SBATCH --mail-type=NONE

#SBATCH --signal=B:15@30


ulimit -v unlimited

cd /home/tes001/DT/tes001/
source ./SETUP_PT2.source
cd /home/tes001/DT/tes001/LJSpeech-1.1/PT2

srun everyvoice train text-to-spec --devices 2 --nodes 1 config/everyvoice-text-to-spec.yaml
```

### Sleeper Job

Start a sleeper job

```sh
psub -N sleeper -Q nrc_ict -gpu 'sleep 3600'
```

### Connect to Node

```sh
srun \
  --overlap \
  --oversubscribe \
  --pty \
  --nodes=1 \
  --ntasks=1 \
  --mem-per-cpu=0 \
  --jobid=<JOBID> \
  /bin/bash -l
```

## Running pytorch on Multiple Nodes

### Accelerate/HuggingFace Specific

```sh
#!/bin/bash

# [SLURM/ACCELERATE](https://github.com/huggingface/accelerate/blob/main/examples/slurm/submit_multinode.sh)
# [MNIST](https://huggingface.co/blog/pytorch-ddp-accelerate-transformers)

#SBATCH --partition=TrixieMain
#SBATCH --account=dt-mtp

#SBATCH --time=12:00:00
#SBATCH --job-name=MNIST.distributed
#SBATCH --gres=gpu:4
#SBATCH --nodes=2
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=8

#SBATCH --output=%x-%j.out
#SBATCH --signal=B:USR1@30
#SBATCH --requeue

# Requeueing on Trixie
# [source](https://www.sherlock.stanford.edu/docs/user-guide/running-jobs/)
# [source](https://hpc-uit.readthedocs.io/en/latest/jobs/examples.html#how-to-recover-files-before-a-job-times-out)

function _requeue {
   echo "BASH - trapping signal 10 - requeueing $SLURM_JOBID"
   date
   # This would allow to generically requeue any job but since we are using XLM
   # which is slurm aware, XLM could save its model before requeueing.
   scontrol requeue "$SLURM_JOBID"
}

if [[ -n "$SLURM_JOBID" ]]; then
   trap _requeue USR1
fi


# Keep a copy of the code in case it changes between runs.
head -n 112312 "$0" my_huggingface_trainer.py

# Setup your working environment.
source setup_env.sh ""

# These are required to setup the distributed framework.
head_node_ip=$(scontrol show hostnames "$SLURM_JOB_NODELIST" | head -n 1)
head_node_port=29507  # You can choose your own port

# Dump the environment.
( set -o posix ; set )


srun accelerate launch \
  --num_processes=$((SLURM_NNODES * SLURM_GPUS_ON_NODE)) \
  --num_machines="$SLURM_NNODES" \
  --rdzv_backend=c10d \
  --main_process_ip="$head_node_ip" \
  --main_process_port="$head_node_port" \
  --machine_rank="$SLURM_NODEID" \
  my_huggingface_trainer.py
```

## Submitting on Multiple Cluster

On GPSC, we can now submit up to 1000 6 hour-job with `#SBATCH --qos=low`.

```sh
sbatch \
  --time=00:10:00 \
  --clusters=gpsc5,gpsc6,gpsc7,gpsc8 \
  --account=${NRC_RC} \
  --partition=standard \
  myjob.sh
```

## Template.slurm

```sh
#!/bin/bash
# vim:syntax=bash:

# sbatch WMT2025_eval.slurm

#SBATCH --job-name=WMT25.eval
#SBATCH --comment="Official WMT2025 LRSL eval script"

# Trixie
#SBATCH --partition=TrixieMain,JobTesting
#SBATCH --account=dt-mtp
# On GPSC7
##SBATCH --partition=gpu_a100
##SBATCH --account=nrc_ict__gpu_a100
# On GPSC5
##SBATCH --partition=gpu_v100
##SBATCH --account=nrc_ict__gpu_v100
# On GPSC-C
##SBATCH --partition=gpu_a100
##SBATCH --account=nrc_ict__gpu_a100
##SBATCH --comment="image=nrc/nrc_all_default_ubuntu-22.04-amd64_latest"
##SBATCH --comment="image=registry.maze-c.collab.science.gc.ca/sschpcs/generic-job:ubuntu22.04_master"

#SBATCH --gres=gpu:1
#SBATCH --time=12:00:00
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=32
#SBATCH --mem=6G

#SBATCH --open-mode=append
#SBATCH --output=%x-%j.out

#SBATCH --requeue
#SBATCH --signal=B:USR1@30

# Fix SLURM environment variables.
SLURM_JOB_CPUS_PER_NODE=${SLURM_JOB_CPUS_PER_NODE%%(*)}   # '24(x2)' => '24'
SLURM_JOB_CPUS_PER_NODE_PACK_GROUP_0=${SLURM_JOB_CPUS_PER_NODE_PACK_GROUP_0%%(*)}   # '24(x2)' => '24'
SLURM_STEP_TASKS_PER_NODE=${SLURM_STEP_TASKS_PER_NODE%%(*)}   # '4(x2)' => '4'
SLURM_TASKS_PER_NODE=${SLURM_TASKS_PER_NODE%%(*)}   # '4(x2)' => '4'

# NOTE: We set OMP_NUM_THREADS or else we get the following Warning:
# WARNING:torch.distributed.run:
# *****************************************
# Setting OMP_NUM_THREADS environment variable for each process to be 1 in
# default, to avoid your system being overloaded, please further tune the
# variable for optimal performance in your application as needed.
# *****************************************
export OMP_NUM_THREADS=${SLURM_CPUS_PER_TASK:-$(nproc)}

# Requeueing on Trixie
# [source](https://www.sherlock.stanford.edu/docs/user-guide/running-jobs/)
# [source](https://hpc-uit.readthedocs.io/en/latest/jobs/examples.html#how-to-recover-files-before-a-job-times-out)
function _requeue {
   echo "BASH - trapping signal 10 - requeueing $SLURM_JOBID"
   date
   # This would allow to generically requeue any job but since we are using XLM
   # which is slurm aware, XLM could save its model before requeueing.
   scontrol requeue "$SLURM_JOBID"
}

if [[ -n "$SLURM_JOBID" ]]; then
  declare -a FORMAT_STRING=(
    JobID
    Submit
    Start
    End
    Elapsed
    ExitCode
    State
    CPUTime
    MaxRSS
    MaxVMSize
    MaxDiskRead
    MaxDiskWrite
    AllocCPUs
    AllocTRES%-50
    NodeList
    JobName%-30
    Comment%-80
  )
  SACCT_FORMAT=$(IFS=","; echo "${FORMAT_STRING[*]}")
  export SACCT_FORMAT
  trap "sacct --jobs $SLURM_JOBID --format=$SACCT_FORMAT" 0
  trap _requeue USR1
  unset FORMAT_STRING
fi

head -n 123123 "$0" >&2


# Output debugging information
function debug_info {
  echo "DEBUGGING INFO" >&2

  {
    hostname
    uname --all
    cat /etc/issue
    pwd
    which python
    #env
    # [How to list variables declared in script in bash?](https://stackoverflow.com/a/1305273)
    # In section SHELL BUILTIN COMMANDS (in the set section) it says: "In
    # posix mode, only shell variables are listed."
    ( set -o posix; set; )
    tr ":" "\n" <<< "$LD_LIBRARY_PATH"
    # What conda environment are we in?
    # Record all package versions.
    uv pip list
    uv pip freeze
    conda env export
  } >&2

  {
    # What GPU id is assigned to this job.
    nvidia-smi --list-gpus
    nvidia-smi
  } | sed 's/^/   /' >&2

  echo "DEBUGGING INFO END"
  echo;echo;echo
}

# GPSC
# [[ $JOBCTL_SLURM_CELLS =~ gpsc[^3] ]] && export https_proxy=http://webproxy.science.gc.ca:8888
# [[ $JOBCTL_SLURM_CELLS =~ gpsc[^3] ]] && export http_proxy=http://webproxy.science.gc.ca:8888
# GPSC-C
# [[ $JOBCTL_SLURM_CELLS =~ gpscc3 ]] && export https_proxy=http://webproxy.collab.science.gc.ca:8888
# [[ $JOBCTL_SLURM_CELLS =~ gpscc3 ]] && export http_proxy=http://webproxy.collab.science.gc.ca:8888
export TQDM_MININTERVAL=90
head_node_ip=$(scontrol show hostnames "$SLURM_JOB_NODELIST" | head -n 1)
readonly head_node_ip
readonly head_node_port=$(( SLURM_JOBID % (50000 - 30000 + 1 ) + 30000 ))


source wmt25-lrsl-evaluation/venv/bin/activate ""

# Call this after you have setup your environemnt and all your local variables
debug_info

command time --portability lm_eval \
  --model hf \
  --model_args pretrained=unsloth/Qwen2.5-3B-Instruct-unsloth-bnb-4bit \
  --tasks sorbian \
  --device cuda:0 \
  --batch_size 8 \
  --output_path baseline_output_sorbian \
  --log_samples
```
