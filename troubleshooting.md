# Troubleshooting

Symptom first, then the cause the lab
or the documentation supports, then
the fix. Several of these the guide's
lab met personally; those say so.

## "Permission denied", in four costumes

**1. It happens when a rootless
command starts anything.**

```
newuidmap: write to uid_map failed: Operation not permitted
```

Rootless mode cannot set up its user
namespace (transcript
t00-rootless-attempt, from the lab's
sandbox). On a real machine: check
that `uidmap` is installed and that
`/etc/subuid` holds a range for your
user (chapter 3). If both are right and it still
fails, the machine itself restricts
user namespaces — some hardened
kernels and nested sandboxes do —
and rootful is the remaining road.

**2. It happens inside a path the
container mounts.** Usually ownership,
not malice: a rootful container wrote
root-owned files into a bind mount, or
a rootless one shows odd IDs. Fix the
ownership on the host side, and
re-read chapter 7's warning about
rootful writes.

**3. `crun: ... Operation not
permitted` while starting.** The
sandbox's signature error family —
chapter 15's lab hit it for
`setns 'mnt'` (exec) and for kernel
keyrings (a systemd service). On a
real machine this points at a kernel
or LSM (AppArmor/SELinux) policy
blocking the runtime; the runtime's
name at the front of the error —
`crun:` — tells you the fences failed
before your program ever ran.

**4. A published port refuses
connections from *other* machines.**
Localhost works, the LAN does not:
that is the host's firewall, not
Podman. Check the firewall first —
it is the cause more often than the
container is.

## The image will not pull

```
Error: initializing source docker://...: unexpected EOF
```

or a flat `404`. Walk it:

1. Name first: registry, namespace,
   tag — `alpine:3.20` exists,
   `alpine:3.2o` (letter o) does not,
   and `podman search` on a full
   registry path is a blunt instrument;
   check the project's page instead.
2. Then the network: can the machine
   reach the registry at all
   (`curl -I https://docker.io/v2/` —
   a 401 there is *success*: the
   registry answered).
3. Then auth: private registries need
   `podman login` first; public ones
   rate-limit heavy pullers, which a
   shared or corporate address can
   look like.

## "That name is already in use"

A stopped container keeps its name
(chapter 4). `podman ps -a` will show
the ghost; `podman rm` it, or start
containers ephemeral by habit
(`--rm`) for one-shots. `--replace`
on a long-running `run` is the
deliberate version of the same fix.

## The port is not listening

Walk chapter 8's list, in order:
container actually running (`podman
ps` — a crashed-at-start container
shows in `ps -a`, and its `podman
logs` will say why); the program
inside listening on the port the
image *says* (inspect shows EXPOSEd
ports, which are documentation, not
configuration); the mapping published
(`podman port web`); the host
firewall. Publishing `8080:80` and
then curling port 80 has happened to
everyone exactly once.

## It ran yesterday; the disk is full today

```
podman system df
```

Images accumulate (every tag you ever
tried), stopped containers linger,
build cache grows. `podman system
prune` reclaims the unloved; volumes
are excluded unless you insist
(chapter 7) — insist rarely. The
lab's first rootful attempt also
teaches a storage variant worth
recognising: on an overlay-backed
root filesystem the store silently
falls back to **fuse-overlayfs**, and
if FUSE is unavailable the start fails
with "fuse: device not found"
(transcript t90-first-attempt). On
a real machine the cure is giving the
store a real filesystem
(`/etc/containers/storage.conf`,
`graphroot`) — the lab instead ran
its whole store as **vfs**
(chapter 7), which is the diagnostic
step, not the destination.

## generate systemd works; the service won't start

Then the problem is systemd-side, and
it will tell you so: `systemctl
--user status web` and
`journalctl --user -u web` carry the
real error. The three regulars:
linger not enabled, so nothing starts
until login (chapter 10); a stale
unit after editing the Quadlet file
without `daemon-reload`; and an image
name in the file that only exists in
root's store — rootless and rootful
stores are different cupboards
(chapter 2), and a unit runs from
exactly one of them.

## When in doubt

```bash
podman info            # which world am I in?
podman ps -a           # what exists?
podman logs <name>     # what did it say?
podman inspect <name> --format '{{.State.Error}}'
```

Most mysteries survive contact with
those four commands for about a
minute. The lab's notes
(`examples/lab-notes.txt`) record
every error this guide met, with what
it turned out to mean.
