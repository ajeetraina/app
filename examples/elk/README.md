## Using ```docker-app``` tool to share your Application on DockerHub

The docker-app is a new tool which allows you to share your complete application on Dockerhub(rather than just a Docker Image). It is still an experimental utility to help make Compose files more reusable and sharable.


### Visualize app configuration

```yaml
# docker-app render elk

version: "3.3"
services:
  elasticsearch:
    environment:
      ES_JAVA_OPTS: -Xmx256m -Xms256m
    image: docker.elastic.co/elasticsearch/elasticsearch-oss:6.4.1
    networks:
      elk: null
    ports:
    - mode: ingress
      target: 9200
      published: 9200
      protocol: tcp
    - mode: ingress
      target: 9300
      published: 9300
      protocol: tcp
    volumes:
    - type: bind
      source: elasticsearch/config/elasticsearch.yml
      target: /usr/share/elasticsearch/config/elasticsearch.yml
      read_only: true
  kibana:
    depends_on:
    - elasticsearch
    image: docker.elastic.co/kibana/kibana-oss:6.4.1
    networks:
      elk: null
    ports:
    - mode: ingress
      target: 5601
      published: 5601
      protocol: tcp
    volumes:
    - type: bind
      source: kibana/config
      target: /usr/share/kibana/config
      read_only: true
  logstash:
    depends_on:
    - elasticsearch
    environment:
      LS_JAVA_OPTS: -Xmx256m -Xms256m
    image: docker.elastic.co/logstash/logstash-oss:6.4.1
    networks:
      elk: null
    ports:
    - mode: ingress
      target: 5000
      published: 5000
      protocol: tcp
    volumes:
    - type: bind
      source: logstash/config/logstash.yml
      target: /usr/share/logstash/config/logstash.yml
      read_only: true
    - type: bind
      source: logstash/pipeline
      target: /usr/share/logstash/pipeline
      read_only: true
networks:
  elk:
    driver: bridge
```

```
root@ubuntu:~/app/examples/elk# docker-app inspect elk
openusmelk 0.1.0

Maintained by: Ajeet_Raina <ajeetraina@gmail.com>

ELK for OpenUSM

Services (3)  Replicas Ports     Image
------------  -------- -----     -----
logstash      1        5000      docker.elastic.co/logstash/logstash-oss:6.4.1
kibana        1        5601      docker.elastic.co/kibana/kibana-oss:6.4.1
elasticsearch 1        9200,9300 docker.elastic.co/elasticsearch/elasticsearch-oss:6.4.1

Network (1)
-----------
elk
```
