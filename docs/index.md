# Snakemake workflow documentation

Documentation for Snakemake workflows development.

## Installation

Snakemake is a python package which needs to be installed prior running the
workflows. It can be installed either via `pip`, or in a designated `conda`
environment. The latter option is preferred, while it is also recommended to use
`mamba` instead of `conda`. `mamba` is practically same as `conda`, but the
dependency solver is much faster and it uses the `conda-forge` channel as the
base channel. For instructions on installing `mamba`, refer to the
[documentation](https://mamba.readthedocs.io/en/latest/installation/mamba-installation.html).

With running `mamba` (or `conda`), create a new environment with required
dependencies:

```sh
mamba create -c conda-forge -c bioconda --name snakemake_v7_25 python=3.11 snakemake=7.25 peppy snakemake-wrapper-utils
```

!!! note
    Snakemake version should be frozen to a specific version to prevent any
    dependency problems. Furthermore, this should be set for each workflow
    separately, so the development of one workflow is independent.

## Snakemake basics

Snakemake is a workflow automation tool, which uses rules to define
functionality. The comprehensive documentation is provided in the following
site:
[https://snakemake.readthedocs.io/en/v7.25.0/](https://snakemake.readthedocs.io/en/v7.25.0/).

!!! warning
    Try to refer only to the documentation of the Snakemake version you declare to
    use, as some features or their usage can change among versions.

## Workflow structure

The recommended file structure:

```bash
├── config
│   ├── config.yaml
│   └── pep/
│       ├── config.yaml
│       └── samples.csv
├── results/
│   └── workflow/
│   ├── scripts/
│   ├── envs/
│   │   └── {...}.yaml
│   ├── report/
│   │   └── {...}.rst
│   ├── rules/
│   │   ├── common.smk
│   │   └── {...}.smk
│   ├── schemas/
│   │   ├── config.schema.yaml
│   │   └── samples.schema.yaml
│   └── Snakefile
```

!!! note
    When creating a new workflow repository, tt is recommended to use
    [github template](https://github.com/cuspuk/workflow_template). Follow the
    README of the template.

There are 3 main directories:

- `results` - here should be created all workflow-specific outputs.
- `config` - here should be defined the configuration for workflow-specific
  parameters and inputs.
- `workflow` - here should be defined everything related to the workflow logic.

### `config/`

Workflow should be configurable, i.e. the user should be provided with the
possibility to fine-tune the tools in the workflow. Snakemake recommends using a
file in YAML format. This file should also offer possibility to provide external
inputs, such as reference genome or various biological databases. At last, there
should be also configuration for resources, such as number of threads or size of
memory to be used.

!!! note
    Snakemake expects the configuration in `config/config.yaml`, so it will be
    loaded automatically. However, when running snakemake, you can modify this by
    using `--configfile {custom_path}`.

In this directory there should be configuration of inputs for the workflow.
Snakemake recommends using CSV file to allow user to provide inputs with its
attributes. Configuration of attributes should be defined in YAML format.

### `workflow/`

The entrypoint for the Snakemake is the `Snakefile` file, which defines exactly
one rule which is by default the **target rule**. This rule should be named
`all` by convention and should request the final outputs of the workflow as
inputs. Other Snakemake rules or Python code should not be defined here, but
only included.

!!! note
    It is expected of the entrypoint to be named as `Snakefile` as snakemake expects
    the entrypoint in `workflow/Snakefile`.

Other workflow logic should be put in the subdirectory `rules/`, Python code for
reading configuration and other auxiliary functions should be by convention put
into `common.smk`, whereas pure Snakemake rules should be put into other `*.smk`
files, balancing cohesion and readability of individual `.smk` files.

!!! warning
    In `Snakefile` you should first include `common.smk`, then other rules in
    `.smk`, and at last define the `all` rule.

Rules should be written to represent high-level abstraction view of mappings
between inputs and outputs, thus only the simplest processing logic should be
defined directly in rule (i.e. readable bash one-liners like using `cat` for
multiple inputs). Processing logic should be abstracted away, either using
`wrapper` directive for external modular scripts, or `script` directive for
internal scripts. Internal scripts should be put into `scripts/` subdirectory.

!!! note
    When the script is called via Snakemake, a `snakemake` object is made available
    in the script, storing all input and output paths, parameters, threads, and

other arguments passed to the specific rule. See
[the documentation for specific languages.](https://snakemake.readthedocs.io/en/v7.25.0/snakefiles/rules.html#external-scripts)

!!! warning
    Snakemake files are not included automatically, but must be defined to be
    included.

Other stuff that should be defined in `workflow/` directory are conda
environments, report templates and validation schemas. Conda environments should
be put in `envs/` subdirectory, and they should be defined as specific as
possible, and they should not contain anything else in addition to the
dependencies required by the rule. Even in the case, where one rule has
dependencies that are subset of the other rule, environments should still be
defined separately to prevent any conflicts.

If you use `report` directive for outputs in Snakemake rules, you can provide
custom report templates. These templates should be stored in the `report/`
subdirectory.

Configuration for the workflow should be validated, so workflow should not fail
due to misconfiguration halfway execution. To validate configuration, it is
recommended to use validation schemas, that should be defined in `schemas/`
subdirectory. Currently, Snakemake allows JSON or YAML format conforming to the
[JSON schema specification](https://json-schema.org/specification). Input
configuration should be validated as well, e.g. to ensure that required
attributes of inputs have been provided.

!!! tip
    In validation schema, you can supply default values or advanced validation such
    as minimum values, enumeration and so on.

!!! note
    Stricter validation means earlier feedback to the user.

## Rules

It is a good idea to create one file for common operations (in the file
structure above, it is the `common.smk` file), such as loading paths, extracting
resource limits and threads from config, parsing extra arguments, etc. These
operations can be regular Python functions, but when the function is used in the
rule, Snakemake passes `wildcards` argument, so it should be present in the
function signature. The argument contains all wildcards the rule works with and
can be freely used in the function.

The specifics of writing the rules are available in the
[documentation](https://snakemake.readthedocs.io/en/v7.25.0/snakefiles/rules.html).

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
