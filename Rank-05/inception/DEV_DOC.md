# DEV_DOC

## Purpose of this document

This file describes how a developer can prepare, configure, build, run, inspect, and clean the Inception project from scratch.

## 1. Prerequisites

Install these tools on the host machine:

- Docker Engine
- Docker Compose plugin
- GNU Make
- `sudo`
- Git

Recommended checks:

```bash
docker --version
docker compose version
make --version
```

## 2. Repository layout

The mandatory part of the project is built around `mariadb`, `wordpress`, and `nginx`. The remaining services are bonus features.

| Path | Meaning |
| --- | --- |
| `Makefile` | Main automation entrypoint |
| `clean.sh` | Extra full-clean helper |
| `srcs/docker-compose.yml` | Multi-container definition |
| `srcs/requirements/mariadb/` | Mandatory MariaDB image, config, database bootstrap |
| `srcs/requirements/wordpress/` | Mandatory PHP-FPM image and WordPress install logic |
| `srcs/requirements/nginx/` | Mandatory HTTPS frontend |
| `srcs/requirements/bonus/adminer/` | Bonus Adminer service |
| `srcs/requirements/bonus/redis/` | Bonus Redis service |
| `srcs/requirements/bonus/ftp/` | Bonus FTP service and `vsftpd` config |
| `srcs/requirements/bonus/static_site/` | Bonus static website service |
| `srcs/requirements/bonus/netdata/` | Bonus monitoring service |

## 3. Environment setup from scratch

### 3.1 Clone the repository

```bash
git clone <your-repository-url>
cd Inception
```

### 3.2 Review host-specific paths

This project stores persistent data under:

```text
/home/gsanin-m/data
```

That path is hardcoded in:

- `Makefile`
- `srcs/docker-compose.yml`

If you are running the project under another Linux user, update those paths before building.

### 3.3 Create the environment file

Create `srcs/.env`.

Suggested template:

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

Variables are consumed by:

- MariaDB bootstrap script
- WordPress installation script
- FTP user creation
- Nginx/WordPress public URL flow

### 3.4 Configure local DNS resolution

Add the project domain to `/etc/hosts`:

```text
127.0.0.1 gsanin-m.42.fr
```

If you change `DOMAIN_NAME`, keep these elements aligned:

- the `.env` value
- the Nginx `server_name`
- the certificate subject generated in the Nginx Dockerfile

## 4. Build and launch

### 4.1 Preferred workflow with Makefile

Start the full project from the repository root:

```bash
make
```

The `Makefile` does the following:

1. installs `lftp` and `curl`
2. creates host data directories
3. runs `docker compose -f srcs/docker-compose.yml up -d --build`

Available targets:

```bash
make          # build and start
make down     # stop containers
make clean    # stop and prune docker artifacts
make fclean   # remove data path and docker volumes
make re       # full rebuild from scratch
```

### 4.2 Direct Docker Compose workflow

If you want to bypass the `Makefile`:

```bash
docker compose -f srcs/docker-compose.yml up -d --build
```

Stop only:

```bash
docker compose -f srcs/docker-compose.yml down
```

Inspect status:

```bash
docker compose -f srcs/docker-compose.yml ps
```

## 5. Service orchestration details

### 5.1 Startup flow

1. `mariadb` starts and initializes `/var/lib/mysql` if needed.
2. `wordpress` waits until MariaDB answers to `mysqladmin ping`.
3. `wordpress` installs WordPress with `wp-cli` on first boot.
4. `wordpress` creates the admin account and a second user.
5. `wordpress` installs and enables the Redis Cache plugin.
6. `nginx` serves the shared WordPress volume over HTTPS.
7. Bonus services join the same bridge network as required.

### 5.2 Networking

Compose defines one bridge network:

- `inception`

Containers communicate by service name, for example:

- `wordpress` reaches the database at `mariadb`
- `nginx` forwards PHP requests to `wordpress:9000`
- `nginx` proxies `/adminer/` to `adminer:8080`
- `wordpress` connects to Redis at `redis:6379`

### 5.3 Volumes and persistence

Compose defines named volumes backed by host directories:

| Volume | Container path | Host path |
| --- | --- | --- |
| `mariadb_data` | `/var/lib/mysql` | `/home/gsanin-m/data/mariadb` |
| `wordpress_data` | `/var/www/html` | `/home/gsanin-m/data/wordpress` |
| `adminer_data` | `/var/www/html` | `/home/gsanin-m/data/adminer` |

This means data survives container recreation as long as the host directories are preserved.

## 6. Configuration files and what they control

| File | Responsibility |
| --- | --- |
| `srcs/docker-compose.yml` | Services, networks, ports, volumes, restart policy |
| `srcs/requirements/mariadb/conf/50-server.cnf` | MariaDB bind address and runtime settings |
| `srcs/requirements/mariadb/tools/setup.sh` | Initial database creation and user provisioning |
| `srcs/requirements/wordpress/conf/www.conf` | PHP-FPM pool settings |
| `srcs/requirements/wordpress/tools/setup.sh` | WordPress install, users, Redis configuration |
| `srcs/requirements/nginx/conf/nginx.conf` | TLS vhost, FastCGI forwarding, Adminer reverse proxy |
| `srcs/requirements/bonus/ftp/conf/vsftpd.conf` | FTP runtime and passive mode settings |
| `srcs/requirements/bonus/ftp/tools/setup.sh` | FTP user creation and ownership handling |

## 7. Useful development commands

### Rebuild one service

```bash
docker compose -f srcs/docker-compose.yml build wordpress
```

### Restart one service

```bash
docker compose -f srcs/docker-compose.yml restart nginx
```

### Follow logs

```bash
docker compose -f srcs/docker-compose.yml logs -f
```

Single service logs:

```bash
docker compose -f srcs/docker-compose.yml logs -f mariadb
```

### Open a shell inside a container

```bash
docker exec -it wordpress sh
docker exec -it mariadb sh
docker exec -it nginx sh
```

### Inspect volumes

```bash
docker volume ls
docker volume inspect inception_mariadb_data
```

Volume names may vary depending on the Compose project name.

### Inspect containers and networks

```bash
docker ps
docker network ls
docker network inspect srcs_inception
```

Network names may also vary with the Compose project name.

## 8. Data storage and persistence model

### Database data

MariaDB stores its database files under the `mariadb_data` volume, which is backed by `/home/gsanin-m/data/mariadb`.

### WordPress application data

WordPress core files, plugins, themes, uploads, and generated configuration persist in `wordpress_data`, backed by `/home/gsanin-m/data/wordpress`.

### Adminer files

Adminer content is stored in `adminer_data`, backed by `/home/gsanin-m/data/adminer`.

### What survives a restart

These survive `docker compose down` and container rebuilds:

- MariaDB data
- WordPress files
- uploaded content
- installed plugins and generated `wp-config.php`

These do **not** survive `make fclean`:

- all files inside `/home/gsanin-m/data`
- Docker volumes removed by the cleanup target

## 9. Notes about secrets and configuration

The project currently uses environment variables from `srcs/.env` instead of Docker secrets.

Implications:

- easier local development
- simple bootstrap scripts
- weaker security posture than a real secret-management workflow

For a more production-oriented variant, move passwords to Docker secrets or an external vault and mount them at runtime.

## 10. Validation checklist for developers

Before presenting the project, confirm that:

- all containers build from their own Dockerfiles
- the website opens on HTTPS only
- WordPress works with MariaDB
- Redis is enabled in WordPress
- Adminer is reachable through `/adminer/`
- FTP access works with the configured credentials
- the static site responds on port `8080`
- Netdata responds on port `19999`
- data persists after `docker compose down` and a restart

## 11. Cleanup and reset

### Standard stop

```bash
make down
```

### Docker cleanup

```bash
make clean
```

### Full reset

```bash
make fclean
```

> Important: `make fclean` removes the project data directory and then removes all Docker volumes currently present on the machine.

### Extra cleanup script

```bash
./clean.sh
```

This script is more aggressive. It stops every container, prunes Docker resources with volumes, removes `/home/gsanin-m/data`, and drops filesystem caches. Use it carefully.

## 12. Recommended improvement points

If you continue developing this repository, useful next steps would be:

- replace plaintext environment credentials with Docker secrets
- parameterize host-specific paths and domain names
- add healthchecks to more services
- add an `.env.example` file for onboarding
- add backup/restore scripts for MariaDB and WordPress content
- harden Adminer and FTP exposure for non-local environments