# Prerequisites

This tutorial will be using several software packages. How to use these packages are outside of the scope of this tutorial. Nevertheless, the prerequisites to understand this tutorial are listed below:

- Proficiency in compiling code, preferably in [C++](https://isocpp.org/)
- Knowledge of the [CMake](https://cmake.org/) software build system
- Linux
- Shell scripting, preferably in [Bash](https://www.gnu.org/software/bash/)
  - Bash documentation [link](https://www.gnu.org/software/bash/manual/bash.html)
- Docker

# Problem Statement

There is a [source code repository](https://bitbucket.org/donotalo/starter/). Compile the source code in a Docker container. Extract the output of the compilation in the host system. Hence, the host system will only require Docker to be installed to compile the code and get output.

The source code is in C++. The repository is set up to use CMake to compile the code. There is a build script (`build.sh`) in the repository written for Bash. Calling this script will compile the code. The build directory, where the output will be located, can be found in the build script.

# Host system

The code in this tutorial was run on Debian Linux with [Docker CLI](https://docs.docker.com/engine/install/) installed in it.

# The Dockerfile

Let's start with the [Dockerfile](https://docs.docker.com/engine/reference/builder/). It will define a Docker image that will be used to run a container.

```Dockerfile
FROM debian:12.1

# Install necessary software packages and clean up
RUN apt update && apt install -y \
  cmake \
  gcc \
  g++ \
  git && \
  apt-get clean && \
  rm -rf /var/lib/apt/lists/*

# Define working directory
WORKDIR /src

# Download source code
RUN git clone https://donotalo@bitbucket.org/donotalo/starter.git

# Mark the build script as executable
RUN chmod +x starter/build.sh

# Start bash shell for an interactive session
CMD ["/bin/bash"]
```

Save this file as `Dockerfile`.

# Building a Docker Image

Let's have a convenient script to help with various docker commands. Write a file called `docker.sh` with the following code:

```bash
#!/usr/bin/bash

DEFAULT_REPO=default-repo
DEFAULT_TAG=default-tag

FUNCTION=$1

# Build docker image
build()
{
  repo="${1:-${DEFAULT_REPO}}"
  tag="${2:-${DEFAULT_TAG}}"
  image=${repo}:${tag}

  docker build \
    --file Dockerfile \
    --metadata-file docker-build.log \
    --progress auto \
    --tag "${image}" \
    .
}

${FUNCTION} $2 $3
```

A Docker image can be built by invoking the following command:
```bash
./docker.sh build <name> <tag>
```
Replace `<name>` & `<tag>` with your desired name and tag for the image. Once built, it can be listed by [`docker images`](https://docs.docker.com/engine/reference/commandline/images/) command.
