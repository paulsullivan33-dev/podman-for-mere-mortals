# Glossary

**aardvark-dns** — The small DNS server
Podman runs for a bridge network, so
containers can reach each other by name
(chapter 8).

**Bind mount** — A host folder placed
inside a container with `-v
/path:/inside` — no copy; the container
reads the folder itself (chapter 7).

**Bridge network** — Podman's default
private network for containers, run by
netavark; the outside world reaches in
only through published ports (chapter
8).

**Buildah** — The image-building engine
inside `podman build` (chapter 6).

**cgroups** (control groups) — The
kernel's mechanism for limiting and
accounting a group of processes' CPU,
memory and I/O; one of the fences
(chapter 1).

**Compose** — Describing several
containers in one YAML file and running
them as a set; driven by `podman
compose` (chapter 9).

**conmon** — The tiny monitor process
that watches one container, holds its
logs and records its exit (chapter 2).

**Container** — A process (or a few)
fenced into its own view of the
filesystem, processes and network,
sharing the host's kernel (chapter 1).

**Containerfile** — The recipe file
`podman build` follows, line by line,
to make an image; a `Dockerfile` is
the same thing under its older name
(chapter 6).

**crun** — The OCI runtime that
actually creates a container's fences
and starts its first process; runc is
its sibling (chapter 2).

**Daemon** — A long-running background
service. Docker has one; Podman
deliberately does not (chapter 1).

**Digest** — The SHA-256 hash that
identifies an image's exact content;
tags are labels, digests are identity
(chapter 5).

**Host networking** — `--network host`:
the container shares the host's network
outright (chapter 8).

**Image** — A packaged filesystem plus
instructions (start command, ports,
environment), built in shareable layers
(chapter 5).

**Layer** — One step of an image's
recipe, stored once and shared by every
image and container built on it
(chapters 5, 6).

**linger** — The systemd setting
(`loginctl enable-linger`) that keeps a
user's services running after logout
and starts them at boot (chapter 10).

**Namespace** — The kernel's fencing
system: separate views of processes,
network, mounts, users and more, one
set per container (chapter 1).

**netavark** — Podman's network setup
engine for bridge networks (chapter
8).

**OCI** — The Open Container Initiative:
the shared specifications for image and
runtime formats that let Podman and
Docker use the same images (chapter
2).

**Overlay** — The storage driver that
stacks image layers using the kernel's
overlay filesystem; the normal driver
on a normal machine (chapter 7).

**pasta / slirp4netns** — The programs
that carry network traffic for rootless
containers, through the user's account
(chapters 3, 8).

**Pod** — A group of containers sharing
one network namespace and lifecycle
(chapter 9).

**Publish** — Forwarding a host port
into a container: `-p 8080:80`
(chapter 8).

**Quadlet** — A short `.container`
(and friends) declaration file that
systemd, through Podman's generator,
turns into a service unit (chapter
10).

**Registry** — A server that stores
and hands out images: Docker Hub,
quay.io, ghcr.io, or your own (chapter
5).

**Rootful / rootless** — Running Podman
as root, or as your ordinary user with
the container's root mapped back to
you. Rootless is the recommended
default (chapters 1, 3).

**Store** — Podman's on-disk home for
images, container layers and volumes
(chapter 2).

**subuid / subgid** — The ranges of
user and group IDs granted to an
account (`/etc/subuid`), which rootless
containers map their accounts onto
(chapter 3).

**Tag** — A movable human label on an
image version: `:3.20`, `:latest`
(chapter 5).

**vfs** — The plain-directory storage
driver: no layering cleverness, each
layer a full copy; a fallback, and the
lab's necessity (chapters 3, 7).

**Volume** — A Podman-managed
directory, attached to a container,
that outlives it (chapter 7).
