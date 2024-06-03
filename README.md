# Streaming 📺
Create your own media/streaming service at home using Sandarr, Radarr, Qbittorrent and Jellyfin. 

## Recommended
1. Decent Computer/Server (personally running on a old dell i3 2nd gen with 4gb Ram)
2. 1TB SSD or Fast HDD

## Quickstart
### installation
You need to have an idea where you want your media server/streaming to run. you can run it on your personal computer (which i don't recommended unless you have a extra HDD attached). Currently i use my PC, with a seperate drive with ubuntu server installed. You don't have to install a serperate os or anything as the applications themselves are dockerized for us. So that means you can also run them on windows(not that they don't have a windows versions though), running these applications with docker make setup much easier and less risky in case you break something (configs etc...).  

#### Docker 
Installing [Docker Engine](https://docs.docker.com/engine/install/) is a must. Docker allows us to run bare bone VMs that containerizes an application on our system, docker just works everywhere which is why it's usefull.

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
Qbittorrent will give us a temporary login details, just copy and save them real quick.
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

### Setup
#### Jackett
Sonarr, Radarr use indexers (oraganized and searchable databases of torrent links) to find appropiate torrent files for tv shows and movies. So will need to sort that out hence ***Jackett***. Jackett is api that sits between the indexers and Sonarr & Radarr making it easy to manage, add or remove indexers whether private or semi private or public.

go to jacket at  `<host_computer_ipaddress>:9117`
1. Copy the api key at the top right and save it for later.
2. Click add indexes.
3. Adding as many indexes as you can make sure there either tv, movies, and anime indexes. some indexes are private (invite only) but some are semi private which means you can sign up to use them. Try to add some semi-privates one too.

#### qbittorent
qbittorrent will be our download client to download videos using the torrent links.

go to `<host_computer_ipaddress>:8080` 
1. login using the login details you saved earlier.
2. click settinggs ->  webui, change you password and username, optionally you can disable auth for local addresses.

#### Sonarr
Sonarr monitors and download our tv shows for us, using indexes and download client.

go to `<host_computer_ipaddress>:8989`
1. add your login details.
2. go to settings -> media management, click add root folder, scroll down until you see `/tv/`  then save. root folder is where sonarr will import and manage downloaded shows. if you gett an error `Folder is not writable by user abc` its an permission issue then run  `sudo chmod -R 777 /your/path/streaming/tv` to give acess to all. modify the permisson however you want.
3. go to settings -> Indexers, click add, under torrents click `Torznab`.
4. go back to Jackett, click copy Torznab feed of the indexer you wish to add.
5. go back to Sonarr, fill in the indexers properties, you can use the same name as the indexer, in `url field` paste the `Torznab Feed` , and in the api key use the jacket api key you saved earlier. click on category to refresh, then click TV and or select the specifc catergories you want.
6. click test if successfull then click save.
7. Add all the indexers from Jackett, repeating steps 1-6.

1. go to settings -> Download Clients, click add -> qbittorrent.
2. fill in the client properties,, placing the host `ip address`, `port`, `username`  and `password`/
3. click test if successful then click save.
4. since were in docker we need a remote mapping, at the bottom in Download Clients add a remote mapping, for the `host` it is the qbittorrent host ip address `<host_computer_ipaddress>`, `remote path` is the path on your computer to the download client folder `/your/path/streaming/qbittorrent/downloads` and lastly the local path is the path in the sonarr docker thats mapp to it which is `/downloads/`
5. click save

#### Testing Sonarr
Your finished just search for a show eg. `Symbionic Titan` to add and test it out. it takes time for the shows to be downloaded.
![Screenshot from 2024-06-02 17-12-52](https://github.com/geovanigaldemsugar/streaming/assets/67174852/64257fac-01b3-42ee-b6ee-9c1b1c257210)

#### Radarr
Rodarr monitors and download our movies for us, using indexes and download client.

go to `<host_computer_ipaddress>:7878`
1. add your login details.
2. go to settings -> media management, click add root folder, scroll down until you see `/movies/`  then save. root folder is where sonarr will import and manage downloaded shows. if you gett an error `Folder is not writable by user abc` its an permission issue then run  `sudo chmod -R 777 /your/path/streaming/movies` to give acess to all. modify the permisson however you want.
3. go to settings -> Indexers, click add, under torrents click `Torznab`.
4. go back to Jackett, click copy Torznab feed of the indexer you wish to add.
5. go back to Sonarr, fill in the indexers properties, you can use the same name as the indexer, in `url field` paste the `Torznab Feed` , and in the api key use the jacket api key you saved earlier. click on category to refresh, then click TV and or select the specifc catergories you want.
6. click test if successfull then click save.
7. Add all the indexers from Jackett, repeating steps 1-6.

1. go to settings -> Download Clients, click add -> qbittorrent.
2. fill in the client properties,, placing the host `ip address`, `port`, `username`  and `password`/
3. click test if successful then click save.
4. since were in docker we need a remote mapping, at the bottom in Download Clients add a remote mapping, for the `host` it is the qbittorrent host ip address `<host_computer_ipaddress>`, `remote path` is the path on your computer to the download client folder `/your/path/streaming/qbittorrent/downloads` and lastly the local path is the path in the sonarr docker thats mapp to it which is `/downloads/`
5. click save

#### Testing Radarr
Your finished just search for a movie `The Dark Knight` to add and test it out. Sometimes it nessesary for you pick the torrent yourself in interactive search as some files are huge ( might contian different languages, uncompressed etc..).

![Screenshot from 2024-06-02 17-19-14](https://github.com/geovanigaldemsugar/streaming/assets/67174852/71848e11-3d96-458b-b28d-f1d4f6c58c54)

#### Jellyfin
Jellyfin create a nice platform for us to watch movies,tv shows and anime. across multiply devices with it being avaible on web andriod and ios.
go to `<host_computer_ipaddress>:8096`

1. complete the wizard, adding your libaries, `/data/tv` and `/data/movies`.


#### Testing Jellyfin
Your finished click on a show and start watching!

![Screenshot from 2024-06-02 17-29-55](https://github.com/geovanigaldemsugar/streaming/assets/67174852/501a4b75-bb59-44a7-997a-284867818bc3)


> [!WARNING]
> A Fast and Ample storage device is nessarcessary, i've had problems with Jellyfin stuttering, which is due to my slow HDD being a HUGE bottleneck.






