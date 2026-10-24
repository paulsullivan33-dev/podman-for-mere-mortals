# Writing Criteria

Every guide in this repo is written to the same
standard. If you're adding or editing a guide —
human or AI — hold it to these.

1. **Plain words, real ideas.** Simple language,
   never dumbed-down concepts. Explain jargon on
   first use; collect terms in the glossary.
2. **No hype.** No marketing adjectives, no
   "revolutionary," no promises about what
   something will do for the reader.
3. **Honest about limits.** Say what the thing
   *can't* do. State uncertainty instead of
   smoothing it over. Validation notes include
   what remains unverified.
4. **No invented numbers.** Benchmarks, defaults,
   and specs are either measured or qualified with
   "depends on your setup." If it wasn't tested,
   say so.
5. **Checkable.** Commands are meant to be run,
   with "Check:" blocks showing what success looks
   like. Sources are linked, not asserted.
6. **Respect the reader's setup.** Call out
   Windows/Mac/Linux differences. Don't assume a
   GPU, a budget, or prior experience.
7. **Safety isn't skipped.** Real risks get
   stated plainly, not buried.
8. **Scoped.** Each guide says what it covers and
   points elsewhere for the rest. No guide
   pretends to be the whole subject.
9. **Structure over prose.** Tables, before/after
   examples, symptom → fix. Scannable beats essay.

## Verification notes for this guide

Built and verified 2026-10-23/24 against
**Podman 4.9.3** (Ubuntu package
4.9.3+ds1-1ubuntu0.2), with crun, netavark
and Buildah as shipped in Ubuntu 24.04,
on a 2-CPU, 7.7 GB sandbox machine
running Ubuntu 24.04.

**Primary sources read, not remembered:**
Podman's official documentation and man
pages for installation and rootless
setup (the subuid, uidmap, fuse-overlayfs
and slirp4netns requirements in chapter
3), networking (publishing, rootless
port limits, pasta and slirp4netns
behaviour in chapter 8), pods and
`podman generate kube` (chapter 9), and
systemd integration — Quadlet
(`podman-systemd.unit(5)`) and `podman
generate systemd`, including its
deprecation (chapter 10).

**The sandbox, declared.** The lab
machine was a restricted sandbox whose
kernel policy refuses three things for
ordinary processes: writing user-ID maps
for a further namespace (so **rootless
Podman cannot start**), the `setns`
system call (so **netavark cannot set
up bridge networking**, and **crun
cannot exec into a container**), and
delegating control groups (so **pods
cannot be created**). Overlay mounts
are refused too. The lab therefore ran
Podman **rootful**, with the store on
the plain-copy **vfs** driver,
cgroup management set aside, and its
one long-running container on **host
networking**. Those are the sandbox's
properties, not Podman's, and no
chapter presents the lab's shape as
the recommended one: chapter 3 teaches
the normal rootless install, and every
sandbox-shaped exception is named where
it occurs.

**Ran for real** (transcripts in
`podman-lab/transcripts/`):

- t00 — `podman --version` → 4.9.3;
  `podman info` → crun runtime, vfs
  store, netavark backend, rootless
  false (the lab's declared shape).
- t00-rootless-attempt — running
  `podman info` as an unprivileged user
  fails at `newuidmap` with "Operation
  not permitted": the rootless block,
  kept as evidence.
- t01 — `podman pull` of
  docker.io/library/alpine:3.20
  (image ID bf8527eb54c3…, 8.1 MB) and
  docker.io/library/nginx:alpine;
  `podman images`, an inspect excerpt,
  and `podman system df` on the result.
- t02 — a foreground container runs a
  shell command and prints Alpine's
  release (3.20.10) and a container
  hostname; Nginx detached on host
  networking answers `curl` with HTTP
  200; `podman ps` and `podman logs`
  show it live. The same transcript
  keeps the lab's `podman exec` error
  (`crun: setns 'mnt'`) — exec is
  documented-only here for that reason.
- t03 — a named volume written by one
  container, read back by a second
  after the first was deleted; a host
  folder bind-mounted read-only and
  read; `volume ls` and `volume
  inspect`.
- t04 — the greeter image built from
  the Containerfile in
  `podman-lab/build/` (`podman build`
  succeeds), run, and listed by
  `podman history`.
- t06 — `podman generate systemd
  --name web` emits a full unit (with
  Podman's own deprecation warning for
  the command); the Quadlet file in
  `podman-lab/quadlet/` is translated
  by Podman's generator in dry-run mode
  into a complete systemd service.
- t07 — `podman commit` of the running
  web container to a new image; the
  image `save`d to a 64 MB tar,
  deleted, `load`ed back and listed;
  the container then stopped, removed,
  and the space re-shown by `system
  df`.

**Errors hit and diagnosed for real,**
written up as chapter 15: the rootless
uid_map refusal; the store's silent
fallback to fuse-overlayfs on an
overlay-backed root, failing with
"fuse: device not found" when FUSE is
unavailable (transcript
t90-first-attempt); netavark's `setns`
failure on bridge networking
(transcript t90); crun's `setns`
failure on exec (t02) and its keyring
failure when a container is started by
a systemd system unit; and pods
refused at cgroup creation (t05).

**Documented only, and labelled as
such in the chapters:** rootless
installation and behaviour end to end;
bridge networking and published
ports; containers reaching each other
by name; `podman exec` and `podman
cp`; pods (the command was run and
its sandbox error kept, t05);
`podman compose` runs; actually
starting a generated systemd or
Quadlet unit — user units need a user
bus the sandbox does not run, and a
system-unit attempt failed in crun as
noted above; a volume *restore*
rehearsal (volume writes, reads and
mounts were proven, the tar round trip
is the documented pattern); pulling
from registries other than Docker
Hub; and `podman machine` on Windows
and macOS.

**Fixtures:** the two pulled images
are public (Alpine 3.20, Nginx
alpine). Everything the lab made up
is small and generic — a greeter
script that says hello, a volume
note, a host note — written for any
reader to reproduce. The one invented
user account used while probing
rootless mode (`mortal`) appears in
no transcript. Nothing in the lab is
modelled on any real person,
organisation or event.
