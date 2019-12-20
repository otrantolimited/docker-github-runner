# GitHub Runner

[![Docker Pulls](https://img.shields.io/docker/pulls/tcardonne/github-runner)](https://hub.docker.com/r/tcardonne/github-runner)

-----------
GitHub allows developpers to run GitHub Actions workflows on your own runners.
This Docker image allows you to create your own runners on Docker.

For now, there is only a Debian Buster image, but I may add more variants in the future. Feel free to create an issue if you want another base image.

## Important notes

GitHub [recommends](https://help.github.com/en/github/automating-your-workflow-with-github-actions/about-self-hosted-runners#self-hosted-runner-security-with-public-repositories) that you do **NOT** use self-hosted runners with public repositories, for security reasons.

## Usage

### Basic usage
Use the following command to start listening for jobs:
```shell
docker run -it --name my-runner \
    -e RUNNER_NAME=my-runner \
    -e RUNNER_TOKEN=token \
    -e RUNNER_REPOSITORY_URL=https://github.com/... \
    tcardonne/github-runner
```

### Using Docker inside your Actions

If you want to use Docker inside your runner (ie, build images in a workflow), you can enable Docker siblings by binding the host Docker daemon socket. Please keep in mind that doing this gives your actions full control on the Docker daemon.

```shell
docker run -it --name my-runner \
    -e RUNNER_NAME=my-runner \
    -e RUNNER_TOKEN=token \
    -e RUNNER_REPOSITORY_URL=https://github.com/... \
    -v /var/run/docker.sock:/var/run/docker.sock \
    tcardonne/github-runner
```

### Using docker-compose.yml

In `docker-compose.yml` :
```yaml
version: "3.7"

services:
    runner:
      image: tcardonne/github-runner:latest
      environment:
        RUNNER_NAME: "my-runner"
        RUNNER_REPOSITORY_URL: ${RUNNER_REPOSITORY_URL}
        RUNNER_TOKEN: ${RUNNER_TOKEN}
      volumes:
        - /var/run/docker.sock:/var/run/docker.sock
```

You can create a `.env` to provide environment variables when using docker-compose :
```
RUNNER_REPOSITORY_URL=https://github.com/your_url/your_repo
RUNNER_TOKEN=the_runner_token
```

## Environment variables

The following environment variables allows you to control the configuration parameters.

| Name | Description | Default value |
|------|---------------|-------------|
| RUNNER_REPOSITORY_URL | The runner will be linked to this repository URL | Required |
| RUNNER_TOKEN | Personal Access Token provided by GitHub | Required
| RUNNER_WORK_DIRECTORY | Runner's work directory | `"_work"`
| RUNNER_NAME | Name of the runner displayed in the GitHub UI | Hostname of the container
| RUNNER_REPLACE_EXISTING | `"true"` will replace existing runner with the same name, `"false"` will use a random name if there is conflict | `"true"`

## Runner auto-update behavior

The GitHub runner (the binary) will update itself when receiving a job, if a new release is available.
In order to allow the runner to exit and restart by itself, the binary is started by a supervisord process.