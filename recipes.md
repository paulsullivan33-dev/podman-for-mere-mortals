# Recipes

Short, complete answers to common
wants. Each says which earlier chapter
it stands on, and each is honest about
what was verified in the lab.

## A web server that starts at boot

Chapter 10, end to end: write the
Quadlet file (the repo's
`podman-lab/quadlet/web.container` is
the whole thing), put it in
`~/.config/containers/systemd/`, then

```bash
loginctl enable-linger $USER
systemctl --user daemon-reload
systemctl --user enable --now web.service
```

Check: reboot, log in, and
`curl http://localhost:8080/` answers —
before you run anything by hand. The
lab verified the unit's *generation*;
the boot itself is documented only
(see chapter 10's opening note).

## Try a database without installing it

Chapter 4's shape, pointed at software
you would never let near your package
manager:

```bash
podman run -d --name pg-try \
  -e POSTGRES_PASSWORD=tryme \
  -p 5433:5432 docker.io/library/postgres:16
psql -h localhost -p 5433 -U postgres
podman rm -f pg-try     # gone, entirely, when done
```

Nothing was installed on the host; the
database, its config and its default
data all lived and died inside the
container. (No volume is attached on
purpose — this is a *try*, and its data
is meant to be disposable. Chapter 7
is the difference when it is not.)

## A throwaway test box

You need a clean Linux to test a script
or open a dubious tarball:

```bash
podman run --rm -it --network none \
  -v "$PWD":/work:ro \
  docker.io/library/alpine:3.20 sh
```

Inside, you are root in a small world
whose only view of your files is the
read-only folder you offered, with no
network. `--rm` means leaving is
cleaning up. Transcript t02's first
session is this pattern in miniature.

## A scheduled job, in a container

For "run this every night, with these
tools, and leave my system out of it":
build a small image (chapter 6) whose
CMD is the job, then let cron or a
systemd timer run it one-shot:

```bash
# crontab -e
30 2 * * * podman run --rm -v backups:/data \
  localhost/nightly-job:latest
```

The failure mode to plan for: a job
that fails silently at 2:30 AM. Give
the image a command that exits
non-zero on failure, and let cron's
mail or the journal (chapter 10) carry
the news.

## Move an image to a machine with no network

Chapter 11's round trip as a recipe:

```bash
podman save -o greeter.tar localhost/mere-greeter:latest
# carry the file over, then on the other machine:
podman load -i greeter.tar
podman run --rm localhost/mere-greeter:latest
```

Verified in the lab end to end
(transcript t07 used the web snapshot;
the greeter travels the same way).

## Run the other tool's image anyway

Someone hands you a `docker run`
command from a README. Translation is
nearly mechanical (chapter 13): replace
`docker` with `podman`, keep the flags,
and where Docker assumed its daemon
defaults, Podman usually agrees. The
two flags to double-check are `-v`
paths (rootless ownership, chapter 7)
and anything that names the Docker
socket — there is no Docker socket
here, and chapter 13 explains the
Podman socket for the rare tool that
truly needs one.
