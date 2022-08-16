
# Prolog Jupyter Kernel

A [Jupyter](https://jupyter.org/) kernel for Prolog based on the [IPython kernel](https://github.com/ipython/ipykernel).

By default, [SICStus Prolog](https://sicstus.sics.se/) and [SWI-Prolog](https://www.swi-prolog.org/) (which is the actual default) are supported. The kernel is implemented in a way that basically all functionality except the loading of configuration files can easily be overriden. This is especially useful for extending the kernel for further Prolog implementations or running code with a different version of an implementation. For further information about this, see [Configuration](#configuration).

**Note:** The project is still under development and so far, only a [development installation](#development-install) is possible.
Furthermore, the usage of the kernel is limited. In order to execute Prolog code, the Jupyter kernel needs to communicate with a Prolog server, the code of which can be found in the directory [prolog_server](./prolog_server). By default, the kernel expects this folder to be in the same directory as the folder which contains the Jupyter notebook. Unless configured differently, if this is not the case, no code can be executed.


## Requirements

- At least Python 3.5
  - Tested with Python 3.8.10
- Jupyter installation with JupyterLab and/or Juypter Notebook
  - Tested with
    - jupyter_core: 4.10.0
    - jupyterlab: 3.2.9
    - notebook: 6.4.8
- A Prolog installation for the configured implementation
  - In order to use the default configuration, SWI-Prolog is needed and needs to be on the PATH
  - Tested with version 8.4.3 of SWI-Prolog and SICStus 4.5.1


## Development Install

1. `git clone https://github.com/anbre/prolog-jupyter-kernel.git`
2. Change to the root directory of the repository
3. `pip install .`
4. Install the kernel specification directory:
    - `python -m prolog_kernel.install`
    - There are the following options which can be seen when running `python -m prolog_kernel.install -h`
      - `--user`: install to the per-user kernel registry (default if not root and no prefix is specified)
      - `--sys-prefix`: install to Python's sys.prefix (e.g. virtualenv/conda env)
      - `--prefix PREFIX`: install to the given prefix: PREFIX/share/jupyter/kernels/ (e.g. virtualenv/conda env)


## Uninstall

1. Change to the root directory of the repository
2. `pip uninstall prolog_kernel`
3. `jupyter kernelspec remove prolog_kernel`


## Usage

The directory [notebooks](./notebooks) contains notebooks demonstrating the features of the kernel when used with SWI- and SICStus Prolog.

Additionally, the file [jsonrpc_server_tests.pl](./prolog_server/jsonrpc_server_tests.pl) defines some PL-Unit tests.
They provide further examples of what kind of code the Prolog server (and therefore the kernel) can handle and what the expected behavior is.


## Configuration

The kernel can be configured by defining a Python config file named `prolog_kernel_config.py`. The kernel will look for this file in the Jupyter config path (can be retrieved with `jupyter --paths`) and the current working directory. An example of such a configuration file with an explanation of the options and their default values commented out can be found [here](./prolog_kernel/prolog_kernel_config.py).

**Note:** If a config file exists in the current working directory, its configuration overrides values from other configuration files.

In general, the kernel can be configured to use a different Prolog implementation, Prolog server (which is responsible for code execution), and kernel implementation. This can be achieved by configuring these two options:
- `implementation_id`: The ID of the Prolog implementation which is used to execute code
  - For SWI- and SICStus Prolog, the IDs `swi` and `sicstus` are expected.
- `implementation_data`: The implementation specific data which is needed to run the Prolog server for code execution.
  - This needs to be a dictionary which needs to at least contain an entry for the configured `implementation_id`.
  - Each entry needs to define values for
    - `failure_response`: The output which is displayed if a query fails
    - `success_response`: The output which is displayed if a query succeeds without any variable bindings
    - `error_prefix`: The prefix that is output for error messages
    - `informational_prefix`: The prefix that is output for informational messages
    - `program_arguments`: Command line arguments with which the Prolog server can be started
  - Additionally, a `kernel_implementation_path` (which needs to be absolute) can be provided:
    - The corresponding module needs to define a class `PrologKernelImplementation` as a subclass of `PrologKernelBaseImplementation`, which can be used to override the kernel's behavior (see [Overriding the Kernel Implementation](#overriding-the-kernel-implementation)).

In addition to configuring the Prolog implementation to be used, the Prolog server implements a predicate with which the implementation can be changed. In order for this to work, the configured `implementation_data` dictionary needs to contain data for more than one Prolog implementation.
(TODO implement!)


### Overriding the Kernel Implementation

The actual kernel code is not implemented by the kernel class itself. Instead, there is the file [prolog_kernel_base_implementation.py](./prolog_kernel/prolog_kernel_base_implementation.py) which defines the class `PrologKernelBaseImplementation`. When the kernel is started, a (sub)object of this class is created. It handles the starting of and communication with the Prolog server. For all requests (execution, shutdown, completion, inspection) the kernel receives, a `PrologKernelBaseImplementation` method is called. By creating a subclass of this and defining the path to it as `kernel_implementation_path`, the actual implementation code can be replaced.

If no such path is defined, the path itself or the defined class is invalid, a default implementation is used instead. In case of SWI- and SICStus Prolog, the files [swi_kernel.py](./prolog_kernel/swi_kernel.py) and [sicstus_kernel.py](./prolog_kernel/sicstus_kernel.py) are used. Otherwise, the base implementation from the file [prolog_kernel_base_implementation.py](./prolog_kernel/prolog_kernel_base_implementation.py) is loaded.
