## Using ```docker-app``` tool to share your Application on DockerHub

The docker-app is a new tool which allows you to share your complete application on Dockerhub(rather than just a Docker Image). It is still an experimental utility to help make Compose files more reusable and sharable.


### Visualize app configuration

```yaml
# docker-app render elk

version: "3.3"
services:
  elasticsearch:
    command:
    - elasticsearch
    - -Enetwork.host=0.0.0.0
    - -Ediscovery.zen.ping.unicast.hosts=elasticsearch
    deploy:
      mode: replicated
      replicas: 1
      endpoint_mode: dnsrr
    environment:
      ES_JAVA_OPTS: -Xms2g -Xmx2g
    image: elasticsearch:5
    networks:
      elk: null
    ulimits:
      memlock: -1
      nofile:
        soft: 65536
        hard: 65536
      nproc: 65538
    volumes:
    - type: volume
      target: /usr/share/elasticsearch/data
  kibana:
    deploy:
      mode: replicated
      replicas: 1
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
  elk:
    driver: overlay
    driver_opts:
      encrypted: "true"
```

```
$  docker-app inspect elk
openusmelk 0.1.0

Maintained by: Ajeet_Raina <ajeetraina@gmail.com>

ELK for OpenUSM

Services (3)  Replicas Ports             Image
------------  -------- -----             -----
elasticsearch 1                          elasticsearch:5
kibana        1        5601              kibana:latest
logstash      2        10514,10514,12201 logstash:alpine

Network (1)
-----------
elk
```


