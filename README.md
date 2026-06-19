# Local n8n (Docker + Postgres)

Run [n8n](https://n8n.io) — a free, self-hosted workflow automation tool — on
your own computer using Docker. It stores all your data in a PostgreSQL 18
database. Tested on a MacBook (Apple Silicon / M1), but works on any machine
with Docker.

## What you get

| Service  | What it is                                         |
| -------- | -------------------------------------------------- |
| n8n      | The automation tool you build workflows in         |
| postgres | The database where n8n saves everything            |

Your workflows and data are stored in Docker volumes, so they stay safe even if
you stop or restart the containers.

---

## Quick start (5 steps)

### 1. Install Docker

Download and install [Docker Desktop](https://www.docker.com/products/docker-desktop/),
then **open it and wait until it says it's running**. Everything below needs
Docker to be on.

### 2. Get this project

```bash
git clone https://github.com/blyncnov/n8n-local.git
cd n8n-local
```

### 3. Create your settings file

This project ships with an example settings file. Copy it to make your own real
one:

```bash
cp .env.example .env
```

Now open `.env` in any text editor and:

- Set a **password** for `POSTGRES_PASSWORD` and `N8N_BASIC_AUTH_PASSWORD`.
- Generate an **encryption key** and paste it into `N8N_ENCRYPTION_KEY`. Run this
  in your terminal and copy the result:

  ```bash
  openssl rand -hex 24
  ```

- Set `TIMEZONE` to your own timezone (so scheduled tasks run at the right time).
  Find yours with:

  ```bash
  readlink /etc/localtime | sed 's#.*/zoneinfo/##'
  ```

> **Important:** Keep your `N8N_ENCRYPTION_KEY` somewhere safe and never change
> it. It locks the credentials you save inside n8n. If you lose it, those saved
> logins can't be recovered.

### 4. Start it

```bash
docker compose up -d
```

The first start takes a few seconds while it sets up the database.

### 5. Open n8n

Go to **http://localhost:5678** in your browser.

The first time, n8n asks you to create an **owner account** (your email and a
password). Fill that in, and you're ready to build workflows.

---

## Everyday commands

```bash
docker compose ps            # see if it's running
docker compose logs -f n8n   # watch what n8n is doing
docker compose stop          # pause it (keeps your data)
docker compose start         # turn it back on
docker compose up -d         # start, or apply changes after editing .env
docker compose down          # stop and remove containers (data is kept)
docker compose down -v       # stop and DELETE everything (fresh start)
```

> If you edit `.env`, run `docker compose up -d` again so the changes take
> effect. (`start` alone won't pick them up.)

## Start over from scratch

This deletes all workflows and data, then gives you a clean install:

```bash
docker compose down -v
docker compose up -d
```

---

## Good to know

- **This is for local use.** The default settings are fine on your own computer.
  If you ever put this on the internet, use strong passwords first.
- **Postgres version is pinned to 18 on purpose.** Don't change it to `latest` —
  the database files are tied to the version number and it could refuse to start.
- **Changing the database password later won't work** on an existing setup —
  Postgres only sets it the first time. To change it, start fresh with
  `docker compose down -v`.
- **Python code nodes aren't included** in this image. JavaScript code nodes work
  fine. You'll see a harmless note about Python missing in the logs — ignore it.

## Files in this project

| File                  | What it's for                                      |
| --------------------- | -------------------------------------------------- |
| `docker-compose.yml`  | Defines the n8n + Postgres setup                   |
| `.env.example`        | Template for your settings — copy it to `.env`     |
| `.env`                | Your real settings (not shared / not in git)       |
| `.gitignore`          | Tells git to never upload your `.env`              |
| `README.md`           | This file                                          |
