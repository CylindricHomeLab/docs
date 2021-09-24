---
title: 'TrueNAS Notes'
taxonomy:
    category:
        - TrueNAS
    tag:
        - truenas
        - plex
---

This is not a guide to teach TrueNAS. If you need help installing TrueNAS, managing disks, pools, jails, or anything, go a googlin'...

# Multimedia Sharing

* Plex - [http://jail_ip:32400/](http://172.29.14.117:32400/)
* Radarr - [http://jail_ip:7878/](http://172.29.14.124:7878)
* Sonarr - [http://jail_ip:8989/](http://172.29.14.125:8989)
* SABnzbd - [http://jail_ip:8090/](http://172.29.14.194:8080/)

## TrueNAS Setup
1. Create a Pool to hold the media (e.g. `data1`)
1. Create a Dataset to hold the media (e.g. `data1/media`)
1. Create a new User:  
   name: `multimedia`
   id: `1000` (make a note of whatever this is on your NAS, also the group ID)
1. Create new Users and Groups for the various jails:  
    ```
    pw groupadd plex -g 972
    pw useradd plex -u 972 -g plex -s /usr/sbin/nologin -d /var/empty

    pw groupadd sabnzbd -g 350
    pw useradd sabnzbd -u 350 -g sabnzbd -s /usr/sbin/nologin -d /var/empty
    ```
3. Create a new SMB share:  
   path: `/mnt/data1/media`  
   name: `media`  
   purpose: `Default share parameters`  
   description: `Multimedia`  
   enabled: `yes`
4. Edit the Share ACLs:  
   Share Name: `media`  
   * ACL1 SID: `S-1-1-0`  
     ACL1 Domain: `blank`  
     ACL1 Name: `Everyone`  
     ACL1 Permission: `FULL`  
     ACL1 Type: `ALLOWED`
5. Edit the Filesystem ACL:  
   Path: `/mnt/data1/media`  
   User: `multimedia`
   Group: `multimedia`
6. In the shell create the main data root folders:  
   `cd /mnt/data1/media`  
   `mkdir -p films tv incoming/complete/films incoming/complete/tv`  
   `chown -R multimedia:multimedia .`

## Plex Setup
1. Add the "Plex" plugin to TrueNAS (I like to use DHCP, but you can NAT too)
1. Stop the Plex Jail
1. Add mounts to the Plex Jail:  
    * Mount 1 Source: `/mnt/data1/media/films`  
      Mount 1 Destination: `/mnt/data1/iocage/jails/plex/root/media/films`  
      Mount 1 Read Only: `no`  
    * Mount 2 Source: `/mnt/data1/media/tv`  
      Mount 2 Destination: `/mnt/data1/iocage/jails/plex/root/media/tv`  
      Mount 2 Read Only: `no`
1. Start the Plex Jail
1. In the Jail Shell create a user and group to match the `multimedia` ones:  
    `pw groupadd multimedia -g 1000`  
    `pw useradd multimedia -u 1000 -g multimedia -s /usr/sbin/nologin -d /var/empty`
1. Add the Plex user to the multimedia group:  
    `pw usermod plex -G multimedia`
1. Restart the Plex Jail to reload the user config

## SABnzbd Setup
1. Add the "SABnzbd" plugin to TrueNAS
1. Stop the SABnzbd Jail
1. Add mounts to the SABnzbd Jail:  
    * Mount 1 Source: `/mnt/data1/media/films`  
      Mount 1 Destination: `/mnt/data1/iocage/jails/sabnzbd/root/media/films`  
      Mount 1 Read Only: `no`  
    * Mount 2 Source: `/mnt/data1/media/tv`  
      Mount 2 Destination: `/mnt/data1/iocage/jails/sabnzbd/root/media/tv`  
      Mount 2 Read Only: `no`
    * Mount 3 Source: `/mnt/data1/media/incoming`  
      Mount 3 Destination: `/mnt/data1/iocage/jails/sabnzbd/root/media/incoming`  
      Mount 3 Read Only: `no`
1. Start the SABnzbd Jail
1. In the Jail Shell create a user and group to match the `multimedia` ones:  
    `pw groupadd multimedia -g 1000`  
    `pw useradd multimedia -u 1000 -g multimedia -s /usr/sbin/nologin -d /var/empty`
1. Add the SABnzbd user to the multimedia group:  
    `pw usermod _sabnzbd -G multimedia`
1. Restart the SABnzbd Jail to reload the user config:  
   `service sabnzbd restart`

## Radarr Setup
1. Add the "Radarr" plugin to TrueNAS
1. Stop the Radarr Jail
1. Add mounts to the Radarr Jail:  
    * Mount 1 Source: `/mnt/data1/media/films`  
      Mount 1 Destination: `/mnt/data1/iocage/jails/radarr/root/media/films`  
      Mount 1 Read Only: `no`  
    * Mount 2 Source: `/mnt/data1/media/incoming`  
      Mount 2 Destination: `/mnt/data1/iocage/jails/radarr/root/media/incoming`  
      Mount 2 Read Only: `no`
1. Start the Radarr Jail
1. In the Jail Shell create a user and group to match the `multimedia` ones:  
    `pw groupadd multimedia -g 1000`  
    `pw useradd multimedia -u 1000 -g multimedia -s /usr/sbin/nologin -d /var/empty`
1. Add the Radarr user to the multimedia group:  
    `pw usermod radarr -G multimedia`
2. Restart the Radarr Jail to reload the user config:  
   `service radarr restart`


## Sonarr Setup
1. Add the "Sonarr" plugin to TrueNAS
1. Stop the Sonarr Jail
1. Add mounts to the Sonarr Jail:  
    * Mount 1 Source: `/mnt/data1/media/tv`  
      Mount 1 Destination: `/mnt/data1/iocage/jails/sonarr/root/media/tv`  
      Mount 1 Read Only: `no`  
    * Mount 2 Source: `/mnt/data1/media/incoming`  
      Mount 2 Destination: `/mnt/data1/iocage/jails/sonarr/root/media/incoming`  
      Mount 2 Read Only: `no`
1. Start the Sonarr Jail
1. In the Jail Shell create a user and group to match the `multimedia` ones:  
    `pw groupadd multimedia -g 1000`  
    `pw useradd multimedia -u 1000 -g multimedia -s /usr/sbin/nologin -d /var/empty`
1. Add the Sonarr user to the multimedia group:  
    `pw usermod sonarr -G multimedia`
2. Restart the Sonarr Jail to reload the user config:  
   `service sonarr restart`

