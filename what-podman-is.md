# What Podman Is

Podman runs **containers**. A container is
not a separate computer and not a separate
operating system. It is an ordinary Linux
process — a web server, a database, a
shell — that the kernel has been told to
see the world through fences:

- It sees only its own filesystem tree,
  assembled from an **image** (chapter 5).
- It sees only its own process list, in
  which it is process number 1.
- It gets its own network corner, or
  shares the host's, as you decide
  (chapter 8).
- It can be limited in memory and CPU.

Everything else — the kernel itself, the
hardware, the machine's clock — is shared
with the host. That sharing is the whole
trade: containers start in a blink and
weigh almost nothing, but they are only
ever as separated as the kernel's fences
make them. Chapter 14 is honest about
where the fences end.

## The part Podman is known for

Two design decisions set Podman apart
from Docker, the older tool it resembles.

**No daemon.** Docker's work is done by a
long-running background service, the
Docker daemon, which your commands ask to
act for you — and which therefore runs
with root rights and owns everything.
Podman has no equivalent. When you type
`podman run`, the `podman` process does
the work directly and then steps aside,
leaving a tiny monitor (conmon, chapter 2)
to watch each container. Fewer moving
privileges, nothing to restart, and
nothing between you and the process tree:
a container is a child of your session,
like any other program.

**Rootless by default.** On a normal
install you run Podman as your ordinary
user. Your containers' "root" is mapped
to your unprivileged account outside, so
a program that breaks out of its
container lands as *you*, not as the
machine's root. Chapter 3 covers the
setup. The guide's lab could not do this
— its sandbox forbids the user-namespace
plumbing rootless mode stands on — so the
lab ran as root and says so wherever it
matters. Rootless is the recommended way;
the commands are the same either way.

## A familiar command line

Podman's commands deliberately mirror
the ones Docker made familiar:

```bash
podman pull docker.io/library/alpine:3.20
podman run --rm docker.io/library/alpine:3.20 echo hello
podman ps
podman images
```

If you have used Docker, most of your
muscle memory transfers; chapter 13
covers the differences that matter. If
you have not, nothing in this guide
assumes you have.

Check: after installing (chapter 3), the
first command above downloads a small
Linux image, and the second runs a
container that prints `hello` and exits.
The lab's transcripts of both are in
`podman-lab/transcripts/` (t01, t02).

## Who it is for

Podman suits anyone who wants to run
packaged software on a Linux machine they
control without handing a root daemon the
keys: the home-lab keeper who wants a
wiki or a media server in a box they can
delete cleanly; the developer who wants
to try a database without installing it;
the small team running a handful of
services on one server. The common thread
is one machine, honestly run — this guide
never leaves that ground.

## What it is not

A container is not a virtual machine:
there is no second kernel, no emulated
hardware, and the isolation is fencing,
not a wall (chapter 14). An image is not
an installer: nothing runs at "install"
time, and an image from a stranger is
code from a stranger (chapter 5). Podman
is not a cluster tool: it runs containers
on the machine you are standing on.
Running fifty machines as one is
Kubernetes' job, and chapter 13 says when
to go there instead.
