# playKafka

## Concept
### Broker
Servers: Kafka is run as a cluster of one or more servers that can span multiple datacenters or cloud regions. Some of these servers form the storage layer, called the brokers.

### Event 消息事件
Producers generate event

event:
- key
- value
- timestamp

Producers and consumers are decoupled and agnostic of each other 生产者和消费者互相解耦和不依赖对方的，不用互相等待

### Topics
负责存储 event，保证序列化

Events are organized and durably stored in topics

Topics in Kafka are always multi-producer and multi-subscriber

events are not deleted after consumption，虽然不删除，但是可以定义过期时间，到期自动删除

### Partition 分区
Topics are partitioned, meaning a topic is spread over a number of "buckets" located on different Kafka brokers

不同生产者生产的events中如果拥有相同key的会被分配并附加到同一个 “分区” 中。
Kafka保证订阅特定key的消费者会找到那个“分区”，并且可以保证读取event的顺序和写入顺序一样。

### replication 备份
multiple brokers that have a copy of the data just in case things go wrong


## install kafka
brew install openjdk@17

```
For the system Java wrappers to find this JDK, symlink it with
  sudo ln -sfn /usr/local/opt/openjdk@17/libexec/openjdk.jdk /Library/Java/JavaVirtualMachines/openjdk-17.jdk

openjdk@17 is keg-only, which means it was not symlinked into /usr/local,
because this is an alternate version of another formula.

If you need to have openjdk@17 first in your PATH, run:
  echo 'export PATH="/usr/local/opt/openjdk@17/bin:$PATH"' >> /Users/frank/.bash_profile

For compilers to find openjdk@17 you may need to set:
  export CPPFLAGS="-I/usr/local/opt/openjdk@17/include"
```

```
MacBook-Pro:playKafka frank$ source ~/.bash_profile 
MacBook-Pro:playKafka frank$ java -version
openjdk version "17.0.13" 2024-10-15
OpenJDK Runtime Environment Homebrew (build 17.0.13+0)
OpenJDK 64-Bit Server VM Homebrew (build 17.0.13+0, mixed mode, sharing)
```

brew install kafka
```
==> Installing dependencies for kafka: harfbuzz, openjdk and zookeeper
==> Installing kafka dependency: harfbuzz
==> Downloading https://ghcr.io/v2/homebrew/core/harfbuzz/manifests/10.1.0
Already downloaded: /Users/frank/Library/Caches/Homebrew/downloads/7942a4b7c93b609fd780ade60fd720c899f44e3a2859d9327732371179192224--harfbuzz-10.1.0.bottle_manifest.json
==> Pouring harfbuzz--10.1.0.sonoma.bottle.tar.gz
🍺  /usr/local/Cellar/harfbuzz/10.1.0: 77 files, 9.8MB
==> Installing kafka dependency: openjdk
==> Downloading https://ghcr.io/v2/homebrew/core/openjdk/manifests/23.0.1
Already downloaded: /Users/frank/Library/Caches/Homebrew/downloads/d65952cb25f9d90a766cbfa0f6989689ce4c9516cff131754f9615bce1560a01--openjdk-23.0.1.bottle_manifest.json
==> Pouring openjdk--23.0.1.sonoma.bottle.tar.gz
🍺  /usr/local/Cellar/openjdk/23.0.1: 602 files, 336.9MB
==> Installing kafka dependency: zookeeper
==> Downloading https://ghcr.io/v2/homebrew/core/zookeeper/manifests/3.9.3
Already downloaded: /Users/frank/Library/Caches/Homebrew/downloads/7cfc5f181d59303c636ba3350deca15fce15c7ef4593ff5d9da8db0f845110d1--zookeeper-3.9.3.bottle_manifest.json
==> Pouring zookeeper--3.9.3.sonoma.bottle.tar.gz
==> Downloading https://raw.githubusercontent.com/apache/zookeeper/release-3.9.3/conf/logback.xml
```
自动安装依赖之后，java 版本发生变化
```
MacBook-Pro:playKafka frank$ java -version
openjdk version "17.0.13" 2024-10-15
OpenJDK Runtime Environment Homebrew (build 17.0.13+0)
OpenJDK 64-Bit Server VM Homebrew (build 17.0.13+0, mixed mode, sharing)
```

## start kafka

以下命令执行成功之后可知 zookeeper 的进程ID是 88322
```
MacBook-Pro:playKafka frank$ brew services start zookeeper
==> Successfully started `zookeeper` (label: homebrew.mxcl.zookeeper)
MacBook-Pro:playKafka frank$ ps -ef|grep zookeeper
  501 88322     1   0 10:58PM ??         0:01.83 /usr/local/opt/openjdk/bin/java -Dzookeeper.log.dir=/usr/local/Cellar/zookeeper/3.9.3/libexec/bin/../logs -Dzookeeper.log.file=zookeeper-frank-server-MacBook-Pro.local.log -XX:+HeapDumpOnOutOfMemoryError -XX:OnOutOfMemoryError=kill -9 %p -cp /usr/local/Cellar/zookeeper/3.9.3/libexec/bin/../zookeeper-metrics-providers/zookeeper-prometheus-metrics/target/classes:/usr/local/Cellar/zookeeper/3.9.3/libexec/bin/../zookeeper-server/target/classes:/usr/local/Cellar/zookeeper/3.9.3/libexec/bin/../build/classes:/usr/local/Cellar/zookeeper/3.9.3/libexec/bin/../zookeeper-metrics-providers/zookeeper-prometheus-metrics/target/lib/*.jar:/usr/local/Cellar/zookeeper/3.9.3/libexec/bin/../zookeeper-server/target/lib/*.jar:/usr/local/Cellar/zookeeper/3.9.3/libexec/bin/../build/lib/*.jar:/usr/local/Cellar/zookeeper/3.9.3/libexec/bin/../lib/zookeeper-prometheus-metrics-3.9.3.jar:/usr/local/Cellar/zookeeper/3.9.3/libexec/bin/../lib/zookeeper-jute-3.9.3.jar:/usr/local/Cellar/zookeeper/3.9.3/libexec/bin/../lib/zookeeper-3.9.3.jar:/usr/local/Cellar/zookeeper/3.9.3/libexec/bin/../lib/snappy-java-1.1.10.5.jar:/usr/local/Cellar/zookeeper/3.9.3/libexec/bin/../lib/slf4j-api-1.7.30.jar:/usr/local/Cellar/zookeeper/3.9.3/libexec/bin/../lib/simpleclient_servlet-0.9.0.jar:/usr/local/Cellar/zookeeper/3.9.3/libexec/bin/../lib/simpleclient_hotspot-0.9.0.jar:/usr/local/Cellar/zookeeper/3.9.3/libexec/bin/../lib/simpleclient_common-0.9.0.jar:/usr/local/Cellar/zookeeper/3.9.3/libexec/bin/../lib/simpleclient-0.9.0.jar:/usr/local/Cellar/zookeeper/3.9.3/libexec/bin/../lib/netty-transport-native-unix-common-4.1.113.Final.jar:/usr/local/Cellar/zookeeper/3.9.3/libexec/bin/../lib/netty-transport-native-epoll-4.1.113.Final-linux-x86_64.jar:/usr/local/Cellar/zookeeper/3.9.3/libexec/bin/../lib/netty-transport-classes-epoll-4.1.113.Final.jar:/usr/local/Cellar/zookeeper/3.9.3/libexec/bin/../lib/netty-transport-4.1.113.Final.jar:/usr/local/Cellar/zookeeper/3.9.3/libexec/bin/../lib/netty-tcnative-classes-2.0.66.Final.jar:/usr/local/Cellar/zookeeper/3.9.3/libexec/bin/../lib/netty-tcnative-boringssl-static-2.0.66.Final.jar:/usr/local/Cellar/zookeeper/3.9.3/libexec/bin/../lib/netty-tcnative-boringssl-static-2.0.66.Final-windows-x86_64.jar:/usr/local/Cellar/zookeeper/3.9.3/libexec/bin/../lib/netty-tcnative-boringssl-static-2.0.66.Final-osx-x86_64.jar:/usr/local/Cellar/zookeeper/3.9.3/libexec/bin/../lib/netty-tcnative-boringssl-static-2.0.66.Final-osx-aarch_64.jar:/usr/local/Cellar/zookeeper/3.9.3/libexec/bin/../lib/netty-tcnative-boringssl-static-2.0.66.Final-linux-x86_64.jar:/usr/local/Cellar/zookeeper/3.9.3/libexec/bin/../lib/netty-tcnative-boringssl-static-2.0.66.Final-linux-aarch_64.jar:/usr/local/Cellar/zookeeper/3.9.3/libexec/bin/../lib/netty-resolver-4.1.113.Final.jar:/usr/local/Cellar/zookeeper/3.9.3/libexec/bin/../lib/netty-handler-4.1.113.Final.jar:/usr/local/Cellar/zookeeper/3.9.3/libexec/bin/../lib/netty-common-4.1.113.Final.jar:/usr/local/Cellar/zookeeper/3.9.3/libexec/bin/../lib/netty-codec-4.1.113.Final.jar:/usr/local/Cellar/zookeeper/3.9.3/libexec/bin/../lib/netty-buffer-4.1.113.Final.jar:/usr/local/Cellar/zookeeper/3.9.3/libexec/bin/../lib/metrics-core-4.1.12.1.jar:/usr/local/Cellar/zookeeper/3.9.3/libexec/bin/../lib/logback-core-1.2.13.jar:/usr/local/Cellar/zookeeper/3.9.3/libexec/bin/../lib/logback-classic-1.2.13.jar:/usr/local/Cellar/zookeeper/3.9.3/libexec/bin/../lib/jline-2.14.6.jar:/usr/local/Cellar/zookeeper/3.9.3/libexec/bin/../lib/jetty-util-ajax-9.4.56.v20240826.jar:/usr/local/Cellar/zookeeper/3.9.3/libexec/bin/../lib/jetty-util-9.4.56.v20240826.jar:/usr/local/Cellar/zookeeper/3.9.3/libexec/bin/../lib/jetty-servlet-9.4.56.v20240826.jar:/usr/local/Cellar/zookeeper/3.9.3/libexec/bin/../lib/jetty-server-9.4.56.v20240826.jar:/usr/local/Cellar/zookeeper/3.9.3/libexec/bin/../lib/jetty-security-9.4.56.v20240826.jar:/usr/local/Cellar/zookeeper/3.9.3/libexec/bin/../lib/jetty-io-9.4.56.v20240826.jar:/usr/local/Cellar/zookeeper/3.9.3/libexec/bin/../lib/jetty-http-9.4.56.v20240826.jar:/usr/local/Cellar/zookeeper/3.9.3/libexec/bin/../lib/javax.servlet-api-3.1.0.jar:/usr/local/Cellar/zookeeper/3.9.3/libexec/bin/../lib/jackson-databind-2.15.2.jar:/usr/local/Cellar/zookeeper/3.9.3/libexec/bin/../lib/jackson-core-2.15.2.jar:/usr/local/Cellar/zookeeper/3.9.3/libexec/bin/../lib/jackson-annotations-2.15.2.jar:/usr/local/Cellar/zookeeper/3.9.3/libexec/bin/../lib/commons-io-2.17.0.jar:/usr/local/Cellar/zookeeper/3.9.3/libexec/bin/../lib/commons-cli-1.5.0.jar:/usr/local/Cellar/zookeeper/3.9.3/libexec/bin/../lib/audience-annotations-0.12.0.jar:/usr/local/Cellar/zookeeper/3.9.3/libexec/bin/../zookeeper-*.jar:/usr/local/Cellar/zookeeper/3.9.3/libexec/bin/../zookeeper-server/src/main/resources/lib/*.jar:/usr/local/etc/zookeeper: -Xmx1000m -Dapple.awt.UIElement=true -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.local.only=false org.apache.zookeeper.server.quorum.QuorumPeerMain /usr/local/etc/zookeeper/zoo.cfg
  501 88382 83348   0 10:59PM ttys002    0:00.01 grep -G zookeeper
```

以下命令执行之后可见 Kafka的进程ID为 88634 
```
MacBook-Pro:playKafka frank$ brew services start kafka
==> Successfully started `kafka` (label: homebrew.mxcl.kafka)
MacBook-Pro:playKafka frank$ ps -ef|grep kafka
  501 88634     1   0 11:00PM ??         0:04.48 /usr/local/opt/openjdk/libexec/openjdk.jdk/Contents/Home/bin/java -Xmx1G -Xms1G -server -XX:+UseG1GC -XX:MaxGCPauseMillis=20 -XX:InitiatingHeapOccupancyPercent=35 -XX:+ExplicitGCInvokesConcurrent -XX:MaxInlineLevel=15 -Djava.awt.headless=true -Xlog:gc*:file=/usr/local/Cellar/kafka/3.9.0/libexec/bin/../logs/kafkaServer-gc.log:time,tags:filecount=10,filesize=100M -Dcom.sun.management.jmxremote=true -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false -Dkafka.logs.dir=/usr/local/Cellar/kafka/3.9.0/libexec/bin/../logs -Dlog4j.configuration=file:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../config/log4j.properties -cp /usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/activation-1.1.1.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/aopalliance-repackaged-2.6.1.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/argparse4j-0.7.0.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/audience-annotations-0.12.0.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/caffeine-2.9.3.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/commons-beanutils-1.9.4.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/commons-cli-1.4.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/commons-collections-3.2.2.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/commons-digester-2.1.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/commons-io-2.14.0.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/commons-lang3-3.12.0.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/commons-logging-1.2.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/commons-validator-1.7.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/connect-api-3.9.0.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/connect-basic-auth-extension-3.9.0.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/connect-json-3.9.0.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/connect-mirror-3.9.0.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/connect-mirror-client-3.9.0.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/connect-runtime-3.9.0.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/connect-transforms-3.9.0.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/error_prone_annotations-2.10.0.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/hk2-api-2.6.1.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/hk2-locator-2.6.1.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/hk2-utils-2.6.1.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/jackson-annotations-2.16.2.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/jackson-core-2.16.2.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/jackson-databind-2.16.2.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/jackson-dataformat-csv-2.16.2.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/jackson-datatype-jdk8-2.16.2.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/jackson-jaxrs-base-2.16.2.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/jackson-jaxrs-json-provider-2.16.2.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/jackson-module-afterburner-2.16.2.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/jackson-module-jaxb-annotations-2.16.2.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/jackson-module-scala_2.13-2.16.2.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/jakarta.activation-api-1.2.2.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/jakarta.annotation-api-1.3.5.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/jakarta.inject-2.6.1.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/jakarta.validation-api-2.0.2.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/jakarta.ws.rs-api-2.1.6.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/jakarta.xml.bind-api-2.3.3.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/javassist-3.29.2-GA.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/javax.activation-api-1.2.0.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/javax.annotation-api-1.3.2.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/javax.servlet-api-3.1.0.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/javax.ws.rs-api-2.1.1.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/jaxb-api-2.3.1.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/jersey-client-2.39.1.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/jersey-common-2.39.1.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/jersey-container-servlet-2.39.1.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/jersey-container-servlet-core-2.39.1.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/jersey-hk2-2.39.1.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/jersey-server-2.39.1.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/jetty-client-9.4.56.v20240826.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/jetty-continuation-9.4.56.v20240826.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/jetty-http-9.4.56.v20240826.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/jetty-io-9.4.56.v20240826.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/jetty-security-9.4.56.v20240826.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/jetty-server-9.4.56.v20240826.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/jetty-servlet-9.4.56.v20240826.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/jetty-servlets-9.4.56.v20240826.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/jetty-util-9.4.56.v20240826.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/jetty-util-ajax-9.4.56.v20240826.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/jline-3.25.1.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/jopt-simple-5.0.4.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/jose4j-0.9.4.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/jsr305-3.0.2.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/kafka-clients-3.9.0.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/kafka-group-coordinator-3.9.0.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/kafka-group-coordinator-api-3.9.0.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/kafka-metadata-3.9.0.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/kafka-raft-3.9.0.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/kafka-server-3.9.0.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/kafka-server-common-3.9.0.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/kafka-shell-3.9.0.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/kafka-storage-3.9.0.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/kafka-storage-api-3.9.0.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/kafka-streams-3.9.0.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/kafka-streams-examples-3.9.0.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/kafka-streams-scala_2.13-3.9.0.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/kafka-streams-test-utils-3.9.0.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/kafka-tools-3.9.0.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/kafka-tools-api-3.9.0.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/kafka-transaction-coordinator-3.9.0.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/kafka_2.13-3.9.0.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/lz4-java-1.8.0.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/maven-artifact-3.9.6.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/metrics-core-2.2.0.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/metrics-core-4.1.12.1.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/netty-buffer-4.1.111.Final.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/netty-codec-4.1.111.Final.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/netty-common-4.1.111.Final.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/netty-handler-4.1.111.Final.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/netty-resolver-4.1.111.Final.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/netty-transport-4.1.111.Final.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/netty-transport-classes-epoll-4.1.111.Final.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/netty-transport-native-epoll-4.1.111.Final.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/netty-transport-native-unix-common-4.1.111.Final.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/opentelemetry-proto-1.0.0-alpha.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/osgi-resource-locator-1.0.3.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/paranamer-2.8.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/pcollections-4.0.1.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/plexus-utils-3.5.1.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/protobuf-java-3.25.5.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/reflections-0.10.2.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/reload4j-1.2.25.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/rocksdbjni-7.9.2.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/scala-collection-compat_2.13-2.10.0.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/scala-java8-compat_2.13-1.0.2.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/scala-library-2.13.14.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/scala-logging_2.13-3.9.5.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/scala-reflect-2.13.14.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/slf4j-api-1.7.36.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/slf4j-reload4j-1.7.36.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/snappy-java-1.1.10.5.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/swagger-annotations-2.2.8.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/trogdor-3.9.0.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/zookeeper-3.8.4.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/zookeeper-jute-3.8.4.jar:/usr/local/Cellar/kafka/3.9.0/libexec/bin/../libs/zstd-jni-1.5.6-4.jar kafka.Kafka /usr/local/etc/kafka/server.properties
  501 89026 83348   0 11:01PM ttys002    0:00.01 grep -G kafka
MacBook-Pro:playKafka frank$ 
```

## 使用Kafka

创建一个 topic
```
MacBook-Pro:playKafka frank$ kafka-topics --create --topic test --bootstrap-server localhost:9092 --partitions 1 --replication-factor 1
Created topic test.

```

以下根据端口查出来的进程ID 88634 为Kafka的进程，与前面一致
```
MacBook-Pro:playKafka frank$ lsof -i:9092
COMMAND   PID  USER   FD   TYPE             DEVICE SIZE/OFF NODE NAME
java    88634 frank  169u  IPv6 0x593444b764705437      0t0  TCP localhost:63512->localhost:XmlIpcRegSvc (ESTABLISHED)
java    88634 frank  170u  IPv6 0x51f5012e5c1fb657      0t0  TCP *:XmlIpcRegSvc (LISTEN)
java    88634 frank  171u  IPv6 0x7afed040990a5d95      0t0  TCP localhost:XmlIpcRegSvc->localhost:63512 (ESTABLISHED)
```

检查topic是否创建成功
```
MacBook-Pro:playKafka frank$ kafka-topics --list --bootstrap-server localhost:9092
test
```

### 开两个终端，分别运行生产者和消费者进程
Producer
```
MacBook-Pro:playKafka frank$ kafka-console-producer --topic test --bootstrap-server localhost:9092
>hello
>work
>hello
>world
>

```

Consumer
```
MacBook-Pro:~ frank$ kafka-console-consumer --topic test --from-beginning --bootstrap-server localhost:9092
hello
work
hello
world

```
可以看到消息在生产者进程中输入之后，会在消费者终端自动读取出来。


## References
- https://kafka.apache.org/intro

