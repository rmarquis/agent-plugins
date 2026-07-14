# Behavior Specification Example

Language-neutral BDD-style behavior specification for user registration, derived from a requirements document.

## Source Requirements

From `docs/requirements/user-management.md`:

```markdown
## Functional Requirements

### FR-1: User Registration
**Description**: Users can create new accounts with email and password.
**Acceptance Criteria**:
- Given valid email and password, when user registers, then account is created
- Given email already registered, when user registers, then registration fails with duplicate error
- Given password less than 8 characters, when user registers, then registration fails with password policy error
- Given invalid email format, when user registers, then registration fails with validation error
**Priority**: Must Have

### FR-2: Email Verification
**Description**: New accounts require email verification before full access.
**Acceptance Criteria**:
- Given registration complete, when user checks account, then status is "pending verification"
- Given verification email sent, when user clicks link, then account status becomes "active"
- Given verification link expired (>24h), when user clicks link, then error shown with resend option
**Priority**: Must Have

### FR-3: Password Requirements
**Description**: Passwords must meet security requirements.
**Acceptance Criteria**:
- Password must be at least 8 characters
- Password must contain at least one uppercase letter
- Password must contain at least one number
- Password must not be in common password list
**Priority**: Must Have
```

## Behavior Specification

```
// File: UserRegistrationBehaviorSpec.<ext>
// Package/module: specifications.usermanagement.behaviors

// Import: test framework assertions

/**
 * Behavior specification for User Registration.
 *
 * Verifies acceptance criteria from requirements document.
 * Test names are derived directly from acceptance criteria language.
 *
 * Requirements: docs/requirements/user-management.md
 * Traces: FR-1, FR-2, FR-3
 */
class UserRegistrationBehaviorSpec {

    // Replace with real implementations when available
    registrationService: RegistrationService = unimplemented("Provide RegistrationService")
    userRepository: UserRepository = unimplemented("Provide UserRepository")
    verificationService: VerificationService = unimplemented("Provide VerificationService")

    // =========================================================================
    // FR-1: User Registration
    // =========================================================================

    test "given valid email and password when user registers then account is created" {
        // Given
        email = "newuser@example.com"
        password = "SecurePass123"

        // When
        result = registrationService.register(email, password)

        // Then
        assertTrue(result.isSuccess, "Registration should succeed")
        assertNotNull(result.userId, "User ID should be assigned")

        user = userRepository.findById(result.userId)
        assertNotNull(user, "User should be persisted")
        assertEquals(email, user.email)
    }

    test "given email already registered when user registers then registration fails with duplicate error" {
        // Given
        existingEmail = "existing@example.com"
        registrationService.register(existingEmail, "ValidPass123") // create existing user

        // When
        result = registrationService.register(existingEmail, "AnotherPass456")

        // Then
        assertFalse(result.isSuccess, "Registration should fail")
        assertEquals(RegistrationError.DUPLICATE_EMAIL, result.error)
        assertNull(result.userId)
    }

    test "given password less than 8 characters when user registers then registration fails with password policy error" {
        // Given
        email = "short-password@example.com"
        shortPassword = "Short1" // only 6 characters

        // When
        result = registrationService.register(email, shortPassword)

        // Then
        assertFalse(result.isSuccess)
        assertEquals(RegistrationError.PASSWORD_TOO_SHORT, result.error)
    }

    test "given invalid email format when user registers then registration fails with validation error" {
        // Given
        invalidEmail = "not-an-email"
        password = "ValidPass123"

        // When
        result = registrationService.register(invalidEmail, password)

        // Then
        assertFalse(result.isSuccess)
        assertEquals(RegistrationError.INVALID_EMAIL, result.error)
    }

    // =========================================================================
    // FR-2: Email Verification
    // =========================================================================

    test "given registration complete when user checks account then status is pending verification" {
        // Given
        result = registrationService.register("pending@example.com", "ValidPass123")
        userId = result.userId

        // When
        user = userRepository.findById(userId)

        // Then
        assertNotNull(user)
        assertEquals(AccountStatus.PENDING_VERIFICATION, user.status)
    }

    test "given verification email sent when user clicks link then account status becomes active" {
        // Given
        result = registrationService.register("verify@example.com", "ValidPass123")
        userId = result.userId
        verificationToken = verificationService.getTokenForUser(userId)

        // When
        verifyResult = verificationService.verify(verificationToken)

        // Then
        assertTrue(verifyResult.isSuccess)
        user = userRepository.findById(userId)
        assertEquals(AccountStatus.ACTIVE, user.status)
    }

    test "given verification link expired when user clicks link then error shown with resend option" {
        // Given
        result = registrationService.register("expired@example.com", "ValidPass123")
        userId = result.userId
        expiredToken = verificationService.createExpiredTokenForTesting(userId)

        // When
        verifyResult = verificationService.verify(expiredToken)

        // Then
        assertFalse(verifyResult.isSuccess)
        assertEquals(VerificationError.TOKEN_EXPIRED, verifyResult.error)
        assertTrue(verifyResult.canResend, "Should offer resend option")
    }

    // =========================================================================
    // FR-3: Password Requirements
    // =========================================================================

    test "given password without uppercase when user registers then registration fails" {
        // Given
        email = "nouppercase@example.com"
        passwordWithoutUppercase = "alllowercase123"

        // When
        result = registrationService.register(email, passwordWithoutUppercase)

        // Then
        assertFalse(result.isSuccess)
        assertEquals(RegistrationError.PASSWORD_NO_UPPERCASE, result.error)
    }

    test "given password without number when user registers then registration fails" {
        // Given
        email = "nonumber@example.com"
        passwordWithoutNumber = "NoNumberHere"

        // When
        result = registrationService.register(email, passwordWithoutNumber)

        // Then
        assertFalse(result.isSuccess)
        assertEquals(RegistrationError.PASSWORD_NO_NUMBER, result.error)
    }

    test "given password in common password list when user registers then registration fails" {
        // Given
        email = "commonpw@example.com"
        commonPassword = "Password123" // typically blocked

        // When
        result = registrationService.register(email, commonPassword)

        // Then
        assertFalse(result.isSuccess)
        assertEquals(RegistrationError.PASSWORD_TOO_COMMON, result.error)
    }

    test "given password meets all requirements when user registers then registration succeeds" {
        // Given
        email = "strongpw@example.com"
        strongPassword = "MyStr0ngP@ssword!"

        // When
        result = registrationService.register(email, strongPassword)

        // Then
        assertTrue(result.isSuccess, "Registration with strong password should succeed")
        assertNotNull(result.userId)
    }
}
```

## Key Behavior Patterns Demonstrated

### 1. Direct Acceptance Criteria Mapping
Each test name is derived directly from the acceptance criteria language:
- "Given X, when Y, then Z" becomes `given X when Y then Z`
- This creates living documentation that matches requirements

### 2. Requirement Traceability
- Class header references the requirements document
- Comment headers group tests by requirement ID (FR-1, FR-2, FR-3)
- Each test can be traced back to a specific acceptance criterion

### 3. Given-When-Then Structure
Every test follows the structure:
```
test "given <precondition> when <action> then <expected outcome>" {
    // Given - set up preconditions

    // When - perform the action

    // Then - verify outcomes
}
```

### 4. Behavior-Focused Assertions
- Assertions verify business outcomes, not implementation details
- Use result objects to check success/failure states
- Verify side effects (persisted data, status changes)

### 5. Complete Scenario Coverage
For FR-1 (registration), the spec covers:
- Happy path (valid registration)
- Business rule violation (duplicate email)
- Validation failures (short password, invalid email)

## Notes

- Test names read as complete sentences
- Requirements are traceable through comments and naming
- No implementation details leak into the tests
- Each test exercises one scenario from acceptance criteria
- `unimplemented(...)` placeholders allow specs to compile but fail meaningfully
