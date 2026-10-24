# Backup and Migration

Three different things get called
"backing up a container", and they are
not interchangeable:

| You want | The tool | What you get |
|---|---|---|
| The image, portable | `podman save` / `load` | A tar file any Podman can load |
| A snapshot of a container now | `podman commit` | A new image from a live container |
| The *data* | Volume backup (below) | Your actual files |

The third row is the one that matters
most and the one people skip: images
can be re-pulled or rebuilt, but the
volume holds the only copy of your
data. Everything in transcript t07
was run in the lab.

## save and load: images as files

```bash
podman save -o web-snapshot.tar \
  localhost/web-snapshot:latest
podman load -i web-snapshot.tar
```

`save` writes the image — every layer
plus its card of instructions — into
one tar file. `load` reads it back on
any machine with Podman, no registry,
no network. The lab's round trip
(transcript t07): the Nginx-based
image saved to a 64 MB tar, was
deleted with `podman rmi`, then loaded
back and listed again by `podman
images`. A USB stick is a perfectly
good registry for a house with two
computers; chapter 12 has the recipe.

Check: after `load`, `podman images`
shows the image with its name and tag
intact, and an image saved by digest-
pinned content loads byte-identical —
the layers carry their hashes with
them, and `load` verifies as it goes.

## commit: photographing a container

```bash
podman commit web localhost/web-snapshot:latest
```

`commit` freezes a *container's*
current filesystem layer into a new
image. It is how the lab made the
snapshot above. Use it knowingly: a
commit captures whatever state —
including the junk, the half-finished
edit, and anything secret that was
typed into the container — and it
says nothing about how the state was
reached. A Containerfile (chapter 6)
is a recipe anyone can re-run; a
commit is a photograph. Photographs
are for emergencies and archaeology,
not for publishing.

## The volume is the backup

A named volume is a directory; back it
up as one, through a throwaway
container so Podman manages the
permissions (the documented pattern,
and the one this guide recommends):

```bash
podman run --rm -v webdata:/data:ro \
  -v "$PWD":/backup \
  docker.io/library/alpine:3.20 \
  tar -czf /backup/webdata-backup.tar.gz -C /data .
```

and restore by reversing the direction
into a fresh volume. **Labelled
honestly:** the lab verified volumes,
reads, writes and bind mounts end to
end (transcript t03), but did not
rehearse a full restore — the tar
commands are the standard documented
shape, and your first restore should
be a rehearsal on a volume you can
afford to lose, done *before* the day
you need it for real. Chapter 10 of
the BookStack guide in this series
makes the same point about databases;
it is the series' oldest sermon
because it keeps being true.

## Migrating a whole setup

Moving house, machine to machine:

1. Images: `podman save` them, or just
   re-pull and re-build on the new
   machine — usually less data than the
   volumes.
2. Volumes: tar each one as above,
   restore on arrival.
3. The declarations: your Quadlet
   files, Containerfiles and compose
   files — which is why chapters 9 and
   10 put the setup *in files*. A setup
   that lives in files migrates by
   copying a folder; a setup that lives
   in your shell history migrates by
   archaeology.
