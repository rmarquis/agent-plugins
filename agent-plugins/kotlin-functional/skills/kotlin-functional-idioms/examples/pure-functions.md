# Pure Functions and Side Effect Management

Examples of separating pure business logic from side effects.

## Pure Functions

### Definition
A function is **pure** if:
1. **Deterministic**: Same inputs always produce same outputs
2. **No side effects**: Doesn't modify external state, perform I/O, or depend on external state

### Examples of Pure Functions

```kotlin
package com.example.pure

/**
 * Calculates discount amount (pure function).
 *
 * ## Properties
 * - Deterministic: Same inputs always produce same output
 * - No side effects: No I/O, no state mutation
 * - Referentially transparent: Can replace call with result
 *
 * @param price Original price
 * @param discountPercentage Discount percentage (0.0-1.0)
 * @return Discount amount
 */
fun calculateDiscount(price: Money, discountPercentage: Double): Money {
    require(discountPercentage in 0.0..1.0) { "Discount must be 0.0-1.0" }
    return Money((price.cents * discountPercentage).toLong())
}

/**
 * Validates password strength (pure function).
 *
 * @param password The password to validate
 * @return [Result]<[String]> with validated password or errors
 */
fun validatePassword(password: String): Result<String> {
    val violations = buildList {
        if (password.length < 8) add("Must be at least 8 characters")
        if (!password.any { it.isUpperCase() }) add("Must contain uppercase")
        if (!password.any { it.isLowerCase() }) add("Must contain lowercase")
        if (!password.any { it.isDigit() }) add("Must contain digit")
    }

    return if (violations.isEmpty()) {
        Result.success(password)
    } else {
        Result.failure(WeakPassword(violations))
    }
}

/**
 * Determines user eligibility for feature (pure function).
 *
 * @param user The user to check
 * @param feature The feature to check eligibility for
 * @return `true` if user is eligible
 */
fun isEligibleFor(user: User, feature: Feature): Boolean = when (feature) {
    Feature.PREMIUM_SUPPORT -> user.role == Role.ADMIN || user.isPremium
    Feature.BETA_FEATURES -> user.isBetaTester
    Feature.ANALYTICS -> user.role in setOf(Role.ADMIN, Role.MODERATOR)
}

/**
 * Calculates final price with tax and discount (pure function).
 *
 * @param basePrice Original price
 * @param discount Discount amount
 * @param taxRate Tax rate (e.g., 0.08 for 8%)
 * @return Final price after discount and tax
 */
fun calculateFinalPrice(
    basePrice: Money,
    discount: Money,
    taxRate: Double
): Money =
    (basePrice - discount).let { discounted ->
        Money((discounted.cents * (1.0 + taxRate)).toLong())
    }
```

### Benefits of Pure Functions

```kotlin
// ✅ Easy to test (no mocks needed)
@Test
fun `test discount calculation`() {
    val result = calculateDiscount(Money(10000), 0.10)
    assertEquals(Money(1000), result)
}

// ✅ Can memoize results
val memoizedCalculateDiscount = memoize(::calculateDiscount)

// ✅ Safe to run in parallel
val discounts = prices.parallelStream()
    .map { calculateDiscount(it, 0.10) }
    .collect(toList())

// ✅ Referentially transparent (can replace call with result)
val discount1 = calculateDiscount(Money(10000), 0.10)
val discount2 = Money(1000)  // Same as discount1
```

## Examples of Impure Functions

```kotlin
package com.example.impure

// ❌ Impure: Reads external state (current time)
fun isUserSessionExpired(user: User): Boolean =
    user.lastActivity < Clock.System.now() - Duration.parse("1h")

// ❌ Impure: Modifies external state
var requestCount = 0
fun incrementRequestCount() {
    requestCount++  // Side effect
}

// ❌ Impure: Performs I/O
fun loadUserFromDatabase(id: UserId): User =
    database.query("SELECT * FROM users WHERE id = ?", id)

// ❌ Impure: Non-deterministic (random)
fun generateSessionToken(): String =
    UUID.randomUUID().toString()
```

## Pattern: Separate Pure Logic from Side Effects

### Example 1: Order Processing

```kotlin
package com.example.patterns

// ✅ Pure: Business logic
fun calculateOrderTotal(items: List<OrderItem>): Money =
    items.fold(Money(0)) { acc, item -> acc + item.totalPrice() }

// ✅ Pure: Validation logic
fun validateOrder(order: Order): Result<Order> = when {
    order.items.isEmpty() ->
        Result.failure(EmptyOrder)

    order.total() <= Money(0) ->
        Result.failure(InvalidTotal(order.total()))

    else ->
        Result.success(order)
}

// ❌ Impure: I/O operations
suspend fun saveOrder(order: Order): Result<Order> {
    val total = calculateOrderTotal(order.items)  // Pure
    val validated = validateOrder(order.copy(total = total))  // Pure

    return validated.flatMap { validOrder ->
        database.save(validOrder)  // Impure (I/O)
    }
}

// ❌ Impure: Sends email
suspend fun confirmOrder(order: Order): Result<Unit> {
    return emailService.send(
        to = order.userEmail,
        subject = "Order Confirmation",
        body = formatOrderConfirmation(order)  // Pure
    )
}
```

### Example 2: User Registration

```kotlin
package com.example.patterns

// ✅ Pure: Validation
data class ValidatedCredentials(
    val email: Email,
    val password: String,
    val name: String
)

fun validateCredentials(
    email: String,
    password: String,
    name: String
): Result<ValidatedCredentials> =
    validateEmail(email)
        .flatMap { validEmail ->
            validatePassword(password)
                .flatMap { validPassword ->
                    validateName(name)
                        .map { validName ->
                            ValidatedCredentials(validEmail, validPassword, validName)
                        }
                }
        }

// ✅ Pure: Domain model construction
fun createUserFromCredentials(
    credentials: ValidatedCredentials,
    id: UserId,
    timestamp: Instant
): User = User(
    id = id,
    email = credentials.email,
    name = credentials.name,
    role = Role.USER,
    createdAt = timestamp
)

// ❌ Impure: Complete registration with side effects
suspend fun registerUser(
    email: String,
    password: String,
    name: String
): Result<User> {
    // Pure validation
    val validated = validateCredentials(email, password, name)
        .getOrElse { return Result.failure(it) }

    // Pure domain model creation
    val user = createUserFromCredentials(
        credentials = validated,
        id = UserId.generate(),  // Impure (random)
        timestamp = Clock.System.now()  // Impure (current time)
    )

    // Impure: I/O operations
    return userRepository.save(user)  // Database write
        .flatMap { savedUser ->
            emailService.sendWelcome(savedUser.email)  // Email send
                .map { savedUser }
        }
        .onSuccess { savedUser ->
            metrics.increment("users.registered")  // Metrics
            logger.info("User registered: ${savedUser.id}")  // Logging
        }
}
```

## Pattern: Push Side Effects to the Edges

```kotlin
package com.example.patterns

/**
 * Core domain logic (pure).
 *
 * This layer contains only pure functions and business rules.
 */
object Domain {
    fun calculateDiscount(user: User, items: List<OrderItem>): Money {
        val total = items.sumOf { it.totalPrice().cents }
        val discountRate = when {
            user.isPremium -> 0.20
            total >= 10000 -> 0.10
            else -> 0.0
        }
        return Money((total * discountRate).toLong())
    }

    fun createOrder(
        userId: UserId,
        items: List<OrderItem>,
        discount: Money,
        timestamp: Instant
    ): Order = Order(
        id = OrderId.generate(),
        userId = userId,
        items = items,
        discount = discount,
        status = OrderStatus.Pending(timestamp),
        createdAt = timestamp
    )
}

/**
 * Application layer (impure).
 *
 * This layer coordinates I/O and calls pure domain logic.
 */
class OrderService(
    private val userRepository: UserRepository,
    private val orderRepository: OrderRepository,
    private val emailService: EmailService
) {
    suspend fun placeOrder(
        userId: UserId,
        items: List<OrderItem>
    ): Result<Order> {
        // Fetch user (impure: I/O)
        val user = userRepository.findById(userId)
            .getOrElse { return Result.failure(it) }

        // Calculate discount (pure: domain logic)
        val discount = Domain.calculateDiscount(user, items)

        // Create order (pure: domain logic)
        val order = Domain.createOrder(
            userId = userId,
            items = items,
            discount = discount,
            timestamp = Clock.System.now()  // Impure: current time
        )

        // Save order (impure: I/O)
        return orderRepository.save(order)
            .onSuccess { savedOrder ->
                // Send confirmation (impure: I/O)
                emailService.sendOrderConfirmation(savedOrder)
            }
    }
}
```

## Pattern: Dependency Injection for Testability

```kotlin
package com.example.patterns

/**
 * Pure business logic with injected dependencies.
 *
 * All side-effecting operations are abstracted as function parameters,
 * making the core logic pure and easy to test.
 */
suspend fun processPayment(
    amount: Money,
    paymentMethod: PaymentMethod,
    // Injected side-effecting operations:
    chargeCard: suspend (Money, PaymentMethod) -> Result<Transaction>,
    saveTransaction: suspend (Transaction) -> Result<Unit>,
    sendReceipt: suspend (Transaction) -> Result<Unit>
): Result<Transaction> {
    // Validate (pure)
    if (amount <= Money(0)) {
        return Result.failure(InvalidAmount(amount))
    }

    // Charge card (impure, but injected)
    val transaction = chargeCard(amount, paymentMethod)
        .getOrElse { return Result.failure(it) }

    // Save (impure, but injected)
    saveTransaction(transaction)
        .getOrElse { return Result.failure(it) }

    // Send receipt (impure, but injected)
    sendReceipt(transaction)

    return Result.success(transaction)
}

// Testing is easy with mock functions:
@Test
fun `test payment processing`() = runBlocking {
    val mockCharge: suspend (Money, PaymentMethod) -> Result<Transaction> = { amount, _ ->
        Result.success(Transaction(TransactionId("123"), amount))
    }
    val mockSave: suspend (Transaction) -> Result<Unit> = { Result.success(Unit) }
    val mockSend: suspend (Transaction) -> Result<Unit> = { Result.success(Unit) }

    val result = processPayment(
        amount = Money(1000),
        paymentMethod = testCard,
        chargeCard = mockCharge,
        saveTransaction = mockSave,
        sendReceipt = mockSend
    )

    assertTrue(result.isSuccess)
}
```

## Pattern: Command Pattern for Deferred Side Effects

```kotlin
package com.example.patterns

/**
 * Represents a side effect to be executed.
 *
 * Commands describe what to do without doing it immediately.
 */
sealed interface Command {
    data class SendEmail(val to: Email, val subject: String, val body: String) : Command
    data class SaveToDatabase(val entity: Any) : Command
    data class LogMessage(val level: LogLevel, val message: String) : Command
    data class IncrementMetric(val name: String) : Command
}

/**
 * Pure function that returns commands instead of performing side effects.
 *
 * @param order The order to process
 * @return Pair of result and list of side effect commands
 */
fun processOrder(order: Order): Pair<Result<Order>, List<Command>> {
    // Pure validation
    val validated = validateOrder(order)
        .getOrElse { error ->
            return Result.failure(error) to listOf(
                Command.LogMessage(LogLevel.WARN, "Order validation failed: $error")
            )
        }

    // Pure transformation
    val processed = validated.copy(status = OrderStatus.Processing(Clock.System.now()))

    // Return result and commands to execute
    return Result.success(processed) to listOf(
        Command.SaveToDatabase(processed),
        Command.SendEmail(
            to = validated.userEmail,
            subject = "Order Processing",
            body = "Your order is being processed"
        ),
        Command.IncrementMetric("orders.processing"),
        Command.LogMessage(LogLevel.INFO, "Order ${processed.id} processing started")
    )
}

/**
 * Executor that runs commands (impure).
 */
class CommandExecutor(
    private val database: Database,
    private val emailService: EmailService,
    private val metrics: Metrics,
    private val logger: Logger
) {
    suspend fun execute(commands: List<Command>) {
        commands.forEach { command ->
            when (command) {
                is Command.SendEmail -> emailService.send(command.to, command.subject, command.body)
                is Command.SaveToDatabase -> database.save(command.entity)
                is Command.LogMessage -> logger.log(command.level, command.message)
                is Command.IncrementMetric -> metrics.increment(command.name)
            }
        }
    }
}

// Usage:
val (result, commands) = processOrder(order)  // Pure
if (result.isSuccess) {
    executor.execute(commands)  // Impure
}
```

## Key Principles

1. **Separate pure logic from side effects**
   - Pure functions for business rules and calculations
   - Impure functions for I/O at the edges

2. **Push side effects to the boundaries**
   - Core domain is pure
   - Application/infrastructure layers handle I/O

3. **Inject side-effecting operations**
   - Pass functions as parameters
   - Makes testing easy with mock implementations

4. **Use command pattern for deferred execution**
   - Pure functions return descriptions of side effects
   - Executor runs commands separately

5. **Document purity in KDoc**
   - Mark pure functions explicitly
   - Document side effects for impure functions

## Benefits

- **Testability**: Pure functions need no mocks
- **Reasoning**: Pure code is easier to understand
- **Composability**: Pure functions compose naturally
- **Parallelism**: Pure functions are thread-safe
- **Memoization**: Pure functions can be cached
- **Refactoring**: Pure code is safer to change
