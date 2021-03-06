.. Copyright 2019 Ververica GmbH.

   Licensed under the Apache License, Version 2.0 (the "License");
   you may not use this file except in compliance with the License.
   You may obtain a copy of the License at
   
        http://www.apache.org/licenses/LICENSE-2.0
   
   Unless required by applicable law or agreed to in writing, software
   distributed under the License is distributed on an "AS IS" BASIS,
   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   See the License for the specific language governing permissions and
   limitations under the License.

############
Apache Kafka
############

Stateful Functions offers an Apache Kafka I/O Module for reading from and writing to Kafka topics.
It is based on Apache Flink's universal `Kafka connector <https://ci.apache.org/projects/flink/flink-docs-stable/dev/connectors/kafka.html>`_ and provides exactly-once processing semantics.

.. contents:: :local:

Dependency
===========

To use the Kafka I/O Module, please include the following dependency in your pom.

.. code-block:: xml

    <dependency>
        <groupId>com.ververica</groupId>
        <artifactId>stateful-functions-kafka-io</artifactId>
        <version>{version}</version>
        <scope>provided</scope>
    </dependency>

Kafka Ingress Builder
=====================

A ``KafkaIngressBuilder`` declares an ingress spec for consuming from Kafka cluster. 

It accepts the following arguments: 

1) The ingress identifier associated with this ingress
2) The topic name / list of topic names
3) The address of the bootstrap servers
4) A ``KafkaIngressDeserializer`` for deserializing data from Kafka
5) Properties for the Kafka consumer

.. literalinclude:: ../../../src/main/java/com/ververica/statefun/docs/io/kafka/IngressSpecs.java
    :language: java
    :lines: 16-

Please refer to the Kafka `consumer configuration <https://docs.confluent.io/current/installation/configuration/consumer-configs.html>`_ documentation for the full list of available properties.

Kafka Deserializer
""""""""""""""""""

The Kafka ingress needs to know how to turn the binary data in Kafka into Java objects.
The ``KafkaIngressDeserializer`` allows users to specify such a schema.
The ``T deserialize(ConsumerRecord<byte[], byte[]> record)`` method gets called for each Kafka message, passing the key, value, and metadata from Kafka.

.. literalinclude:: ../../../src/main/java/com/ververica/statefun/docs/io/kafka/UserDeserializer.java
    :language: java
    :lines: 16-

Kafka Egress Spec
=================

A ``KafkaEgressBuilder`` declares an egress spec for writing data out to a Kafka cluster.

It accepts the following arguments: 

1) The egress identifier associated with this egress
2) The address of the bootstrap servers
3) A ``KafkaEgressSerializer`` for serializing data into Kafka
4) The fault tolerance semantic
5) Properties for the Kafka producer

.. literalinclude:: ../../../src/main/java/com/ververica/statefun/docs/io/kafka/EgressSpecs.java
    :language: java
    :lines: 16-

Please refer to the Kafka `producer configuration <https://docs.confluent.io/current/installation/configuration/producer-configs.html>`_ documentation for the full list of available properties.

Kafka Egress and Fault Tolerance
""""""""""""""""""""""""""""""""

With fault tolerance enabled, the Kafka egress can provide exactly-once delivery guarantees.
You can choose three different modes of operating based through the ``KafkaEgressBuilder``.

* ``KafkaEgressBuilder#withNoProducerSemantics``: Nothing is guaranteed. Produced records can be lost or duplicated.
* ``KafkaEgressBuilder#withAtLeastOnceProducerSemantics``: Stateful Functions will guarantee that nor records will be lost but they can be duplicated.
* ``KafkaEgressBuilder#withExactlyOnceProducerSemantics``: Stateful Functions uses Kafka transactions to provide exactly-once semantics.

Kafka Serializer
""""""""""""""""

The Kafka egress needs to know how to turn Java objects into binary data.
The ``KafkaEgressSerializer`` allows users to specify such a schema.
The ``ProducerRecord<byte[], byte[]> serialize(T out)`` method gets called for each message, allowing users to set a key, value, and other metadata.

.. literalinclude:: ../../../src/main/java/com/ververica/statefun/docs/io/kafka/UserSerializer.java
    :language: java
    :lines: 16-
