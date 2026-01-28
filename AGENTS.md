# AGENTS.md - AI Agent Guidelines for AutoMQ

## Project Overview

AutoMQ is a cloud-native, Apache Kafka-compatible streaming platform that decouples storage to S3 and EBS for cost efficiency and scalability. It is a fork of Apache Kafka with the storage layer replaced by a cloud-first streaming storage engine called S3Stream.

- **Language**: Java
- **Build System**: Gradle (primary), Maven (s3stream module)
- **Version**: 1.6.3
- **License**: Apache 2.0

## Repository Structure

```
automq/
├── s3stream/           # Core cloud storage engine (S3/EBS) - Maven build
├── core/               # Kafka broker core
├── clients/            # Kafka client libraries
├── streams/            # Kafka Streams
├── connect/            # Kafka Connect
├── raft/               # KRaft consensus
├── metadata/           # Metadata management
├── server/             # Server components
├── server-common/      # Shared server utilities
├── storage/            # Storage abstractions
├── group-coordinator/  # Consumer group coordination
├── transaction-coordinator/ # Transaction management
├── automq-shell/       # AutoMQ CLI tools
├── automq-metrics/     # Metrics collection
├── automq-log-uploader/ # Log upload utilities
├── bin/                # Shell scripts for running Kafka/AutoMQ
├── config/             # Configuration files
├── docker/             # Docker build files
├── tests/              # Integration tests
├── jmh-benchmarks/     # Performance benchmarks
└── docs/               # Documentation
```

## Build Commands

```bash
# Full build
./gradlew build

# Build without tests
./gradlew build -x test

# Run specific tests
./gradlew :core:test
./gradlew :clients:test

# Build s3stream (Maven)
cd s3stream && mvn clean install

# Generate IDE project files
./gradlew idea
./gradlew eclipse
```

## Key Architecture Concepts

1. **S3Stream**: The core storage engine that replaces Kafka's local disk storage with cloud object storage (S3) and block storage (EBS for WAL)

2. **Stateless Brokers**: Unlike traditional Kafka, AutoMQ brokers are stateless, enabling rapid scaling

3. **Shared Storage**: Uses a storage-compute separation architecture with shared streaming storage

4. **100% Kafka Compatible**: Maintains full compatibility with Apache Kafka APIs and protocols

## Code Style & Conventions

- Follow existing Apache Kafka code style
- Use checkstyle configuration in `checkstyle/` directory
- Java code should be compatible with the project's target Java version
- Follow the contributing guide in `CONTRIBUTING_GUIDE.md`

## Testing

- Unit tests are located alongside source code in `src/test/`
- Integration tests are in the `tests/` directory
- Use JMH benchmarks in `jmh-benchmarks/` for performance testing

## Important Files

- `build.gradle` - Main build configuration
- `settings.gradle` - Gradle project settings
- `gradle.properties` - Build properties
- `s3stream/pom.xml` - Maven config for s3stream module
- `config/` - Sample configuration files

## Documentation

- Main docs: `docs/` directory
- Module-specific READMEs in each subdirectory
- External docs: https://docs.automq.com

## Common Tasks

### Adding a new feature
1. Check if changes affect s3stream (Maven) or other modules (Gradle)
2. Follow Kafka's existing patterns for the component being modified
3. Add appropriate tests
4. Update documentation if needed

### Debugging storage issues
- S3Stream is the key module for storage-related issues
- Check `s3stream/src/` for storage engine implementation

### Configuration changes
- Broker configs follow Kafka conventions
- AutoMQ-specific configs are documented in the docs
