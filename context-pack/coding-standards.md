# **Coding Standards: Calc-Service**

---

## **1. Naming Conventions**

### **Classes**

* **Entity:** PascalCase, singular
  `CalculationHistory`

* **DTO:** PascalCase + `DTO` suffix
  `CalculationRequestDTO`, `CalculationResultDTO`

* **Services:** PascalCase + `Service`
  `ArithmeticService`, `AdvancedMathService`, `HistoryService`

* **Controllers:** PascalCase + `Controller`
  `MathController`

* **Repositories:** PascalCase + `Repository`
  `HistoryRepository`

* **Converters:** PascalCase + `Converter`

---

### **Methods**

* camelCase with action verbs
  `performCalculation`, `calculatePower`, `fetchHistory`

* Repository queries must follow Spring conventions
  `findByOperationType`, `findByTimestampAfter`

* Boolean checks must start with `is` or `has`
  `isZero`, `hasPrecisionLoss`

---

### **Variables & Fields**

* DTO fields: **public camelCase** (legacy compatibility)
* DTOs are serialization models only and must not contain business logic.

Example:

```java
public class CalculationRequestDTO {
    public Double operandOne;
    public Double operandTwo;
    public String operation;
}
```

* Entity fields: package-private camelCase with explicit column mapping

---

## **2. Code Structure & Layering**

### **Package Layout**

Lowercase packages are preferred while preserving legacy alignment:

```
com.mihneacristian.calc_service
  ├── controller
  ├── service
  ├── repository
  ├── entity
  ├── dto
  └── converter
```

---

### **Component Rules**

**Controllers**

* Define HTTP contract only
* No business logic
* JSON enforced

**Services**

* Contain deterministic computation logic
* Persistence orchestration allowed
* `@Transactional` only on persistence methods

**Repositories**

* Extend `JpaRepository`
* No custom SQL unless justified

---

### **Dependency Injection**

* Constructor injection is standard
* Field injection allowed only for legacy parity

---

## **3. Error Handling**

### **Service Layer**

* Throw domain runtime exceptions
* No HTTP semantics

Example:

```java
if (operandTwo == 0) {
    throw new IllegalArgumentException("Cannot divide by zero");
}
```

---

### **Controller Layer**

* Controllers should not manually catch service exceptions

* Use a global handler:

* `@ControllerAdvice`

* Maps domain exceptions to HTTP responses

This ensures consistency and removes boilerplate.

---

## **4. Numerical & Persistence Standards**

### **Precision**

* Use `BigDecimal` internally
* Convert to `Double` at API boundary

### **Database Naming**

* Tables: snake_case
* IDs: `Long` with identity strategy

Persistence remains optional and must not affect computation flow.

---

## **5. Operation Extensibility Rules**

When adding a new math operation:

1. Extend operation enum
2. Implement logic in the appropriate service
3. Keep DTO contract backward compatible
4. Add deterministic unit tests

No controller changes should be required beyond enum support.

---

## **6. Testing Standards**

### **Unit Tests**

* Focus on deterministic math correctness
* Edge cases mandatory (zero, precision, overflow)

Naming:

```
<ClassName>Test
```

---

### **Integration Tests (Optional)**

When history enabled:

* Repository slice tests
* Flyway migration validation
* H2-backed persistence verification

---

**Document Version:** 1.1
**Scope:** Defines implementation conventions complementing architecture and technical policy.