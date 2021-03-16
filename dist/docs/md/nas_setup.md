# IOTstack homelab

Author: Ewout van Nimwegen

## Guide

#### <b>Hardware</b>

- [Raspberry Pi 3B](https://www.raspberrypi.org/products/raspberry-pi-3-model-b/)
  with [Raspberry Pi OS Lite](https://www.raspberrypi.org/software/operating-systems/#raspberry-pi-os-32-bit) kernel version 5.4
- Seagate 1 TB HDD & cable

- Ethernet cable

- MicroUSB cable to USB

- Ali Express 5V 3A power supply

- Ziggo router

<b>Update system</b>

```console
PI :~ $ sudo apt-get update -y && sudo apt-get dist-upgrade -y
```

#### <b>Security</b>

To secure the pi setup a ssh key then remove the remote password setting.

<b>SSH key</b>

```console
PI :~ $ mkdir ~/.ssh
PC :~ $ ssh key-gen && scp ~/.ssh/id_rsa.pub hostname@ip:~/.ssh/authorized_keys
```

<b>Remove remote password login</b>

Edit the ssh demon config file.

```console
PI :~ $ sudo vim /etc/ssh/sshd_config
```

Restart the ssh demon.

```console
PI :~ $ sudo systemctl restart sshd.service
```

#### <b>Dependencies</b>

Raspberry Pi

```console
PI :~ $ sudo apt-get install git curl vim -y
```

PC

- [nextcloud](https://nextcloud.com/install/#install-clients)

- [wireguard](https://www.wireguard.com/install/)

<b>[IOTstack](https://github.com/SensorsIot/IOTstack)</b>

```console
PI :~ $ curl -fsSL \
https://raw.githubusercontent.com/SensorsIot/IOTstack/master/install.sh | bash
PI :~ $ cd IOTstack && ./menu.sh
```

For this examples I will install the following containers:

- [adminer](https://www.adminer.org/)

- [mariadb](https://mariadb.org/)

- [mosquitto](https://mosquitto.org/)

- [nextcloud](https://nextcloud.com/)

- [nodered](https://nodered.org/)

- [portainer-ce](https://www.portainer.io/)

- [wireguard](https://www.wireguard.com/)

Create the <b>compose_override.yml</b> file before building the stack.
As the filename suggests, this file overrides the generated config file.
So we do not need to worry about editting the config file after adding
or removing containers.

```console
PI :~ $ touch ~/IOTstack/compose_override.yml && \
vim ~/IOTstack/compose_override.yml
```

```yaml
services:
  wireguard:
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Amsterdam
      - SERVERURL=connect-me.duckdns.org
      - SERVERPORT=51820
      - PEERS=10

  nextcloud:
    volumes:
      - ./volumes/nextcloud/html:/var/www/html

  nextcloud_db:
    environment:
      - MYSQL_ROOT_PASSWORD=rootPassword
      - MYSQL_PASSWORD=mysqlPassword
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud

  mariadb:
    environment:
      - MYSQL_ROOT_PASSWORD=rootPassword
      - MYSQL_PASSWORD=mysqlPassword
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud
```

Build and launch the stack. Then launch the portainer-ce browser page with
<b>http://ip:9000</b>. Use this gui to easily manage your containers.

#### <b>Dynamic DNS</b>

Dynamic DNS is a method to automatically update the I.P. address of
your self hosted instance. We will use <b>DuckDNS</b> for this example.
Go to <b>[duckdns.org](https://www.duckdns.org/)</b> and create an account.
Get a domain, I'm using <b>connect-me.duckdns.org</b>, it is possible to
duplicate mine, because every account is bound to an address and a specific
token.

```console
PI :~ $ vim ~/IOTstack/duck/duck.sh
```

Add the domain and token.

```sh
DOMAINS="connect-me.duckdns.org"
DUCKDNS_TOKEN="r61bg77a-cd9b-4f48-ad34-560bcf543221"
```

Execute the script every 5 minutes with crontab.

```console
PI :~ $ crontab -e
```

Add this line.

```sh
*/5 * * * * sudo ~/IOTstack/duck/duck.sh >/dev/null 2>&1
```

#### <b>VPN setup</b>

<b>Security key</b>
We've already installed Wireguard. Copy the security key or QR code to your
devices.

With scp:

```console
PI :~ $ scp IOTstack/services/wireguard/config/peer1/peer1.png \
hostname@ip:~/peer1.png
```

<b>Port forwarding</b>
Open a webpage and enter your routers I.P. address.
Allow all external I.P.'s to forward data through <b>UDP port 51820</b> to your
device's I.P. address.

Use the wireguard app to connect to your VPN.

## Issues

I've walked into a few issues they are listed below with the recommended
solutions.

1. Wireguard missing kernel headers.

console log

```sh
**** Kernel headers don't seem to be available, can't compile the module. Sleeping now. . . ****
```

<b>Solution</b> install the stable kernel headers.
[source](https://github.com/linuxserver/docker-wireguard/issues/57)

```console
PI :~ $ sudo apt install --reinstall libraspberrypi0  \
libraspberrypi-{bin,dev,doc} raspberrypi-bootloader raspberrypi-kernel
```

Then restart the containers and it should all be fixed.

2. IPv4 port forwarding not possible with Ziggo router.

<b>Solution</b> contact Ziggo, ask them to downgrade your routers firmware.
to support the IPv4 forwarding.

## Sources

- [IOTstack](https://sensorsiot.github.io/IOTstack)

- [Docker](https://www.docker.com/)
