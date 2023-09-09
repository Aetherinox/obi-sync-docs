# Reversed Obsidian Sync (Unofficial)
This is an unofficial Obsidian Sync library which allows you to host your own server for [syncing](https://obsidian.md/sync) and [publishing](https://obsidian.md/publish) obsidian notes. Behind the scenes, it re-routes Sync and Publish tasks from the official Obsidian servers to your own self-hosted server.

---

## Features

- End to end encryption
- Live sync (across devices)
- File history/recovery/snapshots
- Works natively on IOS/Android/Linux/MacOS/Windows... (via the plugin)
- Vault sharing
- [Publish (markdown only. no rendering yet)](https://github.com/acheong08/obi-sync/wiki/Obsidian-Publish)

---

## Installation
Follow the instructions below for setting up Obsi-Sync with your Obsidian.md program.

### DNS Records
Access your Domain or Cloudflare control panel and create new records for your subdomains.

![NNOxQSI](https://github.com/Aetherinox/obi-sync-docs/assets/118329232/2cf070f5-4ee5-4733-9e6e-db2770fb4599)

| Subdomain | Purpose |
| --- | --- |
| api.domain.com | Will be used for all API communication between your self-hosted solution and the Obsidian app. |
| publish.domain.com | Handles Obsidian Publish documents that can be viewed online |

### Docker
To host a [Docker](https://docker.com) container running Obsi-Sync, create a new `docker-compose.yml` with the following code inside:
```yml
version: '3.8'

services:
  obsidian_sync:
    image: ghcr.io/acheong08/obi-sync:v0.1.4@sha256:e2877d09c33201c7e69297a4614c4cafebd3d8f23a20707d4bb1c2a55dbd3111
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

---

### Creating New User
Once you have configured Docker + Nginx or your alternative solution, you must create a user that will be allowed to access your self-hosted server from within Obsidian.md.
This can be done by opening Powershell in Windows, or Terminal in Linux and executing the following:

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

A **successful** registration will return the following response:
```json
{"email":"example@example.com","name":"Example User"}
```

A **failed** registration will return the following response:
> [!WARNING]
> If you receive a **failure** from registration, make sure you're not trying to sign up the same user multiple times.
```json
{"error":"not sign up!"}
```
