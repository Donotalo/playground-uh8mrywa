# Extra

## Export Docker Container's Filesystem to a tar Archive

It's possible to extract the full filesystem of a Docker container to a tar archive using [`docker export`](https://docs.docker.com/engine/reference/commandline/export/) command like below:

```bash
docker export --output ${filename} ${container_name}
```
