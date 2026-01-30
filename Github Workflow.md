# Github Workflow

## Docker Container

To work, kubernetes need to have the apps on a docker container, so we need to first of all create a container on ghcr.io with our app.

To do it, firstly you need to create a Dockerfile with the needs of your app (Remember to use a base image that can run on arm), in this case i'll showcase how i did to deploy my discord bot [echobot](https://github.com/000Volk000/echoBot).

In my case the Dockerfile ended like this:

```Dockerfile
FROM python:3.14 AS echobot

RUN apt update && apt install -y ffmpeg libopus-dev

WORKDIR /app
COPY . /app

RUN pip install -r requirements.txt

CMD ["python", "main.py"]
```

Once created the Dockerfile we need to make the container and publish it to ghcr.io.

To do that I'll do a github actions workflow by creating a deploy.yaml file on `.github/workflows/deploy.yaml` on the root of the desired repo.

Inside the file we'll need to put something similar to this:

```yaml
name: Build, Push, and Deploy to GHCR

on:
  push:
    tags:
      - "v*.*.*"
  workflow_dispatch:

env:
  DOCKER_BUILDKIT: 1
  IMAGE_NAME: ghcr.io/000Volk000/echobot

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Set up QEMU
        uses: docker/setup-qemu-action@v3

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Login to GitHub Container Registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          file: ./Dockerfile
          push: true
          target: echoBot
          platforms: linux/amd64,linux/arm64
          tags: |
            ${{ env.IMAGE_NAME }}:latest
            ${{ env.IMAGE_NAME }}:${{ github.ref_name }}
          cache-from: type=local,src=/tmp/.buildx-cache
          cache-to: type=local,dest=/tmp/.buildx-cache,mode=max
```

Thats it, you can make all the changes that you want and when you push a commit and add a tag (You can do so with the next commands)

```bash
git tag v1.0.0                    
git push origin v1.0.0
```

It will automatically trigger and push the image to the ghcr.io repo.