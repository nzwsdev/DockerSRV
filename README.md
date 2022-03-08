# DockerSRV

How build up nzws.dev (sample)


## Request map

Example hello.nzws.dev

```

---------------------------------------------
| Client: Browser » https://hello.nzws.dev/ |
---------------------------------------------
                        |
                        |
                  --------------
                  | Cloudflare |
                  --------------
                        |
                        |
                  ----------------
                  | Home, router |
                  ----------------
                        |
                        |
                  --------------------------
                  | Docker: Traefik        |
                  | Routing, and balancing |
                  --------------------------
                        |
                        |
-----------------     -----------------     -----------------
| Docker: main  |     | Docker: hello |     | Docker: static |
-----------------     -----------------     -----------------
```


## Host

```
-------------------------------------
| Docker layer, dokcer-compose .... |
-------------------------------------
    |
    |-- mounted: /srv/ (/nzws/srv/docker, custom uid/gid)
    |
------------------------------------------------------------------------------
| Guest host: Apline, service: docker, cloudfared, vmhgfs-mounter, sshd, ... |
------------------------------------------------------------------------------
    |
    |
-------------------------------------------------------
| VMWare, CPU: 2, Mem: 2G, dedicated LAN, external IP |
-------------------------------------------------------
    |
    |-- /nzws/srv/docker (rw)
    |
----------------------
| Hot machine, macOS |
----------------------
```


## File, project, service structure

Name convention
```
nzws.dev
 |- dockersrv        » (ProjectName)
 |--- feapp          » (NetworkName)
 |---- httpd_nzwsdev » (DockerServiceName)
```

Docker service path
```
gitea.nzws.dev/DockerSRV/feapp-httpd_nzwsdev
nzws/srv/docker/feapp-httpd_nzwsdev
```

GIT path
```
nzws.dev/dockersrv.feapp-httpd_nzwsdev
```

DockerSRV structure

* Makefile:
  * Create .env for docker-compose (with prod or dev)
  * Create new docker service (net-docker-service) from ./tmpl_networkapp-service_name template dir
* net-docker-service/src
  * Stuff for the Docker
* net-docker-service/src
  * Stuff for the service (docker mount)

```
DockerSRV
 |- common
 |-- env
 |--- dev
 |---- .common-base.dev.env ------------------|
 |--- common-env_1                            |
 |---- .common-env_1.abc.env                  |
 |---- .common-env_1.xyz.env                  |
 |--- common-env_2                            |
 |--- common-env_n                            |
 |-- tools                                    |
 |--- Makefile ---------->|>--------------|   |
 |                        |               |   |
 |~ Makefile <---symlink--|               |   |
 |- README.md                             |   |
 |                                        |   |
 |+ net-docker-service-1 (git repo)       |   |
 |   |- src                               |   |
 |   |-- .base.env                        |   |
 |   |-- .user.dev.env                    |   |
 |   |~~ Makefile <---symlink-------------|   |
 |   |~~ .common-base.dev.env <---symlink-----|
 |   |-- docker-compose.yml
 |   |
 |   |- srv
 |   |-- config
 |   |-- data
 |   |-- log
 |+ net-docker-service-2
 |+ net-docker-service-n
```

Legend:
* |- » normal file or dir
* |~ » symlink
* |+ » git repository


### Makefile

Create docker service from template
```sh
make snet=feapp sname=project_name create-service-dir (optional: cmode=sys)
```

Create .env
```sh
make create-dotenv-prod
make create-dotenv-dev
```


## Docker networks, ports and UID

| UID | Scale | Network | Service Name         |  WAN Port | WAN Proxy | D External Port | D Internal Port  | Service Description                                             |
| --: | ----: | :------ | :------------------- | --------: | --------- | :-------------- | :--------------- | :-------------------------------------------------------------- |
| 444 |     1 | gateway | traefik              |   80, 443 | -         | 80, 443, 8080   | 80, 443, 8080    | Reverse proxy, loadbalancer, make HTTPS/TSL Lets Encrypt, ...   |
| 444 |     2 | feapp   | httpd_nzwsdev_main   |       443 | traefik   | 44001           | 44001            | HTT server for nzws.dev                                         |
| 444 |     2 | feapp   | httpd_nzwsdev_hello  |       443 | traefik   | 44002           | 44002            | HTT server for hello.nzws.dev                                   |
| 444 |     2 | feapp   | httpd_nzwsdev_static |       443 | traefik   | 44003           | 44003            | HTT server for static.nzws.dev                                  |
| 444 |     2 | feapp   | httpd_nzwsdev_cv     |       443 | traefik   | 44004           | 44004            | HTT server for cv.nzws.dev                                      |
| 444 |     1 | bedb    | mysql                |         - | -         | 44041           | 44041            | MySQL BE server                                                 |
| 444 |     1 | bedb    | influxdb             |         - | -         | 44046           | 44046            | InfluxDB BE server                                              |
| 444 |     1 | beapp   | gitea                |443, xxxxx | traefik   | 44053, 44052 ssh| 44053 http, 44052| Git with a cup of tea. A painless self-hosted Git service.      |
| 444 |     1 | beapp   | homeassistant        |       443 | traefik   | 44055           | 44055            | Home automation, the central control system for smart home, ... |
| 444 |     1 | beapp   | adminer              |       443 | traefik   | 44051           | 44051            | Database management in a single PHP file                        |
| 444 |     1 | beapp   | svnserve             |     xxxxx | traefik   | 44054           | 44054            | SVN (subversion) server                                         |
| 444 |     1 | beapp   | telegraf             |         - | -         | -               | -                | Telegraf is an agent for collecting metrics and writing them to InfluxDB or other outputs. |
