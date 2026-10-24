# Running as Services

A container you started by hand stops
being a service when you log out, and
does not come back when the machine
reboots. To give containers a proper
service life — start at boot, restart
on failure, logs in the journal — you
put them under **systemd**, Linux's
service manager. Podman was designed
for this; it is one of the main reasons
it exists.

**What the lab verified:** both ways of
*writing* the unit, for real — the
Quadlet below was translated into a
service by Podman's own generator, and
`podman generate systemd` produced its
unit for the lab's web container;
transcript t06 holds both outputs.
**What it could not verify:** a unit
actually *running* a container. The
lab's sandbox has no user bus for
rootless units, and its kernel refused
a system unit at the container-runtime
step (the attempt is recorded in
`examples/lab-notes.txt`). Following
this chapter on a normal machine, the
units below simply start.

## The modern way: Quadlet

A **Quadlet** file is a short
declaration — what to run, not how to
run it. For our web server, save this
as `~/.config/containers/systemd/
web.container` (rootless) — the file
lives in this repo at
`podman-lab/quadlet/web.container`:

```ini
[Unit]
Description=A small web server

[Container]
Image=docker.io/library/nginx:alpine
ContainerName=web
PublishPort=8080:80

[Service]
Restart=always

[Install]
WantedBy=default.target
```

Then:

```bash
systemctl --user daemon-reload
systemctl --user start web.service
systemctl --user enable web.service   # at boot/login
```

Quadlet's generator (`podman-systemd.
unit` in the man pages) translates the
`.container` file into an ordinary
systemd unit — the lab ran the
generator in dry-run mode and
transcript t06 shows the full service
it produced, the long `podman run`
command with systemd supervision woven
in. That translation is the honest
way to see what Quadlet does: it is
not magic, it is a well-kept template.
Sibling file types — `.volume`,
`.network`, `.pod`, `.kube` — declare
the other pieces the same way.

Two rootless habits matter here:

- **Linger.** A user service normally
  dies at logout. `loginctl enable-
  linger $USER` tells systemd to keep
  your services running and to start
  them at boot without you logging in —
  the difference between "runs while I
  watch" and "runs".
- **The [Install] section.** Without
  `WantedBy=default.target`, `enable`
  has nothing to attach the service to.

## The older way: generate systemd

```bash
podman create --name web -p 8080:80 \
  docker.io/library/nginx:alpine
podman generate systemd --name web --files --new
```

prints (or, with `--files`, writes) a
complete unit for an *existing*
container. The lab generated one for
its web container; transcript t06
shows the header and the important
line is the warning Podman itself
prints first: the command is
**deprecated in favour of Quadlet**.
Generated units also freeze the
container's settings at generation
time, where a Quadlet file *is* the
setting. Use generate to migrate an
old setup; write Quadlets for anything
new.

## Living with services

```bash
systemctl --user status web
journalctl --user -u web -f     # the container's logs, live
systemctl --user restart web
podman ps                       # the same container, Podman's view
```

The journal is where a service's logs
belong, and Quadlet routes them there
by default — `podman logs` still works
too. Updates are deliberately boring:
change the tag in the `.container`
file (or let the image be pulled
fresh), `daemon-reload`, `restart`.
There is no auto-update fairy in this
chapter; pulling a new `latest`
silently is how a Tuesday surprise
happens, and chapter 5 already warned
what `latest` means.

For system services rather than user
ones, the same Quadlet files live in
`/etc/containers/systemd/` and are run
with plain `systemctl` — the rootful
shape, chosen when the machine, not a
person, owns the service.
