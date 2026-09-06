
# Payroll System Specification

## 1. Purpose

The Payroll System is a batch-oriented business system responsible for maintaining employee records and processing employee payments.

The system runs once each working day and calculates correct payments up to a specified date, including applicable deductions and the employee's selected payment method.

This Payroll System is the **software system used to demonstrate an AI-Native Software Development Lifecycle (SDLC)**.

The system will be developed using Clean Architecture principles as established in [ADR-001](../../docs/decisions/ADR-001-architectural-foundation.md).

---

## 2. System Overview

The system maintains employee records and supports different employee classifications, payment schedules, payment methods, and union-related deductions.

### 2.1 Employee Classifications

The system supports three employee classifications.

#### Hourly Employees

* Paid according to an hourly rate.
* Submit daily time cards containing the date and hours worked.
* Hours worked beyond eight hours in a single day are compensated at one and a half times the standard hourly rate.
* Paid every Friday.

#### Salaried Employees

* Receive a fixed monthly salary.
* Paid on the last working day of the month.

#### Commissioned Employees

* Receive a fixed monthly salary.
* Receive additional commission based on sales.
* Submit sales receipts containing the date and amount of each sale.
* Paid every other Friday.

---

## 3. Payment Methods

An employee can select how their paycheck is delivered.

The default payment method is **Hold**.

The system supports:

### Hold

The paycheck is held by the paymaster for employee pickup.

### Mail

The paycheck is mailed to an address selected by the employee.

### Direct Deposit

The payment is deposited directly into a specified bank account.

---

## 4. Union Membership and Deductions

The system supports employees who belong to a union.

### Union Dues

A weekly union dues rate is associated with the employee and deducted from the employee's pay.

### Service Charges

The union may submit service charges for individual members.

A service charge is deducted from the member's next paycheck.

---

# 5. Supported Transactions

The system processes transactions from an input stream.

The initial transaction types are:

| Transaction     | Purpose                                      |
| --------------- | -------------------------------------------- |
| `AddEmp`        | Add a new employee                           |
| `DelEmp`        | Delete an employee                           |
| `TimeCard`      | Post an hourly employee's time card          |
| `SalesReceipt`  | Post a commissioned employee's sales receipt |
| `ServiceCharge` | Post a union service charge                  |
| `ChgEmp`        | Change employee details                      |
| `Payday`        | Run payroll for a specified date             |

If a transaction is poorly structured or contains an invalid identifier, the system must report an error and take **no action**.

---

# 6. Use Cases

## UC-01: Add New Employee

Adds a new employee using:

* Employee ID
* Name
* Address
* Employee classification
* Compensation information

Supported classifications:

### Hourly

```text
AddEmp <EmpID> "<name>" "<address>" H <hourly-rate>
```

### Salaried

```text
AddEmp <EmpID> "<name>" "<address>" S <monthly-salary>
```

### Commissioned

```text
AddEmp <EmpID> "<name>" "<address>" C <monthly-salary> <commission-rate>
```

An invalid transaction must produce an error and must not modify the system.

---

## UC-02: Delete Employee

Deletes an employee using the employee ID.

```text
DelEmp <EmpID>
```

If the employee ID is invalid, the transaction is poorly structured, or the employee does not exist, the system must report an error and take no action.

---

## UC-03: Post Time Card

Creates and associates a time card with an employee.

```text
TimeCard <EmpID> <date> <hours>
```

The target employee must be classified as **Hourly**.

An invalid transaction or incompatible employee classification must produce an error and must not modify the system.

---

## UC-04: Post Sales Receipt

Creates and associates a sales receipt with an employee.

```text
SalesReceipt <EmpID> <date> <amount>
```

The target employee must be classified as **Commissioned**.

An invalid transaction or incompatible employee classification must produce an error and must not modify the system.

---

## UC-05: Post Union Service Charge

Creates and associates a service charge with a union member.

```text
ServiceCharge <memberID> <amount>
```

The member ID must identify an active union member.

An invalid transaction or invalid member ID must produce an error and must not modify the system.

---

## UC-06: Change Employee Details

Changes an existing employee's information or configuration.

### Change Name

```text
ChgEmp <EmpID> Name <name>
```

### Change Address

```text
ChgEmp <EmpID> Address <address>
```

### Change to Hourly

```text
ChgEmp <EmpID> Hourly <hourlyRate>
```

### Change to Salaried

```text
ChgEmp <EmpID> Salaried <salary>
```

### Change to Commissioned

```text
ChgEmp <EmpID> Commissioned <salary> <rate>
```

### Change Payment Method to Hold

```text
ChgEmp <EmpID> Hold
```

### Change Payment Method to Direct Deposit

```text
ChgEmp <EmpID> Direct <bank> <account>
```

### Change Payment Method to Mail

```text
ChgEmp <EmpID> Mail <address>
```

### Enroll Employee in Union

```text
ChgEmp <EmpID> Member <memberID> Dues <rate>
```

### Remove Employee from Union

```text
ChgEmp <EmpID> NoMember
```

If the transaction is invalid, the employee ID is invalid, or the union member ID is already registered, the system must report an error and take no action.

---

## UC-07: Run Payroll

Processes payroll for a specified date.

```text
Payday <date>
```

The system must:

1. Identify employees eligible for payment on the specified date.
2. Calculate gross pay.
3. Calculate applicable deductions.
4. Calculate net pay.
5. Execute the employee's selected payment method.
6. Produce an audit-trail report describing the payroll processing performed.

Net pay is calculated as:

```text
Net Pay = Gross Pay - Deductions
```

---

# 7. Initial Domain Concepts

The requirements identify the following concepts:

* Employee
* Employee Classification
* Hourly Employee
* Salaried Employee
* Commissioned Employee
* Time Card
* Sales Receipt
* Union Membership
* Union Dues
* Service Charge
* Payment Method
* Pay Schedule
* Payroll
* Gross Pay
* Deductions
* Net Pay

These concepts are candidates for the domain model.

Their final responsibilities and relationships will be determined during the design of the corresponding use cases.

---

# 8. Architectural Constraints

The implementation must comply with the Clean Architecture foundation defined in ADR-001.

In particular:

* Business rules must remain independent of frameworks.
* Domain logic must not depend on infrastructure.
* Use Cases must not depend on Controllers or Presenters.
* External systems must be accessed through appropriate boundaries.
* AI/LLM infrastructure must not become a dependency of core payroll business rules.
* Architectural dependencies must point inward.

The transaction parser, database, payment mechanisms, and external services are implementation details and must remain replaceable.

---

# 9. AI-Native Development Constraints

This specification is also a contract for AI agents participating in development.

An AI agent must:

* Treat this specification as the authoritative source for the defined requirements.
* Implement only behavior supported by an approved specification.
* Identify ambiguity instead of silently inventing important business rules.
* Preserve the established architectural boundaries.
* Produce verification evidence for implemented requirements.
* Avoid modifying unrelated functionality.
* Identify assumptions explicitly when an implementation decision is not defined by the specification.

An agent must not consider a task complete solely because the code compiles or tests pass.

---

# 10. Verification Expectations

Each implemented use case should eventually have verification covering:

* Valid transaction processing
* Invalid transaction handling
* Business rule enforcement
* Appropriate employee classification
* Payroll eligibility
* Pay calculation
* Deduction calculation
* Payment method behavior
* Architectural constraints

Verification should provide evidence that the implementation satisfies both the **functional specification** and the **architectural constraints**.

---

# 11. Scope Boundaries

The initial implementation is intentionally limited to the requirements defined in the available system specification.

The following are **not yet specified** and must not be invented by implementation agents:

* Country-specific taxation
* Statutory deductions
* Benefits administration
* Leave management
* Payroll accounting
* Employee authentication
* Multi-company payroll
* Payroll reporting beyond the specified audit trail
* External banking integration

Future capabilities must be introduced through explicit specifications.

---

# 12. Evolution of the Specification

This document defines the initial Payroll system scope.

Detailed requirements will be progressively decomposed into individual feature specifications.

The expected flow is:

**Business Intent → Feature Specification → Governance → Planning → Implementation → Verification → Approval**

Each significant feature should remain traceable from its original requirement through implementation and verification.
