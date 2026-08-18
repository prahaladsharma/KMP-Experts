# Chapter 7 — Source Sets Deep Dive

## Part 2 — `commonTest`

> **If `commonMain` is where shared production behavior lives, `commonTest` is where that shared behavior earns its confidence.**

One of the biggest advantages of Kotlin Multiplatform is the ability to share production code.

But shared production code creates an equally important question:

> **How do we verify that the shared behavior works correctly across platforms?**

That is where `commonTest` becomes important.

A simplified KMP module looks like:

```text
shared/
└── src/
    ├── commonMain/
    │   └── kotlin/
    │       └── shared production code
    │
    └── commonTest/
        └── kotlin/
            └── shared tests
```

The relationship is straightforward:

```text
commonMain
     │
     ▼
commonTest
```

`commonMain` contains shared production code.

`commonTest` contains tests for behavior that can be tested in the common Kotlin environment.

The goal is not simply to create more tests.

The goal is to avoid testing the same shared business behavior independently for every platform when one common test can validate it.

---

# 1. What Is `commonTest`?

`commonTest` is the common test source set in a Kotlin Multiplatform project.

It is intended for tests that can be written using APIs available to the common source-set environment.

A typical structure is:

```text
shared/
└── src/
    ├── commonMain/
    │   └── kotlin/
    │       └── com/example/shared/
    │
    └── commonTest/
        └── kotlin/
            └── com/example/shared/
```

The important distinction is:

```text
commonMain
→ shared production code

commonTest
→ shared tests
```

---

# 2. Why `commonTest` Matters

Imagine a business rule:

```kotlin
class PriceCalculator {

    fun total(
        price: Double,
        tax: Double
    ): Double {
        return price + tax
    }
}
```

If this class lives in:

```text
commonMain
```

you do not want to create duplicate tests like:

```text
AndroidPriceCalculatorTest
IOSPriceCalculatorTest
```

for the exact same platform-independent behavior.

Instead:

```text
commonMain
    │
    ▼
PriceCalculator
    │
    ▼
commonTest
    │
    ├── calculates total
    ├── handles zero tax
    └── handles decimal values
```

The shared behavior gets a shared test.

---

# 3. Source Sets Work Together

The relationship between the two source sets can be visualized as:

```text
                 KMP MODULE
                     │
          ┌──────────┴──────────┐
          ▼                     ▼
     commonMain             commonTest
          │                     │
          ▼                     ▼
   Production Code        Shared Tests
```

A practical architecture is:

```text
commonMain
    │
    ├── domain
    ├── data
    ├── state
    └── utilities

commonTest
    │
    ├── domain tests
    ├── data tests
    ├── state tests
    └── utility tests
```

This separation keeps production and test concerns clear.

---

# 4. A Basic `commonTest`

A simple test might look like:

```kotlin
import kotlin.test.Test
import kotlin.test.assertEquals

class PriceCalculatorTest {

    private val calculator = PriceCalculator()

    @Test
    fun calculatesTotal() {
        val result = calculator.total(
            price = 100.0,
            tax = 18.0
        )

        assertEquals(118.0, result)
    }
}
```

The test uses Kotlin's multiplatform-compatible testing APIs.

The key imports are:

```kotlin
import kotlin.test.Test
import kotlin.test.assertEquals
```

This makes the test suitable for the common test environment.

---

# 5. `kotlin.test`

The Kotlin Multiplatform testing model commonly uses the `kotlin.test` library.

It provides familiar testing primitives such as:

```kotlin
@Test
assertEquals(...)
assertTrue(...)
assertFalse(...)
assertNull(...)
assertNotNull(...)
assertFails(...)
```

The benefit is that the test API is designed to work across Kotlin targets.

---

# 6. Why Not Use AndroidJUnit4 Everywhere?

An Android-specific test framework is tied to Android.

For example:

```kotlin
@RunWith(AndroidJUnit4::class)
```

belongs to an Android testing context.

It should not be placed into:

```text
commonTest
```

because `commonTest` must remain independent of Android-specific test infrastructure.

The distinction is:

```text
commonTest
→ multiplatform-compatible tests

androidTest / android-specific tests
→ Android-specific behavior

iOS-specific tests
→ iOS-specific behavior where applicable
```

---

# 7. Common Tests Are Not Android Tests

This distinction is important for Android developers moving into KMP.

A test in:

```text
commonTest
```

should not assume:

```text
Android Context
Activity
Application
Android resources
Android lifecycle
Android framework classes
```

Similarly, it should not directly depend on Apple-specific APIs.

Think:

```text
commonTest
       │
       ▼
Platform-neutral behavior
```

---

# 8. Testing Business Rules

Business rules are some of the best candidates for `commonTest`.

For example:

```kotlin
class DiscountCalculator {

    fun calculate(
        price: Double,
        percentage: Double
    ): Double {
        return price - (price * percentage / 100)
    }
}
```

Test:

```kotlin
@Test
fun appliesDiscount() {
    val calculator = DiscountCalculator()

    val result = calculator.calculate(
        price = 100.0,
        percentage = 10.0
    )

    assertEquals(90.0, result)
}
```

There is nothing Android-specific or iOS-specific here.

Therefore the test belongs naturally in:

```text
commonTest
```

---

# 9. Testing Domain Models

Suppose:

```kotlin
data class User(
    val id: String,
    val name: String
)
```

A test can validate equality and behavior without knowing anything about a platform:

```kotlin
@Test
fun createsUserCorrectly() {
    val user = User(
        id = "100",
        name = "Alex"
    )

    assertEquals("100", user.id)
    assertEquals("Alex", user.name)
}
```

These tests are simple, portable, and useful.

---

# 10. Testing Validation

Validation logic is another strong candidate.

Production code:

```kotlin
class PasswordValidator {

    fun isValid(password: String): Boolean {
        return password.length >= 8
    }
}
```

Test:

```kotlin
@Test
fun rejectsShortPassword() {
    val validator = PasswordValidator()

    assertFalse(
        validator.isValid("1234567")
    )
}
```

And:

```kotlin
@Test
fun acceptsValidPassword() {
    val validator = PasswordValidator()

    assertTrue(
        validator.isValid("12345678")
    )
}
```

The same tests can validate the shared implementation.

---

# 11. Testing State Models

Suppose common presentation logic contains:

```kotlin
sealed interface UserState {

    data object Loading : UserState

    data class Success(
        val name: String
    ) : UserState

    data class Error(
        val message: String
    ) : UserState
}
```

Tests can verify state transitions or transformation logic without touching Android or iOS UI.

```text
commonTest
    │
    ├── Loading state
    ├── Success state
    └── Error state
```

This is one of the areas where common tests can provide substantial value.

---

# 12. Testing Use Cases

Consider:

```kotlin
class GetUserUseCase(
    private val repository: UserRepository
) {
    suspend operator fun invoke(id: String): User {
        return repository.getUser(id)
    }
}
```

The use case can be tested using a fake repository:

```kotlin
class FakeUserRepository : UserRepository {

    override suspend fun getUser(id: String): User {
        return User(
            id = id,
            name = "Alex"
        )
    }
}
```

Then:

```kotlin
@Test
fun returnsUserFromRepository() {
    val repository = FakeUserRepository()
    val useCase = GetUserUseCase(repository)

    // Test the expected behavior.
}
```

The test remains completely platform-independent.

---

# 13. Fakes in `commonTest`

A fake is often useful when testing common code.

Example:

```kotlin
class FakeUserRepository : UserRepository {

    var requestedId: String? = null

    override suspend fun getUser(id: String): User {
        requestedId = id

        return User(
            id = id,
            name = "Test User"
        )
    }
}
```

This allows the test to control dependencies without bringing a platform framework into the test.

---

# 14. Why Fakes Can Be Valuable

A fake gives you:

```text
Predictable behavior
Simple setup
No external service
No platform dependency
Easy assertions
```

For example:

```text
Real API
   │
   └── unnecessary for business-rule tests

Fake API
   │
   └── predictable test behavior
```

This is especially useful in common tests because the goal is to validate shared behavior independently.

---

# 15. Testing Repository Logic

A repository may combine:

```text
API
Cache
Mapper
Business rules
```

A common repository implementation can be tested in `commonTest` when its dependencies are multiplatform-compatible.

For example:

```text
Fake API
    +
Fake storage
    +
Repository
    ↓
commonTest
```

You can verify:

```text
Successful response
Cache hit
Cache miss
Mapping
Error handling
```

without launching an Android application.

---

# 16. Testing Mappers

Mapping logic is usually an excellent candidate for common tests.

Example:

```kotlin
data class UserDto(
    val id: String,
    val fullName: String
)

data class User(
    val id: String,
    val name: String
)
```

Mapper:

```kotlin
fun UserDto.toDomain(): User {
    return User(
        id = id,
        name = fullName
    )
}
```

Test:

```kotlin
@Test
fun mapsUserCorrectly() {
    val dto = UserDto(
        id = "10",
        fullName = "Alex"
    )

    val user = dto.toDomain()

    assertEquals("10", user.id)
    assertEquals("Alex", user.name)
}
```

This is exactly the kind of test that benefits from `commonTest`.

---

# 17. Testing Serialization

If the project uses a multiplatform-compatible serialization library, serialization behavior can often be tested in `commonTest`.

For example:

```text
Domain model
     │
     ▼
Serialize
     │
     ▼
JSON
     │
     ▼
Deserialize
     │
     ▼
Domain model
```

A common test can validate the round trip.

The exact APIs depend on the serialization library and project configuration.

---

# 18. Testing Coroutines

Shared asynchronous code can also be tested in common tests when the selected coroutine testing APIs support the configured targets.

For example:

```kotlin
@Test
fun loadsUser() = runTest {
    val repository = FakeUserRepository()
    val useCase = GetUserUseCase(repository)

    val user = useCase("100")

    assertEquals("100", user.id)
}
```

The important idea is that asynchronous shared behavior can be tested without turning the test into an Android-specific test.

---

# 19. Testing `Flow`

If shared code exposes:

```kotlin
Flow<UserState>
```

the flow can be tested using multiplatform-compatible coroutine testing tools and collection patterns.

Conceptually:

```text
Action
   │
   ▼
Shared logic
   │
   ▼
Flow<State>
   │
   ▼
commonTest
```

Tests can verify:

```text
Initial state
Loading
Success
Error
State ordering
```

---

# 20. Testing Exceptions

Common code can also test expected failures.

Example:

```kotlin
@Test
fun rejectsInvalidAmount() {
    assertFails {
        validateAmount(-1.0)
    }
}
```

Or use more specific assertion patterns when the project requires them.

The key is that the error behavior belongs to the common business contract.

---

# 21. Testing Edge Cases

Shared code deserves strong edge-case coverage because one defect can affect multiple platforms.

Examples:

```text
Empty input
Null or missing values
Zero
Negative values
Boundary values
Large values
Duplicate entries
Unexpected states
Invalid configuration
```

A useful test set is:

```text
Happy Path
Boundary Cases
Invalid Input
Failure Cases
```

---

# 22. Testing Shared Logic Once

Consider:

```text
Android
    │
    ▼
commonMain
    │
    ▼
Business Rule
    ▲
    │
iOS ─┘
```

A common test verifies the rule itself.

You do not need to recreate the same business-rule test separately in Android and iOS merely because the rule is consumed by both.

This is one of the most practical benefits of shared testing.

---

# 23. But One Common Test Does Not Test Everything

A common test verifies common behavior.

It does not automatically verify:

```text
Android UI rendering
iOS UI rendering
Android permission behavior
iOS permission behavior
Platform lifecycle integration
Native SDK integration
Platform navigation
```

Those require platform-specific testing where appropriate.

Therefore:

```text
commonTest
+
platform-specific tests
```

is usually stronger than either approach alone.

---

# 24. The Testing Pyramid in KMP

A useful model is:

```text
                 UI / Integration
                /                         Android tests          iOS tests

             Shared Integration
                    │
                    ▼
               commonTest
                    │
                    ▼
             Shared Unit Tests
```

The exact distribution depends on the application.

The important principle is:

> **Test shared behavior at the common level and platform behavior at the platform level.**

---

# 25. `commonTest` and Platform Tests

A simplified project can look like:

```text
src/
├── commonMain/
├── commonTest/
├── androidMain/
├── androidUnitTest/
├── androidInstrumentedTest/
├── iosMain/
└── platform-specific tests
```

The exact directory names depend on the project setup and Gradle configuration.

The conceptual separation remains:

```text
Common behavior
→ commonTest

Platform behavior
→ platform-specific tests
```

---

# 26. Common Tests and Android Developers

An Android developer may already know:

```text
JUnit
Mockito
MockK
Robolectric
Espresso
AndroidX Test
```

The KMP question is not:

> "Which Android testing library should I move into commonTest?"

Instead:

> **"Which testing capability is available in the common Kotlin environment?"**

That change in thinking is important.

---

# 27. `commonTest` Dependency Configuration

A typical KMP configuration may include common test dependencies:

```kotlin
kotlin {
    sourceSets {
        commonTest.dependencies {
            implementation(kotlin("test"))
        }
    }
}
```

Depending on the Kotlin and Gradle setup, the exact generated configuration may differ.

The important concept is:

```text
commonTest dependencies
```

should be available to the common test compilation and compatible with the intended targets.

---

# 28. Dependency Placement Matters

Consider a test dependency that is Android-only.

Putting it into:

```text
commonTest
```

can break the multiplatform test model.

Instead:

```text
commonTest
    → multiplatform-compatible test dependency

android-specific test source set
    → Android-specific testing dependency
```

The same source-set discipline used in production code applies to tests.

---

# 29. Test Dependencies Follow the Source Set

A simple mental model:

```text
commonMain
    ↓
common dependencies

commonTest
    ↓
common test dependencies

androidMain
    ↓
Android dependencies

Android-specific tests
    ↓
Android test dependencies
```

This prevents accidental coupling.

---

# 30. Common Tests Should Be Deterministic

A common test should ideally be:

```text
Fast
Repeatable
Isolated
Deterministic
Platform-independent
```

Avoid unnecessary dependencies on:

```text
Real network
Real filesystem
Real clock
Real device
Real OS state
```

When those capabilities are required, inject an abstraction.

---

# 31. Testing Time

Time is a common source of flaky tests.

Instead of directly calling a platform clock everywhere:

```kotlin
interface Clock {
    fun now(): Instant
}
```

A fake clock can be used:

```kotlin
class FakeClock(
    private var current: Instant
) : Clock {

    override fun now(): Instant {
        return current
    }
}
```

Then the test controls time.

This is a strong example of designing common code for testability.

---

# 32. Testing Randomness

Randomness can create the same problem.

Instead of:

```kotlin
generateRandomValue()
```

inside business logic, introduce an abstraction:

```kotlin
interface RandomProvider {
    fun nextInt(): Int
}
```

The test can supply a predictable implementation.

---

# 33. Testing Network Behavior

Do not make every common test depend on a real backend.

Instead:

```text
Repository
    │
    ▼
Fake API
```

or a multiplatform-compatible mock/test server where appropriate.

Test:

```text
Success
Failure
Timeout
Invalid response
Empty response
```

without making the test suite dependent on production infrastructure.

---

# 34. Testing Storage Behavior

A common repository might depend on:

```kotlin
interface UserStorage {
    suspend fun save(user: User)
    suspend fun get(): User?
}
```

The test can use:

```kotlin
class FakeUserStorage : UserStorage {
    private var user: User? = null

    override suspend fun save(user: User) {
        this.user = user
    }

    override suspend fun get(): User? {
        return user
    }
}
```

This allows the repository logic to be tested without Android or iOS storage APIs.

---

# 35. Common Tests and Dependency Injection

Dependency injection improves testability.

Production:

```text
Repository
    ↓
Real API
Real Storage
Real Clock
```

Test:

```text
Repository
    ↓
Fake API
Fake Storage
Fake Clock
```

The production code remains unchanged.

Only the dependency graph changes.

---

# 36. Testing Error Mapping

Suppose an API returns an error.

The common repository may map it:

```text
HTTP error
    ↓
Repository
    ↓
AppError
```

Test:

```text
Fake API
    ↓
Unauthorized
    ↓
Repository
    ↓
AppError.Unauthorized
```

This mapping should often be verified in `commonTest`.

---

# 37. Testing Cache Behavior

Suppose the repository follows:

```text
Try cache
   │
   ├── found → return cached value
   │
   └── missing
          │
          ▼
        API
          │
          ▼
       Save cache
          │
          ▼
       Return data
```

This is shared business/data behavior.

It is therefore a strong candidate for `commonTest`.

---

# 38. Testing Offline Behavior

A common repository might support:

```text
Online
Offline
Cached
Stale
No cached data
```

Tests can verify the behavior without needing an actual Android or iOS device.

This is where KMP testing can provide significant confidence.

---

# 39. Common Test Naming

Good names describe behavior.

Prefer:

```kotlin
@Test
fun returnsCachedUserWhenNetworkIsUnavailable()
```

over:

```kotlin
@Test
fun test1()
```

The test name becomes documentation for the shared business contract.

---

# 40. Arrange, Act, Assert

A familiar pattern works well:

```kotlin
@Test
fun appliesDiscountCorrectly() {

    // Arrange
    val calculator = DiscountCalculator()

    // Act
    val result = calculator.calculate(
        price = 100.0,
        percentage = 20.0
    )

    // Assert
    assertEquals(80.0, result)
}
```

The structure makes common tests easy to understand.

---

# 41. Avoid Platform Details in Common Tests

A common test should not contain code like:

```kotlin
val context = ApplicationProvider.getApplicationContext<Context>()
```

or:

```kotlin
val activity = ...
```

Those belong to Android-specific testing.

Similarly, Apple framework types do not belong in common tests.

---

# 42. Common Tests and Resources

Some tests may require:

```text
JSON fixtures
Sample files
Configuration
Test data
```

When resources are required, configure them according to the KMP project's supported resource mechanism and source-set setup.

The key principle remains:

```text
Common test resources
→ must be usable by common tests across intended targets
```

---

# 43. Test Fixtures Should Be Meaningful

Avoid huge opaque fixtures.

Instead of:

```text
test-data-very-large.json
```

prefer focused fixtures:

```text
user-success.json
user-error.json
user-empty.json
```

The test should make it obvious why the fixture exists.

---

# 44. Common Tests as Executable Documentation

A good test explains:

```text
What the system does
What inputs are accepted
What outputs are expected
What happens when something fails
```

For shared business logic, this becomes especially valuable because the same contract applies to multiple platforms.

---

# 45. Common Tests and Refactoring

Suppose you move logic from:

```text
Android
```

to:

```text
commonMain
```

A strong test-first migration can look like:

```text
Existing behavior
      │
      ▼
Write characterization tests
      │
      ▼
Move/refactor logic
      │
      ▼
Run common tests
      │
      ▼
Verify platform integrations
```

This reduces the risk of changing business behavior while changing architecture.

---

# 46. A Practical Migration Example

Before KMP:

```text
Android
└── OrderCalculator.kt
```

Later:

```text
commonMain
└── OrderCalculator.kt

commonTest
└── OrderCalculatorTest.kt
```

The business logic moves once.

The tests move with the behavior.

Platform UI continues consuming the shared implementation.

---

# 47. Common Tests and API Contracts

Suppose:

```kotlin
interface PaymentValidator {
    fun validate(amount: Double): ValidationResult
}
```

The common tests can establish:

```text
Zero amount
→ invalid

Negative amount
→ invalid

Positive amount
→ valid
```

Any platform using that implementation receives the same contract.

---

# 48. Testing Shared Algorithms

Algorithms are ideal for common tests:

```text
Sorting
Filtering
Pagination
Calculation
Validation
Transformation
Parsing
Formatting
State reduction
```

For example:

```kotlin
fun paginate(
    items: List<Item>,
    page: Int,
    size: Int
): List<Item>
```

The behavior is platform-independent.

Test it once in `commonTest`.

---

# 49. Testing Shared Formatting

Formatting is more nuanced.

A purely domain-level formatter can be common:

```text
Order ID
Business status
Domain labels
```

But locale- and platform-specific formatting may require platform support.

Do not assume that every formatting problem belongs in common code.

---

# 50. Testing Localization

Localization often involves platform resources.

A common test can verify language-independent business behavior.

Platform tests may verify:

```text
Android resources
iOS resources
Platform-specific localization
```

Again:

```text
Common behavior
→ commonTest

Platform resource behavior
→ platform test
```

---

# 51. Testing UI State vs UI Rendering

This distinction is extremely useful.

Common:

```text
Button clicked
→ state changes
```

Platform-specific:

```text
Button appears correctly
```

The first may belong in:

```text
commonTest
```

The second belongs in:

```text
Android/iOS UI tests
```

This allows substantial logic coverage without making common tests UI-dependent.

---

# 52. Testing View Models and Presenters

If a ViewModel or presentation component is platform-independent, parts of its behavior may be testable in `commonTest`.

For example:

```text
Action
   ↓
ViewModel
   ↓
State
```

Test:

```text
Load
→ Loading

API success
→ Success

API failure
→ Error
```

The UI rendering itself remains outside the common test.

---

# 53. Common Tests and Flows

For reactive state:

```text
Event
   ↓
StateFlow
   ↓
Consumer
```

the test should focus on the state contract:

```text
Initial state
Expected transitions
Expected final state
```

Avoid testing UI framework behavior from `commonTest`.

---

# 54. Testing Concurrency

Shared coroutine logic can introduce concurrency issues.

Examples:

```text
Multiple requests
Cancellation
Race conditions
Duplicate events
State updates
```

These can often be tested at the common layer using appropriate multiplatform-compatible coroutine testing tools.

The goal is to verify deterministic behavior under controlled conditions.

---

# 55. Testing Cancellation

Cancellation is especially important in asynchronous common code.

A test might verify:

```text
Start operation
      ↓
Cancel operation
      ↓
No invalid state update
```

The exact implementation depends on the coroutine architecture.

The principle is:

> **If cancellation is part of the common contract, test it in the common layer.**

---

# 56. Testing Retry Logic

Retry logic is another strong common-test candidate.

Example behavior:

```text
Attempt 1 → failure
Attempt 2 → failure
Attempt 3 → success
```

A fake dependency can simulate the sequence.

The test verifies:

```text
Number of attempts
Final result
Backoff policy where applicable
Failure behavior
```

without depending on a real network.

---

# 57. Testing State Machines

State machines are particularly well suited to common tests.

```text
Idle
 │
 ▼
Loading
 │
 ├── Success
 │
 └── Error
```

Tests can verify valid transitions and reject invalid ones.

This creates a strong executable specification for shared application behavior.

---

# 58. Testing Platform Abstractions

Suppose common code depends on:

```kotlin
interface SecureStorage
```

The common test can test repository behavior with:

```kotlin
FakeSecureStorage
```

The actual Android and iOS implementations should have their own tests where platform behavior needs verification.

This gives two levels of confidence:

```text
Common tests
→ Does the application use storage correctly?

Platform tests
→ Does this platform implementation store data correctly?
```

---

# 59. Common Tests and Architecture Boundaries

A healthy architecture often produces tests that look like:

```text
commonTest
    │
    ├── Domain
    ├── Use Cases
    ├── Repository
    ├── Mappers
    ├── State
    └── Utilities
```

If every common test requires:

```text
Context
Activity
UIKit
UIViewController
```

that is often a signal that platform concerns have leaked into the common layer.

---

# 60. What `commonTest` Should Not Become

Avoid turning `commonTest` into:

```text
A dumping ground for every test.
```

Do not place platform-specific tests there just because the test file is convenient.

A clean test architecture follows the same principle as production code:

```text
Common behavior
→ commonTest

Platform behavior
→ platform-specific test source set
```

---

# 61. Common Test Checklist

Before adding a test to `commonTest`, ask:

```text
[ ] Is the production behavior in commonMain?
[ ] Is the behavior platform-independent?
[ ] Does the test use common-compatible APIs?
[ ] Are its dependencies multiplatform-compatible?
[ ] Can platform services be replaced with fakes?
[ ] Is the test deterministic?
[ ] Does it avoid real network calls?
[ ] Does it avoid real device state?
[ ] Does it verify behavior rather than implementation details?
[ ] Would the same test be meaningful for multiple targets?
```

If the answer is yes, `commonTest` is usually the right place.

---

# 62. CommonTest vs Platform Tests

| Concern | `commonTest` | Platform Test |
|---|:---:|:---:|
| Business rules | ✅ | Usually unnecessary duplicate |
| Domain models | ✅ | Usually unnecessary duplicate |
| Mappers | ✅ | Usually unnecessary duplicate |
| Use cases | ✅ | Usually unnecessary duplicate |
| Shared repository logic | ✅ | Sometimes |
| Shared state logic | ✅ | Sometimes |
| Android `Context` behavior | ❌ | ✅ |
| Android UI | ❌ | ✅ |
| iOS UIKit behavior | ❌ | ✅ |
| Platform permissions | ❌ | ✅ |
| Native SDK integration | ❌ | ✅ |
| Platform storage implementation | ❌ / limited | ✅ |
| Shared serialization | ✅ when compatible | Optional |
| Shared networking logic | ✅ when compatible | Integration tests may also be useful |

The table is a guideline, not a rigid rule.

---

# 63. A Strong KMP Testing Model

A mature KMP project can use:

```text
                         Tests
                           │
             ┌─────────────┴─────────────┐
             ▼                           ▼
        commonTest                 Platform Tests
             │                           │
     Shared behavior             Platform behavior
             │                           │
       ┌─────┴─────┐              ┌──────┴──────┐
       ▼           ▼              ▼             ▼
     Domain      Data          Android          iOS
```

This creates clear ownership.

---

# 64. CommonTest and CI

A multiplatform CI pipeline can run common tests as part of target test/build tasks.

Conceptually:

```text
Pull Request
     │
     ▼
Build
     │
     ├── Common tests
     │
     ├── Android tests
     │
     └── iOS tests
```

A failure in shared logic can therefore be detected before platform-specific validation even begins.

The exact Gradle tasks depend on the configured targets.

---

# 65. Why Common Tests Increase Confidence

Without common tests:

```text
Shared code
   │
   ├── Android works
   └── iOS works
```

may still hide gaps in shared business behavior.

With common tests:

```text
Shared code
    │
    ▼
commonTest
    │
    ├── Business rules
    ├── Data behavior
    ├── State transitions
    └── Error handling
```

the shared contract becomes explicit.

---

# 66. One Test, Multiple Consumers

This is the most important mental model:

```text
                     commonMain
                         │
                         ▼
                  Shared behavior
                         │
                         ▼
                    commonTest
                         │
             ┌───────────┴───────────┐
             ▼                       ▼
          Android                    iOS
```

The test is not "an Android test that happens to run elsewhere."

It is a test of the shared behavior itself.

---

# 67. Testing for Portability

A useful additional question is:

> **Would this test still make sense if Android did not exist?**

If yes, it is probably a strong candidate for `commonTest`.

For example:

```text
"Discount of 10% should reduce 100 to 90."
```

Yes.

But:

```text
"Activity should rotate without losing state."
```

No.

That belongs to Android-specific testing.

---

# 68. Testing for Architecture

Tests can also reveal whether the architecture is genuinely multiplatform.

If a supposedly common business rule requires:

```text
Context
Activity
UIViewController
Android resource IDs
UIKit
```

the architecture is probably mixing platform concerns into shared logic.

Common tests make that problem visible early.

---

# 69. A Useful Refactoring Signal

When a class in `commonMain` is difficult to test because it requires a platform API, ask:

```text
Can I extract the platform capability?
```

For example:

```text
Before

OrderService
   │
   └── Android Context

After

OrderService
   │
   └── Storage abstraction
          │
          ├── Android implementation
          └── iOS implementation
```

Now the common behavior becomes easy to test.

---

# 70. Testability Is an Architectural Property

A KMP architecture that is easy to test usually has:

```text
Clear boundaries
Small interfaces
Dependency injection
Limited platform leakage
Deterministic business logic
Explicit state
```

`commonTest` therefore does more than validate code.

It encourages better architecture.

---

# 71. CommonTest and Clean Architecture

A possible structure:

```text
commonMain
│
├── domain
│   ├── model
│   └── usecase
│
├── data
│   ├── repository
│   └── mapper
│
└── core
    └── utility

commonTest
│
├── domain
│   ├── model
│   └── usecase
│
├── data
│   ├── repository
│   └── mapper
│
└── core
    └── utility
```

The test structure can mirror the production structure.

This makes large projects easier to navigate.

---

# 72. Testing the Right Layer

Do not test everything at the highest level.

For example:

```text
Business calculation
→ unit test

Repository behavior
→ common unit/integration-style test

Android database implementation
→ Android-specific test

Android screen
→ Android UI test

iOS native integration
→ iOS-specific test
```

Testing at the correct layer improves speed and clarity.

---

# 73. CommonTest and Mocking

Mocking can be useful, but it is not the only option.

For common code, consider:

```text
Fake
Stub
Test implementation
In-memory repository
```

before introducing a mocking framework.

A simple fake is often easier to maintain and more portable.

---

# 74. When a Mocking Library Is Appropriate

A mocking library may still be useful when:

```text
Many interactions must be verified
Dependencies are large
Test setup would otherwise become excessive
The selected library supports the target/test environment
```

The important requirement is compatibility with the common test compilation.

Do not assume an Android-focused mocking library automatically works in `commonTest`.

---

# 75. Test Isolation

A common test should ideally not depend on another test.

Avoid:

```text
Test A changes global state
        ↓
Test B depends on it
```

Prefer:

```text
Test A → independent setup

Test B → independent setup
```

This becomes especially important when the same test suite participates in multiple target compilations.

---

# 76. Test Data Builders

For complex models, builders or helper functions can make common tests readable.

Example:

```kotlin
fun user(
    id: String = "1",
    name: String = "Test User"
) = User(
    id = id,
    name = name
)
```

Then:

```kotlin
val user = user(name = "Alex")
```

Keep helpers focused and understandable.

---

# 77. Avoid Over-Abstraction in Tests

Do not create:

```text
TestFrameworkFactory
CommonTestDependencyProvider
PlatformAwareAssertionManager
```

unless the project genuinely needs them.

Tests should be easier to understand than the production architecture they validate.

Prefer:

```text
Arrange
Act
Assert
```

with small reusable helpers.

---

# 78. CommonTest and Regression Protection

Once a business rule is fixed, add a common test.

For example:

```text
Bug found
   ↓
Fix common implementation
   ↓
Add regression test
   ↓
commonTest
```

Now the same defect is less likely to return on another platform.

---

# 79. Shared Bugs Need Shared Tests

Suppose a pricing bug appears on both Android and iOS.

If the root cause is:

```text
commonMain
```

the correct long-term fix is often:

```text
Fix common implementation
+
Add common regression test
```

rather than:

```text
Fix Android
+
Fix iOS
```

independently.

---

# 80. CommonTest and Refactoring Safety

Large KMP refactors can move code between:

```text
commonMain
androidMain
iosMain
```

A strong common test suite acts as a safety net.

```text
Refactor
   │
   ▼
commonTest
   │
   ├── Pass → continue validation
   └── Fail → investigate shared behavior
```

This makes source-set migration safer.

---

# 81. What `commonTest` Really Represents

At a deeper level:

```text
commonMain
=
Shared implementation

commonTest
=
Shared behavioral contract
```

The production source set answers:

> What does the application do?

The test source set answers:

> What must continue to be true?

That distinction is valuable in large multiplatform systems.

---

# 82. A Complete Example

Production:

```kotlin
data class CartItem(
    val price: Double,
    val quantity: Int
)

class CartCalculator {

    fun total(items: List<CartItem>): Double {
        return items.sumOf {
            it.price * it.quantity
        }
    }
}
```

Common test:

```kotlin
import kotlin.test.Test
import kotlin.test.assertEquals

class CartCalculatorTest {

    @Test
    fun calculatesCartTotal() {
        val calculator = CartCalculator()

        val items = listOf(
            CartItem(price = 10.0, quantity = 2),
            CartItem(price = 5.0, quantity = 3)
        )

        val result = calculator.total(items)

        assertEquals(35.0, result)
    }
}
```

The complete relationship is:

```text
commonMain
    │
    ├── CartItem
    └── CartCalculator
             │
             ▼
commonTest
    │
    └── CartCalculatorTest
```

No Android dependency.

No iOS dependency.

Just shared application behavior and its test.

---

# 83. A More Realistic Example

Consider:

```text
UserRepository
      │
      ├── UserApi
      └── UserStorage
```

All three can be represented by common abstractions.

Test setup:

```text
FakeUserApi
FakeUserStorage
      │
      ▼
UserRepository
      │
      ▼
commonTest
```

Tests can verify:

```text
1. Returns cached user
2. Fetches user when cache misses
3. Saves fetched user
4. Maps API response
5. Handles API failure
6. Returns expected domain error
```

This can provide meaningful coverage for the shared data layer.

---

# 84. The Platform Still Matters

A common test does not eliminate platform testing.

Suppose Android uses:

```text
Room
```

and iOS uses:

```text
Core Data
```

The common repository contract can be tested in:

```text
commonTest
```

But the actual platform integrations should still be validated independently.

Think in layers:

```text
Shared contract
        │
        ▼
commonTest

Android implementation
        │
        ▼
Android tests

iOS implementation
        │
        ▼
iOS tests
```

---

# 85. CommonTest and Integration Boundaries

Some shared integrations can be tested in common tests if the dependencies themselves are multiplatform-compatible.

Others may require a platform environment.

Do not force every integration into commonTest.

The question is always:

> **What behavior is actually platform-independent?**

---

# 86. CommonTest as a Contract Between Platforms

Imagine two teams:

```text
Android Team
```

and:

```text
iOS Team
```

Both consume:

```text
commonMain
```

The common tests provide a shared behavioral contract.

```text
             commonMain
                 │
                 ▼
          commonTest contract
             /                      ▼            ▼
        Android          iOS
```

This can reduce misunderstandings about how shared APIs should behave.

---

# 87. Documentation Through Tests

A test such as:

```kotlin
@Test
fun returnsCachedUserWhenNetworkIsUnavailable()
```

communicates more than implementation comments.

It tells another developer:

```text
Expected behavior:
Use cached data when network access fails.
```

Good common tests therefore become living documentation for the shared layer.

---

# 88. CommonTest and Code Review

During code review, reviewers can ask:

```text
Is this behavior shared?
```

If yes:

```text
Is it tested in commonTest?
```

This creates a healthy review habit:

```text
New common behavior
       │
       ├── Production code
       └── Common test
```

---

# 89. CommonTest and Quality Gates

A mature CI pipeline can treat common tests as a quality gate.

Conceptually:

```text
Pull Request
     │
     ▼
Compile
     │
     ▼
commonTest
     │
     ├── Fail → stop
     │
     └── Pass
           │
           ▼
   Platform validation
```

The exact CI implementation depends on the project's Gradle setup.

---

# 90. CommonTest Performance

Common tests should remain fast.

Fast tests encourage developers to run them frequently:

```text
Write code
   ↓
Run common tests
   ↓
Get feedback quickly
```

If every small business-rule change requires a full device test cycle, feedback becomes slower.

Common tests help keep shared logic inexpensive to validate.

---

# 91. CommonTest and Developer Experience

A good KMP project should make this workflow easy:

```text
Change commonMain
      ↓
Run commonTest
      ↓
Fix behavior
      ↓
Run platform builds/tests
```

This gives developers a short feedback loop before expensive platform validation.

---

# 92. A Practical Test Strategy

For a shared business feature:

```text
1. Write common production logic.
2. Write common tests.
3. Verify business behavior.
4. Add platform implementations.
5. Test platform integrations.
6. Test platform UI where required.
```

This keeps responsibilities separated.

---

# 93. CommonTest and Native Code

Native code can be perfectly valid in a KMP application.

The key is not to pretend it is common.

For example:

```text
Android native API
→ androidMain + Android test

iOS native API
→ iOS source set + iOS test

Shared abstraction
→ commonMain + commonTest
```

This is a cleaner model than trying to make every API common.

---

# 94. CommonTest and Long-Term Maintenance

A shared test suite becomes more valuable as the number of targets grows.

Imagine:

```text
Android
iOS
Desktop
Web
```

If the business logic is common:

```text
One common implementation
+
One common behavioral test suite
```

can protect the shared contract across all consumers.

---

# 95. The Wider the Sharing, the More Valuable the Tests

Consider:

```text
2 platforms
→ common behavior affects 2 consumers

4 platforms
→ common behavior affects 4 consumers

6 platforms
→ common behavior affects 6 consumers
```

As sharing increases, regressions in common code can have a larger blast radius.

Therefore:

> **The more important the common layer becomes, the more important its tests become.**

---

# 96. CommonTest Is Not About Testing Kotlin

The goal is not:

```text
"Test that Kotlin works."
```

The goal is:

```text
"Test that our shared application behavior is correct."
```

Focus on:

```text
Business outcomes
Contracts
State transitions
Data transformations
Error handling
```

rather than implementation trivia.

---

# 97. Test Behavior, Not Implementation

Avoid tests that tightly couple to private implementation details.

Prefer:

```text
Given input
When action occurs
Then expected result appears
```

For example:

```text
Given a valid order
When total is calculated
Then tax is included
```

This style survives refactoring better.

---

# 98. A CommonTest Mental Model

Remember this:

```text
                  commonMain
                      │
             Shared application logic
                      │
                      ▼
                  commonTest
                      │
             Behavioral confidence
                      │
          ┌───────────┴───────────┐
          ▼                       ▼
       Android                   iOS
```

Common tests protect the center.

Platform tests protect the edges.

---

# 99. CommonTest Review Checklist

Before considering a shared feature complete:

```text
[ ] Production logic is in the correct source set.
[ ] Common behavior has common tests.
[ ] Tests use multiplatform-compatible APIs.
[ ] Platform dependencies are isolated.
[ ] Fakes replace unnecessary external systems.
[ ] Error paths are tested.
[ ] Boundary conditions are tested.
[ ] Async behavior is tested where necessary.
[ ] Shared state transitions are covered.
[ ] Platform-specific integrations have their own tests.
[ ] Tests are deterministic.
[ ] Tests describe behavior clearly.
```

---

# 100. Final Mental Model

The source-set relationship can be reduced to one diagram:

```text
                    KMP MODULE
                        │
          ┌─────────────┴─────────────┐
          ▼                           ▼
     commonMain                  commonTest
          │                           │
          │                           │
 Shared implementation       Shared verification
          │                           │
          └─────────────┬─────────────┘
                        ▼
             Platform Applications
                  /                            ▼             ▼
             Android           iOS
```

The central idea is simple:

> **If behavior is common, implement it in `commonMain` and verify that behavior in `commonTest`.**

Platform-specific behavior should remain in platform-specific source sets and should be tested at the platform level.

---

# Chapter Takeaways

> [!TIP]
> **`commonTest` is the shared safety net for the shared KMP layer. It lets you validate platform-independent behavior once at the level where that behavior actually lives.**

Remember:

1. `commonTest` is the common test source set in KMP.
2. `commonMain` contains shared production code.
3. `commonTest` contains tests for shared behavior.
4. Common tests should use APIs and dependencies compatible with the common test environment.
5. `kotlin.test` provides multiplatform-oriented testing primitives.
6. Android-specific testing frameworks should not be placed directly into `commonTest`.
7. Business rules are excellent candidates for common tests.
8. Domain models are often easy to test in commonTest.
9. Mappers are strong common-test candidates.
10. Use cases are often best tested in commonTest.
11. Shared repository behavior can often be tested with fakes.
12. Shared state transitions can often be verified in commonTest.
13. Shared validation should generally have common tests.
14. Shared error mapping should be tested.
15. Shared caching behavior can often be tested without a real device.
16. Real network calls should generally be avoided in unit-level common tests.
17. Real platform storage should generally be replaced with a fake when testing common behavior.
18. Dependency injection improves common-code testability.
19. Fakes can be simpler and more portable than mocking frameworks.
20. A common test should not depend on Android `Context`, `Activity`, or iOS framework types.
21. Platform-specific behavior still requires platform-specific tests.
22. `commonTest` does not replace Android or iOS UI and integration testing.
23. Shared UI state can often be tested in commonTest even when UI rendering is platform-specific.
24. Compose Multiplatform can move more UI into common source sets, but UI testing remains a separate concern.
25. Common tests should be deterministic.
26. Avoid unnecessary dependencies on real clocks, random generators, network services, or device state.
27. Abstract platform capabilities when shared logic needs them.
28. Common tests can expose platform leakage in `commonMain`.
29. A difficult-to-test common class may indicate a missing abstraction boundary.
30. Test business behavior rather than implementation details.
31. Good test names document the shared contract.
32. Regression bugs in common code should generally receive common regression tests.
33. Shared tests become increasingly valuable as the number of supported platforms grows.
34. Common tests provide a fast feedback loop for shared changes.
35. CI can run common tests as an early quality gate.
36. Common tests can support architecture refactoring.
37. Common tests can reduce duplicated business-rule tests across platforms.
38. Common tests do not prove that platform integrations work correctly.
39. Platform implementations need their own validation.
40. The correct test source set depends on where the behavior actually belongs.
41. `commonTest` should not become a dumping ground for every test.
42. A test belongs in commonTest when the behavior it verifies is genuinely common.
43. Shared tests form a behavioral contract for teams consuming the KMP module.
44. `commonMain` defines shared behavior; `commonTest` protects that behavior.
45. **The goal is not to test once because there are multiple platforms. The goal is to test once because the behavior itself is shared.**

---

# Final Thought

Kotlin Multiplatform is often introduced through code sharing:

```text
Write once.
Run on multiple platforms.
```

But experienced multiplatform engineering goes one step further:

```text
Share the implementation.
Share the behavioral contract.
Keep platform-specific behavior at the edges.
```

That is where `commonTest` becomes powerful.

The architecture becomes:

```text
                  commonMain
                      │
              Shared application logic
                      │
                      ▼
                  commonTest
                      │
             Shared behavior verified
                      │
          ┌───────────┴───────────┐
          ▼                       ▼
       Android                   iOS
          │                       │
   Platform tests          Platform tests
```

The result is more than less duplicated test code.

It creates a clear boundary around what the application considers **platform-independent behavior**.

> **`commonMain` gives shared code a home. `commonTest` gives that shared code confidence.**
