# Startup Services Using Docker Compose

Provides an easy-to-extend stack of services via Docker Compose.

## Requirements

- Docker and Docker Compose installed
- a group (such as `mygroup`), as well as its GID
- a user (such as `torgo`), as well as their UID, whose primary group is `valleylodge` and who is also a member of the `docker` group

## Usage

To bring the stack up once at a terminal prompt, type `docker compose --env-file /path/to/docker-compose.env --file /path/to/docker-compose.yml up -d` and press Enter.

## Shared Variables

- `PROJECT_NAME`: Overall name of the project (or "stack", as it is called in Portainer). Default: `startup`
- `HOST_GROUP_NAME`: The primary group name, used for the default network name. Default: `valleylodge`
