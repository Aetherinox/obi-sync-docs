# Reversed Obsidian Sync (Unofficial)
This is an unofficial Obsidian Sync library which allows you to host your own server for [syncing](https://obsidian.md/sync) and [publishing](https://obsidian.md/publish) obsidian notes. Behind the scenes, it re-routes Sync and Publish tasks from the official Obsidian servers to your own self-hosted server.

# Index

- [Features](#Features)
- [Installation](#Installation)
  - [Create DNS Records](#Create-DNS-Records)
  - [Install with Docker-compose](#Docker-Compose-Option-1)
  - [Install with Docker](#Docker-Option-2)
  - [Nginx Configuration](#Nginx-Configuration)
- [Creating New User](#Creating-New-User)
  - [Windows (Powershell)](#Windows-Powershell)
  - [Linux (Terminal)](#Linux-Terminal)
- [Build & Run](#Build-Run)

<br />

---

## Features

- End to end encryption
- Live sync (across devices)
- File history/recovery/snapshots
- Works natively on IOS/Android/Linux/MacOS/Windows... (via the plugin)
- Vault sharing
- [Publish (markdown only. no rendering yet)](https://github.com/acheong08/obi-sync/wiki/Obsidian-Publish)



<br /><br /><br />

## Installation
Follow the instructions below to set up Obsi-Sync with your [Obsidian.md](https://obsidian.md/) program.

### Create DNS Records
Access your Domain or Cloudflare control panel and create new records for two new subdomains.

![NNOxQSI](https://github.com/Aetherinox/obi-sync-docs/assets/118329232/2cf070f5-4ee5-4733-9e6e-db2770fb4599)

| Subdomain | Purpose |
| --- | --- |
| api.domain.com | Will be used for all API communication between your self-hosted solution and the Obsidian app. |
| publish.domain.com | Handles Obsidian Publish documents that can be viewed online |



<br /><br /><br />

### Docker-Compose (Option 1)
To use `docker-compose` for Obsi-Sync, create a new `docker-compose.yml` with the following code inside:
```yml
version: '3.8'

services:
  obsidian_sync:
    image: ghcr.io/acheong08/obi-sync:latest
    container_name: obsidian_sync
    restart: always
    ports:
      - 3000:3000
    environment:
      - DOMAIN_NAME=api.domain.com
      - ADDR_HTTP=0.0.0.0:3000
      - SIGNUP_KEY=YOUR_PRIVATE_STRING_HERE
      - DATA_DIR=/obsi-sync/
      - MAX_STORAGE_GB=10
      - MAX_STORAGE_GB=5
    volumes:
      - ./obsi-sync:/obsi-sync

volumes:
  obsi-sync:
```

| Variable | Description | Required | Default |
| --- | --- | --- | --- | 
| `DOMAIN_NAME` | This is the URL to your API subdomain | Yes | `localhost:3000` |
| `ADDR_HTTP` | The address to run Obsi-Sync on | No | `127.0.0.1:3000` |
| `SIGNUP_KEY` | Required later when creating users who will be able to access your self-hosted server | No | None |
| `DATA_DIR` | Where encrypted `vault.db` and other files will be stored | No | `./` |
| `MAX_STORAGE_GB` | The maximum storage per user in GB. | No | `10` |
| `MAX_SITES_PER_USER` | The maximum number of sites per user. | No | `5` |

<br />

After creating the above `docker-compose.yml` file, `cd` to the folder where this new file is and execute the following:
```shell
docker compose up -d
```

To shut down the container:
```shell
docker compose down
```

To confirm it is running:
```shell
docker ps
```
![jddB3lb](https://github.com/Aetherinox/obi-sync-docs/assets/118329232/2954845f-20ba-43da-8873-69d520b697d2)



<br /><br /><br />

---

### Docker (Option 2)
To install Obsi-Sync using the docker `pull` command, view the [Package Release](https://github.com/acheong08/obi-sync/pkgs/container/obi-sync) page and copy the command for the OS you are running.

<br /><br /><br />

---

### Nginx Configuration
You must configure nginx to handle your new subdomains by adding a few entries to your nginx config file. If you wish to add this to your current running nginx webserver, you can paste it somewhere in `\etc\nginx\sites-enabled\default`.

<br />
<br />

> [!NOTE]
> Do not copy/paste the server blocks below without looking them over. If you have changed port `3000` in your docker container, you must change it below to that same port. Also change `domain.com` to your correct domain.

<br />
<br />

```nginx
map $http_upgrade $connection_upgrade {
        default upgrade;
        '' close;
}

#
#   Obsidian API
#

    server
    {
        listen 443 ssl http2;
        listen [::]:443 ssl http2;
        listen 80;
        listen [::]:80;

        server_name         	www.api.domain.com api.domain.com;

        location /
        {
            proxy_http_version  1.1;
            proxy_set_header    Upgrade $http_upgrade;
            proxy_set_header    Connection $connection_upgrade;
            proxy_set_header    Host $host;
            proxy_pass          http://127.0.0.1:3000/;
        }
        
    }

#
#   Obsidian Publish
#

    server
    {
        listen 443 ssl http2;
        listen [::]:443 ssl http2;
        listen 80;
        listen [::]:80;

        server_name         	www.publish.domain.com publish.domain.com;
        
        location /
        {
            proxy_pass     	http://127.0.0.1:3000/published/;
        }
    }
```
<br />

After modifying your nginx service, **restart** it to load up the new configs.
```shell
sudo systemctl restart nginx
```
Or restart nginx via docker if you run it through a container.

<br /><br /><br />

## Creating New User
Once you have configured Docker + Nginx or your alternative solution, you must create a user that will be allowed to access your self-hosted server from within Obsidian.md.
This can be done by opening Powershell in Windows, or Terminal in Linux and executing the following:

<br />

> [!NOTE]
> `signup_key`: If you removed `signup_key` from your docker container's variables and don't want to require a signup key for registration, remove that line from the command below.
> 
> `example_password`: Pick any password you wish to use. This is the password you will use in the Obsidian.md program once you configure the plugin to connect to your self-hosted server.

<br />

#### Windows `Powershell`
```powershell
curl --request POST `
  --url https://api.domain.com/user/signup `
  --header 'Content-Type: application/json' `
  --data '{
	"email": "example@example.com",
	"password": "example_password",
	"name": "Example User",
	"signup_key": "<SIGNUP_KEY>"
}'
```

#### Linux `Terminal`
```bash
curl --request POST \
  --url https://api.domain.com/user/signup \
  --header 'Content-Type: application/json' \
  --data '{
	"email": "example@example.com",
	"password": "example_password",
	"name": "Example User",
	"signup_key": "<SIGNUP_KEY>"
}'
```

<br />

A **successful** registration will return the following response:
```json
{"email":"example@example.com","name":"Example User"}
```

<br />

A **failed** registration will return the following response:
> [!WARNING]
> If you receive a **failure** from registration, make sure you're not trying to sign up the same user multiple times.
```json
{"error":"not sign up!"}
```

<br /><br /><br />

---

## Build & Run

<br />

> [!NOTE]
> This requires you to install the Go interrupter on your machine.
> You may download it [here](https://go.dev/doc/install).
> 
<br />

To build and run Obsi-sync directly from the git repo, execute the following:
- `git clone https://github.com/acheong08/obsidian-sync`
- `cd obsidian-sync`
- `go run cmd/obsidian-sync/main.go`
