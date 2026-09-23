# Channel-Backend Gateway

A Spring Boot middleware service that simulates a common enterprise integration pattern: a gateway service sitting between a client-facing **channel** and a **backend** system, validating and transforming requests between the two.

![CI](https://github.com/ZhengLong9/channel-backend-gateway/actions/workflows/build.yml/badge.svg)
![Quality Gate Status](https://sonarcloud.io/api/project_badges/measure?project=ZhengLong9_channel-backend-gateway&metric=alert_status)

## Overview

Channel and backend systems rarely speak the same "language" — different field names, different validation rules, different data shapes. A middleware gateway sits in between and reconciles that gap, so neither side needs to know about the other.

This project implements that pattern for a balance inquiry use case:
- Accepts a simplified request from a channel
- Validates it (field presence, format)
- Attaches a correlation ID for request tracing
- Calls a backend system for the actual data
- Returns a clean, channel-friendly response

## Why this project exists

Built to get hands-on with CI/CD end to end — designing, building, and reasoning about every stage of a pipeline (build, test, static analysis, containerization, deployment) rather than just operating one someone else built.

## Tech stack

- **Java 21** / **Spring Boot 4.1**
- **Maven** (via the Maven Wrapper — no local Maven install required)
- **JUnit 5** for testing
- **SonarQube Cloud** for static analysis and quality gates
- **GitHub Actions** for CI/CD
- **Docker** & **Kubernetes** *(in progress — see Roadmap)*

## API

### `POST /api/v1/balance-inquiry`

**Request:**
```json
{
  "accountNumber": "1234567890",
  "channelId": "MOBILE"
}
```

**Response:**
```json
{
  "accountNumber": "1234567890",
  "balance": "1500.00",
  "currency": "SGD",
  "correlationId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890"
}
```

`accountNumber` must be exactly 10 digits. Invalid requests return a `400` with a structured error body:
```json
{
  "status": "VALIDATION_ERROR",
  "errors": { "accountNumber": "Account Number must be exactly 10 digits" }
}
```

## Running locally

Requires JDK 21.

```bash
./mvnw spring-boot:run
```

The service starts on `http://localhost:8080`. Test it with:
```bash
curl -X POST http://localhost:8080/api/v1/balance-inquiry \
  -H "Content-Type: application/json" \
  -d '{"accountNumber": "1234567890", "channelId": "MOBILE"}'
```

## Testing

```bash
./mvnw test
```

## CI/CD pipeline

Every pull request into `main` triggers a GitHub Actions workflow that builds the project, runs the full test suite, and runs a SonarQube Cloud static analysis scan, with the result posted as a check directly on the PR.

Planned: on merge to `main`, a second workflow builds and pushes a Docker image, then deploys it to a local Kubernetes cluster via a self-hosted runner.

## Roadmap

- [x] Core balance inquiry endpoint with request validation
- [x] Centralized error handling
- [x] Unit test coverage
- [x] CI pipeline: build, test, SonarQube Cloud scan on every PR
- [ ] Backend mock service
- [ ] Dockerize the service
- [ ] Kubernetes deployment manifests (local cluster)
- [ ] CD pipeline: build & push image, deploy via self-hosted runner
- [ ] Approval-gated deployment (GitHub Environments)

## License

MIT
