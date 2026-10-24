# Limits

What Podman — and the container idea
itself — cannot do, stated plainly.

## One kernel, shared by everything

Every container on a machine runs on
the host's single Linux kernel. Three
consequences:

- **No other operating systems.** A
  container cannot be Windows, or a
  different Linux kernel, or a kernel
  module your host does not have. If
  the program needs those, it needs a
  virtual machine (chapter 13).
- **A kernel bug is everyone's bug.**
  The fences are kernel features
  (namespaces, cgroups, seccomp). They
  are good fences, audited and old —
  and a fence is not a wall. Run truly
  hostile code in a VM, not in a
  container of any brand.
- **"Root" inside can matter outside
  if you let it.** Rootful containers
  run programs whose root is real
  root, fenced. Rootless mode exists
  precisely to shrink this (chapter 1),
  and features like `--privileged`,
  which removes most fences at once,
  should read as "no container, with
  extra steps" — sometimes right for a
  hardware tool on your own bench,
  never a default.

## Rootless has rough edges

- Ports below 1024 are root's; publish
  8080, not 80 (chapter 8).
- Rootless networking (slirp4netns,
  pasta) carries traffic through your
  user account: throughput is lower
  than rootful bridge networking, and
  some tricks (ping's raw sockets,
  seeing the real source address) are
  limited or emulated. Depends on your
  setup and your Podman version; the
  trend is improvement.
- File ownership inside a rootless
  container is a mapping (chapter 3's
  subuid ranges). Programs that check
  many ownerships — some databases —
  occasionally need the range set up
  properly rather than the single-ID
  fallback.

## One machine

Podman runs containers on the machine
you are standing on. It does not
balance load, restart containers on a
machine that has died (the machine is
dead), or heal a fleet. systemd
(chapter 10) covers "come back after
reboot and after crashes" for one
machine; anything beyond that is
orchestration, and it is a different
subject with a different book
(chapter 13).

## Not a packaging system for the host

Containers do not update your host,
manage its configuration, or agree
with each other about versions. Two
containers may carry two different
copies of the same library — that is
their charm and their weight: images
duplicate what packages would share,
and `podman system df` (chapter 5)
is the honest bill.

## What this guide could not test

Restated from `criteria.md`, because a
limits chapter is where it belongs:
the lab's sandbox blocked rootless
mode, bridge networking and port
publishing, `podman exec` and `podman
cp`, pods, compose runs, and running
generated systemd units. Those are
covered from official documentation
and labelled wherever they appear;
the everyday core — pull, run, logs,
volumes, build, commit, save, load,
generate — was run for real. Your
machine almost certainly allows all
of the blocked list; the sandbox's
refusals are its own, and chapter 15
teaches reading such errors rather
than fearing them.
