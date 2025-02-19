[![GitHub release](https://img.shields.io/github/release/docker/setup-buildx-action.svg?style=flat-square)](https://github.com/docker/setup-buildx-action/releases/latest)
[![GitHub marketplace](https://img.shields.io/badge/marketplace-docker--setup--buildx-blue?logo=github&style=flat-square)](https://github.com/marketplace/actions/docker-setup-buildx)
[![CI workflow](https://img.shields.io/github/actions/workflow/status/docker/setup-buildx-action/ci.yml?branch=master&label=ci&logo=github&style=flat-square)](https://github.com/docker/setup-buildx-action/actions?workflow=ci)
[![Test workflow](https://img.shields.io/github/actions/workflow/status/docker/setup-buildx-action/test.yml?branch=master&label=test&logo=github&style=flat-square)](https://github.com/docker/setup-buildx-action/actions?workflow=test)
[![Codecov](https://img.shields.io/codecov/c/github/docker/setup-buildx-action?logo=codecov&style=flat-square)](https://codecov.io/gh/docker/setup-buildx-action)

## About

GitHub Action to set up Docker [Buildx](https://github.com/docker/buildx).

This action will create and boot a builder that can be used in the following
steps of your workflow if you're using Buildx or the [`build-push` action](https://github.com/docker/build-push-action/).
By default, the [`docker-container` driver](https://docs.docker.com/build/building/drivers/docker-container/)
will be used to be able to build multi-platform images and export cache using
a [BuildKit](https://github.com/moby/buildkit) container.

![Screenshot](.github/setup-buildx-action.png)

___

* [Usage](#usage)
* [Configuring your builder](#configuring-your-builder)
* [Version pinning](#version-pinning)
* [Customizing](#customizing)
  * [inputs](#inputs)
  * [outputs](#outputs)
  * [environment variables](#environment-variables)
* [Notes](#notes)
  * [`nodes` output](#nodes-output)
  * [BuildKit container logs](#buildkit-container-logs)
* [Using on GHES](#using-on-ghes)
* [Contributing](#contributing)

## Usage

```yaml
name: ci

on:
  push:

jobs:
  buildx:
    runs-on: ubuntu-latest
    steps:
      -
        name: Checkout
        uses: actions/checkout@v3
      -
        # Add support for more platforms with QEMU (optional)
        # https://github.com/docker/setup-qemu-action
        name: Set up QEMU
        uses: docker/setup-qemu-action@v2
      -
        name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2
```

## Configuring your builder

See https://docs.docker.com/build/ci/github-actions/configure-builder/

## Version pinning

This action builds images using [Buildx](https://github.com/docker/buildx) and
[BuildKit](https://github.com/moby/buildkit). By default, the action will
attempt to use the latest version of Buildx available on the GitHub Runner
(the build client) and the latest release of BuildKit (the build server).

To pin to a specific version of Buildx, use the `version` input. For example,
to pin to Buildx v0.10.0:

```yaml
- name: Set up Docker Buildx
  uses: docker/setup-buildx-action@v3
  with:
    version: v0.10.0
```

To pin to a specific version of BuildKit, use the `image` option in the
`driver-opts` input. For example, to pin to BuildKit v0.11.0:

```yaml
- name: Set up Docker Buildx
  uses: docker/setup-buildx-action@v3
  with:
    driver-opts: image=moby/buildkit:v0.11.0
```

## Customizing

### inputs

Following inputs can be used as `step.with` keys:

> `List` type is a newline-delimited string
> ```yaml
> driver-opts: |
>   image=moby/buildkit:master
>   network=host
> ```

> `CSV` type must be a newline-delimited string
> ```yaml
> platforms: linux/amd64,linux/arm64
> ```

| Name              | Type     | Description                                                                                                                                                                                     |
|-------------------|----------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `version`         | String   | [Buildx](https://github.com/docker/buildx) version. (eg. `v0.3.0`, `latest`, `https://github.com/docker/buildx.git#master`)                                                                     |
| `driver`          | String   | Sets the [builder driver](https://docs.docker.com/engine/reference/commandline/buildx_create/#driver) to be used (default `docker-container`)                                                   |
| `driver-opts`     | List     | List of additional [driver-specific options](https://docs.docker.com/engine/reference/commandline/buildx_create/#driver-opt) (eg. `image=moby/buildkit:master`)                                 |
| `buildkitd-flags` | String   | [Flags for buildkitd](https://docs.docker.com/engine/reference/commandline/buildx_create/#buildkitd-flags) daemon (since [buildx v0.3.0](https://github.com/docker/buildx/releases/tag/v0.3.0)) |
| `install`         | Bool     | Sets up `docker build` command as an alias to `docker buildx` (default `false`)                                                                                                                 |
| `use`             | Bool     | Switch to this builder instance (default `true`)                                                                                                                                                |
| `endpoint`        | String   | [Optional address for docker socket](https://docs.docker.com/engine/reference/commandline/buildx_create/#description) or context from `docker context ls`                                       |
| `platforms`       | List/CSV | Fixed [platforms](https://docs.docker.com/engine/reference/commandline/buildx_create/#platform) for current node. If not empty, values take priority over the detected ones.                    |
| `config`¹         | String   | [BuildKit config file](https://docs.docker.com/engine/reference/commandline/buildx_create/#config)                                                                                              |
| `config-inline`¹  | String   | Same as `config` but inline                                                                                                                                                                     |
| `append`          | YAML     | [Append additional nodes](docs/advanced/append-nodes.md) to the builder                                                                                                                         |

> * ¹ `config` and `config-inline` are mutually exclusive

### outputs

Following outputs are available

| Name        | Type   | Description                                     |
|-------------|--------|-------------------------------------------------|
| `name`      | String | Builder name                                    |
| `driver`    | String | Builder driver                                  |
| `platforms` | String | Builder node platforms (preferred or available) |
| `nodes`     | JSON   | Builder [nodes metadata](#nodes-output)         |

### environment variables

The following [official docker environment variables](https://docs.docker.com/engine/reference/commandline/cli/#environment-variables) are supported:

| Name            | Type   | Default     | Description                                     |
|-----------------|--------|-------------|-------------------------------------------------|
| `DOCKER_CONFIG` | String | `~/.docker` | The location of your client configuration files |

## Notes

### `nodes` output

```json
[
  {
     "name": "builder-3820d274-502c-4498-ae24-d4c32b3023d90",
     "endpoint": "unix:///var/run/docker.sock",
     "driver-opts": [
       "network=host",
       "image=moby/buildkit:master"
     ],
    "status": "running",
    "buildkitd-flags": "--allow-insecure-entitlement security.insecure --allow-insecure-entitlement network.host",
    "buildkit": "3fab389",
    "platforms": "linux/amd64,linux/amd64/v2,linux/amd64/v3,linux/amd64/v4,linux/386"
  }
]
```

| Name              | Type   | Description                |
|-------------------|--------|----------------------------|
| `name`            | String | Node name                  |
| `endpoint`        | String | Node endpoint              |
| `driver-opts`     | List   | Options for the driver     |
| `status`          | String | Node status                |
| `buildkitd-flags` | String | Flags for buildkitd daemon |
| `buildkit`        | String | BuildKit version           |
| `platforms`       | String | Platforms available        |

### BuildKit container logs

See https://docs.docker.com/build/ci/github-actions/configure-builder/#buildkit-container-logs

## Using on GHES

GitHub Runners come [pre-installed with Docker Buildx](https://github.com/actions/runner-images/blob/main/images/linux/Ubuntu2204-Readme.md)
following your virtual environment. If you specify a version or `latest` of
Docker Buildx in your workflow, the version will be downloaded from [GitHub Releases in `docker/buildx`](https://github.com/docker/buildx/releases)
repository. These calls to `docker/buildx` are made via unauthenticated requests,
which are limited to [60 requests per hour per IP](https://docs.github.com/en/rest/overview/resources-in-the-rest-api#rate-limiting).

If more requests are made within the time frame, then you will start to see
rate-limit errors during downloading that looks like:

```
##[error]API rate limit exceeded for...
```

To get a higher rate limit, you can [generate a personal access token on github.com](https://github.com/settings/tokens/new)
and pass it as the `github_token` input for the action:

```yaml
uses: docker/setup-buildx-action@v3
with:
  github_token: ${{ secrets.GH_DOTCOM_TOKEN }}
  version: v0.10.1
```

If the runner is not able to access `github.com`, it will take the default one
available on the GitHub Runner or runner's tool cache. See "[Setting up the tool cache on self-hosted runners without internet access](https://docs.github.com/en/enterprise-server@3.2/admin/github-actions/managing-access-to-actions-from-githubcom/setting-up-the-tool-cache-on-self-hosted-runners-without-internet-access)"
for more information.

## Contributing

Want to contribute? Awesome! You can find information about contributing to
this project in the [CONTRIBUTING.md](/.github/CONTRIBUTING.md)
