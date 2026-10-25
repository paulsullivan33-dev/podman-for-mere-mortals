# Podman for Mere Mortals

A container is a way of running a program
with its own little world around it: its
own files, its own view of the network,
its own process list — while sharing the
one Linux kernel underneath with
everything else on the machine. **Podman**
is the open-source tool that runs
containers this way *without a daemon*:
there is no background service that owns
your containers, no root-privileged
process sitting in the middle. The
`podman` command you type does the work
itself, and on most systems it does it as
your ordinary user, with no root rights at
all. This guide explains how containers
work in those terms, how to install Podman
and run your first containers, how to
build images of your own, keep data in
volumes, group containers into pods, and
have containers start at boot like proper
services — with the limits stated plainly,
including the ones the lab ran into.

The lab behind this guide ran the real
thing: **Podman 4.9.3**, the version in
Ubuntu 24.04's repositories, installed
with apt onto a small Linux machine. Onto
the disk went two small public images
(Alpine Linux and the Nginx web server),
and the system was exercised end to end:
images pulled and inspected, containers
run in the foreground and detached, a web
server answering real HTTP requests, logs
read, a named volume written by one
container and read back by another after
the first was deleted, a bind mount shown
read-only, an image built from a
Containerfile written for the guide and
run, a running container committed to a
new image, and that image saved to a tar
file, deleted, and loaded back. systemd
unit files were generated two ways — the
older `podman generate systemd` and a
modern Quadlet file translated by
Podman's own generator.

What the lab could not do, it did not
fake, and the list is not short. The lab
machine was a restricted sandbox: it
refuses the system calls that let one
process step inside another's namespaces,
and it refuses to delegate its control
groups. So containers there ran as root
with the simple `vfs` storage driver and
host networking, and four everyday things
are taught from the documentation and
labelled as such wherever they appear:
**rootless mode** (the way most people
should run Podman, and the way chapter 3
teaches it), **bridge networking and
published ports**, **`podman exec`** (and
its cousin `podman cp`), and **pods** —
the pod command was run and its error is
in the transcripts. Actually *starting* a
generated systemd unit is likewise
documented only: the unit text is real,
the boot is not. `criteria.md` keeps the
full ledger of what was run and what was
only read.

## What this guide covers

- [**What Podman is**](what-podman-is.md) —
  the mental model: containers as
  processes with fences around them, why
  "no daemon" matters, and what a
  container is not (a virtual machine, a
  package manager, a security guarantee)
- [**The moving parts**](the-moving-parts.md) —
  the podman command, the runtime (crun),
  the storage, the registry, and conmon:
  what each holds, and what breaks when
  each stops
- [**Installing Podman**](installing-podman.md) —
  the package install, the rootless setup
  most people should use, the extra
  pieces (`uidmap`, `fuse-overlayfs`,
  `slirp4netns`), and the sandbox the lab
  actually used, told in full
- [**Your first container**](your-first-container.md) —
  running a command in a container, a web
  server detached, `ps`, `logs`, `exec`,
  and the lifecycle: stop, start, remove
- [**Images and registries**](images-and-registries.md) —
  where images come from, names and tags,
  digests, layers, and looking before you
  run something a stranger built
- [**Building images**](building-images.md) —
  the Containerfile, line by line; the
  build cache; tags; and the tiny greeter
  image the lab built and ran
- [**Storage and volumes**](storage-and-volumes.md) —
  why a container's own disk vanishes,
  named volumes that outlive containers,
  bind mounts that share a host folder,
  and which to use when
- [**Networking**](networking.md) —
  the three honest choices: no network,
  the host's network, and the default
  bridge with published ports; names,
  DNS between containers, and checking
  what is actually listening
- [**Pods and Compose**](pods-and-compose.md) —
  containers that share a network and
  travel together, and the compose file
  for running several services from one
  description
- [**Running as services**](running-as-services.md) —
  containers under systemd: Quadlet files
  (the modern way), `generate systemd`
  (the older way), restarting, and logs
  through the journal
- [**Backup and migration**](backup-and-migration.md) —
  `commit`, `save` and `load`, moving an
  image between machines on a USB stick,
  and why the volume is the part that
  matters
- [**Recipes**](recipes.md) — a web server
  that starts at boot, a throwaway test
  box, a scheduled job in a container, and
  trying software without installing it
- [**The neighbours**](the-neighbours.md) —
  Docker, nerdctl, Kubernetes, LXC and
  plain virtual machines: what each one
  trades, and why the commands look so
  alike
- [**Limits**](limits.md) — one kernel
  shared by everything, rootless
  networking's rough edges, no Windows or
  Mac containers on Linux, stated plainly
- [**Troubleshooting**](troubleshooting.md) —
  "permission denied" in four different
  costumes, the image that will not pull,
  the port that is not listening, and the
  traps this guide's lab fell into so you
  don't have to
- [**Glossary**](glossary.md) — the terms,
  collected in one place

## What it doesn't cover

Running containers for a whole company —
orchestration across many machines
(Kubernetes and friends) is a subject of
its own, and this guide stays on one
machine, where a household or a small
team actually lives. Building images in
CI pipelines, signing images and supply
chain policy, and running Podman on
Windows or macOS (where it works through
a Linux virtual machine it manages
for you) are likewise pointed at, not
taught. Exposing any server to the
internet safely gets a warning, not a
tutorial.

## How to use this guide

Read chapter 1 for the model and chapter 2
before installing anything — knowing which
part holds what turns later confusion into
a named problem. Chapter 3 gets Podman
installed; chapter 4 runs your first
containers in the same sitting, because
the ideas are easiest while the install is
fresh. Chapter 7 (volumes) is not the
optional chapter: the first time a
container takes your data with it when it
is deleted, you will wish you had read it
before, not after. Chapter 11 is the other
one not to skip — save an image and back
up a volume while everything is small
enough that a rehearsal costs minutes.
The lab's transcripts live in
`podman-lab/transcripts/`; its notes are
`examples/lab-notes.txt`.

---

## Part of the series

This guide is one of the [For Mere Mortals](https://paulsullivan33-dev.github.io/mere-mortals/) guides: plain-language guides to the tools programmers and makers use.

Related guides:

- [Docker for Mere Mortals](https://paulsullivan33-dev.github.io/docker-for-mere-mortals/)
- [Docker Compose for Mere Mortals](https://paulsullivan33-dev.github.io/docker-compose-for-mere-mortals/)
- [systemd for Mere Mortals](https://paulsullivan33-dev.github.io/systemd-for-mere-mortals/)

## License

The writing in this guide is licensed under [CC BY 4.0](https://github.com/paulsullivan33-dev/podman-for-mere-mortals/blob/main/LICENSE). Code examples and lab scripts are licensed under the [MIT License](https://github.com/paulsullivan33-dev/podman-for-mere-mortals/blob/main/LICENSE-CODE).
