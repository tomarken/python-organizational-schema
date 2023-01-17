# python-organizational-schema
A coherent set of guidelines for python versioning, virtual environment management, and dependency management

## Motivation
When I first started working on python projects I was overwhelmed by all of the different tools available for python installations, virtual environment management, and project dependency management. Part of the problem is the sheer number of tools available for each and the high degree of coupling between them. I found it challenging to read about best practices for any one of these topics in isolation. Below I've laid out one pattern that has worked well for me across many different projects. I've tried to motivate my choices with comparisons to other tools. 

## The tools:
* `pyenv`: Used only for python version downloading and organization (not used for virtual environments)
* `venv`: Built-in python module that creates virtual environments 
* `poetry`: Python dependency management tool
* `pyproject.toml` and `poetry.lock` file are the "source of truth" for the project

## Installation
