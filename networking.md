# Networking

A container gets one of three network
shapes. Knowing which one you picked —
and what each costs — prevents nearly
every "it won't connect" you'll meet.

| Shape | Flag | The container's network |
|---|---|---|
| None | `--network none` | A loopback and nothing else |
| Host | `--network host` | The host's, exactly, ports and all |
| Bridge (default) | `-p 8080:80` | A private network; only published ports cross |

## None: the honest default for commands

A batch job that reads a folder and
writes a folder needs no network, and
giving it none is a security decision
you can make in four words. Most of
this guide's one-shot examples use it.
Loopback still exists inside, so a
program can talk to itself.

## Host: no fence at the network

With `--network host` the container
shares the host's network interfaces.
Its port 80 is the machine's port 80 —
nothing to publish, nothing translated.
The lab's web server ran this way
(chapter 3 declared why) and answered
at `http://localhost/` with a plain
`HTTP 200` (transcript t02).

The costs are real: two containers
cannot both want port 80, a container
can listen on anything it likes, and
"netstat shows it" means the whole
network sees it too. Host networking
is for infrastructure you trust and
single-purpose boxes, not for trying
strangers' images.

## Bridge: the default, and publishing

Left alone, `podman run` puts the
container on a private **bridge**
network (Podman 4's engine for this is
**netavark**), with an address like
10.88.0.5 that only this host and its
containers can see. To let the outside
in, you **publish** a port:

```bash
podman run -d --name web -p 8080:80 \
  docker.io/library/nginx:alpine
```

reads as: the host's port 8080 forwards
to the container's port 80. Check:
`podman ps` shows the mapping in its
PORTS column, and `podman port web`
prints it on its own.

**Documented only in this guide:** the
lab's sandbox refuses the `setns` call
netavark needs to set up a bridge, so
publishing was never live-verified —
transcript t90 keeps the real error
(`netavark: setns: IO error...`). The
commands above follow Podman's own
documentation, and on a normal machine
they are the ordinary, daily way to run
a server in a container.

Two rootless wrinkles worth knowing
before they surprise you (also from the
documentation):

- A rootless container cannot publish a
  port below 1024 — those belong to
  root. Publish `-p 8080:80`, not
  `-p 80:80`.
- Rootless networking is provided by
  **slirp4netns** (Podman 4) or **pasta**
  (Podman 5), which carry the traffic
  through your user account. That is
  why a rootless server is reachable
  exactly as far as the host's firewall
  allows for you.

## Containers talking to each other

On the default bridge, containers can
reach each other by **name**: Podman
runs a small DNS for the network (the
`aardvark-dns` companion), so a second
container can `ping web` or fetch
`http://web/`. This is the quiet fact
that makes multi-container setups
(chapter 9) pleasant: names instead of
addresses that change.

Containers on *different* custom
networks cannot see each other at all;
`podman network create`, `ls` and
`inspect` manage them. One host, one
or two networks covers most honest
setups; a network per project is the
usual tidy habit.

## Checking what is actually listening

```bash
podman ps                       # PORTS column
podman port web                 # this container's mappings
ss -tlnp | grep 8080            # the host's view
curl -v http://localhost:8080/  # the truth, end to end
```

When a connection fails, walk the
list in order — is the container up,
is the port published, is anything
listening, does the end-to-end fetch
work — before changing anything.
Chapter 15's networking entry follows
the same walk.
