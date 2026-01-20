# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ZPan is a self-hosted cloud disk server written in Go that enables direct client-to-cloud-storage connections, bypassing server bandwidth limitations. It supports all S3-compatible storage providers (AWS S3, Aliyun OSS, Tencent COS, MinIO, etc.).

## Build Commands

```bash
make mod              # Download dependencies
make lint             # Run golangci-lint (excludes errcheck)
make test             # Run tests with coverage
make build            # Build binary to build/bin/zpan
make swag             # Generate Swagger API documentation
make coverage-html    # Generate HTML coverage report
```

## Running the Server

```bash
./build/bin/zpan server                           # Default config at /etc/zpan/config.yml
./build/bin/zpan server --config ./config.yml     # Custom config path
./build/bin/zpan server --port 8222               # Custom port
```

## Quick Local Development

Use the docker-compose in `quickstart/` to spin up MinIO + ZPan:
```bash
docker-compose -f quickstart/docker-compose.yaml up
```

## Architecture

The project follows Clean Architecture with Google Wire for dependency injection:

```
cmd/                    # Cobra CLI commands (server, version)
internal/
├── app/
│   ├── api/            # HTTP handlers with Gin, Swagger annotations
│   ├── usecase/        # Business logic
│   │   ├── vfs/        # Virtual filesystem operations
│   │   ├── storage/    # Cloud storage management
│   │   ├── uploader/   # File upload handling
│   │   └── authz/      # Authorization logic
│   ├── repo/           # Repository layer (data access abstraction)
│   ├── dao/            # Database access objects
│   ├── entity/         # GORM database models
│   ├── model/          # DTOs
│   ├── docs/           # Generated Swagger docs
│   └── wire.go         # DI wiring
├── pkg/
│   ├── middleware/     # Auth, RBAC, installer middleware
│   ├── authed/         # Authentication utilities
│   └── provider/       # Cloud storage providers
└── mock/               # Mock implementations for testing
pkg/nos/                # NetEase Object Storage SDK (public)
web/                    # Embedded frontend assets
```

## Key Technologies

- **Web Framework**: Gin
- **ORM**: GORM (supports SQLite, MySQL, PostgreSQL, SQL Server)
- **DI**: Google Wire
- **Auth**: JWT + OAuth2, Open Policy Agent for RBAC
- **API Docs**: swaggo/swag (annotations in `internal/app/api/router.go`)
- **CLI**: Cobra + Viper for config

## Database

Supports multiple drivers via GORM. Default is SQLite. Configure via DSN in config file:
- SQLite: `zpan.db`
- MySQL: `user:password@tcp(127.0.0.1:3306)/zpan?parseTime=true`
- PostgreSQL: `postgres://user:pass@localhost:5432/zpan`

## API Documentation

Swagger annotations are in `internal/app/api/router.go`. Regenerate with `make swag`. Access at `/swagger/` endpoint when server is running.

## Testing

Tests use testify for assertions and go-sqlmock for database mocking. Mock implementations are in `internal/mock/`.

```bash
make test                    # Run all tests
go test ./internal/app/...   # Run specific package tests
go test -run TestName ./...  # Run single test
```

## Commit Convention

Use Conventional Commits format (feat:, fix:, docs:, etc.). PRs should target the master branch.
