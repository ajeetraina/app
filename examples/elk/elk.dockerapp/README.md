
## Deploy 5 Node Docker Swarm Cluster

```
$ docker node ls
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUSENGINE VERSION
iy9mbeduxd4mmjxoikbn5ulds *   manager1            Ready               Active              Reachable18.03.1-ce
mx916kgqg6gfgqdr2gn1eksxy     manager2            Ready               Active              Leader18.03.1-ce
xaeq943o84g9spy6mebj64tw3     manager3            Ready               Active              Reachable18.03.1-ce
8umdv6m82nrpevuris1e45wnq     worker1             Ready               Active18.03.1-ce
o3yobqgg7wjvjw2ec5ythszgw     worker2             Ready               Active18.03.1-ce
```

## Cloning the Repository

```
$ git clone https://github.com/ajeetraina/app
Cloning into 'app'...remote: Enumerating objects: 134, done.
remote: Counting objects: 100% (134/134), done.remote: Compressing objects: 100% (134/134), done.
remote: Total 14511 (delta 95), reused 0 (delta 0), pack-reused 14377Receiving objects: 100% (14511/14511), 17.37 MiB | 13.35 MiB/s, done.
Resolving deltas: 100% (5391/5391), done.
```

## Install Docker-app

```
$ cd app/examples/elk/
[manager1] (local) root@192.168.0.30 ~/app/examples/elk$ ls
README.md          devel              elk.dockerapp      install-dockerapp  prod
[manager1] (local) root@192.168.0.30 ~/app/examples/elk
$ chmod +x install-dockerapp[manager1] (local) root@192.168.0.30 ~/app/examples/elk
$ sh install-dockerappConnecting to github.com (192.30.253.112:443)
Connecting to github-production-release-asset-2e65be.s3.amazonaws.com (52.216.131.187:443)docker-app-linux.tar 100% |*************************************************************|  8895k  0:00:00 ETA
[manager1] (local) root@192.168.0.30 ~/app/examples/elk
```

## Verifying Docker-app Version

```
$ docker-app version
Version:      v0.4.0
Git commit:   525d93bc
Built:        Tue Aug 21 13:02:46 2018
OS/Arch:      linux/amd64
Experimental: off
Renderers:    none
```

Say, you have already created a docker compose for ELK stack application. Put it under the same directory. Now with docker-app installed let's create an Application Package based on this Compose file:

```
$ docker-app init elk
```

Once you run thie above command, it create a new directory elk.dockerapp that contains three different YAML files:

```
docker-compose.yml  elk.dockerapp
[manager1] (local) root@192.168.0.30 ~/myelk
$ tree elk.dockerapp/
elk.dockerapp/
├── docker-compose.yml
├── metadata.yml
└── settings.yml

0 directories, 3 files

```

Edit each of these files as shown under this link.

## Rendering Docker Compose file

```
$ docker-app render elk
version: "3.4"services:
  elasticsearch:    command:
    - elasticsearch    - -Enetwork.host=0.0.0.0
    - -Ediscovery.zen.ping.unicast.hosts=elasticsearch
    deploy:
      mode: replicated
      replicas: 2
    environment:
      ES_JAVA_OPTS: -Xms2g -Xmx2g
    image: elasticsearch:5
    networks:
      elk: null
    volumes:
    - type: volume
      target: /usr/share/elasticsearch/data
  kibana:
    deploy:
      mode: replicated
      replicas: 2
    environment:
      ELASTICSEARCH_URL: http://elasticsearch:9200
    healthcheck:
      test:
      - CMD-SHELL
      - wget -qO- http://localhost:5601 > /dev/null
      interval: 30s
      retries: 3
    image: kibana:latest
    networks:
      elk: null
    ports:
    - mode: ingress
      target: 5601
      published: 5601
      protocol: tcp
  logstash:
    command:
    - sh
    - -c
    - logstash -e 'input { syslog  { type => syslog port => 10514   } gelf { } } output
      { stdout { codec => rubydebug } elasticsearch { hosts => [ "elasticsearch" ]
      } }'
    deploy:
      mode: replicated
      replicas: 2
    hostname: logstash
    image: logstash:alpine
    networks:
      elk: null
    ports:
    - mode: ingress
      target: 10514
      published: 10514
      protocol: tcp
    - mode: ingress
      target: 10514
      published: 10514
      protocol: udp
    - mode: ingress
      target: 12201
      published: 12201
      protocol: udp
networks:
  elk: {

```

## Setting the kernel parameter for ELK stack

```
sysctl -w vm.max_map_count=262144
```

## Deploying the Application Stack

```

[manager1] (local) root@192.168.0.30 ~/app/examples/elk
$ docker-app deploy elk --settings-files elk.dockerapp/settings.yml
Creating network elk_elk
Creating service elk_kibana
Creating service elk_logstash
Creating service elk_elasticsearch
[manager1] (local) root@192.168.0.30 ~/app/examples/elk
$

```

## Inspecting ELK Stack 

```
[manager1] (local) root@192.168.0.30 ~/app/examples/elk
$ docker-app inspect elk
myelk 0.1.0
Maintained by: Ajeet_Raina <ajeetraina@gmail.com>

ELK using Dockerapp

Setting                       Default
-------                       -------
elasticsearch.deploy.mode     replicated
elasticsearch.deploy.replicas 2
elasticsearch.image.name      elasticsearch:5
kibana.deploy.mode            replicated
kibana.deploy.replicas        2
kibana.image.name             kibana:latest
kibana.port                   5601
logstash.deploy.mode          replicated
```

## Verifying Stack services are up & running

```
[manager1] (local) root@192.168.0.30 ~/app/examples/elk/docker101/play-with-docker/visualizer
$ docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
uk2whax6f3jq        elk_elasticsearch   replicated          2/2                 elasticsearch:5
nm4p3yswvh5y        elk_kibana          replicated          2/2                 kibana:latest       *:5601->56
01/tcp
g5ubng6rhcyp        elk_logstash        replicated          2/2                 logstash:alpine     *:10514->1
0514/tcp, *:10514->10514/udp, *:12201->12201/udp
[manager1] (local) root@192


```

## Pushing the App Package to Dockerhub

```
Password:[manager1] (local) root@192.168.0.30 ~/app/examples/elk
$ docker loginLogin with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to
 https://hub.docker.com to create one.Username: ajeetraina
Password:Login Succeeded
```

## Pushing the App package to DockerHub

```

[manager1] (local) root@192.168.0.30 ~/app/examples/elk$ docker-app push --namespace ajeetraina --tag 1.0.2
The push refers to repository [docker.io/ajeetraina/elk.dockerapp]
15e73d68a400: Pushed
1.0.2: digest: sha256:c5a8e3b7e2c7a5566a3e4247f8171516033e7e9791dfdb6ebe622d3830884d9b size: 524
[manager1] (local) root@192.168.0.30 ~/app/examples/elk
$
```

Important Note: If you are using Docker-app v0.5.0, you won't be able to push it to Dockerhub. For more details, refer this bug.

## Testing the Application Package

Open up a new PWD window. Install docker-app as shown above and try to run the below command:

```
docker-app deploy ajeetraina/elk.dockerapp:1.0.2
```

This should bring up your complete Elastic Stack Platform.


