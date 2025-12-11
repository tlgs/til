+++
title = "How to write multi-mode Docker images"
description = "How to write basic multi-mode Docker images for maintainability and robustness"
tags = ["docker", "bash"]
+++

If you want to create a single Docker image that can do different things,
one useful pattern is the "multi-mode" image.
This practice removes the need to create two different images with
the same build steps that only differ in the running command.

The concept boils down to **identifying the different run modes
and mapping them to verbs**.
You then use a simple argument-parsing script as an entrypoint to do the right thing:

```bash
#!/usr/bin/env bash

case $1 in
  'server')
    echo "I'm a server! Here are my args:" "${@:2}"
    ;;
  'daemon')
    echo "I'm a daemon! Here are my args:" "${@:2}"
    ;;
  *)
    printf 'unknown verb %s\n' >&2
    exit 1
    ;;
esac
```

Instead of dummy printing you'll want to use [`exec [command [argument...]]`][5]
to ensure proper signal handling in containers.

The script is then set as the image's `ENTRYPOINT` like so:

```Dockerfile
# syntax=docker/dockerfile:1

FROM alpine:3.19
RUN apk add --no-cache bash

COPY docker-entrypoint /usr/local/bin
ENTRYPOINT ["docker-entrypoint"]
```

Creating and running a container with the above image
(e.g., `docker run multi-mode-example`) will result in the "unknown verb" error.
A default verb can be set with `CMD`.

Below is an example of how to use this image in a Compose stack:

```yaml
services:
  server:
    build: .
    command: ["server", "--port", "8080"]

  daemon:
    build: .
    command: ["daemon", "--debug"]
```

## Backstory

I came across this idea while following ["Deploying Dagster to Docker"][1] and peeking into
[their example on GitHub][2] which uses the same image for two different services.
They do so by declaring a different [`entrypoint`][3] on each service.
Trying to deeply understand `ENTRYPOINT` vs `CMD`, I landed on [a very good blog post][4]
by Michael Fischer which formally introduced me to the pattern of multi-mode images.


[1]: https://docs.dagster.io/deployment/guides/docker
[2]: https://github.com/dagster-io/dagster/tree/master/examples/deploy_docker
[3]: https://docs.docker.com/compose/compose-file/05-services/#entrypoint
[4]: https://aws.amazon.com/blogs/opensource/demystifying-entrypoint-cmd-docker/
[5]: https://www.man7.org/linux/man-pages/man1/exec.1p.html
