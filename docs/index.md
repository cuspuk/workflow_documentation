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
mamba create -c conda-forge -c bioconda --name snakemake python=3.11 snakemake=7.25 peppy snakemake-wrapper-utils
```

!!! warning
    Although there are more recent Snakemake versions, they are not fully backwards
    compatible, so for now, we use the slightly older Snakemake 7.25

## Snakemake basics

Snakemake is a workflow automation tool, which uses rules to define
functionality. The comprehensive documentation is provided in the following
site:
[https://snakemake.readthedocs.io/en/v7.25.0/](https://snakemake.readthedocs.io/en/v7.25.0/).

## Workflow structure

The workflow file structure usually looks like this:

```bash
├── config
│   ├── config.yaml
│   └── pep/
│       ├── config.yaml
│       └── samples.csv
├── resources/
└── workflow/
├── scripts/
├── envs/
│   └── bwa.yaml
├── report/
│   └── template.rst
├── rules/
│   ├── common.smk
│   └── mapping.smk
├── schemas/
│   ├── config.schema.yaml
│   └── samples.schema.yaml
└── Snakefile
```

### `workflow/`

This folder contains the main functionality of the workflow. The entrypoint for
the Snakemake is the `Snakefile` file, which should either contain all rules
(not a good practice) or include the additional rule files, housed in the
`rules/` folder. Usually, the Snakefile contains `rule all` which specifies all
outputs the workflow should return.

#### `rules/`

Files with specific Snakemake rules go to this folder. The granularity is up to
the user, there can be individual file for each rule, they can be grouped
together according to some common characteristic, or all rules can be defined in
one file, but this is not recommended, especially in large workflows.

It is a good idea to create one file for common operations (in the file
structure above, it is the `common.smk` file), such as loading paths, extracting
resource limits and threads from config, parsing extra arguments, etc. These
operations can be regular Python functions, but when the function is used in the
rule, Snakemake passes `wildcards` argument, so it should be present in the
function signature. The argument contains all wildcards the rule works with and
can be freely used in the function.

The specifics of writing the rules are available in the
[documentation](https://snakemake.readthedocs.io/en/v7.25.0/snakefiles/rules.html).
It is, however, important to note that the rules are not automatically included
in the `Snakefile`, so they must be added manually using the `include()`
function.

#### `scripts/`

In the scripts folder are stored custom scripts used in the workflow. The
scripts can be of any programming language, but Snakemake has a native support
for Python, R, Rust, Julia and Bash scripts, and they can be called directly
from the rule by replacing `shell` or `wrapper` parameter with `script`.

When the script is called via Snakemake, a `snakemake` object is made available
in the script, storing all input and output paths, parameters, threads, and
other arguments passed to the specific rule. Each programming language has a
slightly different syntax on how to access the arguments, so the examples are
provided
[here](https://snakemake.readthedocs.io/en/v7.25.0/snakefiles/rules.html#external-scripts).

#### `envs/`

This folder is used to define conda environments used by the rules. In this file
structure above is one environment saved in _bwa.yml_, which probably contains
the dependencies for running BWA. The environments are created automatically by
conda and used in the required rule in the workflow.

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

### `config/`

Configuration files with user-defined parameters. The user can change these
parameters to fine-tune the tools in the workflow. It should also contain paths
to reference genome and various databases, so they are defined only once and
therefore it's very unlikely that the user will accidentally supply a path to
another reference or incorrect database.

`config.yaml` is the main configuration file with all custom parameters for
tools in the workflow. Besides parameters and database paths, user can also use
the config file for specifying how much memory and how many cores each rule can
use.

Note that the parameters are not loaded automatically, they need to be parsed
and supplied to the rules in the format the tool expects them to be. The good
place for parsing functions is in the `common.smk` file.

The pep folder contains the list of samples in the `samples.csv` file and the
configuration for each column in `pep/config.yaml`. Besides paths to input
files, the CSV file can contain additional columns (if they are specified in the
config) with additional data. This data can be then extracted from the `pep`
object during runtime.

### `schemas/`

The workflow may require many parameters to be specified, but majority of them
can be set to default value or completely omitted. It also often requires some
mandatory parameters, that need to be supplied by the user, or the tool won't
start. To ensure all required parameters are filled-in and all parameters in the
config file are in the correct format, a schema can be used.

The schema is used when the config is being loaded, to validate whether it
conforms to the schema. When required parameters are not supplied, the
validation fails, which prevents the workflow failing halfway execution.
Additionally, the parameters can have default values, so if it is not present in
the user-defined config, the default value is loaded from the schema.

The schema can be in a JSON or YAML format conforming to the JSON schema
specification. More information is available
[here](https://json-schema.org/specification).

## Running

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
