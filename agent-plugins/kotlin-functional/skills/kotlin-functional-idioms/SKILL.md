---
name: kotlin-functional-idioms
description: |
  Use this skill when generating Kotlin implementation code, reviewing existing Kotlin code,
  refactoring to functional patterns, writing domain models and business logic, creating
  type-safe APIs, or documenting contracts and invariants in Kotlin.
version: 1.0.0
---

# Skill: Kotlin Functional Idioms

Core knowledge of functional Kotlin patterns and idiomatic usage for generating high-quality, maintainable code.

## Overview

This skill provides expertise in writing **functional-first, idiomatic Kotlin** code that emphasizes:

1. **Immutability**: `val` properties, immutable collections, pure transformations
2. **Type Safety**: Sealed classes, value classes, Result types
3. **Expression-Oriented**: Minimal statements, maximum expressions
4. **Comprehensive Documentation**: KDoc with contracts, invariants, postconditions
5. **Functional Composition**: Higher-order functions, extension functions, pipelines

## Core Principles

### 1. Immutability First

**Default to `val`, not `var`**

```kotlin
// Avoid
var user = User(id, email)
user.email = newEmail

// Prefer
val user = User(id, email)
val updatedUser = user.copy(email = newEmail)
```

**Use immutable collections**

```kotlin
// Avoid
val users = mutableListOf<User>()
users.add(newUser)

// Prefer
val users = existingUsers + newUser
```

**Benefits:**
- No temporal coupling
- Thread-safe by default
- Easier to reason about
- No defensive copying needed

### 2. Type-Safe Error Handling

**Use Result<T> for expected failures**

```kotlin
// Avoid
fun findUser(id: Long): User {
    return repository.find(id) ?: throw UserNotFoundException(id)
}

// Prefer
fun findUser(id: Long): Result<User> =
    repository.find(id)
        ?.let { Result.success(it) }
        ?: Result.failure(UserNotFoundException(id))
```

**Use sealed interfaces for domain errors**

```kotlin
sealed interface ValidationError {
    data class InvalidEmail(val email: String) : ValidationError
    data class WeakPassword(val violations: List<String>) : ValidationError
}

fun validateUser(email: String, password: String): Result<ValidatedUser> =
    validateEmail(email)
        .flatMap { validEmail ->
            validatePassword(password)
                .map { validPassword ->
                    ValidatedUser(validEmail, validPassword)
                }
        }
```

**Benefits:**
- Errors explicit in type signature
- Exhaustive when-expression handling
- No hidden control flow
- Composable via map/flatMap

### 3. Expression-Oriented Code

**Prefer when expressions**

```kotlin
// Avoid (statement-based)
fun getDiscount(tier: Tier): Double {
    if (tier == Tier.GOLD) {
        return 0.15
    } else if (tier == Tier.SILVER) {
        return 0.10
    } else {
        return 0.05
    }
}

// Prefer (expression-based)
fun getDiscount(tier: Tier): Double = when (tier) {
    Tier.GOLD -> 0.15
    Tier.SILVER -> 0.10
    Tier.BRONZE -> 0.05
}
```

**Use expression bodies**

```kotlin
// Verbose
fun isAdmin(): Boolean {
    return role == Role.ADMIN
}

// Concise
fun isAdmin(): Boolean = role == Role.ADMIN
```

**Benefits:**
- Forces you to return values
- More concise
- Easier to test
- Functional style

### 4. Null Safety

**Only use nullable types where nullability is semantic**

```kotlin
// Avoid (nullability hides failure)
fun findUser(id: Long): User? = repository.find(id)

// Prefer (explicit failure semantics)
fun findUser(id: Long): Result<User> =
    repository.find(id)
        ?.let { Result.success(it) }
        ?: Result.failure(UserNotFound(id))
```

**Never use `!!` operator**

```kotlin
// Avoid (can throw NPE)
val email = user.email!!.lowercase()

// Prefer (safe navigation)
val email = user.email?.lowercase()

// Or handle explicitly
val email = user.email?.lowercase() ?: return Result.failure(MissingEmail)
```

**Benefits:**
- Explicit nullability semantics
- No runtime NPEs
- Clear intent

### 5. Higher-Order Functions

**Abstract patterns with function parameters**

```kotlin
// Avoid (inheritance-based abstraction)
abstract class BaseService {
    abstract fun process()

    fun withLogging() {
        logger.info("Starting")
        process()
        logger.info("Done")
    }
}

// Prefer (function-based abstraction)
inline fun <T> withLogging(operation: () -> T): T {
    logger.info("Starting")
    return operation().also { logger.info("Done") }
}
```

**Use inline for zero overhead**

```kotlin
inline fun <T> withRetry(
    maxAttempts: Int = 3,
    operation: () -> Result<T>
): Result<T> {
    repeat(maxAttempts - 1) {
        operation().onSuccess { return Result.success(it) }
    }
    return operation()
}
```

**Benefits:**
- Behavior composition without inheritance
- Reusable patterns
- Zero overhead with inline

### 6. Extension Functions

**Add domain operations as extensions**

```kotlin
// Avoid (utility class)
object UserUtils {
    fun filterActive(users: List<User>): List<User> =
        users.filter { it.isActive }
}

// Prefer (extension function)
fun List<User>.filterActive(): List<User> =
    filter { it.isActive }
```

**Chain operations naturally**

```kotlin
val admins = users
    .filterActive()
    .withRole(Role.ADMIN)
    .sortedByName()
```

**Benefits:**
- Discoverable via IDE completion
- Natural chaining
- No utility classes
- Keeps types focused

### 7. Scope Functions

**Use appropriate scope function for each case**

```kotlin
// let: transformation and null handling
val email = user.email?.let { it.lowercase() }

// run: multi-statement expressions
val result = run {
    val validated = validate(input)
    val processed = process(validated)
    save(processed)
}

// apply: object configuration
val user = User(id, email).apply {
    logger.info("Created user $id")
}

// also: side effects in pipeline
val user = createUser(data)
    .also { logger.info("Created: $it") }
    .also { metrics.increment("users.created") }

// with: operating on context object
with(configuration) {
    connectToDatabase(host, port, database)
}
```

**Benefits:**
- Readable pipelines
- No temporary variables
- Clear intent

### 8. Lazy Sequences

**Use for large or infinite collections**

```kotlin
// Avoid (creates intermediate lists)
val result = users
    .filter { it.isActive }  // List
    .map { it.email }        // List
    .take(10)                // List

// Prefer (lazy evaluation)
val result = users.asSequence()
    .filter { it.isActive }
    .map { it.email }
    .take(10)
    .toList()  // Only creates one list
```

**Generate infinite sequences**

```kotlin
val fibonacci = generateSequence(0 to 1) { (a, b) -> b to (a + b) }
    .map { it.first }

val first10Fibonacci = fibonacci.take(10).toList()
```

**Benefits:**
- Efficient transformations
- Lazy evaluation
- Support infinite streams
- Compose without overhead

### 9. Value Classes (Type-Safe Primitives)

**Wrap primitives for type safety**

```kotlin
// Avoid (easy to mix up)
fun transfer(fromId: Long, toId: Long, amount: Long)

// Prefer (type-safe, zero overhead)
@JvmInline
value class UserId(val value: Long)

@JvmInline
value class Money(val cents: Long)

fun transfer(from: UserId, to: UserId, amount: Money)
```

**Smart constructors with validation**

```kotlin
@JvmInline
value class Email private constructor(val value: String) {
    companion object {
        fun of(value: String): Result<Email> =
            if (isValid(value)) Result.success(Email(value))
            else Result.failure(InvalidEmail(value))

        private fun isValid(value: String): Boolean =
            value.matches(Regex("""[\w.+-]+@[\w.-]+\.[a-zA-Z]{2,}"""))
    }
}
```

**Benefits:**
- Zero runtime overhead
- Type safety at compile time
- Invalid states unrepresentable
- Self-documenting code

### 10. Data Classes for Domain Models

**Immutable domain entities**

```kotlin
/**
 * Represents an immutable user entity.
 *
 * ## Invariants
 * - [email] is validated and unique
 * - [createdAt] is never in the future
 */
data class User(
    val id: UserId,
    val email: Email,
    val name: String,
    val role: Role,
    val createdAt: Instant
) {
    init {
        require(name.isNotBlank()) { "Name cannot be blank" }
    }

    fun withRole(newRole: Role): User = copy(role = newRole)
    fun isAdmin(): Boolean = role == Role.ADMIN
}
```

**Benefits:**
- Automatic equals/hashCode/toString
- copy() for transformations
- Destructuring support
- Immutable by default

## Comprehensive KDoc Standards

### Class Documentation

```kotlin
/**
 * [One-sentence summary of what this class represents]
 *
 * [Detailed description: purpose, usage, design decisions]
 *
 * ## Invariants
 * - [Property 1] always satisfies constraint X
 * - [Property 2] is maintained across all operations
 *
 * ## Usage Example
 * ```kotlin
 * val user = User(id, email, name, role, timestamp)
 * val admin = user.withRole(Role.ADMIN)
 * ```
 *
 * @property prop1 Description (include constraints)
 * @property prop2 Description (include valid ranges)
 * @see RelatedClass
 */
data class ClassName(...)
```

### Function Documentation

```kotlin
/**
 * [One-sentence summary of what this function does]
 *
 * [Detailed description: behavior, edge cases, design rationale]
 *
 * ## Preconditions
 * - [param1] must satisfy constraint X
 * - System state Y must be true before calling
 *
 * ## Postconditions
 * - On success: Guarantee A holds
 * - On failure: No state changes occur
 *
 * ## Side Effects
 * - [List any observable effects beyond return value]
 *
 * @param param1 Description (include constraints)
 * @param param2 Description (include valid values)
 * @return Description (include success/failure semantics)
 * @throws ExceptionType Only for truly exceptional cases
 * @see relatedFunction
 */
fun functionName(param1: Type1, param2: Type2): Result<ReturnType>
```

### Sealed Hierarchy Documentation

```kotlin
/**
 * Represents all possible errors during user registration.
 *
 * This sealed hierarchy ensures exhaustive when-expression handling,
 * making error handling explicit and type-safe.
 *
 * @see registerUser
 */
sealed interface RegistrationError {
    /**
     * The provided email is already registered.
     *
     * @property email The duplicate email address
     */
    data class EmailExists(val email: Email) : RegistrationError

    /**
     * The password doesn't meet security requirements.
     *
     * @property violations List of requirement violations
     */
    data class WeakPassword(val violations: List<String>) : RegistrationError
}
```

## Common Patterns

### Pattern: Validation Pipeline

```kotlin
fun validateUser(data: UserData): Result<ValidatedUser> =
    validateEmail(data.email)
        .flatMap { email ->
            validatePassword(data.password)
                .flatMap { password ->
                    validateName(data.name)
                        .map { name ->
                            ValidatedUser(email, password, name)
                        }
                }
        }
```

### Pattern: Smart Constructor

```kotlin
data class User private constructor(
    val id: UserId,
    val email: Email,
    val name: String
) {
    companion object {
        fun create(id: UserId, email: Email, name: String): Result<User> {
            if (name.isBlank()) {
                return Result.failure(InvalidName("Name cannot be blank"))
            }
            return Result.success(User(id, email, name))
        }
    }
}
```

### Pattern: Retry with Backoff

```kotlin
suspend fun <T> withRetry(
    maxAttempts: Int = 3,
    initialDelay: Duration = 100.milliseconds,
    operation: suspend () -> Result<T>
): Result<T> {
    var currentDelay = initialDelay

    repeat(maxAttempts - 1) {
        operation().onSuccess { return Result.success(it) }
        delay(currentDelay)
        currentDelay *= 2
    }

    return operation()
}
```

### Pattern: Extension DSL

```kotlin
fun List<User>.query(): UserQuery = UserQuery(this)

class UserQuery(private val users: List<User>) {
    fun withRole(role: Role) = UserQuery(users.filter { it.role == role })
    fun active() = UserQuery(users.filter { it.isActive })
    fun sortedByName() = UserQuery(users.sortedBy { it.name })
    fun take(n: Int): List<User> = users.take(n)
}

// Usage:
val admins = users.query()
    .active()
    .withRole(Role.ADMIN)
    .sortedByName()
    .take(10)
```

### Pattern: Sealed Hierarchy with Exhaustive Handling

```kotlin
sealed interface PaymentResult {
    data class Success(val transactionId: TransactionId) : PaymentResult

    sealed interface Failure : PaymentResult {
        data class InsufficientFunds(val available: Money) : Failure
        data class CardExpired(val expirationDate: YearMonth) : Failure
        data object NetworkError : Failure
    }
}

fun handlePayment(result: PaymentResult): String = when (result) {
    is PaymentResult.Success -> "Transaction ${result.transactionId} successful"
    is PaymentResult.Failure.InsufficientFunds -> "Insufficient funds: ${result.available} available"
    is PaymentResult.Failure.CardExpired -> "Card expired on ${result.expirationDate}"
    is PaymentResult.Failure.NetworkError -> "Network error, please retry"
}
```

## Anti-Patterns to Avoid

See `references/anti-patterns.md` for comprehensive list.

Quick reference:
1. `var` properties in data classes
2. Mutable collections
3. Nullable returns instead of Result
4. Exception-based flow control
5. Imperative loops with accumulators
6. `!!` operator
7. Missing KDoc
8. Inheritance for behavior composition

## Related Resources

- `examples/` - Comprehensive pattern examples
- `references/` - Reference guides and checklists
- Kotlin official docs: https://kotlinlang.org/docs/home.html
- Arrow library (advanced FP): https://arrow-kt.io/

## When to Use This Skill

Use functional-idioms when:
- Generating Kotlin implementation code
- Reviewing existing Kotlin code
- Refactoring to functional patterns
- Writing domain models and business logic
- Creating type-safe APIs
- Documenting contracts and invariants

## Success Metrics

Code following these patterns should achieve:
- 90%+ immutability (val properties)
- Type-safe error handling throughout
- Comprehensive KDoc on all public APIs
- Zero `!!` operators
- Minimal imperative loops
- Expression-oriented style
