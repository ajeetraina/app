## Using ```docker-app``` tool to share your Application on DockerHub

The docker-app is a new tool which allows you to share your complete application on Dockerhub(rather than just a Docker Image). It is still an experimental utility to help make Compose files more reusable and sharable.


### Visualize app configuration

```yaml
# docker-app render elk

version: "3.4"services:  elasticsearch:    
command:
    - elasticsearch    - -Enetwork.host=0.0.0.0    - -Ediscovery.zen.ping.unicast.hosts=elasticsearch    
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
```

```
$  docker-app inspect elk
$ docker-app inspect elk
myelk 0.1.0

Maintained by: Ajeet_Raina <ajeetraina@gmail.com>

ELK using Dockerapp

Services (3)  Replicas Ports             Image
------------  -------- -----             -----
kibana        2        5601              kibana:latest
logstash      2        10514,10514,12201 logstash:alpine
elasticsearch 2                          elasticsearch:5

Network (1)
-----------
elk

Settings (13)                      Value
-------------                      -----
elasticsearch.deploy.endpoint_mode dnsrr
elasticsearch.deploy.mode          replicated
elasticsearch.deploy.replicas      2
elasticsearch.image.name           elasticsearch:5
kibana.deploy.endpoint_mode        dnsrr
kibana.deploy.mode                 replicated
kibana.deploy.replicas             2
kibana.image.name                  kibana:latest
kibana.port                        5601
logstash.deploy.endpoint_mode      dnsrr
logstash.deploy.mode               replicated
logstash.deploy.replicas           2
logstash.image.name                logstash:alpine
```


# ELK Stack for Docker EE using Docker-app

```
openusm@master01:~/app/examples/elk$ sudo sh install-dockerapp
--2018-10-16 01:20:45--  https://github.com/docker/app/releases/download/v0.4.0/docker-app-linux.tar.gz
Resolving github.com (github.com)... 192.30.253.113, 192.30.253.112
Connecting to github.com (github.com)|192.30.253.113|:443... connected.
HTTP request sent, awaiting response... 302 Found
Location: https://github-production-release-asset-2e65be.s3.amazonaws.com/128402269/b443a200-a553-11e8-804f-253442b219d6?X-Amz
-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20181016%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=201810
16T012046Z&X-Amz-Expires=300&X-Amz-Signature=7376c6278e43fbd483dc994b64a8fc19b842431b4a5c8afb722e44fcb136e453&X-Amz-SignedHead
ers=host&actor_id=0&response-content-disposition=attachment%3B%20filename%3Ddocker-app-linux.tar.gz&response-content-type=appl
ication%2Foctet-stream [following]
--2018-10-16 01:20:46--  https://github-production-release-asset-2e65be.s3.amazonaws.com/128402269/b443a200-a553-11e8-804f-253
442b219d6?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20181016%2Fus-east-1%2Fs3%2Faws4_request&X-
Amz-Date=20181016T012046Z&X-Amz-Expires=300&X-Amz-Signature=7376c6278e43fbd483dc994b64a8fc19b842431b4a5c8afb722e44fcb136e453&X
-Amz-SignedHeaders=host&actor_id=0&response-content-disposition=attachment%3B%20filename%3Ddocker-app-linux.tar.gz&response-co
ntent-type=application%2Foctet-stream
Resolving github-production-release-asset-2e65be.s3.amazonaws.com (github-production-release-asset-2e65be.s3.amazonaws.com)...
 52.216.1.8
Connecting to github-production-release-asset-2e65be.s3.amazonaws.com (github-production-release-asset-2e65be.s3.amazonaws.com
)|52.216.1.8|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 9109406 (8.7M) [application/octet-stream]
Saving to: ‘docker-app-linux.tar.gz’
docker-app-linux.tar.gz                      100%[===========================================================================================>]   8.69M   985KB/s    in 9.6s    
2018-10-16 01:20:57 (928 KB/s) - ‘docker-app-linux.tar.gz’ saved [9109406/9109406]
```

## Verifying Docker-app version
```
openusm@master01:~/app/examples/elk$ docker-app version
Version:      v0.4.0
Git commit:   525d93bc
Built:        Tue Aug 21 13:02:46 2018
OS/Arch:      linux/amd64
Experimental: off
Renderers:    none
```

## 

docker-app helm wordpress will output a Helm package in the ./wordpress.helm folder. --compose-file (or -c), --set (or -e) and --settings-files (or -s) flags apply the same way they do for the render subcommand.

```
openusm@master01:~/app/examples/elk$ docker-app helm --stack-version=v1beta1
```

## Deploy WordPress Application on Kubernetes Cluster

```
openusm@master01:~/app/examples/wordpress$ docker-app deploy -o kubernetes
top-level network "overlay" is ignored
service "mysql": network "overlay" is ignored
service "wordpress": network "overlay" is ignored
service "wordpress": depends_on are ignored
Waiting for the stack to be stable and running...
wordpress: Ready                [pod status: 1/1 ready, 0/1 pending, 0/1 failed]

```

##

