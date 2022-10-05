# Monitoring

## run grafana/prometheus/lag-exporter
 
 `docker-compose up -d`

## go to grafana webUI (admin/admin)
 `<docker-host>:3000`

## prometheus webUI
`<docker-host>:9000`

## swiss army knife (kcat) for devs
`docker run -it --rm --network confluent-dev  --name producer  confluentinc/cp-kafkacat kafkacat -b kafka-0:9092 -L -J`

## kafdrop

`docker run -d --rm -p 9001:9000 -e KAFKA_BROKERCONNECT=kafka-0:9092,kafka-1:9092 -e JVM_OPTS="-Xms32M -Xmx64M" -e SERVER_SERVLET_CONTEXTPATH="/" --network="confluent-dev" obsidiandynamics/kafdrop`

## akhq

`cd monitoring/akhq` & `docker-compose up -d`

## kafka-ui

`docker run -p 8085:8080 -e KAFKA_CLUSTERS_0_NAME=local -e KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS=kafka-0:9092 --network=confluent-dev -d provectuslabs/kafka-ui:latest`

## zk-web 
`docker run -d -p 8091:8080 --network=confluent-dev -e ZK_DEFAULT_NODE=192.168.64.11:2181/ --name zk-web -t tobilg/zookeeper-webui`

## swiss army knife for ops
`docker run -it --rm -v /var/run/docker.sock:/var/run/docker.sock  --net container:kafka-2.dev.confluent nicolaka/netshoot`
