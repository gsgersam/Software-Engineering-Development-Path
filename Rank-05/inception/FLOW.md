# INCEPTION PROJECT FLOW

## Purpose of This Document

This file explains, step by step and in detail, how this project flows from when you execute `make` until the services are running. It also describes how `docker-compose.yml`, the `Dockerfile`s, startup scripts, the network, volumes, and environment variables relate to each other.

---

## 1. Project Overview

This project sets up an infrastructure based on Docker containers. Each service has a concrete responsibility:

### Mandatory Services

- `mariadb`: database
- `wordpress`: PHP-FPM application where WordPress runs
- `nginx`: HTTPS web server that exposes WordPress to the outside world

### Bonus Services

- `redis`: cache for WordPress
- `adminer`: web panel for managing the database
- `ftp`: FTP access to WordPress shared files
- `static_site`: bonus static site
- `netdata`: bonus monitoring

The mandatory project flow revolves around `mariadb`, `wordpress`, and `nginx`. The other containers extend the practice with bonus features.

The basic idea is this:

1. `docker compose` orchestrates all services.
2. Each service is built from its own `Dockerfile`.
3. Some services use scripts (`setup.sh`) to initialize data the first time.
4. All containers connect to the same Docker network.
5. Important data is saved in persistent volumes.
6. `nginx` is the entry point for the main site.

---

## 2. Real Entry Point: What Happens When You Run `make`

In this project, the flow actually begins in the `Makefile` at the root.

### 2.1 Main Rule

When you run:

```bash
make
```

the `all` rule is executed, which depends on `inception`.

### 2.2 What That Rule Does

The `Makefile` performs these actions in order:

1. installs auxiliary tools on the host:
   - `lftp`
   - `curl`
2. creates persistent folders on the host:
   - `/home/gsanin-m/data/mariadb`
   - `/home/gsanin-m/data/wordpress`
   - `/home/gsanin-m/data/adminer`
3. executes:

```bash
docker compose -f srcs/docker-compose.yml up -d --build
```

That command is the heart of the startup.

---

## 3. The Role of `docker-compose.yml`

The file [srcs/docker-compose.yml](srcs/docker-compose.yml) is the orchestration conductor.

Its job is to:

- define what services exist
- indicate how each image is built
- connect containers to each other
- define ports exposed to the host
- mount persistent volumes
- load variables from `.env`
- decide startup dependencies
- automatically restart services with `restart: always`

### 3.1 What `docker-compose.yml` Does NOT Do

`docker-compose.yml` does not install packages inside containers by itself.

The `Dockerfile`s do that.

### 3.2 What It DOES Do

Compose tells Docker:

- "for this service, use this context and this Dockerfile"
- "build the image"
- "create the container"
- "inject these environment variables"
- "mount these volumes for it"
- "connect it to this network"
- "launch it with this `ENTRYPOINT` or `CMD`"

In summary:

- `Dockerfile` = how the image is manufactured
- `docker-compose.yml` = how those images are used and connected

---

## 4. Relationship Between `docker-compose.yml` and `Dockerfile`s

This is the most important relationship in the project.

### 4.1 Conceptual Flow

For each service, Compose reads something like:

- `build: ./requirements/mariadb`
- `build: ./requirements/wordpress`
- `build: ./requirements/nginx`

Or, in some bonus services:

- `context: .`
- `dockerfile: requirements/bonus/ftp/Dockerfile`

With that information, Docker does this:

1. enters the build context
2. locates the `Dockerfile`
3. executes its instructions line by line:
   - `FROM`
   - `RUN`
   - `COPY`
   - `EXPOSE`
   - `ENTRYPOINT`
   - `CMD`
4. generates a final image
5. Compose uses that image to create the container

### 4.2 Difference Between Image and Container

It's worth distinguishing them:

- **Image**: immutable template built from a `Dockerfile`
- **Container**: running instance of that image

The image is built first.
Then the container is created and executed.

---

## 5. Detailed Flow by Phases

# PHASE 1: Host Preparation

Before Docker launches anything, the host is already involved.

### 5.1 Persistent Folders

The `Makefile` creates host folders so that data survives even if you destroy the containers.

This is important because:

- MariaDB needs to persist the database
- WordPress needs to persist its files, plugins, uploads, and `wp-config.php`
- Adminer has its own configured volume

### 5.2 `.env` File

In `docker-compose.yml` you see `env_file: .env` for some services.

That means Compose expects to find a file:

- `srcs/.env`

That file contains variables like:

- `SQL_DATABASE`
- `SQL_USER`
- `SQL_PASSWORD`
- `SQL_ROOT_PASSWORD`
- `DOMAIN_NAME`
- `ADMIN_USER`
- `ADMIN_PASSWORD`
- `FTP_USER`
- `FTP_PASS`

Compose doesn't invent those values. It just loads and injects them into containers that need them.

---

# PHASE 2: Image Building

Now Compose starts building each service.

## 6. `mariadb` Service

### 6.1 Which Dockerfile It Uses

Compose uses [srcs/requirements/mariadb/Dockerfile](srcs/requirements/mariadb/Dockerfile).

### 6.2 What That Dockerfile Does

Step by step:

1. starts from `debian:bookworm`
2. installs `mariadb-server`
3. creates `/var/run/mysqld`
4. adjusts permissions for the `mysql` user
5. copies custom configuration:
   - [srcs/requirements/mariadb/conf/50-server.cnf](srcs/requirements/mariadb/conf/50-server.cnf)
6. copies the startup script:
   - [srcs/requirements/mariadb/tools/setup.sh](srcs/requirements/mariadb/tools/setup.sh)
7. gives execute permissions to the script
8. exposes port `3306`
9. defines the `ENTRYPOINT` as the `setup.sh` script

### 6.3 What This Means in Practice

When the `mariadb` container starts, it doesn't directly execute `mysqld`.
It first enters `setup.sh`.

### 6.4 What MariaDB's `setup.sh` Does

That script controls the real initialization:

1. checks if `/var/lib/mysql/mysql` exists
2. if it doesn't, initializes the data directory
3. starts MariaDB in temporary safe mode:
   - without network
   - without active permission tables
4. waits for MariaDB to respond
5. generates temporary SQL with environment variables
6. creates:
   - the database
   - the normal user
   - the permissions
   - the `root` password
7. shuts down the temporary instance
8. launches MariaDB in normal mode with `exec mysqld_safe`

### 6.5 Conclusion of MariaDB Flow

The `Dockerfile` prepares the system.
The `setup.sh` executes the first-startup logic.
The volume persists the data for future restarts.

---

## 7. `wordpress` Service

### 7.1 Which Dockerfile It Uses

Compose uses [srcs/requirements/wordpress/Dockerfile](srcs/requirements/wordpress/Dockerfile).

### 7.2 What This Dockerfile Does

Step by step:

1. starts from `debian:bookworm`
2. installs:
   - `wget`
   - `php-fpm`
   - `php-mysql`
   - `mariadb-client`
   - Redis extension for PHP
3. downloads `wp-cli`
4. makes it executable
5. creates `/run/php`
6. copies PHP-FPM configuration:
   - [srcs/requirements/wordpress/conf/www.conf](srcs/requirements/wordpress/conf/www.conf)
7. copies the script:
   - [srcs/requirements/wordpress/tools/setup.sh](srcs/requirements/wordpress/tools/setup.sh)
8. defines `/var/www/html` as `WORKDIR`
9. defines the `ENTRYPOINT` as `setup.sh`

### 7.3 What Compose Does With This Service

Compose also:

- injects variables from `.env`
- mounts the `wordpress_data` volume to `/var/www/html`
- connects it to the `inception` network
- makes it depend on `mariadb`

### 7.4 What WordPress's `setup.sh` Does

The WordPress script is the brain of application deployment.

It does this:

1. waits until `mariadb` responds with `mysqladmin ping`
2. enters `/var/www/html`
3. checks if `wp-config.php` already exists

#### If It Doesn't Exist:

4. cleans the folder
5. downloads WordPress core with `wp core download`
6. creates `wp-config.php` with the variables:
   - database
   - user
   - password
   - host `mariadb`
7. installs WordPress with:
   - URL
   - site title
   - admin user
   - admin password
   - admin email
8. creates the mandatory second user
9. installs and activates the `redis-cache` plugin
10. adds Redis configuration to `wp-config.php`
11. enables Redis Object Cache
12. fixes permissions on `/var/www/html`
13. starts PHP-FPM in foreground with `php-fpm8.2 -F`

#### If It Does Exist:

- understands that WordPress was already configured before
- skips the initial setup
- starts PHP-FPM directly

### 7.5 Importance of Volume in WordPress

The volume is fundamental here.

Thanks to `wordpress_data`, when you restart the container:

- the setup is not lost
- `wp-config.php` is not lost
- plugins and uploads are not lost

---

## 8. `nginx` Service

### 8.1 Which Dockerfile It Uses

Compose uses [srcs/requirements/nginx/Dockerfile](srcs/requirements/nginx/Dockerfile).

### 8.2 What That Dockerfile Does

1. starts from `debian:bookworm`
2. installs `nginx` and `openssl`
3. creates a self-signed certificate
4. copies the Nginx configuration:
   - [srcs/requirements/nginx/conf/nginx.conf](srcs/requirements/nginx/conf/nginx.conf)
5. exposes port `443`
6. starts Nginx in foreground with `daemon off;`

### 8.3 What Compose Does for Nginx

Compose also:

- mounts the same `wordpress_data` volume to `/var/www/html`
- publishes `443:443`
- connects Nginx to the `inception` network
- specifies dependency on `wordpress`

### 8.4 How Nginx Interacts With WordPress

Nginx does not execute PHP.
It only receives HTTPS requests and decides what to do with them.

When the browser requests:

- `https://gsanin-m.42.fr`

Nginx:

1. accepts the SSL connection
2. uses the root `/var/www/html`
3. if the request is for a PHP file, forwards it via FastCGI to:
   - `wordpress:9000`
4. PHP-FPM in the `wordpress` container processes the script
5. the response comes back to Nginx
6. Nginx delivers it to the browser

### 8.5 How Nginx Interacts With Adminer

In the configuration appears:

- `/adminer/`

That route is not served directly from Nginx itself.
It uses `proxy_pass` towards:

- `http://adminer:8080/`

That is:

1. the browser enters through Nginx
2. Nginx detects the `/adminer/` route
3. forwards the request to the `adminer` container
4. Adminer responds
5. Nginx returns the response to the browser

---

## 9. `adminer` Service

### 9.1 Which Dockerfile It Uses

Compose uses [srcs/requirements/bonus/adminer/Dockerfile](srcs/requirements/bonus/adminer/Dockerfile).

### 9.2 What That Dockerfile Does

1. uses `alpine`
2. installs PHP and necessary extensions
3. downloads the single Adminer PHP file
4. places it in `/var/www/index.php`
5. exposes port `8080`
6. starts the embedded PHP server

### 9.3 How Traffic Enters

Adminer is not published directly to the host in Compose.
It's accessed through Nginx:

- `https://gsanin-m.42.fr/adminer/`

That's important because Nginx acts as the central entry point.

---

## 10. `redis` Service

### 10.1 Which Dockerfile It Uses

Compose uses [srcs/requirements/bonus/redis/Dockerfile](srcs/requirements/bonus/redis/Dockerfile).

### 10.2 What That Dockerfile Does

1. starts from `alpine`
2. installs Redis
3. modifies `redis.conf` to:
   - listen on `0.0.0.0`
   - disable `protected-mode`
4. exposes `6379`
5. starts `redis-server`

### 10.3 How It Interacts With WordPress

Redis does not receive browser requests.
It receives requests from WordPress.

Flow:

1. WordPress installs and activates the `redis-cache` plugin
2. WordPress writes to `wp-config.php`:
   - Redis host = `redis`
   - port = `6379`
3. when WordPress needs object cache, it connects to the `redis` container
4. Redis temporarily stores that data in memory

### 10.4 What the Project Gains From This

- fewer repeated database queries
- better response time in certain operations
- real integration between internal services

---

## 11. `ftp` Service

### 11.1 Which Dockerfile It Uses

Compose uses [srcs/requirements/bonus/ftp/Dockerfile](srcs/requirements/bonus/ftp/Dockerfile).

### 11.2 What That Dockerfile Does

1. uses `alpine`
2. installs `vsftpd`
3. copies:
   - [srcs/requirements/bonus/ftp/conf/vsftpd.conf](srcs/requirements/bonus/ftp/conf/vsftpd.conf)
   - [srcs/requirements/bonus/ftp/tools/setup.sh](srcs/requirements/bonus/ftp/tools/setup.sh)
4. gives permissions to the script
5. exposes:
   - `21`
   - `21100-21110`
6. uses the script as `ENTRYPOINT`

### 11.3 What the `setup.sh` Script Does

1. checks if the FTP user already exists
2. if it doesn't, creates it with `FTP_USER` and `FTP_PASS`
3. adds it to the `www-data` group
4. changes ownership of `/var/www/html`
5. starts `vsftpd`

### 11.4 How It Interacts With WordPress

The key here is the shared volume.

`ftp` mounts the same volume as `wordpress`:

- `wordpress_data:/var/www/html`

That means that:

- if you upload a file via FTP
- WordPress sees it within its own filesystem
- and Nginx can serve it if appropriate

In other words, FTP doesn't "talk" to WordPress via HTTP or PHP.
It interacts with the same persistent storage.

---

## 12. `static_site` Service

### 12.1 Which Dockerfile It Uses

Compose uses [srcs/requirements/bonus/static_site/Dockerfile](srcs/requirements/bonus/static_site/Dockerfile).

### 12.2 What It Does

1. uses `alpine`
2. installs `nginx`
3. generates minimal configuration on the fly
4. copies `index.html`
5. exposes `8080`
6. starts Nginx in foreground

### 12.3 How It Works Within the Project

It's an independent service from the main stack.
It doesn't participate in the WordPress → MariaDB flow.
It simply demonstrates an additional bonus by serving a static page.

---

## 13. `netdata` Service

### 13.1 Which Dockerfile It Uses

Compose uses [srcs/requirements/bonus/netdata/Dockerfile](srcs/requirements/bonus/netdata/Dockerfile).

### 13.2 What It Does

1. uses `debian:bookworm`
2. installs `netdata`
3. adjusts its configuration to listen on all IPs
4. exposes `19999`
5. starts Netdata

### 13.3 How It Interacts With the Host and Docker

Compose mounts for it:

- `/proc`
- `/sys`
- `/var/run/docker.sock`

That allows it to observe system state and part of the Docker environment.

---

# PHASE 3: Network and Volume Creation

## 14. The Docker Network `inception`

Compose defines a network:

- `inception`

Type:

- `bridge`

### 14.1 What This Means

All containers connected to that network can resolve each other by name.

Examples:

- `wordpress` can connect to `mariadb`
- `wordpress` can connect to `redis`
- `nginx` can connect to `wordpress`
- `nginx` can connect to `adminer`

### 14.2 Why This Is Important

There's no need for fixed IPs.
Services discover each other by service name defined in Compose.

---

## 15. The Volumes

Compose defines three main volumes:

- `mariadb_data`
- `wordpress_data`
- `adminer_data`

But they're not just anonymous Docker volumes.
They've been configured with `driver_opts` to point to real host paths.

### 15.1 Conceptual Example

`mariadb_data` ends up storing data in:

- `/home/gsanin-m/data/mariadb`

`wordpress_data` in:

- `/home/gsanin-m/data/wordpress`

### 15.2 Practical Result

Even though the container is destroyed:

- the database remains on disk
- WordPress files remain on disk
- initial configuration is not lost

---

# PHASE 4: Container Startup

## 16. Logical Startup Order

Although Compose can launch multiple services, the logical flow is this:

1. images are built
2. network and volumes are created
3. `mariadb` starts
4. `wordpress` starts
5. `wordpress` waits for `mariadb`
6. `nginx` starts
7. bonus services start: `adminer`, `redis`, `ftp`, `static_site`, `netdata`

### 16.1 Important Note About `depends_on`

`depends_on` doesn't guarantee that a service is completely ready internally.
It only helps with startup order.

That's why the WordPress script does real active waiting against MariaDB.

---

# PHASE 5: Flow of a Real Web Request

## 17. Example: A User Enters the Main Site

Suppose the user opens:

- `https://gsanin-m.42.fr`

The exact flow is:

1. the browser resolves the domain to `127.0.0.1`
2. connects to port `443` of the host
3. Docker redirects that connection to the `nginx` container
4. Nginx receives the HTTPS request
5. if the request requires PHP, Nginx sends it to `wordpress:9000`
6. PHP-FPM executes WordPress
7. WordPress queries MariaDB if it needs data
8. WordPress queries Redis if the cache plugin can serve or store objects
9. PHP-FPM returns the result to Nginx
10. Nginx responds to the browser

---

## 18. Example: Accessing Adminer

1. browser → `https://gsanin-m.42.fr/adminer/`
2. host `443` → `nginx` container
3. Nginx detects the `/adminer/` route
4. Nginx proxies to `adminer:8080`
5. Adminer responds with the web interface
6. from Adminer you can connect to `mariadb`

---

## 19. Example: File Upload via FTP

1. an FTP client connects to the host on port `21`
2. Docker forwards to the `ftp` container
3. `vsftpd` authenticates using `FTP_USER` and `FTP_PASS`
4. the user uploads files to `/var/www/html`
5. that path is within the shared `wordpress_data` volume
6. WordPress sees those files
7. Nginx can serve them if they're part of the web content

---

## 20. Example: Caching With Redis

1. WordPress processes a page or internal query
2. the Redis plugin decides to save or read cached objects
3. WordPress connects to `redis:6379`
4. Redis responds from memory
5. WordPress reduces some of the load towards MariaDB

---

# PHASE 6: Persistence and Restarts

## 21. What Happens if You Run `make down`

- containers are stopped
- the network may disappear if Compose removes it
- volumes remain
- data remains intact in `/home/gsanin-m/data/...`

When you run `make` again:

- MariaDB will detect that data already exists
- WordPress will see `wp-config.php`
- the initial setup won't be repeated

## 22. What Happens if You Run `make fclean`

- containers are stopped
- Docker resources are cleaned up
- `/home/gsanin-m/data` is deleted
- Docker volumes are removed

That leaves the project practically like a fresh installation.

---

## 23. General ASCII FLOW

```text
                           +----------------------+
                           |      make / re       |
                           +----------+-----------+
                                      |
                                      v
                        +-----------------------------+
                        | Makefile                     |
                        | - installs curl/lftp        |
                        | - creates /home/.../data    |
                        | - launches docker compose   |
                        +--------------+--------------+
                                       |
                                       v
                    +---------------------------------------+
                    | srcs/docker-compose.yml               |
                    | - defines services                   |
                    | - image building                     |
                    | - inception network                  |
                    | - persistent volumes                 |
                    | - ports and env_file                 |
                    +----------------+----------------------+
                                     |
          +--------------------------+---------------------------+
          |                          |                           |
          v                          v                           v
+-------------------+    +--------------------+      +--------------------+
| Dockerfile DB     |    | Dockerfile WP      |      | Dockerfile Nginx   |
| mariadb           |    | wordpress          |      | nginx              |
+---------+---------+    +---------+----------+      +---------+----------+
          |                        |                           |
          v                        v                           v
+-------------------+    +--------------------+      +--------------------+
| setup.sh DB       |    | setup.sh WP        |      | nginx.conf         |
| init DB + users   |    | install WP         |      | TLS + FastCGI      |
| root password     |    | create users       |      | proxy /adminer/    |
+---------+---------+    | enable redis       |      +---------+----------+
          |              +---------+----------+                |
          |                        |                           |
          +------------+-----------+---------------------------+
                       | 
                       v
              +-------------------+
              |  Network: inception|
              |  bridge network   |
              +---+---+---+---+---+
                  |   |   |   |   |
                  |   |   |   |   +--------------------+
                  |   |   |   |                        |
                  v   v   v   v                        v
            +-----+ +---+ +------+              +-------------+
            | DB  | | WP| |Redis |              |  Adminer    |
            +--+--+ +-+-+ +--+---+              +------+------+
               ^      |      ^                         ^
               |      |      |                         |
               |      +------+-------------------------+
               |             |
               |             v
               |      +-------------+
               |      |   Nginx     |
               |      | host:443    |
               |      +------+------+ 
               |             |
               |             v
               |      +-------------+
               |      |  Browser    |
               |      +-------------+
               |
               +--------------------------------------+
                                                      |
                         +----------------------------+------------------+
                         |                                               |
                         v                                               v
                +------------------+                           +------------------+
                | ftp              |                           | static_site      |
                | host:21          |                           | host:8080        |
                | same WP volume   |                           | independent      |
                +------------------+                           +------------------+

                         +-------------------------------------+
                         | netdata host:19999                  |
                         | system/stack monitoring             |
                         +-------------------------------------+
```

---

## 24. ASCII FLOW OF WEB REQUEST

```text
Browser
  |
  | HTTPS request to gsanin-m.42.fr:443
  v
Host port 443
  |
  v
Docker port mapping
  |
  v
nginx container
  |
  |-- if path is /adminer/ ------------> adminer:8080
  |
  |-- if PHP page ---------------------> wordpress:9000 (PHP-FPM)
                                           |
                                           |-- query data --> mariadb:3306
                                           |
                                           |-- cache read/write --> redis:6379
                                           |
                                           v
                                      generated HTML
  |
  v
Browser response
```

---

## 25. ASCII FLOW OF PERSISTENT DATA

```text
Host filesystem
  |
  +-- /home/gsanin-m/data/mariadb   <---- volume mariadb_data  <---- mariadb:/var/lib/mysql
  |
  +-- /home/gsanin-m/data/wordpress <---- volume wordpress_data <---- wordpress:/var/www/html
  |                                                           \
  |                                                            +---- nginx:/var/www/html
  |                                                            +---- ftp:/var/www/html
  |
  +-- /home/gsanin-m/data/adminer   <---- volume adminer_data  <---- adminer:/var/www/html
```

---

## 26. Final Summary of Flow

If the entire project were to be summarized in a simple chain, it would be this:

1. `make` prepares the host and calls Compose
2. Compose builds images using the `Dockerfile`s
3. each `Dockerfile` installs software, copies configs and defines the initial process
4. Compose creates network, volumes and containers
5. `setup.sh` scripts do the first-startup initialization
6. `nginx` exposes the site to the outside
7. `wordpress` processes the application logic
8. `mariadb` stores permanent data
9. `redis` speeds up certain operations
10. `ftp` and `nginx` share WordPress files via volume
11. `adminer` allows database administration via web
12. `netdata` observes the system
13. volumes guarantee persistence between restarts

---

## 27. Key Idea to Understanding the Entire Project

The most correct way to understand it is this:

- **Compose decides who exists and how they connect**
- **each Dockerfile decides how each service is built**
- **each setup script decides how the service initializes on startup**
- **the network allows internal communication by name**
- **volumes make data survive**
- **Nginx centralizes the web entry point**

If you understand that chain, you understand the complete flow of the project.
