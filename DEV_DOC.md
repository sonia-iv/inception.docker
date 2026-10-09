# Developer documentation

This document provides technical instructions for developers who want to 
understand, set up, and maintain the Inception project.

## Environment Setup from Scratch

### Prerequisites

A Linux virtual machine with the following tools installed:

- Docker Engine and the Docker Compose plugin.
- Make.
- Git.
### Local Domain Configuration

### Local Domain Configuration

Inside the VM, open the hosts file:
```bash
sudo vim /etc/hosts
```
Add the following line:
```text
127.0.0.1 sivashch.42.fr
```
*This maps the project domain to the VM’s address.*

>[!NOTE]
> If you do not have administrator access on the host computer, use an
> SSH tunnel instead of editing its hosts file:
```bash
ssh -N -D 1080 login@ip_address
```
Replace `login` and `ip_address` with the VM’s username and reachable
IP address.
Configure Firefox to use SOCKS v5 at `127.0.0.1:1080` and enable
**Proxy DNS when using SOCKS v5**. 
*This allows domain resolution through the VM.*

Keep the tunnel open while browsing. **See `USER_DOC.md` for the complete
browser configuration and cleanup instructions.**
### Configuration and Secrets

Before the first launch, provide the following files:

- `srcs/.env`: Service configuration values and usernames.
- `secrets/credentials.txt`: WordPress administrator and second-user passwords.
- `secrets/db_password.txt`: MariaDB user password.
- `secrets/db_root_password.txt`: MariaDB root password.

The `secrets/` directory is located at the project root.
Both this directory and `srcs/.env` are ignored by Git.

>[!NOTE]
> All the following instructions in this section do not concern this evaluation.
> All the files are already present on the machine.
On the prepared VM, these files already exist. For a fresh clone,
create them or securely copy them into the project.

Fill in the values using the variable names and formats expected by
the Compose file and startup scripts.

## Build and launch the project using the Makefile and Docker Compose

The entire infrastructure is orchestrated by both Docker Compose and a Makefile,
where the Makefile is used to run Docker Compose commands.

From the project’s root directory, run:
```bash
make
```

The default target creates the required data directories, builds the
images, and starts the containers in the background using:
```bash
docker compose -f srcs/docker-compose.yml up -d --build
```
To check container status, run:
```bash
docker compose -f srcs/docker-compose.yml ps -a
```
-a - lists all existing containers, even if not running.


## Containers and volumes management

Run these commands from the project’s root directory inside the VM
or its SSH session.

### Check service status

```bash
docker compose -f srcs/docker-compose.yml ps
```

### View recent service logs

```bash
docker compose -f srcs/docker-compose.yml logs --tail=40 nginx wordpress mariadb
```

### Access a container shell

```bash
docker exec -it wordpress sh
```
Replace `wordpress` with another container name if needed.
Type `exit` to leave the container.

### Restart services

Restart all services:
```bash
docker compose -f srcs/docker-compose.yml restart
```
Restart one service:
```bash
docker compose -f srcs/docker-compose.yml restart wordpress
```
Restarting does not rebuild images or apply changed container
environment variables.

### Rebuild and recreate services

After changing the WordPress or NGINX build configuration:
```bash
docker compose -f srcs/docker-compose.yml up -d --build --force-recreate wordpress nginx
```
Mounted persistent data is preserved.
### Inspect persistent storage

Display the volumes mounted by each container:
```bash
docker inspect mariadb wordpress nginx --format '{{.Name}}: {{range .Mounts}}{{if eq .Type "volume"}}{{.Name}} -> {{.Destination}} {{end}}{{end}}'
```
Inspect the project volumes and their backing storage configuration:
```bash
docker volume inspect srcs_mariadbVol srcs_wordpressVol --format '{{.Name}}: {{.Options}}'
```
### Inspect the project network
```bash
docker network inspect srcs_inception --format '{{.Driver}}'
```
Expected driver: `bridge`.


## Data storage and Persistence
The project uses named Docker volumes backed by directories inside
the VM, which acts as the Docker host.

- **MariaDB data:** `/home/sivashch/data/mariadb`
  Stores WordPress content, users, and settings.
  Mounted at `/var/lib/mysql` inside the MariaDB container.

- **WordPress files:** `/home/sivashch/data/wordpress`
  Stores WordPress files, themes, plugins, and uploaded media.
  Mounted at `/var/www/html` inside the WordPress and NGINX containers.

These directories exist outside the containers. Their data survives
container restarts, container recreation, and VM reboots.

Deleting the backing directories deletes the stored data.
```bash
docker volume inspect srcs_mariadbVol srcs_wordpressVol --format '{{.Name}}: {{.Options}}'
```

