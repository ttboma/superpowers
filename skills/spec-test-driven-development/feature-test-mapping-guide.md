# Feature-to-Test Mapping Guide

How to systematically derive test cases from features and their spec requirements.

## Core Principle

Every test answers one question: "Does this feature work as the spec says it should?"

Tests are derived from spec requirements, not from implementation. You test WHAT the spec says, not HOW you built it.

## From Spec Requirement to Test

### EARS Pattern Mapping

Spec requirements often follow patterns. Each pattern maps to a test structure:

**Event-driven:** `WHEN [event] THEN [system] SHALL [response]`
```
Test structure:
  Arrange: system in default state
  Act: trigger [event]
  Assert: [response] occurred
```

**Condition-based:** `IF [precondition] THEN [system] SHALL [response]`
```
Test structure:
  Arrange: establish [precondition]
  Act: trigger relevant action
  Assert: [response] occurred
```

**Combined:** `WHEN [event] AND [condition] THEN [system] SHALL [response]`
```
Test structure:
  Arrange: establish [condition]
  Act: trigger [event]
  Assert: [response] occurred
```

**Negative (derived):** For every SHALL, derive a SHALL NOT
```
Original: WHEN valid email THEN system SHALL create account
Negative: WHEN invalid email THEN system SHALL NOT create account
Boundary: WHEN email is exactly at max length THEN system SHALL create account
```

### Worked Example

**Feature F1: User Registration**

**Spec requirements:**
1. WHEN user provides valid email and password THEN system SHALL create new account
2. WHEN user provides existing email THEN system SHALL display "email already registered" error
3. IF password is shorter than 8 characters THEN system SHALL display "password too short" error
4. WHEN account creation succeeds THEN system SHALL send confirmation email

**Derived tests:**

| Test ID | Test Name | Derived From | Type |
|---------|-----------|-------------|------|
| F1.T1 | test_creates_account_with_valid_credentials | Req 1 (happy path) | unit |
| F1.T2 | test_rejects_duplicate_email | Req 2 (error case) | unit |
| F1.T3 | test_rejects_short_password | Req 3 (validation) | unit |
| F1.T4 | test_sends_confirmation_on_success | Req 4 (side effect) | integration |
| F1.T5 | test_rejects_empty_email | Req 1 (boundary — implied) | unit |
| F1.T6 | test_rejects_empty_password | Req 3 (boundary — implied) | unit |
| F1.T7 | test_accepts_password_exactly_8_chars | Req 3 (boundary) | unit |

**Test skeletons (assertion-first):**

```python
# F1.T1 — from Req 1
def test_creates_account_with_valid_credentials():
    result = create_account("user@example.com", "secure123")
    assert result.success is True
    assert result.account.email == "user@example.com"

# F1.T2 — from Req 2
def test_rejects_duplicate_email():
    create_account("user@example.com", "secure123")
    result = create_account("user@example.com", "other456")
    assert result.success is False
    assert result.error == "email already registered"

# F1.T3 — from Req 3
def test_rejects_short_password():
    result = create_account("user@example.com", "short")
    assert result.success is False
    assert result.error == "password too short"

# F1.T7 — from Req 3 (boundary)
def test_accepts_password_exactly_8_chars():
    result = create_account("user@example.com", "exactly8")
    assert result.success is True
```

## Test Types per Feature

Most features need multiple test types:

### Happy path (required)
The feature works as specified with valid inputs. At least one per feature.

### Edge cases (required for any feature with inputs)
Boundary values, minimum/maximum, empty inputs, exact limits. Derived from the constraints in the spec.

### Error cases (required for any feature that can fail)
Invalid inputs, precondition violations, resource exhaustion. Derived from the spec's error-handling requirements or implied by the valid-input requirements.

### Integration tests (for features that interact with other components)
End-to-end verification that the feature works in context, not just in isolation. One per feature that crosses component boundaries.

## Traceability Matrix Format

```markdown
## Traceability Matrix

### F1: [Feature Name]
**Spec ref:** [document, section/page]
**Acceptance criteria:**
- WHEN [condition] THEN [system] SHALL [behavior]
- ...

**Tests:**

| Test ID | Test Name | What it verifies | Type |
|---------|-----------|-----------------|------|
| F1.T1 | test_[behavior]_[condition] | [spec assertion] | unit |
| F1.T2 | test_[behavior]_[edge_case] | [boundary check] | unit |
| F1.T3 | test_[error_scenario] | [error handling] | unit |
| F1.T4 | test_[integration] | [cross-component] | integration |
```

**Completeness check:** After building the matrix:
- Every feature has ≥1 test (most have 3+)
- Every spec requirement is covered by ≥1 test
- Every test traces to exactly one feature
- No orphan tests (tests without feature mapping)
- No orphan features (features without tests)

## Test Skeleton Conventions

### Assertion-first design

Write the assertion BEFORE arrange/act. This forces you to think about what you're verifying before how to set it up.

Think: "What should be true?" → "What action produces this?" → "What setup is needed?"

### Naming

`test_<behavior>_<condition>`

- `test_creates_account_with_valid_email` — behavior + condition
- `test_rejects_password_shorter_than_8` — behavior + specific boundary
- `test_returns_error_when_connection_fails` — behavior + error scenario

Names describe spec behavior, not implementation details. Never: `test_hashmap_insert`, `test_sql_query`.

### One behavior per test

Each test verifies one thing. If a test name contains "and", split it.

- Bad: `test_validates_email_and_creates_account`
- Good: `test_validates_email_format` + `test_creates_account_with_valid_email`

### Concrete values

Test skeletons use concrete values, not abstract descriptions.

- Bad: `assert result.error == "[appropriate error message]"`
- Good: `assert result.error == "email already registered"`

If you don't know the concrete value from the spec, that's an ambiguity — ask your human partner.

## Common Mistakes

### Testing implementation, not spec behavior
The spec says "sort items by date." You test that a specific sorting algorithm is used. Instead, test that the output is sorted by date — the spec doesn't care how.

### Missing negative tests
You derived 5 happy-path tests from the spec. But you didn't derive any tests for what happens when inputs are invalid, connections fail, or resources are exhausted. The spec often implies these by defining valid behavior.

### One-test-per-feature syndrome
Every feature has exactly one test. This means you only tested the happy path. Go back to the spec — what about edge cases? Error cases? Boundary values?

### Placeholder assertions
"Assert appropriate behavior" or "Check result is correct" are not tests. If you can't write a concrete assertion, the spec requirement is unclear — clarify with your human partner.

### Test names that describe implementation
`test_database_insert` tells you nothing about what the feature should do. `test_stores_user_profile` tells you the feature's purpose. Name tests from the spec's perspective.
