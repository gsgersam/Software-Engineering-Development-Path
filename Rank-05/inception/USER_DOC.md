# USER_DOC

## Purpose of this document

This file explains how to use the Inception stack as an end user or administrator after the project has already been configured.

## What services are provided

The stack includes a **mandatory core** and several **bonus services**.

| Service | Type | What it does | How you use it |
| --- | --- | --- | --- |
| WordPress | Mandatory | Main website and CMS | Visit the website and log in to manage content |
| Nginx | Mandatory | Public HTTPS web server | Handles browser access on port `443` |
| MariaDB | Mandatory | Database for WordPress | Usually used indirectly through WordPress or through bonus tools |
| Adminer | Bonus | Database administration UI | Open `/adminer/` in the browser |
| Redis | Bonus | Cache for WordPress | Improves performance automatically |
| FTP | Bonus | File transfer service | Upload or edit site files remotely |
| Static site | Bonus | Extra standalone page | Open port `8080` in the browser |
| Netdata | Bonus | Monitoring dashboard | Inspect runtime status and metrics |

## Start the project

From the repository root, run:

```bash
make
```

This command builds the images, creates the data directories, and starts the services in the background.

## Stop the project

To stop the containers without deleting persistent data:

```bash
make down
```

## Restart after a stop

To start everything again:

```bash
make
```

## Access the website and administration interfaces

Before opening the site, make sure the project domain resolves locally. Your host should contain an entry like:

```text
127.0.0.1 gsanin-m.42.fr
```

### Main website

Open:

- `https://gsanin-m.42.fr`

The TLS certificate is self-signed, so the browser may show a warning the first time. Accept the local exception and continue.

### WordPress administration panel

Open:

- `https://gsanin-m.42.fr/wp-admin`

Log in with the administrator account defined in `srcs/.env`:

- `ADMIN_USER`
- `ADMIN_PASSWORD`

### Adminer database panel

Open:

- `https://gsanin-m.42.fr/adminer/`

Use these values from `srcs/.env`:

- system: `MariaDB`
- server: `mariadb`
- username: `SQL_USER` or `root`
- password: `SQL_PASSWORD` or `SQL_ROOT_PASSWORD`
- database: `SQL_DATABASE`

### Static site

Open:

- `http://localhost:8080`

### Monitoring dashboard

Open:

- `http://localhost:19999`

### FTP access

Connect with any FTP client using:

- host: `localhost`
- port: `21`
- username: `FTP_USER`
- password: `FTP_PASS`

Passive mode uses ports `21100-21110`, so your client must allow that range.

## Where credentials are stored

All runtime credentials are stored in:

- `srcs/.env`

This file is expected to contain values such as:

- database name and passwords
- WordPress administrator credentials
- secondary WordPress user credentials
- FTP credentials
- public site domain

### Good practice

- never commit `srcs/.env`
- change default or weak passwords before presenting the project
- share credentials only with authorized evaluators or administrators

## How to check that services are running correctly

### Quick check

Run:

```bash
docker compose -f srcs/docker-compose.yml ps
```

You should see the containers in a running state.

### Browser checks

Confirm that these endpoints respond:

- `https://gsanin-m.42.fr`
- `https://gsanin-m.42.fr/wp-admin`
- `https://gsanin-m.42.fr/adminer/`
- `http://localhost:8080`
- `http://localhost:19999`

### HTTPS check from terminal

```bash
curl -k https://gsanin-m.42.fr
```

The `-k` flag is used because the certificate is self-signed.

### Service logs

If something does not work, inspect logs:

```bash
docker compose -f srcs/docker-compose.yml logs
```

For a single service:

```bash
docker compose -f srcs/docker-compose.yml logs nginx
```

Replace `nginx` with `wordpress`, `mariadb`, `redis`, `ftp`, `adminer`, `static_site`, or `netdata` as needed.

## Common user tasks

### Publish content in WordPress

1. Open `https://gsanin-m.42.fr/wp-admin`
2. Log in with the admin account
3. Create or edit posts, pages, users, themes, or plugins

### Inspect database content

1. Open `https://gsanin-m.42.fr/adminer/`
2. Log in with database credentials from `srcs/.env`
3. Review WordPress tables, exports, or manual SQL queries carefully

### Transfer files with FTP

Use your FTP client to upload files into the shared WordPress content directory. Because WordPress and FTP use the same persistent storage, transferred files remain available after container restarts.

## Cleanup warnings

### Standard stop

`make down` stops containers only.

### Strong cleanup

`make clean` removes unused Docker artifacts.

### Full reset

`make fclean` removes persistent data under `/home/gsanin-m/data` and also removes Docker volumes from the machine. Use it only when you really want a fresh start.

## Troubleshooting

### The site name does not resolve

Check `/etc/hosts` and confirm the project domain points to `127.0.0.1`.

### Browser shows certificate warning

Expected behavior. The certificate is self-signed.

### WordPress page is unavailable

Check whether `nginx`, `wordpress`, and `mariadb` are all running, then inspect logs.

### Adminer does not open

Confirm Nginx is running because Adminer is exposed through Nginx at `/adminer/`.

### FTP cannot connect

Check that ports `21` and `21100-21110` are reachable and verify the `FTP_USER` / `FTP_PASS` values in `srcs/.env`.