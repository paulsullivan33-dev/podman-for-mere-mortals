# Storage and Volumes

A container's own filesystem is a thin
writable layer over its image — and it
is deleted with the container. That is
the right behaviour for the program, and
a disaster for the program's *data* the
first time you `podman rm` the container
holding your only copy. This chapter is
the fix, and it is the chapter to read
before you run anything you care about.

## The three places data can live

| Place | Survives `podman rm`? | Use it for |
|---|---|---|
| The container's layer | No | The program, its scratch space |
| A **named volume** | Yes | Data the container owns: databases, uploads |
| A **bind mount** | Yes (it is the host's folder) | Data you own: configs, files you edit by hand |

## Named volumes

A volume is a managed directory in
Podman's store, attached to a container
at a path you choose:

```bash
podman volume create webdata
podman run --rm -v webdata:/data \
  docker.io/library/alpine:3.20 \
  sh -c 'echo "written by a container that
         is about to be deleted" > /data/note.txt'
podman run --rm -v webdata:/data \
  docker.io/library/alpine:3.20 \
  cat /data/note.txt
```

The lab ran exactly this (transcript
t03): the first container was deleted
the moment it finished (`--rm`), and
the second, brand-new container read
the note back:

```
written by a container that is about to be deleted
```

Check: `podman volume ls` lists the
volume even with no containers at all,
and `podman volume inspect webdata`
shows where it lives on disk (the
Mountpoint) — inside the store, which
is why you manage volumes with podman
commands and back them up the chapter
11 way, rather than poking the folder.

The `-v name:/path` form creates the
volume automatically if it is missing,
which is convenient and occasionally
surprising: a typo creates a fresh,
empty volume instead of erroring.
Creating volumes deliberately, as
above, is the safer habit.

## Bind mounts

```bash
podman run --rm \
  -v /tmp/hostdir-demo:/host:ro \
  docker.io/library/alpine:3.20 \
  cat /host/host-note.txt
```

A bind mount puts a real host folder
inside the container. No copy is made;
the container reads (and, without
`:ro`, *writes*) the folder itself.
The lab read a host file through a
read-only mount (transcript t03).

Bind mounts are the right shape for
things that are *yours* rather than
the container's: a config file you edit
with your normal editor, a folder of
media a server should serve, the
output directory of a batch job. Named
volumes are the right shape for things
that are the container's: a database's
data directory, an app's uploads.

One warning that belongs here, not
buried: a bind mount is a hole in the
fence exactly the size of the folder.
`-v /:/host` hands the container your
whole disk; people do it for a reason
in chapter 12's recipes, at read-only,
with their eyes open. Rootless podman
maps the ownership sensibly (the files
appear as yours on both sides);
rootful containers see and write real
root-owned files — chapter 14 again.

## Drivers, briefly

Underneath, the store stacks image
layers with a **storage driver**:
`overlay` (the kernel's overlay
filesystem) on any normal machine.
Chapter 3 declared the lab's exception:
its sandbox refuses overlay mounts, so
the lab used **vfs**, which stores each
layer as a plain directory copy —
slower and fatter, fine for a lab, and
the reason the lab's disk numbers in
transcript t01 are not yours to
expect. If `podman info` says vfs on
a machine you own, something chose it
for you; fixing that is chapter 15
material, and the answer starts with
checking the filesystem underneath.

## Housekeeping

```bash
podman volume ls
podman volume rm webdata          # only when truly done
podman system df                  # what is using the disk
podman system prune               # clear stopped containers,
                                  # dangling images, build cache
```

`prune` asks before it deletes and
does not touch volumes unless you add
`--volumes` — a flag to type only
after reading the list it prints.
Data you cannot recreate does not
belong one flag away from deletion;
chapter 11 is the backup that makes
housekeeping safe.
