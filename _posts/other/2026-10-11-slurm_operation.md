---
title: Slurm operation
tag: Slurm
---

```sh
# slurm submit another job after a currently running job is done
# 
# <job_id>: the id of the currently running job 
sbatch --dependency=afterany:<job_id> scripts/train.sh
```



```sh
#!/bin/bash This is an example of running a gpu monitor for the job
mkdir -p gpu_usage
gpumon 30 > >(tee "gpu_usage/${SLURM_JOB_NAME}_${SLURM_JOB_ID}.log") &
NVSMI_PID=$!
trap 'kill $NVSMI_PID 2>/dev/null' EXIT`
