# NetServa

**Code removed due to copyright notice removal complaint**

A server admin and user control panel for [Ubuntu] (14.10) using [nginx],
[php5-fpm], [bind] and [courier-mta/imap].

This a port of [Froxlor] 0.9.32 with the [Twinkle] theme as default.

## Goals

- reduce overall code size by targetting Ubuntu 14.04+
- focus on only nginx + php5-fpm, Bind and ssh/sftp
- replace the postfix mailserver with courier-mta
- replace Smarty templates with functions that return strings
- require all code to go through index.php front controller
- adopt AGPL-3.0 license when more than 50% of the code is rewritten
- add client LXC VPS-like container options
- eventually include the [Amberdms] billing system


As of **2014-10-01** the new page layout seems to be working with the main
menu down the LHS. In mobile mode it slides in OVER the main content instead
of pushing the page to the right.

![First snapshot](/lib/img/NetServa_20141007.jpg)

## Note

As of **2014-10-07** the web interface seems to work fairly well but the
services under the hood still need a lot of work, especially courier-mta
which may work once set up but is largely untested. For now I will leave
this version alone and start on a fork of this project to convert most of
the Smarty templates to a native PHP variation. I'd rather not work in a
branch because it's a massive change so I will simply fork this project
yet again and call the new fork `sysadm`, for now. What will remain is
the database schema, the PDO database calls and the templates converted
to native PHP. The code base itself will be rewritten from scratch using
a very simple "framework" I have used for other projects. A lot of
functionality lke the current session and logging schemes will be left
behind, as well and languages and themes, all will be reduced to a bare
minimum... and hopefully stay that way!

## Setup

This is a first attempt at an install script. These 2 files install the
right packages and attempt to auto configure them. The rest of the system
still needs to be set up and this script(s) will evolve.

#### /netserva

```
#!/bin/bash
# netserva script created on Mon Oct  6 13:01:24 AEST 2014

cd

echo "Adding nginx and ondrej PPA keys"
apt-key adv -qq --recv-keys --keyserver keyserver.ubuntu.com 00A6F0A3C300EE8C > /dev/null 2>&1
apt-key adv -qq --recv-keys --keyserver keyserver.ubuntu.com 4F4EA0AAE5267A6C > /dev/null 2>&1

echo "Updating package lists"
apt-get -qq update

echo "Initial package count: "$(dpkg --get-selections | wc -l)

echo "Downloading required NetServa packages"
debconf-set-selections /preseed
DEBIAN_FRONTEND=noninteractive apt-get -qq install -y -d --no-install-recommends \
 bind9 bind9utils ca-certificates courier-authlib-mysql courier-authlib-sqlite \
 courier-imap-ssl courier-maildrop courier-mta-ssl git libterm-readline-gnu-perl \
 mariadb-client mariadb-server nano nginx openssh-server php5-cli php5-curl \
 php5-fpm php5-json php5-mcrypt php5-mysql php5-sqlite rsync wget whois > /dev/null 2>&1

echo "Installing required NetServa packages"
debconf-set-selections /preseed
DEBIAN_FRONTEND=noninteractive apt-get -qq install -y --no-install-recommends \
 bind9 bind9utils ca-certificates courier-authlib-mysql courier-authlib-sqlite \
 courier-imap-ssl courier-maildrop courier-mta-ssl git libterm-readline-gnu-perl \
 mariadb-client mariadb-server nano nginx openssh-server php5-cli php5-curl \
 php5-fpm php5-json php5-mcrypt php5-mysql php5-sqlite rsync wget whois > /dev/null 2>&1

echo "Updated package count: "$(dpkg --get-selections | wc -l)

if [ ! -d .sh ]; then
  echo "Installing custom shell aliases and commands"
  git clone http://github.com/markc/sh .sh > /dev/null 2>&1
  .sh/shm install > /dev/null 2>&1
fi
```

#### /preseed

```
tzdata tzdata/Areas select Australia
tzdata tzdata/Zones/Australia select Brisbane
debconf debconf/priority select critical
debconf debconf/frontend select Noninteractive
courier-base courier-base/webadmin-configmode boolean true
mariadb-server-5.5 mysql-server/root_password password changeme
mariadb-server-5.5 mysql-server/root_password_again password changeme
mariadb-server-5.5 mysql-server-5.5/postrm_remove_databases boolean false
courier-mta courier-mta/dsnfrom string mailer-daemon@ns2.lan
courier-mta courier-mta/defaultdomain string ns2.lan
```

Change the "changeme" password reference and execute "bash /netserva".

[Ubuntu]: http://www.ubuntu.com/download/
[nginx]: http://nginx.com/
[php5-fpm]: http://php.net/manual/en/install.fpm.php
[bind]: http://www.bind9.net/
[courier-mta/imap]: http://www.courier-mta.org/
[Froxlor]: http://www.froxlor.org/
[Twinkle]: https://github.com/oschn0r/Twinkle
[Bootstrap]: http://getbootstrap.com/
[Amberdms]: http://www.amberdms.com/

