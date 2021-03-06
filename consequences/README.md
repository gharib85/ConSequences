# `ConSequences` Codebase
## Overview

| ![ConSequences Code Pipeline](code.png "ConSequences Code Pipeline")  |
|---|
| **Figure**: ConSequences Code Pipeline. Code is represented with rounded corners and data with sharp corners. Code blocks are further split into Docker (blue) and Python (gray).|

ConSequences is primarily a pipeline of Python code used to pre- and post-process data, along with a dispatcher for calling contraction sequence algorithms wrapped in Docker images.
In this README we overview how to install the software, and how to run it with various use cases.

## Installation (General)

This codebase requires Python 3 and Docker.

In general, we recommend wrangling Python dependencies using an environment like `virtualenv`; see below for example instructions for installing on Ubuntu 18.04, and refer to the [virtualenv docs](https://virtualenv.pypa.io/en/stable/) page for more information.
Regardless of whether a virtual environment is used or not, the Python requirements are included in `requirements.txt` and may be installed with `pip install -r requirements.txt`.

For stability, we recommend installing Docker Community Edition (CE) on the stable channel.
Detailed instructions for installing Docker CE can be found in the [docker docs](https://docs.docker.com/install/).

## Installation (Ubuntu 18.04)

_(This guide is current as of July 23, 2018)_


#### Python3 Setup
At time of writing, a fresh installation of Ubuntu 18.04 comes with `python3` already, but requires `pip3` to install the `virtualenv` virtual environment controller. Install it with

```bash
sudo apt-get install python3-pip
```

then install virtualenv:

```bash
pip3 install virtualenv
```

and finally initialize a new Python3 virtual environment:

```bash
virtualenv env -p python3
```

To activate this environment the from `consequences/` directory:

```bash
source env/bin/activate
```

You should now see `(env)` in the prefix of your terminal session. To deactivate the environment:

```
deactivate
```

The deactivation command can be run from any directory.

With the virtual environment activated, install the Python3 `ConSequences` dependencies by running:

```bash
pip -r requirements.txt
```

#### Docker-CE Setup

Docker-CE should be set up using the official [Docker Docs](https://docs.docker.com/install/linux/docker-ce/ubuntu/#install-docker-ce-1) installation guide. Follow it until you install docker-ce with the following command:

```bash
sudo apt-get install docker-ce
```

Natively, Docker must be run by root. Instead of typing `sudo` with every command, we instead follow the post-installation guidelines by creating a `docker` group for users who will be running docker images:

```bash
sudo groupadd docker
sudo usermod -aG docker $USER
```

After rebooting, run the following command to verify that Docker has been installed correctly:

```bash
docker run hello-world
```

## How to run

At its core, `ConSequences` is a framework for testing contraction sequence algorithms on a user-defined dataset of interest.
In this section we detail the algorithms provided and data formats supported natively, along with instructions for executing single runs and larger batch experiments from the command-line interface.


#### Provided algorithms

| Algorithm    | Description                    | Source Link                                       |
|--------------|--------------------------------|---------------------------------------------------|
| FreeTDI      | PACE 2017 winner               | https://github.com/freetdi/p17                    |
| Meiji        | PACE 2017 runner-up            | https://github.com/TCS-Meiji/PACE2017-TrackA      |
| netcon       | BFS-based from MERA literature | https://arxiv.org/abs/1304.6112                   |
| QuickBB      | Elimination-ordering--based    | http://www.hlt.utdallas.edu/~vgogate/quickbb.html |
| qTorch-Stoch | Heuristic provided in qTorch   | https://github.com/aspuru-guzik-group/qtorch      |

#### Supported data formats

All data must be in PACE's `.gr` format, specified here: https://pacechallenge.org/

#### Defining a config file
The single interaction that a user has with the interface occurs at the config file.
The configuration is split into two steps: data and experimental parameters.

Three types of data input are supported: `regular` (as in r-regular graphs used for QAOA circuits), `mera`, and `manual`.

The regular graph parameters specify a corpus of graphs to generate into the `output_dir`:
```
# Data
type: regular
n: 30 31 1
r: 3 4 1
graph_seeds: 1 1 1
output_dir: /test
```

The MERA parameters similarly define a corpus:
```
# Data
type: mera
...
output_dir: /test
```

The manual option provides an `input_dir` containing graph files from a user-generated corpus (e.g. classic treewidth benchmarks, a new application, etc.):
```
# Data
type: manual
input_dir: /my_graph_files (Folder where input files are kept)
output_dir: /test
```

The experiment parameters included the following:
```
# Experiment
num_threads: [from 1 to n]
algo_seed: [default, or a positive int. Currently not used]
timeout: 1 [seconds]
verbose: true [print stdout and stderr from Python code and Docker calls]
algorithms: [a space-separated list from meiji-e quickbb freetdi netcon qtorch]
csv_filename: [default for config_filename.csv, or specified filename]
```

Once the experiment is defined in a config file (e.g. `experiment.cfg`), it can be executed with 
```bash
python consequences.batch.py -config experiment.cfg
```
