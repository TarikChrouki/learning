## utilize ksqlDB (create topic)
`docker-compose exec kafka-0 bash`

`kafka-topics --bootstrap-server kafka-0:9092 --create --topic test_filled --replication-factor 3 --partitions 10`

## utilize ksqlDB (create stream pipe)
`docker-compose exec ksql-0 bash`

`ksql http://localhost:8088`

`set 'auto.offset.reset'='earliest';`

`create or replace stream test_filled_stream (some_name VARCHAR) with (kafka_topic='test_filled', value_format='DELIMITED');`

`create stream letters_stream as select explode(split(some_name,'')) as letter from test_filled_stream emit changes;`

`select letter, count() as occurrence from letters_stream group by letter emit changes;`

## utilize ksqlDB (produce records)
`docker-compose exec kafka-0 bash`

`kafka-producer-perf-test  --topic test_filled --throughput 1 --print-metrics --record-size 20 --num-records 1000  --producer-props bootstrap.servers=http://kafka-0:9092`

## observe ksql metrics

## lag investigation (create topic)
`docker-compose exec kafka-0 bash`

`kafka-topics --bootstrap-server kafka-0:9092 --create --topic test1`

## lag investigation (produce and consume in parallel-separate terminals)
`kafka-producer-perf-test  --topic test1 --throughput 10000000 --print-metrics --record-size 200 --num-records 10000000  --producer-props bootstrap.servers=http://localhost:9092`

`kafka-consumer-perf-test --bootstrap-server kafka-0:9092 --topic test1 --group testconsumer2  --show-detailed-stats --print-metrics --fetch-size 1 --messages 1000000000 --threads 1`

## observe lag metrics
i.e. `cd ./monitoring`

`docker-compose exec kafka-lag-exporter bash`

`curl -X GET -g http://localhost:9999 | grep testconsumer2`

## try rebalance (find out broker ids)
`docker-compose exec kafka-0 bash`

`zookeeper-shell  zk1:2181 get /brokers/ids`

## try rebalance (create a skewed topic)
`kafka-topics --bootstrap-server localhost:9092  --create --topic to_rebalance  --replica-assignment 1:2,1:2,1:2,1:2,1:2,1:2,1:2,1:2`

## try rebalance (produce messages)
`kafka-producer-perf-test  --topic to_rebalance  --num-records 20000 --record-size 1000 --throughput 1000000000 --producer-props bootstrap.servers=kafka-0:9092,kafka-1:9092`

## try rebalance (consume messages)
`kafka-consumer-perf-test --bootstrap-server kafka-0:9092,kafka-1:9092  --topic to_rebalance --group test-group --threads 1 --show-detailed-stats --timeout 1000000 --reporting-interval 5000 --messages 10000000`

## try rebalance (check current state)
`kafka-consumer-groups --bootstrap-server kafka-0:9092 --describe --group test-group`

`kafka-configs --bootstrap-server kafka-1:9092 --describe --topic to_rebalance`

`kafka-topics --bootstrap-server kafka-1:9092 --describe --topic to_rebalance`

## try rebalance (start rebalance)

`confluent-rebalancer execute --bootstrap-server kafka-1:9092  --metrics-bootstrap-server kafka-0:9092,kafka-1:9092  --throttle 1000000 --verbose` take a look at the output confirm with 'y'

check status
`confluent-rebalancer status  --bootstrap-server kafka-1:9092`

## perform rolling upgrade (check version)
`docker-compose exec kafka-0 bash`

`kafka-topics --bootstrap-server kafka-0:9092 -version`

## perform rolling update (investigate properties)
`https://github.com/confluentinc/cp-ansible/blob/7.2.1-post/docs/VARIABLES.md`

##  perform rolling update (adapt inventory)
add `confluent_package_version: 7.2.0`

## perform rolling update (execute playbook)
`ansible-playbook -i inventories/development/development.yml confluent.platform.all --extra-vars deployment_strategy=rolling`

## perform rolling update (check new version)
`kafka-topics --bootstrap-server kafka-0:9092 -version`

## acks vs performance measurements (create topic)
`kafka-topics --bootstrap-server kafka-0:9092 --create --topic withmininsync --replication-factor 3 --partitions 5 --config min.insync.replicas=3`

`kafka-topics --bootstrap-server kafka-0:9092 --describe --topic withmininsync`

`kafka-configs --bootstrap-server kafka-0:9092 --describe --entity-type topics --entity-name  withmininsync --all`

## acks vs performance measurement (perf produce with diff acks modes)
`kafka-producer-perf-test  --topic withmininsync --num-records 10000 --record-size 1000 --producer-props acks=all  bootstrap.servers=kafka-0:9092 --throughput -1 `

`kafka-producer-perf-test  --topic withmininsync --num-records 10000 --record-size 1000 --producer-props acks=0  bootstrap.servers=kafka-0:9092 --throughput -1 `

what about other producer configs `batch.size=400000`
`linger.ms=500`

## acks vs performance measurement (endToEnd latency)
`kafka-run-class kafka.tools.EndToEndLatency kafka-0:9092 withmininsync 1000 -1 1024`

`kafka-run-class kafka.tools.EndToEndLatency kafka-0:9092 withmininsync 1000 1 1024`

## compression vs performance measurement (create topic with broker compression set)
`kafka-topics --bootstrap-server kafka-0:9092 --create --topic withcompression --partitions 1 --config compression.type=snappy`

confirm settings `kafka-configs --bootstrap-server kafka-0:9092 --describe --entity-type topics --entity-name  withcompression --all`

produce few records `kafka-console-producer --bootstrap-server kafka-0:9092 --topic withcompression`

## compression vs performance measurement (check log file)
`kafka-topics --bootstrap-server kafka-0:9092 --topic withcompression --describe`

`cat /etc/kafka/server.properties` and then `kafka-run-class kafka.tools.DumpLogSegments --deep-iteration --files /var/lib/kafka/data/withcompression-0/00000000000000000000.log`

## compression vs performance measurement (producer with compression type setting)
`kafka-console-producer --bootstrap-server kafka-0:9092 --topic withcompression --compression-codec gzip`
and then check again

`kafka-run-class kafka.tools.DumpLogSegments --deep-iteration --files /var/lib/kafka/data/withcompression-0/00000000000000000000.log`

## compression vs performance measurement (confirm behaviour)
`kafka-producer-perf-test  --topic withcompression  --num-records 10 --record-size 40 --throughput 1 --producer-props bootstrap.servers=kafka-0:9092,kafka-1:9092` and

`kafka-producer-perf-test  --topic withcompression  --num-records 10 --record-size 40 --throughput -1 --producer-props bootstrap.servers=kafka-0:9092,kafka-1:9092` and look at the content

`kafka-run-class kafka.tools.DumpLogSegments --deep-iteration --files /var/lib/kafka/data/withcompression-0/00000000000000000000.log`

## compression vs performance measurement  (compare results with different producer-side compression settings)
`kafka-producer-perf-test  --topic withcompression  --num-records 1000000 --record-size 1000 --throughput -1 --producer-props compression.type=gzip bootstrap.servers=kafka-0:9092,kafka-1:9092`

`kafka-producer-perf-test  --topic withcompression  --num-records 1000000 --record-size 1000 --throughput -1 --producer-props compression.type=snappy bootstrap.servers=kafka-0:9092,kafka-1:9092`

`kafka-run-class kafka.tools.EndToEndLatency kafka-0:9092 withcompression 10000 1 1024`