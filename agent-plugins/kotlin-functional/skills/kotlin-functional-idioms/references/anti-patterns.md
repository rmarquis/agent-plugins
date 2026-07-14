# Anti-Patterns to Avoid

Common anti-patterns in Kotlin and their functional alternatives.

## 1. Mutable Data Structures

### Anti-Pattern: Mutable Properties

```kotlin
data class User(
    val id: Long,
    var email: String,    // Mutable
    var name: String,     // Mutable
    var role: Role        // Mutable
)

fun promoteUser(user: User) {
    user.role = Role.ADMIN  // Mutating in place
}
```

**Problems:**
- Temporal coupling (order of operations matters)
- Difficult to reason about state
- Not thread-safe
- Cannot safely share instances

### Functional Alternative

```kotlin
data class User(
    val id: Long,
    val email: String,    // Immutable
    val name: String,     // Immutable
    val role: Role        // Immutable
) {
    fun withRole(newRole: Role): User = copy(role = newRole)
}

fun promoteUser(user: User): User =
    user.withRole(Role.ADMIN)  // Returns new instance
```

## 2. Nullable Return for Failure

### Anti-Pattern: Returning Null

```kotlin
fun findUser(id: Long): User? {
    return repository.find(id)  // Null hides why lookup failed
}

// Caller doesn't know if user doesn't exist, DB failed, etc.
val user = findUser(123)
if (user == null) {
    // What happened? Don't know!
}
```

**Problems:**
- Hides the reason for failure
- Forces null checks everywhere
- Easy to forget null checks
- Cannot distinguish different failure modes

### Functional Alternative

```kotlin
sealed interface UserLookupError {
    data class NotFound(val id: Long) : UserLookupError
    data class DatabaseError(val cause: Exception) : UserLookupError
    data object Unauthorized : UserLookupError
}

fun findUser(id: Long): Result<User> =
    repository.find(id)

// Caller knows exactly what happened
findUser(123).fold(
    onSuccess = { user -> processUser(user) },
    onFailure = { error ->
        when (error) {
            is UserLookupError.NotFound -> showNotFoundMessage()
            is UserLookupError.DatabaseError -> retryOrShowError()
            is UserLookupError.Unauthorized -> redirectToLogin()
        }
    }
)
```

## 3. Exception-Based Flow Control

### Anti-Pattern: Throwing for Expected Failures

```kotlin
fun processPayment(amount: Money): Transaction {
    if (amount <= Money(0)) {
        throw IllegalArgumentException("Invalid amount")  // Expected failure
    }

    if (balance < amount) {
        throw InsufficientFundsException()  // Business rule violation
    }

    // ... process payment
}

// Caller forced to use try-catch for normal control flow
try {
    val transaction = processPayment(amount)
} catch (e: InsufficientFundsException) {
    // Using exceptions for flow control
}
```

**Problems:**
- Exceptions not visible in function signature
- Expensive (stack trace construction)
- Forces try-catch blocks for normal flow
- Unclear which exceptions a function can throw

### Functional Alternative

```kotlin
sealed interface PaymentError {
    data class InvalidAmount(val amount: Money) : PaymentError
    data class InsufficientFunds(val available: Money, val required: Money) : PaymentError
}

fun processPayment(amount: Money): Result<Transaction> =
    when {
        amount <= Money(0) ->
            Result.failure(PaymentError.InvalidAmount(amount))

        balance < amount ->
            Result.failure(PaymentError.InsufficientFunds(balance, amount))

        else ->
            Result.success(/* process payment */)
    }

// Caller handles errors explicitly
processPayment(amount).fold(
    onSuccess = { showSuccess(it) },
    onFailure = { error ->
        when (error) {
            is PaymentError.InvalidAmount -> showInvalidAmountError()
            is PaymentError.InsufficientFunds -> showInsufficientFundsError(error)
        }
    }
)
```

## 4. Imperative Loops with Accumulators

### Anti-Pattern: Mutable Accumulators

```kotlin
fun getActiveUserEmails(users: List<User>): List<String> {
    val emails = mutableListOf<String>()  // Mutable accumulator
    for (user in users) {                  // Imperative loop
        if (user.isActive) {
            emails.add(user.email.value)
        }
    }
    return emails
}
```

**Problems:**
- Mutable state
- Verbose
- Error-prone (easy to forget to add to accumulator)
- Not composable

### Functional Alternative

```kotlin
fun getActiveUserEmails(users: List<User>): List<String> =
    users
        .filter { it.isActive }
        .map { it.email.value }

// Or using extension function for clarity:
fun List<User>.activeEmails(): List<String> =
    filter { it.isActive }
        .map { it.email.value }
```

## 5. Inheritance for Behavior Composition

### Anti-Pattern: Abstract Base Classes

```kotlin
abstract class BaseService {
    abstract fun process()

    fun withLogging() {
        logger.info("Starting")
        process()
        logger.info("Done")
    }

    fun withMetrics() {
        metrics.start()
        process()
        metrics.stop()
    }
}

class UserService : BaseService() {
    override fun process() {
        // Can only use inherited behaviors
    }
}

class OrderService : BaseService() {
    override fun process() {
        // Tightly coupled to base class
    }
}
```

**Problems:**
- Tight coupling to base class
- Fragile base class problem
- Cannot compose behaviors dynamically
- Single inheritance limitation
- Difficult to test in isolation

### Functional Alternative

```kotlin
inline fun <T> withLogging(operation: () -> T): T {
    logger.info("Starting")
    return operation().also { logger.info("Done") }
}

inline fun <T> withMetrics(operation: () -> T): T {
    metrics.start()
    return operation().also { metrics.stop() }
}

class UserService {
    fun process() {
        withLogging {
            withMetrics {
                // Compose behaviors freely
                processUsers()
            }
        }
    }
}

class OrderService {
    fun process() {
        withMetrics {  // Use only what you need
            processOrders()
        }
    }
}
```

## 6. Missing or Superficial KDoc

### Anti-Pattern: No Documentation or "Useless" Documentation

```kotlin
/**
 * Gets the user.
 */
fun getUser(id: Long): User? = repository.find(id)

// Or worse:
fun processPayment(amount: Money): Result<Transaction> = /* ... */
```

**Problems:**
- Doesn't explain preconditions, postconditions, or failure modes
- No information about side effects
- Doesn't document business rules or constraints

### Functional Alternative

```kotlin
/**
 * Retrieves a user by their unique identifier.
 *
 * This function queries the primary user repository. For cached lookups,
 * consider using [findUserCached] instead.
 *
 * ## Preconditions
 * - [id] must be a valid user identifier
 *
 * ## Postconditions
 * - On success: Returns user with matching [id]
 * - On failure: Returns [UserLookupError.NotFound] if no user exists
 * - On failure: Returns [UserLookupError.DatabaseError] if query fails
 *
 * ## Side Effects
 * - Queries the database
 * - Increments user_lookup metric
 *
 * @param id The unique identifier of the user to retrieve
 * @return [Result]<[User]> containing the user or an error
 * @see findUserCached
 */
fun getUser(id: Long): Result<User> = repository.find(id)
```

## 7. The `!!` Operator

### Anti-Pattern: Forcing Unwrap with `!!`

```kotlin
fun processUser(id: Long) {
    val user = repository.find(id)
    val email = user!!.email  // Crashes on null
    sendEmail(email)
}
```

**Problems:**
- Can throw NullPointerException
- Defeats Kotlin's null safety
- No error message explaining the failure
- Not compositional

### Functional Alternatives

**Option 1: Safe Navigation**
```kotlin
fun processUser(id: Long) {
    val user = repository.find(id)
    user?.email?.let { email ->
        sendEmail(email)
    }
}
```

**Option 2: Early Return**
```kotlin
fun processUser(id: Long): Result<Unit> {
    val user = repository.find(id) ?: return Result.failure(UserNotFound(id))
    return sendEmail(user.email)
}
```

**Option 3: Result Type**
```kotlin
fun processUser(id: Long): Result<Unit> =
    repository.find(id)
        .map { it.email }
        .flatMap { sendEmail(it) }
```

## 8. Statements Instead of Expressions

### Anti-Pattern: Statement-Based Code

```kotlin
fun calculateDiscount(tier: LoyaltyTier): Double {
    var discount = 0.0  // Mutable variable
    if (tier == LoyaltyTier.GOLD) {
        discount = 0.15
    } else if (tier == LoyaltyTier.SILVER) {
        discount = 0.10
    } else {
        discount = 0.05
    }
    return discount
}
```

**Problems:**
- Mutable variable
- Verbose
- Easy to forget to set all branches
- Not an expression

### Functional Alternative

```kotlin
fun calculateDiscount(tier: LoyaltyTier): Double = when (tier) {
    LoyaltyTier.GOLD -> 0.15
    LoyaltyTier.SILVER -> 0.10
    LoyaltyTier.BRONZE -> 0.05
}  // Expression, immutable, exhaustive
```

## 9. Anemic Domain Models

### Anti-Pattern: Data Classes with No Behavior

```kotlin
data class Order(
    val id: Long,
    val items: List<OrderItem>,
    val status: String
)

// Business logic in separate service class
class OrderService {
    fun calculateTotal(order: Order): Money {
        // Logic separated from data
    }

    fun canCancel(order: Order): Boolean {
        // Logic separated from data
    }
}
```

**Problems:**
- Business logic scattered in service classes
- Violates encapsulation
- Difficult to maintain invariants
- Data and behavior are separate

### Functional Alternative

```kotlin
data class Order(
    val id: OrderId,
    val items: List<OrderItem>,
    val status: OrderStatus
) {
    /**
     * Calculates the total price of all items.
     */
    fun total(): Money =
        items.fold(Money(0)) { acc, item -> acc + item.totalPrice() }

    /**
     * Checks if this order can be cancelled.
     */
    fun canBeCancelled(): Boolean =
        status !is OrderStatus.Delivered

    /**
     * Creates a copy with updated status.
     */
    fun withStatus(newStatus: OrderStatus): Order =
        copy(status = newStatus)
}
```

## 10. Ignoring Kotlin's Scope Functions

### Anti-Pattern: Temporary Variables

```kotlin
fun createUser(data: UserData): User {
    val validatedEmail = validateEmail(data.email)
    val user = User(
        id = generateId(),
        email = validatedEmail,
        name = data.name
    )
    logger.info("Created user: ${user.id}")
    return user
}
```

### Functional Alternative

```kotlin
fun createUser(data: UserData): User =
    User(
        id = generateId(),
        email = validateEmail(data.email),
        name = data.name
    ).also { user ->
        logger.info("Created user: ${user.id}")
    }
```

## Summary Table

| Anti-Pattern | Problem | Functional Alternative |
|--------------|---------|----------------------|
| Mutable properties (`var`) | Temporal coupling, not thread-safe | Immutable properties (`val`), `copy()` |
| Nullable return for failure | Hides failure reason | `Result<T>` or sealed error hierarchies |
| Exception-based flow control | Hidden control flow, expensive | `Result<T>`, sealed error types |
| Imperative loops | Verbose, mutable accumulators | `map`, `filter`, `fold`, `reduce` |
| Inheritance for behavior | Tight coupling, inflexible | Higher-order functions, composition |
| Missing KDoc | Poor maintainability | Comprehensive KDoc with contracts |
| `!!` operator | Can crash, defeats null safety | Safe navigation, Result types |
| Statements vs expressions | Mutable state, verbose | Expression-oriented code |
| Anemic domain models | Scattered logic, poor encapsulation | Rich domain models with behavior |
| Temporary variables | Noise, mutable state | Scope functions (`let`, `also`, etc.) |

## Refactoring Checklist

When reviewing code, ask:

- [ ] Are all properties `val` (or is `var` justified)?
- [ ] Are failures represented as Result types or sealed hierarchies?
- [ ] Are exceptions only for truly exceptional cases?
- [ ] Are loops functional (`map`, `filter`, `fold`)?
- [ ] Is behavior composed with functions, not inheritance?
- [ ] Does every public declaration have comprehensive KDoc?
- [ ] Is there any use of `!!` operator?
- [ ] Is code expression-oriented?
- [ ] Do domain models have relevant business logic?
- [ ] Are scope functions used appropriately?
