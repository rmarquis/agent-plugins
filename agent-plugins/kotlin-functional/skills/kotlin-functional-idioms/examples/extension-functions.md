# Extension Functions

Examples of using extension functions for domain operations and readable APIs.

## Domain Extensions for Collections

```kotlin
package com.example.extensions

/**
 * Filters this list to only users with the specified role.
 *
 * This is a pure function—does not modify the original list.
 *
 * @param role The role to filter by
 * @return New list containing only users with [role]
 */
fun List<User>.withRole(role: Role): List<User> =
    filter { it.role == role }

/**
 * Filters this list to only active users.
 *
 * @return New list containing only users where [User.isActive] is true
 */
fun List<User>.active(): List<User> =
    filter { it.isActive }

/**
 * Checks if this list contains any administrators.
 *
 * @return `true` if at least one user has [Role.ADMIN]
 */
fun List<User>.hasAdmins(): Boolean =
    any { it.isAdmin() }

/**
 * Groups users by their role.
 *
 * @return Map where keys are [Role] values and values are lists of users
 */
fun List<User>.byRole(): Map<Role, List<User>> =
    groupBy { it.role }

/**
 * Sorts users by name in ascending order.
 *
 * @return New list sorted alphabetically by [User.name]
 */
fun List<User>.sortedByName(): List<User> =
    sortedBy { it.name }

/**
 * Finds the first user with the specified email.
 *
 * @param email The email to search for
 * @return [Result]<[User]> containing matching user or [UserNotFound]
 */
fun List<User>.findByEmail(email: Email): Result<User> =
    firstOrNull { it.email == email }
        ?.let { Result.success(it) }
        ?: Result.failure(UserNotFound(email))

// Usage:
val admins = users
    .active()
    .withRole(Role.ADMIN)
    .sortedByName()

val hasAnyAdmins = users.hasAdmins()

val usersByRole = users.byRole()
```

## Fluent Query DSL with Extensions

```kotlin
package com.example.extensions

/**
 * Starts a fluent query on this list of users.
 *
 * @return [UserQuery] wrapper enabling fluent chaining
 */
fun List<User>.query(): UserQuery = UserQuery(this)

/**
 * Fluent query builder for users.
 *
 * Provides a readable, chainable API for filtering and transforming
 * user lists. All operations are lazy until a terminal operation
 * ([toList], [first], etc.) is called.
 *
 * ## Usage Example
 * ```kotlin
 * val result = users.query()
 *     .active()
 *     .withRole(Role.ADMIN)
 *     .sortedByName()
 *     .take(10)
 * ```
 *
 * @property users The underlying user list
 */
class UserQuery internal constructor(private val users: List<User>) {
    /**
     * Filters to only active users.
     *
     * @return New query with active filter applied
     */
    fun active(): UserQuery = UserQuery(users.filter { it.isActive })

    /**
     * Filters to users with specified role.
     *
     * @param role The role to filter by
     * @return New query with role filter applied
     */
    fun withRole(role: Role): UserQuery =
        UserQuery(users.filter { it.role == role })

    /**
     * Filters to users whose name contains the search term.
     *
     * Search is case-insensitive.
     *
     * @param term The search term (case-insensitive)
     * @return New query with name filter applied
     */
    fun nameContains(term: String): UserQuery =
        UserQuery(users.filter { it.name.contains(term, ignoreCase = true) })

    /**
     * Sorts users by name in ascending order.
     *
     * @return New query with sort applied
     */
    fun sortedByName(): UserQuery =
        UserQuery(users.sortedBy { it.name })

    /**
     * Sorts users by creation date, newest first.
     *
     * @return New query with sort applied
     */
    fun sortedByNewest(): UserQuery =
        UserQuery(users.sortedByDescending { it.createdAt })

    /**
     * Takes the first N users from the query.
     *
     * @param n Number of users to take (must be non-negative)
     * @return List of at most [n] users
     */
    fun take(n: Int): List<User> = users.take(n)

    /**
     * Returns all users matching the query.
     *
     * @return List of matching users
     */
    fun toList(): List<User> = users

    /**
     * Returns the first user matching the query.
     *
     * @return [Result]<[User]> containing first match or [NoUsersFound]
     */
    fun first(): Result<User> =
        users.firstOrNull()
            ?.let { Result.success(it) }
            ?: Result.failure(NoUsersFound())

    /**
     * Counts the number of users matching the query.
     *
     * @return Number of matching users
     */
    fun count(): Int = users.size
}

// Usage examples:

// Find first 5 active admins sorted by name
val topAdmins = users.query()
    .active()
    .withRole(Role.ADMIN)
    .sortedByName()
    .take(5)

// Count users whose name contains "alice"
val aliceCount = users.query()
    .nameContains("alice")
    .count()

// Find newest active user
val newestActive = users.query()
    .active()
    .sortedByNewest()
    .first()
```

## Result Extensions for Error Handling

```kotlin
package com.example.extensions

/**
 * Maps the error of this Result to a different error type.
 *
 * Useful for converting low-level errors (e.g., database errors)
 * to domain-specific errors.
 *
 * @param E The source error type
 * @param F The target error type
 * @param transform Function to transform the error
 * @return New [Result] with transformed error type
 */
inline fun <T, E : Throwable, F : Throwable> Result<T>.mapError(
    transform: (E) -> F
): Result<T> = fold(
    onSuccess = { Result.success(it) },
    onFailure = { error ->
        @Suppress("UNCHECKED_CAST")
        Result.failure(transform(error as E))
    }
)

/**
 * Recovers from failure with a default value.
 *
 * @param default Value to return on failure
 * @return Success value or [default] if failed
 */
fun <T> Result<T>.getOrDefault(default: T): T =
    getOrElse { default }

/**
 * Recovers from failure by computing a default value.
 *
 * @param defaultProvider Function to compute default on failure
 * @return Success value or computed default
 */
inline fun <T> Result<T>.getOrDefault(defaultProvider: () -> T): T =
    getOrElse { defaultProvider() }

/**
 * Converts this Result to a nullable value.
 *
 * @return Success value or `null` on failure
 */
fun <T> Result<T>.orNull(): T? = getOrNull()

/**
 * Logs this Result and returns it unchanged.
 *
 * Useful for debugging pipelines without breaking the chain.
 *
 * @param message Message to log
 * @return This Result unchanged
 */
fun <T> Result<T>.alsoLog(message: String): Result<T> = also { result ->
    result.fold(
        onSuccess = { logger.debug { "$message: Success($it)" } },
        onFailure = { logger.warn { "$message: Failure($it)" } }
    )
}

// Usage:

val user = findUser(id)
    .mapError { error -> DomainError.UserLookupFailed(error) }
    .alsoLog("User lookup")
    .getOrDefault { createGuestUser() }
```

## String Extensions for Validation

```kotlin
package com.example.extensions

/**
 * Validates this string as an email address.
 *
 * @return [Result]<[Email]> containing validated email or error
 */
fun String.toEmail(): Result<Email> = Email.of(this)

/**
 * Checks if this string is a valid email format.
 *
 * @return `true` if valid email format
 */
fun String.isValidEmail(): Boolean =
    matches(Regex("""^[\w.+-]+@[\w.-]+\.[a-zA-Z]{2,}$"""))

/**
 * Normalizes this string for comparison.
 *
 * Trims whitespace and converts to lowercase.
 *
 * @return Normalized string
 */
fun String.normalize(): String =
    trim().lowercase()

/**
 * Checks if this string is blank or contains only whitespace.
 *
 * More explicit than checking [isBlank] directly.
 *
 * @return `true` if blank or whitespace-only
 */
fun String.isBlankOrEmpty(): Boolean =
    isBlank()

/**
 * Truncates this string to maximum length with ellipsis.
 *
 * @param maxLength Maximum length (must be at least 3 for ellipsis)
 * @return Truncated string with "..." if longer than [maxLength]
 */
fun String.truncate(maxLength: Int): String {
    require(maxLength >= 3) { "maxLength must be at least 3" }

    return if (length <= maxLength) {
        this
    } else {
        take(maxLength - 3) + "..."
    }
}

// Usage:

val email = "user@example.com".toEmail()

val normalized = "  HELLO  ".normalize()  // "hello"

val truncated = "Long message".truncate(8)  // "Long ..."
```

## Money Extensions for Arithmetic

```kotlin
package com.example.extensions

/**
 * Formats this money amount as a currency string.
 *
 * @param currencySymbol Currency symbol to prefix (default: "$")
 * @return Formatted currency string (e.g., "$12.99")
 */
fun Money.format(currencySymbol: String = "$"): String =
    "$currencySymbol%.2f".format(toDecimal())

/**
 * Applies a discount percentage to this amount.
 *
 * @param percentage Discount percentage (0.0 - 1.0)
 * @return Discounted amount
 */
fun Money.withDiscount(percentage: Double): Money {
    require(percentage in 0.0..1.0) { "Percentage must be 0.0-1.0" }
    return Money((cents * (1.0 - percentage)).toLong())
}

/**
 * Applies a tax percentage to this amount.
 *
 * @param taxRate Tax rate (e.g., 0.08 for 8%)
 * @return Amount with tax added
 */
fun Money.withTax(taxRate: Double): Money {
    require(taxRate >= 0.0) { "Tax rate cannot be negative" }
    return Money((cents * (1.0 + taxRate)).toLong())
}

/**
 * Splits this amount into equal parts.
 *
 * Handles rounding by distributing remainder across first N parts.
 *
 * @param parts Number of parts to split into (must be positive)
 * @return List of [parts] Money amounts that sum to original
 */
fun Money.splitEqually(parts: Int): List<Money> {
    require(parts > 0) { "Parts must be positive" }

    val baseAmount = cents / parts
    val remainder = cents % parts

    return List(parts) { index ->
        val amount = if (index < remainder) baseAmount + 1 else baseAmount
        Money(amount)
    }
}

// Usage:

val price = Money(1299)  // $12.99
val formatted = price.format()  // "$12.99"
val discounted = price.withDiscount(0.10)  // 10% off
val withTax = price.withTax(0.08)  // 8% tax

val split = Money(100).splitEqually(3)  // [34, 33, 33]
```

## Instant Extensions for Date Operations

```kotlin
package com.example.extensions

import kotlinx.datetime.*
import kotlin.time.Duration

/**
 * Checks if this instant is in the past.
 *
 * @return `true` if this instant is before now
 */
fun Instant.isPast(): Boolean =
    this < Clock.System.now()

/**
 * Checks if this instant is in the future.
 *
 * @return `true` if this instant is after now
 */
fun Instant.isFuture(): Boolean =
    this > Clock.System.now()

/**
 * Formats this instant as a human-readable string.
 *
 * @param timeZone Time zone for formatting (default: system)
 * @return Formatted date-time string (e.g., "2025-01-29 14:30:00")
 */
fun Instant.format(timeZone: TimeZone = TimeZone.currentSystemDefault()): String =
    toLocalDateTime(timeZone).toString()

/**
 * Calculates the duration from this instant until now.
 *
 * @return [Duration] from this instant to now (negative if future)
 */
fun Instant.durationUntilNow(): Duration =
    Clock.System.now() - this

/**
 * Checks if this instant is within the specified duration of now.
 *
 * @param duration The duration to check within
 * @return `true` if this instant is within [duration] of now
 */
fun Instant.isWithin(duration: Duration): Boolean {
    val diff = durationUntilNow()
    return diff.absoluteValue <= duration
}

// Usage:

val createdAt = user.createdAt

if (createdAt.isPast()) {
    println("User was created ${createdAt.durationUntilNow()} ago")
}

if (createdAt.isWithin(Duration.parse("7d"))) {
    println("Recent user!")
}
```

## Collection Extensions for Domain Logic

```kotlin
package com.example.extensions

/**
 * Finds the maximum value by the specified selector, or null if empty.
 *
 * More expressive than [maxByOrNull].
 *
 * @param selector Function to extract comparable value
 * @return Element with maximum value, or null if empty
 */
fun <T, R : Comparable<R>> Iterable<T>.findMax(selector: (T) -> R): T? =
    maxByOrNull(selector)

/**
 * Finds the minimum value by the specified selector, or null if empty.
 *
 * More expressive than [minByOrNull].
 *
 * @param selector Function to extract comparable value
 * @return Element with minimum value, or null if empty
 */
fun <T, R : Comparable<R>> Iterable<T>.findMin(selector: (T) -> R): T? =
    minByOrNull(selector)

/**
 * Partitions this list into two lists based on a predicate.
 *
 * More expressive than [partition] for domain-specific logic.
 *
 * @param predicate Function to determine which list each element goes to
 * @return Pair of (matching, non-matching) lists
 */
fun <T> List<T>.split(predicate: (T) -> Boolean): Pair<List<T>, List<T>> =
    partition(predicate)

/**
 * Converts this list to a non-empty list or returns null.
 *
 * @return [NonEmptyList] if this list is not empty, otherwise null
 */
fun <T> List<T>.toNonEmptyOrNull(): NonEmptyList<T>? =
    if (isNotEmpty()) NonEmptyList(first(), drop(1)) else null

// Usage:

val oldestUser = users.findMin { it.createdAt }
val newestUser = users.findMax { it.createdAt }

val (active, inactive) = users.split { it.isActive }
```

## Key Takeaways

1. **Extensions keep types focused**: Add behavior without modifying classes
2. **Readable chains**: Extensions enable fluent, left-to-right APIs
3. **Domain-specific operations**: Express business concepts as extensions
4. **No utility classes**: Extensions replace static utility methods
5. **Discoverable**: IDE completion shows extensions on the type
6. **Composable**: Chain multiple extensions naturally
7. **Pure functions**: Extensions should be pure where possible
8. **Well-documented**: Extensions need clear KDoc like any function
