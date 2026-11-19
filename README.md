# Startup Services Using Docker Compose

Provides an easy-to-extend stack of services via Docker Compose.

## Requirements

- Docker and Docker Compose installed
- a group (such as `valleylodge`), as well as its GID (such as `1000`)
- a user (such as `torgo`), as well as their UID (such as `1000`), whose 
  primary group is `valleylodge` and who is also a member of the `docker` group.

## Usage

Clone the repo and customize it to your liking. Comment out any unwanted services in `docker-compose.yml`.

To bring the stack up manually at a terminal prompt, use:

    docker compose --file /path/to/docker-compose.yml --env-file /path/to/docker-compose.env up --detach

To bring the stack down manually, use:

    docker compose --file /path/to/docker-compose.yml --env-file /path/to/docker-compose.env down

The repo also includes a [`systemd.service(5)`][svcunit] unit file, for bringing
the stack up and down automatically with a Linux host. To use it:

1. Edit the service file. Change the paths in the unit file to point to your compose and env files, and change the user and group.
1. Put the file in `/etc/systemd/system`.
1. Reload systemd with `systemctl daemon-reload`.
1. Activate it with `systemctl enable --now docker-compose-startup.service`.

[svcunit]: https://www.freedesktop.org/software/systemd/man/251/systemd.service.html

## Shared Variables

This stack uses a number of variables throughout, most of which are found in
`docker-compose.env`. Customizing this file to match your system, and having
valid config files for services in place, should make the stack work.

- `PROJECT_NAME`: Overall name of the project (or "stack", as it is called in
  Portainer). Default: `startup`
- `DNS_ZONE_LOCAL`: The DNS zone for your LAN. Default: `zone.example.com`
- `HOST_COMPOSE_FOLDER`: The folder where the Docker Compose files to import
  exist. Default: `./compose.d`
- `HOST_DATA_FOLDER`: The folder or mount on the host machine in which stateful
- `HOST_FQDN_LOCAL`: The domain name at which the host system can be found on
  the LAN. Default: `hostname.zone.example.com`
  data lives. See below for more info. Default: `./data`
- `HOST_GROUP_NAME`: The primary group name, used for the default network name.
  Default: `valleylodge`
- `HOST_IPV4_LOCAL`: The IP address of the host machine on the LAN. Default:
  `192.168.1.2`
- `HOST_NAME`: The host name at which the host system can be found on
  the LAN. Default: `manos`
- `HOST_TIME_ZONE`: The default time zone for the host system. Default: `Etc/UTC`
- `HOST_USER_GID`: The group ID of the user running this stack. Default: `1000`
- `HOST_USER_UID`: The user ID of the user running this stack. Default: `1000`

### `HOST_DATA_FOLDER`

This folder is where stateful data lives for your services. It should exist and 
be owned by the same user/group combination as in the service unit file. `HOST_DATA_FOLDER` looks like this:

    HOST_DATA_FOLDER
    ├── apps
    │   ├── backrest
    │   │   ├── config
    │   │   └── data
    │   ├── syncthing
    │   ├── technitium
    │   ├── traefik
    │   └── ...
    ├── backups
    │   ├── database-snapshots
    │   ├── restic
    │   ├── time-machine
    │   └── ...
    ├── databases
    │   ├── mysql
    │   ├── pgsql
    │   └── ...
    ├── secrets <-- This should be secured!
    └── ...

Major sections include:

- `apps` is for named service folders, inside each of which are sub-folders for
  configuration, data, logs, etc.  (Caches and other temporary, frequently-flushed
  mounts should use Docker named volumes.)
- `backups` is for backup data, such as snapshots of databases, Time Machine
  shares, [restic][] or [borg][] repositories, etc.
- `databases` is for "hot" database data, the equivalent of folders like 
  `/var/lib/mysql` or `/var/lib/postgresql`.
- `secrets` is for passwords, tokens, certificates, etc. For
  security reasons, care should be taken to secure the contents of this folder
  (for example, by mounting a LUKS volume at `$HOST_DATA_FOLDER/secrets`).

[restic]: https://restic.net/
[borg]: https://www.borgbackup.org/

## Memory and Resources

All containers are set by default to have a CPU limit of 2.0 and a memory limit
of 1 gigabyte. For memory- and CPU-intensive services (such as databases) this
will result in poor performance. Tune these values in individual compose files
to ones more appropriate to your system.

## TODO

- Make CPU/memory settings variables in environment files
- Move dummy API key for Tautulli from `docker-compose.env` to file in `secrets`
