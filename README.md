# nodepyenv

`nodepyenv` (no dep py env) is a small Python package that creates an isolated, no-dependency conda virtual environment using `micromamba`. It uses only standard Python libraries and can be easily used with `PyInstaller` to create a standalone executable with zero dependencies.

## What `nodepyenv` does

The program:
- Checks to see if `micromamba` is present in the `mamba_root` directory.
- If needed, downloads the latest `micromamba` from the `micromamba` GitHub release page.
- Creates a conda environment in the `mamba_root` directory by installing the packages specified in the `environment.yaml` file using `micromamba`.
- Returns the path to the Python executable in the new environment.

The returned Python executable can be used to run commands directly in the environment. For example, to run the Python interpreter in the environment, use the command:

```
./envs/<env_name>/bin/python <your python application>
```

Or use `micromamba` and the Python executable in the environment with:

```
./micromamba -r . run -n <env_name> python <your python application>
```
**Note:**
`micromamba` is a small, self-contained conda package manager written in C++. It is available for Linux, Windows, and MacOS (although only Linux and Windows are supported/tested in this version of npdepyenv). 
- [https://mamba.readthedocs.io/en/latest/user_guide/micromamba.html](https://mamba.readthedocs.io/en/latest/user_guide/micromamba.html)
- [https://github.com/mamba-org/micromamba-releases/releases](https://github.com/mamba-org/micromamba-releases/releases)

The environment files are created in the same directory (`mamba_root`) as the `environment.yaml` file within subdirectories `./envs` and `./pkgs`.

The optional arguments are the path to the `mamba_root` directory (default is current directory) and the name of the environment YAML file (default `environment.yaml`).

## Usage

`nodepyenv` can be "compiled" to a standalone executable using `PyInstaller` with the command:
```
pyinstaller --onefile nodepyenv.py
```
The resulting executable can then be used to create localized complex conda environments with zero system dependencies i.e. a local Python install is not needed. 

It can also be run directly with a local Python interpreter:

```
python nodepyenv.py -r <MAMBA_ROOT> -f <ENVFILE>
```

Or it can be imported as a module:

```python
from nodepyenv import create_env
...
python_executable = create_env(mamba_root, envfile)
...
subprocess.run([python_executable, <your python application>])
```

That can be used to create a small installer/runner application for a much larger python application.

## Help

`./nodepyenv --help`
```
usage: nodepyenv [-h] [-v] [-f ENVFILE] [-r MAMBA_ROOT]

Create a conda environment in the current mamba_root directory from an environment.yaml file. See
https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html#create-env-file-manually for
YAML file format.

options:
  -h, --help            show this help message and exit
  -v, --version         show program's version number and exit
  -f ENVFILE, --envfile ENVFILE
                        environment.yaml file to use
  -r MAMBA_ROOT, --mamba_root MAMBA_ROOT
                        mamba root directory
```

## Examples

#### Create an env containing `pyinstaller` and "compile" `nodepyenv.py`
This is a simple example that uses `nodepyenv.py` to create an environment containing `pyinstaller` and then uses `pyinstaller` to create a standalone executable version of `nodepyenv`.

On a system with a Python interpreter and a copy of nodepyenv.py,
```
mkdir PyInstaller
```
Create env yaml file `pyinstaller-env.yaml` with contents:
```yaml
name: pyinstaller
channels:
  - conda-forge
dependencies:
  - pyinstaller
```
Create the environment:
```
python nodepyenv.py -r PyInstaller -f pyinstaller-env.yaml
```
This creates the environment in the `PyInstaller` directory.
Note: this env will contain a pyinstaller executable as well as a python executable. 
```
cd PyInstaller
```
Create the standalone executable:
```
./envs/pyinstaller/bin/pyinstaller --onefile ../nodepyenv.py
```
This creates the standalone executable `nodepyenv` in the `dist` directory from the PyInstaller build.
The executable is approximately 15MB.

You can take that small standalone executable and use it to create a localized conda environment with zero dependencies on other systems without needing a local Python interpreter.

#### Create an env with packages needed to run Stable Diffusion
This is an example of using the small independent `nodepyenv` executable to create a more complex environment. (Possibly on a system that does not have a local Python interpreter.) 

We will create an env with a nightly PyTorch build and the packages needed to run a Stable Diffusion demo app.

Create env yaml file `stable-diffusion-env.yaml` with contents:
```yaml
name: stable-diffusion
channels:
  - conda-forge
  - nvidia
dependencies:
  - python=3.10
  - pytorch-nightly::pytorch
  - pytorch-nightly::torchvision
  - pytorch-cuda=12.1
  - transformers
  - diffusers
  - accelerate
  - ftfy
  - pillow
  - ninja
  - altair
  - pip
  - pip:
      - gradio
```
This illustrates a more complex environment YAML file.

Suppose you have a directory `StableDiffusion` that contains the `nodepyenv` executable, the `stable-diffusion-env.yaml` file, and your nifty demo `stable_diffusion_demo.py` file.

Create the environment:
```
./nodepyenv -f stable-diffusion-env.yaml
```
Note: we are in the directory we want for mamba_root so we can let nopdepyenv use the default.

Now you can run your demo app with:
```
./envs/stable-diffusion/bin/python stable_diffusion_demo.py
```

Of course, you could wrap that in an installer/runner application that includes code like;
```python
from nodepyenv import create_env
...
python_executable = create_env(mamba_root, envfile)
...
subprocess.run([python_executable, <your python application>])
```

I hope you find this useful!