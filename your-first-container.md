# Your First Container

Everything in this chapter was run in
the lab; transcripts t01 and t02 hold
the sessions exactly as they happened
(the two sandbox-shaped exceptions are
named where they occur).

## A command in a container

```bash
podman run --rm --network none \
  docker.io/library/alpine:3.20 \
  sh -c 'echo "hello from inside the container";
         cat /etc/alpine-release; hostname'
```

The lab's output:

```
hello from inside the container
3.20.10
f4d32eef1c7e
```

Read the command left to right: `run`
a container from the image
`docker.io/library/alpine:3.20` (if it is
not on disk, Podman pulls it first —
chapter 5), run the command after the
image name, then throw the container
away (`--rm`). The hostname is a random
container ID, not your machine's name:
that is the fence made visible. The
`--network none` gives the container no
network at all; chapter 8 covers the
choices, and for a quick command it is
the tidy default.

Check: the command's exit code is the
container's exit code. `podman run --rm
alpine false; echo $?` prints `1`.

## A server, detached

```bash
podman run -d --name web \
  docker.io/library/nginx:alpine
curl http://localhost:8080/
```

`-d` detaches: Podman prints the
container's full ID and returns your
prompt while the server keeps running
under conmon's eye (chapter 2). On a
normal machine this is the whole story:
the command above also **publishes**
the server's port with `-p 8080:80` —
add it before the image name:

```bash
podman run -d --name web -p 8080:80 \
  docker.io/library/nginx:alpine
```

The lab's sandbox forbids the bridge
networking publishing needs (its error
is transcript t90), so the lab ran its
server with `--network host` instead —
the container shares the host's network
outright and answers on port 80. The
result is the same lesson:

Check: the lab's checks, all real,

```
$ podman ps
CONTAINER ID  IMAGE                           NAMES
ef414f9ef8e6  docker.io/library/nginx:alpine  web
$ curl ... http://localhost/
GET http://localhost/ -> HTTP 200
```

## Looking at a running container

```bash
podman ps                # running containers
podman ps -a             # all, including stopped
podman logs web          # what it printed
podman inspect web       # everything, as JSON
```

`podman logs` reads what the container's
standard output and error received —
transcript t02 shows Nginx's start-up
lines. `inspect` is the firehose: one
JSON document with the image, the
network, the mounts, the state. You will
rarely read it whole; chapter 15 shows
picking single fields out of it with
`--format`.

## Going inside: exec

```bash
podman exec -it web sh
```

starts a shell *inside* the running
container — the same filesystem, the
same processes, from the inside. `exec`
is also how you run one-off commands
(`podman exec web nginx -t` tests its
config). **In this guide exec is
documented only:** the lab's sandbox
refuses the `setns` call exec is built
on, and transcript t02 keeps the exact
error (`crun: setns 'mnt': Operation not
permitted`). On your machine it simply
works; it is the same call that copy
— `podman cp web:/etc/nginx/nginx.conf
.` — also needs, so `cp` carries the
same label in this guide.

## The lifecycle

```bash
podman stop web      # ask nicely (SIGTERM), then force
podman start web     # run a stopped container again
podman restart web   # stop + start
podman rm web        # delete a stopped container
podman rm -f web     # delete even if running
```

A stopped container still exists: its
filesystem layer, its name, its logs.
`rm` is what deletes — and deleting is
what makes chapter 7 (volumes) matter,
because everything in the container's
own layer goes with it. Names are
unique, so a forgotten stopped container
explains most "that name is already in
use" errors (chapter 15). The lab's
transcript t07 ends with `stop`, `rm`,
and a `podman system df` showing the
space handed back.
