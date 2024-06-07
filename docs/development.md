# Development

In this section, we talk about the development process. It is advised to start
by using the [github template](https://github.com/cuspuk/workflow_template) and
following the README there.

## Automatic linting and formatting

It is a good practice to run linting and formatting. We can utilize `pre-commit`
to run it automatically.

First, you need to install `pre_commit` in your development environment.

Secondly, you need to create pre_commit config named `.pre-commit-config.yaml`,
[see example](https://github.com/cuspuk/workflow_amrWatch/blob/main/.pre-commit-config.yaml)

Then at last set up pre-commit using:

`pre-commit install`

Now before any commit, a defined set of actions will be performed and
information will be printed out about the results. Sometimes, changes will
automatically fixed, or you will be prompted to fix them on your own.

## Testing

There should always be a possibility to quickly test the workflow. Tests should
be bundled together with the workflow repository, in the `.tests` directory.

In the test directory there should be some mock data and test config or multiple
test configs.

Further, this should be enforced on the repository level. If you created your
workflow repository from the template, there is already github action for
testing, presuming the existence of `.tests` directory with valid test config
and data.

## Commit messages

When committing, you must follow the
[Conventional Commits spec](https://www.conventionalcommits.org/en/v1.0.0/).
Each PR is automatically validated by the GH action.

Further, any push (i.e. after merged PR) to the main branch results in an
another PR:

- a new release following [the Semantic Versioning](https://semver.org/)
- an automatic changelog as parsed from the commit history

!!! tip
    Try to write commit messages more in detail so the automatically created
    changelog could be useful, for example when writing to the users about changes.

!!! tip
    Try to not bundle all work in one massive commit.
