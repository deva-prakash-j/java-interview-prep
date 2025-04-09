# CQRS and Saga Pattern with Spring Boot Microservices – A Real-World Banking Example

Distributed systems often face challenges such as performance, scalability, and maintaining consistency across services. Two architectural patterns—**CQRS (Command Query Responsibility Segregation)** and **Saga**—tackle these challenges from different angles:  
- **CQRS** separates read (query) and write (command) operations so that each can be optimized independently.  
- **Saga** manages distributed transactions by breaking a business operation (such as funds transfer) into multiple local transactions with compensating actions to ensure eventual consistency.

In this guide, we explain these patterns, show how they solve distributed transaction problems in a banking domain, and provide sample Spring Boot code.

---

## Table of Contents

1. [Introduction to CQRS](#introduction-to-cqrs)
2. [Introduction to the Saga Pattern](#introduction-to-the-saga-pattern)
3. [Problem Statement in a Banking System](#problem-statement-in-a-banking-system)
4. [CQRS and Saga in a Banking Example](#cqrs-and-saga-in-a-banking-example)
   - [Step-by-Step Saga Flow for Funds Transfer](#step-by-step-saga-flow-for-funds-transfer)
   - [Example Code](#example-code)
5. [Advantages and Disadvantages](#advantages-and-disadvantages)
6. [Pitfalls and How to Overcome Them](#pitfalls-and-how-to-overcome-them)
7. [Conclusion](#conclusion)

---

## 1. Introduction to CQRS

**Command Query Responsibility Segregation (CQRS)** is an architectural pattern that separates operations that modify state (commands) from those that retrieve state (queries).  
- **Command Side**: Contains business logic to handle create, update, or delete operations.  
- **Query Side**: Offers optimized read-only endpoints for data retrieval.

**Benefits of CQRS include**:  
- Scalability by independently optimizing read and write paths.  
- Flexibility to tailor data models for commands and queries.  
- Easier integration with event sourcing, where state changes are recorded as events.

---

## 2. Introduction to the Saga Pattern

The **Saga pattern** manages distributed transactions by decomposing a long-running business process into a series of smaller local transactions. Each local transaction publishes an event that triggers the next step. In case of failure, the Saga executes compensating transactions to roll back the preceding steps.

**Key Concepts**:  
- **Local Transaction**: Operates on a single microservice and its data.  
- **Compensation Transaction**: Undo the effects of a previously successful transaction when needed.  
- **Orchestration vs. Choreography**:  
  - **Orchestration** uses a central coordinator to execute steps.  
  - **Choreography** relies on events broadcast between services.

---

## 3. Problem Statement in a Banking System

In a **banking system**, a funds transfer often involves multiple microservices. For instance:  
- **Debit Service**: Deducts funds from the source account.  
- **Credit Service**: Credits funds to the destination account.

The challenges include:  
- **Distributed Transaction**: Ensuring that both debit and credit operations succeed (or the debit gets rolled back) across separate services.  
- **Scalability**: Read and write operations need to scale independently.  
- **Resilience**: Failures in one service should trigger compensating actions to keep the overall system consistent.

CQRS can separate the command (transactional) operations from the query (reporting) operations, and the Saga pattern can ensure that a multi-step funds transfer is either completely applied or safely compensated.

---

## 4. CQRS and Saga in a Banking Example

### Step-by-Step Saga Flow for Funds Transfer

1. **Initiate Transfer (Command Side)**  
   The client sends a transfer command (e.g., withdraw funds from account A and deposit them to account B) via a dedicated REST endpoint.

2. **Debit Source Account**  
   - The command service calls the debit operation on the account service.  
   - If successful, the service publishes an event or returns a success result.

3. **Credit Destination Account**  
   - Next, the command service calls the credit operation on the destination account.  
   - If the credit fails (e.g., due to a rule violation or system error), the Saga triggers a compensating transaction.

4. **Compensation**  
   - The compensating transaction reverts the debit on the source account so that funds are restored.
  
5. **Query Side**  
   - Separately, a query service exposes endpoints to retrieve the latest account state.  
   - This allows for optimized and separate scaling of read operations.

---

### Example Code

Below is a simplified Spring Boot application that demonstrates a CQRS–based banking microservice integrated with a Saga-style transfer.

> **Note**: In a full microservices architecture, the debit and credit operations might exist in different services communicating over REST or messaging. In this example, we simulate both command and query operations in one project.

#### 4.1 Main Application

```java
package com.example.banking;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class BankingCqrsSagaApplication {
    public static void main(String[] args) {
        SpringApplication.run(BankingCqrsSagaApplication.class, args);
    }
}
```

#### 4.2 Domain: Account Model

```java
package com.example.banking.model;

public class Account {
    private String accountId;
    private double balance;

    public Account() { }

    public Account(String accountId, double balance) {
        this.accountId = accountId;
        this.balance = balance;
    }

    // Getters and Setters
    public String getAccountId() {
        return accountId;
    }
    public void setAccountId(String accountId) {
        this.accountId = accountId;
    }
    public double getBalance() {
        return balance;
    }
    public void setBalance(double balance) {
        this.balance = balance;
    }
}
```

#### 4.3 In-Memory Account Repository

```java
package com.example.banking.repository;

import com.example.banking.model.Account;
import org.springframework.stereotype.Repository;

import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.ConcurrentMap;

@Repository
public class AccountRepository {
    // Simulated persistence storage
    private final ConcurrentMap<String, Account> accountStore = new ConcurrentHashMap<>();

    public AccountRepository() {
        // Initialize with dummy accounts
        accountStore.put("A001", new Account("A001", 1000.0));
        accountStore.put("B001", new Account("B001", 500.0));
    }

    public Account findById(String accountId) {
        return accountStore.get(accountId);
    }

    public void save(Account account) {
        accountStore.put(account.getAccountId(), account);
    }
}
```

#### 4.4 Service Layer: AccountService (Command and Query Operations)

```java
package com.example.banking.service;

import com.example.banking.model.Account;
import com.example.banking.repository.AccountRepository;
import org.springframework.stereotype.Service;

@Service
public class AccountService {

    private final AccountRepository repository;

    public AccountService(AccountRepository repository) {
        this.repository = repository;
    }

    // Command operations
    public boolean deposit(String accountId, double amount) {
        Account account = repository.findById(accountId);
        if (account == null) return false;
        account.setBalance(account.getBalance() + amount);
        repository.save(account);
        System.out.println("Deposited " + amount + " to account " + accountId);
        return true;
    }

    public boolean withdraw(String accountId, double amount) {
        Account account = repository.findById(accountId);
        if (account == null || account.getBalance() < amount) return false;
        account.setBalance(account.getBalance() - amount);
        repository.save(account);
        System.out.println("Withdrew " + amount + " from account " + accountId);
        return true;
    }

    // Query operation
    public Account getAccount(String accountId) {
        return repository.findById(accountId);
    }
}
```

#### 4.5 Saga Orchestrator Service: TransferSagaService

This service coordinates the transfer using a simple orchestrator approach. If the credit fails after withdrawal, it compensates by re-depositing the withdrawn amount.

```java
package com.example.banking.service;

import org.springframework.stereotype.Service;

@Service
public class TransferSagaService {

    private final AccountService accountService;

    public TransferSagaService(AccountService accountService) {
        this.accountService = accountService;
    }

    /**
     * Executes a funds transfer between two accounts as a Saga.
     */
    public String transferFunds(String sourceAccount, String destinationAccount, double amount) {
        System.out.println("Initiating funds transfer...");
        // Step 1: Withdraw from source account (Command)
        boolean withdrawn = accountService.withdraw(sourceAccount, amount);
        if (!withdrawn) {
            return "Transfer failed: Insufficient funds or invalid source account.";
        }

        // Step 2: Deposit into destination account (Command)
        boolean deposited = accountService.deposit(destinationAccount, amount);
        if (!deposited) {
            // Compensation: Revert the withdrawal on the source account
            accountService.deposit(sourceAccount, amount);
            return "Transfer failed: Unable to credit destination account. Rollback executed.";
        }
        return "Transfer successful.";
    }
}
```

#### 4.6 Controllers

**Command Controller (for Transfer Operations):**

```java
package com.example.banking.controller;

import com.example.banking.service.TransferSagaService;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/account/command")
public class AccountCommandController {

    private final TransferSagaService transferSagaService;

    public AccountCommandController(TransferSagaService transferSagaService) {
        this.transferSagaService = transferSagaService;
    }

    @PostMapping("/transfer")
    public ResponseEntity<String> transferFunds(@RequestParam String sourceAccount,
                                                  @RequestParam String destinationAccount,
                                                  @RequestParam double amount) {
        String result = transferSagaService.transferFunds(sourceAccount, destinationAccount, amount);
        return ResponseEntity.ok(result);
    }
}
```

**Query Controller (for Reading Account Information):**

```java
package com.example.banking.controller;

import com.example.banking.model.Account;
import com.example.banking.service.AccountService;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/account/query")
public class AccountQueryController {

    private final AccountService accountService;

    public AccountQueryController(AccountService accountService) {
        this.accountService = accountService;
    }

    @GetMapping("/{accountId}")
    public ResponseEntity<Account> getAccount(@PathVariable String accountId) {
        Account account = accountService.getAccount(accountId);
        if (account == null) {
            return ResponseEntity.notFound().build();
        }
        return ResponseEntity.ok(account);
    }
}
```

> **Usage Example**:  
> - To transfer funds, send a POST request to:  
>   `/account/command/transfer?sourceAccount=A001&destinationAccount=B001&amount=100`  
> - To get an account balance, send a GET request to:  
>   `/account/query/A001`

---

## 5. Advantages and Disadvantages

**Advantages**:
- **Separation of Concerns**: CQRS cleanly divides the command and query sides, allowing independent scaling and optimization.
- **Resilience and Flexibility**: Saga pattern manages distributed transactions reliably with compensating actions.
- **Better Performance**: Optimized read and write data models improve overall application performance.

**Disadvantages**:
- **Increased Complexity**: Splitting operations and managing sagas requires additional infrastructure and design effort.
- **Eventual Consistency**: Data on the query side may be slightly delayed compared to the command side.
- **Operational Overhead**: Monitoring and debugging distributed sagas across multiple services can be challenging.

---

## 6. Pitfalls and How to Overcome Them

1. **Complex Saga Implementation**:  
   - *Pitfall*: Writing and testing compensating transactions can be error-prone.  
   - *Mitigation*: Use a well-defined saga orchestration framework and thoroughly test each scenario.

2. **Eventual Consistency**:  
   - *Pitfall*: The query view may lag behind the command view, causing temporary inconsistencies.  
   - *Mitigation*: Define clear SLA expectations and consider real-time streaming or caching mechanisms.

3. **Increased Infrastructure Complexity**:  
   - *Pitfall*: Separating services and managing distributed transactions increases deployment and monitoring challenges.  
   - *Mitigation*: Adopt centralized logging, distributed tracing (e.g., Spring Cloud Sleuth, Zipkin), and robust monitoring tools.

4. **Error Handling in Sagas**:  
   - *Pitfall*: Failing compensations can leave data inconsistent.  
   - *Mitigation*: Implement retry strategies, ensure idempotency, and design manual intervention workflows for critical cases.

---

## 7. Conclusion

By combining **CQRS** and the **Saga** pattern in a Spring Boot microservices environment, you can build scalable, resilient distributed systems. In our banking example, we demonstrated how funds transfer can be divided into separate command operations (withdraw and deposit) with compensation in case of failure. Although these patterns introduce complexity and eventual consistency challenges, careful design, monitoring, and testing will lead to robust and maintainable systems.

Happy coding!

---
