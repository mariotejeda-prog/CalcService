# **Architecture Standards: Math Calculation Microservice**

## `Spring-Calc-Service`

---

## **1. SYSTEM ARCHITECTURE**

**Application Type:** Stateless Spring Boot microservice
**Service Role:** Deterministic math computation with optional calculation history persistence.

### **Domain Boundaries**

The service follows a **vertical slice architecture** organized by capability:

* **Arithmetic Domain**

  * Basic operations: addition, subtraction, multiplication, division
* **Advanced Math Domain**

  * Higher-order operations: powers, roots, trigonometry
* **History Domain (Optional)**

  * Persists calculation metadata for audit and traceability
  * Uses embedded H2 via repository abstraction

### **Communication Flow**

* **External → System:** HTTP/REST controllers
* **Controller → Service:** Dependency injection
* **Service → Repository:** Spring Data JPA (only when history enabled)
* **Payload format:** JSON

The service does not participate in distributed transactions and remains functionally independent.

---

## **2. LAYERED DESIGN**

The system enforces a strict layered structure:

1. **Presentation Layer**

   * REST controllers
   * Request validation
   * HTTP error mapping

2. **Business Layer**

   * Deterministic computation logic
   * Transaction boundary for persistence operations
   * No framework leakage into domain logic

3. **Persistence Layer (Optional)**

   * Repository interfaces extending `JpaRepository`
   * Isolated from computation logic

4. **Transfer & Conversion**

   * DTOs for all external contracts
   * Explicit converters between entities and DTOs

**Rule:** Controllers never access repositories directly.

---

## **3. CODING & NAMING CONVENTIONS**

Naming follows service-oriented clarity rather than entity-driven modeling.

### **Class Naming**

* Entity: `CalculationHistory`
* DTOs: `CalculationRequestDTO`, `CalculationResultDTO`
* Services: `ArithmeticService`, `AdvancedMathService`, `HistoryService`
* Controller: `MathController`
* Converters: `CalculationConverter`

### **Methods & Variables**

* camelCase methods (e.g., `performCalculation`, `calculatePower`)
* DTO fields remain public camelCase to preserve legacy interoperability patterns

Example:

```java
public class CalculationResultDTO {
    public Double resultValue;
    public String operationType;
    public String timestamp;
}
```

---

## **4. DATA ARCHITECTURE**

### **State Model**

* Core computation is stateless
* History persistence is optional and append-only
* No user session data retained

### **Persistence Model**

* Embedded H2 file database used only for lightweight history storage
* Persistence isolated behind repository abstraction to allow future database substitution without service refactor

### **Transactions**

* Only history write operations are transactional
* Computation logic must remain side-effect free

---

## **5. NUMERICAL PRECISION STRATEGY**

* Internal math operations should use **BigDecimal** where precision matters
* DTOs may expose **Double** for ecosystem compatibility
* Conversion boundaries must be explicit

This prevents floating-point drift while keeping API contracts simple.

---

## **6. ERROR HANDLING ARCHITECTURE**

Error responsibility is layered:

* **Domain/Service layer**

  * Throws runtime domain exceptions
  * No HTTP semantics

* **Controller layer**

  * Maps domain failures to HTTP status via `ResponseStatusException`

Example categories:

* Validation errors → 400
* Unsupported operation → 422
* Unexpected failure → 500

---

## **7. EXTENSIBILITY PRINCIPLES**

The service is designed for incremental capability growth.

### **Operation Extension**

New math operations must:

* Be implemented in the appropriate domain service
* Extend the operation enum
* Remain backward compatible with existing DTO contract

### **Persistence Evolution**

History persistence is intentionally decoupled so the service can evolve from:

* No persistence
* Embedded H2
* Managed relational database

without breaking computation APIs.

---

## **8. API DESIGN PRINCIPLES**

* Single responsibility endpoints
* Deterministic responses
* Idempotent read operations
* Calculation endpoint treated as pure function with optional side-effect logging

Controllers must remain orchestration layers, not business logic containers.

---

## **9. NON-FUNCTIONAL ARCHITECTURE DECISIONS**

* Service remains horizontally scalable because computation is stateless
* Embedded H2 does not participate in clustering and is treated as local auxiliary storage
* Failure of persistence must not block calculation response (graceful degradation allowed)

---

**Document Version:** 1.1
**Relationship:** Complements Technical Policy (environment/config) by defining structural and design standards.