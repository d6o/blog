+++
title = 'From Bitcoin Trading Bot to Docker Caddy Server Setup'
date = 2024-09-20:59:21-03:00
draft = false
tags = ["Docker", "Caddy Server", "Personal Server", "Vultr", "Bitcoin Trading", "Server Setup", "Self-Hosting", "SSL Certificates", "Reverse Proxy", "DevOps"]
+++

**Building and Evolving My Personal Server: From Naive Trading Bots to a Streamlined Docker Setup with Caddy**

Back in 2017, I embarked on an exciting (and somewhat naive) adventure when I created my first personal server (a
$2.50/month instance on Vultr). It was a modest setup—just half a shared CPU, 512 MB of RAM, and 10 GB of SSD
storage—but at the time, it felt like more than enough for my needs. The main objective was to experiment with Bitcoin
trading using a small bot that I wrote. Little did I know how challenging that would turn out to be!

### The Naive Bitcoin Trading Bot

I named the server `trader` because its purpose was straightforward: buy low, sell high, and make some profit with
Bitcoin. I whipped up a small script using Kraken’s API. The bot’s logic was simple but, in hindsight, quite naive: it
would ping Kraken’s API every minute to get the price. If the bot saw three consecutive descending prices, it would buy.
It would then hold until it observed four consecutive ascending prices, at which point it would sell.

Of course, I soon realized that predicting highs and lows with no access to future data wasn’t going to work. My grand
idea of profiting from this naive algorithm quickly turned sour, and within a few days, my €100 had been reduced to just
€50. I turned off the bot, chalking it up as a learning experience.

### Evolving the Server for New Purposes

Although the Bitcoin bot was retired, I kept the server around for small personal projects and testing. Over time,
however, my server’s modest resources became limiting, especially when I started running more complex applications. The
10 GB of SSD storage filled up quickly, and half a shared CPU wasn’t cutting it anymore. So, I upgraded to a slightly
beefier instance: 1 CPU, 1 GB of RAM, and 25 GB of SSD. This was the perfect upgrade for my evolving needs.

At the time of creating the server, I was already a big Docker enthusiast. However, it wasn’t until later that I made
the decision to migrate everything into Docker containers. This shift made it much easier to manage and deploy my
various applications. By running each service in its own container, I could isolate dependencies and ensure smoother
updates, all while keeping the system lightweight and efficient.

### Discovering Caddy: A Game Changer

As my server evolved, I started looking for a more efficient way to handle reverse proxies, SSL certificates, and
general web hosting. This is when I stumbled upon [Caddy](https://caddyserver.com/), a fantastic web server that
automatically manages SSL certificates and makes setting up reverse proxies a breeze. I decided to migrate from Nginx to
Caddy, and it turned out to be a game-changer for my server setup.

I structured everything neatly using Docker and Caddy. Each service on my server lives in its own folder, which contains
the necessary files, Dockerfile (if applicable), and a `data` folder for any persistent storage. I also have a
single `docker-compose.yml` file that orchestrates all the containers, making it easy to manage and scale my server over
time.

Here’s a glimpse of the folder structure:

```
# ls -lh
total 76K
drwx------.  3 user  group  4.0K Jul 31  2023 service1
drwx------.  3 user  group  4.0K Sep 12 00:30 service2
drwx------.  8 user  group  14K May 30  2021 service3
...
drwx------.  6 user  group  4.0K Sep 12 04:04 caddy
-rw-------.  1 user  group  2.4K Sep 12 01:30 docker-compose.yml
```

The `docker-compose.yml` file is where the magic happens. It defines all the services, Dockerfile contexts, volume
mounts, and any environment variables that each service needs. Here’s a snippet:

```yaml
version: '3'
services:

  caddy:
    container_name: caddy
    image: caddy
    restart: always
    volumes:
      - /path/caddy/Caddyfile:/etc/caddy/Caddyfile
      - /path/caddy/data/Certs/:/root/.caddy/
    ports:
      - '80:80'
      - '443:443'
    environment:
      ACME_AGREE: 'true'

  service1:
    container_name: service1
    build:
      context: /path/service1
      dockerfile: Dockerfile
      args:
        BUILDKIT_INLINE_CACHE: 1
    restart: always
    volumes:
      - /path/service1/data/vif:/data
    depends_on:
      - service1_db
    environment:
      LOGLEVEL: info

  service1_db:
    container_name: service1_db
    image: redis:7.4-alpine
    restart: always
    volumes:
      - /path/service1/data/db:/data
```

Caddy serves as the entry point for all web traffic, handling HTTPS for every service with automatic certificate
renewal. Each of my Dockerized services lives behind a reverse proxy, which is defined in Caddy's `Caddyfile`. For
example, here’s how I expose one of my services over HTTPS:

```caddy
# Exposes the port 3000 of service1 on https://service1.domain.com
service1.domain.com:443 {
	reverse_proxy service1:3000
}
```

I also use Caddy to serve static files for some websites:

```caddy
# Serves static files from service5 on https://service5.com
service5.com:443 {
    root * /path/service5
    file_server
}
```

### Why Caddy?

There are several reasons why Caddy has been such a great fit for my personal server:

1. **Automatic SSL**: Caddy handles SSL certificates automatically, so I don’t have to worry about renewing them
   manually.
2. **Easy Configuration**: The `Caddyfile` syntax is simple and concise, making it easy to configure reverse proxies and
   static file servers.
3. **Fine-grained Control**: I have complete control over which services are exposed to the internet, thanks to Caddy’s
   flexible configuration options.
4. **Docker Compatibility**: Caddy works seamlessly with Docker, making it perfect for my setup.

### Conclusion: A Setup That Works

This journey, from starting with a naive Bitcoin trading bot to managing a small but powerful personal server, has been
an invaluable learning experience. Over time, I’ve refined my setup to be efficient, scalable, and easy to manage. The
combination of Docker and Caddy has allowed me to build a system that is both powerful and simple, capable of running
multiple services while staying secure and easy to maintain.

What started as a small experiment has evolved into a stable and flexible environment for hosting my personal projects.
The setup I have now, with Caddy managing all external connections and Docker handling the internal services, is one I’m
proud of and continue to enjoy to this day.
