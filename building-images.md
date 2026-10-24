# Building Images

Chapter 5's images came from somebody
else's recipe. A **Containerfile** is
yours: a short, plain-text recipe that
Podman's builder (**Buildah**, built
into the `podman build` command) follows
one line at a time, each line becoming
one layer of the new image. (A file named
`Dockerfile` works identically; the
name Containerfile is Podman's own.)

## The lab's greeter, whole

The build context lives in this repo at
`podman-lab/build/`. Two files:

```dockerfile
# Containerfile
FROM docker.io/library/alpine:3.20
COPY greet.sh /usr/local/bin/greet
RUN chmod +x /usr/local/bin/greet
LABEL description="A tiny example image for the Podman guide"
CMD ["/usr/local/bin/greet"]
```

```sh
# greet.sh
#!/bin/sh
echo "hello from my own image"
echo "running on Alpine $(cat /etc/alpine-release)"
```

Line by line:

- **FROM** — every image starts from
  another image (or from `scratch`,
  the empty floor). Alpine again: small,
  present, and already on disk.
- **COPY** — take a file from the build
  **context** (the folder you point the
  build at) and place it in the image.
- **RUN** — run a command *at build
  time*, in a throwaway container, and
  keep the filesystem changes as a layer.
  Here it just makes the script
  executable. A RUN never runs again —
  which is why installing packages
  belongs here, not in CMD.
- **LABEL** — metadata on the image's
  card. Harmless, occasionally handy.
- **CMD** — the command the image runs
  when nobody says otherwise (chapter 4's
  `podman run image` runs this).

## Building and running

```bash
cd podman-lab/build
podman build -t localhost/mere-greeter:latest .
podman run --rm localhost/mere-greeter:latest
```

The trailing `.` is the context folder.
`-t` names the result; the `localhost/`
namespace marks it as home-made rather
than fetched. The lab's build and run
(transcript t04):

```
hello from my own image
running on Alpine 3.20.10
```

Check: `podman history localhost/
mere-greeter:latest` lists the layers
your recipe made — FROM's layers first,
then one line per instruction that
changed a file. It is the quickest way
to see what your build actually did,
and (chapter 5) to read how a stranger's
image was made.

## The cache, and honest rebuilds

Build twice and the second build is
nearly instant: each instruction's layer
is reused while the instruction and
everything it depends on are unchanged.
Change `greet.sh` and only the COPY
layer and its children rebuild. The
cache is a friend with one trap — a
`RUN apt-get update`-style line can be
served stale results from cache long
after the world moved on. `podman build
--no-cache` forces the honest, slower
rebuild when it matters.

## Habits that keep images small and sane

- **One job per image.** The greeter
  does one thing; a real image should
  be nearly as single-minded.
- **Order lines from calm to busy.**
  Instructions that rarely change
  (FROM, package installs) go first, so
  the cache survives edits to the files
  that change daily.
- **Don't COPY the world.** The context
  is everything in the folder, including
  the junk; a `.containerignore` file
  (same idea as `.gitignore`) keeps
  backups and `.git` out of your layers.
- **Never bake secrets.** Anything in
  a layer is in the image forever, for
  anyone who gets the image. Passwords
  belong in the run, not the build
  (chapters 7 and 12 show mounts and
  environment for that).

What builds *in* a RUN step needs the
network sometimes — the lab's build ran
with `--network none` because nothing in
the recipe downloads. On your machine
the default is fine.
