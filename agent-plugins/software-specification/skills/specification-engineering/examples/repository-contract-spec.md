# Repository Contract Specification Example

Language-neutral contract specification for a `UserRepository` interface, derived from an architecture document.

## Source Architecture

From `docs/architecture/user-management.md`:

```markdown
### Module: UserRepository
- **Responsibility**: Persist and retrieve user entities
- **Interface**: CRUD operations for User entities
- **Hidden Complexity**: Storage mechanism, ID generation, caching
- **Depth Score**: Deep
```

## Contract Specification

```
// File: UserRepositoryContractSpec.<ext>
// Package/module: specifications.usermanagement.contracts

// Import: test framework assertions

/**
 * Contract specification for UserRepository interface.
 *
 * Verifies the formal contract that any UserRepository implementation must satisfy.
 * These contracts define WHAT the repository must do, not HOW it does it.
 *
 * Architecture: docs/architecture/user-management.md
 * Module: UserRepository
 */
class UserRepositoryContractSpec {

    // Replace with real implementation when available
    sut: UserRepository = unimplemented("Provide UserRepository implementation")

    // --- save ---

    test "save returns the saved entity with generated id" {
        // Arrange
        user = User(id: null, name: "John Doe", email: "john@example.com")

        // Act
        saved = sut.save(user)

        // Assert
        assertNotNull(saved.id, "Saved entity must have a generated ID")
        assertEquals(user.name, saved.name)
        assertEquals(user.email, saved.email)
    }

    test "save with existing id updates the entity" {
        // Arrange
        original = sut.save(User(id: null, name: "Original", email: "original@example.com"))
        modified = original.copy(name: "Modified")

        // Act
        updated = sut.save(modified)

        // Assert
        assertEquals(original.id, updated.id, "ID should not change on update")
        assertEquals("Modified", updated.name)
    }

    test "save throws for invalid user" {
        // Arrange
        invalidUser = User(id: null, name: "", email: "not-an-email")

        // Act & Assert
        assertThrows(IllegalArgumentException) {
            sut.save(invalidUser)
        }
    }

    test "save throws for duplicate email" {
        // Arrange
        sut.save(User(id: null, name: "First User", email: "duplicate@example.com"))
        duplicate = User(id: null, name: "Second User", email: "duplicate@example.com")

        // Act & Assert
        assertThrows(DuplicateEmailError) {
            sut.save(duplicate)
        }
    }

    // --- findById ---

    test "findById returns the entity for existing id" {
        // Arrange
        saved = sut.save(User(id: null, name: "Test User", email: "test@example.com"))

        // Act
        found = sut.findById(saved.id)

        // Assert
        assertNotNull(found)
        assertEquals(saved, found)
    }

    test "findById returns null for non-existent id" {
        // Arrange
        nonExistentId = UserId(999999)

        // Act
        result = sut.findById(nonExistentId)

        // Assert
        assertNull(result, "Should return null for non-existent ID")
    }

    // --- findByEmail ---

    test "findByEmail returns the entity for existing email" {
        // Arrange
        saved = sut.save(User(id: null, name: "Email Test", email: "findme@example.com"))

        // Act
        found = sut.findByEmail("findme@example.com")

        // Assert
        assertNotNull(found)
        assertEquals(saved, found)
    }

    test "findByEmail returns null for non-existent email" {
        // Act
        result = sut.findByEmail("nonexistent@example.com")

        // Assert
        assertNull(result)
    }

    test "findByEmail is case-insensitive" {
        // Arrange
        saved = sut.save(User(id: null, name: "Case Test", email: "CaseTest@Example.com"))

        // Act
        found = sut.findByEmail("casetest@example.com")

        // Assert
        assertNotNull(found)
        assertEquals(saved.id, found.id)
    }

    // --- findAll ---

    test "findAll returns empty list when repository is empty" {
        // Act
        result = sut.findAll()

        // Assert
        assertTrue(result.isEmpty())
    }

    test "findAll returns all saved entities" {
        // Arrange
        user1 = sut.save(User(id: null, name: "User 1", email: "user1@example.com"))
        user2 = sut.save(User(id: null, name: "User 2", email: "user2@example.com"))
        user3 = sut.save(User(id: null, name: "User 3", email: "user3@example.com"))

        // Act
        result = sut.findAll()

        // Assert
        assertEquals(3, result.size)
        assertTrue(result.contains(user1))
        assertTrue(result.contains(user2))
        assertTrue(result.contains(user3))
    }

    // --- delete ---

    test "delete removes the entity" {
        // Arrange
        saved = sut.save(User(id: null, name: "To Delete", email: "delete@example.com"))

        // Act
        sut.delete(saved.id)

        // Assert
        assertNull(sut.findById(saved.id))
    }

    test "delete is idempotent" {
        // Arrange
        saved = sut.save(User(id: null, name: "Idempotent Delete", email: "idemp@example.com"))
        id = saved.id

        // Act - delete twice
        sut.delete(id)
        sut.delete(id) // should not throw

        // Assert
        assertNull(sut.findById(id))
    }

    test "delete for non-existent id does not throw" {
        // Arrange
        nonExistentId = UserId(999999)

        // Act & Assert - should not throw
        sut.delete(nonExistentId)
    }

    // --- count ---

    test "count returns zero for empty repository" {
        // Act
        count = sut.count()

        // Assert
        assertEquals(0, count)
    }

    test "count returns correct number after saves and deletes" {
        // Arrange
        user1 = sut.save(User(id: null, name: "Count 1", email: "count1@example.com"))
        sut.save(User(id: null, name: "Count 2", email: "count2@example.com"))
        sut.save(User(id: null, name: "Count 3", email: "count3@example.com"))
        sut.delete(user1.id)

        // Act
        count = sut.count()

        // Assert
        assertEquals(2, count)
    }

    // --- exists ---

    test "exists returns true for existing id" {
        // Arrange
        saved = sut.save(User(id: null, name: "Exists Test", email: "exists@example.com"))

        // Act
        exists = sut.exists(saved.id)

        // Assert
        assertTrue(exists)
    }

    test "exists returns false for non-existent id" {
        // Arrange
        nonExistentId = UserId(999999)

        // Act
        exists = sut.exists(nonExistentId)

        // Assert
        assertEquals(false, exists)
    }
}
```

## Key Contract Patterns Demonstrated

### 1. Save Contract
- Returns entity with generated ID
- Updates existing entity (same ID)
- Throws for invalid input
- Throws for business rule violations (duplicate email)

### 2. Find Contract
- Returns entity for existing key
- Returns null for non-existent key
- Handles case-insensitivity where specified

### 3. Delete Contract
- Removes the entity
- Is idempotent (can be called multiple times)
- Does not throw for non-existent entities

### 4. Query Contract
- Returns empty collection when no data
- Returns all matching entities
- Count matches actual entity count

## Notes

- Each method's contract is documented in its own test group.
- Tests use descriptive names that read as specification statements.
- No implementation details leak into the tests.
- `unimplemented("Provide implementation")` placeholder allows specs to compile but fail meaningfully.
- Error conditions are explicitly specified with expected error types.
