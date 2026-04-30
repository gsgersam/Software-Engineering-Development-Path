*This project has been created as part of the 42 curriculum by gsanin-m.*

# Inception

## Description

Inception is a system administration project from the 42 curriculum focused on building a small, production-inspired service stack with Docker. The goal is to replace a single monolithic setup with isolated containers that each solve one clear responsibility.

This repository builds a complete web platform with a **mandatory core** and several **bonus services**.

### Mandatory services

- **MariaDB** as the database backend
- **WordPress + PHP-FPM** as the application layer
- **Nginx with TLS** as the public HTTPS entrypoint

### Bonus services

- **Redis** for WordPress object caching
- **Adminer** for database administration
- **FTP** for file transfers into the shared WordPress volume
- **A static website** served separately on port `8080`
- **Netdata** for runtime monitoring

The project demonstrates container image creation, networking, persistent storage, service orchestration, automated bootstrap scripts, and separation of concerns.

## Project overview

### Services included in this stack

| Service | Type | Purpose | Internal/External access |
| --- | --- | --- | --- |
| `nginx` | Mandatory | Terminates TLS and serves the WordPress site | Host port `443` |
| `wordpress` | Mandatory | Runs WordPress on PHP-FPM | Internal only |
| `mariadb` | Mandatory | Stores WordPress data | Internal only |
| `redis` | Bonus | Provides object cache for WordPress | Internal only |
| `adminer` | Bonus | Web database administration tool | Available through Nginx at `/adminer/` |
| `ftp` | Bonus | Uploads and manages WordPress files in the shared volume | Host ports `21` and `21100-21110` |
| `static_site` | Bonus | Standalone static page | Host port `8080` |
| `netdata` | Bonus | Monitoring dashboard | Host port `19999` |

### Source layout

| Path | Role |
| --- | --- |
| `Makefile` | Main entrypoint for build, start, stop, cleanup |
| `clean.sh` | Aggressive cleanup helper |
| `srcs/docker-compose.yml` | Service orchestration, volumes, networks, ports |
| `srcs/requirements/mariadb/` | Mandatory MariaDB image, config, bootstrap script |
| `srcs/requirements/wordpress/` | Mandatory PHP-FPM/WordPress image and automated installation |
| `srcs/requirements/nginx/` | Mandatory HTTPS reverse proxy and TLS certificate generation |
| `srcs/requirements/bonus/adminer/` | Bonus Adminer image |
| `srcs/requirements/bonus/redis/` | Bonus Redis image |
| `srcs/requirements/bonus/ftp/` | Bonus FTP image and `vsftpd` configuration |
| `srcs/requirements/bonus/static_site/` | Bonus standalone Nginx static page |
| `srcs/requirements/bonus/netdata/` | Bonus monitoring service |

## Docker usage in this project

Docker is used to package each service with its own filesystem, process space, and startup logic. Instead of installing MariaDB, PHP, Nginx, Redis, and monitoring tools directly on the host, the project builds one image per service and connects them with Docker Compose.

Key effects of this design:

- each service can be rebuilt independently
- dependencies stay isolated
- networking is explicit
- persistent data is stored through Docker volumes backed by host directories
- the full stack can be started with one command

### Main design choices

- **One service per container** to keep responsibilities clean and aligned with the project subject.
- **Custom Dockerfiles** for all services instead of prebuilt all-in-one images.
- **Bridge networking** so containers can resolve each other by service name such as `mariadb`, `wordpress`, and `redis`.
- **Persistent storage** through named volumes mapped to host directories under `/home/gsanin-m/data`.
- **Automated bootstrap** scripts for MariaDB and WordPress so the stack can configure itself on first launch.
- **TLS-only public entrypoint** through Nginx on port `443`.
- **Adminer behind Nginx** so database administration stays inside the same HTTPS-facing entrypoint instead of exposing another host port.
- **Redis cache integration** enabled during WordPress installation with `wp-cli`.

## Comparison of required concepts

### Virtual Machines vs Docker

| Virtual Machines | Docker |
| --- | --- |
| Virtualize full operating systems | Share the host kernel and isolate processes |
| Heavier in disk, RAM, and boot time | Lightweight and fast to start |
| Good for full OS isolation and mixed kernels | Best when services can run as isolated processes on the same kernel |
| Usually managed as larger long-lived environments | Better suited to disposable, reproducible service units |

For this project, Docker is the better fit because the objective is to split a web stack into small cooperating services, rebuild them quickly, and keep the environment reproducible.

### Secrets vs Environment Variables

| Secrets | Environment Variables |
| --- | --- |
| Intended for sensitive data such as passwords and API keys | Generic runtime configuration mechanism |
| Better for production security workflows | Simpler for learning projects and local orchestration |
| Can avoid exposing values in process listings or logs depending on platform | Easier to read, inject, and debug |
| Preferred in hardened deployments | Often acceptable for local/dev setups |

This project currently uses **environment variables** through `env_file`. That keeps setup simple, but it is less secure than dedicated secret management. In a production version, database passwords and FTP credentials should be provided through Docker secrets or an external secret manager.

### Docker Network vs Host Network

| Docker bridge network | Host network |
| --- | --- |
| Containers communicate on an isolated virtual network | Container shares the host network stack |
| Service discovery by container name | No built-in name-based isolation |
| Safer default separation | Less isolation and higher port collision risk |
| Easier multi-service Compose topology | Useful mainly for special performance or low-level network cases |

This stack uses a **bridge network** named `inception`, which is the right choice for controlled service-to-service communication.

### Docker Volumes vs Bind Mounts

| Docker volumes | Bind mounts |
| --- | --- |
| Managed by Docker | Managed directly by the host filesystem |
| Portable and easy for container data | Useful when host path visibility matters |
| Better abstraction | Stronger host coupling |
| Can still be backed by a host path through driver options | Direct path mapping with fewer abstractions |

This repository uses **named Docker volumes backed by host directories**. That is a hybrid approach: Compose defines named volumes, but `driver_opts` bind them to `/home/gsanin-m/data/...` so the data is persistent and still easy to inspect from the host.

## Instructions

### Prerequisites

Install the following on the host:

- Docker Engine
- Docker Compose plugin
- `make`
- `sudo` access
- Internet access for package downloads during image builds

The `Makefile` also installs `curl` and `lftp` automatically with `apt-get` when running `make`.

### Configuration

Create a file named `srcs/.env` before starting the stack.

Example template:

```env
DOMAIN_NAME=gsanin-m.42.fr
SITE_TITLE=Inception

SQL_DATABASE=wordpress
SQL_USER=wp_user
SQL_PASSWORD=change_me_wp_password
SQL_ROOT_PASSWORD=change_me_root_password

ADMIN_USER=wp_admin
ADMIN_PASSWORD=change_me_admin_password
ADMIN_EMAIL=admin@example.com

USER_LOGIN=user42
USER_PASSWORD=change_me_user_password
USER_EMAIL=user42@example.com

FTP_USER=ftpuser
FTP_PASS=change_me_ftp_password
```

Also add the domain to `/etc/hosts` so the browser resolves it locally:

```text
127.0.0.1 gsanin-m.42.fr
```

If you change `DOMAIN_NAME`, keep it aligned with the Nginx `server_name` and certificate subject.

### Build and run

From the repository root:

```bash
make
```

What this does:

1. installs helper tools (`lftp`, `curl`)
2. creates persistent data directories under `/home/gsanin-m/data`
3. builds all images
4. starts the full stack in detached mode

### Stop the project

```bash
make down
```

### Clean Docker artifacts

```bash
make clean
```

### Full cleanup including persistent data

```bash
make fclean
```

> Warning: `make fclean` removes `/home/gsanin-m/data` and then removes **all Docker volumes present on the machine**, not only the project volumes.

### Access points

- WordPress site: `https://gsanin-m.42.fr`
- WordPress admin dashboard: `https://gsanin-m.42.fr/wp-admin`
- Adminer: `https://gsanin-m.42.fr/adminer/`
- Static site: `http://localhost:8080`
- Netdata: `http://localhost:19999`
- FTP: host `localhost`, port `21`, passive ports `21100-21110`

## Additional notes

### Persistent data

The project stores persistent data on the host under:

- `/home/gsanin-m/data/mariadb`
- `/home/gsanin-m/data/wordpress`
- `/home/gsanin-m/data/adminer`

### Runtime behavior

- MariaDB initializes its data directory and creates the configured database/user on first boot.
- WordPress waits for MariaDB, downloads the core files, creates `wp-config.php`, installs the site, creates the second user, and enables Redis cache.
- Nginx serves HTTPS with a self-signed certificate.
- Adminer is not directly exposed on a dedicated host port; it is reverse-proxied through Nginx.

## Resources

### Classic references

- [Docker documentation](https://docs.docker.com/)
- [Docker Compose file reference](https://docs.docker.com/reference/compose-file/)
- [Nginx documentation](https://nginx.org/en/docs/)
- [MariaDB documentation](https://mariadb.com/kb/en/documentation/)
- [WordPress documentation](https://wordpress.org/documentation/)
- [WP-CLI documentation](https://developer.wordpress.org/cli/commands/)
- [Redis documentation](https://redis.io/docs/)
- [Adminer project page](https://www.adminer.org/)
- [vsftpd documentation](https://security.appspot.com/vsftpd.html)
- [Netdata documentation](https://learn.netdata.cloud/docs/)

### AI usage disclosure

AI assistance was used for this documentation task to:

- inspect the repository structure
- summarize the role of each container
- draft and refine `README.md`, `USER_DOC.md`, and `DEV_DOC.md`
- improve wording, organization, and readability in English

No extra project features were added as part of this documentation pass.