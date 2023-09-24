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

Let's start with the [Dockerfile](https://docs.docker.com/engine/reference/builder/). It will define a Docker image that will be used to run a container.

```Dockerfile
# Start with Debian 12.1
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
  image=${name}:${tag}
  echo "===== Building Docker image ${image} ====="

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
Replace `<name>` & `<tag>` with your desired name and tag for the image. Once built, it can be listed by `docker images` command.

# Extracting Data from a Stopped Docker Container

## Running the Docker Image in a Container

We'll run the image in a Docker container using [`docker run`](https://docs.docker.com/engine/reference/run/) command. The container will be assigned a name (`${container_name}`) so that it can be referred later. The path to the build script (`${build_script_path}`) will be passed as the command to `docker run`. This will override the `CMD` command in the `Dockerfile` and the script will be executed.
```bash
docker run \
    --name ${container_name} \
    ${image} \
    ${build_script_path}
```

Once the script execution is completed, the container will be stopped. All modifications to the filesystem will be retained in the container.

## Copy Data from Stopped Docker Container

Any file from the Docker container's filesystem can be copied to the host system using [`docker cp`](https://docs.docker.com/engine/reference/commandline/cp/) command. We need to pass the container name (`${container_name}`), path to the file (`${path_to_copy}`), and destination location (`${OUTPUT_DIR}`) where to copy the content.
```bash
docker cp ${container_name}:${path_to_copy} ${OUTPUT_DIR}
```

## Updated Docker Script

A new function `extract()` is added to the `docker.sh` script that will run a Docker image in a container and extract desired file(s) out of it.

```bash
#! /usr/bin/bash

DEFAULT_REPO=default-repo
DEFAULT_TAG=default-tag
OUTPUT_DIR=out

FUNCTION=$1

# Build docker image
build()
{
# ...
}

# Run the docker image in a container and extract output
extract()
{
  repo="${1:-${DEFAULT_REPO}}"
  tag="${2:-${DEFAULT_TAG}}"
  image=${repo}:${tag}
  container_name="cont-$(date +%s)"

  # The variables below should be updated to suit a project's need
  build_script_path=/src/starter/build.sh
  path_to_copy=/src/starter/build/bin

  echo "===== Running ${image} with name ${container_name} ====="

  # Run the container
  # Pass the command to execute, this will override CMD in the Dockerfile
  docker run \
    --name ${container_name} \
    ${image} \
    ${build_script_path}
  if [ $? -ne 0 ]; then
    echo "docker run fails for ${image}"
    exit 1;
  fi

  rm -rf ${OUTPUT_DIR}
  mkdir ${OUTPUT_DIR}

  # Copy files from the container filesystem to host
  docker cp ${container_name}:${path_to_copy} ${OUTPUT_DIR}
 
  if [ $? -ne 0 ]; then
    echo "docker cp fails for ${container_name}"
    exit 1;
  fi

  # Data copied, delete the container
  docker container rm -fv ${container_name}
  if [ $? -ne 0 ]; then
    echo "docker container rm fails for ${container_name}"
    exit 1;
  fi
}

${FUNCTION} $2 $3
```

Once the image is ready, it can be run in a container and necessary files can be copied to host system using the following command:
```bash
./docker.sh extract <repo> <tag>
```

The `extract()` function will delete the container and all filesystems associated with it if all docker command execution is successful.

# Export Docker Container's Filesystem to a tar Archive

```bash
docker export --output ${filename} ${container_name}
```
