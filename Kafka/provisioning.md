### kafka 프로비저닝

#### 1. kafka 30.1.8 설치
```bash
$ helm install kafka -f values.yaml oci://registry-1.docker.io/bitnamicharts/kafka --version 30.1.8

Pulled: registry-1.docker.io/bitnamicharts/kafka:30.1.8
Digest: sha256:6714a9cb82b6fff281bb1fc8a7d8eea3b263ae802798bed8a9ebcc5f54366924
NAME: kafka
LAST DEPLOYED: Thu Jan  9 10:12:47 2025
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
CHART NAME: kafka
CHART VERSION: 30.1.8
APP VERSION: 3.8.1

** Please be patient while the chart is being deployed **

Kafka can be accessed by consumers via port 9092 on the following DNS name from within your cluster:

    kafka.default.svc.cluster.local

Each Kafka broker can be accessed by producers via port 9092 on the following DNS name(s) from within your cluster:

    kafka-controller-0.kafka-controller-headless.default.svc.cluster.local:9092
    kafka-controller-1.kafka-controller-headless.default.svc.cluster.local:9092
    kafka-controller-2.kafka-controller-headless.default.svc.cluster.local:9092

To create a pod that you can use as a Kafka client run the following commands:

    kubectl run kafka-client --restart='Never' --image docker.io/bitnami/kafka:3.8.1-debian-12-r0 --namespace default --command -- sleep infinity
    kubectl exec --tty -i kafka-client --namespace default -- bash

    PRODUCER:
        kafka-console-producer.sh \
            --bootstrap-server kafka.default.svc.cluster.local:9092 \
            --topic test

    CONSUMER:
        kafka-console-consumer.sh \
            --bootstrap-server kafka.default.svc.cluster.local:9092 \
            --topic test \
            --from-beginning

WARNING: There are "resources" sections in the chart not set. Using "resourcesPreset" is not recommended for production. For production installations, please set the following values according to your workload needs:
  - volumePermissions.resources
+info https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/
```

#### 2. Persistent Volume 생성
- Persistent Volume Claim 목록 조회
  ```bash
  $ kubectl get pvc

  NAME                      STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
  data-kafka-controller-0   Pending                                                     28h
  data-kafka-controller-1   Pending                                                     28h
  data-kafka-controller-2   Pending                                                     28h
  ```

- Persistent Volume 생성
  ```bash
  $ kubectl create -f kafka-pv-1.yaml
  $ kubectl create -f kafka-pv-2.yaml
  $ kubectl create -f kafka-pv-3.yaml
  ```

- Persistent Volume 생성
  ```bash
  $ kubectl get pv

  NAME         CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                             STORAGECLASS   REASON   AGE
  kafka-pv-1   8Gi        RWO            Recycle          Bound    default/data-kafka-controller-0                           38h
  kafka-pv-2   8Gi        RWO            Recycle          Bound    default/data-kafka-controller-1                           37h
  kafka-pv-3   8Gi        RWO            Recycle          Bound    default/data-kafka-controller-2                           37h
  ```

- Persistent Volume Claim와 Persistent Volume 연결 확인
  ```bash
  $ kubectl get pvc
  NAME                      STATUS   VOLUME       CAPACITY   ACCESS MODES   STORAGECLASS   AGE
  data-kafka-controller-0   Bound    kafka-pv-1   8Gi        RWO                           10m
  data-kafka-controller-1   Bound    kafka-pv-2   8Gi        RWO                           10m
  data-kafka-controller-2   Bound    kafka-pv-3   8Gi        RWO                           10m
  ```

#### 3. [새로운 터미널] Producer pod 생성 (kafka 설치 시 출력된 명령어를 참고하여 사용)
  ```bash
  $ kubectl run kafka-pro --restart='Never' --image docker.io/bitnami/kafka:3.8.1-debian-12-r0 --namespace default --command -- sleep infinity
  $ kubectl exec --tty -i kafka-pro --namespace default -- bash
  I have no name!@kafka-pro:/$ kafka-console-producer.sh --bootstrap-server kafka.default.svc.cluster.local:9092 --topic test
  >123
  >12
  >
  ```

#### 4. [새로운 터미널] Consumer pod 생성 (kafka 설치 시 출력된 명령어를 참고하여 사용)
  ```bash
  $ kubectl run kafka-con --restart='Never' --image docker.io/bitnami/kafka:3.8.1-debian-12-r0 --namespace default --command -- sleep infinity
  $ kubectl exec --tty -i kafka-con --namespace default -- bash
  I have no name!@kafka-con:/$ kafka-console-consumer.sh --bootstrap-server kafka.default.svc.cluster.local:9092 --topic test --from-beginning
  123
  12
  ```

#### 5. External Java Server와 연동
- password 알아내기
  ```bash
  $ kubectl get secret kafka-user-passwords -o yaml
  apiVersion: v1
  data:
    client-passwords: NkxOemJrSlZNQQ==
    controller-password: Yk9Sck9SY0x4Sw==
    inter-broker-password: WGFYR2xHNTVRUA==
    system-user-password: NkxOemJrSlZNQQ==
  kind: Secret
  metadata:
    annotations:
      meta.helm.sh/release-name: kafka
      meta.helm.sh/release-namespace: default
    creationTimestamp: "2025-01-09T02:12:03Z"
    labels:
      app.kubernetes.io/instance: kafka
      app.kubernetes.io/managed-by: Helm
      app.kubernetes.io/name: kafka
      app.kubernetes.io/version: 3.8.1
      helm.sh/chart: kafka-30.1.8
    name: kafka-user-passwords
    namespace: default
    resourceVersion: "81332330"
    uid: efcdf157-f72c-4b76-b6a9-3ae8c3c02bff
  type: Opaque

  $ echo "NkxOemJrSlZNQQ==" | base64 --decode
  6LNzbkJVMA
  ```

- Java Producer 
  ```java
  import java.util.Properties;
  import org.apache.kafka.clients.producer.ProducerRecord;
  import org.apache.kafka.clients.producer.KafkaProducer;
  import org.apache.kafka.clients.producer.Producer;
  import com.fasterxml.jackson.databind.ObjectMapper;

    ...

  Properties props = new Properties();
  props.put("bootstrap.servers", "110.165.19.180:30921,110.165.19.180:30922,110.165.19.180:30923");
  props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
  props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
  props.put("client.id", "test-producer");
  props.put("security.protocol", "SASL_PLAINTEXT");
  props.put("request.timeout.ms", "5000");
  props.put("sasl.mechanism", "PLAIN");
  props.put("sasl.jaas.config",
          "org.apache.kafka.common.security.plain.PlainLoginModule required " +
                  "username=\"user1\" " +
                  "password=\"6LNzbkJVMA\";");
  Producer<String, String> producer = new KafkaProducer<>(props);

  ProducerRecord<String, String> record = new ProducerRecord<>("test", value);
  producer.send(record).get(5, TimeUnit.SECONDS);
  for (int i = 0; i < 1000000; i++) {
      final int index = i;
      new Thread(() -> {
          try {
              Message message = new Message(value + index);
              String jsonMessage = new ObjectMapper().writeValueAsString(message);
              ProducerRecord<String, String> threadRecord = new ProducerRecord<>("test", jsonMessage);
              producer.send(threadRecord).get(5, TimeUnit.SECONDS);
          } catch (Exception e) {
              System.out.println(e);
          }
      }).start();
  }

    ...
  ```

- Java Consumer
  ```java
  import java.util.Properties;
  import org.apache.kafka.clients.consumer.Consumer;
  import org.apache.kafka.clients.consumer.ConsumerRecord;
  import org.apache.kafka.clients.consumer.ConsumerRecords;
  import org.apache.kafka.clients.consumer.KafkaConsumer;
  import com.fasterxml.jackson.databind.ObjectMapper;

    ...

  Properties props = new Properties();
  props.put("bootstrap.servers", "110.165.19.180:30921,110.165.19.180:30922,110.165.19.180:30923");
  props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
  props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
  props.put("client.id", "test-consumer");
  props.put("group.id", "test-group");
  props.put("max.poll.records", 100);
  props.put("security.protocol", "SASL_PLAINTEXT");
  props.put("request.timeout.ms", "5000");
  props.put("sasl.mechanism", "PLAIN");
  props.put("sasl.jaas.config",
          "org.apache.kafka.common.security.plain.PlainLoginModule required " +
                  "username=\"user1\" " +
                  "password=\"6LNzbkJVMA\";");
  Consumer<String, String> consumer = new KafkaConsumer<>(props);
  consumer.subscribe(Arrays.asList("test"));
  new Thread(() -> {
      try {
          while (true) {
              ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(1));
              for (ConsumerRecord<String, String> record : records) {
                  Message message;
                  try {
                      message = new ObjectMapper().readValue(record.value(), Message.class);
                  } catch (JsonProcessingException e) {
                      logger.error("JSON Parsing Error: " + e.getMessage());
                      continue;
                  }
                  System.out.printf("Received message: %s%n", message.getValue());
              }
          }
      } catch (Exception e) {
          logger.info("Consumer thread interrupted");
      } finally {
          consumer.close();
      }
  }).start();

    ...
  ```