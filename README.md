# nodepyenv

`nodepyenv` is a small Python package that creates an isolated, no-dependency conda virtual environment using `micromamba`. It uses only standard Python libraries and can be easily used with `PyInstaller` to create a standalone executable with zero dependencies.

## Installation

`nodepyenv` can be "compiled" to a standalone executable using `PyInstaller` with the command:

```
pyinstaller --onefile nodepyenv.py
```

It can also be run directly with Python:

```
python nodepyenv.py
```

Or it can be imported as a module:

```python
from nodepyenv import create_env
```

## Usage

`micromamba` is a small, self-contained conda package manager written in C++. It is available for Linux, Windows, and MacOS (arm64) (although only Linux and Windows are supported in this version). MacOS support should be simple to add since there are precompiled `micromamba` binaries for MacOS (arm64) and MacOS (intel) available.

The environment files are created in the same directory (`mamba_root`) as the `environment.yaml` file within subdirectories `./envs` and `./pkgs`.

The optional arguments are the path to the `mamba_root` directory (default is current directory) and the name of the environment YAML file (default `environment.yaml`). If no arguments are given, the defaults are used.

The program:
- Checks to see if `micromamba` is present in the `mamba_root` directory.
- If needed, downloads the latest `micromamba` from the `micromamba` GitHub release page.
- Creates a conda environment in the `mamba_root` directory by installing the packages specified in the `environment.yaml` file.
- Prints the path to the Python executable in the environment.

The returned Python executable can be used to run commands directly in the environment. For example, to run the Python interpreter in the environment, use the command:

```
./envs/<env_name>/bin/python <your python application>
```

Or use `micromamba` and the Python executable in the environment with:

```
./micromamba -r . run -n <env_name> python <your python application>
```

## License

This project is licensed under the terms of the MIT license. See [LICENSE](LICENSE) for more information.