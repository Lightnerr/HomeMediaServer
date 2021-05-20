# HomeMediaServer
A manual for the perfect mediaserver

## My almost perfect media server setup consists of:
### Media Servers
Plex


### Download Automation
Sonarr
Radarr
Jackett
Transmission

### Tools
Heimdall
Portainer
Docker
Gluetun

## 1 Install Ubuntu Server
Flash ubuntu server iso to a bootable USB device
This can be done with belana etcher

links:
Etcher: https://www.balena.io/etcher/
Ubuntuserver: https://ubuntu.com/download/server

## 2 Install SSH
In Ubuntu, install SSH server
run sudo apt-get install openssh-server

Check what your IP address is by running:
run ifconfig
The guest's IP address is the "inet addr" of "eth0". This is the IP that will be used to establish the SSH connection.

## 3. Install Docker
https://docs.docker.com/engine/install/ubuntu/
https://docs.docker.com/compose/install/

## 4. Install Portainer Server Deployment
cd ~/
docker volume create portainer_data
docker run -d -p 8000:8000 -p 9000:9000 --name=portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer

go to http://<your internal ip>:9000

## 5. Mount external drive
fdisk -l
mount -t ntfs /dev/sda1  /mnt/

## 6. Install vpn container
https://hub.docker.com/r/qmcgaw/gluetun

Change the docker file below to you VPN provider, region, username and password

Check https://github.com/qdm12/gluetun/wiki/Environment-variables for your VPNSP and REGION variables

```dockerfile
docker run -d --name gluetun --cap-add=NET_ADMIN \
-e VPNSP="NordVPN" -e REGION="Netherlands" \
-e OPENVPN_USER=username -e OPENVPN_PASSWORD=password \
-v /mnt/appdata/gluetun:/gluetun \
qmcgaw/gluetun
```

## 7. Install mediaserver stack
1. Go to portainer and click stacks
2. Click add stack
3. Name it download-automations
4. Paste this code in the web editor

```dockerfile
version: '2.1'
services:
    jackett:
        image: ghcr.io/linuxserver/jackett
        container_name: jackett
        environment:
          - PUID=998
          - PGID=100
          - TZ=Europe/Amsterdam
          - AUTO_UPDATE=true
        volumes:
          - /appdata/jackett:/config
          - /downloads/watch:/downloads
        network_mode: container:gluetun
        restart: unless-stopped
    radarr:
        image: ghcr.io/linuxserver/radarr
        container_name: radarr
        environment:
          - PUID=998
          - PGID=100
          - TZ=Europe/Amsterdam
          - UMASK_SET=022
        volumes:
          - /appdata/radarr:/config
          - /media/movies:/movies
          - /downloads/completed:/downloads
        network_mode: container:gluetun
        restart: unless-stopped
    sonarr:
        image: ghcr.io/linuxserver/sonarr
        container_name: sonarr
        environment:
          - PUID=998
          - PGID=100
          - TZ=Europe/Amsterdam
          - UMASK_SET=022
        volumes:
          - /appdata/sonarr:/config
          - /media/tv:/tv
          - /downloads/completed:/downloads
        network_mode: container:gluetun
        restart: unless-stopped
	transmission:
        image: ghcr.io/linuxserver/transmission
        container_name: transmission
        environment:
          - PUID=998
          - PGID=100
          - TZ=Europe/Amsterdam
        volumes:
          - /appdata/transmission:/config
          - /downloads/completed:/downloads
          - /downloads/watch:/watch
        network_mode: container:gluetun
        restart: unless-stopped
```

## 7. Install plex stack
1. Go to portainer and click stacks
2. Click add stack
3. Name it media
4. Paste this code in the web editor

```dockerfile
version: "2.1"
services:
  plex:
    image: ghcr.io/linuxserver/plex
    container_name: plex
    network_mode: host
    environment:
      - PUID=998
      - PGID=100
      - VERSION=docker
      - UMASK_SET=022
    volumes:
      - /appdata/plex:/config
      - /media/tv/:/tv
      - /media/movies/:/movies
    restart: unless-stopped
```