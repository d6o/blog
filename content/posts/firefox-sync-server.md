+++
title = 'Self-Hosting a Firefox Sync Server'
date = 2024-09-06T08:52:32-03:00
draft = false
tags = ["Firefox Sync","Self-hosting","LibreWolf","Firefox","Syncstorage-rs","MariaDB","Reverse Proxy","Caddy","Server Management","Database","Docker","Open Source"]
+++

After switching from Firefox to LibreWolf, I became interested in the idea of self-hosting my own Firefox Sync server.
Although I had seen this was possible before, I had never really looked into it—until now. I embarked on a journey to
set this up, and while it wasn't completely smooth sailing, I eventually got it working. Here's how it went.

## Finding the Right Sync Server

### Initial Search: Mozilla's Sync Server Repo

I started by searching for "firefox sync server github" and quickly
found [Mozilla's syncserver repo](https://github.com/mozilla-services/syncserver). This is an all-in-one package
designed for self-hosting a Firefox Sync server. It bundles both the tokenserver for authentication and syncstorage for
storage, which sounded like exactly what I needed.

However, there were two red flags:

1. The repository had "failed" tags in the build history.
2. A warning was prominently displayed stating that the repository was no longer being maintained and pointing to a new
   project in Rust.

### Switching to Rust: syncstorage-rs

With that in mind, I followed the link to [syncstorage-rs](https://github.com/mozilla-services/syncstorage-rs), which is
a modern, Rust-based version of the original project. It seemed like the more viable option, so I decided to move
forward with this one. But first, I wanted to check if there was a ready-to-go Docker image to make deployment easier.
Unfortunately, there wasn't one, but the documentation did mention running it with Docker.

This is where things started to get complicated.

## Diving Into Docker: Confusion and Complexity

### Documentation Woes

The Docker documentation had some strange parts. For example, it mentioned:

- Ensuring that `grpcio` and `protobuf` versions matched the versions used by `google-cloud-rust-raw`. This sounded
  odd—shouldn't Docker handle version dependencies automatically?
- Another confusing part was the instruction to manually copy the contents of `mozilla-rust-sdk` into the top-level root
  directory. Again, why wasn't this step automated in the Dockerfile?

At this point, I was feeling a bit uneasy but decided to push forward. I reviewed the repo, the Dockerfile, the
Makefile, and the circleci workflows. Despite all that, I was still unsure how to proceed.

### A Simpler Solution: syncstorage-rs-docker

I then stumbled upon [dan-r's syncstorage-rs-docker repo](https://github.com/dan-r/syncstorage-rs-docker), which had a
much simpler Docker setup. The description explained that the author had also encountered issues with the
original documentation and decided to create a Docker container for their own infrastructure.

At this point, I felt reassured that I wasn't alone in my confusion, and decided to give this setup a try.

## Setting Up the Server: Docker Compose and MariaDB

### Docker Compose Setup

I copied the following services into my `docker-compose.yaml`:

```yaml
  firefox_mariadb:
    container_name: firefox_mariadb
    image: linuxserver/mariadb:10.6.13
    volumes:
      - /data/ffsync/dbdata:/config
    restart: unless-stopped
    environment:
      MYSQL_DATABASE: syncstorage
      MYSQL_USER: sync
      MYSQL_PASSWORD: syncpass
      MYSQL_ROOT_PASSWORD: rootpass

  firefox_syncserver:
    container_name: firefox_syncserver
    build:
      context: /root/ffsync
      dockerfile: Dockerfile
      args:
        BUILDKIT_INLINE_CACHE: 1
    restart: unless-stopped
    ports:
      - "8000:8000"
    depends_on:
      - firefox_mariadb
    environment:
      LOGLEVEL: info
      SYNC_URL: https://mydomain/sync
      SYNC_CAPACITY: 5
      SYNC_MASTER_SECRET: mastersecret
      METRICS_HASH_SECRET: metricssecret
      SYNC_SYNCSTORAGE_DATABASE_URL: mysql://sync:usersync@firefox_mariadb:3306/syncstorage_rs
      SYNC_TOKENSERVER_DATABASE_URL: mysql://sync:usersync@firefox_mariadb:3306/tokenserver_rs
```

A few tips:

- Be cautious with the database passwords. Avoid using special characters like `"/|%"` as they can cause issues during
  setup.
- I optimized the Dockerfile to make better use of caching, which reduced compilation time while testing.

### Initializing the Database

I cloned the repository and copied the Dockerfile and `initdb.sh` script to my server. After making some tweaks, I ran
the following steps to get the database up and running:

1. Bring up the MariaDB container:
   ```bash
   docker-compose up -d firefox_mariadb
   ```
2. Make the initialization script executable and run it:
   ```bash
   chmod +x initdb.sh
   ./initdb.sh
   ```

### Bringing the Stack Online

Finally, I brought up the entire stack with:

```bash
docker-compose up -d
```

## Configuring Reverse Proxy with Caddy

Next, I needed to update my Caddy reverse proxy to point to the new Sync server. I added the following configuration:

```bash
mydomain:443 {
     reverse_proxy firefox_syncserver:8000 {
    }
}
```

After updating Caddy with the DNS entry, I restarted the proxy and the sync server was up and running.

## Challenges Faced

While I eventually got everything working, there were a few notable challenges along the way:

1. **Database persistence**: I had issues with persistent data when restarting the MariaDB container. Make sure to clear
   out old data if needed.
2. **Server storage**: My server ran out of space during the build process due to the size of the Docker images and
   intermediate files.
3. **Following the right steps**: It took me a while to figure out the right steps, and much of the time was spent
   experimenting with the Docker setup.

## Final Thoughts

Setting up a self-hosted Firefox Sync server is not the easiest task, especially if you're not very familiar with Docker
or database management. The official documentation is confusing, but thanks to community efforts like
the `syncstorage-rs-docker` repo, it’s doable.

In the end, it took me about two hours to get everything running, but it was worth it. If you’re looking to control your
own Firefox Sync server, this guide should help you avoid some of the pitfalls I encountered.

Happy syncing!