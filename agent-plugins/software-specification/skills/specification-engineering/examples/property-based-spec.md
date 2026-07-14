# Property-Based Specification Example

Language-neutral property-based specification for a `UserRepository`, using the standard library random module for lightweight generators.

## Source Architecture

From `docs/architecture/user-management.md`:

```markdown
### Module: UserRepository
- **Responsibility**: Persist and retrieve user entities
- **Interface**: CRUD operations for User entities
- **Hidden Complexity**: Storage mechanism, ID generation, caching
- **Depth Score**: Deep

**Invariants**:
- Entity saved then retrieved is equivalent to saved entity
- Count is always non-negative
- Delete is idempotent
- FindAll size equals count
```

## Property Specification

```
// File: UserRepositoryPropertySpec.<ext>
// Package/module: specifications.usermanagement.properties

// Import: test framework assertions, standard library random

/**
 * Property-based specification for UserRepository.
 *
 * Verifies invariants that must hold for ALL valid inputs, not just specific examples.
 * Uses deterministic random generation for reproducible tests.
 *
 * Architecture: docs/architecture/user-management.md
 * Requirements: docs/requirements/user-management.md
 * Module: UserRepository
 */
class UserRepositoryPropertySpec {

    // Deterministic seed for reproducibility
    random = seededRandom(42)

    // Replace with real implementation when available
    function createRepository(): UserRepository = unimplemented("Provide UserRepository implementation")

    // =========================================================================
    // Generators
    // =========================================================================

    /**
     * Generate a random alphabetic string.
     */
    function randomAlphaString(length: IntRange = 1..50): String {
        count = random.nextInt(length.start, length.endInclusive + 1)
        return string of count random lowercase letters
    }

    /**
     * Generate a random valid email address.
     */
    function randomEmail(): String {
        local = randomAlphaString(3..15)
        domain = randomAlphaString(3..10)
        tld = randomChoice(["com", "org", "net", "io"])
        return "{local}@{domain}.{tld}"
    }

    /**
     * Generate a random positive long ID.
     */
    function randomPositiveLong(max: Long = Long.MAX_VALUE - 1): Long {
        return random.nextLong(1, max)
    }

    /**
     * Generate a random valid User (without ID, for saving).
     */
    function randomNewUser(): User {
        return User(
            id = null,
            name = randomAlphaString(1..50),
            email = randomEmail()
        )
    }

    /**
     * Generate a list of unique users (unique by email).
     */
    function randomUniqueUsers(count: Int): List<User> {
        emails = empty set
        users = empty list
        repeat count times {
            user = randomNewUser()
            while (user.email in emails) {
                user = randomNewUser()
            }
            emails.add(user.email)
            users.add(user)
        }
        return users
    }

    // =========================================================================
    // Roundtrip Properties
    // =========================================================================

    test "roundtrip - save then findById returns equivalent entity" {
        repository = createRepository()

        repeat(100) {
            user = randomNewUser()
            saved = repository.save(user)
            found = repository.findById(saved.id)

            assertEquals(saved, found, "Roundtrip failed for user: $user")
        }
    }

    test "roundtrip - save then findByEmail returns equivalent entity" {
        repeat(100) {
            repository = createRepository() // fresh for each iteration
            user = randomNewUser()
            saved = repository.save(user)
            found = repository.findByEmail(saved.email)

            assertEquals(saved, found, "FindByEmail roundtrip failed for: ${saved.email}")
        }
    }

    test "roundtrip - update preserves identity" {
        repository = createRepository()

        repeat(100) {
            original = repository.save(randomNewUser())
            modified = original.copy(name = randomAlphaString())
            updated = repository.save(modified)
            found = repository.findById(original.id)

            assertEquals(original.id, updated.id, "ID changed after update")
            assertEquals(updated, found, "Updated entity not retrievable")
        }
    }

    // =========================================================================
    // Idempotence Properties
    // =========================================================================

    test "idempotence - delete called twice has same effect as once" {
        repository = createRepository()

        repeat(100) {
            saved = repository.save(randomNewUser())
            id = saved.id

            // First delete
            repository.delete(id)
            countAfterFirst = repository.count()

            // Second delete - should not throw or change state
            repository.delete(id)
            countAfterSecond = repository.count()

            assertNull(repository.findById(id), "Entity should not exist after delete")
            assertEquals(countAfterFirst, countAfterSecond, "Count changed on second delete")
        }
    }

    test "idempotence - delete non-existent id has no effect" {
        repository = createRepository()

        repeat(100) {
            // Save some entities
            repeat(random.nextInt(1, 5)) {
                repository.save(randomNewUser())
            }
            countBefore = repository.count()

            // Delete non-existent
            nonExistentId = UserId(randomPositiveLong())
            repository.delete(nonExistentId)

            assertEquals(countBefore, repository.count(), "Count changed when deleting non-existent ID")
        }
    }

    // =========================================================================
    // Invariant Properties
    // =========================================================================

    test "invariant - count is never negative" {
        repository = createRepository()

        repeat(100) {
            // Random operations
            choice = random.nextInt(4)
            if choice == 0: repository.save(randomNewUser())
            if choice == 1: {
                existing = randomOrNull(repository.findAll())
                if existing != null: repository.delete(existing.id)
            }
            if choice == 2: repository.findById(UserId(randomPositiveLong()))
            if choice == 3: repository.count()

            assertTrue(repository.count() >= 0, "Count became negative")
        }
    }

    test "invariant - findAll size equals count" {
        repository = createRepository()

        repeat(100) {
            // Random operations
            repeat(random.nextInt(1, 10)) {
                choice = random.nextInt(3)
                if choice == 0: repository.save(randomNewUser())
                if choice == 1: {
                    existing = randomOrNull(repository.findAll())
                    if existing != null: repository.delete(existing.id)
                }
                if choice == 2: { /* no-op */ }
            }

            count = repository.count()
            findAllSize = repository.findAll().size

            assertEquals(count, findAllSize, "count() = $count but findAll().size = $findAllSize")
        }
    }

    test "invariant - all IDs in findAll are unique" {
        repository = createRepository()

        repeat(50) {
            // Save multiple users
            repeat(random.nextInt(5, 20)) {
                repository.save(randomNewUser())
            }

            all = repository.findAll()
            ids = all.map { it.id }
            uniqueIds = ids.toSet()

            assertEquals(ids.size, uniqueIds.size, "Duplicate IDs found in findAll()")
        }
    }

    test "invariant - saved entity always retrievable before delete" {
        repository = createRepository()

        repeat(100) {
            saved = repository.save(randomNewUser())

            // Should be retrievable immediately
            found = repository.findById(saved.id)
            assertEquals(saved, found, "Saved entity not immediately retrievable")

            // Should be in findAll
            assertTrue(repository.findAll().contains(saved), "Saved entity not in findAll()")

            // Should be findable by email
            assertEquals(saved, repository.findByEmail(saved.email), "Saved entity not findable by email")
        }
    }

    // =========================================================================
    // Bulk Operation Properties
    // =========================================================================

    test "bulk - saving multiple users maintains all invariants" {
        repeat(20) {
            repository = createRepository()
            users = randomUniqueUsers(random.nextInt(10, 50))

            savedUsers = users.map { repository.save(it) }

            // All saved
            assertEquals(users.size, repository.count())

            // All retrievable
            savedUsers.forEach { saved ->
                assertEquals(saved, repository.findById(saved.id))
            }

            // All unique IDs
            ids = savedUsers.map { it.id }.toSet()
            assertEquals(savedUsers.size, ids.size)
        }
    }
}
```

## Key Property Patterns Demonstrated

### 1. Lightweight Generators
Functions produce random domain objects:
- `randomAlphaString()` — random strings
- `randomEmail()` — valid email format
- `randomNewUser()` — complete domain object
- `randomUniqueUsers()` — list with constraints

### 2. Deterministic Reproducibility
```
random = seededRandom(42)
```
Same seed = same sequence of random values = reproducible test failures.

### 3. Roundtrip Properties
Verify that data survives persistence operations:
```
saved = repository.save(user)
found = repository.findById(saved.id)
assertEquals(saved, found)
```

### 4. Idempotence Properties
Verify that repeated operations don't change state:
```
repository.delete(id)
repository.delete(id) // second delete should not throw or change state
```

### 5. Invariant Properties
Verify conditions that must always hold:
- Count is never negative
- findAll().size equals count()
- All IDs are unique
- Saved entities are retrievable

## Notes

- Uses only the standard library random module — no external property testing libraries
- All tests use `repeat(N)` for a controlled iteration count
- Failure messages include the input that failed for debugging
- Fresh repository instances are used where needed to ensure test isolation
- Generator functions are private to the spec class
