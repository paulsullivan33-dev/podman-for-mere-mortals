# Images and Registries

An **image** is a packaged filesystem
plus a card of instructions: which
program to start, which ports it means
to use, which environment it expects.
It is built in **layers** — one per
step of its recipe (chapter 6) — and
layers are shared: ten containers from
one image cost one image's disk, and
two images built on Alpine share the
Alpine layer between them.

## Names are addresses

An image's full name is an address:

```
docker.io/library/alpine:3.20
└──┬───┘ └────┬────┘ └─┬─┘
registry  namespace   tag
```

- The **registry** is the server. Leave
  it off and Podman assumes `docker.io`;
  that is why `podman pull alpine` works.
  This guide writes the full form,
  because guessing servers is how people
  pull the wrong thing.
- The **namespace** is the owner:
  `library/` is Docker Hub's word for
  "official images"; `docker.io/redhat/
  ubi9` is Red Hat's base image.
- The **tag** is a name for a version —
  `:3.20`, `:latest`, `:1.27-alpine`.
  Tags are movable labels, not promises:
  `latest` today is not `latest` next
  month, which is why transcripts in
  this guide pin `alpine:3.20`.

Under the tag sits the **digest**, a
SHA-256 hash of the image's content.
Pulling by digest —
`alpine@sha256:<digest>` — fetches
exactly those bytes or fails; `podman
images --digests` shows the digest of
everything you hold. Tags are for
humans, digests are for certainty.

## Pulling and looking

```bash
podman pull docker.io/library/alpine:3.20
podman images
podman image inspect docker.io/library/alpine:3.20
podman system df
```

Check: the lab's `podman images` after
pulling Alpine and Nginx (transcript
t01):

```
REPOSITORY                TAG         IMAGE ID      SIZE
docker.io/library/nginx   alpine      3d6cab9c2941  64.4 MB
docker.io/library/alpine  3.20        bf8527eb54c3  8.1 MB
```

Small is not a gimmick — Alpine is a
complete little Linux userland in 8 MB,
and it is the base this guide builds on
in chapter 6. `podman system df` sums
the whole store: images, containers,
volumes (chapter 7).

## Registries worth knowing

- **docker.io** (Docker Hub) — the
  biggest public collection, including
  the `library/` official images,
  curated and rebuilt by Docker's team.
- **quay.io** — Red Hat's registry;
  Podman's own and many Fedora-world
  images live here.
- **ghcr.io** — GitHub's, where projects
  that live on GitHub often publish.
- **Your own** — `podman run` a
  `registry` container, or just move
  images as files (chapter 11).

Podman's `/etc/containers/registries.
conf` can add search registries and
mirrors; the defaults suffice here.

## Look before you run

An image is code from whoever built
it, and a container runs it with
whatever access you give it. Habits
that cost nothing:

- Prefer official (`library/`) or the
  project's own namespace over a
  stranger's re-upload.
- Pin a real tag, and the digest when
  it matters; avoid living on `latest`.
- Read the image before running it:
  `podman image inspect` shows the
  start command, exposed ports and
  labels; `podman history <image>`
  shows the recipe it was built from,
  layer by layer (chapter 6 runs it on
  the lab's own image, transcript t04).
- Remember what rootless buys you
  (chapter 1): a bad image run rootless
  can reach what *you* can reach. Run
  rootful, it reaches further.

None of this makes images safe; it
makes your choice an informed one.
Chapter 14 returns to the point.
