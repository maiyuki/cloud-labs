# Simple Dockerfile

## Prerequisites

- docker
- Docker Hub Account

## Build and image and run it in a container

The `Dockerfile` example creates a super simple flask app.

Build:

```
docker build -t <YOUR_DOCKERHUB_NAME>/<IMAGE_NAME>:<IMAGE_TAG> .
docker images
```

Run image in container:

```
docker run -dp 8000:5000 <YOUR_DOCKERHUB_NAME>/<IMAGE_NAME>:<IMAGE_TAG>
docker start <container> (if your container is not running)
docker ps -a
curl localhost:8000
```

Push image to Docker Hub:
```
docker login
docker push <YOUR_DOCKERHUB_NAME>/<IMAGE_NAME>:<IMAGE_TAG>
```

## Links

https://docs.docker.com/develop/develop-images/dockerfile_best-practices/

https://docs.docker.com/language/python/

https://docs.docker.com/get-started/
