# Higher-Order Functions

Examples of using functions as first-class values for behavior composition and abstraction.

## Basic Higher-Order Function

```kotlin
package com.example.utils

/**
 * Executes an operation with timing measurement.
 *
 * This higher-order function abstracts the timing pattern,
 * separating the concern of measurement from business logic.
 *
 * ## Side Effects
 * - Logs execution time
 *
 * @param T The return type of the operation
 * @param operationName Name of the operation being timed (for logging)
 * @param operation The operation to execute and time
 * @return The result of [operation]
 */
inline fun <T> measureTime(
    operationName: String,
    operation: () -> T
): T {
    val startTime = System.currentTimeMillis()
    return operation().also {
        val duration = System.currentTimeMillis() - startTime
        logger.info { "$operationName took ${duration}ms" }
    }
}

// Usage:
val users = measureTime("User query") {
    repository.findAllActiveUsers()
}
```

## Retry Pattern with Higher-Order Function

```kotlin
package com.example.utils

import kotlinx.coroutines.delay
import kotlin.time.Duration
import kotlin.time.Duration.Companion.milliseconds

/**
 * Executes an operation with automatic retry logic for failures.
 *
 * This higher-order function abstracts the retry pattern with
 * exponential backoff, making any operation retryable.
 *
 * ## Preconditions
 * - [maxAttempts] must be > 0
 *
 * ## Postconditions
 * - On success (any attempt): Returns successful [Result]
 * - On failure (all attempts): Returns last [Result] failure
 *
 * ## Side Effects
 * - Suspends between retry attempts
 * - Logs retry attempts
 *
 * @param T The type of value returned by the operation
 * @param maxAttempts Maximum number of attempts (must be > 0)
 * @param initialDelay Initial delay between attempts
 * @param maxDelay Maximum delay between attempts (caps exponential growth)
 * @param operation The operation to execute (potentially failing)
 * @return [Result]<[T]> from successful attempt or last failure
 */
suspend fun <T> withRetry(
    maxAttempts: Int = 3,
    initialDelay: Duration = 100.milliseconds,
    maxDelay: Duration = 10.seconds,
    operation: suspend () -> Result<T>
): Result<T> {
    require(maxAttempts > 0) { "maxAttempts must be positive" }

    var currentDelay = initialDelay

    repeat(maxAttempts - 1) { attempt ->
        val result = operation()

        result.onSuccess {
            logger.debug { "Operation succeeded on attempt ${attempt + 1}" }
            return Result.success(it)
        }

        logger.warn { "Operation failed on attempt ${attempt + 1}, retrying in $currentDelay" }
        delay(currentDelay)

        currentDelay = (currentDelay * 2).coerceAtMost(maxDelay)
    }

    logger.warn { "Final attempt (attempt $maxAttempts)" }
    return operation()
}

// Usage:
val user = withRetry(maxAttempts = 3) {
    repository.findById(userId)
}
```

## Transaction Pattern

```kotlin
package com.example.database

/**
 * Executes an operation within a database transaction.
 *
 * Automatically commits on success and rolls back on failure.
 *
 * ## Postconditions
 * - On success: Transaction is committed, returns operation result
 * - On failure: Transaction is rolled back, returns error
 *
 * ## Side Effects
 * - Begins database transaction
 * - Commits or rolls back transaction
 * - Logs transaction lifecycle
 *
 * @param T The return type of the transactional operation
 * @param operation The operation to execute within transaction
 * @return [Result]<[T]> containing operation result or error
 */
inline fun <T> withTransaction(
    operation: (TransactionContext) -> Result<T>
): Result<T> {
    val transaction = beginTransaction()

    return try {
        val result = operation(transaction)

        result.onSuccess {
            transaction.commit()
            logger.debug { "Transaction committed successfully" }
        }.onFailure { error ->
            transaction.rollback()
            logger.warn { "Transaction rolled back due to: $error" }
        }

        result
    } catch (e: Exception) {
        transaction.rollback()
        logger.error(e) { "Transaction rolled back due to exception" }
        Result.failure(e)
    }
}

// Usage:
val result = withTransaction { tx ->
    userRepository.save(user, tx)
        .flatMap { savedUser ->
            orderRepository.save(order, tx)
                .map { savedOrder ->
                    savedUser to savedOrder
                }
        }
}
```

## Resource Management Pattern

```kotlin
package com.example.utils

/**
 * Executes an operation with automatic resource cleanup.
 *
 * Ensures [resource] is properly closed even if [operation] throws.
 * This is similar to Java's try-with-resources or Kotlin's [use],
 * but returns a [Result] instead of throwing.
 *
 * @param R The resource type
 * @param T The return type of the operation
 * @param resource The resource to manage
 * @param operation The operation to execute with the resource
 * @return [Result]<[T]> containing operation result or error
 */
inline fun <R : AutoCloseable, T> withResource(
    resource: R,
    operation: (R) -> Result<T>
): Result<T> =
    try {
        resource.use { operation(it) }
    } catch (e: Exception) {
        Result.failure(e)
    }

// Usage:
val data = withResource(FileInputStream("data.json")) { stream ->
    parseJson(stream)
}
```

## Strategy Pattern with Functions

```kotlin
package com.example.service

/**
 * Pricing strategies for products.
 *
 * Instead of an inheritance hierarchy, we use function types
 * to represent different pricing strategies.
 */
typealias PricingStrategy = (basePrice: Money, quantity: Int) -> Money

/**
 * No discount—returns base price multiplied by quantity.
 */
val standardPricing: PricingStrategy = { basePrice, quantity ->
    basePrice * quantity
}

/**
 * Bulk discount—10% off for quantities >= 10.
 */
val bulkPricing: PricingStrategy = { basePrice, quantity ->
    val total = basePrice * quantity
    if (quantity >= 10) {
        Money((total.cents * 0.9).toLong())
    } else {
        total
    }
}

/**
 * Tiered discount based on quantity.
 */
val tieredPricing: PricingStrategy = { basePrice, quantity ->
    val total = basePrice * quantity
    val discount = when {
        quantity >= 100 -> 0.20  // 20% off
        quantity >= 50 -> 0.15   // 15% off
        quantity >= 10 -> 0.10   // 10% off
        else -> 0.0
    }
    Money((total.cents * (1.0 - discount)).toLong())
}

/**
 * Calculates the final price using the specified strategy.
 *
 * @param basePrice Price per unit
 * @param quantity Number of units
 * @param strategy Pricing strategy to apply
 * @return Final calculated price
 */
fun calculatePrice(
    basePrice: Money,
    quantity: Int,
    strategy: PricingStrategy = standardPricing
): Money = strategy(basePrice, quantity)

// Usage:
val price1 = calculatePrice(Money(1000), 5, standardPricing)
val price2 = calculatePrice(Money(1000), 15, bulkPricing)
val price3 = calculatePrice(Money(1000), 100, tieredPricing)
```

## Function Composition

```kotlin
package com.example.utils

/**
 * Composes two functions: `g(f(x)`.
 *
 * This operator allows chaining functions together to create
 * new functions.
 *
 * @param A Input type
 * @param B Intermediate type
 * @param C Output type
 * @param g The outer function
 * @return Composed function
 */
infix fun <A, B, C> ((B) -> C).compose(g: (A) -> B): (A) -> C =
    { a -> this(g(a)) }

/**
 * Pipes a value through a function: `x.pipe(f) == f(x)`.
 *
 * This provides a more natural left-to-right reading order.
 *
 * @param T Input type
 * @param R Output type
 * @param f The function to apply
 * @return Result of applying [f] to this value
 */
infix fun <T, R> T.pipe(f: (T) -> R): R = f(this)

// Usage examples:

// Composing functions
val toLowerCase: (String) -> String = { it.lowercase() }
val trim: (String) -> String = { it.trim() }
val normalize = toLowerCase compose trim

val result1 = normalize("  HELLO  ")  // "hello"

// Piping values through functions
val result2 = "  HELLO  "
    .pipe(trim)
    .pipe(toLowerCase)  // "hello"

/**
 * Validates and normalizes an email address.
 */
fun validateAndNormalizeEmail(email: String): Result<Email> =
    email
        .pipe(String::trim)
        .pipe(String::lowercase)
        .pipe { Email.of(it) }
```

## Currying and Partial Application

```kotlin
package com.example.utils

/**
 * Creates a logger function configured for a specific component.
 *
 * This demonstrates partial application—fixing some arguments
 * of a function to create a new function with fewer parameters.
 *
 * @param component The component name for logging
 * @return A partially applied logging function
 */
fun createLogger(component: String): (level: LogLevel, message: String) -> Unit =
    { level, message ->
        val timestamp = Clock.System.now()
        println("[$timestamp] [$component] [$level] $message")
    }

// Usage:
val userLogger = createLogger("UserService")
val orderLogger = createLogger("OrderService")

userLogger(LogLevel.INFO, "User created")
orderLogger(LogLevel.WARN, "Low inventory")

/**
 * Creates a pricing calculator with a fixed strategy.
 *
 * @param strategy The pricing strategy to use
 * @return A function that calculates prices using [strategy]
 */
fun createPriceCalculator(strategy: PricingStrategy): (Money, Int) -> Money =
    { basePrice, quantity ->
        calculatePrice(basePrice, quantity, strategy)
    }

// Usage:
val bulkCalculator = createPriceCalculator(bulkPricing)
val tieredCalculator = createPriceCalculator(tieredPricing)

val price1 = bulkCalculator(Money(1000), 15)
val price2 = tieredCalculator(Money(1000), 100)
```

## Memoization Pattern

```kotlin
package com.example.utils

/**
 * Creates a memoized version of a function.
 *
 * The memoized function caches results for each input,
 * avoiding redundant computation.
 *
 * ## Side Effects
 * - Maintains internal cache (thread-unsafe)
 *
 * @param A Input type (must be suitable for Map key)
 * @param R Output type
 * @param f The function to memoize
 * @return Memoized version of [f]
 */
fun <A, R> memoize(f: (A) -> R): (A) -> R {
    val cache = mutableMapOf<A, R>()

    return { input ->
        cache.getOrPut(input) {
            logger.debug { "Cache miss for $input, computing..." }
            f(input)
        }
    }
}

// Usage:

/**
 * Expensive computation (for demonstration).
 */
fun expensiveComputation(n: Int): Int {
    Thread.sleep(1000)  // Simulate expensive work
    return n * n
}

val memoizedComputation = memoize(::expensiveComputation)

// First call: takes ~1 second
val result1 = memoizedComputation(10)  // Computes

// Second call with same input: instant
val result2 = memoizedComputation(10)  // Cached
```

## Collection Processing with Higher-Order Functions

```kotlin
package com.example.service

/**
 * Processes a list of items with error handling.
 *
 * Continues processing all items even if some fail, collecting
 * both successes and failures.
 *
 * @param T Input type
 * @param R Output type
 * @param items Items to process
 * @param process Processing function
 * @return Pair of successes and failures
 */
fun <T, R> processAll(
    items: List<T>,
    process: (T) -> Result<R>
): Pair<List<R>, List<Pair<T, Throwable>>> {
    val successes = mutableListOf<R>()
    val failures = mutableListOf<Pair<T, Throwable>>()

    items.forEach { item ->
        process(item).fold(
            onSuccess = { successes.add(it) },
            onFailure = { error -> failures.add(item to error) }
        )
    }

    return successes to failures
}

// Usage:
val (processedOrders, failedOrders) = processAll(orders) { order ->
    validateOrder(order)
        .flatMap { processPayment(it) }
        .flatMap { shipOrder(it) }
}

logger.info { "Processed ${processedOrders.size} orders successfully" }
logger.warn { "Failed to process ${failedOrders.size} orders" }
```

## Key Takeaways

1. **Inline functions** for zero overhead with lambdas
2. **Function types** as strategy pattern replacement
3. **Higher-order functions** abstract common patterns (retry, transaction, timing)
4. **Composition** creates new functions from existing ones
5. **Partial application** creates specialized functions
6. **Memoization** caches expensive computations
7. **Type aliases** make function types readable
8. **No inheritance needed** for behavior composition
