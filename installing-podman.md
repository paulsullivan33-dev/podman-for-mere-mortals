# Installing Podman

Podman is a normal distro package. On
Debian or Ubuntu:

```bash
sudo apt update
sudo apt install podman
```

On Fedora it is `sudo dnf install podman`
(Fedora is Podman's home distro, so its
packages are newest); on Arch,
`sudo pacman -S podman`. The lab used
Ubuntu 24.04's package, Podman **4.9.3**.

Check:

```bash
podman --version
podman info
```

`podman --version` should print 4.x (5.x
on newer distros — every command in this
guide works the same). `podman info` is
the long health report: runtime, store,
network backend. Transcript t00 holds
the lab's.

## The rootless pieces (do not skip)

The headline install is one package, but
*rootless* Podman — running containers
as your ordinary user, the setup this
guide recommends — leans on three small
companions that most distros pull in
automatically. Check for them:

```bash
sudo apt install uidmap fuse-overlayfs slirp4netns
```

- **uidmap** provides `newuidmap`, which
  grants your account a range of user
  IDs (listed in `/etc/subuid` and
  `/etc/subgid`) so a container's "root"
  and its other accounts can all map back
  to you. Without a range, rootless
  containers can only ever be one user,
  and many images misbehave.
- **fuse-overlayfs** is the file-layer
  machinery for users who cannot mount
  the kernel's overlay filesystem
  directly. Rootful installs use the
  kernel driver and never notice it.
- **slirp4netns** gives rootless
  containers their network (chapter 8).
  Newer Podman (5.x) prefers **pasta**,
  from the `passt` package, for the same
  job.

Check: rootless is working when

```bash
podman info --format '{{.Host.Security.Rootless}}'
```

prints `true` — and when `grep ^$USER
/etc/subuid` shows your account with a
generous range (typically 65536 IDs).

Red Hat's own documentation is the
primary source for rootless setup, and
it is good; this chapter follows it.

## Rootful: sometimes, knowingly

Running Podman as root (`sudo podman
...`) is called **rootful**. It is the
older shape — Docker's only shape — and
it has legitimate uses: system services
managed by an admin, appliances, and
sandboxes like this guide's lab. The
cost is real: a rootful container's
root is the machine's root, fenced but
with the fence as the only protection,
and root-owned files land in root's
store where your user cannot manage
them. Rule of thumb: rootless for
anything you run as yourself, rootful
only when the machine itself is the
operator.

## What the lab actually did (declared)

The lab machine was a restricted
sandbox, and honesty requires its setup
to be stated rather than quietly
generalised:

- Rootless mode **could not run there**:
  the sandbox refuses writes to
  `/proc/<pid>/uid_map` for anyone but
  its own runtime, so `newuidmap`
  failed with "Operation not permitted"
  (transcript t00-rootless-attempt).
  The lab therefore ran **rootful**.
- The sandbox also refuses **overlay
  mounts** and **`setns`** (the call that
  steps into a namespace), so the lab's
  store used the plain-copy **vfs**
  driver (chapter 7 explains drivers),
  its containers used **host networking**
  (chapter 8), and `podman exec` and
  pods are documented-only in this
  guide, with their real errors in
  transcripts t02, t05 and t90.

None of that is how your machine will
behave. It is recorded in full in
`criteria.md` and `examples/lab-notes.txt`
so that every claim in this guide can be
traced to either a transcript or a
label. On a normal laptop, desktop or
server, install the package, keep your
own account, and run everything in this
guide rootless.
