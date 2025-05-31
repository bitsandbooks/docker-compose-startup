# Startup Services Using Docker Compose

Provides an easy-to-extend stack of services via Docker Compose.

## Requirements

- Docker and Docker Compose installed
- a group (such as `valleylodge`), as well as its GID
- a user (such as `torgo`), as well as their UID, whose primary group is
  `valleylodge` and who is also a member of the `docker` group

## Usage

To bring the stack up manually at a terminal prompt, type `docker compose --file
/path/to/docker-compose.yml --env-file /path/to/docker-compose.env up --detach`
and press Enter. To bring it down manually, use `docker compose --file
/path/to/docker-compose.yml --env-file /path/to/docker-compose.env down`.

The repo also includes a [`systemd.service(5)`][svcunit] unit file, for bringing
the stack up and down automatically with a Linux host. Just change the paths in
the unit file to point to your compose and env files, change the user and group,
put the file in `/etc/systemd/system`, reload systemd with `systemctl 
daemon-reload`, and activate it with `systemctl enable --now 
docker-compose-startup.service`.

[svcunit]: https://www.freedesktop.org/software/systemd/man/251/systemd.service.html

## Shared Variables

This stack uses a number of variables throughout, most of which are found in
`docker-compose.env`. Customizing this file to match your system, and having
valid config files for services in place, should make the stack work.

- `PROJECT_NAME`: Overall name of the project (or "stack", as it is called in
  Portainer). Default: `startup`
- `HOST_COMPOSE_FOLDER`: The folder where the Docker Compose files to import
  exist. Default: `./compose.d`
- `HOST_DATA_FOLDER`: The folder or mount on the host machine in which stateful
  data lives. See below for more info. Default: `/var/local/trunk`
- `HOST_FQDN_LOCAL`: The domain name at which the host system can be found on
  the LAN. Default: `hostname.zone.domain.tld` (or `hostname.localdomain`)
- `HOST_GROUP_NAME`: The primary group name, used for the default network name.
  Default: `valleylodge`
- `HOST_IPV4_LOCAL`: The IP address of the host machine on the LAN. Default:
  `192.168.1.2`
- `HOST_USER_UID`: The user ID of the user running this stack. Default: `1000`
- `HOST_USER_GID`: The group ID of the user running this stack. Default: `1000`

### `HOST_DATA_FOLDER`

This folder is where stateful data lives for your services. Major sections 
include:

- `apps` is for named service folders, inside each of which are sub-folders for
  configuration, data, logs, etc. 
- `backups` is for backup data, such as snapshots of databases, or [restic][] or
  [borg][] repositories.
- `databases` is for "hot" database data, the equivalent of folders like 
  `/var/lib/mysql` or `/var/lib/postgresql`.
- `secrets` is for passwords, tokens, certificates, etc. For
  security reasons, care should be taken to secure the contents of this folder
  (for example, by mounting a LUKS volume at `$HOST_DATA_FOLDER/secrets`).

[restic]: https://restic.net/
[borg]: https://www.borgbackup.org/

`$HOST_DATA_FOLDER` looks like this:

    $HOST_DATA_FOLDER
    ├── apps
    │    ├── bind9
    │    ├── syncthing
    │    │    ├── config
    │    │    └── data
    │    ├── traefik
    │    └── ...
    ├── backups
    │    ├── databases
    │    ├── restic
    │    └── ...
    ├── databases
    │    ├── mysql
    │    ├── pgsql
    │    └── ...
    ├── secrets <-- This should be secured!
    └── ...

## Memory and Resources

All containers are set by default to have a CPU limit of 2.0 and a memory limit
of 1 gigabyte. For memory- and CPU-intensive services (such as databases) this
will result in poor performance. Tune these values in individual compose files
to ones more appropriate to your system.

## TODO

- Make CPU/memory settings variables in environment files
- Provide example `$HOST_DATA_FOLDER` with reference configs
