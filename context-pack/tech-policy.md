# **TECHNICAL POLICY DOCUMENT**

## Spring-Calc-Service

---

## **1. TECHNOLOGY STACK**

### **Languages**

* **Java 1.8** — Enforced in `pom.xml` (`<java.version>1.8</java.version>`).

### **Frameworks**

* **Spring Boot 2.2.7.RELEASE** — Parent dependency.
* **Spring Boot Starter Web** — REST API layer.

### **Required Dependencies**

* **Spring Boot Starter Test** — JUnit 5 integration (test scope).
* **Spring Boot DevTools** — Optional runtime auto-restart.
* **Maven 3.6.3** — Build tool.
* *(If History Persistence is enabled)*:

  * **H2 Database Engine** — Embedded relational database (runtime).
  * **Spring Data JPA / Hibernate** — ORM layer.
  * **Flyway Maven Plugin 6.4.1** — Schema migration.

---

## **2. SECURITY**

### **Authentication**

* None implemented — Service intended for internal microservice communication.
* All endpoints under `/api/v1/math/*` are accessible without authentication.

### **Secrets Management**

* **No hardcoded database credentials.**
* Embedded H2 uses default local credentials:

  * Username: `sa`
  * Password: empty
* Externalized configuration via environment variables is required if a non-embedded database is introduced.

### **Dependencies**

* No automated dependency scanning configured.
* Versions are fixed to maintain compatibility with the legacy ecosystem.

---

## **3. DATA MANAGEMENT**

### **Storage**

* **Primary store:** Embedded **H2 file database** (optional feature for calculation history).
* Database name: `calc_db`
* Connection URL:

```
jdbc:h2:./calc_db;DB_CLOSE_ON_EXIT=FALSE
```

* Database files are created relative to the application working directory.

### **Schema Management**

* Flyway manages schema evolution.
* Migration scripts located at:

```
src/main/resources/db/migration/
```

* Hibernate DDL auto generation disabled:

```
spring.jpa.hibernate.ddl-auto=none
```

### **Runtime Configuration**

Recommended Spring properties:

```
spring.datasource.url=jdbc:h2:./calc_db;DB_CLOSE_ON_EXIT=FALSE
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=
```

### **Privacy**

* Data stored in plain text (local persistence only).
* Service remains stateless — no user session storage.

---

## **4. DEPLOYMENT**

### **Environments**

#### **Local Development**

* Port: `8090`
* Server: Embedded Tomcat
* H2 Console may be enabled:

```
spring.h2.console.enabled=true
spring.h2.console.path=/h2-console
```

#### **Production**

* Default behavior remains **stateless**.
* H2 persistence is considered **non-durable and single-instance only**.
* If durable history is required, migration to a managed relational database (e.g., RDS/Postgres/MySQL) is mandated.

---

## **5. INFRASTRUCTURE**

* Packaging: Executable JAR via Spring Boot Maven Plugin.
* CI/CD: Manual Maven build and deployment.
* Filesystem persistence of H2 is not guaranteed across redeployments in cloud environments.

---

## **6. MONITORING**

### **Logging**

* Console logging via standard output.
* SQL logging enabled for persistence debugging:

```
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
```

### **Metrics**

* No metrics stack configured.
* Actuator integration optional for future observability.

---

## **7. API SPECIFICATIONS**

### **REST Standards**

* Base path: `/api/v1/math`
* Content type: `application/json`

### **Error Handling**

* `ResponseStatusException` maps domain errors to HTTP responses.

### **Data Transfer**

* DTO pattern enforced.
* Persistence entities must not be exposed directly.

Example mapping:

```
CalculationHistory → CalculationResultDTO
```

---

## **8. BUILD & PACKAGING**

* **Group ID:** `com.mihneacristian`
* **Artifact ID:** `calc_service`
* **Version:** `0.0.1-SNAPSHOT`
* **Java Version:** 1.8

---

**Document Version:** 1.1
**Alignment:** Legacy standards maintained while adopting embedded H2 for lightweight persistence.
