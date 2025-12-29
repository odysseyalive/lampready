# Why Debian LAMPready?

---

A LAMP is an acronym describing a Web server that runs on a Linux platform with
[Apache](https://httpd.apache.org/), [MariaDB](https://www.mariadb.org/) (MySQL) and
[PHP](https://www.php.net/).  Web pages with [Python](https://www.python.org/)
and [Perl](https://www.perl.org/) can also be served under this environment.
References to a LAMP stack generally relate to an environment where website
code can be tested prior to professional deployment.

There are a lot of great LAMP packages out there. However, I wanted something I could spin up fast without a lot of manual intervention. Each website is unique in the way it is served and maintained. This LAMP stack is not too different from an actual working Web server, but built on Debian 12 (Bookworm).

As a general warning though, this environment is ***only intended*** to be used for debugging code. If anything, most LAMP stacks are way oversimplified and are in no way ready for production.

How it Works
---

This script syncs your project's root folder ***safely*** inside a [container](https://developers.redhat.com/blog/2019/01/15/podman-managing-containers-pods#). That allows you to work on and debug code, mimicking a production Web server environment. Most of these virtual environments do not have mail utilities set up, because dealing with the fallout of development spam is never fun!

Sometimes those forms and crons need tested. LAMPready uses [Postfix](https://www.postfix.org/) to route all outbound mail to a single convenient inbox which can be accessed with the ***bash box mail*** command.

This script is based off [Scott McCarty's](https://crunchtools.com/moving-linux-services-to-containers) conversation on using a single Podman container to build a LAMP stack. If you use [Podman](https://podman.io/), you might want to test this example out. If you aren't familiar with Podman, hopefully a quick scan through this script will demystify the process.

The Debian Box
---

In this example Podman will download the [jrei/systemd-debian](https://hub.docker.com/r/jrei/systemd-debian) image. This is an init image running the 12 release of [Debian Bookworm](https://www.debian.org/releases/bookworm/).

It should work on any Linux distribution that runs on [systemd](https://systemd.io/), which is most of them. Both Podman and Buildah will also need installed.

Enhanced Security Features
---

This version includes several security and usability enhancements:

* Enhanced permission management with proper user mapping instead of running Apache as root
* Apache now runs on standard ports (80/443) using proper user permissions
* Files created by the container will have proper ownership matching your host user
* Container processes run with limited privileges using security options

NOTE: If you receive permission denied notices for vm.max_map_count, run the following
command from root:

```
sysctl -w vm.max_map_count=262144
```

This increases the mmap counts in mappfs.  To make this change permanent add the
following line to the bottom of the /etc/sysctl.conf file and reboot.

```
vm.max_map_count=262144
```

Being Rootless
---

LAMPReady is a rootless container. As such, there are standard practices to
setting up a web server that have been broken simply for the sake of
productivity.  This was done to simplify the permissions issues that arise
when working with shared data within the container.

If you plan on using the standard ports 80 and 443, you will need to expose privileged
ports.  Run the following commands under root to do that:

```
echo "net.ipv4.ip_unprivileged_port_start=0" > /etc/sysctl.d/05-expose-privileged.conf
```

```
sysctl --system
```

```
sysctl net.ipv4.ip_unprivileged_port_start=0
```

User Linger Required
---

Rootless containers managed by systemd require user lingering to be enabled. Before running `bash box up`, you'll need to enable it for your user:

```
sudo loginctl enable-linger $USER
```

How to Use Debian LAMPready
---

Download the box script into your project's root directory.

Debian 12 Bookworm

```
curl https://raw.githubusercontent.com/odysseyalive/lampready/refs/heads/master/debian-box > box
```

**PLEASE NOTE:** Be sure to review the notes at the head of the box script.
Xdebug is disabled by default but can be enabled for debugging.

The rootless container now uses proper user mapping to preserve the proper permissions
of shared files while running services with appropriate user permissions rather than root.
The upside is that this temporary web server will run
***much more efficiently*** than most other local VM solutions!

As you will notice further on, this script's commands closely model
[Vagrant](https://www.vagrantup.com/).  The advantage of running a container
this way is that I no longer need to maintain a fully encapsulated
operating system image on an existing database. This bodes well with
keeping better abreast with the latest release changes.

To see a list of commands for LAMPReady, just run:

```
bash box
```

* Run the following command to spin up the new virtual environment in minutes

```
bash box up
```

* Log into to shell to check out the virtual environment with:

```
bash box ssh
```

* Access the website via <https://127.0.0.1> or use your custom domain
settings (see below)

* Access outbound mail using the following command:

```
bash box mail 
```

How to Configure this Script for Your Project
---

The very first time you run the ***bash box up*** command, the hostname and
***user defined settings*** will be used to set up Apache, Xdebug, the SSL
certificate, mail relay and any databases without further intervention.

Here is a breakdown of the available options:

**HOSTNAME:**
The hostname should contain your domain name.

**SUB_DOMAIN:**
It is recommended that you choose a unique sub domain for your virtual machine.
This way your public domain is active for comparison.  Next, alter your system's
hosts file to allow the new domain to be accessed from any of your browsers.  On
Linux systems, this host file can be found at /etc/hosts.  You'll need root
access to add the following line or something similar:

```
127.0.0.1   dev.lampready.com
```

**PUBLIC_ROOT_PATH:**
This script should be placed in your project's root directory.  If your
root directory contains an index file, you'll need to leave this setting blank.
Otherwise, you'll include the local path to your index file.

The example default entry in this script assumes the following directory structure:

```
box
user_database.sql
public_html/index.php
```

In another example, let's say a Magento store resides within a subdirectory of
an existing website, you might change the root path to "public_html/store"
if the store is registered under a different domain.  Let's assume your directory
structure is as follows

```
box
user_database.sql
public_html/index.php
public_html/store/index.php
public_html/store/app
```

The above examples should demonstrate the fact that this script is
designed to set up one website with one corresponding URL out of the box.
However, you'll notice a new hidden directory formed in your project root
directory  You can add more URLS by adding the appropriate Apache configuration
files to: .container/sites-enabled

**HTTP_PORT:**
This port generally defaults to 80.  If you have root access, you are
encouraged to use this port for full compatibility of the code that
is now running in a virtual environment.

**SSL_PORT:**
This port defaults to 443, but may be changed.  Remember though, if the
port is set to something different it should be part of the URL entered
into the browser:

```
https://dev.lampready.com:443
```

**TIMEZONE**
Set the timezone for Apache using one of
[these options](https://www.php.net/manual/en/timezones.php).

**PHP_VERSION:**
This option is intended to provide easier debugging during PHP upgrades.  Any
PHP version in the [Sury repository](https://packages.sury.org/php/) is
fair game.  Most versions have access to the commonly used set of PHP modules.

**PHP_MAX_EXEC_TIME:**
This is the maximum execution time a PHP script runs before it times out.

**PHP_MEM_LIMIT:**
This is the maximum amount of memory a single PHP script can use.  Remember,
this is not a virtual environment per se.  This is a rootless container where
memory is not reserved, only used during the course of the Web server's
operations.  This makes for a more efficient environment for testing
scripts.

**XDEBUG_ENABLE:**
Xdebug is not enabled by default. Xdebug must be enabled before the initial
setup process begins.  If Xdebug is not enabled during the first initialization
of the LAMP stack, it will have to be destroyed and recreated.

**XDEBUG_PORT:**
This is the default port normally allocated for Xdebug.  You'll want to be sure
to open up your firewall to allow communication between Xdebug and your IDE.
Think about the following changes that will need to be made with the root user
in order for Xdebug to function:

```
firewall-cmd --permanent --zone=webserver --add-port=9003/tcp
```

```
semanage port -a -t http_port -p tcp 9003
```

**XDEBUG_FORCE_ERROR_DISPLAY:**
For sites constructed from many packages with different settings, it is nice to
to easily override error suppression.  Set this option to 1 to override individual
error settings throughout the code.

**XDEBUG_SCREAM:**
This is another setting that can help ensure all PHP errors are described
in the browser.

**XDEBUG_DISPLAY_MAX_DATA:**
Controls the maximum string length that will be shown in var_dump() output.

**XDEBUG_DISPLAY_MAX_DEPTH:**
Controls the maximum nested array/object depth that will be shown in var_dump() output.

**XDEBUG_DISPLAY_MAX_CHILDREN:**
Controls the maximum number of array/object children that will be shown in var_dump() output.

**DB1:**
The script can automatically set up and install any number of MySQL databases.
By filling out this information, the script will create the database and inject
your data from a sql file you have placed in the project's root directory.

**DB_NAME, DB_USER and DB_PASS:**
These settings should mirror the same database name and user credentials
written in the website's database configuration file.  

***Notice the single quotes used for the password.***  This is an important
work-around to allow the bash script to insert certain special characters correctly.

**DB_PERM:**
Specific DB permissions may be applied.

**DB_PREFIX:**
If there are any prefixes associated with your database as commonly is the case
with WordPress, this should be entered here.  An example would be: wp_

**DB_TYPE:**
If the database is Magento or WordPress, you will want to set this option to say
so.  The following arguments are valid: custom, wordpress, magento_1 or
magento_2.  The script will update the appropriate tables to match this
script's HOSTNAME and SUB_DOMAIN settings.  For typical installations no manual
fiddling should be necessary.

**DB_FILENAME:**
This option is not necessary if there is no database to import.  While the
DB_NAME option will create an empty database, this option populates that
database.  The script will look for the sql file in the same directory as
the script resides.

**DB_CUSTOM_FUNCTIONS:**
This option is generally reserved for importing custom database functions.

**CONTAINER_USER, CONTAINER_UID, CONTAINER_GID:**
Enhanced permission management allows you to specify the user account that will run
services inside the container. By default, the container will map to your host user
for seamless file sharing.

How to Use this Box?
---

After installing, LAMPready must be started and managed from the terminal:

**to initialize or start the server:**

```
bash box up
```

**stop the server:**

```
bash box halt
```

**delete the server:**

```
bash box destroy
```

**Note:** If you run bash box destroy, the box will be built from scratch again
-- which may be a good thing.

**to shell into the server:**

```
bash box ssh
```

**check if you are running any other LAMPReady instances:**

```
bash box ps 
```

**pull a fresh base image:**

```
bash box reset
```

**rebuild site with fresh base image (preserves data):**

```
bash box rebuild
```

Troubleshooting: Container Fails to Start
---

If an existing container fails to start after a system update or base image change, the issue is likely that your site image was built with an older version of the Debian base image. The solution is to pull the fresh base image and rebuild your site.

**Step 1: Pull the fresh base image**

Run this once from any project directory:

```
bash box reset
```

This will download the latest `docker.io/jrei/systemd-debian:12` image without affecting your project files or database.

**Step 2: Rebuild each project**

Navigate to each project directory and rebuild:

```
cd /path/to/your-project
bash box rebuild
```

The rebuild command will:
* Stop the running container (if any)
* Remove the old site image
* Rebuild with the fresh base image
* Start all services (Apache, MariaDB, etc.)
* **Preserve your database, site files, and Apache configs**

**Important Notes:**
* `reset` only affects the base Debian image — your projects stay intact
* `rebuild` recreates the site image but preserves all data in `.container/`
* `destroy` wipes everything including your database — use with caution

**Typical workflow when things break:**

```
bash box reset              # Pull fresh base image (run once)
cd /path/to/site1
bash box rebuild            # Rebuild site1 with new base
cd /path/to/site2
bash box rebuild            # Rebuild site2 with new base
```

Dap-Mode for Emacs
---

Dap-Mode is part of the [Emacs-Lsp](https://emacs-lsp.github.io/) package which
is a very awesome PHP IDE!  The website is running from a rootless container and
therefore needs the correct path mapping to the project's root directory.  This
would include the path within the container as well as the directory on your
machine. There are some excellent notes on
[Spacemacs's implementation](https://develop.spacemacs.org/layers/+lang/php/README.html#backends)

DAP-Mode actually uses the PHP VSCode plugin
[vscode-php-debug](https://github.com/xdebug/vscode-php-debug) which can be
installed with: M-x dap-php-setup

You'll also want to create a file called launch.json in your project's root
directory and place the following information in it:

```
{
"configurations": [

    {
        "name": "Use custom launch.json script",
        "type": "php",
        "request": "launch",
        "port": 9003,
        "pathMappings": {
            "/var/www": "/home/username/your-project-folder"
          },
        "sourceMaps": true,
        "log": false 
    }
]
}
```

After running dap-php-setup, you should have a new option called "Use custom
launch.json script" to choose.  For more information and an example Emacs
setup, check out my [dotfiles](https://github.com/odysseyalive/dotfiles).

Dap-Mode for NeoVIM
---

Dap-Mode for NeoVIM is even better than VSCode's implementation, and I have
found it much more stable than the Emacs version.

I used [mason.vim](https://github.com/williamboman/mason.nvim) to add the
vscode-php-debug extension, which never seems to load using the Emacs alternative.
Afterwards, all I needed to do was add a few lines of code to my [Nvim init file](https://github.com/odysseyalive/dotfiles/blob/master/workstation/nvim/init.lua).
The same configuration used above for Emacs will work, but it should be placed into
a subdirectory of your root folder: ~/your-project-root/.vscode/launch.json.

VIM with the Vdebug Plugin
---

Here is a sample user configuration for your .vimrc file.  This tells VIM
to look in your projects root folder for a file called .vimrc.local with
custom path mapping needed to run Vdebug with the rootless container.

```
" Look for .vimrc.local inside each project
if filereadable('.vimrc.local')
    source .vimrc.local
    endif
```

Create a file called .vimrc.local and place it in the root directory of your
project -- the same one as your box script. Be sure that the path
mapping matches the project's root directory or your break points will
not work.  The /var/www will stay the same as it is the synced location of
your project.

```
" Project Specific Settings for Vdebug
let g:vdebug_options = {
\    "port" : 9003,
\    "timeout" : 20,
\    "server" : '',
\    "on_close" : 'stop',
\    "break_on_open" : 0,
\    "ide_key" : 'xdebug',
\    "debug_window_level" : 0,
\    "debug_file_level" : 0,
\    "debug_file" : "~/.vdebug.log",
\    "path_maps" : {"/var/www" : "/home/username/your-project-folder"},
\    "watch_window_style" : 'expanded',
\    "marker_default" : '⬦',
\    "marker_closed_tree" : '▸',
\    "marker_open_tree" : '▾',
\    'sign_breakpoint' : '▷',
\    'sign_current' : '▶',
\    "continuous_mode"  : 1,
\    'simplified_status': 1,
\    'layout': 'vertical',
\}
let g:vdebug_features = {
\ 'max_data': 512,
\ 'max_children': 128,
\ 'max_depth': 3,
\}
```

Enhanced Security with Proper User Mapping
---

This development environment uses proper user mapping instead of running Apache as root, which provides enhanced security while maintaining the convenience of seamless file sharing between host and container.

**The Problem That Was Solved**

When you're working with containers and trying to share files between your host
machine and the container, you run into this annoying permission nightmare:

* Your files on the host are owned by you (let's say UID 1000)
* Apache inside the container runs as www-data (UID 33)
* Suddenly nothing works right - Apache can't read your files, you can't edit
  files that Apache creates, and you spend more time fighting permissions than
  actually developing

**How It's Solved Now**

The enhanced version of this script implements proper user mapping, which allows:

* Services inside the container to run with appropriate user permissions
* Seamless file sharing between host and container with correct ownership
* Enhanced security by avoiding root privileges for web services
* Proper isolation while maintaining development convenience

**CONTAINER_USER, CONTAINER_UID, CONTAINER_GID Settings**

You can now customize the user account that will run services inside the container
using the CONTAINER_USER, CONTAINER_UID, and CONTAINER_GID settings. By default,
the container will map to your host user for seamless file sharing.

**Benefits of This Approach**

* Services run with appropriate permissions rather than as root
* Files created by the container maintain proper ownership matching your host user
* Enhanced security while preserving development convenience
* No more permission headaches when accessing files from both host and container

**Why This is Better for Development and Security**

This approach provides the best of both worlds:

* Your container is properly isolated with appropriate user permissions
* It's temporary - you destroy it when you're done working
* It solves real problems - no more permission headaches
* You can actually focus on developing instead of fighting with file ownership
* Enhanced security by avoiding unnecessary root privileges

---

**CREDITS:**

This script was based on Scott McCarty's contribution to this subject,
A Hacker's Guide to Moving Linux Services Into Containers:

<https://crunchtools.com/moving-linux-services-to-containers>

The Debian adaptation maintains compatibility with the original Ubuntu version
while using Debian 12 (Bookworm) as the base system. Because sometimes you
just want things to work without fighting the operating system.
