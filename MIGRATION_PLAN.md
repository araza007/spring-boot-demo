# Migration Plan: Java 17 & Spring Boot 3 Upgrade

This document outlines a comprehensive migration plan for upgrading the `spring-boot-demo` repository from Java 11 and Spring Boot 2.7.5 to Java 17 and Spring Boot 3.x.

**Reference Documentation:** [Spring Boot 3.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.0-Migration-Guide)

---

## Pre-Migration Checklist

Before starting the migration, ensure the following prerequisites are met:

- [ ] All tests pass on the current Java 11 / Spring Boot 2.7.5 configuration
- [ ] Create a dedicated branch for the migration work
- [ ] Backup or tag the current working state
- [ ] Review the official Spring Boot 3.0 Migration Guide
- [ ] Ensure all team members are aware of the migration timeline
- [ ] Verify that your IDE supports Java 17 (IntelliJ IDEA 2021.2+, Eclipse 2021-09+)
- [ ] Review any custom configurations or extensions that may be affected

---

## Step-by-Step Migration Order

### Step 1: Java Version Upgrade

**Priority:** High  
**Risk Level:** Medium  
**Estimated Effort:** Low

#### 1.1 Update `pom.xml` Java Version Property

**File:** `pom.xml` (line 18)

**Current Configuration:**
```xml
<java.version>11</java.version>
```

**Target Configuration:**
```xml
<java.version>17</java.version>
```

#### 1.2 Update GitHub Actions Workflow

**File:** `.github/workflows/build.yml` (lines 16-19)

**Current Configuration:**
```yaml
- name: Set up JDK 11
  uses: actions/setup-java@v1
  with:
    java-version: 11
```

**Target Configuration:**
```yaml
- name: Set up JDK 17
  uses: actions/setup-java@v3
  with:
    java-version: '17'
    distribution: 'temurin'
```

**Considerations:**
- Update `actions/setup-java` from v1 to v3 for better Java 17 support
- Specify the `distribution` parameter (recommended: `temurin` for Eclipse Adoptium builds)
- Verify any other CI/CD workflows (e.g., `.circleci/config.yml`, `docker-image.yml`) also use Java 17

---

### Step 2: Spring Boot Parent Version Upgrade

**Priority:** High  
**Risk Level:** High  
**Estimated Effort:** Medium

**File:** `pom.xml` (lines 6-9)

**Current Configuration:**
```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.7.5</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>
```

**Target Configuration:**
```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.2.2</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>
```

**Considerations:**
- Spring Boot 3.2.2 is the latest stable version as of this writing
- Spring Boot 3.x requires Java 17 as the minimum version
- This change will cascade to all managed dependencies

---

### Step 3: Jakarta EE Migration (javax to jakarta)

**Priority:** Critical  
**Risk Level:** High  
**Estimated Effort:** High

Spring Boot 3 migrates from Java EE to Jakarta EE 9+, which requires changing all `javax.*` package imports to `jakarta.*`. This is one of the most significant breaking changes.

#### 3.1 Validation Annotations Migration

Replace all `javax.validation.*` imports with `jakarta.validation.*`.

**Files Affected:**

| File | Lines | Current Import | Target Import |
|------|-------|----------------|---------------|
| `src/main/java/com/springpageable/dto/DateRequestDTO.java` | 13-15 | `javax.validation.Valid`, `javax.validation.constraints.Future`, `javax.validation.constraints.NotNull` | `jakarta.validation.Valid`, `jakarta.validation.constraints.Future`, `jakarta.validation.constraints.NotNull` |
| `src/main/java/com/springpageable/dto/FutureDeviceDTO.java` | 6-8 | `javax.validation.constraints.NotBlank`, `NotNull`, `Positive` | `jakarta.validation.constraints.NotBlank`, `NotNull`, `Positive` |
| `src/main/java/com/springpageable/controller/ReadingJsonController.java` | 17-19 | `javax.validation.constraints.Max`, `NotNull`, `Positive` | `jakarta.validation.constraints.Max`, `NotNull`, `Positive` |
| `src/main/java/com/springpageable/controller/DeviceController.java` | 23-24 | `javax.validation.Valid`, `javax.validation.constraints.Positive` | `jakarta.validation.Valid`, `jakarta.validation.constraints.Positive` |
| `src/main/java/com/springpageable/controller/DateController.java` | 14 | `javax.validation.Valid` | `jakarta.validation.Valid` |
| `src/main/java/com/springpageable/validator/CompareDateClassAnnotationValidator.java` | 5-6 | `javax.validation.ConstraintValidator`, `ConstraintValidatorContext` | `jakarta.validation.ConstraintValidator`, `ConstraintValidatorContext` |
| `src/main/java/com/springpageable/validator/DateFieldValidator.java` | 3-4 | `javax.validation.ConstraintValidator`, `ConstraintValidatorContext` | `jakarta.validation.ConstraintValidator`, `ConstraintValidatorContext` |
| `src/main/java/com/springpageable/validator/CompareDateClassAnnotation.java` | 3-4 | `javax.validation.Constraint`, `Payload` | `jakarta.validation.Constraint`, `Payload` |
| `src/main/java/com/springpageable/validator/DateFieldAnnotation.java` | 3-4 | `javax.validation.Constraint`, `Payload` | `jakarta.validation.Constraint`, `Payload` |
| `src/main/java/com/springpageable/advice/GlobalExceptionHandler.java` | 20 | `javax.validation.ConstraintViolationException` | `jakarta.validation.ConstraintViolationException` |

#### 3.2 JPA/Persistence Annotations Migration

Replace all `javax.persistence.*` imports with `jakarta.persistence.*`.

**Files Affected:**

| File | Lines | Current Import | Target Import |
|------|-------|----------------|---------------|
| `src/main/java/com/springpageable/model/Auditable.java` | 14-16 | `javax.persistence.Column`, `EntityListeners`, `MappedSuperclass` | `jakarta.persistence.Column`, `EntityListeners`, `MappedSuperclass` |
| `src/main/java/com/springpageable/model/FutureDevice.java` | 5-8 | `javax.persistence.Entity`, `GeneratedValue`, `GenerationType`, `Id` | `jakarta.persistence.Entity`, `GeneratedValue`, `GenerationType`, `Id` |
| `src/main/java/com/springpageable/model/User.java` | 8 | `javax.persistence.*` | `jakarta.persistence.*` |
| `src/main/java/com/springpageable/repository/custom/UserRepositoryCriteriaImpl.java` | 10-16 | `javax.persistence.EntityManager`, `PersistenceContext`, `TypedQuery`, `criteria.*` | `jakarta.persistence.EntityManager`, `PersistenceContext`, `TypedQuery`, `criteria.*` |

#### 3.3 Test Files Migration

**Files Affected:**

| File | Lines | Current Import | Target Import |
|------|-------|----------------|---------------|
| `src/test/java/com/device/repository/UserRepositoryCriteriaImplTest.java` | 16-18 | `javax.persistence.EntityManager`, `TypedQuery`, `criteria.*` | `jakarta.persistence.EntityManager`, `TypedQuery`, `criteria.*` |
| `src/test/java/com/device/validator/DateFieldValidatorTest.java` | 10 | `javax.validation.ConstraintValidatorContext` | `jakarta.validation.ConstraintValidatorContext` |
| `src/test/java/com/device/validator/CompareDateClassAnnotationValidatorTest.java` | 11 | `javax.validation.ConstraintValidatorContext` | `jakarta.validation.ConstraintValidatorContext` |

#### 3.4 Servlet API Dependency Update

**File:** `pom.xml` (lines 47-50)

**Current Configuration:**
```xml
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <scope>provided</scope>
</dependency>
```

**Target Configuration:**
```xml
<dependency>
    <groupId>jakarta.servlet</groupId>
    <artifactId>jakarta.servlet-api</artifactId>
    <scope>provided</scope>
</dependency>
```

**Considerations:**
- The version will be managed by Spring Boot 3's dependency management
- If you have any direct servlet code, update those imports as well

---

### Step 4: SpringDoc OpenAPI Upgrade

**Priority:** High  
**Risk Level:** Medium  
**Estimated Effort:** Low

**File:** `pom.xml` (lines 42-45)

**Current Configuration:**
```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-ui</artifactId>
    <version>1.6.12</version>
</dependency>
```

**Target Configuration:**
```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.3.0</version>
</dependency>
```

**Considerations:**
- The artifact ID changes from `springdoc-openapi-ui` to `springdoc-openapi-starter-webmvc-ui`
- Version 2.x is required for Spring Boot 3 compatibility
- Swagger UI will still be available at `/swagger-ui.html`
- Review `application.yml` SpringDoc configuration for any deprecated properties

---

### Step 5: Spring Data Commons Version Management

**Priority:** Medium  
**Risk Level:** Low  
**Estimated Effort:** Low

**File:** `pom.xml` (lines 70-73)

**Current Configuration:**
```xml
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-commons</artifactId>
    <version>2.7.5</version>
</dependency>
```

**Target Configuration:**
```xml
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-commons</artifactId>
</dependency>
```

**Considerations:**
- Remove the explicit version declaration to use Spring Boot 3's managed version
- Spring Boot 3.2.x uses Spring Data 2023.1.x (Spring Data Commons 3.2.x)
- This ensures version compatibility across all Spring Data modules

---

### Step 6: Hibernate Dialect Configuration

**Priority:** Medium  
**Risk Level:** Low  
**Estimated Effort:** Low

**File:** `src/main/resources/application.yml` (line 20)

**Current Configuration:**
```yaml
spring:
  jpa:
    properties:
      hibernate:
        dialect: org.hibernate.dialect.PostgreSQLDialect
```

**Target Configuration:**
```yaml
spring:
  jpa:
    properties:
      hibernate:
        # dialect property can be removed - Hibernate 6 auto-detects the dialect
        # If explicit dialect is needed, use:
        # dialect: org.hibernate.dialect.PostgreSQLDialect
```

**Considerations:**
- Hibernate 6 (used by Spring Boot 3) has improved dialect auto-detection
- The explicit dialect configuration can typically be removed
- If you need to specify a dialect explicitly, the class name remains the same but the behavior may differ
- Test database operations thoroughly after migration
- Review Hibernate 6 migration guide for any entity mapping changes

---

### Step 7: Actuator Configuration Review

**Priority:** Low  
**Risk Level:** Low  
**Estimated Effort:** Low

**File:** `src/main/resources/application.yml` (lines 61-83)

**Current Configuration:**
```yaml
management:
  info:
    git:
      enabled: true
      mode: full
  endpoints:
    enabled-by-default: false
    jmx:
      exposure:
        include: "health,info,metrics,env"
    web:
      exposure:
        include: "health,info"
  endpoint:
    info:
      enabled: true
    health:
      enabled: true
    metrics:
      enabled: true
    env:
      enabled: true
  profiles.active: local
```

**Changes Required:**
- The current configuration is largely compatible with Spring Boot 3
- No major property changes are required for this configuration

**Considerations:**
- Review the `/actuator/info` endpoint output after migration
- The `management.info.git.mode` property remains valid
- Health endpoint groups and probes have enhanced features in Spring Boot 3
- Consider enabling the new `management.endpoint.health.probes.enabled` for Kubernetes deployments

---

### Step 8: Git Commit ID Plugin Compatibility

**Priority:** Low  
**Risk Level:** Low  
**Estimated Effort:** Low

**File:** `pom.xml` (lines 116-118)

**Current Configuration:**
```xml
<plugin>
    <groupId>io.github.git-commit-id</groupId>
    <artifactId>git-commit-id-maven-plugin</artifactId>
    <version>5.0.0</version>
    ...
</plugin>
```

**Target Configuration:**
```xml
<plugin>
    <groupId>io.github.git-commit-id</groupId>
    <artifactId>git-commit-id-maven-plugin</artifactId>
    <version>7.0.0</version>
    ...
</plugin>
```

**Considerations:**
- Version 5.0.0 should work with Spring Boot 3, but updating to 7.0.0 is recommended for full Java 17 support
- The plugin configuration remains the same

**Custom GitInfoContributor Verification:**

**File:** `src/main/java/com/springpageable/configuration/HistoryLinkProvidingGitInfoContributor.java` (line 12)

The custom `HistoryLinkProvidingGitInfoContributor` class extends `GitInfoContributor` from Spring Boot Actuator. This class should continue to work without changes as:
- The `GitInfoContributor` class API remains stable in Spring Boot 3
- The `GitProperties` class is unchanged
- The `@Component`, `@Autowired`, and `@Value` annotations are Spring Framework annotations (not Jakarta EE)

**Verification Steps:**
1. Compile the application after migration
2. Start the application and verify `/actuator/info` endpoint returns git information with the history link

---

### Step 9: MapStruct Version Update

**Priority:** Medium  
**Risk Level:** Low  
**Estimated Effort:** Low

#### 9.1 MapStruct Dependency

**File:** `pom.xml` (lines 88-91)

**Current Configuration:**
```xml
<dependency>
    <groupId>org.mapstruct</groupId>
    <artifactId>mapstruct</artifactId>
    <version>1.5.3.Final</version>
</dependency>
```

**Target Configuration:**
```xml
<dependency>
    <groupId>org.mapstruct</groupId>
    <artifactId>mapstruct</artifactId>
    <version>1.5.5.Final</version>
</dependency>
```

#### 9.2 MapStruct Processor in Compiler Plugin

**File:** `pom.xml` (lines 149-151)

**Current Configuration:**
```xml
<path>
    <groupId>org.mapstruct</groupId>
    <artifactId>mapstruct-processor</artifactId>
    <version>1.4.2.Final</version>
</path>
```

**Target Configuration:**
```xml
<path>
    <groupId>org.mapstruct</groupId>
    <artifactId>mapstruct-processor</artifactId>
    <version>1.5.5.Final</version>
</path>
```

**Considerations:**
- MapStruct 1.5.5.Final has full support for Java 17 and Jakarta EE
- Ensure both the dependency and processor versions match
- The current configuration has a version mismatch (1.5.3.Final vs 1.4.2.Final) which should be corrected

---

### Step 10: Jackson Serialization Compatibility

**Priority:** Low  
**Risk Level:** Low  
**Estimated Effort:** Low

**File:** `src/main/java/com/springpageable/dto/DateRequestDTO.java` (lines 4-6)

**Current Configuration:**
```java
import com.fasterxml.jackson.databind.annotation.JsonDeserialize;
import com.fasterxml.jackson.databind.annotation.JsonSerialize;
import com.fasterxml.jackson.datatype.jsr310.deser.LocalDateDeserializer;
import com.fasterxml.jackson.datatype.jsr310.ser.LocalDateSerializer;
```

**Considerations:**
- Spring Boot 3.2.x uses Jackson 2.15.x
- The `LocalDateSerializer` and `LocalDateDeserializer` classes remain in the same package
- The `jackson-datatype-jsr310` module is auto-configured by Spring Boot
- No code changes are required for Jackson serialization

**Verification Steps:**
1. Test date serialization/deserialization endpoints after migration
2. Verify JSON output format matches expected patterns
3. Check for any deprecation warnings related to Jackson

---

## Additional Considerations

### Flyway Compatibility

**File:** `pom.xml` - Flyway dependency

Spring Boot 3.2.x uses Flyway 9.x or 10.x. Verify that:
- Existing migration scripts are compatible
- The `flyway-core` dependency version is managed by Spring Boot
- PostgreSQL-specific Flyway features work correctly

Consider adding the PostgreSQL-specific Flyway module if needed:
```xml
<dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-database-postgresql</artifactId>
</dependency>
```

### Lombok Compatibility

Lombok is compatible with Java 17 and Spring Boot 3. Ensure you're using a recent version (1.18.30+) which is managed by Spring Boot's dependency management.

### JaCoCo Plugin

**File:** `pom.xml` (lines 178-180)

The current JaCoCo version (0.8.8) supports Java 17. Consider updating to 0.8.11 for the latest improvements:
```xml
<plugin>
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
    <version>0.8.11</version>
    ...
</plugin>
```

### Dockerfile Update

If the project uses Docker, update the base image to use Java 17:

**File:** `Dockerfile`

```dockerfile
FROM eclipse-temurin:17-jdk-alpine
# or
FROM eclipse-temurin:17-jre-alpine
```

---

## Migration Execution Order Summary

Execute the migration in the following order to minimize issues:

1. **Java Version** - Update `pom.xml` and CI/CD workflows
2. **Spring Boot Parent** - Upgrade to 3.2.2
3. **Jakarta EE Migration** - Update all `javax.*` to `jakarta.*` imports
4. **SpringDoc OpenAPI** - Change artifact and update version
5. **Spring Data Commons** - Remove explicit version
6. **Hibernate Dialect** - Review and optionally remove explicit dialect
7. **MapStruct** - Update both dependency and processor versions
8. **Git Commit ID Plugin** - Update to version 7.0.0
9. **JaCoCo** - Update to version 0.8.11
10. **Verification** - Run all tests and verify application startup

---

## Post-Migration Verification Checklist

- [ ] Application compiles without errors
- [ ] All unit tests pass
- [ ] All integration tests pass
- [ ] Application starts successfully
- [ ] Swagger UI is accessible at `/swagger-ui.html`
- [ ] Actuator endpoints respond correctly (`/actuator/health`, `/actuator/info`)
- [ ] Database operations work correctly (CRUD operations)
- [ ] Flyway migrations run successfully
- [ ] Custom validators work as expected
- [ ] Date serialization/deserialization works correctly
- [ ] Git info is displayed in actuator info endpoint
- [ ] CI/CD pipeline passes with Java 17

---

## Rollback Plan

If critical issues are encountered during migration:

1. Revert to the pre-migration branch/tag
2. Document the specific issues encountered
3. Research solutions in the Spring Boot 3.0 Migration Guide
4. Address issues incrementally before attempting migration again

---

## Resources

- [Spring Boot 3.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.0-Migration-Guide)
- [Spring Boot 3.0 Release Notes](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.0-Release-Notes)
- [Hibernate 6 Migration Guide](https://github.com/hibernate/hibernate-orm/blob/6.0/migration-guide.adoc)
- [Jakarta EE 9 Migration](https://jakarta.ee/resources/jakarta-ee-9-migration-guide/)
- [SpringDoc OpenAPI 2.x Migration](https://springdoc.org/v2/)
- [MapStruct Documentation](https://mapstruct.org/documentation/stable/reference/html/)
