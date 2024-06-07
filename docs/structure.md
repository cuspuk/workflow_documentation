# Workflow structure

The recommended file structure:

```bash
├── config
│   ├── config.yaml
│   └── pep/
│       ├── config.yaml
│       └── samples.csv
├── results/
└── workflow/
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
    When creating a new workflow repository, it is recommended to use
    [github template](https://github.com/cuspuk/workflow_template). Follow the
    README of the template.

There are 3 main directories:

- `results` - here should be created all workflow-specific outputs.
- `config` - here should be defined the configuration for workflow-specific
  parameters and inputs.
- `workflow` - here should be defined everything related to the workflow logic.

## Config directory

Workflow should be defined to be configurable, i.e. the user should be provided
with the possibility to fine-tune the tools in the workflow. Snakemake
recommends using a file in YAML format. This file should also offer possibility
to provide external inputs, such as reference genome or various biological
databases. At last, there should be also configuration for resources, such as
number of threads or size of memory to be used.

!!! note
    Snakemake expects the configuration in `config/config.yaml`, so it will be
    loaded automatically. However, when running snakemake, you can modify this by
    using `--configfile {custom_path}`.

In this directory there should be configuration of inputs for the workflow.
Snakemake recommends using CSV file to allow user to provide inputs with its
attributes. Configuration of attributes should be defined in YAML format.

We advise to expose as much functionality as possible to the user. This
decreases the chance that you, as a developer, will need to make potential
future changes to the code, and to allow you to focus on adding new features.

## Workflow directory

The entrypoint for the Snakemake is the `Snakefile` file, which defines exactly
one rule which is by default the **target rule**. This rule should be named
`all` by convention and should request the final outputs of the workflow as
inputs. Other Snakemake rules or Python code should not be defined here, but
only included.

!!! note
    It is expected of the entrypoint to be named as `Snakefile` as snakemake expects
    the entrypoint in `workflow/Snakefile`.

### Rules subdirectory

Other workflow logic should be put in the subdirectory `rules/`, Python code for
reading configuration and other auxiliary functions should be by convention put
into `common.smk`, whereas pure Snakemake rules should be put into other `*.smk`
files, balancing cohesion and readability of individual `.smk` files.

!!! warning
    In `Snakefile` you should first include `common.smk`, then include other rules
    in `.smk` files, and at last, directly define the `all` rule.

Rules should be written to represent high-level abstraction view of mappings
between inputs and outputs, thus, only the simplest processing logic should be
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

### Environments and reports

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

### Schemas

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

## Writing rules

The specifics of writing the rules are available in the
[documentation](https://snakemake.readthedocs.io/en/v7.25.0/snakefiles/rules.html).
