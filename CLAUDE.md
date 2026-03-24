# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AutoMQ is a cloud-native fork of Apache Kafka that replaces the storage layer with S3Stream, a shared streaming storage library. It maintains 100% Kafka protocol compatibility while achieving stateless brokers through storage-compute separation.

## Build Commands

```bash
export JAVA_HOME=$(/usr/libexec/java_home -v 17)

# Build without tests
./gradlew jar -x test

# Build with tests
./gradlew build

# Build system test libraries (required before running integration tests)
./gradlew systemTestLibs

# Create release tarball
./gradlew releaseTarGz
```

## Testing

```bash
export JAVA_HOME=$(/usr/libexec/java_home -v 17)

# Run all tests for a module
./gradlew :core:test
./gradlew :s3stream:test

# Run a single test class
./gradlew :core:test --tests "kafka.server.KafkaServerTest"

# Run a single test method
./gradlew :core:test --tests "kafka.server.KafkaServerTest.testMethod"

# Run AutoMQ-specific S3 unit tests (tagged with @Tag("S3Unit"))
./gradlew :core:S3UnitTest
./gradlew :metadata:S3UnitTest
./gradlew :s3stream:test

# Run unit tests only (excludes integration tests)
./gradlew unitTest

# Run integration tests only (tagged with @Tag("integration"))
./gradlew integrationTest

# Re-run tests ignoring up-to-date checks
./gradlew test -Prerun-tests
```

## Code Quality

```bash
export JAVA_HOME=$(/usr/libexec/java_home -v 17)
# Run checkstyle
./gradlew checkstyleMain checkstyleTest

# Run spotbugs
./gradlew spotbugsMain spotbugsTest

# Run spotless (Java import ordering) - requires JDK 11 or 17
./gradlew spotlessJavaCheck
./gradlew spotlessApply  # to fix issues

# Full CI checks
./gradlew rat checkstyleMain checkstyleTest spotlessJavaCheck
```

## Architecture

### Key Modules

- **s3stream**: Core cloud storage library - the main AutoMQ innovation. Provides streaming storage on S3/EBS with APIs for append, fetch, and trim operations.
- **core**: Kafka broker implementation (Scala). Contains server logic, log management, and the integration point with S3Stream.
- **clients**: Kafka client libraries (producer, consumer, admin).
- **metadata**: KRaft metadata management.
- **server-common**: Shared server utilities.
- **storage**: Tiered storage APIs.
- **connect**: Kafka Connect runtime and connectors.
- **streams**: Kafka Streams processing library.

### S3Stream Architecture

S3Stream replaces Kafka's local storage with a cloud-native approach:
- **WAL (Write-Ahead Log)**: Uses EBS or S3 Express for low-latency writes
- **S3 Storage**: Primary data storage for durability and cost efficiency
- **Message Cache**: In-memory cache for hot data and prefetched cold data

Data flow: Write to WAL → Upload to S3 (near real-time) → Serve reads from cache or S3

### Configuration

AutoMQ-specific settings in `config/kraft/server.properties`:
- `elasticstream.enable=true`: Enables S3Stream storage
- `s3.data.buckets`: S3 bucket for data storage
- `s3.wal.path`: WAL storage location (S3 or local path)
- `s3.block.cache.size`: Block cache size for S3 reads

## Local Development

### Requirements
- JDK 17
- Scala 2.13

### Running Locally with LocalStack

1. Install LocalStack for local S3: `pip install localstack`
2. Create bucket: `aws s3api create-bucket --bucket ko3 --endpoint=http://127.0.0.1:4566`
3. Configure `config/kraft/server.properties`:
   ```
   s3.endpoint=http://127.0.0.1:4566
   s3.region=us-east-1
   s3.data.buckets=0@s3://ko3?region=us-east-1
   ```
4. Format storage: `bin/kafka-storage.sh format -t $(bin/kafka-storage.sh random-uuid) -c config/kraft/server.properties`
5. Set env vars: `KAFKA_S3_ACCESS_KEY=test KAFKA_S3_SECRET_KEY=test`
6. Start broker: Main class is `kafka.Kafka` with arg `config/kraft/server.properties`

### IDE Setup (IntelliJ)
- Main class: `core/src/main/scala/kafka/Kafka.scala`
- Classpath: `-cp kafka.core.main`
- VM options: `-Xmx1G -Xms1G -server -XX:+UseZGC -XX:MaxDirectMemorySize=2G -Dkafka.logs.dir=logs/ -Dlog4j.configuration=file:config/log4j.properties`
- Args: `config/kraft/server.properties`

## System Tests (Ducktape)

```bash
# Build required artifacts
./gradlew clean systemTestLibs

# Run all tests in Docker
bash tests/docker/run_tests.sh

# Run specific test file
TC_PATHS="tests/kafkatest/tests/client/pluggable_test.py" bash tests/docker/run_tests.sh

# Run specific test class
TC_PATHS="tests/kafkatest/tests/client/pluggable_test.py::PluggableConsumerTest" bash tests/docker/run_tests.sh
```
