# Functional Domain Models

Examples of functional, immutable domain models using data classes, sealed hierarchies, and value classes.

## Immutable Entity with Invariants

```kotlin
package com.example.domain

import kotlinx.datetime.Instant

/**
 * Represents an immutable user entity in the system.
 *
 * Users are uniquely identified by [id] and [email]. All state changes
 * produce new instances via [copy] rather than mutating existing objects.
 *
 * ## Invariants
 * - [email] is validated and unique across the system
 * - [name] must not be blank
 * - [createdAt] is never in the future
 * - [role] determines authorization level
 *
 * ## Usage Example
 * ```kotlin
 * val user = User(
 *     id = UserId(1),
 *     email = Email.of("user@example.com").getOrThrow(),
 *     name = "Alice",
 *     role = Role.USER,
 *     createdAt = Clock.System.now()
 * )
 *
 * val admin = user.withRole(Role.ADMIN)
 * ```
 *
 * @property id Unique identifier for this user
 * @property email Validated email address (unique across system)
 * @property name User's display name (must not be blank)
 * @property role User's authorization role
 * @property createdAt When this user account was created (UTC)
 * @see Role
 * @see Email
 */
data class User(
    val id: UserId,
    val email: Email,
    val name: String,
    val role: Role,
    val createdAt: Instant
) {
    init {
        require(name.isNotBlank()) { "User name cannot be blank" }
    }

    /**
     * Creates a copy of this user with an updated role.
     *
     * This is preferred over exposing [copy] directly, as it makes
     * role changes explicit and intentional, following the principle
     * of least surprise.
     *
     * ## Postconditions
     * - Returns a new [User] instance
     * - Original instance is unchanged
     * - Only [role] differs from original
     *
     * @param newRole The role to assign to the user
     * @return A new [User] instance with [role] set to [newRole]
     */
    fun withRole(newRole: Role): User = copy(role = newRole)

    /**
     * Checks if this user has administrative privileges.
     *
     * @return `true` if [role] is [Role.ADMIN], `false` otherwise
     */
    fun isAdmin(): Boolean = role == Role.ADMIN

    /**
     * Checks if this user account is active.
     *
     * Future extension point: could check suspension status,
     * email verification, etc.
     *
     * @return `true` if the user account is active
     */
    fun isActive(): Boolean = true  // Placeholder for future logic
}

/**
 * Authorization roles in the system.
 *
 * Roles follow a simple hierarchy: USER < MODERATOR < ADMIN.
 * For more complex authorization, consider using a permission-based
 * system instead.
 *
 * @see User.role
 */
enum class Role {
    /** Standard user with basic privileges */
    USER,

    /** Moderator with content management privileges */
    MODERATOR,

    /** Administrator with full system access */
    ADMIN
}
```

## Value Class with Smart Constructor

```kotlin
package com.example.domain

/**
 * A validated email address.
 *
 * This value class wraps a [String] with compile-time type safety
 * and zero runtime overhead. The only way to construct an instance
 * is via [Email.of], which validates the format.
 *
 * This makes invalid email addresses **unrepresentable** in the type system.
 *
 * ## Invariants
 * - Always contains a valid email format
 * - Never null or blank
 *
 * ## Usage Example
 * ```kotlin
 * val result = Email.of("user@example.com")
 * result.fold(
 *     onSuccess = { email -> println("Valid: ${email.value}") },
 *     onFailure = { error -> println("Invalid: $error") }
 * )
 * ```
 *
 * @property value The validated email string
 */
@JvmInline
value class Email private constructor(val value: String) {
    companion object {
        private val EMAIL_REGEX = Regex(
            pattern = """^[\w.+-]+@[\w.-]+\.[a-zA-Z]{2,}$"""
        )

        /**
         * Creates an [Email] from a string, validating the format.
         *
         * ## Preconditions
         * - [value] should contain an email-like string
         *
         * ## Postconditions
         * - On success: Returns [Email] with validated [value]
         * - On failure: Returns [InvalidEmailFormat] error
         *
         * @param value The email string to validate
         * @return [Result]<[Email]> containing validated email or error
         */
        fun of(value: String): Result<Email> =
            when {
                value.isBlank() ->
                    Result.failure(InvalidEmailFormat("Email cannot be blank"))

                !EMAIL_REGEX.matches(value) ->
                    Result.failure(InvalidEmailFormat("Invalid email format: $value"))

                else ->
                    Result.success(Email(value))
            }
    }

    override fun toString(): String = value
}

/**
 * Error indicating an invalid email format was provided.
 *
 * @property message Description of why the email is invalid
 */
data class InvalidEmailFormat(override val message: String) : Exception(message)
```

## Sealed Hierarchy for Domain States

```kotlin
package com.example.domain

import kotlinx.datetime.Instant

/**
 * Represents all possible states of an order in the system.
 *
 * This sealed hierarchy ensures exhaustive when-expression handling,
 * making state transitions explicit and impossible to handle incorrectly.
 *
 * ## State Transitions
 * ```
 * Pending → Processing → Shipped → Delivered
 *    ↓
 * Cancelled
 * ```
 *
 * @see Order
 */
sealed interface OrderStatus {
    /**
     * Order has been created but not yet processed.
     *
     * @property createdAt When the order was placed
     */
    data class Pending(val createdAt: Instant) : OrderStatus

    /**
     * Order is being prepared for shipment.
     *
     * @property startedAt When processing began
     * @property estimatedCompletion When processing is expected to finish
     */
    data class Processing(
        val startedAt: Instant,
        val estimatedCompletion: Instant
    ) : OrderStatus

    /**
     * Order has been shipped to the customer.
     *
     * @property shippedAt When the order was shipped
     * @property trackingNumber Shipment tracking identifier
     * @property carrier Name of shipping carrier
     */
    data class Shipped(
        val shippedAt: Instant,
        val trackingNumber: String,
        val carrier: String
    ) : OrderStatus

    /**
     * Order has been delivered to the customer.
     *
     * @property deliveredAt When delivery was confirmed
     * @property signedBy Name of person who received the order (if available)
     */
    data class Delivered(
        val deliveredAt: Instant,
        val signedBy: String?
    ) : OrderStatus

    /**
     * Order was cancelled before completion.
     *
     * @property cancelledAt When the order was cancelled
     * @property reason Human-readable cancellation reason
     * @property refundIssued Whether a refund was processed
     */
    data class Cancelled(
        val cancelledAt: Instant,
        val reason: String,
        val refundIssued: Boolean
    ) : OrderStatus
}

/**
 * Represents an order in the system.
 *
 * Orders are immutable—status changes produce new instances.
 *
 * @property id Unique order identifier
 * @property userId ID of user who placed the order
 * @property items List of ordered items
 * @property status Current order status
 */
data class Order(
    val id: OrderId,
    val userId: UserId,
    val items: List<OrderItem>,
    val status: OrderStatus
) {
    /**
     * Creates a copy with updated status.
     *
     * @param newStatus The new status to transition to
     * @return A new [Order] with updated [status]
     */
    fun withStatus(newStatus: OrderStatus): Order = copy(status = newStatus)

    /**
     * Checks if this order can be cancelled.
     *
     * Orders can only be cancelled if they haven't been delivered.
     *
     * @return `true` if the order can be cancelled
     */
    fun canBeCancelled(): Boolean = status !is OrderStatus.Delivered
}

/**
 * A single item in an order.
 *
 * @property productId ID of the ordered product
 * @property quantity Number of units ordered (must be positive)
 * @property pricePerUnit Price per unit at time of order
 */
data class OrderItem(
    val productId: ProductId,
    val quantity: Int,
    val pricePerUnit: Money
) {
    init {
        require(quantity > 0) { "Quantity must be positive" }
    }

    /**
     * Calculates the total price for this line item.
     *
     * @return Total price ([quantity] × [pricePerUnit])
     */
    fun totalPrice(): Money = Money(pricePerUnit.cents * quantity)
}
```

## Type Aliases for Domain Clarity

```kotlin
package com.example.domain

/**
 * Unique identifier for a user entity.
 *
 * Type alias provides semantic clarity while remaining compatible
 * with Long-based operations (database IDs, etc.).
 */
typealias UserId = Long

/**
 * Unique identifier for an order entity.
 */
typealias OrderId = Long

/**
 * Unique identifier for a product entity.
 */
typealias ProductId = Long

/**
 * Unique identifier for a transaction.
 */
typealias TransactionId = String

/**
 * Monetary amount in the smallest currency unit (e.g., cents for USD).
 *
 * Using [Long] prevents floating-point precision errors in financial
 * calculations. Always store money as integers in the smallest unit.
 *
 * ## Usage Example
 * ```kotlin
 * val price = Money(1299)  // $12.99
 * val tax = Money((price.cents * 0.08).toLong())  // 8% tax
 * val total = Money(price.cents + tax.cents)
 * ```
 */
@JvmInline
value class Money(val cents: Long) {
    init {
        require(cents >= 0) { "Money cannot be negative" }
    }

    operator fun plus(other: Money): Money = Money(cents + other.cents)
    operator fun minus(other: Money): Money = Money(cents - other.cents)
    operator fun times(multiplier: Int): Money = Money(cents * multiplier)

    /**
     * Converts to decimal representation (e.g., 1299 → 12.99).
     *
     * @return Decimal amount (dollars, euros, etc.)
     */
    fun toDecimal(): Double = cents / 100.0

    override fun toString(): String = "%.2f".format(toDecimal())
}
```

## Aggregate Root with Business Logic

```kotlin
package com.example.domain

/**
 * An aggregate root representing a shopping cart.
 *
 * The cart encapsulates business rules for adding/removing items
 * and calculating totals.
 *
 * ## Invariants
 * - [items] never contains duplicate products (quantities are merged)
 * - [items] never contains items with quantity ≤ 0
 * - Cart operations are pure—always return new instances
 *
 * @property userId ID of user who owns this cart
 * @property items Items currently in the cart
 */
data class Cart(
    val userId: UserId,
    val items: List<CartItem> = emptyList()
) {
    /**
     * Adds a product to the cart with specified quantity.
     *
     * If the product already exists, quantities are merged.
     *
     * ## Preconditions
     * - [quantity] must be positive
     *
     * ## Postconditions
     * - Returns new [Cart] with updated items
     * - If product existed, quantity is sum of old and new
     * - If product is new, it's added to items
     *
     * @param productId The product to add
     * @param quantity Number of units to add (must be > 0)
     * @param pricePerUnit Current price per unit
     * @return New [Cart] with added/updated item
     */
    fun addItem(
        productId: ProductId,
        quantity: Int,
        pricePerUnit: Money
    ): Cart {
        require(quantity > 0) { "Quantity must be positive" }

        val existingItem = items.find { it.productId == productId }

        val updatedItems = if (existingItem != null) {
            items.map { item ->
                if (item.productId == productId) {
                    item.copy(quantity = item.quantity + quantity)
                } else {
                    item
                }
            }
        } else {
            items + CartItem(productId, quantity, pricePerUnit)
        }

        return copy(items = updatedItems)
    }

    /**
     * Removes a product from the cart entirely.
     *
     * @param productId The product to remove
     * @return New [Cart] without the specified product
     */
    fun removeItem(productId: ProductId): Cart =
        copy(items = items.filterNot { it.productId == productId })

    /**
     * Updates the quantity of a product in the cart.
     *
     * If [newQuantity] is 0, the item is removed.
     *
     * @param productId The product to update
     * @param newQuantity The new quantity (0 removes the item)
     * @return New [Cart] with updated quantity
     */
    fun updateQuantity(productId: ProductId, newQuantity: Int): Cart {
        require(newQuantity >= 0) { "Quantity cannot be negative" }

        return if (newQuantity == 0) {
            removeItem(productId)
        } else {
            copy(
                items = items.map { item ->
                    if (item.productId == productId) {
                        item.copy(quantity = newQuantity)
                    } else {
                        item
                    }
                }
            )
        }
    }

    /**
     * Calculates the total price of all items in the cart.
     *
     * @return Sum of all item totals
     */
    fun total(): Money =
        items.fold(Money(0)) { acc, item ->
            acc + item.totalPrice()
        }

    /**
     * Checks if the cart is empty.
     *
     * @return `true` if there are no items in the cart
     */
    fun isEmpty(): Boolean = items.isEmpty()

    /**
     * Gets the total number of items in the cart.
     *
     * @return Sum of quantities across all items
     */
    fun itemCount(): Int = items.sumOf { it.quantity }
}

/**
 * A single item in a shopping cart.
 *
 * @property productId ID of the product
 * @property quantity Number of units in cart (must be positive)
 * @property pricePerUnit Price per unit at time of addition
 */
data class CartItem(
    val productId: ProductId,
    val quantity: Int,
    val pricePerUnit: Money
) {
    init {
        require(quantity > 0) { "Quantity must be positive" }
    }

    fun totalPrice(): Money = pricePerUnit * quantity
}
```

## Key Takeaways

1. **Immutability**: All domain models use `val` properties and return new instances for changes
2. **Invariants**: `init` blocks and `require` enforce business rules
3. **Smart Constructors**: Value classes with private constructors ensure validity
4. **Sealed Hierarchies**: Make state exhaustive and type-safe
5. **Rich Behavior**: Domain logic lives in the domain models, not anemic DTOs
6. **Comprehensive KDoc**: Every type and function documents contracts and invariants
