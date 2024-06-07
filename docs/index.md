# Snakemake workflow documentation

Documentation for Snakemake workflows development.

## Running

Generally, Snakemake creates envs in the working directory, but this can lead to
many duplicate environments across projects, so it is a good idea to use
`--conda-prefix $CONDA_REPOS_DIR` flag in the Snakemake call, in which the
`$CONDA_REPOS_DIR` variable is a path to the directory with environments created
by Snakemake.

If you want Snakemake to use the conda envs, you must specify `--use-conda` in
the Snakemake call. Note that the same environment can be used in multiple
rules, if they use the same tool.

More information on the use of conda envs is available
[here](https://snakemake.readthedocs.io/en/v7.25.0/snakefiles/deployment.html#integrated-package-management).

There are many arguments to use when running a snakemake workflow, see the
[documentation](https://snakemake.readthedocs.io/en/stable/executing/cli.html).
Recommended arguments to use are the aforementioned `--use-conda` to use conda
environments, and the `--conda-prefix` to specify the directory where snakemake
will install workflow conda environments.

First, it is advised to dry-run the snakemake workflow using `--dry-run`:

```bash
snakemake --cores {THREADS} --use-conda --conda-prefix $CONDA_REPOS_DIR --rerun-incomplete --printshellcmds --dry-run
```

When the workflow appears to be assembled correctly, user can omit the
`--dry-run` argument to start the execution:

```bash
snakemake --cores {THREADS} --use-conda --conda-prefix $CONDA_REPOS_DIR --rerun-incomplete --printshellcmds
```

For debugging purposes, the user can add `--notemp` flag to ignore temp() files
in snakemake rules, `--show-failed-logs` to automatically show failed log
messages.

### Running using SLURM

To run in cluster using SLURM, the command is slightly different. First, it is
advised to prepare a different directory for cluster logs, for example we can
use `./sbatch_logs`. You must create the directory manually as slurm does not
create it:

```bash
mkdir -p sbatch_logs
```

Also you will probably need to set executive permissions to the sbatch
monitoring script:

```bash
chmod +x sbatch_status.py
```

Then, the recommended command is:

```bash
snakemake --jobs 40 --rerun-incomplete --use-conda --printshellcmds --cluster "sbatch --partition=gen-compute --cpus-per-task=8 --parsable --output=`pwd`/sbatch_logs/%j.log --error=`pwd`/sbatch_logs/%j.err" --cluster-status `pwd`/sbatch_status.py --keep-going --cluster-cancel scancel --retries 3 --dry-run
```

- `%j` is used to create a log for each cluster job named the same as the job
  itself.

- `--cpus-per-task=8` is important to also specify the number of threads in the
  config.

- Snakemake process must be kept alive during the whole analysis as a master
  process. Sometimes a SLURM job can fail before sending the failure message to
  the master process and the job will hang forever. To prevent this, snakemake
  recommends the usage of `--cluster-status` argument, requiring to pass
  parsable argument to sbatch.

- Snakemake jobs can be expected to fail such as download jobs, but also they
  can fail due to memory constraints, i.e. when a job requests more memory than
  expected due to an unusually large .fastq.gz file, you should pass
  `--retries {n}` option to tell snakemake to retry the job. This is further
  used for example in picard deduplication where more memory is requested for
  the subsequent retries (up to the maximum memory specified in the config).

Do not forget to adjust the numbers accordingly for your cluster configuration,
and also to run it without the `--dry-run` option.

Alternatively, we can simplify the command by passing additional arguments,
`--profile` with path to profile specifying the submission command and other
additional parameters, and `--cluster-status` with path to script preventing the
job hang:

```bash
snakemake --profile /data/workflows/profiles/basic/ --cluster-status /data/workflows/extra/status-sacct.sh --conda-prefix $CONDA_REPOS_DIR --rerun-incomplete
```
