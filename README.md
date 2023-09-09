# Reversed Obsidian Sync (Unofficial)
This is an unofficial Obsidian Sync library which allows you to host your own server for [syncing](https://obsidian.md/sync) and [publishing](https://obsidian.md/publish) obsidian notes. Behind the scenes, it re-routes Sync and Publish tasks from the official Obsidian servers to your own self-hosted server.

This system consists of two parts:
1. Self-hosted Server | [Repo](https://github.com/acheong08/obi-sync)
2. Obsidian Plugin | [Repo](https://github.com/acheong08/rev-obsidian-sync-plugin)

You will first need to configure the self-hosted server which will be responsible for storing all of your Obsidian.md vault plugins and files. Then you will need to install the associated plugin in your Obsidian.md software which will be responsible for telling Obsidian to use your servers instead of the official Obsidian servers.

<br /><br />

> [!WARNING]
> The main branch is the development branch. For stable usage, use the latest release.

> [!NOTE]
> If you have the time and energy, feel free to help out with PRs or suggestions.

<br /><br />

# Index

- [Features](#Features)
- [Server Installation](#Server-Installation)
  - [Create DNS Records](#Create-DNS-Records)
  - [Install with Docker-compose](#Docker-Compose-Option-1)
  - [Install with Docker](#Docker-Option-2)
  - [Nginx Configuration](#Nginx-Configuration)
  - [Creating New User](#Creating-New-User)
    - [Windows (Powershell)](#Windows-Powershell)
    - [Linux (Terminal)](#Linux-Terminal)
- [Plugin Installation](#Plugin-Installation)
  - [Install -> WGET](#install-with-wget)
  - [Install -> Manually](#install-manually)
  - [Enable Plugin](#Enable-Plugin)
  - [Configure Sync](#Configure-Sync)
  - [Configure Publish](#Configure-Publish)
- [Build & Run](#Build--Run)
- [FAQ](#FAQ)
  - [Where Are Files Stored](#Where-Are-Files-Stored)
  - [What Files Are Created On Server?](#what-files-are-created-on-server)
  - [Error: User Not Signed Up](#error-user-not-signed-up)
  - [Network Error Occured. Network Unavailable](#network-error-occured-network-unavailable)
  - [Error: No Matching Manifest linux/amd64](#error-no-matching-manifest-linuxamd64)
  - [Docker-compose.yml vs .Env File](#docker-composeyml-vs-env-file)
  - [Can't Find .obsidian Folder or .Env File](#plugin-cant-find-obsidian-folder-or-env-file)
  - [Privacy & Security](#privacy--security)

<br />


---

## Features

- End to end encryption
- Live sync (across devices)
- File history/recovery/snapshots
- Works natively on IOS/Android/Linux/MacOS/Windows... (via the plugin)
- Vault sharing
- [Publish (markdown only. no rendering yet)](https://github.com/acheong08/obi-sync/wiki/dian-Publish)



<br /><br /><br />

## Server Installation
Follow the instructions below to set up obsidian sync with your [Obsidian.md](https://obsidian.md/) program.

### Create DNS Records
Access your Domain or Cloudflare control panel and create new records for two new subdomains.

![NNOxQSI](https://github.com/Aetherinox/obi-sync-docs/assets/118329232/2cf070f5-4ee5-4733-9e6e-db2770fb4599)

| Subdomain | Purpose |
| --- | --- |
| api.domain.com | Will be used for all API communication between your self-hosted solution and the Obsidian app. |
| publish.domain.com | Handles Obsidian Publish documents that can be viewed online |



<br /><br /><br />

### Docker-Compose (Option 1)
If using this option, you must decide which setup you would like to use. 
<br /><br />
You can either create a single `docker-compose.yml` file and keep all settings inside that one file, OR you can use the two file method which requires you to create a `docker-compose.yml` file, and an `.env` environment variable file.

Using the method with the `.env` environment variable file is slightly more secure in regards to your `SIGNUP_KEY` being exposed to logs. 

- If you are not concerned about security, use [DOCKER-COMPOSE.YML ONLY](#docker-composeyml-only) option.
- If you want the extra security, use [DOCKER-COMPOSE.YML + .ENV](#docker-composeyml--env) option.

<br /><br />

---

<br /><br />

##### DOCKER-COMPOSE.YML ONLY
To use `docker-compose.yml` only for obi-sync, create a new `docker-compose.yml` with the following code inside:
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
      - DATA_DIR=/obi-sync/
      - MAX_STORAGE_GB=10
      - MAX_SITES_PER_USER=5
    volumes:
      - ./obi-sync:/obi-sync

volumes:
  obi-sync:
```

<br /><br />

---

<br /><br />

##### DOCKER-COMPOSE.YML + .ENV
To use `docker-compose.yml` and `.env` file option; create the following two files in the same folder:
- `docker-compose.yml`
- `.env`

Inside each file, paste the following:

#### docker-compose.yml
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
      DOMAIN_NAME: ${DOMAIN}
      ADDR_HTTP: ${ADDR_HTTP}
      SIGNUP_KEY: ${USER_SIGNUP_KEY}
      DATA_DIR: ${DIR_DATA}
      MAX_STORAGE_GB: ${USER_MAX_STORAGE}
      MAX_SITES_PER_USER: ${USER_MAX_SITES}
    volumes:
      - ./obi-sync:/obi-sync

volumes:
  obi-sync:
```

#### .env
```env
DOMAIN='api.domain.com'
ADDR_HTTP='0.0.0.0:3000'
DIR_DATA='/obi-sync/'
USER_SIGNUP_KEY='YOUR_PRIVATE_STRING_HERE'
USER_MAX_STORAGE=10
USER_MAX_SITES=5
```

<br /><br />

---

<br /><br />

A description of each variable is provided below:

<br />

| Variable | Description | Required | Default |
| --- | --- | --- | --- | 
| `DOMAIN_NAME` | This is the URL to your API subdomain | Yes | `localhost:3000` |
| `ADDR_HTTP` | The address to run Obi-Sync on | No | `127.0.0.1:3000` |
| `SIGNUP_KEY` | Required later when creating users who will be able to access your self-hosted server | No | None |
| `DATA_DIR` | Where encrypted `vault.db` and other files will be stored | No | `./` |
| `MAX_STORAGE_GB` | The maximum storage per user in GB. | No | `10` |
| `MAX_SITES_PER_USER` | The maximum number of sites per user. | No | `5` |

<br /><br />

After creating the file(s) above depending on your selection, you need to `cd` to the folder where you created the file(s) and execute:
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
To install Obi-Sync using the docker `pull` command, view the [Package Release](https://github.com/acheong08/obi-sync/pkgs/container/obi-sync) page and copy the command for the OS you are running.

The default pull command is

```shell
docker pull ghcr.io/acheong08/obi-sync:latest
```

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
#   Obsidian Sync
#
#   defined within the obi-sync plugin for Obsidian so that Obsidian and
#   the self-hosted server can sync files.
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
#   used for viewing published Obsidian .md files on a webserver.
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

---

### Creating New User
You must create a user that will be allowed to access your self-hosted server from within Obsidian.md.
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

## Plugin Installation
In order for your new self-hosted Publish and Sync server to function properly, you must install a plugin to your copy of Obsidian.md

### Install with WGET
Create a new folder for the plugin and enter the folder:

```shell
cd /path/to/vault/.obsidian
mkdir -p plugins/custom-sync-plugin && cd plugins/custom-sync-plugin
```

Use `wget` to download the plugin files:

```shell
wget https://github.com/acheong08/rev-obsidian-sync-plugin/raw/master/main.js https://github.com/acheong08/rev-obsidian-sync-plugin/raw/master/manifest.json
```

<br /><br />

### Install Manually
To manually get the latest copy of the unofficial Obi-Sync Plugin, [download here](https://github.com/acheong08/rev-obsidian-sync-plugin/releases).
- Navigate to the folder where your vault is on your local machine, and enter the `.obsidian\plugins\` folder.
- Create a new folder: `custom-sync-plugin`
- Inside the `custom-sync-plugin` folder, install the plugin files:
  - 📄 `main.js`
  - 📄 `manifest.json`

<br />

> [!NOTE]
> Alternatively, you can use https://github.com/TfTHacker/obsidian42-brat which can be found in the official community plugins list.

<br /><br />

### Enable Plugin
Once the plugin is installed, activate it by launching Obsidian.md. 
- Open `Obsidian Settings` ![uJ5MSWk](https://github.com/Aetherinox/obi-sync-docs/assets/118329232/f5695ae4-0730-496c-b182-3bf4836ba571)
- On left menu, click `Community Plugins`
- Scroll down to `Custom Native Sync` and enable ![f8iiGTI](https://github.com/Aetherinox/obi-sync-docs/assets/118329232/e38ac70d-60ea-4cf7-939a-ab56d5302f11)
- To the left of the enable button for `Custom Native Sync`, press the options ![j4JYNxN](https://github.com/Aetherinox/obi-sync-docs/assets/118329232/1de68f02-17be-4759-b21b-d344579477a4) button.
- Configure the `Obsidian Sync URL` in the Plugin settings after setting up your sync server
  - By default, it is `https://api.domain.com`
- Go to the `Core Plugins` section of Obsidian
- Locate the core plugin `Sync` and enable ![f8iiGTI](https://github.com/Aetherinox/obi-sync-docs/assets/118329232/e38ac70d-60ea-4cf7-939a-ab56d5302f11)

![HzDfnB4](https://github.com/Aetherinox/obi-sync-docs/assets/118329232/44363513-699b-49c9-84fe-94faf1785981)

![ioHy4jQ](https://github.com/Aetherinox/obi-sync-docs/assets/118329232/61d154df-430d-4785-983c-633418607d12)


<br /><br />

### Configure Sync
Before doing these steps, ensure that your obi-sync self-hosted server is running.

Open your Obsidian.md Settings ![uJ5MSWk](https://github.com/Aetherinox/obi-sync-docs/assets/118329232/f5695ae4-0730-496c-b182-3bf4836ba571) and select `About`.

You need to sign in and connect Obsidian.md to your self-hosted server by clicking `Log In`:

![fs9PioG](https://github.com/Aetherinox/obi-sync-docs/assets/118329232/941f8b5a-485f-4f6b-bdd0-ad0e6ab092a5)

The login dialog will appear:

![aVD8YRs](https://github.com/Aetherinox/obi-sync-docs/assets/118329232/5d065b57-708c-4bbe-8373-8d11354b09f5)

Fill in your email address and password that you used to register your account with the `cURL` command in the section [Creating New User](#Creating-New-User) above.

```powershell
curl --request POST `
  --url https://api.domain.com/user/signup `
  --header 'Content-Type: application/json' `
  --data '{
	"email": "example@example.com",      <----- The email you will use
	"password": "example_password",	     <----- The password you will use
	"name": "Example User",
	"signup_key": "<SIGNUP_KEY>"
}'
```

<br />

> [!NOTE]
> After signing in successfully, the `Log In` button will change to `Manage settings`. Keep in mind though that clicking the `Manage Settings` button will take you to Obsidian's official website. It has nothing to do with your own self-hosted server.

<br />

Next, on the left side under `Core Plugins`, select `Sync`. 

On the right side, click the `Choose` button to the right of `Remote Vault`

![eV6KKFy](https://github.com/Aetherinox/obi-sync-docs/assets/118329232/4a58bc85-2fb1-4bf6-882e-596a681f9385)

A new dialog will then appear. Select `Create New Vault`

![97sJzUP](https://github.com/Aetherinox/obi-sync-docs/assets/118329232/dca34861-a62d-4714-819e-acc71b579671)

Then fill in the information for your new vault. The `Encryption Password` can be anything you want. Do **not** lose this password. You cannot unlock / decrypt your vault if you can't remember it.

![tj19O8m](https://github.com/Aetherinox/obi-sync-docs/assets/118329232/e204489c-5a03-46bd-b5ba-94402280477e)

Your new vault will be created, along with a few options you can select, and a `Connect` button.

![RDFF70c](https://github.com/Aetherinox/obi-sync-docs/assets/118329232/c264782c-6f92-4f7c-97b5-bfb0b0c857cd)

Click the `Connect` button to start Syncing between your local vault and your self-hosted sync server. If you get a warning message labeled `Confirm Merge Vault`, click `Continue`.

If you provided an `Encryption Password` a few steps back, you will now be asked to enter that password, and then click `Unlock Vault`.

You will then get one more confirmation that your vault is now synced.

![6znaxbj](https://github.com/Aetherinox/obi-sync-docs/assets/118329232/2dbe1e86-3b91-4cb4-a5da-fc354d13e40d)

From this point on, you can adjust any of the Sync settings you wish to modify, which includes syncing things like images, videos, plugins, settings, and more. Whatever options you enable, will be included in the Sync job.

Ensure `Sync Status` is set to `running`. You can enable / disable the sync's state by clicking the button to the right.

![UkqH5w1](https://github.com/Aetherinox/obi-sync-docs/assets/118329232/0c786122-3ac6-45e7-843a-2aa4cc8312da)

You can confirm that your files are successfully syncing by clicking the `View` button to the right of `Sync Activity`.

![iyeEBrs](https://github.com/Aetherinox/obi-sync-docs/assets/118329232/bc1c591b-dc75-4ae2-a3f3-ac121ffe9643)

Once pressing the `View` button, a new dialog will appear and show you the status of your Sync job.

```console
2023-09-09 01:38 - Uploading file Test Folder/Subfolder/My File 1.md
2023-09-09 01:38 - Upload complete Test Folder/Subfolder/My File 1.md
2023-09-09 01:38 - Uploading file Test Folder/Subfolder/My File 2.md
2023-09-09 01:38 - Upload complete Test Folder/Subfolder/My File 2.md
2023-09-09 01:40 - Fully synced
```

<br /><br />

### Configure Publish
These instructions help guide you through setting up `Obsidian Publish`. This feature will allow you to view your Vault .md notes on your webserver.

Open your Obsidian.md settings, and under `Options` on the left, select `Core Plugins`. In the middle of the screen, enable ![f8iiGTI](https://github.com/Aetherinox/obi-sync-docs/assets/118329232/e38ac70d-60ea-4cf7-939a-ab56d5302f11) `Publish`.

![NvmH8hV](https://github.com/Aetherinox/obi-sync-docs/assets/118329232/794d9820-1f45-4f9c-afbc-243fd3537260)

Back on the main Obsidian interface, select the `Publish Changes` ![Publish Changes Button](https://github.com/Aetherinox/obi-sync-docs/assets/118329232/e9cd7054-0a41-472f-accb-d9fa0426436d) button in your Obsidian side / toolbar.

![iYTDAi4](https://github.com/Aetherinox/obi-sync-docs/assets/118329232/e7f24105-432b-4436-a8a1-71862c873640)

In the new dialog, enter a `Site ID` in the textfield provided. This will become the `slug` name for your new vault.

![nofm4EB](https://github.com/Aetherinox/obi-sync-docs/assets/118329232/4254380c-331e-4a80-8299-3204fb098cc2)

Once you have a name, click `Create`.

You will then see a list of all your vault's associated files. Select the checkbox to the left of each folder you wish to upload to your self-hosted Publish server.

![FiUou2Z](https://github.com/Aetherinox/obi-sync-docs/assets/118329232/936fbb7f-4867-4b44-9e80-02b7221f3c62)

Once you've selected the desired folders, click `Publish` in the bottom right.

Yet another dialog will appear which confirms your uploaded vault documents.

![sTxX4Ax](https://github.com/Aetherinox/obi-sync-docs/assets/118329232/1a2501d8-338d-455e-8829-0ef2b9439494)

<br />

> [!NOTE]
> Because we are hosting our own server, the link provided at the bottom of the dialog **will not work** since it goes to Obsidian's official publish server.

<br />

Open your web browser and go to the URL for your self-hosted publish server. In our example, we would view our `Testing Folder/Tags.md` page by entering the following URL:

```
https://publish.domain.com/myvault/Testing Folder/Tags.md
```

- `myvault`: Name given to example vault
- `Testing Folder`: Folder name the note resides in
- `Tags.md`: Note filename

To see an overview of files you have uploaded to your publish server, go to your publish subdomain, and add your vault name at the end. Nothing else needs added.

```
https://publish.domain.com/myvault
```

In our example vault, visiting the URL above displays `JSON` and includes a list of all files uploaded from the test vault:

![uBIwPoS](https://github.com/Aetherinox/obi-sync-docs/assets/118329232/c73e23cd-3a3e-46ca-8bf1-62bb948930d2)

From this point, you can upload whatever files you wish to have published and play around with the settings.


<br /><br /><br />

---

## Build & Run

<br />

> [!NOTE]
> This requires you to install the Go interrupter on your machine.
> You may download it [here](https://go.dev/doc/install).

<br />

To build and run Obi-sync directly from the git repo, execute the following:
- `git clone https://github.com/acheong08/obi-sync`
- `cd obsidian-sync`
- `go run cmd/obsidian-sync/main.go`

<br /><br /><br />

---

## FAQ

### Where Are Files Stored?
This depends on how you've configured obsidian sync server to run. 
If you installed this project using [Install with Docker-compose](#Docker-Compose-Option-1), and specified the environment variable `DATA_DIR`, then the files should exist in the same folder you specified the variable to use. By default, vault files are placed in the folder `obi-sync`.

If you installed this project using [Install with Docker](#Docker-Option-2) or did not specify a variable for `DATA_DIR`, then the files are typically stored in `/var/lib/docker/*`

Some users may be running [Portainer](https://www.portainer.io/), which allows you to view your docker containers and volumes within a graphical user interface. Login to the portainer web admin panel and under `Volumes`, find out which volume is assigned to `obsidian_sync`, and copy the value `Mount path`. Open your File Explorer and go to that location to view your vault files.

<br /><br />

---

<br /><br />

### What Files Are Created On Server?
This project will store your vault data in the following files:
```
📁 obi-sync
   📄 publish.db
   📄 secret.gob
   📄 vault.db
   📄 vaultfiles.db
```

<br /><br />

---

<br /><br />

### Error: User Not Signed Up
This error typically occurs when you're creating a user with `cURL` and shows the following:
```json
{"error":"not sign up!"}
```
<br />

If you receive this error when creating your first API user, ensure the user did not previously exist. 

<br /><br />

---

<br /><br />

### Network Error Occured. Network Unavailable
If you are attempting to use the `Obsidian Publish` feature and receive the error:

![nJ66in6](https://github.com/Aetherinox/obi-sync-docs/assets/118329232/6094db18-a523-40d2-891c-f59d2e868556)

Ensure you have provided the correct environment variable `DOMAIN_NAME=api.domain.com`.
If you provided the wrong domain name and need to change it, you must do the following:
- Edit `docker-compose.yml`
- Change `DOMAIN_NAME` variable to correct domain.
- Locate the project file `publish.db` and DELETE it completely.
- Restart the docker container and Obsidian.md program

Once back in Obsidian.md program, click `Publish Changes` ![Publish Changes Button](https://github.com/Aetherinox/obi-sync-docs/assets/118329232/e9cd7054-0a41-472f-accb-d9fa0426436d) button again.

You can also attempt to locate the root cause of the issue by pressing `CTRL + SHIFT + I` inside of Obsidian.md. Then on the right side, click `Network` and then re-open the `Publish Changes` interface again.

![266769187-37688550-ae87-4b75-87e1-c3024fe0fef6](https://github.com/Aetherinox/obi-sync-docs/assets/118329232/8f462c26-503b-4cc5-a2f5-3272ee7c2ff6)

Anything listed in `red` is an error and needs to be corrected. Ensure that it is trying to fetch the correct domain from the `Request URL`. Anything listed in `white` and with a `Status Code` of `200` is properly connecting.

<br /><br />

---

<br /><br />

### Error: No Matching Manifest linux/amd64
Make sure you are pulling the correct docker image.

You can also visit the [Package Release](https://github.com/acheong08/obi-sync/pkgs/container/obi-sync) page, select `OS / Arch` tab, and copy the pull command listed below `linux/amd64`

<br /><br />

---

<br /><br />

### Docker-compose.yml vs .Env File
In the section above titled [Install with Docker-compose](#Docker-Compose-Option-1), there are two ways to create your `docker-compose.yml` file.
1. Single `docker-compose.yml` - See [DOCKER-COMPOSE.YML ONLY](#docker-composeyml-only)
2. `docker-compose.yml` and `.env` file - See [DOCKER-COMPOSE.YML + .ENV](#docker-composeyml--env)

#### 🟢 Single `docker-compose.yml` File
This method involves creating a single `docker-compose.yml` file which will hold all of your settings for this project. 

#### 🟢 `docker-compose.yml` and `.env` File
This method requires you to create two files which both exist in the same folder. The benefit of this method is that your `SIGNUP_KEY` environment variable will not be leaked in your docker logs and is slightly better if you are worried about security.

You can decide to use either one of the two options.

If using `Portainer` web manager for Docker, you can access the environment variables clicking `Containers` on left side menu, then click `obsidian-sync` in your list of containers. Then scroll down to the `ENV` section.

![PZkqLFM](https://github.com/Aetherinox/obi-sync-docs/assets/118329232/50ed9c37-d17c-4b1a-a67a-8f2d404484e1)

<br /><br />

---

<br /><br />

### Plugin: Can't Find `.obsidian` Folder or `.Env` File
The `.obsidian` plugin folder may be hidden. You can configure your operating system's File Explorer show hidden files or use a terminal. For IOS, you might need to plug it into a computer, go to `Documents on iPhone`, then go to `Obsidian/<your vault name>/.obsidian`.

If you are using Linux and cannot locate the `.obsidian` folder or `.env` docker file, Linux by default hides all folders and files that start with a period. To view these files in your File Explorer, press the key combination `CTRL + H`. You may be asked for your account or sudo password.

<br /><br />

---

<br /><br />

### Privacy & Security
The plugin and self-hosted server are dead simple. No data is collected. You can even run your sync server on the local network without access to the internet. All vault data is stored behind an encrypted password and inside a database file.
