# Functional Transformations

Examples of transforming data using map, flatMap, fold, and sequences.

## Map: One-to-One Transformations

```kotlin
package com.example.transformations

/**
 * Transforms user entities to DTOs for API responses.
 *
 * Each user is mapped to a DTO containing only public information.
 *
 * @return List of user DTOs
 */
fun List<User>.toUserDTOs(): List<UserDTO> =
    map { user ->
        UserDTO(
            id = user.id,
            name = user.name,
            email = user.email.value
        )
    }

/**
 * Extracts email addresses from users.
 *
 * @return List of email strings
 */
fun List<User>.emails(): List<String> =
    map { it.email.value }

/**
 * Converts monetary amounts to formatted strings.
 *
 * @return List of formatted currency strings
 */
fun List<Money>.formatted(): List<String> =
    map { it.format() }

// Nested map:
val orderTotals: List<Money> = orders.map { order ->
    order.items.map { it.totalPrice() }.fold(Money(0), Money::plus)
}
```

## FlatMap: One-to-Many Transformations

```kotlin
package com.example.transformations

/**
 * Extracts all items from all orders into a single list.
 *
 * @return Flattened list of all order items
 */
fun List<Order>.allItems(): List<OrderItem> =
    flatMap { it.items }

/**
 * Gets all email addresses for users with multiple emails.
 *
 * @return Flattened list of all email addresses
 */
fun List<UserProfile>.allEmails(): List<Email> =
    flatMap { it.emails }

/**
 * Expands each user into a list of their orders.
 *
 * @param orderRepo Repository to fetch orders
 * @return List of all orders for all users
 */
fun List<User>.allOrders(orderRepo: OrderRepository): List<Order> =
    flatMap { user -> orderRepo.findByUser(user.id) }

// FlatMap with filtering:
val activeUserEmails = users
    .filter { it.isActive }
    .flatMap { listOfNotNull(it.email) }
```

## FlatMap with Result: Chaining Operations

```kotlin
package com.example.transformations

/**
 * Validates and processes user registration in a pipeline.
 *
 * Each step depends on the previous step's success. The pipeline
 * short-circuits on the first failure.
 *
 * @param credentials The registration credentials
 * @return [Result]<[User]> after full registration pipeline
 */
fun registerUser(credentials: RegistrationCredentials): Result<User> =
    validateEmail(credentials.email)
        .flatMap { email ->
            validatePassword(credentials.password)
                .flatMap { password ->
                    createUser(email, password, credentials.name)
                        .flatMap { user ->
                            sendWelcomeEmail(user)
                                .map { user }  // Return user after email sent
                        }
                }
        }

/**
 * Processes payment and creates order.
 *
 * @param cart The shopping cart
 * @param paymentMethod The payment method to use
 * @return [Result]<[Order]> if payment and order creation succeed
 */
fun checkout(cart: Cart, paymentMethod: PaymentMethod): Result<Order> =
    validateCart(cart)
        .flatMap { validCart ->
            processPayment(validCart.total(), paymentMethod)
                .flatMap { transaction ->
                    createOrder(validCart, transaction)
                }
        }
```

## Filter: Selecting Elements

```kotlin
package com.example.transformations

/**
 * Filters users by role.
 *
 * @param role The role to filter by
 * @return List containing only users with specified role
 */
fun List<User>.withRole(role: Role): List<User> =
    filter { it.role == role }

/**
 * Filters to only active users.
 *
 * @return List of active users
 */
fun List<User>.activeUsers(): List<User> =
    filter { it.isActive }

/**
 * Filters orders above a minimum amount.
 *
 * @param minimumAmount The minimum order total
 * @return Orders with total >= [minimumAmount]
 */
fun List<Order>.aboveAmount(minimumAmount: Money): List<Order> =
    filter { it.total() >= minimumAmount }

// Chaining filters:
val recentHighValueOrders = orders
    .filter { it.createdAt.isWithin(Duration.parse("7d")) }
    .filter { it.total() >= Money(10000) }
    .sortedByDescending { it.total() }
```

## Fold: Aggregating to Single Value

```kotlin
package com.example.transformations

/**
 * Calculates the total of all order amounts.
 *
 * @return Sum of all order totals
 */
fun List<Order>.totalRevenue(): Money =
    fold(Money(0)) { acc, order -> acc + order.total() }

/**
 * Counts users by role.
 *
 * @return Map of role to count
 */
fun List<User>.countByRole(): Map<Role, Int> =
    fold(emptyMap()) { acc, user ->
        acc + (user.role to (acc[user.role] ?: 0) + 1)
    }

/**
 * Combines all validation errors into a single list.
 *
 * @return Concatenated list of all errors
 */
fun List<Result<*>>.allErrors(): List<Throwable> =
    fold(emptyList()) { acc, result ->
        result.fold(
            onSuccess = { acc },
            onFailure = { error -> acc + error }
        )
    }

/**
 * Builds a summary string from multiple parts.
 *
 * @return Combined summary
 */
fun List<OrderItem>.toSummary(): String =
    fold("") { acc, item ->
        "$acc\n- ${item.quantity}x ${item.productId}: ${item.totalPrice().format()}"
    }.trim()
```

## Reduce: Folding Without Initial Value

```kotlin
package com.example.transformations

/**
 * Finds the maximum money amount in the list.
 *
 * @return Maximum amount, or null if empty
 */
fun List<Money>.max(): Money? =
    reduceOrNull { acc, money -> if (money > acc) money else acc }

/**
 * Combines all orders into a single mega-order.
 *
 * Useful for batch processing.
 *
 * @return Combined order with all items
 */
fun List<Order>.mergeOrders(): Order? =
    reduceOrNull { acc, order ->
        acc.copy(items = acc.items + order.items)
    }
```

## GroupBy: Partitioning into Groups

```kotlin
package com.example.transformations

/**
 * Groups users by their role.
 *
 * @return Map from role to list of users with that role
 */
fun List<User>.byRole(): Map<Role, List<User>> =
    groupBy { it.role }

/**
 * Groups orders by their status.
 *
 * @return Map from status type to orders in that status
 */
fun List<Order>.byStatus(): Map<OrderStatus, List<Order>> =
    groupBy { it.status }

/**
 * Groups orders by date.
 *
 * @return Map from date to orders placed on that date
 */
fun List<Order>.byDate(): Map<LocalDate, List<Order>> =
    groupBy { it.createdAt.toLocalDateTime(TimeZone.UTC).date }

// Using groupingBy for counting:
val orderCountByStatus = orders
    .groupingBy { it.status }
    .eachCount()
```

## Partition: Splitting into Two Groups

```kotlin
package com.example.transformations

/**
 * Partitions users into active and inactive.
 *
 * @return Pair of (active users, inactive users)
 */
fun List<User>.partitionByActivity(): Pair<List<User>, List<User>> =
    partition { it.isActive }

/**
 * Partitions results into successes and failures.
 *
 * @return Pair of (successful results, failures with errors)
 */
fun <T> List<Result<T>>.partitionResults(): Pair<List<T>, List<Throwable>> {
    val successes = mutableListOf<T>()
    val failures = mutableListOf<Throwable>()

    forEach { result ->
        result.fold(
            onSuccess = { successes.add(it) },
            onFailure = { failures.add(it) }
        )
    }

    return successes to failures
}

// Usage:
val (active, inactive) = users.partitionByActivity()
val (succeeded, failed) = results.partitionResults()
```

## Sequences for Lazy Evaluation

```kotlin
package com.example.transformations

/**
 * Finds first 10 active admin users, evaluated lazily.
 *
 * Processing stops as soon as 10 matches are found.
 *
 * @return List of up to 10 active admin users
 */
fun List<User>.first10ActiveAdmins(): List<User> =
    asSequence()
        .filter { it.isActive }
        .filter { it.isAdmin() }
        .take(10)
        .toList()

/**
 * Processes large list of orders lazily.
 *
 * Avoids creating intermediate collections.
 *
 * @return Sequence of processed orders
 */
fun List<Order>.processLazily(): Sequence<ProcessedOrder> =
    asSequence()
        .filter { it.status is OrderStatus.Pending }
        .map { order -> validateOrder(order) }
        .filterNotNull()
        .map { validOrder -> processOrder(validOrder) }

/**
 * Generates infinite sequence of retry delays.
 *
 * @param initial Initial delay in milliseconds
 * @param max Maximum delay (caps exponential growth)
 * @return Infinite sequence of delay values
 */
fun exponentialBackoff(initial: Long = 100, max: Long = 10000): Sequence<Long> =
    generateSequence(initial) { delay ->
        (delay * 2).coerceAtMost(max)
    }

// Usage:
val delays = exponentialBackoff().take(5).toList()
// [100, 200, 400, 800, 1600]
```

## Zip: Combining Two Lists

```kotlin
package com.example.transformations

/**
 * Combines users with their order counts.
 *
 * @param orderCounts Map of user ID to order count
 * @return List of user/count pairs
 */
fun List<User>.withOrderCounts(orderCounts: Map<UserId, Int>): List<Pair<User, Int>> =
    map { user -> user to (orderCounts[user.id] ?: 0) }

/**
 * Zips two lists together.
 *
 * @param other The list to zip with
 * @return List of pairs, length is minimum of the two lists
 */
fun <A, B> List<A>.zipWith(other: List<B>): List<Pair<A, B>> =
    zip(other)

/**
 * Zips with custom combiner.
 *
 * @param other The list to zip with
 * @param transform Function to combine elements
 * @return List of combined results
 */
fun <A, B, R> List<A>.zipWith(other: List<B>, transform: (A, B) -> R): List<R> =
    zip(other, transform)

// Usage:
val names = listOf("Alice", "Bob", "Charlie")
val ages = listOf(25, 30, 35)
val people = names.zipWith(ages) { name, age ->
    Person(name, age)
}
```

## Windowed: Sliding Windows

```kotlin
package com.example.transformations

/**
 * Calculates moving average of order totals.
 *
 * @param windowSize Size of the moving window
 * @return List of average totals for each window
 */
fun List<Order>.movingAverage(windowSize: Int): List<Money> =
    map { it.total() }
        .windowed(windowSize) { window ->
            val sum = window.fold(Money(0), Money::plus)
            Money(sum.cents / window.size)
        }

/**
 * Finds consecutive pairs of elements.
 *
 * @return List of consecutive pairs
 */
fun <T> List<T>.consecutivePairs(): List<Pair<T, T>> =
    windowed(size = 2) { (a, b) -> a to b }

// Usage:
val orders = listOf(order1, order2, order3, order4)
val averages = orders.movingAverage(windowSize = 3)
```

## Distinct and DistinctBy

```kotlin
package com.example.transformations

/**
 * Gets unique email addresses from users.
 *
 * @return List of unique emails
 */
fun List<User>.uniqueEmails(): List<Email> =
    map { it.email }
        .distinct()

/**
 * Gets users with unique email domains.
 *
 * For users with same domain, keeps first occurrence.
 *
 * @return List of users with distinct email domains
 */
fun List<User>.uniqueByDomain(): List<User> =
    distinctBy { it.email.value.substringAfter("@") }

/**
 * Removes duplicate orders by ID.
 *
 * @return List of orders with unique IDs
 */
fun List<Order>.removeDuplicates(): List<Order> =
    distinctBy { it.id }
```

## Chunked: Split into Fixed-Size Batches

```kotlin
package com.example.transformations

/**
 * Splits users into batches for processing.
 *
 * @param batchSize Size of each batch
 * @return List of user batches
 */
fun List<User>.toBatches(batchSize: Int): List<List<User>> =
    chunked(batchSize)

/**
 * Processes orders in batches.
 *
 * @param batchSize Number of orders per batch
 * @param process Function to process each batch
 * @return List of batch processing results
 */
fun <R> List<Order>.processBatches(
    batchSize: Int,
    process: (List<Order>) -> R
): List<R> =
    chunked(batchSize, process)

// Usage:
val batches = orders.toBatches(batchSize = 100)
batches.forEach { batch ->
    database.saveBatch(batch)
}
```

## Combining Multiple Transformations

```kotlin
package com.example.transformations

/**
 * Complete data transformation pipeline.
 *
 * Demonstrates chaining multiple transformation operations.
 *
 * @param minAmount Minimum order amount to include
 * @return Summary report grouped by user role
 */
fun List<Order>.generateReport(minAmount: Money): Map<Role, OrderSummary> {
    return this
        .filter { it.total() >= minAmount }  // Filter by amount
        .filter { it.status is OrderStatus.Delivered }  // Only delivered
        .map { order ->  // Map to report entries
            order.userId to order.total()
        }
        .groupBy { (userId, _) -> userId }  // Group by user
        .mapValues { (_, orders) ->  // Calculate totals per user
            orders.fold(Money(0)) { acc, (_, amount) -> acc + amount }
        }
        .entries
        .groupBy { (userId, _) ->  // Group by user role
            userRepository.find(userId)?.role ?: Role.USER
        }
        .mapValues { (_, entries) ->  // Create summaries
            OrderSummary(
                totalOrders = entries.size,
                totalRevenue = entries.fold(Money(0)) { acc, (_, amount) ->
                    acc + amount
                }
            )
        }
}

data class OrderSummary(
    val totalOrders: Int,
    val totalRevenue: Money
)
```

## Key Takeaways

1. **map**: One-to-one transformation
2. **flatMap**: One-to-many transformation, flattens results
3. **filter**: Select elements matching predicate
4. **fold**: Aggregate to single value with initial accumulator
5. **reduce**: Aggregate without initial value
6. **groupBy**: Partition into map of groups
7. **partition**: Split into two groups (true/false)
8. **Sequence**: Lazy evaluation for efficiency
9. **zip**: Combine two lists element-wise
10. **windowed**: Create sliding windows
11. **distinct**: Remove duplicates
12. **chunked**: Split into fixed-size batches

All transformations create **new collections** without mutating the original.
