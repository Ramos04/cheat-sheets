# VyOS

| Modifications                |
| :--------------------------- |
| Thu Jan 16 13:41:01 CST 2020 |

## Installation 
After booting the device with the iso file run 

```bash 
$ install image
```

Follow the prompts which will appear similar to the following output

```bash 
vyos@vyos:~$ install image
Welcome to the VyOS install program.  This script
will walk you through the process of installing the
VyOS image to a local hard drive.
Would you like to continue? (Yes/No) [Yes]: Yes
Probing drives: OK
Looking for pre-existing RAID groups...none found.
The VyOS image will require a minimum 2000MB root.
Would you like me to try to partition a drive automatically
or would you rather partition it manually with parted?  If
you have already setup your partitions, you may skip this step

Partition (Auto/Parted/Skip) [Auto]:

I found the following drives on your system:
 sda    4294MB

Install the image on? [sda]:

This will destroy all data on /dev/sda.
Continue? (Yes/No) [No]: Yes

How big of a root partition should I create? (2000MB - 4294MB) [4294]MB:

Creating filesystem on /dev/sda1: OK
Done!
Mounting /dev/sda1...
What would you like to name this image? [1.2.0-rolling+201809210337]:
OK.  This image will be named: 1.2.0-rolling+201809210337
Copying squashfs image...
Copying kernel and initrd images...
Done!
I found the following configuration files:
    /opt/vyatta/etc/config.boot.default
Which one should I copy to sda? [/opt/vyatta/etc/config.boot.default]:

Copying /opt/vyatta/etc/config.boot.default to sda.
Enter password for administrator account
Enter password for user 'vyos':
Retype password for user 'vyos':
I need to install the GRUB boot loader.
I found the following drives on your system:
 sda    4294MB

Which drive should GRUB modify the boot partition on? [sda]:

Setting up grub: OK
Done!
```

## Upgrading 

Currently VyOS free rolling releases can be found on the [VyOs website](https://downloads.vyos.io/?dir=rolling/current/amd64)

Find an image you would like and copy the url to the iso file. 

To upgrade the system run the command

```bash 
$ add system image <url to iso image> 
```

The output should be similar to 

```bash 
vyos@vyos:~$ add system image https://downloads.vyos.io/release/1.1.8/vyos-1.1.8-amd64.iso
Trying to fetch ISO file from https://downloads.vyos.io/release/1.1.8/vyos-1.1.8-amd64.iso
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  234M  100  234M    0     0  4903k      0  0:00:48  0:00:48 --:--:-- 6078k
ISO download succeeded.
Checking for digital signature file...
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   836  100   836    0     0   1997      0 --:--:-- --:--:-- --:--:--  5805
Found it.  Checking digital signature...
gpg: directory `/root/.gnupg' created
gpg: new configuration file `/root/.gnupg/gpg.conf' created
gpg: WARNING: options in `/root/.gnupg/gpg.conf' are not yet active during this run
gpg: keyring `/root/.gnupg/pubring.gpg' created
gpg: Signature made Wed Feb 17 06:37:47 2016 MST using RSA key ID A0FE6D7E
gpg: /root/.gnupg/trustdb.gpg: trustdb created
gpg: Good signature from "VyOS Maintainers (VyOS Release) <maintainers@vyos.net>"
gpg: WARNING: This key is not certified with a trusted signature!
gpg:          There is no indication that the signature belongs to the owner.
Primary key fingerprint: 0694 A923 0F51 39BF 834B  A458 FD22 0285 A0FE 6D7E
Digital signature is valid.
Checking MD5 checksums of files on the ISO image...OK.
Done!
What would you like to name this image? [VyOS-1.1.8]: [return]
OK.  This image will be named: VyOS-1.1.8
Installing "VyOS-1.1.8" image.
Copying new release files...
Would you like to save the current configuration 
directory and config file? (Yes/No) [Yes]: yes
Copying current configuration...
Would you like to save the SSH host keys from your 
current configuration? (Yes/No) [Yes]: yes
Copying SSH keys...
Setting up grub configuration...
Done.
```

## Backup Config
To back up the current configuration to a file

### Command
```bash
show configuration commands > config.txt
```

### Script
Copy the following commands into a file, hange the ``$BACKUPDIR`` to the full path of the directory you want to contain your backups, and make the file executable. 

```bash
#!/bin/vbash
source /opt/vyatta/etc/functions/script-template

$BACKUPDIR="/home/vyos/backup/"

run show configuration commands > $BACKUPDIR/$(date "+%Y-%m-%d")_$(hostname).txt
```

## Quick Start

Below is the simple configuration commands for setting up two network interfaces, one connected to a WAN and the other serving DHCP to a LAN. 

#### Set up the WAN network interface

```bash 
set interfaces ethernet eth0 address dhcp
set interfaces ethernet eth0 description 'OUTSIDE'
```

#### Set up the LAN network interface

```bash 
set interfaces ethernet eth1 address '192.168.0.1/24'
set interfaces ethernet eth1 description 'INSIDE'
```

#### Enable SSH access

```bash 
set service ssh port '22'
```

#### Configure NAT masquerading for the LAN interface 

```bash 
set nat source rule 100 outbound-interface 'eth0'
set nat source rule 100 source address '192.168.0.0/24'
set nat source rule 100 translation address masquerade
```

#### Configure DHCP Server for the LAN interface

The DHCP start and stop range can be changed to your choice

```bash 
set service dhcp-server shared-network-name LAN subnet 192.168.0.0/24 default-router '192.168.0.1'
set service dhcp-server shared-network-name LAN subnet 192.168.0.0/24 dns-server '192.168.0.1'
set service dhcp-server shared-network-name LAN subnet 192.168.0.0/24 domain-name 'internal-network'
set service dhcp-server shared-network-name LAN subnet 192.168.0.0/24 lease '86400'
set service dhcp-server shared-network-name LAN subnet 192.168.0.0/24 range 0 start 192.168.0.50
set service dhcp-server shared-network-name LAN subnet 192.168.0.0/24 range 0 stop '192.168.0.254'
```

#### Add a DNS forwarder for the LAN

The name-server can be set to any DNS server of your choice

```bash 
set service dns forwarding cache-size '0'
set service dns forwarding listen-on 'eth1'
set service dns forwarding name-server '8.8.8.8'
set service dns forwarding name-server '8.8.4.4'
```

#### Add firewall rules from WAN interface destined for the LAN interface

This rule allows incoming traffic if its related or established and by default drops all other traffic 

```bash 
set firewall name OUTSIDE-IN default-action 'drop'
set firewall name OUTSIDE-IN rule 10 action 'accept'
set firewall name OUTSIDE-IN rule 10 state established 'enable'
set firewall name OUTSIDE-IN rule 10 state related 'enable'
```

#### Add firewall rule from WAN that is destined for the VyOS box 

These rules allow all related and established traffic, allow ICMP traffic, allow tcp traffic on port 22 (SSH)

```bash
set firewall name OUTSIDE-LOCAL default-action 'drop'
set firewall name OUTSIDE-LOCAL rule 10 action 'accept'
set firewall name OUTSIDE-LOCAL rule 10 state established 'enable'
set firewall name OUTSIDE-LOCAL rule 10 state related 'enable'
set firewall name OUTSIDE-LOCAL rule 20 action 'accept'
set firewall name OUTSIDE-LOCAL rule 20 icmp type-name 'echo-request'
set firewall name OUTSIDE-LOCAL rule 20 protocol 'icmp'
set firewall name OUTSIDE-LOCAL rule 20 state new 'enable'
set firewall name OUTSIDE-LOCAL rule 31 action 'accept'
set firewall name OUTSIDE-LOCAL rule 31 destination port '22'
set firewall name OUTSIDE-LOCAL rule 31 protocol 'tcp'
set firewall name OUTSIDE-LOCAL rule 31 state new 'enable'
```

#### Apply firewalls to the WAN interface 

```bash 
set interfaces ethernet eth0 firewall in name 'OUTSIDE-IN'
set interfaces ethernet eth0 firewall local name 'OUTSIDE-LOCAL'
```

#### Save Configuration 
```bash 
vyos@vyos# commit
vyos@vyos# save
Saving configuration to '/config/config.boot'...
Done

vyos@vyos# exit
vyos@vyos$
```

## Configuration Workflow

Workflow:

```
vyos@vyos $ configure
vyos@vyos # [configuration commands]
vyos@vyos # commit
vyos@vyos # save
vyos@vyos # exit
vyos@vyos $
```

## System information

The show command is used to show system information, tab complete is your friend, after each part of a command you can hit \<tab\> to list all of your options

| Action                  | Command                                             |
| :---                    | :---                                                |
| Show configuration      | `show`                                              |
| Show Interface          | `show interfaces`                                   |
| Show Specific Interface | `show interfaces ethernet <interface name>`         |
| Show Firewall Rules     | `show firewall`                                     |
| Show Specific Firewall  | `show firewall name <FW name>`                      |
| TCP Dump                | `monitor traffic interface <all or interface name>` |

## NAT

### Destination NAT

DNAT is typically referred to as a Port Forward. When using VyOS as a NAT router and firewall, a common configuration task is to redirect incoming traffic to a system behind the firewall.

In this example, we will be using the example Quick Start configuration above as a starting point.

To setup a destination NAT rule we need to gather:
* The interface traffic will be coming in on
* The protocol and port we wish to forward
* The IP address of the internal system we wish to forward traffic to

In our example, we will be forwarding web server traffic to an internal web server on 192.168.0.100.

#### Add NAT destination rule 

**NOTE**: Only add the last line if you are forwarding traffic to a different port than the port it is coming in on, on the WAN interface
```bash
set nat destination rule 10 description 'Port Forward: HTTP to 192.168.0.100'
set nat destination rule 10 destination port '80'
set nat destination rule 10 inbound-interface 'eth0'
set nat destination rule 10 protocol 'tcp'
set nat destination rule 10 translation address '192.168.0.100'
set nat destination rule 10 translation port '22'
```

#### Set firewall rule 

When port forwarding it is important to note that NAT rules are interpreted first, then firewall rules are interpreted. So if you set a NAT rule to forward WAN traffic on port 4444 to port 22 on an internal IP, the firewall rule will need to allow allow traffic destined for the internal IP on the port that you are forwarding to.

```bash 
set firewall name OUTSIDE-IN rule 20 action 'accept'
set firewall name OUTSIDE-IN rule 20 destination address '192.168.0.100'
set firewall name OUTSIDE-IN rule 20 destination port '80'
set firewall name OUTSIDE-IN rule 20 protocol 'tcp'
set firewall name OUTSIDE-IN rule 20 state new 'enable'
```

#### 1 to 1 DNAT

```bash 
set interfaces ethernet eth0 address '192.168.1.1/24'
set interfaces ethernet eth0 description 'Inside interface'
set interfaces ethernet eth1 address '1.2.3.4/24'
set interfaces ethernet eth1 description 'Outside interface'
set nat destination rule 2000 description '1-to-1 NAT example'
set nat destination rule 2000 destination address '1.2.3.4'
set nat destination rule 2000 inbound-interface 'eth1'
set nat destination rule 2000 translation address '192.168.1.10'
set nat source rule 2000 description '1-to-1 NAT example'
set nat source rule 2000 outbound-interface 'eth1'
set nat source rule 2000 source address '192.168.1.10'
set nat source rule 2000 translation address '1.2.3.4'
```

### Source NAT

Used for translation the IP addresses on the LAN to the IP address of the router before sending out the traffic

```bash 
set nat source rule 100 outbound-interface 'eth0'
set nat source rule 100 source address '192.168.0.0/24'
set nat source rule 100 translation address 'masquerade'
```

## Script template

Use the following as a template for a configuration script:

```Bash
#!/bin/vbash
source /opt/vyatta/etc/functions/script-template

configure

# Fix for error "INIT: Id "TO" respawning too fast: disabled for 5 minutes"
delete system console device ttyS0

# Commands here

commit
save
```

## Resources

* [VyOS homepage](http://vyos.net/)
* [User Guide](http://vyos.net/wiki/User_Guide)
* [Unofficial Vyatta Wiki](http://wiki.het.net/)
* [Higebu's Git repos](https://github.com/higebu?tab=repositories)

## Acknowledgement
* [bertvv's VyOS cheatsheet](https://github.com/bertvv/cheat-sheets/blob/master/docs/VyOS.md)
