# Functional Patterns Catalog

Comprehensive catalog of functional programming patterns in Kotlin.

## Pure Functions

### Definition
A function is **pure** if:
1. **Deterministic**: Same inputs always produce same output
2. **No side effects**: Doesn't modify external state or perform I/O

### Example
```kotlin
// Pure function
fun calculateDiscount(price: Money, percentage: Double): Money =
    Money((price.cents * (1.0 - percentage)).toLong())

// Impure (reads external state)
var taxRate = 0.08
fun applyTax(price: Money): Money =
    Money((price.cents * (1.0 + taxRate)).toLong())

// Impure (modifies external state)
fun incrementCounter() {
    counter++
}

// Impure (performs I/O)
fun getUserName(id: Long): String =
    database.query("SELECT name FROM users WHERE id = $id")
```

### Benefits
- **Testable**: No mocks needed, just input/output
- **Cacheable**: Can memoize results
- **Parallelizable**: Safe to execute concurrently
- **Reasonability**: Easier to understand and verify

### Pattern: Separate Pure Logic from Side Effects
```kotlin
// Pure business logic
fun calculateOrderTotal(items: List<OrderItem>): Money =
    items.fold(Money(0)) { acc, item -> acc + item.totalPrice() }

// Side-effecting wrapper
suspend fun saveOrder(order: Order): Result<Unit> {
    val total = calculateOrderTotal(order.items)  // Pure
    return database.save(order.copy(total = total))  // Side effect
}
```

## Immutability

### Definition
**Immutable data** cannot be changed after creation. All transformations create new instances.

### Pattern: Immutable Data Classes
```kotlin
data class User(
    val id: UserId,
    val email: Email,
    val name: String,
    val role: Role
) {
    // Transformation returns new instance
    fun withRole(newRole: Role): User = copy(role = newRole)
}
```

### Pattern: Persistent Data Structures
```kotlin
// Kotlin's collections are immutable by default
val users = listOf(user1, user2, user3)  // Immutable List
val updated = users + newUser            // New list, original unchanged
val filtered = users.filter { it.isActive }  // New list
```

### Benefits
- **Thread-safe**: No synchronization needed
- **No temporal coupling**: Order of operations doesn't matter
- **Easier debugging**: State doesn't change unexpectedly
- **Undo/Redo**: Keep previous versions without cloning

## Higher-Order Functions

### Definition
Functions that:
1. **Accept functions** as parameters, or
2. **Return functions** as results

### Pattern: Function as Parameter
```kotlin
inline fun <T> measureTime(
    operationName: String,
    operation: () -> T
): T {
    val start = System.currentTimeMillis()
    return operation().also {
        val duration = System.currentTimeMillis() - start
        logger.info("$operationName took ${duration}ms")
    }
}

// Usage:
val result = measureTime("User query") {
    repository.findAllActiveUsers()
}
```

### Pattern: Function as Return Value
```kotlin
fun createValidator(minLength: Int): (String) -> Boolean =
    { input -> input.length >= minLength }

val validatePassword = createValidator(8)
val isValid = validatePassword("mypassword")  // true
```

### Pattern: Function Composition
```kotlin
infix fun <A, B, C> ((B) -> C).compose(g: (A) -> B): (A) -> C =
    { a -> this(g(a)) }

val toLowerCase: (String) -> String = { it.lowercase() }
val trim: (String) -> String = { it.trim() }
val normalize = toLowerCase compose trim

val result = normalize("  HELLO  ")  // "hello"
```

## Type-Safe Error Handling

### Pattern: Result Type
```kotlin
fun findUser(id: UserId): Result<User> =
    repository.find(id)
        ?.let { Result.success(it) }
        ?: Result.failure(UserNotFound(id))

// Composing Results
fun processUser(id: UserId): Result<ProcessedUser> =
    findUser(id)
        .map { it.email }
        .flatMap { sendVerificationEmail(it) }
        .map { ProcessedUser(id) }
```

### Pattern: Sealed Error Hierarchies
```kotlin
sealed interface PaymentError {
    data class InsufficientFunds(val available: Money) : PaymentError
    data class CardExpired(val date: YearMonth) : PaymentError
    data object NetworkError : PaymentError
}

fun processPayment(amount: Money): Result<Transaction> = /* ... */

// Exhaustive error handling
processPayment(amount).fold(
    onSuccess = { showSuccess(it) },
    onFailure = { error ->
        when (error) {
            is PaymentError.InsufficientFunds -> showInsufficientFunds(error.available)
            is PaymentError.CardExpired -> promptNewCard()
            is PaymentError.NetworkError -> retry()
        }
    }
)
```

### Pattern: Validation with Accumulation
```kotlin
sealed interface Validated<out T> {
    data class Valid<T>(val value: T) : Validated<T>
    data class Invalid(val errors: List<ValidationError>) : Validated<Nothing>
}

fun validateAll(fields: Map<String, String>): Validated<ValidatedData> {
    val errors = mutableListOf<ValidationError>()

    val email = validateEmail(fields["email"])
        .onFailure { errors.add(it) }
        .getOrNull()

    val password = validatePassword(fields["password"])
        .onFailure { errors.add(it) }
        .getOrNull()

    return if (errors.isEmpty() && email != null && password != null) {
        Validated.Valid(ValidatedData(email, password))
    } else {
        Validated.Invalid(errors)
    }
}
```

## Algebraic Data Types (ADTs)

### Pattern: Sum Types (Sealed Hierarchies)
```kotlin
// Type with multiple mutually exclusive cases
sealed interface OrderStatus {
    data object Pending : OrderStatus
    data class Processing(val estimatedCompletion: Instant) : OrderStatus
    data class Shipped(val trackingNumber: String) : OrderStatus
    data class Delivered(val timestamp: Instant) : OrderStatus
    data class Cancelled(val reason: String) : OrderStatus
}

// Exhaustive pattern matching
fun formatStatus(status: OrderStatus): String = when (status) {
    is OrderStatus.Pending -> "Pending"
    is OrderStatus.Processing -> "Processing (ETA: ${status.estimatedCompletion})"
    is OrderStatus.Shipped -> "Shipped (Tracking: ${status.trackingNumber})"
    is OrderStatus.Delivered -> "Delivered at ${status.timestamp}"
    is OrderStatus.Cancelled -> "Cancelled: ${status.reason}"
}
```

### Pattern: Product Types (Data Classes)
```kotlin
// Type combining multiple values
data class Address(
    val street: String,
    val city: String,
    val state: String,
    val zipCode: String
)

data class User(
    val id: UserId,
    val email: Email,
    val address: Address  // Composed product type
)
```

### Pattern: Phantom Types
```kotlin
// Types that exist only at compile time for safety
sealed class UserState
class Unverified : UserState()
class Verified : UserState()

data class User<S : UserState>(
    val id: UserId,
    val email: Email
)

fun sendVerificationEmail(user: User<Unverified>) { /* ... */ }
fun grantAccess(user: User<Verified>) { /* ... */ }

// Type system prevents calling wrong function:
// grantAccess(unverifiedUser)  // Compile error!
```

## Lazy Evaluation

### Pattern: Sequences
```kotlin
// Lazy evaluation of transformations
val result = users.asSequence()
    .filter { it.isActive }      // Not executed yet
    .map { it.email }             // Not executed yet
    .take(10)                     // Not executed yet
    .toList()                     // Executes all transformations
```

### Pattern: Infinite Sequences
```kotlin
// Generate infinite sequence
val fibonacci = generateSequence(0 to 1) { (a, b) ->
    b to (a + b)
}.map { it.first }

val first10 = fibonacci.take(10).toList()
// [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]

// Exponential backoff
val backoff = generateSequence(100L) { it * 2 }
val delays = backoff.take(5).toList()
// [100, 200, 400, 800, 1600]
```

### Pattern: Memoization
```kotlin
fun <A, R> memoize(f: (A) -> R): (A) -> R {
    val cache = mutableMapOf<A, R>()
    return { input -> cache.getOrPut(input) { f(input) } }
}

fun expensiveComputation(n: Int): Int {
    Thread.sleep(1000)
    return n * n
}

val memoized = memoize(::expensiveComputation)

memoized(10)  // Takes 1 second
memoized(10)  // Instant (cached)
```

## Functor Pattern

### Definition
A type `F<T>` is a **functor** if you can `map` over it:
```kotlin
fun <T, R> F<T>.map(f: (T) -> R): F<R>
```

### Examples in Kotlin
```kotlin
// List is a functor
val numbers = listOf(1, 2, 3)
val doubled = numbers.map { it * 2 }  // [2, 4, 6]

// Result is a functor
val result: Result<Int> = Result.success(42)
val doubled = result.map { it * 2 }  // Result.success(84)

// Nullable is a functor
val value: Int? = 42
val doubled = value?.let { it * 2 }  // 84
```

### Laws
Functors must satisfy:
1. **Identity**: `fx.map { it } == fx`
2. **Composition**: `fx.map(f).map(g) == fx.map { g(f(it)) }`

## Monad Pattern

### Definition
A type `M<T>` is a **monad** if you can:
1. **Unit/Pure**: Wrap a value: `M.pure(value): M<T>`
2. **flatMap/bind**: Chain operations: `M<T>.flatMap(f: (T) -> M<R>): M<R>`

### Examples in Kotlin
```kotlin
// Result is a monad
fun validateEmail(email: String): Result<Email> = /* ... */
fun sendWelcome(email: Email): Result<Unit> = /* ... */

val result = validateEmail(input)
    .flatMap { email -> sendWelcome(email) }

// Nullable is a monad
fun findUser(id: Long): User? = /* ... */
fun getEmail(user: User): Email? = /* ... */

val email = findUser(id)?.let { user -> getEmail(user) }

// List is a monad
val numbers = listOf(1, 2, 3)
val expanded = numbers.flatMap { n -> listOf(n, n * 10) }
// [1, 10, 2, 20, 3, 30]
```

### Laws
Monads must satisfy:
1. **Left identity**: `M.pure(a).flatMap(f) == f(a)`
2. **Right identity**: `m.flatMap { M.pure(it) } == m`
3. **Associativity**: `m.flatMap(f).flatMap(g) == m.flatMap { f(it).flatMap(g) }`

### Pattern: For Comprehension (Manual)
```kotlin
// Simulates Scala's for-comprehension
fun registerAndWelcome(credentials: Credentials): Result<User> {
    return validateEmail(credentials.email)
        .flatMap { email ->
            validatePassword(credentials.password)
                .flatMap { password ->
                    createUser(email, password)
                        .flatMap { user ->
                            sendWelcomeEmail(user)
                                .map { user }
                        }
                }
        }
}
```

## Fold Pattern

### Definition
**Fold** (reduce) collapses a collection into a single value using a binary operation.

### Pattern: Left Fold
```kotlin
// Processes from left to right
val sum = numbers.fold(0) { acc, n -> acc + n }

val total = items.fold(Money(0)) { acc, item ->
    acc + item.price
}
```

### Pattern: Right Fold
```kotlin
// Processes from right to left
val result = numbers.foldRight(0) { n, acc -> n - acc }
```

### Pattern: Fold on Result
```kotlin
result.fold(
    onSuccess = { value -> processValue(value) },
    onFailure = { error -> handleError(error) }
)
```

### Pattern: Fold for Error Handling
```kotlin
sealed interface Outcome<out T> {
    data class Success<T>(val value: T) : Outcome<T>
    data class Failure(val errors: List<Error>) : Outcome<Nothing>
}

fun <T, R> Outcome<T>.fold(
    onSuccess: (T) -> R,
    onFailure: (List<Error>) -> R
): R = when (this) {
    is Outcome.Success -> onSuccess(value)
    is Outcome.Failure -> onFailure(errors)
}
```

## Recursion Patterns

### Pattern: Tail Recursion
```kotlin
// Tail-recursive factorial
tailrec fun factorial(n: Int, acc: Int = 1): Int =
    if (n <= 1) acc
    else factorial(n - 1, n * acc)

// Tail-recursive sum
tailrec fun sum(numbers: List<Int>, acc: Int = 0): Int =
    if (numbers.isEmpty()) acc
    else sum(numbers.drop(1), acc + numbers.first())
```

### Pattern: Mutual Recursion
```kotlin
tailrec fun isEven(n: Int): Boolean =
    if (n == 0) true
    else isOdd(n - 1)

tailrec fun isOdd(n: Int): Boolean =
    if (n == 0) false
    else isEven(n - 1)
```

## Smart Constructors

### Pattern: Private Constructor with Validation
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

// Invalid emails are unrepresentable:
// val bad = Email("invalid")  // Compile error!
val good = Email.of("valid@example.com")  // Ok
```

## Optics Patterns

### Pattern: Lenses (Manual)
```kotlin
// Lens for deeply nested updates
data class Address(val city: String, val zipCode: String)
data class User(val name: String, val address: Address)

// Without lens (verbose):
val updated = user.copy(
    address = user.address.copy(
        city = "New City"
    )
)

// With lens:
data class Lens<S, A>(
    val get: (S) -> A,
    val set: (S, A) -> S
)

val addressLens = Lens<User, Address>(
    get = { it.address },
    set = { user, address -> user.copy(address = address) }
)

val cityLens = Lens<Address, String>(
    get = { it.city },
    set = { address, city -> address.copy(city = city) }
)

fun <S, A, B> Lens<S, A>.compose(other: Lens<A, B>): Lens<S, B> =
    Lens(
        get = { s -> other.get(this.get(s)) },
        set = { s, b -> this.set(s, other.set(this.get(s), b)) }
    )

val userCityLens = addressLens.compose(cityLens)
val updated = userCityLens.set(user, "New City")
```

## Summary Table

| Pattern | Purpose | Kotlin Feature |
|---------|---------|----------------|
| Pure Functions | Deterministic, no side effects | Regular functions |
| Immutability | Unchangeable data | `val`, `data class` |
| Higher-Order Functions | Functions as values | Function types, lambdas |
| Result Type | Type-safe error handling | `Result<T>` |
| Sealed Hierarchies | Exhaustive sum types | `sealed interface` |
| Lazy Evaluation | Deferred computation | `Sequence`, `lazy` |
| Functor | Map over container | `map` |
| Monad | Chain dependent operations | `flatMap` |
| Fold | Reduce to single value | `fold`, `reduce` |
| Tail Recursion | Stack-safe recursion | `tailrec` |
| Smart Constructors | Validated construction | Private constructor, companion object |
| Value Classes | Zero-cost type safety | `@JvmInline value class` |

## Further Reading

- Arrow library (advanced FP): https://arrow-kt.io/
- Kotlin functional programming guide: https://kotlinlang.org/docs/fun-interfaces.html
- Railway-oriented programming: https://fsharpforfunandprofit.com/rop/
