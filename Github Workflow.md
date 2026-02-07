# Github Workflow

## Docker Container

To work, kubernetes need to have the apps on a docker container, so we need to first of all create a container on ghcr.io with our app.

To do it, firstly you need to create a Dockerfile with the needs of your app (Remember to use a base image that can run on arm).

As an example, this is the Dockerfile of my discord bot [echobot](https://github.com/000Volk000/echoBot):

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
  IMAGE_NAME: ghcr.io/<user>/<app>

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
          target: <Docker Name (from --- as name)>
          platforms: linux/amd64,linux/arm64
          tags: |
            ${{ env.IMAGE_NAME }}:latest
            ${{ env.IMAGE_NAME }}:${{ github.ref_name }}
          cache-from: type=local,src=/tmp/.buildx-cache
          cache-to: type=local,dest=/tmp/.buildx-cache,mode=max
```

Thats it, you can make all the changes that you want and when you push a commit and add a tag (You can do so with the next commands)

```bash
git tag vx.x.x
git push origin vx.x.x
```

It will automatically trigger and push the image to the ghcr.io repo.

## Chart Updater

Thats good, but we need to update the charts repo we created that is being watched by ArgoCD.

To do so, i'll add the following section to our `.github/workflows/deploy.yaml`:

```yaml
update-charts:
  runs-on: ubuntu-latest
  needs: build-and-push
  steps:
    - name: Checkout
      uses: actions/checkout@v4
      with:
        repository: ${{ env.CHARTS_REPOSITORY }}
        path: charts
        token: ${{ secrets.CHARTS_REPO_PAT }}

    - name: Update image tag
      run: |
        NEW_TAG=${{ github.ref_name }}
        TARGET_DIR="charts/${{ env.CHARTS_DIRECTORY }}"

        sed -i "s|image: ${{ env.IMAGE_NAME }}:.*|image: ${{ env.IMAGE_NAME }}:$NEW_TAG|g" "$TARGET_DIR/deployment.yaml"

    - name: Commit and push changes
      run: |
        cd charts
        git config --global user.name "github-actions[bot]"
        git config --global user.email "github-actions[bot]@users.noreply.github.com"
        git add .

        if ! git diff --staged --quiet; then
          git commit -m "ci: Update image version to ${{ github.ref_name }} in ${{ env.CHARTS_DIRECTORY }}"
          git push
        else
          echo "No changes to commit. El YAML ya estaba actualizado con la versión ${{ github.ref_name }}."
        fi
```

See the `sed` on the task "Update image tag"? That is the command that changes the version on the deploy.yaml file we'll make on the next section, if you have the image on any other yaml, copy-paste the sed line and change the deployment.yaml of the end to the desired .yaml (This will be needed on CronJobs).

And created 2 new environment variables (CHARTS_REPOSITORY and CHARTS_DIRECTORY) for it to work at the env of the start:

```yaml
env:
  DOCKER_BUILDKIT: 1
  IMAGE_NAME: ghcr.io/<user>/<app>
  CHARTS_REPOSITORY: <user>/charts
  CHARTS_DIRECTORY: <app>
```

You can push the changes to github but DON'T put a tag on it for the moment.

For make the workflow work, we need to create a `repository secret` called exactly `CHARTS_REPO_PAT` to do so, we go to our Repository -> Settings -> Secrets and Variables -> Actions -> New Repository Secret.

In name we'll put `CHARTS_REPO_PAT` and in the content we'll put the same PAT that we put on Argo when initialized the charts repo on it.

## Next Step

That's done, we just need to create the .yaml files that need argo to make all work -> [Argo Workflow](Argo%20Workflow.md)

