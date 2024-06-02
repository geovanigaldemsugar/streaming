# Streaming 📺
#### Create your own media/streaming service at home using Sandarr, Radarr, Qbittorrent and Jellyfin. 

## Requirements
1. Decent Computer/Server

## Quickstart
### installation
you need to have an idea where you want your media server/streaming to run. you can run it on your personal computer (which i dont reccomended unless you have a extra HDD attached). Currently i use my PC, with a seperate drive with ubuntu server installed. You don't have to install a serperate os or anything as the applications themselves are dockerized for us. So that means you can also run them on windows(not that they dont have a windows versions though), running these applications with docker make setup much easier and less risky in case you break something (configs etc...).  

#### Docker 
Installing [Docker Engine](https://docs.docker.com/engine/install/) is a must. Docker allows us to run bare bone VMs that containerizes an application on our system, docker just works everywhere which is its usefull.

#### Docker-Compose file
Create a directory/folder where will create our [docker-compose](https://docs.docker.com/compose/) file.

- ubuntu: `mkdir streaming`
- windows: `dir streaming`

create a file called docker-compose.yml, in the docker-compose file couple a things to note,
1. The shared volume for `/your/path/streaming/qbittorrent`  across the sonarr, raddarr, and qbittorrent must be the same path.
2. The shared volume `/your/path/streaming/tv` - Sonarr and `/your/path/streaming/movies` - Radarr must be the same path as you sepcified for jellyfin.
docker-compose.yml
```yml
services:
  sonarr:
    image: lscr.io/linuxserver/sonarr:latest
    container_name: sonarr
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Etc/UTC
    volumes:
      - /your/path/streaming/sonarr/config:/config
      - /your/path/streaming/tv:/tv 
      - /your/path/streaming/qbittorrent/downloads:/downloads 
    ports:
      - 8989:8989
    restart: unless-stopped
  radarr:
    image: lscr.io/linuxserver/radarr:latest
    container_name: radarr
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Etc/UTC

    volumes:
      - /your/path/streaming/radarr/data:/config
      - /your/path/streaming/movies:/movies 
      - /your/path/streaming/qbittorrent/downloads:/downloads 

    ports:
      - 7878:7878
    restart: unless-stopped
  jackett:
    image: lscr.io/linuxserver/jackett:latest
    container_name: jackett
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Etc/UTC
      - AUTO_UPDATE=true
      - RUN_OPTS= #optional
    volumes:
      - /your/path/streaming/jackett/data:/config
      - /your/path/streaming/qbittorrent/downloads:/downloads
    ports:
      - 9117:9117
    restart: unless-stopped
  qbittorrent:
    image: lscr.io/linuxserver/qbittorrent:latest
    container_name: qbittorrent
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Etc/UTC
      - WEBUI_PORT=8080
      - TORRENTING_PORT=6881
    volumes:
      - /your/path/treaming/qbittorrent/appdata:/config
      - /your/path/streaming/qbittorrent/downloads:/downloads
    ports:
      - 8080:8080
      - 6881:6881
      - 6881:6881/udp
    restart: unless-stopped
  jellyfin:
    image: lscr.io/linuxserver/jellyfin:latest
    container_name: jellyfin
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Etc/UTC
      - JELLYFIN_PublishedServerUrl=192.168.0.5 
    volumes:
      - /your/path/streaming/jellyfin/library:/config
      - /your/path/streaming/tv:/data/tvshows
      - /your/path/streaming/movies:/data/movies
    ports:
      - 8096:8096
      - 8920:8920
      - 7359:7359/udp
      - 1900:1900/udp 
    restart: unless-stopped
```
#### Starting the Applications
Run in the same directoy as the docker compose file - `docker compose up`. docker will now download the images and run them.
Qbittorrent will give us a temporary login details just copy and save them real quick.
```sh
qbittorrent  | The WebUI administrator username is: admin
qbittorrent  | The WebUI administrator password was not set. A temporary password is provided for this session: UKH7QLHF8
```

### where are the Applications ? 
The respective web gui's are hosted
1. Jackett - `<host_computer_ipaddress>:9117`
2. Qbittorrent - `<host_computer_ipaddress>:8080`
3. Sonarr - `<host_computer_ipaddress>:8989`
4. Radarr -  `<host_computer_ipaddress>:7878`
5. Jellyfin - `<host_computer_ipaddress>:8096`

###





