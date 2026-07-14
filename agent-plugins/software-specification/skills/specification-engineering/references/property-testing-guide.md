# Property-Based Testing Guide

A guide to identifying properties, writing generators, and implementing property-based specifications using the language's standard random library (no heavy external dependencies).

## What is Property-Based Testing?

Property-based testing verifies that **invariants hold for all inputs**, not just specific examples. Instead of writing:

```
test "save then find returns the user":
    user = User(name: "John", email: "john@example.com")
    saved = repository.save(user)
    assertEquals(saved, repository.findById(saved.id))
```

Write:

```
test "roundtrip property - save then find returns equivalent user":
    for 100 iterations:
        user = randomUser()  // random valid user
        saved = repository.save(user)
        assertEquals(saved, repository.findById(saved.id))
```

This tests the **property** (roundtrip equivalence) with many random inputs, increasing confidence and discovering edge cases.

## Identifying Properties

### Roundtrip Properties

Operations that convert data and back should return equivalent values.

**Pattern**: `decode(encode(x)) == x` or `load(save(x)) == x`

**Where to look**:
- Serialization/deserialization (JSON, XML, binary)
- Repository save/load operations
- Encoding/decoding (Base64, URL encoding)
- Format conversions (string parsing, date formatting)
- Compression/decompression

**Examples**:
```
decode(encode(entity)) == entity
repository.findById(repository.save(entity).id) == savedEntity
parse(format(date)) == date
```

### Idempotence Properties

Operations that, when applied multiple times, have the same effect as applying once.

**Pattern**: `f(f(x)) == f(x)` or `f(x); f(x)` has same state as `f(x)`

**Where to look**:
- Delete operations
- Normalization functions (trim, lowercase, sort)
- Cache operations (ensure cached, invalidate)
- Set operations (add to set multiple times)
- Cleanup/reset operations

**Examples**:
```
repository.delete(id)
repository.delete(id)  // should not throw, state unchanged

normalize(normalize(text)) == normalize(text)
sort(sort(list)) == sort(list)
```

### Commutativity Properties

Operations where the order of arguments doesn't affect the result.

**Pattern**: `f(a, b) == f(b, a)`

**Where to look**:
- Set operations (union, intersection)
- Mathematical operations (addition, multiplication)
- Merge operations
- Comparison functions (min, max)

**Examples**:
```
setA.union(setB) == setB.union(setA)
merge(configA, configB) == merge(configB, configA)
```

### Associativity Properties

Operations where grouping doesn't affect the result.

**Pattern**: `f(f(a, b), c) == f(a, f(b, c))`

**Where to look**:
- String concatenation
- List concatenation
- Reduction operations

**Examples**:
```
(a + b) + c == a + (b + c)
```

### Invariant Properties

Conditions that must always be true after any operation.

**Pattern**: `invariant(state)` is always true

**Where to look**:
- State validity (accounts have positive balance, counts are non-negative)
- Collection constraints (sorted order maintained, no duplicates in set)
- Business rules (order total equals sum of line items)
- Temporal ordering (created date before modified date)

**Examples**:
```
repository.count() >= 0
order.total == sum(item.price * item.quantity for item in order.items)
sortedList.add(item); sortedList == sorted(sortedList)
```

### Inverse Properties

Operations that undo each other.

**Pattern**: `g(f(x)) == x` where g is the inverse of f

**Where to look**:
- Encrypt/decrypt
- Compress/decompress
- Encode/decode
- Add/remove operations

**Examples**:
```
decrypt(encrypt(plaintext, key), key) == plaintext

list.add(item)
list.remove(item)
list == originalList
```

## Writing Generators

### Basic Generators

Functions that produce random values using the standard library:

```
// String generators
function randomAlphaChar() -> Char:
    return random choice from 'a'..'z'

function randomAlphaString(length: 1..50) -> String:
    return string of randomAlphaChar() repeated random(length) times

function randomAlphaNumericString(length: 1..50) -> String:
    return string of random choice from [a-zA-Z0-9] repeated random(length) times

// Number generators
function randomPositiveInt(max) -> Int:
    return random integer in [1, max]

function randomNonNegativeInt(max) -> Int:
    return random integer in [0, max]

function randomMoney(max: 10000.0) -> Double:
    return random double rounded to 2 decimal places
```

### Domain-Specific Generators

Compose basic generators into domain objects:

```
function randomEmail() -> String:
    return "{randomAlphaString(3..10)}@{randomAlphaString(3..8)}.{randomAlphaString(2..3)}"

function randomUser() -> User:
    return User(
        id: randomPositiveInt(),
        name: randomAlphaString(1..50),
        email: randomEmail(),
        createdAt: randomPastDate()
    )

function randomOrder() -> Order:
    return Order(
        id: randomPositiveInt(),
        customerId: randomPositiveInt(),
        items: randomOrderItems(1..5),
        status: randomChoice(OrderStatus.values)
    )
```

### Constrained Generators

Generate values that satisfy constraints:

```
function randomValidEmail() -> String:
    local = randomAlphaNumericString(3..20)
    domain = randomAlphaString(3..10)
    tld = randomChoice(["com", "org", "net", "io"])
    return "{local}@{domain}.{tld}"

function randomNonEmptyList(maxSize: 10, generator) -> List:
    size = random integer in [1, maxSize]
    return [generator() for _ in 1..size]

function randomUniqueList(maxSize: 10, keySelector, generator) -> List:
    // generate items, deduplicating by key, up to maxSize
```

### Edge Case Generators

Explicitly include edge cases:

```
function randomStringWithEdgeCases() -> String:
    choice = random integer in [0, 9]
    if choice == 0: return ""              // empty
    if choice == 1: return " "             // single space
    if choice == 2: return "   "           // multiple spaces
    if choice == 3: return randomAlphaString(1..1)  // single char
    if choice == 4: return randomAlphaString(100..200) // long string
    else: return randomAlphaString(1..50)  // normal
```

## Property Test Structure

### Basic Pattern

```
class ModulePropertySpec:
    random = seededRandom(42)  // deterministic

    // Generators defined here

    test "property name - description of what must hold":
        for 100 iterations:
            // Generate
            input = randomInput()

            // Act
            result = operation(input)

            // Assert invariant
            assertTrue(invariant(result), "Failed for input: {input}")
```

### Roundtrip Example

```
test "roundtrip - serialize then deserialize returns equivalent":
    for 100 iterations:
        original = randomUser()
        serialized = serializer.serialize(original)
        deserialized = serializer.deserialize(serialized)
        assertEquals(original, deserialized, "Failed for: {original}")
```

### Idempotence Example

```
test "idempotence - normalizing twice equals normalizing once":
    for 100 iterations:
        input = randomAlphaString()
        once = normalizer.normalize(input)
        twice = normalizer.normalize(once)
        assertEquals(once, twice, "Not idempotent for: {input}")
```

### Invariant Example

```
test "invariant - order total always equals sum of line items":
    for 100 iterations:
        order = randomOrder()
        expectedTotal = sum(item.quantity * item.unitPrice for item in order.items)
        assertApproxEquals(expectedTotal, order.calculateTotal(), tolerance: 0.01)
```

### Stateful Property Example

```
test "invariant - repository count matches actual items":
    repository = InMemoryRepository()

    for 100 iterations:
        // Random operation
        choice = random integer in [0, 2]
        if choice == 0: repository.save(randomEntity())
        if choice == 1: pick random existing entity, delete it
        if choice == 2: no-op

        // Invariant must hold after every operation
        count = repository.count()
        actualSize = repository.findAll().size
        assertEquals(actualSize, count, "Count mismatch")
        assertTrue(count >= 0, "Negative count")
```

## Best Practices

### Use Deterministic Seeds

Seed your random generator with a fixed value (e.g., 42) so failures are reproducible. The same seed produces the same sequence of values every run.

### Include Input in Failure Messages

Always include the input that caused the failure in assertion messages. This makes debugging much easier.

### Start with Fewer Iterations

Use ~100 iterations for fast feedback during development. Increase to ~1000 for CI/thorough testing.

### Test One Property Per Test

Each test should verify a single property. Don't mix roundtrip, idempotence, and invariant checks in one test.

### Name Tests as Property Statements

Use descriptive names that state the property:
- "roundtrip - encode then decode returns original"
- "idempotence - trim applied twice equals trim once"
- "invariant - balance never negative after withdrawal"

## Common Property Categories by Module Type

### Repository/Storage
- Roundtrip: save then find
- Idempotence: delete
- Invariant: count >= 0, findAll size matches count

### Serialization/Encoding
- Roundtrip: encode/decode, serialize/deserialize
- Invariant: encoded output is valid format

### Validation
- Invariant: valid input always accepted
- Invariant: invalid input always rejected
- Idempotence: validation result same on repeated calls

### State Machines
- Invariant: only valid transitions allowed
- Invariant: state always in defined set
- Idempotence: repeated events in stable state

### Collections/Aggregates
- Invariant: computed values match item values
- Commutativity: merge operations (if applicable)
- Associativity: combine operations (if applicable)

### Caching
- Idempotence: get from cache multiple times
- Roundtrip: put then get
- Invariant: cache size within limits
