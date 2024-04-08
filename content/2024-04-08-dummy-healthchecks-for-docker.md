+++
title = "Dummy healthcheck test for Docker services"
description = "Control Docker service startup order with a dummy healthcheck test"
tags = ["docker"]
+++

If you need to [control Docker service startup order in Compose][1],
and your dependency service does not have a standard way of querying its
status (e.g., [pg_isready][2] for PostgreSQL) or speak HTTP
(so the common `curl -f http://localhost` test doesn't work),
you can use a dummy healthcheck test:

```yaml
healthcheck:
  test: ["CMD", "/bin/sh", "-c", "exit 0"]
```

Because this approach always tags containers as being healthy,
this approach only delays service startup by a set amount of time.

It should be pretty obvious that **this should only be used as a last resort,**
and that you should try to come up with an useful health check for the service.
Also, don't set this up as the HEALTHCHECK in a Dockerfile -
only use it in Compose stacks where the intent of service dependency is clear.

Below is a fully reproducible Docker Compose stack
that results in service A starting 10 seconds after service B.
See the [Dockerfile HEALTHCHECK reference][3] for information on using
`start_period` and `start_interval` options to tune behavior.

```yaml
services:
  a:
    image: alpine:3.19
    command: sleep 30
    depends_on:
      b:
        condition: service_healthy

  b:
    image: alpine:3.19
    command: sleep 30
    healthcheck:
      test: ["CMD", "/bin/sh", "-c", "exit 0"]
      start_period: 1m
      start_interval: 10s
```

## Backstory

I recently needed to make sure that a service would only start after
a gRPC server was ready to receive requests.
I don't know much about gRPC, but quickly found that I could implement a
simple healthcheck with [grpcurl][4].

Unfortunately, grpcurl only works if: (1) the queried server supports
server reflection, or (2) you supply source or compiled "protocol files".
Neither of these requirements could be satisfied, so I had to resort to a dumber solution.


[1]: https://docs.docker.com/compose/startup-order/
[2]: https://www.postgresql.org/docs/current/app-pg-isready.html
[3]: https://docs.docker.com/reference/dockerfile/#healthcheck
[4]: https://github.com/fullstorydev/grpcurl
