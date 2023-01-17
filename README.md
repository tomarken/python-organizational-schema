# python-organizational-schema
A coherent set of guidelines for Python versioning, virtual environment management, and dependency management

## Motivation
When I first started working on Python projects I was overwhelmed by all of the different tools available for python installations, virtual environment management, and project dependency management. Part of the problem is the sheer number of tools available for each and the high degree of coupling between them. Their names are also very confusingly similar (e.g. `pyenv`, `pyenv-virtualenv`, `virtualenv', `venv`). I found it challenging to read about best practices for each of these topics in isolation without understanding if everything would be compatible in the end. Below I've laid out one pattern that has worked well for me across many different projects. I've tried to motivate my choices with comparisons to other tools. There are certainly other patterns that could be successful but I have not needed to stray from this one. 

## A few notes on virtual environments and project management
Each project you work on should have a set of dependencies (other packages) needed to run/develop the software. For example, if I have a project that uses `numpy` to do some array manipulation, I will, at a minimum, need to install `numpy` and all of its dependencies in a space where my Python binary has access. In fact, if you have multiple branches on a given project or you want to test different Python versions on the same project, you may have multiple distinct sets of dependencies. It would become messy very quickly if we simply insalled all of our projects' Python dependnecies into one giant folder. In Python you cannot install two different versions of the same package alongside one another. This would prevent projects with conflicting version reuirements from being installed. It would also be a nightmare to keep organized. A better system is to silo each distinct set of dependencies into a "virtual environment". You can think of each virtual environment as a tiny bubble with its own Python binary and singular set of dependencies. Note that virtual environments are "many to one" with respect to projects. One project may contain multiple virtual environments, however, no virtual environment should be used for more than one project.

A set of projects and virtual environments might look like this in an abstract sense:

* Project A
  * Virtual environment A1
  * Virtual environment A2
* Project B
  * Virtual environment B
* Project C
  * Virtual environment C1
  * Virtual environment C2
  * Virtual environment C3

## The tools:
* [`pyenv`](https://github.com/pyenv/pyenv): Used only for Python version installation and organization (**not** used for virtual environments)
* [`venv`](https://docs.python.org/3/library/venv.html): Built-in Python module that creates virtual environments 
* [`poetry`](https://python-poetry.org/): Python dependency management tool
* [`pyproject.toml`](https://pip.pypa.io/en/stable/reference/build-system/pyproject-toml/) and [`poetry.lock`](https://python-poetry.org/docs/libraries#lock-file) files are the "source of truth" for the project

### Why did I choose these tools?
* Why not use `conda`?
  * Instead of `pyenv` and `venv`, I could have used one of the `conda` distributions instead. My major issue with `conda` is the location of the project dependencies.  `conda` places dependency source files in a relatiely obscure location like `/opt/miniconda/[env]/lib/[python-ver]/site-packages`. I prefer for these files to be located within the project like `[repo]/.venv/[python-ver]/site-packages`. This makes locating and referencing dependency source code much easier and it makes it clear which virtual environments should go with which project.
  * Deleting virtual environments and Python versions is as simple as `rm -r .venv` or `rm -r ~/.pyenv/versions/[version]`. I don't need to remember the obscure `conda` directory, run any `conda` commands, or download `anaconda-clean`. 
  * This allows for much easier virtual environment management. Imaging making multiple virtual environments with `conda` for several repos and then revisiting a project after several months. Your list of `conda` environments may be quite long and good luck remembering which environment goes with which project. Perhaps your naming scheme from a few months ago doens't make much sense. Perhaps some dummy environments are mixed in. Perhaps your projects have similar names and you were not specific enough. With the built-in module `venv` and choosing to install virtual environments inside the project directory, it's immediately obvious which environments correspond to the project. 
* Why not use `pyenv`'s plug-in `pyenv-virtualenv` to both manage python installations as well as virtual environments?
  * For starters, since Python 3.6 `pyenv` is no longer the officially recommended solution for virtual environment management. `venv` has taken its place and is officially maintained by the Python development team. I always prefer to use officially recommended packages whenever possible.
  * `pyenv` places virtual environments in `~/.pyenv/versions/[version]/envs/[env-name]` which is an improvement on `conda` but still not alongside the actual project. You cannot change the default installation location, you can only create a symlink which is not ideal and potentially confusing.
* Why not use `pip` to install project dependencies instead of `poetry`?
  * The advantage of using `poetry` is that `poetry` will automatically determine the most up-to-date package versions that don't conflict with the user constraints specified in a `pyproject.toml` file. 
  * The generation of a `poetry.lock` file makes installation completely deterministic for other users. There is some burden placed on the developer to make sure `poetry` resolves its dependencies nicely. This can sometimes take a long time for certain corner cases, but generally this works quite well. `pip` installing everything is certainly easier but it may cause package version conflicts and installation by different users may not be identical. 

## Installation

### `pyenv`
* Install with `homebrew`: `brew update` and `brew install`. 
* Add the following to your approopriate shell `rc` file. For MacOS it is likely `~/.zshrc` if you use `Zsh` (default) or `~/.bashrc` if you use `bash`:
  * `export PYENV_ROOT="$HOME/.pyenv"`
  * `export PATH="$PYENV_ROOT/bin:$PATH"`
  * `eval "$(pyenv init -)"`
* Source your new `rc` file. With `Zsh`, run `source ~/.zshrc`. 
* Test your `pyenv` installation with: `pyenv --version`

### `venv`
`venv` is a built-in python module that comes automatically with any Python version >=3.3. For legacy Python versions, you can refer to the `virtualenv` package (not needed for the vast majority of modern projects), however, the next tool `poetry` requires Python 3.7+. 

### `poetry`
* Install with `curl -sSL https://install.python-poetry.org | python3 -`
* By default this is installed on MacOS at `~/.local/bin/` which is by default on the `PATH`. However, if this is not already on your `PATH` (confirm with `echo $PATH`), you can add the following to your appropriate `rc` file: `export PATH="$HOME/.local/bin:$PATH" or replace `$HOME/.local/bin` with your actual installation location.

## Putting everything into action
### Creating the `pyproject.toml` file
* Let's make a new project called `dummy` by creating a directory in `~/repos/dummy`. 
* We begin by generating a `pyproject.toml` file that will describe our project's dependencies. This file will almost certainly change as the project is built and new features are added. That is totally fine and the `pyproject.toml` file can even help us track the project version. 
  * Navigate the project directory `~/repos/dummy`. 
  * Create a file called `pyprojec.toml` and copy the following lines with appropriate modification:
```toml
[tool.poetry]
name = "dummy"
version = "0.1.0"
description = "A dummy test project"
authors = ["User Name <user-email@server.com>"]

[tool.poetry.dependencies]
python = ">=3.9,<3.11"
numpy = ">=1.23"
matplotlib = "^3.6"

[tool.poetry.dev-dependencies]
pytest = "^7.1"

[build-system]
requires = ["poetry-core>=1.0.0"]
build-backend = "poetry.core.masonry.api"
```
  * Let's recap: 
    * We've given our project a name and initial version number `0.1.0` as well as specified our name and email in the first section. 
    * We've specified that we want our project to support Python versions ranging from 3.9 to 3.10 (but not 3.11). 
    * We've specified a few project dependencies (`numpy` and `matplotlib`). Any dependency that a user would need in order to run your software should be here.
    * We've specified a few "dev-dependencies" which are any dependencies that a _developer_ would need in order to contribute to your repository. These are typically packages like `pytest` for unit testing but could also include formatters and linters like `black`, `flake8`, etc. 
    * The section at the bottom is reuired by `poetry`.
    * Package versions have been specified by `>=, <, ^`. Refer to this [guide](https://python-poetry.org/docs/dependency-specification#dependency-specification) in `poetry` for more information about version specifiers.
  *  We could have created this file automatically by running `poetry new dummy` frome the `~/repos` directory. However, this would have added a few extra directories and files by default and I think it's a good exercise to create this manually at least once. 

### Installing Python
* First run `pyenv versions` to see which versions are installed. This will also display the system version. Let's pick Python version 3.9.15 for our installation.
* Run `pyenv install 3.9.15`. 
  * `pyenv` may take a few minutes to install the CPython binaries in `~/.pyenv/versions/3.9.14`.
 * Run `pyenv versions` to verify that `pyenv` has installed 3.9.15. The astirisk shows which version is active. You probably have `system` active.

### Installing the virtual environment
* Create a `pyenv` shell: `pyenv shell 3.9.15`. Now, everything you run will be using Python 3.9.15. 
* Run `pyenv versions` to see the list again and make sure the astirisk indicates 3.9.15. Alternatively you can run `pyenv version`. 
* Run `python3 -m venv .venv --prompt "dummy"`.
  * This uses the active Python version (3.9.15) and the module `venv` to create a virtual environment.
  * It will be located inside a folder called `.venv` (inside the main project directory). 
  * It will display the prompt "dummy" whenever this virtual environment is active.
  * The general form of this command is `python3 -m venv [path/to/venv] --prompt [prompt]. The location and prompt can easily be modified to suit other needs. For example you could install two virtual environments alongside each other (perhaps `.venv-main` for the main branch of a project and `.venv-feature` for a feature branch).
  * Run `ls -a` to see that a directory called `.venv` was installed. This is the location of the virtual environment. This contains the Python binaries and the location where the package dependencies will be installed `.venv/lib/python3.9/site-packages`.
* Run `pyenv shell --unset` to deactivate the `pyenv` shell.
* In order to activate the virtual environment run `source .venv/bin/activate` which runs the `activate` script. You should now see the prompt `(dummy)`. To deactivate at any time run `deactivate`. You will need to run these commands anytime you need to enter or exit the virtual environment.
* For the next step, it's best to keep it activated. Technically it will still work either way since we used the name `.venv` but if we had chosen any other directory it would need to be activated.

### Creating the `poetry.lock` file
We need `poetry` figure out which versions of Python, `numpy`, `matplotlib`, and `pytest` are compatible with one another. `poetry` does this by checking all of the individual depenencies for `numpy`, `matplotlib`, and `pytest` (along with the Python versions) recursively and determining the most up-to-date versions of each required package that satisfy all of the constraints listed in the `pyproject.toml`. For complex projects with lots of dependencies and for support of many Python versions, this can sometimes take several minutes or even hours. However, usually this can be sped up by narrowing the allowed versions of Python and placing reasonable constraints on package versions. 

To create the lock file simply run `poetry lock`. This will likey version solve in a few seconds. If you'd like to see the details of what's goin on, you can run `poetry lock -vvv` instead to flag high verbosity. I almost always run with this option in order to see where `poetry` might be slow to version solve.

`poetry` automatically determined that there was an active virtual environment. If we had not activated, we still would have succeeded because `poetry` knows to look for the directory `.venv`. However, if we had not used this convention (calling it `.venv-main` perhaps), we would need to activate or else `poetry` would create a virtual environment for us and store it in one of its library directories which is not what we want.  

### Installing the repo
Run `poetry install`. You should see the list of packages get installed. 

### Committing the dependency files
In order to make sure that your project's dependencies and metadata are properly tracked, make sure to commit `pyproject.toml` and `poetry.lock` to your repo every time they are changed. When you make modifications to your project's main branch, you can increment the `version` field in the `pyproject.toml` field.
