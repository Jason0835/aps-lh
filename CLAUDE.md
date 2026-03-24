# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Maven multi-module Java project for **硫化排程** (Vulcanization Scheduling) - an APS (Advanced Planning and Scheduling) system module.

### Module Structure

- **aps-lh-parent**: Parent POM module (packaging: pom)
- **aps-lh**: Main service module (packaging: jar) - 硫化服务模块 (Vulcanization Service Module)
- **aps-lh-api**: API module (packaging: jar) - Contains DTOs, enums, and Feign service interfaces

## Build Commands

### Build the entire project
```bash
mvn clean install
```

### Build without tests
```bash
mvn clean install -DskipTests
```

### Package for deployment (with class encryption)
```bash
mvn clean install -DskipTests
# Creates aps-lh-encrypted.jar with encrypted class files
```

### Deploy to Nexus
```bash
mvn clean deploy -DskipTests
```

## Development Commands

### Run a single test class
```bash
cd aps-lh
mvn test -Dtest=ClassName
```

### Run all tests in a module
```bash
cd aps-lh
mvn test
```

### Compile only
```bash
mvn clean compile
```

## Technology Stack

- **Java**: 8
- **Framework**: Spring Boot 2.x with Spring Cloud Alibaba
- **Service Discovery**: Nacos (default: 127.0.0.1:8848)
- **Database**: MySQL (via Druid connection pool)
- **ORM**: MyBatis
- **API Docs**: Swagger with swagger-bootstrap-ui (Knife4j-style, accessible at `/doc.html`)
- **Remote Calls**: OpenFeign
- **Distributed Tracing**: TLog
- **Utilities**: Hutool

## Architecture

### Package Structure (aps-lh module)

```
com.zlt.aps
├── lh/                    # Core vulcanization scheduling logic
│   ├── controller/        # REST API controllers
│   ├── service/           # Business logic interfaces
│   │   └── impl/          # Service implementations
│   ├── mapper/            # MyBatis data access layer
│   ├── handle/            # Business handlers (schedule result processing)
│   └── engine/            # Scheduling algorithm engine
├── sync/                  # Data synchronization
│   └── mapper/            # Sync data mappers
├── common/                # Common utilities
│   ├── CommonLogService
│   ├── CommonRedisService
│   ├── CommonUtils
│   ├── FactoryService
│   └── SyncDataLogsService
├── config/                # Configuration classes
├── constants/             # Constants
├── context/               # DTO contexts
├── entity/                # Domain entities
└── enums/                 # Enumerations
```

### API Module (aps-lh-api)

Contains shared types for inter-service communication:
- `domain/dto/` - Data Transfer Objects
- `domain/vo/` - Value Objects
- `enums/` - Shared enumerations
- `service/` - Feign client interfaces (ILh*RemoteService)

## Configuration

### Application Configuration

Main config: `aps-lh/src/main/resources/bootstrap.yml`

Key settings:
- **Server Port**: 9669
- **Nacos**: 127.0.0.1:8848 (configurable via `nacos.server-addr`)
- **Profile**: prod (changeable via `spring.profiles.active`)

### Nacos Shared Configs

The application loads these shared configurations from Nacos:
1. `application-aps-${env}.yml`
2. `druid_aps_${env}.yml` (database)
3. `system-frame-aps-${env}.yml`
4. `system-api-prefix-${env}.yml`
5. `zipkin-${env}.yml`
6. `sync-data-aps-${env}.yml`

### Environment-Specific Profiles

- `bootstrap-dev.yml` - Development environment
- `bootstrap-test.yml` - Test environment
- `bootstrap-prod.yml` - Production environment

## Running the Application

### Local Development
```bash
cd aps-lh
mvn spring-boot:run
```

Or run the main class: `com.zlt.aps.ApsLhApplication`

### Running the Encrypted JAR

The build produces an encrypted JAR (`aps-lh-encrypted.jar`) via the classfinal-maven-plugin:

```bash
# Without password (default)
java -javaagent:aps-lh-encrypted.jar -jar aps-lh-encrypted.jar

# With password (if configured)
java -javaagent:aps-lh-encrypted.jar='-pwd=PASSWORD' -jar aps-lh-encrypted.jar
```

## Maven Repository Configuration

The project uses a private Nexus repository:
- **Snapshots**: http://192.168.2.95:8081/nexus/content/repositories/snapshots/
- **Releases**: http://192.168.2.95:8081/nexus/content/repositories/releases/
- **Public Group**: http://192.168.2.95:8081/nexus/content/groups/public

## Key Dependencies

- `zlt-module-starter` - Framework starter
- `zlt-bill-common` / `zlt-busi-common` - Common utilities
- `aps-maindata` - Master data module
- `zlt-sync-data` - Data synchronization interface
- `aps-lh-api` - Internal API dependency
- `swagger-bootstrap-ui` - Swagger UI enhancement
- `tlog-all-spring-boot-starter` - Distributed logging
- `hutool-all` - Utility library

## Testing

Test files follow the Maven standard structure:
```
src/test/java/com/zlt/aps/
```

No tests are currently present in the codebase. When adding tests:
- Use JUnit 5
- Place tests in the corresponding package under `src/test/java`

## Code Style Notes

- Project follows standard Java conventions
- Refer to `doc/Java编程规范-代码规范.md` for coding standards (if available)
- MyBatis mappers use XML configuration (check `src/main/resources/mapper/`)
