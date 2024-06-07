
# Introduction

Snakemake is a python package which needs to be installed prior running the
workflows. It can be installed either via `pip`, or in a designated `conda`
environment. The latter option is preferred, while it is also recommended to use
`mamba` instead of `conda`. `mamba` is practically same as `conda`, but the
dependency solver is much faster and it uses the `conda-forge` channel as the
base channel. For instructions on installing `mamba`, refer to the
[documentation](https://mamba.readthedocs.io/en/latest/installation/mamba-installation.html).

With running `mamba` (or `conda`), create a new environment with required
dependencies, for example:

```sh
mamba create -c conda-forge -c bioconda --name snakemake_v7_25 python=3.11 \
    snakemake=7.25 peppy snakemake-wrapper-utils pre-commit
```

!!! note
    Snakemake version should be frozen to a specific version to prevent any
    dependency problems. Furthermore, this should be set for each workflow
    separately, so the development of each workflow is independent from the
    others.

Also you should distinguish between environment for workflow development and for users running the workflow. For example, users usually use only lite version of the environment, without things formatting or linting stuff.

## Snakemake basics

Snakemake is a workflow automation tool, which uses rules to define
functionality. The comprehensive documentation is already provided
[on the official site](https://snakemake.readthedocs.io/en/v7.25.0/).

!!! warning
    Try to refer only to the documentation of the Snakemake version you declare to
    use, as some features or their usage can change among versions.

Now, that you prepared your conda environment, head to the [next section](structure.md)
to know more about the workflow code structure.
