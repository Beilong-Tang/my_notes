---
title: Pueue operation
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
```


# Pueue guide: running ML jobs on rodan

[pueue](https://github.com/Nukesor/pueue) (v4.0.4 here, installed in `~/.cargo/bin`) is a personal job
queue. It plays the role Slurm plays on a cluster, but only for your own jobs on one machine.

## 1. How it works

- **Daemon and client.** `pueued` is a background service that runs as your user. `pueue` is just a
  remote control for it. Tasks run as children of the daemon, not of your shell, so closing the
  terminal, VS Code or SSH does **not** stop them. This replaces `nohup`, `setsid` and `tmux` for jobs.
- **One queue per user per machine.** Every terminal you open on `rodan` talks to the same daemon and
  sees the same tasks. A different machine (for example the NCShare login node) has its own,
  separate daemon and queue. If a terminal shows "nothing", check `hostname`.

### Starting the daemon

```bash
pueued -d          # note: pueued (with a d), not "pueue -d"
```

Add this to `~/.bashrc` so the daemon is started on login if it is not already running:

```bash
pgrep -u "$USER" -x pueued >/dev/null || pueued -d
```

### What survives what

| Event | Effect |
|---|---|
| Close terminal / disconnect SSH | Nothing. Tasks keep running (checked on rodan: logind does not kill user processes on logout). |
| Server reboot | Daemon and running tasks die. Run `pueued -d` again; resume with `pueue restart --in-place <id>` (continues from the checkpoint if your script saves them). |
| A task is killed (out of memory, admin) | Only that task fails; the rest of the queue continues. |

## 2. Rules for adding tasks

1. **The task saves its context when you add it.** `pueue add` records:
   - the **current directory**: the task runs there, so `cd` first;
   - your **whole environment**: `CUDA_VISIBLE_DEVICES`, an activated conda env, or any variable
     set on the `pueue add` line is what the task sees.
     ```bash
     CUDA_VISIBLE_DEVICES=0 pueue add -- python train.py
     ```
2. **Commands run through `sh -c`.** Everything after `--` is joined into one string and run by
   the shell. If you use shell syntax (loops, pipes, `&&`, `>`), pass it as **one quoted string**:
   ```bash
   pueue add -- 'python train.py --run a && python eval.py --run a'
   ```
   A nested `sh -c '...'` loses its quotes and breaks (`Syntax error: "do" unexpected`).
   The `--` also stops pueue from reading your script's flags (`--lr`, ...) as its own.
3. **The job must stay in the foreground.** pueue uses the exit code to decide the result:
   0 is `Success`, anything else is `Failed (N)`. A script that backgrounds itself (for example
   `setsid nohup torchrun ... &`, as in `run_cifar10_polar.sh`) exits at once, so pueue marks it
   Success while training keeps running outside pueue. pueue then starts the next task on GPUs that
   are still busy, and `pueue kill` cannot stop the training. Use `exec torchrun ...` instead.
4. **Flush output** (`print(..., flush=True)` or `python -u`) so `pueue follow` shows it live.
   Output is stored in `~/.local/share/pueue/task_logs/<id>.log`.
5. **pueue does not assign GPUs.** You choose them with `CUDA_VISIBLE_DEVICES` and limit how many
   jobs run at once with groups (below).

## 3. Task states

```
Queued  ->  Running  ->  Success | Failed (N) | Killed
Stashed (held back, will not start until `pueue enqueue`)
Paused  (frozen; GPU memory stays allocated)
Dependency failed (an --after task failed, so this one never runs)
```

## 4. Groups

A group is a separate queue with its own limit on how many tasks run at once (like a Slurm
partition). Every task belongs to exactly one group; without `-g` it goes to `default`.

### Create and configure

```bash
pueue group add gpu                 # create group "gpu"
pueue parallel 4 -g gpu             # at most 4 tasks of "gpu" run at the same time
pueue add -g gpu -l exp1 -- python train.py ...
```

One group per GPU with `parallel 1` gives you one job per GPU and no manual coordination:

```bash
for i in 0 1 2 3; do pueue group add gpu$i; pueue parallel 1 -g gpu$i; done
CUDA_VISIBLE_DEVICES=2 pueue add -g gpu2 -l exp7 -- python train.py ...
```

### See groups

```bash
pueue group                         # list all groups, their parallel limit, running/paused
pueue status                        # all tasks, one table per group
pueue status -g toy                 # only the tasks of group "toy"
pueue group --json                  # groups as JSON (for scripts)
pueue status --json                 # everything as JSON (note: includes each task's full environment)
```

### Pause / resume a whole group

```bash
pueue pause -g toy                  # pause the group: running tasks are frozen, queued ones wait
pueue start -g toy                  # resume it
pueue pause -g toy --wait           # let running tasks finish, but start no new ones
```

### Remove a group

A group can only be removed when it has **no tasks at all** (finished ones included). Otherwise
pueue says `You cannot remove a group, if there're still tasks in it.`

```bash
pueue status -g toy                 # see what is left
pueue kill -g toy                   # stop running tasks in it (if any)
pueue clean -g toy                  # remove finished tasks (Success, Failed, Killed, ...)
pueue remove <ids>                  # remove queued / stashed / remaining tasks by id
pueue group remove toy              # now the group can be removed
```

`pueue clean` only removes **finished** tasks; queued, stashed or paused ones need `pueue remove`.

## 5. Everyday commands

| Command | What it does |
|---|---|
| `pueue add -g G -l NAME -- cmd` | Queue `cmd` in group G with a label |
| `pueue add --print-task-id ...` | Print only the new task's id (for scripts) |
| `pueue add --after 3 4 -- cmd` | Start only after tasks 3 and 4 **succeeded**; otherwise `Dependency failed` |
| `pueue add -s -- cmd` | Add as stashed (does not start until `pueue enqueue <id>`) |
| `pueue status [-g G]` | Show tasks |
| `pueue follow <id>` | Live output (Ctrl+C stops watching, not the task) |
| `pueue log <id>` / `pueue log --lines 50 <id>` | Output of a task |
| `pueue wait [-g G]` | Block until the tasks/group are finished |
| `pueue kill <id>` | Stop a running task (`Killed`) |
| `pueue restart --in-place <id>` | Rerun a finished task with the same id |
| `pueue pause <id>` / `pueue start <id>` | Freeze / unfreeze one task |
| `pueue stash <id>` / `pueue enqueue <id>` | Hold back / release a queued task |
| `pueue edit <id>` | Edit the command of a queued/stashed task |
| `pueue switch <id1> <id2>` | Swap the queue position of two queued tasks |
| `pueue remove <id>` | Remove a task from the list |
| `pueue clean [-g G]` | Remove all finished tasks |

**IDs are reused.** After `remove` / `clean`, new tasks get the lowest free ids again, so name
experiments with labels and run names, not task ids. `clean` also deletes the task's log, so copy
anything you want to keep first.

## 6. The toy example in this directory

- `train.py`: standard-library-only "training" (fits y = 3x + 2). Prints progress, saves
  `runs/<run>/checkpoint.json` every epoch and resumes from it, `--fail-at N` crashes with exit code 3
  at epoch N.
- `submit.sh`: creates group `toy` (2 in parallel) and queues:

  | Label | What it shows |
  |---|---|
  | `lr0.05`, `lr0.2` | Two runs side by side, each with its own `CUDA_VISIBLE_DEVICES` |
  | `crash` | A failing run, `Failed (3)` (only if `--epochs` > 5, see note) |
  | `eval` | `--after` both lr runs; a shell loop passed as one quoted string |
  | `after-crash` | Depends on `crash`, so it ends up `Dependency failed` |

```bash
cd /home/btang5/work/software/slurm_visualization/examples/pueue_toy
./submit.sh
pueue status -g toy
pueue follow <id>
pueue log <id>

# clean up
pueue clean -g toy && pueue group remove toy && rm -rf runs
```

Note: `--epochs` defaults to 2 in `train.py`, so the `crash` run (`--fail-at 5`) finishes before
epoch 5 and succeeds. To see a failure, use `--fail-at 1` or pass `--epochs 20`.

## 7. Using it with run_cifar10_polar.sh

Run `torchrun` in the foreground instead of with `setsid nohup ... &` (for example a `DETACH=0`
switch that does `exec "${CMD[@]}" >> "$LOG" 2>&1`), then:

```bash
cd /home/btang5/work/2026/polar_diffusion/ddpm-torch-polar-sb
pueue group add rodan4 && pueue parallel 1 -g rodan4
DETACH=0 GPUS=4,5,6,7 pueue add -g rodan4 -l cifar10_polar -- ./run_cifar10_polar.sh
DETACH=0 RUN=cifar10_gauss CONFIG=configs/cifar10.json RESUME=0 \
  pueue add -g rodan4 -l cifar10_gauss --after <id> -- ./run_cifar10_polar.sh
```

Training output goes to `exps/<run>/train.log` (not to `pueue log`); watch it with `tail -f`.
`RESUME=1` (the default) means `pueue restart --in-place <id>` continues from the latest checkpoint.
Check `nvidia-smi` before queuing: the script assumes GPUs 0-3 are used by `sbgm_sphere`.

