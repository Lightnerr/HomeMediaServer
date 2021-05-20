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

```dockerfile
docker run -d --name gluetun --cap-add=NET_ADMIN \
-e VPNSP="NordVPN" -e REGION="Netherlands" \
-e OPENVPN_USER=js89ds7 -e OPENVPN_PASSWORD=8fd9s239G \
-v /mnt/appdata/gluetun:/gluetun \
qmcgaw/gluetun
```