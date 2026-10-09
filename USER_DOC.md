# User Documentation

This document explains how to use and manage the Inception infrastructure.

## Services provided by the Stack

*Web server(NGINX)* -> *Website(WordPress)* -> *Database(MariaDB)*

- **NGINX:** Provides HTTPS access to the website.
The main entry point, allowing all the services to use Internet access.
- **WordPress with PHP-FPM:** Allows users to publish and manage website content.
- **MariaDB:** Serves as a database to store the website's content, users, and settings.


## Launch and terminate the project

### Start the project

>[!NOTE]

> Before launching the project, make sure you have all the required configuration files,
> as /secrets with the credential information and the .env file in /srcs.

1. If the VM is stored on an external drive, connect and mount the drive.
2. Open Oracle VirtualBox and start the project's virtual machine.
3. Log in to VM.
4. Open a terminal inside the VM, or connect to it through SSH from your computer.
   ```bash
   ssh login@vm_ip_address
   ```
5. Navigate to the project's root directory, where the Makefile is located.
6. Start the project in the background:
   ```bash
   make
   ```
### Stop the project

From the project's root directory, to stop the services before shutting down the VM, run:
```bash
make stop
```
### Shutdown the virtual machine

When you have finished working, run this command inside the VM or its SSH session:
```bash
sudo shutdown -h now
```
*Wait for until the VM is powered off before ejecting the external drive, if any*


## Website access and the Administration Panel

### Access from the host computer through an SSH Tunnel

If you do not have administrator access on the host computer to configure 
domain resolution, use an SSH tunnel and a browser SOCKS proxy.

1. Start the VM, keep the session on VM opened
2. Open a terminal on your host and run:
   ```bash
   ssh -N -D 1080 login@ip_address
   ```
   *If SSH uses a different port, specify it with `-p PORT`.*
   > [!NOTE]
   > Keep this SSH tunnel terminal open while accessing the website.
3. In Firefox, open **Settings -> General -> Network Settings -> Settings**.
4. Select **Manual proxy configuration** and enter:
   - **SOCKS Host:** `127.0.0.1`
   - **Port:** `1080`
   - Select **SOCKS v5**
   - Enable **Proxy DNS when using SOCKS v5**
5. Save the settings.
### Open the website

- **WordPress website:** <https://sivashch.42.fr>
- **WordPress admin panel and user login:** <https://sivashch.42.fr/wp-admin>

> [!NOTE]
> The browser may display a security warning because the server uses a
> self-signed TLS certificate. Accept the warning to access the website.

Sign in using one of the following WordPress accounts:
- **Administrator (`wpuser`):** Manages website content, users, and settings.
- **Second user (`secwpuser`):** Has access according to its assigned role.

See the credentials section for password locations.

### After browsing
>[!IMPORTANT]
> Restore Firefox’s **Use system proxy settings** option.
> Close the SSH tunnel by pressing `Ctrl+C` in its terminal.


## Location and management of the credentials
Configuration and credentials are stored in the following locations:
- **`srcs/.env`:** Contains service configuration values and usernames.
- **`secrets/credentials.txt`:** Contains the WordPress administrator and second-user passwords.
- **`secrets/db_password.txt`:** Contains the MariaDB user's password.
- **`secrets/db_root_password.txt`:** Contains the MariaDB root password.

The `secrets/` directory is located at the project root and is ignored by Git.
>[!NOTE]
>These files must be provided locally before the first launch.
### Manage existing credentials

>[!IMPORTANT]
> On the prepared VM, the configuration and credential files are already present.
> For a fresh installation from the Git repository, these files must be
created or securely copied separately, because Git ignores them.

After initialization, editing these files and restarting the project
does not automatically change existing accounts.


## Verification of the services running correctly

### Check container status

From the project's root directory, run:

```bash
docker compose -f srcs/docker-compose.yml ps -a
```
*The `nginx`, `wordpress`, and `mariadb` containers should show a running
status (`Up`).*

### Check website response and website access

Inside the VM, run:

```bash
curl -kI https://sivashch.42.fr
```

- `-k` allows the self-signed certificate.
- `-I` requests only the HTTP response headers.

An `HTTP/1.1 200 OK` response indicates that the website responds successfully.
A `502 Bad Gateway` response indicates a problem between NGINX and WordPress.

Open <https://sivashch.42.fr> in your browser, using the connection
method described in **Section 3**.

Confirm that the website loads and that you can sign in to WordPress.
### Investigate a problem
To view service logs, run:

```bash
docker compose -f srcs/docker-compose.yml logs --tail=50
```
