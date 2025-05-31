# Startup Services Using Docker Compose

Provides an easy-to-extend stack of services via Docker Compose.

## Requirements

- Docker and Docker Compose installed
- a group (such as `valleylodge`), as well as its GID
- a user (such as `torgo`), as well as their UID, whose primary group is
  `valleylodge` and who is also a member of the `docker` group

## Usage

To bring the stack up once at a terminal prompt, type `docker compose --file
/path/to/docker-compose.yml --env-file /path/to/docker-compose.env up --detach`
and press Enter.

## Shared Variables

This stack uses a number of variables throughout, most of which are found in
`docker-compose.env`. Customizing this file to match your system should be all
you need to do to make the stack work.

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

### `HOST_DATA_FOLDER`

This folder is where stateful data lives for your services. Major sections are
`apps` for service data, `databases` for raw databas

Each service goes in individually-named folders in `apps`, inside each of which
are sub-folders for configuration, data, logs, etc. Some service files may refer
to the `secrets` folder to get passwords, tokens, certificates, etc. For
security reasons, care should be taken to secure the contents of this folder
(for example, by mounting a LUKS volume at `$HOST_DATA_FOLDER/secrets`).

`$HOST_DATA_FOLDER` looks like this:

    $HOST_DATA_FOLDER
    ├── apps
    │    ├── bind9
    │    ├── syncthing
    │    │    ├── config
    │    │    └── data
    │    ├── traefik
    │    └── ...
    ├── secrets <-- This should be secured!
    └── ...
