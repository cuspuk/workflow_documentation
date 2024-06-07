# Running

In this section, we talk about how to run your snakemake workflows. This section should be partially reused when writing README for the users about how to run the workflow.

## Conda environments

One of advantages of Snakemake is, that it installs and manages conda environments for you. The same environment is then reused in multiple
rules, if they are defined to, and so on. Snakemake defaultly however is not specified to use conda, so you must specify the `--use-conda` flag in
the Snakemake call.

Generally, Snakemake install conda environments directly in the working directory, but this can lead to
many duplicate environments across projects, so it is a good idea to use
`--conda-prefix {DIR}` argument in the Snakemake call, and define `{DIR}` a path to the desired directory.

!!! warning
    There is some possibility of conflicts in case of running a workflow multiple times simultaneously, as each run could be trying to create the same conda environment. This is a rare case, but can happen. Ideally, you should either be testing the workflow for the first time alone, and then creating multiple parallel runs. Then, as everything was already installed by the first run, there would be no conflicts.

More information on the use of conda envs is available
[here](https://snakemake.readthedocs.io/en/v7.25.0/snakefiles/deployment.html#integrated-package-management).

## Useful arguments

There is a [myriad of arguments](https://snakemake.readthedocs.io/en/stable/executing/cli.html) to use when running a snakemake workflow.
Recommended arguments to use are the aforementioned `--use-conda` to use conda
environments, and the `--conda-prefix {DIR}` to specify the directory where snakemake
will install workflow conda environments.

First, it is advised to dry-run the snakemake workflow using `--dry-run`:

```bash
snakemake --cores {THREADS} --use-conda --conda-prefix {DIR} --rerun-incomplete --printshellcmds --dry-run
```

When the workflow appears to be assembled correctly, user can omit the
`--dry-run` argument to start the execution:

```bash
snakemake --cores {THREADS} --use-conda --conda-prefix {DIR} --rerun-incomplete --printshellcmds
```

For debugging purposes, the user can add `--notemp` flag to ignore temp() files
in snakemake rules, `--show-failed-logs` to automatically show failed log
messages.

## Running using SLURM

!!! note
    Workflows are made to be run using some job manager like SLURM, however you should run the workflow only locally when in the development stage, or when you are testing the workflow. You should distinguish between basic tests where you use mock data and pre-production tests where you use real data. Pre-production tests should be run using SLURM.

First, check [the official documentation](https://snakemake.readthedocs.io/en/v7.25.0/executing/cluster.html).

There are some changes. First, when running the workflow using cluster, jobs are usually run independently. Instead of cores, you provide the number of independent jobs in the `--jobs {JOBS} argument. Also, jobs are independently logged, so you need to prepare a directory for cluster logs, for example we can
use`./sbatch_logs`. You will need create the directory manually as slurm does not
create it (or tell the slurm to force create it).

Secondly, Snakemake will need to communicate with SLURM about job statuses, there is already a prepared script at the [workflow manager repository](https://github.com/cuspuk/workflow_manager/blob/main/extra/status-sacct.sh). The script needs to have set executive permissions.

Thirdly, you must provide snakemake how to cancel the job, in case of SLURM it is `scancel` command.

At last, you must define the way how to submit jobs in the `--cluster` argument. The basic way is:

```bash
sbatch --partition={STR} --cpus-per-task={INT} --parsable --output=`pwd`/sbatch_logs/%j.log --error=`pwd`/sbatch_logs/%j.err
```

Here `%j` is used to create a log for each cluster job named the same as the job itself. See [sbatch documentation](https://slurm.schedmd.com/sbatch.html) for more. Also, you can add other calls before the sbatch call, for example, adding log folder creation before sbatch:

```bash
mkdir -p `pwd`/sbatch_logs && sbatch {SBATCH_CALL}
```

!!! warning
    Parsable flag in sbatch call to SLURM is very important. Without it, snakemake cannot parse job statuses on their own and control the workflow execution in cases where SLURM hangs or fails.

To put all together, the final call should be like this:

```bash
snakemake --jobs {INT} --use-conda --conda-prefix {DIR} --cluster "mkdir -p `pwd`/sbatch_logs && sbatch --partition={STR} --cpus-per-task={INT} --parsable --output=`pwd`/sbatch_logs/%j.log --error=`pwd`/sbatch_logs/%j.err" --cluster-status {STATUS_SCRIPT_PATH} --cluster-cancel scancel {OTHER_SNAKEMAKE_ARGUMENTS}
```
