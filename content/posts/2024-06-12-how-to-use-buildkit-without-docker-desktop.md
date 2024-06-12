---
title: How To Use BuildKit without Docker Desktop
date: 2024-06-12
tags:
-   docker
---

*N.B. These instructions are for macOS;
mileage on other platforms may vary.*

[BuildKit](https://docs.docker.com/build/buildkit/)
is a Docker image building engine
that allows more features than the default engine,
like parallelised stages, multi-platforms builds, and better file mounting.
It is usually distributed with Docker Desktop,
but in some environments that cannot be used without a licence.
At work, we use [Colima](https://github.com/abiosoft/colima),
an open-source container runtime,
instead.

However, this does not come with BuildKit,
which must be installed separately.
Assuming `docker` and `colima` have already been installed:

```shell
brew install docker-buildx
```

The plugin then needs to be symlinked
into the Docker config directory:

```shell
mkdir -p "${DOCKER_CONFIG:-$HOME/.docker}/cli-plugins"
ln -sfn $(which docker-buildx) "${DOCKER_CONFIG:-$HOME/.docker}/cli-plugins/docker-buildx"
```

Finally, install the plugin:

```shell
docker buildx install
```

The `docker build` command will now use BuildKit by default.
The legacy engine can be used by setting `DOCKER_BUILDKIT=0` in the environment;
BuildKit can then be used by running `docker buildx build`.
