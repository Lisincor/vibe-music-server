# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Vibe Music Server** is a Spring Boot 3 backend API service for a music platform. It serves both the music client and admin dashboard, providing user authentication, content management, file storage, and caching.

## Build Commands

```bash
# Build (skip tests)
mvn clean package -DskipTests

# Run in development mode (with hot reload)
mvn spring-boot:run

# Run tests
mvn test

# Run single test
mvn test -Dtest=ClassName#methodName
```

## Architecture

### MVC Pattern (Controller → Service → Mapper)

- **Controller** (`src/main/java/.../controller/`): Handles HTTP requests, parameter validation, returns `Result<T>`
- **Service** (`src/main/java/.../service/`): Contains business logic, transaction management
- **Mapper** (`src/main/java/.../mapper/`): Extends MyBatis-Plus `BaseMapper<T>` for CRUD operations

### Authentication Flow

1. Client sends credentials → `UserController.login()`
2. `JwtUtil.generateToken()` creates JWT with user claims
3. Token stored in Redis with 6-hour expiration
4. Subsequent requests include `Authorization: Bearer <token>` header
5. `LoginInterceptor` validates token via Redis and checks role permissions via `RolePermissionManager`
6. Claims stored in `ThreadLocalUtil` for request-scoped access

### File Storage

- **MinIO** handles all file uploads (avatars, covers, audio files)
- `MinioServiceImpl` provides `upload()` and `getFileUrl()` methods
- Bucket name: `vibe-music-data` (configured in `application.yml`)

### Data Model Structure

- `model/entity/`: JPA-style entities mapped to DB tables (prefix: `tb_`)
- `model/dto/`: Request DTOs for receiving client data
- `model/vo/`: Response VOs for returning safe data to clients

### Uniform Response Format

All APIs return `Result<T>`:
```json
{"code": 0, "message": "操作成功", "data": {...}}
```
- `code: 0` = success, `code: 1` = failure

### API Documentation

Knife4j (Swagger) is enabled. Access via `/doc.html` when the server is running.

## Configuration

Key settings in `src/main/resources/application.yml`:
- `spring.datasource`: MySQL connection (database: `vibe_music`)
- `spring.data.redis`: Redis connection (database 1)
- `minio`: MinIO endpoint, credentials, bucket name
- `role-path-permissions`: Maps roles to allowed URL patterns

## Key Files

- `VibeMusicServerApplication.java`: Application entry point
- `WebConfig.java`: Registers `LoginInterceptor` and excluded paths
- `LoginInterceptor.java`: JWT validation and role-based access control
- `JwtUtil.java`: Token generation and parsing (HMAC256, 6-hour expiry)
- `Result.java`: Unified response wrapper

## Database

Initialize using `sql/vibe_music.sql`. Tables use `tb_` prefix with auto-mapping to entity classes.

## Dependencies

- Spring Boot 3.3.7
- MyBatis-Plus 3.5.9
- Druid connection pool
- JWT (java-jwt 4.4.0)
- MinIO 8.5.9