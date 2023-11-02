# Sillycon

The Silly Container Manager

## What is Sillycon

Sillycon is a minimal Linux container runtime created for a
[NooK 2023](https://2023.nook-luebeck.de/) talk, implemented in roughly 75
lines of POSIX shell. It only requires basic system utilities usually installed
on every modern GNU/Linux system:

- tar
- unshare
- mount

The kernel must be configured with namespace support, including unprivileged
user namespace creation.

The `--map-users` and `--map-groups` options passed to unshare by default also
require UID/GID mappings for the current user in `/etc/uidmap` and
`/etc/gidmap` and the `newuidmap`/`newgidmap` utilities installed as SUID. All
of this should be the case by default on a single-user Debian/Ubuntu
installation.

Sillycon is by no means a proper replacement for an actual container runtime or
manager like Docker or Podman + crun/runc. It is only meant as a tool to learn
about and experiment with a few low-level aspects of Linux containers.

## Presentation

Presentation slides for the NooK 2023 lightning talk (in German) about Sillycon
can be found at [slides.pdf](slides.pdf).

## Limitations

- The script currently doesn't enable cgroup namespaces. Enabling cgroup
  namespaces is likely necessary to run a full init system like systemd inside
  the container, but I have not checked if doing so is sufficient.
- The container image must provide `/usr/bin/env`, `/bin/sh`, as well as a
  `umount` command in the `PATH` set in the container's default environment.
- There is no network setup. By default a new network namespace is created,
  isolating the container from the host network. By removing the `--net`
  argument from the `unshare` command, the network configuration can be shared
  with the host instead.

## Configuration

A minimal Debian system does not provide any tools to properly parse a JSON
file. For this reason, Sillycon does not evaluate the container image's
manifest and configuration files. Instead, the relevant settings from these
files must be converted to a format digestible by Sillycon manually.

The `sillycon` script expects to find a file `config.sh` in the current
directory, which might look like the following:

```sh
DIR='image'
LAYERS='
	6a3253cdc9090d53abcb59819549d6c4cc927ff24a5eabc9d8b02d89c3558d57/layer.tar
	24bf8008a403d1ea3f89a787665e5099ccf8ba6e0e812b32b026258d5cbe26ed/layer.tar
'

CMD='bash'
ENV='PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin'
HOSTNAME='sillycon'
```

`DIR` is the path to an unpacked Docker image (relative to the current
directory); `LAYERS` is resolved relative to `DIR`.
