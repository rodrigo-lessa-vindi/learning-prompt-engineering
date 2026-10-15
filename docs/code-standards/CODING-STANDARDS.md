# Coding Standards — .NET Projects

> Based on **Clean Code** (Robert C. Martin) and **Refactoring** (Martin Fowler).  
> These rules apply to all .NET projects in this organization. Every rule listed here is expected to be followed unless explicitly justified and documented in the PR.

---

## Table of Contents

1. [Naming](#1-naming)
2. [Functions & Methods](#2-functions--methods)
3. [Classes & Design](#3-classes--design)
4. [Comments](#4-comments)
5. [Error Handling](#5-error-handling)
6. [Data & State](#6-data--state)
7. [Refactoring Patterns](#7-refactoring-patterns)
8. [Testability](#8-testability)
9. [Architecture & Structure](#9-architecture--structure)
10. [.NET Specifics](#10-net-specifics)

---

## 1. Naming

> *"The name of a variable, function, or class should answer all the big questions. It should tell you why it exists, what it does, and how it is used."* — Uncle Bob

| Rule | ✅ Do | ❌ Avoid |
|---|---|---|
| Use meaningful, pronounceable names | `customerAccount` | `ca`, `x`, `tmp` |
| Classes should be nouns | `OrderProcessor`, `CustomerRepository` | `OrderManager`, `DataHelper` |
| Methods should be verbs | `CalculateTotal()`, `SendNotification()` | `Total()`, `Notif()` |
| Booleans should read as predicates | `isActive`, `hasPermission`, `canExecute` | `active`, `flag`, `check` |
| Constants in UPPER_SNAKE_CASE | `MAX_RETRY_COUNT` | `maxRetry`, `MaxRetry` |

**Additional rules:**

- Avoid mental mapping: never use single-letter names outside of loop counters (`i`, `j`, `k`).
- Avoid noise words such as `Manager`, `Data`, `Info`, `Helper`, and `Utils` in class names — they indicate poor abstraction.
- Avoid Hungarian notation (e.g., `strName`, `intCount`, `bIsValid`).
- Abbreviations are forbidden unless they are universally understood (e.g., `Id`, `Url`, `Http`).
- Names should be searchable: avoid one-letter or cryptic short names in class-level scope.

---

## 2. Functions & Methods

> *"The first rule of functions is that they should be small. The second rule of functions is that they should be smaller than that."* — Uncle Bob

- **Do one thing:** A function should do one thing, do it well, and do it only.
- **Keep them small:** If a method doesn't fit on one screen, it is likely doing too much. Aim for 10–20 lines as a guideline.
- **Limit parameters:** Maximum of **3 parameters** per method. If you need more, introduce a Parameter Object or use a builder.
- **No output parameters:** Prefer return values over `out`/`ref` parameters.
- **No flag parameters:** Avoid passing `bool` to change a method's behavior — split into two clearly named methods instead.
- **Command-Query Separation (CQS):** A method should either *do something* (command) or *answer something* (query), never both.
- **DRY — Don't Repeat Yourself:** Duplicated logic must be extracted into a shared, well-named method.
- **Avoid deep nesting:** Maximum 2–3 levels of nesting. Use guard clauses and early returns to flatten structure.

```csharp
// ❌ Avoid
public void Process(Order order, bool sendEmail)
{
    if (order != null)
    {
        if (order.IsValid())
        {
            // ... processing ...
            if (sendEmail) { /* ... */ }
        }
    }
}

// ✅ Prefer
public void Process(Order order)
{
    if (order is null) throw new ArgumentNullException(nameof(order));
    if (!order.IsValid()) return;

    ExecuteProcessing(order);
}
```

---

## 3. Classes & Design

> *"A class should have only one reason to change."* — Single Responsibility Principle

- **SRP:** Each class should have a single, well-defined responsibility.
- **OCP:** Classes should be open for extension and closed for modification.
- **DIP:** Depend on abstractions (interfaces), not on concrete implementations.
- **Prefer composition over inheritance:** Use interfaces and composed behaviors instead of deep inheritance chains.
- **High cohesion:** All members of a class should relate directly to its stated purpose.
- **Avoid God Classes:** If a class has more than 10 public methods or mixes multiple concerns, break it apart.
- **Small classes:** Strive for classes under 200–300 lines. If a class grows beyond that, it likely violates SRP.
- **Liskov Substitution Principle (LSP):** Derived classes must be usable in place of their base class without breaking behavior.
- **Interface Segregation Principle (ISP):** Prefer small, specific interfaces over large, general-purpose ones.

---

## 4. Comments

> *"Don't comment bad code — rewrite it."* — Uncle Bob

- **Self-documenting code is the goal.** Use expressive names and clear structure instead of explanatory comments.
- **Comments should explain WHY, never WHAT.** The code shows what; the comment explains the business or technical reason.
- **Never leave commented-out code.** Use version control (Git history) instead.
- **TODO comments must reference a ticket:** `// TODO: PROJ-123 - refactor after feature X is stable`
- **Avoid redundant comments** that merely restate what the code already clearly says.
- **XML documentation (`///`)** is required on all `public` APIs, interfaces, and non-obvious methods.

```csharp
// ❌ Avoid — states the obvious
// Increment counter by 1
counter++;

// ✅ Good — explains the WHY
// We retry once before failing to handle transient DNS resolution failures in containerized environments.
if (attempt < 2) Retry(request);
```

---

## 5. Error Handling

> *"Error handling is important, but if it obscures logic, it's wrong."* — Uncle Bob

- **Never return `null`** from a method. Use `Result<T>`, `Optional<T>`, or throw a meaningful exception.
- **Never pass `null` as a parameter.** Validate at system/service boundaries and fail fast.
- **Use specific exception types.** Never throw `Exception` or `ApplicationException` directly.
- **Don't use exceptions for flow control.** Exceptions are for truly exceptional, unexpected situations.
- **Include context in exception messages.** State what operation failed and with what input.
- **Log at the boundary layer only.** Don't log the same error in every intermediate method — it creates noise.
- **Catch what you can handle.** Don't catch exceptions you cannot meaningfully recover from; let them propagate.

```csharp
// ❌ Avoid
public User GetUser(int id)
{
    return _db.Users.FirstOrDefault(u => u.Id == id); // returns null silently
}

// ✅ Prefer
public User GetUser(int id)
{
    return _db.Users.FirstOrDefault(u => u.Id == id)
        ?? throw new UserNotFoundException($"User with id {id} was not found.");
}
```

---

## 6. Data & State

> *"Primitive Obsession is one of the most common smells in code."* — Martin Fowler

- **Encapsulate primitives** when they carry business meaning: use `Email`, `Money`, `Cpf`, `PhoneNumber` instead of raw `string`.
- **Avoid Data Clumps:** Groups of values that always travel together (e.g., `street`, `city`, `zipCode`) should become their own class (`Address`).
- **Prefer immutability:** Mark fields as `readonly` wherever possible. Use `init` accessors on records and DTOs.
- **Avoid public setters on domain entities.** Use constructors, factory methods, or named domain operations.
- **Replace magic numbers and strings** with named constants or enums.
- **Avoid Primitive Obsession:** Don't represent domain concepts with raw `int`, `string`, or `bool`.

```csharp
// ❌ Avoid
public void CreateOrder(string email, decimal amount, string currencyCode) { }

// ✅ Prefer
public void CreateOrder(Email email, Money amount) { }
```

---

## 7. Refactoring Patterns

> *"Whenever you feel the need to write a comment to explain what the code does, extract it into a named method."* — Martin Fowler

Apply these refactoring patterns proactively during development and code reviews:

| Pattern | When to Apply |
|---|---|
| **Extract Method** | A code block needs a comment to be understood → extract into a named method |
| **Introduce Parameter Object** | A method takes 3+ related parameters → group into a class |
| **Replace Conditional with Polymorphism** | Long `switch`/`if-else` on type → use polymorphism or strategy pattern |
| **Replace Magic Literal with Constant** | Hardcoded numbers or strings inline → named constant or enum |
| **Move Method / Move Field** | A member uses more data from another class than its own → move it |
| **Decompose Conditional** | Complex boolean expressions → extract into well-named predicate methods |
| **Replace Temp with Query** | Local variable stores a computed value → replace with a method call |
| **Introduce Null Object** | Repeated null checks for an object → implement a Null Object pattern |
| **Consolidate Duplicate Conditional Fragments** | Same code in all branches of a conditional → move it outside |

---

## 8. Testability

> *"The only way to go fast is to go well."* — Uncle Bob

- **All business logic must have unit tests.** There are no exceptions.
- **Tests follow the AAA pattern:** Arrange, Act, Assert — with a blank line separating each section.
- **Test names describe behavior:**
  ```
  MethodName_Scenario_ExpectedResult
  // e.g.: CalculateTotal_WithDiscountApplied_ReturnsDiscountedValue
  ```
- **Each test asserts a single behavior.** One logical assertion per test (multiple `Assert` calls for a single outcome are acceptable).
- **No logic in tests.** Avoid loops, conditionals, and try/catch blocks inside test methods.
- **Test observable behavior, not implementation details.** Tests should not break on internal refactors that don't change behavior.
- **Use fakes/mocks only for external dependencies:** I/O, network, databases, clocks, and third-party services.
- **Tests must be deterministic:** No reliance on current date/time, random values, or execution order without proper control.

---

## 9. Architecture & Structure

> *"The architecture should scream the intent of the system."* — Uncle Bob

- **Follow Clean/Layered Architecture:** Separate `Domain`, `Application`, `Infrastructure`, and `Presentation` layers.
- **Domain entities must be pure:** No references to infrastructure concerns, frameworks, or EF Core attributes in domain models.
- **Use Dependency Injection for all service dependencies.** Never instantiate services with `new` inside business logic.
- **No hardcoded configuration.** Connection strings, URLs, secrets, and feature flags must come from environment configuration (`IConfiguration`, `IOptions<T>`).
- **Avoid static state and Singleton abuse.** Make all dependencies explicit through constructor injection.
- **Enforce layer boundaries:** The Domain layer cannot reference Application or Infrastructure. Application cannot reference Infrastructure directly.
- **Keep controllers thin.** Controllers orchestrate — they do not contain business logic.

```
Solution/
├── src/
│   ├── Domain/          # Entities, Value Objects, Domain Events, Interfaces
│   ├── Application/     # Use Cases, DTOs, Command/Query Handlers, Validators
│   ├── Infrastructure/  # EF Core, Repositories, External Services, Messaging
│   └── Api/             # Controllers, Middlewares, Filters
└── tests/
    ├── Unit/
    ├── Integration/
    └── Architecture/    # ArchUnit / NetArchTest rules
```

---

## 10. .NET Specifics

- **`async`/`await` must be consistent.** Never mix blocking calls (`Task.Result`, `.Wait()`, `.GetAwaiter().GetResult()`) with async code — this causes deadlocks.
- **Always propagate `CancellationToken`** in async methods that perform I/O or long-running work.
- **Dispose unmanaged resources properly.** Use `IDisposable` / `IAsyncDisposable` and `using` declarations.
- **Prefer `IReadOnlyCollection<T>` or `IEnumerable<T>`** in return types over concrete `List<T>` or `Array`.
- **Use `record` types for immutable DTOs and Value Objects.**
- **Leverage pattern matching and `switch` expressions** over long `if-else` chains where idiomatic.
- **Never suppress nullable warnings with `!`** (null-forgiving operator) without an inline comment explaining why it is safe.
- **Avoid LINQ abuse:** Don't chain 10+ LINQ operators into an unreadable expression. Extract into intermediate named steps or methods.
- **ConfigureAwait(false):** Use in library and infrastructure code to avoid capturing the synchronization context unnecessarily.

```csharp
// ❌ Avoid — blocking on async
var result = _service.GetDataAsync().Result;

// ✅ Prefer
var result = await _service.GetDataAsync(cancellationToken);
```

---

## Enforcement

These standards should be enforced through:

- **Code Reviews:** All PRs must be reviewed against these standards before merging.
- **Static Analysis:** Roslyn Analyzers (`Microsoft.CodeAnalysis`, `StyleCop.Analyzers`, `SonarAnalyzer`) configured in `.editorconfig`.
- **Architecture Tests:** Use `NetArchTest` or `ArchUnitNET` to automate layer boundary rules.
- **Linting in CI:** Analysis warnings must be treated as errors in the build pipeline (`TreatWarningsAsErrors`).

---

*Last updated: 2026 — Maintained by the Engineering team.*
