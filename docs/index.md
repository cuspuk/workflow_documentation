# Welcome to documentation

Documentation is build using `mkdocs` and written in `markdown`. Refer to [mkdocs documentation](https://www.mkdocs.org) and [markdown documentation](https://www.markdownguide.org/).

## Installation

!!! warning
    Python module `mkdocs` is required for building the documentation. We use the virtual environment, which is started by running:
    `pipenv shell`

When running for the first time or when a new module was added, run the following command to install all requirements in the virtual environment:
`pipenv sync`

## Writing documentation

To start writing documentation, please follow these steps:

* Create a markdown file (ending with `.md`) where you can start writing your documentation using `markdown`.
* Adhere to the [project documentation structure](#project-layout) and if creating new substructure, please add it also in the project layout (i.e. edit this file).
* To add a reference from the navigation panel of the documentation, add your `.md` file into the `nav` element of the configuration file `mkdocs.yml`.
* Use `markdown` syntax as most as possible to exploit the power of `mkdocs`, for example use `#` to automatically build the table of contents on the right.

## Previewing documentation

When in the virtual environment, run `mkdocs serve` and head to the <http://127.0.0.1:8005/> where the current version of the documentation can be seen. This uses also auto-reloading, so doing and saving changes in the documentation also automatically updates the served documentation.

## Building documentation

After doing all the changes you wanted to do, the documentation can be built into static .html pages using the virtual environment and the command `mkdocs build`. In `site\` static .html pages will be built.

## Project layout

Documentation is structured as follows:

TBA

## Tips

### Extensions

For easier writing, install IDEs extensions for markdown. For Visual Studio code:

* [markdown all in one](https://marketplace.visualstudio.com/items?itemName=yzhang.markdown-all-in-one) - offer many useful shortcuts such as ctrl-b to get bold, easy referencing and so on.
* [Markdown preview enhanced](https://marketplace.visualstudio.com/items?itemName=shd101wyy.markdown-preview-enhanced) - to get preview of markdown document in VScode (using ctrl-k followed by v)
* [markdownlint](https://marketplace.visualstudio.com/items?itemName=DavidAnson.vscode-markdownlint) - markdown linter.
* [file-tree-generator](https://marketplace.visualstudio.com/items?itemName=Shinotatwu-DS.file-tree-generator) - for generating file trees

### Tables

To create tables in markdown painlessly use table generators, such as [this one](https://www.tablesgenerator.com/markdown_tables).

### Future work

* Versioning using [mike](https://squidfunk.github.io/mkdocs-material/setup/setting-up-versioning/)
* Check [extensions](https://facelessuser.github.io/pymdown-extensions/extensions/details/)
