# Clion/PostgreSQL
[![Docker Pulls](https://img.shields.io/docker/pulls/clion007/postgresql.svg)](https://hub.docker.com/r/clion007/postgresql)
[![Docker Stars](https://img.shields.io/docker/stars/clion007/postgresql.svg)](https://hub.docker.com/r/clion007/postgresql)
[![GitHub Stars](https://img.shields.io/github/stars/clion007/docker-postgresql.svg)](https://github.com/clion007/docker-postgresql)
[![GitHub Last Commit](https://img.shields.io/github/last-commit/clion007/docker-postgresql.svg)](https://github.com/clion007/docker-postgresql/commits/main)
[![Build Status](https://img.shields.io/github/actions/workflow/status/clion007/docker-postgresql/docker-publish.yml?branch=main)](https://github.com/clion007/docker-postgresql/actions)
[![Image Size](https://img.shields.io/docker/image-size/clion007/postgresql/latest)](https://hub.docker.com/r/clion007/postgresql)

<div align="center">
  <img src="https://www.postgresql.org/media/img/about/pgElephant.png" alt="PostgreSQL Logo" width="280" height="280">
  <br><br>
  <strong>The World's Most Advanced Open Source Relational Database</strong>
</div>

<br>

PostgreSQL is a powerful, open source object-relational database system with over 35 years of active development that has earned it a strong reputation for reliability, feature robustness, and performance.

This clion007/postgresql docker image is built from source on the latest Alpine Linux with a slim footprint. It ships a custom-compiled en+zh ICU bundle so Chinese text sorts and collates correctly without the full ICU data set, defaults to the `Asia/Shanghai` timezone, and automatically tracks the latest stable PostgreSQL release. The image publishes to Docker Hub (`clion007/postgresql`) and Alibaba Cloud (`registry.cn-chengdu.aliyuncs.com/clion/postgresql`).

## 🚀 Application Setup

* PostgreSQL listens on port `5432` by default.
* On first start with an empty data directory, the container initializes a new database cluster and creates the superuser/database from the `POSTGRES_USER`, `POSTGRES_PASSWORD` and `POSTGRES_DB` variables.
* Any `*.sh`, `*.sql`, `*.sql.gz`, `*.sql.xz` or `*.sql.zst` scripts placed in `/docker-entrypoint-initdb.d` are executed once during initialization (in alphabetical order).
* Data lives in `/var/lib/postgresql/<PG_MAJOR>/docker` inside the container, so mount the parent `/var/lib/postgresql` directory. A single mount point keeps the database upgradeable with `pg_upgrade --link`.

## 📋 Usage

You can deploy this container using either docker-compose (recommended) or the docker CLI.

### Docker Compose (Recommended)

```yaml
services:
  postgresql:
    container_name: PostgreSQL
    image: registry.cn-chengdu.aliyuncs.com/clion/postgresql:latest
    environment:
      - POSTGRES_PASSWORD=changeme #required
      - POSTGRES_USER=postgres
      - POSTGRES_DB=postgres
      - PUID=70
      - PGID=70
      - UMASK=022
      - TZ=Asia/Shanghai
    ports:
      - 5432:5432
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /path/to/postgresql/data:/var/lib/postgresql
    restart: unless-stopped
```

### Docker CLI

```bash
docker run -d \
  --name=PostgreSQL \
  -e POSTGRES_PASSWORD=changeme `#required` \
  -e POSTGRES_USER=postgres \
  -e POSTGRES_DB=postgres \
  -e PUID=70 \
  -e PGID=70 \
  -e UMASK=022 \
  -e TZ=Asia/Shanghai \
  -p 5432:5432 \
  -v /path/to/postgresql/data:/var/lib/postgresql \
  -v /etc/localtime:/etc/localtime:ro \
  --restart unless-stopped \
  registry.cn-chengdu.aliyuncs.com/clion/postgresql:latest
```

## ⚙️ Parameters

Containers are configured using parameters passed at runtime. These parameters are separated by a colon and indicate `<external>:<internal>` respectively. For example, `-p 5432:5432` would expose port 5432 from inside the container to be accessible from the host's IP on port 5432.

### Port Mappings
| Parameter | Function |
| :----: | --- |
| `-p 5432:5432` | PostgreSQL TCP connection port |

### Environment Variables
| Parameter | Function |
| :----: | --- |
| `-e POSTGRES_PASSWORD=changeme` | Initial superuser password (required on first init) |
| `-e POSTGRES_USER=postgres` | Initial superuser name (default: `postgres`) |
| `-e POSTGRES_DB=postgres` | Initial database name (default: same as `POSTGRES_USER`) |
| `-e POSTGRES_INITDB_ARGS=...` | Extra arguments passed to `initdb` |
| `-e POSTGRES_HOST_AUTH_METHOD=trust` | Host auth method appended to `pg_hba.conf` (use `trust` only for testing, not recommended) |
| `-e PUID=70` | User ID for the postgres user (default: `70`, Alpine standard) |
| `-e PGID=70` | Group ID for the postgres group (default: `70`, Alpine standard) |
| `-e UMASK=022` | Control permission bits for newly created files (see Umask section) |
| `-e TZ=Asia/Shanghai` | Specify timezone (default: `Asia/Shanghai`) |

### Volume Mappings
| Parameter | Function |
| :----: | --- |
| `-v /var/lib/postgresql` | PostgreSQL data storage location. Mount the parent directory (not a subdirectory) to keep the database upgradeable |

## 🔄 Upgrading the Image

Because this image automatically tracks the latest stable PostgreSQL release, upgrading the image may also bump the major version. Since PG 18, data is stored under `/var/lib/postgresql/<major>/docker` so that an existing single mount at `/var/lib/postgresql` stays on the same filesystem, which is required for `pg_upgrade --link`.

To upgrade the database across major versions:

1. Stop the container and back up the data directory;
2. Start a temporary container from the **new** image on a **new** data directory to initialize the new version layout;
3. Run `pg_upgrade` (or `pg_upgrade --link`) between the old and new data directories;
4. Point the container back to the upgraded data directory and start it.

Never start the new image on the old data directory layout without migrating, otherwise the container refuses to start and reports the detected old database locations.

## 🔐 Umask for Running Applications

This image provides the ability to override default permission settings using the optional `-e UMASK=022` parameter. Remember that umask subtracts from permissions based on its value; it does not add permissions.

## 👥 User / Group Identifiers

When using volumes, permission issues can arise between the host OS and the container. To avoid this, specify the user PUID and group PGID.

The postgres user/group defaults to `70` (the standard Alpine uid/gid). Ensure any volume directories on the host are owned by the same user you specify, and permission issues will be resolved automatically. For example, on Unraid set `PUID`/`PGID` to the owner of `/mnt/user/appdata/postgresql`.

## ❓ Troubleshooting

### Cannot Connect to PostgreSQL
- ✅ Verify port `5432` is correctly mapped and no other service occupies it;
- ✅ Check that `POSTGRES_PASSWORD` was set on first initialization;
- ✅ Inspect the container log for errors: `docker logs PostgreSQL`.

### Initialization Failed / Container Restarts
- ✅ A non-empty password is required unless `POSTGRES_HOST_AUTH_METHOD=trust` is used;
- ✅ The data directory must be empty (or owned by the correct user) on first start;
- ✅ If upgrading a major version, follow the [Upgrading the Image](#-upgrading-the-image) section instead of reusing the old layout.

### Permission Issues
- ✅ Verify PUID/PGID match the owner of your host data directory;
- ✅ Check umask settings if files have incorrect permissions.
