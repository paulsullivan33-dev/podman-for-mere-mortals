# The Neighbours

Containers are an idea bigger than any
one tool. These are the tools you will
hear about alongside Podman, and what
each one trades.

## Docker

The tool that made containers ordinary.
Same images, same Containerfile under
its other name (Dockerfile), a command
line Podman deliberately mirrors. The
differences that matter:

| | Docker | Podman |
|---|---|---|
| Engine | A root daemon | No daemon; your process |
| Default mode | Rootful | Rootless |
| Services | Compose, swarm mode | Quadlet + systemd |
| Grouping | Compose stacks | Pods, compose, Quadlet |

Neither is "the real one" — both run
OCI images through an OCI runtime
(chapter 2). Choose Docker for its
enormous mindshare and desktop apps;
choose Podman for the daemonless,
rootless shape and systemd fit. Mixed
households are normal; images do not
care.

## nerdctl and containerd

**containerd** is the quiet engine
inside Docker and most Kubernetes
clusters; **nerdctl** is its
Docker-shaped command line. You meet
them when you work on systems that
already run containerd. Podman's
equivalent layer is its own; the
images, again, are shared.

## Kubernetes

A cluster orchestrator: many machines,
one API, containers (grouped as
**pods** — the word chapter 9 borrows)
scheduled, restarted and networked as
a fleet. Kubernetes is the answer to
"fifty machines", not to "my one
server feels untidy". Podman meets it
at two doors: `podman generate kube`
writes a pod out as Kubernetes YAML,
and `podman play kube` runs such YAML
on one machine. Learn Podman first;
everything in chapters 5–9 transfers.

## LXC and Incus

**LXC** containers are system
containers: a whole Linux userland,
init and all, fenced lightly — closer
to a very cheap virtual machine than
to Podman's one-program application
containers. **Incus** is the friendly
manager over LXC (and over real VMs).
When the thing you want to fence is a
*system* you administer — users, cron,
services inside — reach for Incus;
when it is a *program* you run, reach
for Podman. Households often end up
with both.

## Virtual machines

A VM fences with a second kernel and
emulated hardware: heavier (minutes to
boot, gigabytes of overhead) and
stronger (a Windows guest on a Linux
host is possible; a kernel exploit in
the guest is the guest's problem).
Podman's own answer on Windows and
macOS — `podman machine` — is exactly
this: a managed Linux VM, because
containers need a Linux kernel and
those systems do not have one to
share. Rule of thumb: VM when the
*operating system* must differ or the
fence must be a wall; container when
you are packaging a program.

## The Podman socket, briefly

Some tools expect to *talk* to a
container engine (Docker's socket is
famous). Podman offers a compatible
API service on request —
`systemctl --user start podman.socket`
rootless — which is how `podman
compose`'s Docker-flavoured provider
and various dashboards connect. It is
off until you turn it on, it carries
the full power of your Podman, and
that is the complete case for leaving
it off until a named tool needs it.
