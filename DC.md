```yaml
version: '3'

services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.7.0
    environment:
      - node.name=elasticsearch1
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=elasticsearch2,elasticsearch3
      - cluster.initial_master_nodes=elasticsearch1,elasticsearch2,elasticsearch3
      - bootstrap.memory_lock=true
      - ELASTIC_PASSWORD=Password1
      - xpack.security.enabled=true
      - xpack.security.http.ssl.enabled=true
      - xpack.security.http.ssl.key=certs/localhost.key
      - xpack.security.http.ssl.certificate=certs/localhost.crt
      - xpack.security.transport.ssl.enabled=true
      - xpack.security.transport.ssl.verification_mode=certificate
      - xpack.security.transport.ssl.key=certs/localhost.key
      - xpack.security.transport.ssl.certificate=certs/localhost.crt
      - xpack.security.transport.ssl.certificate_authorities=certs/ca.crt
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - esdata1:/usr/share/elasticsearch/data
      - ./certs:/usr/share/elasticsearch/config/certs
    networks:
      - esnet

  elasticsearch2:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.7.0
    environment:
      - node.name=elasticsearch2
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=elasticsearch1,elasticsearch3
      - cluster.initial_master_nodes=elasticsearch1,elasticsearch2,elasticsearch3
      - bootstrap.memory_lock=true
      - ELASTIC_PASSWORD=Password1
      - xpack.security.enabled=true
      - xpack.security.http.ssl.enabled=true
      - xpack.security.http.ssl.key=certs/localhost.key
      - xpack.security.http.ssl.certificate=certs/localhost.crt
      - xpack.security.transport.ssl.enabled=true
      - xpack.security.transport.ssl.verification_mode=certificate
      - xpack.security.transport.ssl.key=certs/localhost.key
      - xpack.security.transport.ssl.certificate=certs/localhost.crt
      - xpack.security.transport.ssl.certificate_authorities=certs/ca.crt
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - esdata2:/usr/share/elasticsearch/data
      - ./certs:/usr/share/elasticsearch/config/certs
    networks:
      - esnet

  elasticsearch3:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.7.0
    environment:
      - node.name=elasticsearch3
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=elasticsearch1,elasticsearch2
      - cluster.initial_master_nodes=elasticsearch1,elasticsearch2,elasticsearch3
      - bootstrap.memory_lock=true
      - ELASTIC_PASSWORD=Password1
      - xpack.security.enabled=true
      - xpack.security.http.ssl.enabled=true
      - xpack.security.http.ssl.key=certs/localhost.key
      - xpack.security.http.ssl.certificate=certs/localhost.crt
      - xpack.security.transport.ssl.enabled=true
      - xpack.security.transport.ssl.verification_mode=certificate
      - xpack.security.transport.ssl.key=certs/localhost.key
      - xpack.security.transport.ssl.certificate=certs/localhost.crt
      - xpack.security.transport.ssl.certificate_authorities=certs/ca.crt
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - esdata3:/usr/share/elasticsearch/data
      - ./certs:/usr/share/elasticsearch/config/certs
    networks:
      - esnet

  kibana:
    image: docker.elastic.co/kibana/kibana:8.7.0
    environment:
      - ELASTICSEARCH_URL=https://elasticsearch1:9200
      - ELASTICSEARCH_USERNAME=elastic
      - ELASTICSEARCH_PASSWORD=Password1
      - ELASTICSEARCH_HOSTS=https://elasticsearch1:9200 https://elasticsearch2:9200 https://elasticsearch3:9200
      - SERVER_SSL_ENABLED=true
      - SERVER_SSL_KEY=certs/localhost.key
      - SERVER_SSL_CERTIFICATE=certs/localhost.crt
      - ELASTICSEARCH_SSL_CERTIFICATEAUTHORITIES=certs/ca.crt
    volumes:
      - ./certs:/usr/share/kibana/config/certs
    networks:
      - esnet
    ports:
      - 5601:5601

volumes:
  esdata1:
    driver: local
  esdata2:
    driver: local
  esdata3:
    driver: local

networks:
  esnet:

```
In a production environment, different nodes of an Elasticsearch cluster are typically located on different hosts. These hosts could be either physical machines or virtual machines depending on your infrastructure. 

The reason for having separate nodes on separate hosts is for distribution and redundancy. Having multiple nodes allows Elasticsearch to distribute the data and query load across different nodes, increasing the capacity and performance of the cluster. 

Furthermore, in case one of the hosts fails, only the Elasticsearch node on that host would be affected, and the rest of the Elasticsearch cluster would continue to operate. This is because Elasticsearch automatically replicates your data across different nodes in the cluster, depending on your configuration. 

It's also possible to run multiple Elasticsearch nodes on a single host, but this is typically only done in testing or development environments. In production, running multiple nodes on a single host would not provide the benefits of distribution and redundancy.

Each Elasticsearch node within the cluster has its own configuration, and it can play different roles such as master, data, ingest, or coordinating node. Depending on the nature of the workload and the size of the data, you can assign specific roles to specific nodes, providing you with a lot of flexibility in managing and optimizing your Elasticsearch cluster.
