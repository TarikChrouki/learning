# Installing Confluent by CP Ansible

The project contain 4 use case of installing confluent platform.
We simulate a VM host with docker image of centos7, and we build a custom image with pre-installing some mandatory package(java, open-ssl....) to save time during installation.

development : base installation without encryption or security
tls-enabled : Encryption using custom certifications
tls-plain : Encryption using custom certifications, and SASL PLAIN for authentication
mtls : Using mTLS for authentication

## Setup Ansible

Steps to configure ansible env ( tested on WSL2) :

- Install python3

- Create a new Python virtual environment: `python3 -m venv confluent-with-ansible`

- Activate a Python virtual environment: `source confluent-with-ansible/bin/activate`

- Install Ansible in a virtual environment: `python3 -m pip install ansible-core==2.11.12` 

- Install requirements:  `ansible-galaxy install -r requirements.yml`

## 1 Build the custom image for confluent components

`docker build --rm -t local/centos7-efg .`

## 2 Start all the components of our infra

`docker-compose up -d`

## 2a (optional) Generate custom certification (for tls-enabled & tls-plain)

`ansible-playbook certs/certificates.yml -i inventories/tls-enabled/development.yml`

A custom certifications will be generated at certs/generated_ssl_files

## 3 Installing confluent platform with Ansible

`ansible-galaxy collection install confluent.platform:7.2.1`

## 4 go with development (it takes time):

`ansible-playbook -i inventories/development/development.yml confluent.platform.all`

## 4a (optional) tls-enabled:

`ansible-playbook -i inventories/tls-enabled/development.yml confluent.platform.all --skip-tags validate_ssl_keys_certs`

## 4b (optional) tls-plain:

`ansible-playbook -i inventories/tls-plain/development.yml confluent.platform.all --skip-tags validate_ssl_keys_certs`

## 5 available tasks
 `ansible-playbook -i inventories/development/development.yml confluent.platform.all --list-tasks`

## 6 run grafana/prometheus/lag-exporter
 `cd ./monitoring/`

 `docker-compose up -d`

## 7 go to grafana webUI (admin/admin)
 `<docker-host>:3000`

## 8 prometheus webUI
`<docker-host>:9000`

## 9 utilize ksqlDB (create topic)
`cd .`

`docker-compose exec kafka-0 bash`

`kafka-topics --bootstrap-server kafka-0:9092 --create --topic test_filled --replication-factor 3 --partitions 10`

## 10 utilize ksqlDB (create stream pipe)
`docker-compose exec ksql-0 bash`

`ksql http://localhost:8088`

`set 'auto.offset.reset'='earliest';`

`create or replace stream test_filled_stream (some_name VARCHAR) with (kafka_topic='test_filled', value_format='DELIMITED');`

`create stream letters_stream as select explode(split(some_name,'')) as letter from test_filled_stream emit changes;`

`select letter, count() as occurrence from letters_stream group by letter emit changes;`

## 11 utilize ksqlDB (produce records)
`docker-compose exec kafka-0 bash`

`kafka-producer-perf-test  --topic test_filled --throughput 1 --print-metrics --record-size 20 --num-records 1000  --producer-props bootstrap.servers=http://kafka-0:9092`

## 12 observe ksql metrics

## 13 lag investigation (create topic)
`docker-compose exec kafka-0 bash`

`kafka-topics --bootstrap-server kafka-0:9092 --create --topic test1`

## 14 lag investigation (produce and consume in parallel-separate terminals)
`kafka-producer-perf-test  --topic test1 --throughput 10000000 --print-metrics --record-size 200 --num-records 10000000  --producer-props bootstrap.servers=http://localhost:9092`

`kafka-consumer-perf-test --bootstrap-server kafka-0:9092 --topic test1 --group testconsumer2  --show-detailed-stats --print-metrics --fetch-size 1 --messages 1000000000 --threads 1`    

## 15 observe lag metrics
i.e. `cd ./monitoring` 

`docker-compose exec kafka-lag-exporter bash`

`curl -X GET -g http://localhost:9999 | grep testconsumer2`

## 16 try rebalance (find out broker ids)
`docker-compose exec kafka-0 bash`

`zookeeper-shell  zk1:2181 get /brokers/ids`

## 17 try rebalance (create a skewed topic)
`kafka-topics --bootstrap-server localhost:9092  --create --topic to_rebalance  --replica-assignment 1:2,1:2,1:2,1:2,1:2,1:2,1:2,1:2`

## 18 try rebalance (produce messages)
`kafka-producer-perf-test  --topic to_rebalance  --num-records 20000 --record-size 1000 --throughput 1000000000 --producer-props bootstrap.servers=kafka-0:9092,kafka-1:9092`

## 19 try rebalance (consume messages)
`kafka-consumer-perf-test --bootstrap-server kafka-0:9092,kafka-1:9092  --topic to_rebalance --group test-group --threads 1 --show-detailed-stats --timeout 1000000 --reporting-interval 5000 --messages 10000000`

## 20 try rebalance (check current state)
`kafka-consumer-groups --bootstrap-server kafka-0:9092 --describe --group test-group`

`kafka-configs --bootstrap-server kafka-1:9092 --describe --topic to_rebalance`

`kafka-topics --bootstrap-server kafka-1:9092 --describe --topic to_rebalance`

## 21 try rebalance (start rebalance)

`confluent-rebalancer execute --bootstrap-server kafka-1:9092  --metrics-bootstrap-server kafka-0:9092,kafka-1:9092  --throttle 1000000 --verbose` take a look at the output confirm with 'y'

check status 
`confluent-rebalancer status  --bootstrap-server kafka-1:9092`

## 22 perform rolling upgrade (check version)
`docker-compose exec kafka-0 bash`

`kafka-topics --bootstrap-server kafka-0:9092 -version`

## 23 perform rolling update (investigate properties)
`https://github.com/confluentinc/cp-ansible/blob/7.2.1-post/docs/VARIABLES.md`

## 24 perform rolling update (adapt inventory)
add `    confluent_package_version: 7.2.0`

## 25 perform rolling update (execute playbook)
`ansible-playbook -i inventories/development/development.yml confluent.platform.all --extra-vars deployment_strategy=rolling`

## 26 perform rolling update (check new version)
`kafka-topics --bootstrap-server kafka-0:9092 -version`

## 27 acks vs performance measurements (create topic)
`kafka-topics --bootstrap-server kafka-0:9092 --create --topic withmininsync --replication-factor 3 --partitions 5 --config min.insync.replicas=3`

`kafka-topics --bootstrap-server kafka-0:9092 --describe --topic withmininsync`

`kafka-configs --bootstrap-server kafka-0:9092 --describe --entity-type topics --entity-name  withmininsync --all`

## 28 acks vs performance measurement (perf produce with diff acks modes)
`kafka-producer-perf-test  --topic withmininsync --num-records 10000 --record-size 1000 --producer-props acks=all  bootstrap.servers=kafka-0:9092 --throughput -1 `

`kafka-producer-perf-test  --topic withmininsync --num-records 10000 --record-size 1000 --producer-props acks=0  bootstrap.servers=kafka-0:9092 --throughput -1 `

what about other producer configs `batch.size=400000`
`linger.ms=500`

## 29 acks vs performance measurement (endToEnd latency)
`kafka-run-class kafka.tools.EndToEndLatency kafka-0:9092 withmininsync 1000 -1 1024`

`kafka-run-class kafka.tools.EndToEndLatency kafka-0:9092 withmininsync 1000 1 1024`

## 30 compression vs performance measurement (create topic with broker compression set)
`kafka-topics --bootstrap-server kafka-0:9092 --create --topic withcompression --partitions 1 --config compression.type=snappy`

confirm settings `kafka-configs --bootstrap-server kafka-0:9092 --describe --entity-type topics --entity-name  withcompression --all`

produce few records `kafka-console-producer --bootstrap-server kafka-0:9092 --topic withcompression`

## 31 compression vs performance measurement (check log file)
`kafka-topics --bootstrap-server kafka-0:9092 --topic withcompression --describe`

`cat /etc/kafka/server.properties` and then `kafka-run-class kafka.tools.DumpLogSegments --deep-iteration --files /var/lib/kafka/data/withcompression-0/00000000000000000000.log`

## 32 compression vs performance measurement (producer with compression type setting)
`kafka-console-producer --bootstrap-server kafka-0:9092 --topic withcompression --compression-codec gzip`
and then check again

`kafka-run-class kafka.tools.DumpLogSegments --deep-iteration --files /var/lib/kafka/data/withcompression-0/00000000000000000000.log`

## 33 compression vs performance measurement (confirm behaviour)
`kafka-producer-perf-test  --topic withcompression  --num-records 10 --record-size 40 --throughput 1 --producer-props bootstrap.servers=kafka-0:9092,kafka-1:9092` and

`kafka-producer-perf-test  --topic withcompression  --num-records 10 --record-size 40 --throughput -1 --producer-props bootstrap.servers=kafka-0:9092,kafka-1:9092` and look at the content

`kafka-run-class kafka.tools.DumpLogSegments --deep-iteration --files /var/lib/kafka/data/withcompression-0/00000000000000000000.log`

## 34 compression vs performance measurement  (compare results with different producer-side compression settings)
`kafka-producer-perf-test  --topic withcompression  --num-records 1000000 --record-size 1000 --throughput -1 --producer-props compression.type=gzip bootstrap.servers=kafka-0:9092,kafka-1:9092`

`kafka-producer-perf-test  --topic withcompression  --num-records 1000000 --record-size 1000 --throughput -1 --producer-props compression.type=snappy bootstrap.servers=kafka-0:9092,kafka-1:9092`

`kafka-run-class kafka.tools.EndToEndLatency kafka-0:9092 withcompression 10000 1 1024`

## 35 swiss army knife (kcat) for devs
`docker run -it --rm --network confluent-dev  --name producer  confluentinc/cp-kafkacat kafkacat -b kafka-0:9092 -L -J`

## 36 kafdrop

`docker run -d --rm -p 9001:9000 -e KAFKA_BROKERCONNECT=kafka-0:9092,kafka-1:9092 -e JVM_OPTS="-Xms32M -Xmx64M" -e SERVER_SERVLET_CONTEXTPATH="/" --network="confluent-dev" obsidiandynamics/kafdrop`

## 37 akhq

`cd monitoring/akhq` & `docker-compose up -d`

## 38 kafka-ui

`docker run -p 8085:8080 -e KAFKA_CLUSTERS_0_NAME=local -e KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS=kafka-0:9092 --network=confluent-dev -d provectuslabs/kafka-ui:latest`

## 39 zk-web 
`docker run -d -p 8091:8080 --network=confluent-dev -e ZK_DEFAULT_NODE=192.168.64.11:2181/ --name zk-web -t tobilg/zookeeper-webui`

## 40 swiss army knife for ops
`docker run -it --rm -v /var/run/docker.sock:/var/run/docker.sock  --net container:kafka-2.dev.confluent nicolaka/netshoot
`
