# Seafile Server Docker image
[Seafile](http://seafile.com/) server Docker image based on [Alpine Linux](https://hub.docker.com/_/alpine/). 
- No Proxy: Does not contains a proxy. You can run your own proxy such as nginx, lightpd
  or haaproxy with the sample configs provided.
- No DB: Contains by default SQLite in the container. You can however run your own
  mysql instance such as MariaDB outside of the container, with the config
  provided.
- No Custom Config : Simply builds the latest seafile version and it's dependencies and run it inside a docker container. Nothing more.


## Supported tags and respective `Dockerfile` links

* [`6.2.5`](https://github.com/bernardgut/seafile-docker/blob/6.2.5/docker/Dockerfile), [`latest`](https://github.com/bernardgut/seafile-docker/blob/master/docker/Dockerfile) - Seafile Server v6.2.5 - latest available version

Dockerfiles for older versions of Seafile Server you can find [there](https://github.com/bernardgut/seafile-docker/tags).

## Quickstart

To run container you can use following command:
```bash
docker run \  
  -v /home/docker/seafile:/home/seafile \  
  -p 127.0.0.1:8000:8000 \  
  -p 127.0.0.1:8082:8082 \  
  -ti sunx/seafile`
```
Containers, based on this image will automatically configure Seafile enviroment if there isn't any. If Seafile enviroment is from previous version of Seafile, container will automatically upgrade it to the latest version (by calling Seafile upgrade scripts).
 
But I would advise you to do data backups before upgrading image 
 (to not lose your data in case of bugs in upgrade logic of this image or Seafile upgrde scripts).

## Detailed description of image and containers

### Used ports

This image uses 2 tcp ports:
* 8000 - seafile port
* 8082 - seahub port

If you want to run seafdav (WebDAV for Seafile), then port 8080 will be used also.

### Volume
This image uses one volume with internal path `/home/seafile`

I would recommend you use host directory mapping of named volume to run containers, so you will not lose your valuable data after image update and starting new container

### Web server configuration

This image doesnt contain any web-servers, because you, usually, already have some http server running on your server and don't want to run any extra http-servers (because it will cost you some CPU time and Memory). But if you know some really tiny web-server with proxying support, tell me and I would be glad to integrate it to the image.


For Web-server configuration, as media directory location you should enter
`<volume/path>/seafile-server/seahub

In [httpd-conf](proxy-conf) directory you can find
[nginx](https://www.nginx.com/) [config example](proxy-conf/nginx.conf.example),
[lighttpd](https://www.lighttpd.net/) [config example](proxy-conf/lighttpd.conf.example) and [haaproxy](https://www.haproxy.com/) [config example](proxy-conf/haproxy.cfg.example).

You can find 
[Nginx](https://manual.seafile.com/deploy/deploy_with_nginx.html) and 
[Apache](https://manual.seafile.com/deploy/deploy_with_apache.html) sample 
configurations in official Seafile Server [Manual](https://manual.seafile.com/).

### Turn on SSL encryption

1. configure your proxy to use SSL (see [configuration
   samples](proxy-conf/nginx.conf.example))
2. update config files :
    - `ccnet.conf: SERVICE_URL = https://seafile.example.com`
    - `seahub_settings.py: FILE_SERVER_ROOT ='https://seafile.example.com/seafhttp'`
3. restart `docker restart <container>`

### Supported ENV variables

When you running container, you can pass several enviroment variables (with **--env** option of **docker run** command):
* **`INTERACTIVE`**=\<0|1> - if container should ask you about some configuration values (on first run) and about upgrades. Default: 1
* **`SERVER_NAME`**=\<...> - Name of Seafile server (3 - 15 letters or digits), used only for first run in non-interactive mode. Default: Seafile
* **`SERVER_DOMAIN`**=\<...> - Domain or ip of seafile server, used only for first run in non-interactive mode. Default: seafile.domain.com
* **`SEAHUB`**=\<fastcgi> - If seahub should be started in FastCGI mode (set it "fastcgi" for FastCGI mode or leave empty otherwise). Default: empty (not FastCGI mode).
* **`SEAFILE_FASTCGI_HOST`**=\<ip> - Binding ip for seahub in FastCGI mode. Default: 127.0.0.1.
* **`HANDLE_SIGNALS`**=\<0|1> - If container should properly handle signals like SIGHUP and SIGTERM (SIGTERM is sending on `docker stop` command, for example). If signals handling is turned on, then script will run infinity cycle for waiting signal, what, in theory, could slightly increase CPU consumption by container. Default: 1 (i.e. Turned on).

## Useful commands in container

When you're inside of container, in home directory of seafile user, you can use following useful commands:
* `seafile-fsck` - check your libraries for errors (Originally seaf-fsck.sh is used for it)
* `seafile-gc` - remove ald unused data from storage of your seafile libraries (Originally seaf-gc.sh is used for it)
* `seafile-admin start` - start seafile and seahub daemons (if they were stopped)
* `seafile-admin stop` - stop seafile and seahub daemons
* `seafile-admin reset-admin` - reset seafile admin user and/or password
* `seafile-admin setup` - setup ccnet, seafile and seahub services (if they wasn't configured automatically by some reason)
* `seafile-admin create-admin` - create seafile admin user (if it wasn't created automatically by some reason)

## Tips&amp;Tricks and Known issues

* Make sure, that mounted data volume and files are radable and writable by container's seafile user(2016:2016).

* If you want to run seafdav, which is disabled by default, you can read it's [manual](https://manual.seafile.com/extension/webdav.html). Do not forget to publish port 8080 after it.

* If you do not want container to automatically upgrade your Seafile enviroment on image (and Seafile-server) update, 
you can add empty file named `.no-update` to directory `/home/seafile` in your container. You can use **`docker exec <container_name> touch /home/seafile/.no-update`** for it.

* Container uses seafile user to run seafile, so if you need to do something with root access in container, you can use **`docker exec -ti --user=0 <container_name> /bin/sh`** for it.

* On first run (end every image upgrade) container will copy seahub directory from `/usr/local/share/seahub` to `/home/seafile/seafile-server/seahub `(i.e. to the volume), so it cost about 40Mb of space. I'm not sure if it could be changed without using webserver inside of container (But 40Mb of space isn't to much in our days, I think).

* At this moment most seafile scripts (which are located in `/usr/local/share/seafile/scripts` directory) aren't working properly, but I do not think that they are to usefull for this image (scripts `seaf-fsck.sh` and `seaf-gc.sh` are working correctly and also avaliable as `/usr/local/bin/seafile-fsck` and `/usr/local/bin/seafile-gc`).

* On your proxy, you might experience read errors such as ` [error] 4766#4766: *2 open()  "<seafile_dir>/seafile-server/seahub/media/avatars/default.png"  failed (2: No such file or directory) `. That is because `seafile-server/seahub/media/avatars` is a symlink to `/home/seafile/seahub-data/avatars`, which exists only in the container. To fix this simply fix the link with a relative one: In `<seafile_dir>/seafile-server-seahub/media` do `ln -s ../../../seahub-data/avatars avatars`.  

## License

This Dockerfile and scripts are released under [MIT License](https://github.com/bernardgut/seafile-docker/blob/master/LICENSE). All credits go to [VGoshev](https://github.com/VGoshev/seafile-docker) for the original idea.

[Seafile](https://github.com/haiwen/seafile/blob/master/LICENSE.txt) and [Alpine Linux](https://www.alpinelinux.org/) have their own licenses.
