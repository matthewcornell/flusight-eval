This directory contains files to support a containerized Docker version of the FluSight Evaluation Report generator that can be run locally or via [AWS ECS](https://aws.amazon.com/ecs/). Following is information needed to set up and run the image.

# Environment variables

The app uses the helper scripts in https://github.com/reichlab/container-utils/, which require the following environment variables: `SLACK_API_TOKEN`, `CHANNEL_ID`; `GH_TOKEN`; `GIT_USER_NAME`, `GIT_USER_EMAIL`; `GIT_CREDENTIALS`. Please that repo for details. Note that it's easiest and safest to save these in a `*.env` file and then pass that file to `docker run`.

This app supports a `DRY_RUN` environment variable.


# Python and R packages

The project's [Dockerfile](Dockerfile) uses the [requirements.txt](..%2Frequirements.txt) and [renv.lock](..%2Frenv.lock) files to install required Python and R packages. Users should update those files as needed to maintain a working application.

# `/data` dir

The app expects a volume (either a [local Docker one](https://docs.docker.com/storage/volumes/) or an [AWS EFS](https://aws.amazon.com/efs/) file system) to be mounted at `/data` and which contains this required GitHub repo:
- https://github.com/cdcepi/FluSight-forecast-hub

How that volume is populated (i.e., running `git clone` calls) depends on whether you're running locally or on ECS. See [The `/data` dir](https://github.com/reichlab/container-utils/blob/main/README.md#the-data-dir) section in https://github.com/reichlab/container-utils/blob/main/README.md for details.

# To build the image

```bash
cd "path-to-this-repo"
docker build --tag=flusight-eval:1.0 --file=docker/Dockerfile .
```

# To run the image locally

```bash
docker run --rm \
  --mount type=volume,src=data_volume,target=/data \
  --env-file /path-to-env-dir/.env \
  flusight-eval:1.0
```

# To publish the image

> Note: We build for the `amd64` architecture because that's what most Linux-based servers (including AWS) use natively. This is as opposed to Apple Silicon Macs, which have an `arm64` architecture.

```bash
cd "path-to-this-repo"
docker login -u "reichlab" docker.io
docker build --platform=linux/amd64 --tag=reichlab/flusight-eval:1.0 --file=docker/Dockerfile .
docker push reichlab/flusight-eval:1.0
```

# To run the image on AWS ECS

See https://github.com/reichlab/container-utils/blob/main/docs/ecs.md for details.
