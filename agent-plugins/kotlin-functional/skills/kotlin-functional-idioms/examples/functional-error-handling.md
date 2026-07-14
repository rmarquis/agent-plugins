# Functional Error Handling

Examples of type-safe error handling using Result types and sealed error hierarchies.

## Result Type for Expected Failures

```kotlin
package com.example.service

/**
 * Validates an email address format.
 *
 * This is a pure function—deterministic and side-effect-free.
 *
 * ## Postconditions
 * - On success: Returns [Email] with validated address
 * - On failure: Returns [ValidationError.InvalidEmail]
 *
 * @param email The email string to validate
 * @return [Result]<[Email]> containing validated email or error
 */
fun validateEmail(email: String): Result<Email> =
    Email.of(email)
        .mapError { ValidationError.InvalidEmail(email, it.message ?: "Invalid format") }

/**
 * Validates a password against security requirements.
 *
 * ## Password Requirements
 * - Minimum 8 characters
 * - At least one uppercase letter
 * - At least one lowercase letter
 * - At least one digit
 *
 * ## Postconditions
 * - On success: Returns the validated password string
 * - On failure: Returns [ValidationError.WeakPassword] with violations
 *
 * @param password The password to validate
 * @return [Result]<[String]> containing password or error with violations
 */
fun validatePassword(password: String): Result<String> {
    val violations = buildList {
        if (password.length < 8) {
            add("Must be at least 8 characters")
        }
        if (!password.any { it.isUpperCase() }) {
            add("Must contain at least one uppercase letter")
        }
        if (!password.any { it.isLowerCase() }) {
            add("Must contain at least one lowercase letter")
        }
        if (!password.any { it.isDigit() }) {
            add("Must contain at least one digit")
        }
    }

    return if (violations.isEmpty()) {
        Result.success(password)
    } else {
        Result.failure(ValidationError.WeakPassword(violations))
    }
}

/**
 * Validates a user's name.
 *
 * ## Preconditions
 * - [name] should be a non-empty string
 *
 * ## Postconditions
 * - On success: Returns trimmed, validated name
 * - On failure: Returns [ValidationError.InvalidName]
 *
 * @param name The name to validate
 * @return [Result]<[String]> containing validated name or error
 */
fun validateName(name: String): Result<String> {
    val trimmed = name.trim()

    return when {
        trimmed.isBlank() ->
            Result.failure(ValidationError.InvalidName("Name cannot be blank"))

        trimmed.length < 2 ->
            Result.failure(ValidationError.InvalidName("Name must be at least 2 characters"))

        trimmed.length > 100 ->
            Result.failure(ValidationError.InvalidName("Name cannot exceed 100 characters"))

        else ->
            Result.success(trimmed)
    }
}
```

## Sealed Error Hierarchies

```kotlin
package com.example.domain

/**
 * Represents all possible validation errors.
 *
 * This sealed hierarchy ensures exhaustive when-expression handling,
 * making error handling explicit and type-safe.
 *
 * @see validateUser
 */
sealed interface ValidationError {
    /**
     * The provided email address has an invalid format.
     *
     * @property email The invalid email string
     * @property reason Description of why the format is invalid
     */
    data class InvalidEmail(
        val email: String,
        val reason: String
    ) : ValidationError

    /**
     * The provided password doesn't meet security requirements.
     *
     * @property violations List of specific requirement violations
     */
    data class WeakPassword(
        val violations: List<String>
    ) : ValidationError

    /**
     * The provided name is invalid.
     *
     * @property reason Description of why the name is invalid
     */
    data class InvalidName(
        val reason: String
    ) : ValidationError
}

/**
 * Represents all possible user registration errors.
 *
 * Includes both validation errors and domain errors.
 */
sealed interface RegistrationError {
    /**
     * A validation error occurred during registration.
     *
     * @property error The underlying validation error
     */
    data class Validation(val error: ValidationError) : RegistrationError

    /**
     * The provided email address is already registered.
     *
     * @property email The duplicate email address
     */
    data class EmailAlreadyExists(val email: Email) : RegistrationError

    /**
     * User registration is temporarily disabled.
     *
     * @property reason Explanation of why registration is disabled
     */
    data class RegistrationDisabled(val reason: String) : RegistrationError
}

/**
 * Represents all possible errors during payment processing.
 */
sealed interface PaymentError {
    /**
     * Payment card has expired.
     *
     * @property expirationDate The card's expiration date
     */
    data class CardExpired(val expirationDate: YearMonth) : PaymentError

    /**
     * Insufficient funds in the account.
     *
     * @property available Amount available in account
     * @property required Amount required for transaction
     */
    data class InsufficientFunds(
        val available: Money,
        val required: Money
    ) : PaymentError

    /**
     * Payment processor service is unavailable.
     *
     * @property retryAfter Suggested duration before retrying
     */
    data class ServiceUnavailable(val retryAfter: Duration) : PaymentError

    /**
     * Card was declined by the issuing bank.
     *
     * @property reason Human-readable decline reason (if provided by bank)
     */
    data class Declined(val reason: String?) : PaymentError

    /**
     * The payment amount is invalid.
     *
     * @property amount The invalid amount
     * @property reason Why the amount is invalid
     */
    data class InvalidAmount(
        val amount: Money,
        val reason: String
    ) : PaymentError
}
```

## Composing Result Types

```kotlin
package com.example.service

import kotlinx.datetime.Clock

/**
 * Registers a new user with validated credentials.
 *
 * This function composes multiple validation steps using Result's
 * [flatMap] combinator, short-circuiting on the first failure.
 *
 * ## Preconditions
 * - [credentials] should contain registration data
 *
 * ## Postconditions
 * - On success: User is created with generated ID and validated data
 * - On failure: No user is created, returns [RegistrationError]
 *
 * ## Side Effects
 * - None (pure function—actual persistence happens elsewhere)
 *
 * @param credentials The registration credentials to validate
 * @return [Result]<[User]> containing created user or registration error
 * @see RegistrationError
 */
fun registerUser(credentials: RegistrationCredentials): Result<User> =
    validateEmail(credentials.email)
        .mapError { RegistrationError.Validation(it) }
        .flatMap { validEmail ->
            validatePassword(credentials.password)
                .mapError { RegistrationError.Validation(it) }
                .flatMap { validPassword ->
                    validateName(credentials.name)
                        .mapError { RegistrationError.Validation(it) }
                        .map { validName ->
                            User(
                                id = UserId.generate(),
                                email = validEmail,
                                name = validName,
                                role = Role.USER,
                                createdAt = Clock.System.now()
                            )
                        }
                }
        }

/**
 * Registration credentials provided by a user.
 *
 * @property email User's email address (unvalidated)
 * @property password User's password (unvalidated)
 * @property name User's display name (unvalidated)
 */
data class RegistrationCredentials(
    val email: String,
    val password: String,
    val name: String
)

/**
 * Extension function to map errors from one type to another.
 *
 * This is useful for converting low-level errors (e.g., [ValidationError])
 * into domain-specific errors (e.g., [RegistrationError]).
 *
 * @param transform Function to transform the error
 * @return New [Result] with transformed error
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
```

## Handling Multiple Errors (Accumulating)

```kotlin
package com.example.service

/**
 * Validation result that can accumulate multiple errors.
 *
 * Unlike [Result], which short-circuits on first error, this type
 * collects all validation errors before failing.
 *
 * @param T The type of successful value
 */
sealed interface Validated<out T> {
    /**
     * Validation succeeded.
     *
     * @property value The validated value
     */
    data class Valid<T>(val value: T) : Validated<T>

    /**
     * Validation failed with one or more errors.
     *
     * @property errors Non-empty list of validation errors
     */
    data class Invalid(val errors: List<ValidationError>) : Validated<Nothing> {
        init {
            require(errors.isNotEmpty()) { "Invalid must have at least one error" }
        }
    }
}

/**
 * Validates all fields of user registration credentials, accumulating errors.
 *
 * Unlike [registerUser], which short-circuits on first error, this function
 * validates all fields and returns all errors together.
 *
 * This is better UX—users see all validation errors at once rather than
 * fixing them one at a time.
 *
 * @param credentials The credentials to validate
 * @return [Validated]<[ValidatedCredentials]> with all errors or valid data
 */
fun validateAllFields(credentials: RegistrationCredentials): Validated<ValidatedCredentials> {
    val errors = mutableListOf<ValidationError>()

    val validEmail = validateEmail(credentials.email)
        .onFailure { errors.add(it as ValidationError) }
        .getOrNull()

    val validPassword = validatePassword(credentials.password)
        .onFailure { errors.add(it as ValidationError) }
        .getOrNull()

    val validName = validateName(credentials.name)
        .onFailure { errors.add(it as ValidationError) }
        .getOrNull()

    return if (errors.isEmpty() && validEmail != null && validPassword != null && validName != null) {
        Validated.Valid(
            ValidatedCredentials(
                email = validEmail,
                password = validPassword,
                name = validName
            )
        )
    } else {
        Validated.Invalid(errors)
    }
}

/**
 * Validated registration credentials.
 *
 * The existence of this type guarantees that all validation has passed.
 *
 * @property email Validated email address
 * @property password Validated password
 * @property name Validated display name
 */
data class ValidatedCredentials(
    val email: Email,
    val password: String,
    val name: String
)
```

## Handling Errors with when Expression

```kotlin
package com.example.ui

/**
 * Formats a registration error for display to the user.
 *
 * This function demonstrates exhaustive error handling using
 * when-expressions on sealed hierarchies.
 *
 * @param error The registration error to format
 * @return Human-readable error message
 */
fun formatRegistrationError(error: RegistrationError): String = when (error) {
    is RegistrationError.EmailAlreadyExists ->
        "The email ${error.email} is already registered. Please use a different email or log in."

    is RegistrationError.RegistrationDisabled ->
        "Registration is temporarily disabled: ${error.reason}"

    is RegistrationError.Validation -> when (val validationError = error.error) {
        is ValidationError.InvalidEmail ->
            "Invalid email address: ${validationError.reason}"

        is ValidationError.WeakPassword ->
            "Password requirements not met:\n${validationError.violations.joinToString("\n") { "• $it" }}"

        is ValidationError.InvalidName ->
            "Invalid name: ${validationError.reason}"
    }
}

/**
 * Formats a payment error for display to the user.
 *
 * @param error The payment error to format
 * @return Human-readable error message
 */
fun formatPaymentError(error: PaymentError): String = when (error) {
    is PaymentError.CardExpired ->
        "Your card expired on ${error.expirationDate}. Please update your payment method."

    is PaymentError.InsufficientFunds ->
        "Insufficient funds. Available: ${error.available}, Required: ${error.required}"

    is PaymentError.ServiceUnavailable ->
        "Payment service is temporarily unavailable. Please try again in ${error.retryAfter.inWholeMinutes} minutes."

    is PaymentError.Declined -> {
        val reason = error.reason ?: "unknown reason"
        "Your card was declined: $reason"
    }

    is PaymentError.InvalidAmount ->
        "Invalid payment amount (${error.amount}): ${error.reason}"
}
```

## Recovering from Errors

```kotlin
package com.example.service

/**
 * Attempts to find a user by ID, with fallback strategies.
 *
 * Demonstrates error recovery patterns using Result combinators.
 *
 * @param id The user ID to find
 * @param repository The user repository
 * @return [Result]<[User]> from primary or fallback source
 */
fun findUserWithFallback(
    id: UserId,
    repository: UserRepository
): Result<User> =
    repository.findById(id)
        .recoverCatching { error ->
            // If primary lookup fails, try alternative sources
            when (error) {
                is UserNotFound -> findInCache(id).getOrThrow()
                else -> throw error
            }
        }

/**
 * Processes a payment with automatic retry on transient errors.
 *
 * Demonstrates selective error recovery based on error type.
 *
 * @param request The payment request
 * @return [Result]<[Transaction]> after retry logic
 */
fun processPaymentWithRetry(request: PaymentRequest): Result<Transaction> =
    processPayment(request)
        .recoverCatching { error ->
            when (error) {
                is PaymentError.ServiceUnavailable -> {
                    // Retry on service unavailability
                    Thread.sleep(error.retryAfter.inWholeMilliseconds)
                    processPayment(request).getOrThrow()
                }
                else -> throw error  // Don't retry other errors
            }
        }

/**
 * Gets a user by ID, returning a default user if not found.
 *
 * Demonstrates using a default value to recover from errors.
 *
 * @param id The user ID to find
 * @return User if found, otherwise [createGuestUser]
 */
fun getUserOrGuest(id: UserId): User =
    findUser(id).getOrElse { createGuestUser() }
```

## Key Takeaways

1. **Result<T>** for operations that can fail (explicit in signature)
2. **Sealed hierarchies** for domain-specific error types (exhaustive handling)
3. **flatMap** for composing Result types (short-circuit on failure)
4. **mapError** for converting error types (low-level → domain)
5. **Validated** type for accumulating multiple errors (better UX)
6. **when expressions** for exhaustive error handling (compiler-verified)
7. **recoverCatching** for selective error recovery (fallbacks, retries)
8. **No exceptions** for expected failures (only for truly exceptional cases)
