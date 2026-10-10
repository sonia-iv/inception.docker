*This project has been created as part of the 42 curriculum by sivashch*

# Inception

## Description
Inception creates a web infrastructure using Docker and Docker Compose inside a Debian VM. Its goal is to learn service isolation, 
networking, configuration, and persistent storage.

Each service is built from its own Dockerfile based on the penultimate version of Debian-`debian:bookworm`.
| Service | Role | Connection |
|---|---|---|
| NGINX | Serves static files and forwards PHP requests | HTTPS `443`; TLS 1.2/1.3 |
| WordPress + PHP-FPM | Executes PHP and manages website content | `wordpress:9000`|
| MariaDB | Stores content, users and settings | `mariadb:3306`|

The browser connects to NGINX; NGINX forwards PHP requests to PHP-FPM; WordPress queries MariaDB, NGINX and WordPress
share the website files volume.

### Docker and project sources

Docker builds reusable images and runs containers from them. 
The Docker daemon manages images, containers, networks and volumes. 
Here, containers share the Debian VM's kernel. 
Compose defines the stack in YAML; the Makefile runs common Compose commands.

### Project's structure
`Makefile`- Build, start, stop and recreate the project
`srcs/docker-compose.yml` - Services, builds, ports, network, volumes and secrets
`srcs/requirements/<service>/Dockerfile` - Builds each service image
Service `conf/` directories - NGINX and MariaDB configuration
Service `tools/` directories - WordPress and MariaDB initialization scripts
`USER_DOC.md` - Access, credentials and basic checks
`DEV_DOC.md` - Setup, management and storage details

### Project's design
- Three separate service containers; only NGINX publishes port `443`.
- A shared bridge network allows communication by service name.
- WordPress uses PHP-FPM; users are `wpuser`(administartor) and `secwpuser`(Author).
- Startup scripts preserve existing data. WordPress waits for MariaDB readiness.
- MariaDB's `inception_initialized` marker records a completed database/account initialization.
- Services run in the foreground using `exec`, with `restart: always`.
- Two named volumes preserve data under `/home/sivashch/data/` inside the VM.
  
## Instructions

### Setup and execution
Use a Linux VM with Docker Engine, Docker Compose, Make and Git. Initial downloads require internet access.
1. Upload the project and open its root directory,
2. Supply `srcs\.env` and the three secret files listed above. *They already exist on the prepared VM;
a fresh clone excludes them*.
3. Inside the VM, map `127.0.0.1 sivashch.42.fr` in `/etc/hosts`.
4. Run `make`.

`make` creates the data directories and runs `docker compose -f srcs/docker-compose.yml up -d --build`.
| Command | Effect |
|---|---|
| `make` | Build and start in the background |
| `make stop` | Stop containers without removing them |
| `make down` | Remove project containers/network; preserve volumes and data |
| `make re` | Run `down`, then `all`; preserve data and use the build cache |
> [!NOTE]
> See `DEV_DOC.md` for complete setup instructions.

### Website access

- Website: <https://sivashch.42.fr>
- Login/administration: <https://sivashch.42.fr/wp-admin>

Expect a self-signed certificate warning. On Fedora without administrator access, open a SOCKS tunnel:

```bash
ssh -N -D 1080 login@ip_address
```

Use the VM's SSH username/IP and add `-p PORT` if needed. Configure Firefox for SOCKS v5 at `127.0.0.1:1080`, with proxy DNS enabled. Keep the tunnel open while browsing; restore browser proxy settings afterward. See `USER_DOC.md` for details.

## Resources
- [Docker overview](https://docs.docker.com/get-started/docker-overview/)
- [Dockerfile reference](https://docs.docker.com/reference/dockerfile/)
- [Docker Compose documentation](https://docs.docker.com/compose/)
- [NGINX HTTPS configuration](https://nginx.org/en/docs/http/configuring_https_servers.html) and [FastCGI module](https://nginx.org/en/docs/http/ngx_http_fastcgi_module.html).
- [WP-CLI command reference](https://developer.wordpress.org/cli/commands/).
- [MariaDB documentation](https://mariadb.com/docs/).

### Use of AI
AI assisted with explaining command syntax; reviewing when finished Dockerfiles, Compose configuration and startup scripts; troubleshooting MariaDB initialization, WordPress connectivity and NGINX upstream errors; refining `README.md`, `USER_DOC.md` and `DEV_DOC.md`.

## Project Description

### Virtual Machines vs Docker
| Virtual machine | Docker container |
|---|---|
| Own guest OS and kernel | Shares the Docker host's kernel |
| Managed by a hypervisor | Managed by Docker |
| More OS overhead | Usually lighter and faster to start |

This project uses both: VirtualBox runs Debian; Docker isolates services inside Debian.

### Secrets vs Environment Variables

| Environment variables | Compose secrets |
|---|---|
| Configuration passed into the process environment | Files granted to selected services under `/run/secrets/` |
| Usernames and settings from `srcs/.env` | Passwords read by startup scripts |

Passwords are supplied through `secrets/credentials.txt` (WordPress), `secrets/db_password.txt` and `secrets/db_root_password.txt` (MariaDB). Both `.env` and `secrets/` are ignored by Git. Neither mechanism automatically encrypts its local source files. Editing them does not change existing account passwords.

### Docker Network vs Host Network

| Project bridge network | Host network mode |
|---|---|
| Separate container addresses; service-name resolution | Shares the host network namespace |
| Explicit port publication (`443:443`) | Uses host ports directly |

Multiple containers can use the same internal ports without conflict (same port but different IP addresses). External users cannot access them unless we expose a port to the host (like port 443 for Nginx).
The project uses `srcs_inception`, a bridge network. VirtualBox host-to-VM networking is a separate layer.

### Docker Volumes vs Bind Mounts

| Named volume | Direct bind mount |
|---|---|
| Docker-managed identity, inspected with Docker commands | Maps a specific host path into a container |
| Normally Docker-managed storage | Depends on host paths and permissions |

Here, named volumes use local-driver bind options to point to VM directories. 
| Volume | VM directory | Container destination |
|---|---|---|
| `srcs_mariadbVol` | `/home/sivashch/data/mariadb` | `/var/lib/mysql` in MariaDB |
| `srcs_wordpressVol` | `/home/sivashch/data/wordpress` | `/var/www/html` in WordPress and NGINX |

**Database records and website files survive container recreation and VM reboots. Deleting their backing directories deletes the data.**
