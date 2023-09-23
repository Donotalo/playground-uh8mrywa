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
FROM debian:12.1

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

# Extracting Data from a Stopped Docker Container

## Running the Docker Image in a Container

We'll run the image in a Docker container using `docker run` command. The container will be assigned a name (`${container_name}`) so that it can be referred later. The path to the build script will be passed as the command to `docker run`. This will override the `CMD` command in the `Dockerfile` and the script will be executed.
```bash
docker run \
    --name ${container_name} \
    ${image_name}:${image_tag} \
    <path to build script in the image>
```

Once the script execution is completed, the container will be stopped. All modifications to the filesystem will be retained in the container.

## Copy Data from Stopped Docker Container

Any file from the Docker container's filesystem can be copied to the host system using `docker cp` command. We need to pass the container name (`${container_name}`), path to the file, and destination location (`${OUTPUT_DIR}`) where to copy the content.
```bash
docker cp ${container_name}:<path to file> ${OUTPUT_DIR}
```

## Updated Docker Script

A new function `extract()` is added to the `docker.sh` script that will run a Docker image in a container and extract desired file(s) out of it.

```bash
#! /usr/bin/bash
#
# Usage:
#   - Build a docker image with optional <name> and <tag>
#     - ./docker.sh build [<name> [<tag>]]

DEFAULT_NAME=default-name
DEFAULT_TAG=default-tag
DEFAULT_CONTAINER=default-container
OUTPUT_DIR=out

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

# Run the docker image in a container and extract output
extract()
{
  container_name="${1:-${DEFAULT_CONTAINER}}"
  image_name="${2:-${DEFAULT_NAME}}"
  image_tag="${3:-${DEFAULT_TAG}}"
  image=${image_name}:${image_tag}
  echo "===== Running ${image} with name ${container_name} ====="

  # Run the container
  # Pass the command to execute, this will override CMD in the Dockerfile
  docker run \
    --name ${container_name} \
    ${image_name}:${image_tag} \
    /src/starter/build.sh

  if [ $? -ne 0 ]; then
    echo "docker run fails for ${image}"
    exit 1;
  fi

  rm -rf ${OUTPUT_DIR}
  mkdir ${OUTPUT_DIR}

  # Copy files from the container filesystem to host
  docker cp ${container_name}:/src/starter/build/bin ./${OUTPUT_DIR}/
}

${FUNCTION} $2 $3 $4
```

# Export Docker Container's Filesystem to a tar Archive

```bash
docker export --output ${filename} ${container_name}
```
