# Pre-requisites

This tutorial will be using several software packages. How to use these packages are outside of the scope of this tutorial. Nevertheless, the pre-requisites to understand this tutorial are listed below:

- How to compile code, preferrebly in [C++](https://isocpp.org/)
- [CMake](https://cmake.org/) software build system
- Linux
- Shell scripting, preferrebly in [Bash](https://www.gnu.org/software/bash/)
  - Bash documentation [link](https://www.gnu.org/software/bash/manual/bash.html)
- Docker

# Problem Statement

There is a [source code repository](https://bitbucket.org/donotalo/starter/). Compile the source code in a Docker container. Extract the output of the compilation in the host system.

The source code is in C++. The repository is setup to use CMake to compile the code. There is a build script (`build.sh`) in the repository written for Bash. Calling this script will compile the code. The build directory, where the output will be located, can be found in the build script.

# Host system

The code in this tutorial was run on Debian (Linux) with [Docker CLI](https://docs.docker.com/engine/install/) installed in it.

# The Dockerfile

Let's start with the Dockerfile. It will define a Docker image based on which a Docker container will run.

```Dockerfile
FROM debian:bookworm

# Install necessary software packages
RUN apt update
RUN apt install -y \
  cmake \
  gcc \
  g++ \
  git

# Define working directory
WORKDIR /src

# Download source code
RUN git clone https://donotalo@bitbucket.org/donotalo/starter.git

# Mark the build script as executable
RUN chmod +x starter/build.sh

# Start bash shell for interactive session
CMD ["/usr/bin/bash"]
```

Save this file as `Dockerfile`.

# Building a Docker Image

Let's have a convenient script to help with various docker commands. Write a file called `docker.sh` with the following code:

```bash
#! /usr/bin/bash

DEFAULT_NAME=default-name
DEFAULT_TAG=default-tag

FUNCTION=$1

# Build docker image
build()
{
  name="${1:-${DEFAULT_NAME}}"
  tag="${2:-${DEFAULT_TAG}}"
  echo "===== Building Docker image ${name}:${tag} ====="

  docker build \
    --file Dockerfile \
    --metadata-file docker-build.log \
    --progress auto \
    --tag "${name}:${tag}" \
    .
}

${FUNCTION} $2 $3
```

A Docker image can be built by invoking the following command:
```bash
./docker.sh build <name> <tag>
```
Replace `<name>` & `<tag>` with your desired name and tag for the image. Once built, it can be listed by `docker images` command.

# Running the Docker Image

```bash
docker run \
    --name ${container_name} \
    ${name}:${tag} \
    /src/starter/build.sh
```

# Extracting Data from Stopped Docker Container

```bash
docker cp ${container_name}:/src/starter/build ./${OUTPUT_DIR}/
```

# Export Docker Container's Filesystem to a tar Archive

```bash
docker export --output ${filename} ${container_name}
```
