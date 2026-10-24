# Pods and Compose

Two ways to run containers that belong
together — a web app and its database,
a service and its helper. **Pods** are
Podman's own grouping; **Compose**
describes a set of services in one file
and runs it with `podman compose`.

**Read this first:** pods are the one
major feature this guide's lab could
not run at all. Creating a pod asks
the kernel for a shared control-group
parent, and the lab's sandbox refuses
control-group delegation outright —
transcript t05 keeps the real error.
Everything in this chapter about pods
therefore follows Podman's official
documentation and is labelled
**documented only**. Compose fared
slightly better: its file format and
rules are covered here, but no compose
run was live-verified either, because
compose builds on the same container
operations the sandbox restricts. On a
normal machine both simply work.

## Pods: containers that share a room

A pod is a group of containers that
share one **network namespace** — one
IP address, one set of ports, one
localhost. It is borrowed from
Kubernetes (chapter 13), where the pod
is the smallest unit of deployment,
and it fits the "tightly coupled
helpers" shape:

```bash
podman pod create --name app -p 8080:80
podman run -d --pod app my-web-image
podman run -d --pod app my-cache-image
podman pod ps
podman pod stop app
podman pod start app
```

Inside the pod, the web container
reaches the cache at `localhost:<its
port>` — no DNS, no published port,
because they share the room. A tiny
**infra container** (a pause image)
exists inside every pod to hold the
shared namespaces open; `podman pod
inspect app` shows the whole family.

Honest guidance: most single-machine
setups do not need pods. Two containers
on one bridge network, talking by name
(chapter 8), achieve the same result
with fewer concepts. Reach for a pod
when the containers truly are one
application — started, stopped and
restarted together — or when you are
rehearsing for Kubernetes, where
`podman generate kube` can turn a pod
into YAML that a cluster accepts.

## Compose: the group as a file

Compose takes the other approach: one
YAML file describing every service,
its image, volumes, ports and
environment, kept with your project:

```yaml
services:
  web:
    image: docker.io/library/nginx:alpine
    ports:
      - "8080:80"
    volumes:
      - site:/usr/share/nginx/html:ro
  backup:
    image: docker.io/library/alpine:3.20
    volumes:
      - site:/data
    command: sleep infinity
volumes:
  site:
```

```bash
podman compose up -d     # build the set, detached
podman compose ps
podman compose down      # stop and remove the set
```

`podman compose` is a front-end: Podman
looks for an installed compose provider
— `podman-compose` (a Python tool,
installable from your distro or pip) or
Docker's compose binary talking to
Podman's service socket — and hands it
the file. The guide's lab did not run
either provider (see the note at the
top), so treat the compose half of
this chapter as a documented map, and
the transcripts' silence as exactly
what it is.

Compose's real value is not fewer
commands; it is that the whole setup
is a file you can put in git, diff,
and hand to someone else — the same
reason chapter 10 puts services in
files rather than in shell history.

## Which one?

| | Pods | Compose |
|---|---|---|
| Grouping idea | Shared network, one lifecycle | A file describing separate services |
| Network between members | `localhost` | By name, over a bridge |
| File to keep | No (commands) | Yes, `compose.yaml` |
| Kubernetes kinship | Direct (pods, `generate kube`) | None |
| Verified in this guide's lab | No — documented only | No — documented only |

Either way, the parts underneath are
the ones the lab did prove: images,
containers, volumes and networks
behave the same whether they were
started one by one or as a group.
