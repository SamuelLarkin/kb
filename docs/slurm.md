# SLURM

## Update a job

Add a dependency to run a job after another job.

```sh
scontrol update jobid=540913 dependency=afterok:540912
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
scontrol update jobid=<job_id> TimeLimit=<new_timelimit>
```

## Misc

Find out where a job will run
```sh
scontrol show job JOBID
```

```sh
sreport user top start=2020-06-01 end=2020-06-30 -t percent
```

### Stats of a job
Get some stats about a job that ran on `Slurm`.
Get stats about a job that has finished.
```sh
sacct --long --jobs=JOBID
sacct -l -j JOBID
```

See what the nodes really offer.
```sh
scontrol show nodes
```

See the information of an upcoming maintenance.
```sh
scontrol show reservation
```

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

### Node's Specs
Get node's specs.
```sh
sinfo --Node --responding --long
sinfo -N -r -l
```
| NODELIST | NODES | PARTITION   | STATE      | CPUS   | S:C:T  | MEMORY | TMP_DISK | WEIGHT | AVAIL_FE | REASON |
|----------|-------|-------------|------------|--------|--------|--------|----------|--------|----------|--------|
| cn101    | 1     | TrixieMain* | drained    | 64     | 2:16:2 | 192777 | 0        | 1      | (null)   | update |
| cn102    | 1     | TrixieMain* | mixed      | 64     | 2:16:2 | 192777 | 0        | 1      | (null)   | none   |
| cn110    | 1     | TrixieMain* | mixed      | 64     | 2:16:2 | 192777 | 0        | 1      | (null)   | none   |
| cn118    | 1     | TrixieMain* | idle       | 64     | 2:16:2 | 192777 | 0        | 1      | (null)   | none   |
| cn119    | 1     | TrixieMain* | idle       | 64     | 2:16:2 | 192777 | 0        | 1      | (null)   | none   |
| cn125    | 1     | TrixieMain* | idle       | 64     | 2:16:2 | 192777 | 0        | 1      | (null)   | none   |


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
srun --jobid=JOBID --pty bash -l
```

Example of connecting to a GPU running job on GPSC5.
```sh
srun --jobid=JOBID --gres=gpu:8 -N 1 --ntasks=1 --mem-per-cpu=0 --pty -w ib14gpu-001 -s /bin/bash -l
```
```sh
srun \
  --jobid JOBID \
  --gres=gpu:0 \
  --nodes 1 \
  --ntasks 1 \
  --mem-per-cpu=0 \
  --pty \
  --nodelist ib12gpu-001 \
  --oversubscribe \
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
psub -N sleeper -Q nrc_ict 'sleep 3600'
```
