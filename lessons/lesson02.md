# Extracting Data from a Stopped Docker Container

## Running the Docker Image in a Container

We'll run the image in a Docker container using [`docker run`](https://docs.docker.com/engine/reference/run/) command. The container will be assigned a name (`${container_name}`) so that it can be referred later. The path to the build script (`${cmd}`) will be passed as the command to `docker run`. This will override the `CMD` command in the `Dockerfile` and the script will be executed.
```bash
docker run \
    --name ${container_name} \
    ${image} \
    ${cmd}
```

Once the script execution is completed, the container will be stopped. All modifications to the filesystem will be retained in the container.

## Copy Data from Stopped Docker Container

Any file from the Docker container's filesystem can be copied to the host system using [`docker cp`](https://docs.docker.com/engine/reference/commandline/cp/) command. We need to pass the container name (`${container_name}`), path to the file (`${path_to_copy}`), and destination location in the host system (`${output_dir}`) where to copy the content.
```bash
docker cp ${container_name}:${path_to_copy} ${output_dir}
```

## Updated Docker Script

A new function `extract()` is added to the `docker.sh` script that will run a Docker image in a container and extract desired file(s) out of it.

```bash
#! /usr/bin/bash

# Run the docker image in a container and extract output
extract()
{
  repo="${1:-${DEFAULT_REPO}}"
  tag="${2:-${DEFAULT_TAG}}"
  image=${repo}:${tag}
  container_name="cont-$(date +%s)"

  # The variables below should be updated to suit a project's need
  cmd=/src/starter/build.sh
  path_to_copy=/src/starter/build/bin
  output_dir=./out

  # Run the container
  # Pass the command to execute, this will override CMD in the Dockerfile
  docker run \
    --name ${container_name} \
    ${image} \
    ${cmd}

  rm -rf ${output_dir}
  mkdir -p ${output_dir}
  
  # Copy files from the container filesystem to host
  docker cp ${container_name}:${path_to_copy} ${output_dir}

  # Data copied, delete the container
  docker container rm -fv ${container_name}
}
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
