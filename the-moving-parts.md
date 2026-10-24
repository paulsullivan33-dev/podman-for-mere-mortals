# The Moving Parts

Five pieces do the work. Knowing which
one holds what turns most later confusion
into a named problem.

| Part | What it is | What it holds |
|---|---|---|
| **podman** | The command you type | Your requests; nothing at rest |
| **conmon** | A tiny monitor per container | The container's exit, its log stream |
| **crun** (the OCI runtime) | The program that starts containers | Nothing; it sets up fences and execs |
| **containers/storage** | The on-disk store | Images, container filesystems, volumes |
| **A registry** | A server of images (Docker Hub, quay.io) | The images you pull, until you pull them |

## The command

`podman` is a normal program. It reads
your request, arranges things through
the other parts, prints the answer, and
exits. There is no daemon to be "down",
which is why so many Docker-era errors
("cannot connect to the docker daemon")
have no Podman equivalent. Podman *can*
run a service socket for tools that
expect one — chapter 13 mentions it —
but nothing in this guide needs it.

Check: `podman --version` answers from
the binary alone. In the lab:

```
$ podman --version
podman version 4.9.3
```

(Transcript t00.)

## conmon: one small watcher

When a container starts detached,
something has to hold its output, notice
when it exits, and keep its exit code.
That is **conmon**, the container
monitor: one small process per container,
started by Podman, which then outlives
the `podman` command that launched it.
When you read logs with `podman logs`,
you are reading what conmon captured.

## crun: the runtime

The **OCI runtime** — on this guide's
systems, **crun** — is the program that
actually creates the fences: it asks the
kernel for the namespaces, mounts the
container's filesystem, applies the
limits, and executes the container's
first process. Podman and Docker both
sit above a runtime like this (Docker's
is **runc**); the image format and the
specification they share are why images
work in both. You will never run crun by
hand, but its name appears in error
messages, which is why it is worth
knowing: an error beginning `crun:`
means the fences themselves failed to go
up — chapter 15 reads several.

## The store

Images, containers' filesystems and
volumes live in one on-disk store:

- **Rootful** (running as root):
  `/var/lib/containers/storage`
- **Rootless** (running as you):
  `~/.local/share/containers/storage`

Images are stored as stacked **layers**
(chapter 5), each container gets a thin
writable layer on top, and named volumes
sit beside them (chapter 7). The store
is managed, not browsed: you look at it
through `podman images`, `podman volume
ls` and `podman system df`, not with
`ls`, and you never delete files from it
by hand.

Check: `podman info` names your store
and driver. In the lab — which ran
rootful, with the store moved aside for
reasons chapter 3 declares:

```
$ podman info --format '...'
runtime=crun store=vfs graphroot=/home/hatch/workspace/podman-lab/vfsstore network=netavark rootless=false cgroups=v2
```

(Transcript t00. Your machine will show
`store=overlay` and, if you followed the
rootless path, `rootless=true`.)

## The registry

A **registry** is a file server for
images: you `pull` an image from one and
`push` your own to one. Docker Hub
(`docker.io`) is the biggest public one;
`quay.io` is Red Hat's, home of Podman's
own project images; big cloud providers
run their own. Podman talks to all of
them in the same format. Chapter 5
covers names, tags, and looking before
you run.

## When each part stops

- **podman exits** — nothing happens to
  running containers. That is the no-
  daemon point, and it feels wrong the
  first time you `Ctrl-C` a `run`.
- **conmon dies** — you lose that
  container's watcher: logs stop, its
  exit goes unrecorded. Rare, and the
  container process itself often keeps
  running, orphaned.
- **The store's disk fills** — pulls and
  starts fail with space errors; chapter
  15 has the check (`podman system df`).
- **The registry is unreachable** —
  pulls fail, but everything already on
  your disk keeps working. Your machine
  owns what it has pulled.
