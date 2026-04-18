# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

All commands use the Gradle wrapper:

```bash
./gradlew build          # Full build + tests
./gradlew assemble       # Compile + package, skip tests
./gradlew bootRun        # Run the application locally
./gradlew test           # Run all tests
./gradlew test --tests "com.bandage.backstage.SomeTest"  # Run a single test class
./gradlew test --tests "com.bandage.backstage.SomeTest.methodName"  # Run a single test method
./gradlew clean          # Clean build outputs
```

## Project Context

`backstage`는 **Bandage** 프로덕트를 구성하는 서비스 중 하나다. 장기적으로 `../v1`(밴드 합주/공연 관리 API)과 하나의 프로덕트로 통합될 예정이며, 두 서비스는 동일한 PostgreSQL 데이터베이스 생태계를 공유한다.

### v1과의 차이 — backstage에 아직 없는 것들

v1(`../v1`)은 동일한 Kotlin + Spring Boot 기반이지만, backstage에는 아직 적용되지 않은 구조·인프라가 존재한다:

| 항목 | v1 현황 | backstage 현황 |
|------|---------|----------------|
| **코드 포맷터** | Spotless + Ktlint (pre-commit hook 자동 실행) | 미설정 |
| **Spring Boot 버전** | 4.0.2 | 3.5.13 |
| **Kotlin 버전** | 2.2.21 | 1.9.25 |
| **Spring Modulith** | 적용 (모듈 경계 강제) | 미적용 |
| **QueryDSL** | 적용 (타입 세이프 커스텀 쿼리) | 미적용 |
| **Redis** | Refresh Token 저장 + 캐시 용도 | 미적용 |
| **Spring Security + JWT** | 완전 구현 (JJWT 0.12.6, stateless) | 미구현 |
| **Swagger / OpenAPI** | SpringDoc OpenAPI 3.0.1 (`/swagger-ui.html`) | 미적용 |
| **H2 테스트 DB** | `test` 프로필 시 H2 in-memory 사용 | 미설정 |
| **멀티 프로필 설정** | `local` / `dev` / `test` 분리, `.env` 활용 | 미설정 |
| **Spring Actuator** | 적용 (모니터링/헬스체크) | 미적용 |
| **이벤트 기반 처리** | 도메인 이벤트 + `@EnableAsync` | 미적용 |
| **UUIDv7** | uuid-creator 라이브러리로 일부 엔티티에 적용 | 미적용 |
| **소프트 딜리트** | `@SQLRestriction("deleted_at IS NULL")` 패턴 | 미적용 |
| **JPA Audit** | `CreatedBy` / `LastModifiedBy` 추적 | 미설정 |
| **커서 기반 페이지네이션** | `CursorResponse<T, ID>` 공통 구조 | 미구현 |
| **전역 응답 래퍼** | `ApiResponse<T>` (success, message, data, timestamp) | 미구현 |
| **전역 예외 처리** | `GlobalExceptionHandler` + `ErrorCode` enum | 미구현 |

v1의 설계 패턴(팩토리 메서드, `protected set`, Facade, 도메인 이벤트 등)은 backstage 구현 시 참고 기준이 된다.

## Architecture

**Spring Boot 3.x + Kotlin monolith** targeting Java 21.

- **Language:** Kotlin 1.9.25 with strict JSR-305 null safety (`-Xjsr305=strict`)
- **Framework:** Spring Web MVC + Spring Data JPA
- **Database:** PostgreSQL (runtime), Hibernate 6 ORM, HikariCP connection pool
- **Serialization:** Jackson with Kotlin module
- **Build:** Gradle 8 with wrapper (JDK 21 required via toolchain)

All application code lives under `com.bandage.backstage`. The JPA plugin auto-opens `@Entity`, `@MappedSuperclass`, and `@Embeddable` classes for Hibernate proxying — required when using Kotlin `final` classes.

## CI/CD

GitHub Actions runs on pull requests and pushes to `develop` (`./gradlew build`). JDK 21 (Temurin) is required. The workflow uses `GIT_USERNAME` and `GIT_TOKEN` secrets for authentication.

## Commit Convention

Always use this format for every commit message:

```
{type}: {summary}
- {detail 1}
- {detail 2}
- {detail n}

#{issue-number}
```

**Types**: `chore`, `feat`, `ai`, `test`, `refactor`, `fix`

- `{summary}` — concise description of the change
- bullet list — one line per meaningful change (omit if only one trivial change)
- `#{issue-number}` — **REQUIRED** when working on a branch tied to a GitHub issue (e.g. `feat/#12-practice-crud` → `#12`). Always place on its own line at the very end, after a blank line. Omit only if there is genuinely no related issue.

**Example:**
```
feat: 합주 생성 API 구현
- PracticeCreateRequest, PracticeResponse DTO 추가
- PracticeService.createPractice 구현
- POST /practices 엔드포인트 추가

#12

## Task Master AI Instructions
**Import Task Master's development workflow commands and guidelines, treat as if import is in the main CLAUDE.md file.**
@./.taskmaster/CLAUDE.md
