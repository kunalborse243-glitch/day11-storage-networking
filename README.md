# Day 11 — Docker Storage & Networking

## Overview

This project demonstrates Docker named volumes, persistent PostgreSQL storage, custom bridge networking, container DNS resolution, host-port conflicts, and compressed volume backups.

## Technologies Used

- Docker
- PostgreSQL 16 Alpine
- Alpine Linux
- Nginx Alpine
- Docker named volumes
- Docker bridge networks
- Bash and PostgreSQL CLI

## 1. PostgreSQL with Persistent Storage

Created a PostgreSQL container named `db` with:
- Database: `shop`
- User: `postgres`
- Named volume: `pgdata`
- Custom network: `app-net`

The PostgreSQL data directory was mounted at `/var/lib/postgresql/data`.

## 2. Data Persistence Test

Created a `notes` table and inserted records containing `I survive!`.

Removed and recreated the PostgreSQL container while reusing the same named volume. Both records remained available after recreation.

**Result:** Data persisted independently of the container lifecycle.

Evidence: `evidence/03-data-before-recreation.png`, `evidence/04-data-persistence.png`

## 3. Custom Bridge Networking

Connected a temporary PostgreSQL client to the database using hostname `db` on `app-net`.

The client successfully queried the `notes` table and retrieved both records.

**Result:** Container-to-container connectivity and custom-network DNS resolution worked.

Evidence: `evidence/05-container-networking.png`

## 4. Default Bridge vs Custom Bridge DNS

- On Docker's default bridge network, the container hostname did not resolve automatically.
- On the user-defined bridge network `app-net`, the hostname `dns-test-custom` resolved successfully and ping received a reply.
- Legacy `--link` also resolved the name on the default bridge during the comparison.

**Result:** User-defined bridge networks provide automatic container-name DNS resolution.

Evidence: `evidence/06-dns-test.png`

## 5. Host-Port Conflict

Started Nginx container `w1` using host port `18080`. A request to `http://localhost:18080` returned `HTTP/1.1 200 OK`.

Attempted to start `w2` using the same host port. Docker rejected the binding because port `18080` was already allocated.

Started `w2` using host port `18081`, and `http://localhost:18081` returned `HTTP/1.1 200 OK`.

**Result:** Containers can use the same internal port while exposing different host ports.

Evidence: `evidence/07-port-conflict-and-fix.png`

## 6. PostgreSQL Volume Backup

Stopped the PostgreSQL container and archived the named volume using a temporary Alpine container and `tar`.

Backup file: `pgdata-backup.tgz`  
Observed file size: approximately 6.5 MB

Verified the archive by listing its contents with `tar -tzf`. Restarted PostgreSQL and confirmed the database was accepting connections and both records remained available.

**Result:** The compressed volume archive was created and its contents were listed successfully.

Evidence: `evidence/08-volume-backup.png`

## Project Structure

```text
day11-storage-networking/
├── README.md
├── pgdata-backup.tgz
└── evidence/
    ├── 03-data-before-recreation.png
    ├── 04-data-persistence.png
    ├── 05-container-networking.png
    ├── 06-dns-test.png
    ├── 07-port-conflict-and-fix.png
    └── 08-volume-backup.png
```

## Key Learnings

1. Named volumes preserve database data across container recreation.
2. User-defined bridge networks support automatic container-name DNS resolution.
3. Host ports must be available before containers can bind them.
4. A named volume can be archived using a temporary container.
5. Backup creation should be followed by verification.

## Security Note

The password `changeme` is for this local learning exercise only. Do not use it for production workloads. The backup contains database files and should be handled as sensitive data.

