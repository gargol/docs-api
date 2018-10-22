---
title: "Knowledge Base"
date: "2018-10-01"
meta_title: "CLI Knowledge Base"
meta_description: "Advanced information about the Ghost-CLI and how it works!"
keywords:
    - cli
    - ghost
    - advanced
    - knowledge base
sidebar: "ghost-cli"
---

Learn more about using the Ghost-CLI for a smooth install, run and update process when working with Ghost, and how to take advantage of it's flexibility.


## Overview

Below is a list of the things the `ghost-cli` handles:

- Checks for common environment problems
- Creates a **useful folder structure**
- Provides for production or development installs
- Allows for **upgrading and rolling back**
- Handles **user management and permissions**
- Configures Ghost
- Configures **NGINX**
- Sets up **MySQL**
- Configures **systemd**
- Allows access to Ghost log files
- Provides a management console to start/stop/restart Ghost



## Folder Structure

When you install Ghost using Ghost-CLI, the local directory will be setup with a set of folders designed to keep the various parts of your install separate. After installing Ghost, you will have a folder structure like this which should not be changed:

```bash
config.[env].json #env is production or development
/content #owned by ghost user
/current
/system #production only
/versions
.ghost-cli
```

#### config.[env].json

These files contain the config for your site, read more here.
You can add information about your site and it's theme in these files.

#### /content

Themes, images, redirects, custom adapters will live in this folder. This folder is owned by the ghost user and it is recommended to keep a backup of this folder.

#### /current

A symlink to the currently active version of Ghost. To upgrade your version of Ghost, use the [update](/api/ghost-cli/update/) commands and guides.

#### /system

This folder only appears on production installs. It contains additional tools and configuration files needed for nginx, systemd and for generating letsencrypt certificates.

#### /system/files

Inside the `/system` folder is a files folder, this is the location of any configuration that ghost-cli creates for 3rd party tools such as nginx and systemd. These files are always symlinked into their correct locations. They can be customised if needed.

#### /versions

Contains the versions of Ghost which are currently available on your machine. There's no need to worry about this unless space is a particular issue for you.

#### .ghost-cli

This file contains some information that Ghost-CLI needs about your install in order to perform certain tasks. There's no need to worry about this file and it's recommended that you don't edit it!


## NGINX

NGINX is the web server that backs your Ghost publication. The CLI generates a NGINX config file for your domain and saves it in `[path/to/ghost/]system/files/your-domain.com.conf` with a secondary config file for SSL: `[path/to/ghost/]system/files/your-domain.com-ssl.conf`.

## SSL

The CLI generates a free SSL certificate from [Let’s Encrypt](#lets-encrypt) using [acme.sh](#lets-encrypt) and a secondary NGINX config file to serve https traffic via port 443.

#### SSL configuration

After a successful ssl setup, you can find your ssl certificate in `/etc/letsencrypt`.


#### SSL for additional domains

You may wish to have multiple domains that redirect to your site, e.g. to have an extra TLD or to support www. domains. **Ghost itself can only ever have one domain pointed at it.** This is intentional for SEO purposes, however you can always redirect extra domains to your Ghost install using nginx.

If you want to redirect an HTTPS domain, you must have a certificate for it. If you want to use Ghost-CLI to generate an extra SSL setup, follow this guide:

```bash

# Determine your secondary URL
ghost config url https://my-second-domain.com

# Get Ghost-CLI to generate an SSL setup for you:
ghost setup nginx ssl

# Change your config back to your canonical domain
ghost config url https://my-canonical-domain.com

# Edit the nginx config files for your second domain to redirect to your canonical domain. In both files replace the content of the first location block with:
return 301 https://my-canonical-domain.com$request_uri;

# Get nginx to verify your config
sudo nginx -t

# Reload nginx with your new config
sudo nginx -s reload
```


#### Let's Encrypt

[Let’s Encrypt](https://letsencrypt.org/) provides SSL certificates that are accepted by browsers free of charge! This is provided by the non-profit Internet Security Research Group (ISRG). The Ghost-CLI will offer you to generate a free SSL certificate as well as renew it every 60 days.

Ghost uses [acme.sh](https://github.com/Neilpang/acme.sh) for provisioning and renewing SSL certificates from Let's Encrypt. You can call `acme.sh` manually if you need to perform extra tasks. The following command will output all available options:

```bash
/etc/letsencrypt/acme.sh --home "/etc/letsencrypt"
```


## Systemd

`systemd` is the default way of starting and stopping applications on Ubuntu. The advantage is that if Ghost crashes, `systemd` will restart your instance. This is the default recommended process manager.

## Permissions

Ghost-CLI will create a new system user and user-group called `ghost` during the installation process. The `ghost` user will be used to run your Ghost process in `systemd`.

This means that Ghost will run with a user that has no system-wide permissions or a shell that can be used (similar to other services such as NGINX). Sudo is required to modify files in the The  `<install-directory>/content/`.

To prevent accidental permissions changes, it's advisable to execute tasks such as image upload or theme upload using Ghost admin.


### File Permissions

The `ghost-cli` enforces default linux permissions (via `ghost doctor` hooks) for installations.

- For normal users, default directory permissions are 775, and default file permissions are 664.
- For root users, default directory permissions are 755, and default file permissions are 644.

Running ghost install as the non-root user will result in directories created with 775 (`drwxrwxr-x`) permissions and file with 664 (`-rw-rw-r--`) permissions.

These file permissions don't need to be changed. The only change that is executed by ghost-cli is changing ownership, file permissions stay untouched.

If permissions were changed, the following two commands will revert file and directory permissions to the ones of a non-root user.

```bash
sudo find /var/www/ghost/* -type d -exec chmod 775 {} \;
sudo find /var/www/ghost/* -type f -exec chmod 664 {} \;
```

The cli doesn't support directory flags such as `setuid` and `setguid`). If your commands keep failing because of file permissions, ensure your directories have no flags!

## Keeping Ghost CLI updated

It's important to keep `ghost-cli` up to date, as well as Ghost. If your CLI is out of date and you've been running certain commands in your terminal then you may see an error message that your version of the CLI is outdated. To upgrade use:

```bash

# Using npm
sudo npm i -g ghost-cli@latest

# Using yarn
sudo yarn global add ghost-cli@latest
```
