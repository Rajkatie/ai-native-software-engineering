
# UC-001: Add New Employee

**Status:** Draft

## 1. Intent

Add a new employee to the Payroll System with the employee information and compensation arrangement appropriate to the employee's classification.

---

## 2. Trigger

The system receives an `AddEmp` transaction.

---

## 3. Input

The transaction contains:

* Employee ID
* Employee name
* Employee address
* Employee classification
* Compensation information required by that classification

### Hourly Employee

```text
AddEmp <EmpID> "<name>" "<address>" H <hourly-rate>
```

### Salaried Employee

```text
AddEmp <EmpID> "<name>" "<address>" S <monthly-salary>
```

### Commissioned Employee

```text
AddEmp <EmpID> "<name>" "<address>" C <monthly-salary> <commission-rate>
```

---

## 4. Business Rules

### BR-001 — Employee Classification

The employee must be created as exactly one of:

* Hourly
* Salaried
* Commissioned

### BR-002 — Hourly Compensation

An hourly employee must have an hourly rate.

### BR-003 — Salaried Compensation

A salaried employee must have a monthly salary.

### BR-004 — Commissioned Compensation

A commissioned employee must have:

* A monthly salary
* A commission rate

### BR-005 — Employee Identity

The employee is identified by the supplied Employee ID.

---

## 5. Successful Outcome

When a valid transaction is received:

1. A new employee record is created.
2. The supplied employee ID, name, and address are stored.
3. The employee classification is established.
4. The compensation information appropriate to the classification is stored.
5. The new employee becomes available to subsequent Payroll System operations.

---

## 6. Invalid Input

If the transaction is:

* Poorly structured
* Missing required information
* Invalid for the selected employee classification

the system must:

1. Report an error.
2. Take no action.
3. Leave the existing system state unchanged.

---

## 7. Acceptance Criteria

### AC-001 — Add Hourly Employee

**Given** a valid hourly employee transaction
**When** the transaction is processed
**Then** an hourly employee is created with the supplied hourly rate.

### AC-002 — Add Salaried Employee

**Given** a valid salaried employee transaction
**When** the transaction is processed
**Then** a salaried employee is created with the supplied monthly salary.

### AC-003 — Add Commissioned Employee

**Given** a valid commissioned employee transaction
**When** the transaction is processed
**Then** a commissioned employee is created with the supplied monthly salary and commission rate.

### AC-004 — Reject Malformed Transaction

**Given** a malformed `AddEmp` transaction
**When** the transaction is processed
**Then** an error is reported
**And** no employee is created or modified.

### AC-005 — Preserve Existing State

**Given** an invalid `AddEmp` transaction
**When** the transaction is processed
**Then** existing employee records remain unchanged.

---

## 8. Output

The requirements currently specify that invalid transactions produce an error message.

The exact success response and error-message format are **not yet specified**.

Implementation agents must not invent a user-facing response format without an approved specification.

---

## 9. Architectural Constraints

The implementation must comply with ADR-001.

In particular:

* The business rules must not depend on the transaction parser.
* The Use Case must not depend on a database implementation.
* The Use Case must not depend on a Controller or Presenter implementation.
* Infrastructure details must remain outside the core business rules.

The transaction format is an input representation and must be treated as an external concern.

---

## 10. Verification Criteria

The implementation is considered complete only when automated verification demonstrates:

* Valid hourly employee creation.
* Valid salaried employee creation.
* Valid commissioned employee creation.
* Correct classification assignment.
* Correct compensation assignment.
* Rejection of malformed transactions.
* No state change after rejected transactions.

Architectural verification must also confirm that the implementation does not violate the dependency rules established by ADR-001.

---

## 11. Traceability

**System Specification:** `specs/payroll/README.md`

**Use Case:** UC-001 — Add New Employee

**Architectural Decision:** ADR-001 — Architectural Foundation
