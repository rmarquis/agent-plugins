# KDoc Style Guide

Comprehensive guide for writing high-quality KDoc documentation in functional Kotlin code.

## Core Principle

**KDoc documents contracts, not just implementation.**

Every public declaration should answer:
1. **What** does this do? (summary)
2. **How** does it work? (detailed description)
3. **What must be true beforehand?** (preconditions)
4. **What is guaranteed afterward?** (postconditions)
5. **What always holds?** (invariants)
6. **What else happens?** (side effects)
7. **How do I use it?** (examples)

## Class Documentation

### Template

```kotlin
/**
 * [One-sentence summary of what this class represents]
 *
 * [Detailed description: purpose, design decisions, usage patterns, when to use]
 *
 * ## Invariants
 * - [Property 1] always satisfies [constraint]
 * - [Property 2] is maintained across all operations
 * - [Relationship] between properties X and Y
 *
 * ## Usage Example
 * ```kotlin
 * val instance = ClassName(param1, param2)
 * val result = instance.method()
 * ```
 *
 * ## Thread Safety
 * [If relevant: thread-safety guarantees or warnings]
 *
 * @property prop1 [Description including constraints, valid ranges, or meaning]
 * @property prop2 [Description including constraints, valid ranges, or meaning]
 * @constructor [If constructor has complex logic, document it]
 * @see RelatedClass
 * @see relatedFunction
 * @since 1.0.0
 */
data class ClassName(
    val prop1: Type1,
    val prop2: Type2
)
```

### Example: Domain Entity

```kotlin
/**
 * Represents an immutable user entity in the authentication system.
 *
 * Users are uniquely identified by [id] and [email]. All state changes
 * produce new instances via [copy] rather than mutating existing objects,
 * ensuring thread safety and preventing temporal coupling.
 *
 * ## Invariants
 * - [email] is validated and unique across the system
 * - [name] must not be blank (enforced in init block)
 * - [createdAt] is never in the future
 * - [role] determines the user's authorization level
 *
 * ## Usage Example
 * ```kotlin
 * val user = User(
 *     id = UserId(1),
 *     email = Email.of("alice@example.com").getOrThrow(),
 *     name = "Alice",
 *     role = Role.USER,
 *     createdAt = Clock.System.now()
 * )
 *
 * val admin = user.withRole(Role.ADMIN)  // Creates new instance
 * ```
 *
 * ## Thread Safety
 * Immutable and thread-safe. Safe to share across threads.
 *
 * @property id Unique identifier for this user (immutable after creation)
 * @property email Validated email address (guaranteed unique across system)
 * @property name User's display name (must not be blank)
 * @property role User's authorization role (determines permissions)
 * @property createdAt When this user account was created (UTC timestamp)
 * @see Role
 * @see Email
 * @since 1.0.0
 */
data class User(
    val id: UserId,
    val email: Email,
    val name: String,
    val role: Role,
    val createdAt: Instant
)
```

## Function Documentation

### Template

```kotlin
/**
 * [One-sentence summary of what this function does]
 *
 * [Detailed description: behavior, algorithm, edge cases, design rationale,
 * when to use this vs alternatives]
 *
 * ## Preconditions
 * - [param1] must satisfy [constraint]
 * - [system state] must be [condition]
 * - [resource] must be available
 *
 * ## Postconditions
 * - On success: [guaranteed outcome 1]
 * - On success: [guaranteed outcome 2]
 * - On failure: [guaranteed outcome 3]
 * - [Invariant] is maintained
 *
 * ## Side Effects
 * - [List any observable effects beyond return value]
 * - [E.g., writes to database, sends network requests, logs, emits events]
 *
 * ## Performance
 * [If relevant: time/space complexity, performance characteristics]
 *
 * ## Example
 * ```kotlin
 * val result = functionName(arg1, arg2)
 * result.fold(
 *     onSuccess = { value -> /* use value */ },
 *     onFailure = { error -> /* handle error */ }
 * )
 * ```
 *
 * @param param1 [Description including constraints, valid values, special cases]
 * @param param2 [Description including constraints, valid values, special cases]
 * @return [Description including success and failure semantics]
 * @throws ExceptionType [Only for truly exceptional cases, explain when]
 * @see relatedFunction
 * @see RelatedType
 * @since 1.0.0
 */
fun functionName(param1: Type1, param2: Type2): Result<ReturnType>
```

### Example: Pure Function

```kotlin
/**
 * Validates a user's registration credentials.
 *
 * This function composes multiple validation steps (email, password, name)
 * using Result's flatMap combinator, short-circuiting on the first failure.
 * It is a pure function—deterministic and side-effect-free.
 *
 * ## Preconditions
 * - [credentials] should contain registration data
 * - No external state required
 *
 * ## Postconditions
 * - On success: Returns [ValidatedCredentials] with all fields validated
 * - On failure: Returns first [ValidationError] encountered
 * - No state changes occur (pure function)
 *
 * ## Side Effects
 * None (pure function)
 *
 * ## Performance
 * O(1) time complexity. Validates sequentially, stopping at first error.
 *
 * ## Example
 * ```kotlin
 * val credentials = RegistrationCredentials(
 *     email = "alice@example.com",
 *     password = "SecurePass123",
 *     name = "Alice"
 * )
 *
 * validateCredentials(credentials).fold(
 *     onSuccess = { validated -> registerUser(validated) },
 *     onFailure = { error -> showValidationError(error) }
 * )
 * ```
 *
 * @param credentials The registration credentials to validate
 * @return [Result]<[ValidatedCredentials]> containing validated data or first error
 * @see ValidatedCredentials
 * @see ValidationError
 * @see registerUser
 * @since 1.0.0
 */
fun validateCredentials(
    credentials: RegistrationCredentials
): Result<ValidatedCredentials>
```

### Example: Side-Effecting Function

```kotlin
/**
 * Processes a payment transaction.
 *
 * This function coordinates payment validation, authorization with the payment
 * gateway, and transaction persistence. It implements idempotency using
 * [transactionId]—repeated calls with the same ID return the existing transaction.
 *
 * ## Preconditions
 * - [request] must have a positive amount
 * - [request.paymentMethod] must be valid
 * - [transactionId] must be unique for new transactions
 * - Payment gateway must be reachable
 *
 * ## Postconditions
 * - On success: Transaction is persisted with status SUCCESS
 * - On success: Payment gateway has authorized the charge
 * - On success: `payment.processed` event is emitted
 * - On failure: No charge is made (payment gateway rollback)
 * - On failure: Transaction may be persisted with status FAILED
 *
 * ## Side Effects
 * - Calls payment gateway API (network I/O)
 * - Writes transaction to database
 * - Emits `payment.processed` or `payment.failed` event
 * - Increments `payments.processed` or `payments.failed` metric
 * - Logs payment attempt and outcome
 * - Sends receipt email asynchronously
 *
 * ## Performance
 * Typical response time: 200-500ms (network-bound)
 * Retries failed gateway calls up to 3 times with exponential backoff
 *
 * ## Example
 * ```kotlin
 * val request = PaymentRequest(
 *     amount = Money(1999),  // $19.99
 *     paymentMethod = userCard,
 *     description = "Premium subscription"
 * )
 *
 * processPayment(request, TransactionId.generate()).fold(
 *     onSuccess = { transaction -
 *         logger.info("Payment successful: ${transaction.id}")
 *         showSuccessMessage()
 *     },
 *     onFailure = { error -
 *         when (error) {
 *             is PaymentError.InsufficientFunds -> showInsufficientFundsMessage()
 *             is PaymentError.CardExpired -> promptForNewCard()
 *             else -> showGenericError()
 *         }
 *     }
 * )
 * ```
 *
 * @param request The payment request containing amount and payment method
 * @param transactionId Unique identifier for this transaction (for idempotency)
 * @return [Result]<[Transaction]> containing completed transaction or error
 * @see PaymentRequest
 * @see PaymentError
 * @see Transaction
 * @since 1.0.0
 */
suspend fun processPayment(
    request: PaymentRequest,
    transactionId: TransactionId
): Result<Transaction>
```

## Property Documentation

### Template

```kotlin
/**
 * [Description of property's semantic meaning and purpose]
 *
 * [Additional context: constraints, valid ranges, relationships to other properties,
 * when this is populated, what null means if nullable]
 *
 * @see relatedProperty
 * @since 1.0.0
 */
val propertyName: Type
```

### Examples

```kotlin
/**
 * Unique identifier for this user.
 *
 * Generated at user creation time and never changes afterward.
 * Used as primary key in database and for all user lookups.
 */
val id: UserId

/**
 * Validated email address, unique across the system.
 *
 * Guaranteed to be in valid email format due to smart constructor
 * validation in [Email.of]. Used for authentication and communication.
 *
 * @see Email
 */
val email: Email

/**
 * Current order status.
 *
 * Status transitions follow the lifecycle:
 * Pending → Processing → Shipped → Delivered
 * Any status can transition to Cancelled except Delivered.
 *
 * @see OrderStatus
 * @see withStatus
 */
val status: OrderStatus
```

## Sealed Interface Documentation

### Template

```kotlin
/**
 * [Summary of what this sealed hierarchy represents]
 *
 * [Explanation of why a sealed hierarchy is used, what cases are covered,
 * when to use each case]
 *
 * ## State Transitions
 * [If applicable: diagram or description of valid transitions]
 *
 * @see relatedFunction
 * @since 1.0.0
 */
sealed interface TypeName {
    /**
     * [Description of this specific case]
     *
     * [When this case occurs, what it represents, how to handle it]
     *
     * @property prop [If data class: description of property]
     */
    data class Case1(val prop: Type) : TypeName

    /**
     * [Description of this specific case]
     */
    data object Case2 : TypeName
}
```

### Example

```kotlin
/**
 * Represents all possible errors during user registration.
 *
 * This sealed hierarchy ensures exhaustive when-expression handling,
 * making error handling explicit and impossible to handle incorrectly.
 * Each error case carries the specific data needed to understand and
 * communicate the failure to the user.
 *
 * ## Error Handling Example
 * ```kotlin
 * registerUser(credentials).fold(
 *     onSuccess = { user -> showWelcome(user) },
 *     onFailure = { error ->
 *         when (error) {
 *             is RegistrationError.EmailExists -
 *                 showError("Email ${error.email} already registered")
 *             is RegistrationError.Validation -
 *                 showValidationErrors(error.errors)
 *             is RegistrationError.RegistrationDisabled -
 *                 showError("Registration disabled: ${error.reason}")
 *         }
 *     }
 * )
 * ```
 *
 * @see registerUser
 * @see ValidationError
 * @since 1.0.0
 */
sealed interface RegistrationError {
    /**
     * The provided email address is already registered in the system.
     *
     * This occurs during registration when the email uniqueness check fails.
     * The user should be prompted to use a different email or log in instead.
     *
     * @property email The duplicate email address that was attempted
     */
    data class EmailExists(val email: Email) : RegistrationError

    /**
     * One or more validation errors occurred.
     *
     * This wraps low-level validation errors (email format, password strength, etc.)
     * into the registration error hierarchy. All validation errors should be shown
     * to the user so they can fix all issues at once.
     *
     * @property errors Non-empty list of validation errors
     * @see ValidationError
     */
    data class Validation(val errors: List<ValidationError>) : RegistrationError

    /**
     * User registration is temporarily disabled.
     *
     * This might occur during maintenance, security incidents, or when
     * the system is at capacity. The [reason] should be shown to the user.
     *
     * @property reason Human-readable explanation of why registration is disabled
     */
    data class RegistrationDisabled(val reason: String) : RegistrationError
}
```

## Value Class Documentation

### Example

```kotlin
/**
 * A validated email address with zero runtime overhead.
 *
 * This value class wraps a [String] with compile-time type safety,
 * preventing accidental mixing of email addresses with other strings.
 * The private constructor ensures that only validated email addresses
 * can exist—invalid email addresses are **unrepresentable** in the type system.
 *
 * ## Validation Rules
 * - Must match basic email format: local@domain.tld
 * - Domain must have at least one dot
 * - Top-level domain must be at least 2 characters
 * - Cannot be blank or contain whitespace
 *
 * ## Usage Example
 * ```kotlin
 * Email.of("alice@example.com").fold(
 *     onSuccess = { email -> sendMessage(email) },
 *     onFailure = { error -> showError("Invalid email format") }
 * )
 * ```
 *
 * ## Performance
 * Zero runtime overhead due to `@JvmInline`. The wrapper is erased
 * at compile time while preserving type safety.
 *
 * @property value The validated email string (guaranteed valid)
 * @see of
 * @since 1.0.0
 */
@JvmInline
value class Email private constructor(val value: String) {
    companion object {
        /**
         * Creates an [Email] from a string, validating the format.
         *
         * This is the only way to construct an [Email] instance, ensuring
         * that all Email instances are valid.
         *
         * ## Preconditions
         * - [value] should be an email-like string
         *
         * ## Postconditions
         * - On success: Returns [Email] with [value] stored
         * - On failure: Returns [InvalidEmailFormat] with explanation
         *
         * @param value The email string to validate
         * @return [Result]<[Email]> containing validated email or error
         */
        fun of(value: String): Result<Email> = /* ... */
    }
}
```

## Documentation Anti-Patterns

### Useless Documentation

```kotlin
/**
 * Gets the user.
 */
fun getUser(id: Long): User?

/**
 * User name.
 */
val name: String

/**
 * User class.
 */
data class User(...)
```

**Problems:**
- Provides no information beyond the name
- Doesn't explain constraints, preconditions, or postconditions
- No information about failure modes or side effects

### Implementation Details Instead of Contracts

```kotlin
/**
 * Loops through all users and finds the one with matching ID
 * by comparing each ID until a match is found or the list ends.
 */
fun findUser(id: Long): User?
```

**Problems:**
- Documents "how" instead of "what" and "why"
- Ties documentation to specific implementation
- Doesn't document the contract (preconditions, postconditions)

### Missing Critical Information

```kotlin
/**
 * Processes a payment.
 */
fun processPayment(amount: Money): Result<Transaction>
```

**Problems:**
- No information about side effects (charges card, writes to DB)
- No information about idempotency
- No information about failure modes
- No information about preconditions (valid amount, etc.)

## Special Sections

### Preconditions

Document what must be true before calling:

```kotlin
/**
 * ...
 *
 * ## Preconditions
 * - [amount] must be positive
 * - [userId] must exist in the system
 * - User must have a valid payment method configured
 * - System must not be in maintenance mode
 */
```

### Postconditions

Document what is guaranteed after calling:

```kotlin
/**
 * ...
 *
 * ## Postconditions
 * - On success: Transaction is persisted with status SUCCESS
 * - On success: User's account balance is updated
 * - On success: Receipt email is queued for sending
 * - On failure: No charges are made
 * - On failure: Original state is preserved (transactional)
 */
```

### Invariants

Document properties that always hold:

```kotlin
/**
 * ...
 *
 * ## Invariants
 * - [total] always equals sum of [items] prices
 * - [items] never contains negative quantities
 * - [createdAt] is never after [updatedAt]
 * - [userId] never changes after creation
 */
```

### Side Effects

Document observable effects beyond return value:

```kotlin
/**
 * ...
 *
 * ## Side Effects
 * - Writes to database (transactions table)
 * - Calls payment gateway API
 * - Emits `payment.processed` event
 * - Increments `payments.success` metric
 * - Logs payment details (excluding sensitive data)
 * - Sends receipt email asynchronously
 */
```

## Best Practices

1. **Document Contracts, Not Implementation**
   - Focus on what the function guarantees, not how it works
   - Implementation can change; contracts should be stable

2. **Be Specific About Failure Modes**
   - List all possible error cases
   - Explain when each error occurs
   - Guide callers on how to handle each error

3. **Include Examples for Complex APIs**
   - Show typical usage patterns
   - Demonstrate error handling
   - Show how to compose with other functions

4. **Explain Design Decisions**
   - Why this approach vs alternatives?
   - What trade-offs were made?
   - When should callers use this vs other options?

5. **Keep Documentation Up-to-Date**
   - Update KDoc when signatures change
   - Update examples when APIs evolve
   - Mark deprecated features with `@Deprecated`

6. **Link Related Declarations**
   - Use `@see` to reference related functions
   - Link to types mentioned in description
   - Create navigation paths through the codebase

7. **Use Markdown Effectively**
   - Use lists for multiple items
   - Use code blocks for examples
   - Use bold for emphasis (sparingly)
   - Use sections (##) for organization

## Summary

Great KDoc should enable someone to:
- **Understand** what the function does without reading implementation
- **Use** the function correctly without trial and error
- **Handle errors** appropriately by knowing all failure modes
- **Trust** the function's guarantees (postconditions)
- **Learn** from examples and see the function in context
