## Using ```docker-app``` tool to share your Application on DockerHub

The docker-app is a new tool which allows you to share your complete application on Dockerhub(rather than just a Docker Image). It is still an experimental utility to help make Compose files more reusable and sharable.


### Visualize app configuration

```yaml
# docker-app render wordpress
version: "3.6"
services:
  mysql:
    deploy:
      mode: replicated
      replicas: 1
    environment:
      MYSQL_DATABASE: wordpressdata
      MYSQL_PASSWORD: wordpress
      MYSQL_ROOT_PASSWORD: axx[<^cz3d.fPb
      MYSQL_USER: wordpress
    image: mysql:8
    volumes:
    - type: db_data
      source: db_data
      target: /var/lib/mysql
  wordpress:
    deploy:
      mode: global
      replicas: 0
    environment:
      WORDPRESS_DB_HOST: mysql
      WORDPRESS_DB_NAME: wordpressdata
      WORDPRESS_DB_PASSWORD: wordpress
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DEBUG: "true"
    image: wordpress
networks: {}
volumes:
  db_data:
    name: db_data
secrets: {}
configs: {}
```

**Merge with override Compose file**. This example replaces cleartext DB passwords to use secrets instead.

```yaml
# docker-app render wordpress -c with-secrets.yml
version: "3.6"
services:
  mysql:
    deploy:
      mode: replicated
      replicas: 1
    environment:
      MYSQL_DATABASE: wordpressdata
      MYSQL_PASSWORD: ""
      MYSQL_PASSWORD_FILE: /run/secrets/wordpress_app_userpass
      MYSQL_ROOT_PASSWORD: ""
      MYSQL_ROOT_PASSWORD_FILE: /run/secrets/wordpress_app_rootpass
      MYSQL_USER: wordpress
    image: mysql:8
    secrets:
    - source: mysql_rootpass
    - source: mysql_userpass
    volumes:
    - type: volume
      source: db_data
      target: /var/lib/mysql
  wordpress:
    deploy:
      mode: global
      replicas: 0
    environment:
      WORDPRESS_DB_HOST: mysql
      WORDPRESS_DB_NAME: wordpressdata
      WORDPRESS_DB_PASSWORD: ""
      WORDPRESS_DB_PASSWORD_FILE: /run/secrets/sumple_app_userpass
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DEBUG: "true"
    image: wordpress
    secrets:
    - source: mysql_userpass
networks: {}
volumes:
  db_data:
    name: db_data
secrets:
  mysql_rootpass:
    name: wordpress_app_rootpass
    external: true
  mysql_userpass:
    name: wordpress_app_userpass
    external: true
configs: {}
```

**Override default settings**. This example sets `debug` to false.

```yaml
# docker-app render wordpress --settings-files prod-settings.yml
version: "3.6"
[...]
    environment:
      WORDPRESS_DB_HOST: mysql
      WORDPRESS_DB_NAME: wordpressdata
      WORDPRESS_DB_PASSWORD: wordpress
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DEBUG: "false"
[...]
```

**Override from the command line**. This example sets `debug` to false and the database user to a
different value.
```yaml
# docker-app render wordpress --set debug=true --set mysql.user.name=mollydock
version: "3.6"
services:
  mysql:
[...]
    environment:
      MYSQL_DATABASE: wordpressdata
      MYSQL_PASSWORD: wordpress
      MYSQL_ROOT_PASSWORD: axx[<^cz3d.fPb
      MYSQL_USER: mollydock
[...]
  wordpress:
[...]
    environment:
      WORDPRESS_DB_HOST: mysql
      WORDPRESS_DB_NAME: wordpressdata
      WORDPRESS_DB_PASSWORD: wordpress
      WORDPRESS_DB_USER: mollydock
      WORDPRESS_DEBUG: "false"
[...]
```

### View app metadata

```yaml
# docker-app inspect wordpress
wordpress 0.1.0
Maintained by: sakuya.izayoi <sizayoi@sdmansion.jp>



Setting                  Default
-------                  -------
mysql.user.password      wordpress
mysql.rootpass           wordpress101
mysql.database           wordpressdata
mysql.user.name          wordpress
volumes.db_data.name     db_data
debug                    true
mysql.image.version      8
wordpress.scale.mode     global
wordpress.scale.replicas 0
```

### Generate helm package

`docker-app helm wordpress` will output a Helm package in the `./wordpress.helm` folder. `--compose-file` (or `-c`), `--set` (or `-e`) and `--settings-files` (or `-s`) flags apply the same way they do for the `render` subcommand.

```
$ docker-app helm wordpress --compose-file with-secrets.yml --settings-files prod-settings.yml --set mysql.user.name=mollydock
$ tree wordpress.helm
wordpress.helm/
├── Chart.yaml
└── templates
    └── stack.yaml

1 directory, 2 files
$ cat wordpress.helm/templates/stack.yaml
typemeta:
  kind: stacks.compose.docker.com
  apiversion: v1beta2
objectmeta:
[...]
```

### Generate distributable app package

`docker-app save wordpress` creates a Docker image packaging the relevant configuration files:

```
$ docker-app save wordpress
$ docker images wordpress.dockerapp
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
wordpress.dockerapp   latest              61f8cafb7762        4 minutes ago       1.2kB
```

The package can later be retrieved using `docker-app load`:

```
$ rm -rf wordpress.dockerapp
$ docker-app load wordpress.dockerapp
$ tar -tf wordpress.dockerapp  # TODO: should unpack automatically?
metadata.yml
docker-compose.yml
settings.yml
$ mv wordpress.dockerapp wordpress  # TODO: fix UX
$ docker-app unpack wordpress
$ tree wordpress.dockerapp
./wordpress.dockerapp/
├── metadata.yml
├── docker-compose.yml
└── settings.yml

0 directories, 3 files
```

### Archive app package

`docker-app pack wordpress` creates a tar archive containing the relevant configuration files:

```
$ docker-app pack wordpress --output myapp.tar
$ tar -tf myapp.tar
metadata.yml
docker-compose.yml
settings.yml
```
