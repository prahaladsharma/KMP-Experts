# Chapter 8 — `expect` / `actual`

## Part 5 — Best Practices

> **Good `expect` / `actual` design is less about writing platform implementations and more about creating the right boundary between shared intent and native behavior.**

As a KMP project grows, `expect` / `actual` can become either a clean architectural tool or a source of unnecessary complexity.

The difference usually comes down to how the platform boundary is designed.

This part focuses on practical principles that help keep `expect` / `actual` code:

- Small
- Predictable
- Testable
- Maintainable
- Platform-neutral
- Easy to extend
- Easy to review

---

## 1. Start With the Capability, Not the Platform

The first question should not be:

> "What Android API should I expose?"

Instead ask:

> "What capability does the common code need?"

For example, avoid designing the common API around:

```kotlin
expect fun getContext(): Any
```

If the real requirement is to store a token, define:

```kotlin
interface SecureStorage {
    fun save(key: String, value: String)
    fun read(key: String): String?
}
```

The abstraction describes the application requirement rather than the platform implementation.

### Prefer

```text
Application requirement
        ↓
Capability
        ↓
Platform implementation
```

### Avoid

```text
Platform API
        ↓
Wrapper
        ↓
Common code
```

This single decision prevents many architectural problems later.

---

# 2. Keep `expect` Declarations Small

A small expected API is easier to understand and implement.

Prefer:

```kotlin
expect fun openUrl(url: String)
```

over:

```kotlin
expect class PlatformManager {
    fun openUrl(url: String)
    fun copy(text: String)
    fun share(text: String)
    fun getDeviceModel(): String
    fun getBatteryLevel(): Int
    fun requestPermission(): Boolean
    fun vibrate()
    fun openSettings()
}
```

The second design creates a **God object**.

A better structure is:

```text
UrlOpener
Clipboard
ShareService
DeviceInfo
PermissionManager
HapticFeedback
SettingsLauncher
```

Each abstraction represents one meaningful capability.

---

# 3. Use Interfaces When They Improve Testability

`expect` / `actual` is not a replacement for interfaces.

They solve related but different problems.

An interface can represent a dependency:

```kotlin
interface SecureStorage {
    fun save(key: String, value: String)
    fun read(key: String): String?
}
```

The platform implementation can then be supplied through dependency injection.

This is especially useful when:

- The dependency is used by many classes.
- You need fakes in tests.
- Multiple implementations are possible.
- Construction should remain outside business logic.

A common pattern is:

```text
Common interface
       │
       ├── Android implementation
       ├── iOS implementation
       └── Fake implementation for tests
```

---

# 4. Use `expect` / `actual` When the Platform Boundary Is the Real Problem

Do not use `expect` / `actual` simply because it is available.

Use it when:

```text
Common code needs a capability
        +
The implementation genuinely differs by platform
```

For example:

```kotlin
expect fun appDataDirectory(): String
```

makes sense because the appropriate application data location is platform-specific.

But creating:

```kotlin
expect fun calculateDiscount(
    price: Double,
    customerType: String
): Double
```

would be a poor use of `expect` if the calculation is identical everywhere.

Business logic belongs in common code.

---

# 5. Keep Business Logic in `commonMain`

One of the most important KMP principles is:

> **Platform-specific code should implement platform behavior, not duplicate business behavior.**

Avoid:

```text
androidMain
    Login validation

iosMain
    Login validation
```

when the validation rules are identical.

Prefer:

```text
commonMain
    Login validation
```

and only isolate the platform-specific capability.

For example:

```text
commonMain
    LoginUseCase
        ↓
    SecureStorage
        ↓
    expect / actual
```

The login rules remain shared.

---

# 6. Do Not Put Platform Types in `commonMain`

Avoid exposing:

```kotlin
Context
Activity
UIViewController
UIApplication
NSObject
Android Uri
iOS-specific framework types
```

through common APIs.

For example, this is a warning sign:

```kotlin
expect fun getActivity(): Any
```

The common layer now needs to understand an object whose lifecycle and meaning are platform-specific.

Prefer:

```kotlin
expect fun openUrl(url: String)
```

The common code gets the capability it needs.

---

# 7. Prefer Intent-Based APIs

A good abstraction communicates **intent**.

### Weak

```kotlin
expect fun getNativeApplicationObject(): Any
```

### Better

```kotlin
expect fun openApplicationSettings()
```

### Weak

```kotlin
expect fun getNativeClipboard(): Any
```

### Better

```kotlin
expect fun copyToClipboard(text: String)
```

### Weak

```kotlin
expect fun getNativeLocation(): Any
```

### Better

```kotlin
expect suspend fun currentLocation(): Location?
```

Intent-based APIs are easier to understand and less coupled to platform implementation details.

---

# 8. Keep Platform Details at the Edge

A healthy architecture keeps platform-specific code close to the platform.

```text
                commonMain
                    │
             Business Logic
                    │
             Capability API
                    │
        ┌───────────┴───────────┐
        ▼                       ▼
    androidMain             iosMain
        │                       │
    Native API              Native API
```

The closer native API knowledge stays to the edge, the easier the common architecture is to maintain.

---

# 9. Avoid Leaking Native Exceptions

Native APIs can expose platform-specific exceptions.

Do not allow those exceptions to become part of the common business contract unless there is a deliberate reason.

Instead:

```text
Native exception
       ↓
Platform adapter
       ↓
Common error
       ↓
Business logic
```

For example:

```kotlin
sealed interface StorageError {
    data object AccessDenied : StorageError
    data object Unavailable : StorageError
    data object Unknown : StorageError
}
```

The platform implementation can translate its native failures into this common representation.

---

# 10. Design Common Error Models Carefully

Do not create an error model that simply mirrors Android or iOS.

Avoid:

```kotlin
sealed interface Error {
    data object AndroidKeystoreException : Error
    data object IOSKeychainException : Error
}
```

Prefer:

```kotlin
sealed interface SecureStorageError {
    data object AccessDenied : SecureStorageError
    data object StorageUnavailable : SecureStorageError
    data object Unknown : SecureStorageError
}
```

The common model represents application meaning.

---

# 11. Do Not Over-Abstraction

Not every platform difference requires `expect` / `actual`.

Suppose only Android has a feature:

```text
Android-specific UI
```

and common code never needs it.

Keep it in:

```text
androidMain
```

Do not create:

```kotlin
expect fun androidOnlyFeature()
```

just to make the project appear more multiplatform.

A useful rule is:

> **If common code does not need it, it does not need a common abstraction.**

---

# 12. Avoid Giant `Platform` Classes

A common anti-pattern is:

```kotlin
expect class Platform {
    val name: String
    val version: String
    fun openUrl(url: String)
    fun share(text: String)
    fun copy(text: String)
    fun vibrate()
    fun requestPermission(): Boolean
    fun getLocation(): Location?
    fun getDeviceModel(): String
}
```

This becomes difficult to:

- Test
- Mock
- Understand
- Extend
- Review

Instead use focused capabilities.

```text
Platform
   ↓
❌ Giant abstraction

Capabilities
   ↓
✓ UrlOpener
✓ Clipboard
✓ LocationProvider
✓ DeviceInfo
✓ HapticFeedback
```

---

# 13. Prefer Dependency Injection for Construction

Common business logic should not usually construct native implementations itself.

Avoid:

```kotlin
class UserRepository {
    private val storage = AndroidSecureStorage()
}
```

Prefer:

```kotlin
class UserRepository(
    private val storage: SecureStorage
)
```

The platform composition layer provides the implementation.

This gives you:

```text
Production
    ↓
Real platform implementation

Tests
    ↓
Fake implementation
```

---

# 14. Keep Constructors Platform-Neutral

A common class should not require:

```kotlin
Context
```

or:

```kotlin
UIViewController
```

in its constructor unless the architecture deliberately requires a platform-specific component.

Prefer:

```kotlin
class PaymentRepository(
    private val secureStorage: SecureStorage,
    private val networkMonitor: NetworkMonitor
)
```

rather than:

```kotlin
class PaymentRepository(
    private val context: Context
)
```

The second design makes the repository platform-aware.

---

# 15. Use Platform-Specific Composition Roots

The platform layer is a good place to assemble dependencies.

Conceptually:

```text
Android
   │
   ├── AndroidSecureStorage
   ├── AndroidNetworkMonitor
   └── AndroidDeviceInfo
             │
             ▼
       Common Repository
```

And:

```text
iOS
   │
   ├── IosSecureStorage
   ├── IosNetworkMonitor
   └── IosDeviceInfo
             │
             ▼
       Common Repository
```

The business layer does not need to know how dependencies are created.

---

# 16. Prefer Stable Common Models

When a native API returns a platform object, convert it into a common model where appropriate.

For example:

```kotlin
data class Location(
    val latitude: Double,
    val longitude: Double
)
```

rather than exposing a native location object.

The common model should contain only the data the application actually needs.

---

# 17. Do Not Mirror the Native API

A common mistake is to create a one-to-one wrapper around every native API.

For example:

```kotlin
interface NativeCameraWrapper {
    fun startPreview()
    fun stopPreview()
    fun setFocus()
    fun setFlash()
    fun setZoom()
    fun setExposure()
    ...
}
```

If common code only needs:

```kotlin
suspend fun capturePhoto(): ByteArray
```

then the common capability should remain small.

The native implementation can handle all the internal camera complexity.

---

# 18. Use the Platform for Complex Native Features

Some capabilities are inherently complex:

- Camera
- Bluetooth
- Audio
- Video
- Biometrics
- Location
- Background execution

Do not force every implementation detail into `commonMain`.

A better structure is:

```text
commonMain
    Feature contract
        ↓
platformMain
    Native implementation
        ↓
Native framework
```

The common layer should expose only what the shared feature actually needs.

---

# 19. Respect Platform Lifecycle Rules

Android and iOS do not have identical lifecycle models.

Do not create a fake universal lifecycle simply because it looks convenient.

Instead, define the specific event required by the common layer.

For example:

```kotlin
interface AppStateListener {
    fun onAppActive()
}
```

The platform UI/lifecycle layer can notify the common component.

This keeps lifecycle knowledge where it belongs.

---

# 20. Handle Cancellation Correctly

When wrapping callback-based APIs, cancellation must be considered.

For example:

```text
Common coroutine
       │
       ▼
Native asynchronous API
       │
       ▼
Coroutine cancelled
```

The adapter should ensure that:

- The callback is no longer processed unnecessarily.
- Native resources are released where appropriate.
- The operation does not leak.
- The common caller sees a correct cancellation result.

This becomes especially important for:

- Location
- Bluetooth
- Camera
- Sensors
- Long-running operations

---

# 21. Use `suspend` for One-Shot Async Capabilities

If a platform API performs a one-time asynchronous operation, a suspending common API is often easier to consume.

For example:

```kotlin
expect suspend fun requestBiometricAuthentication(
    reason: String
): Boolean
```

Common code becomes:

```kotlin
val authenticated =
    requestBiometricAuthentication("Confirm payment")
```

The platform implementation handles callbacks internally.

---

# 22. Use `Flow` for Continuous Platform Events

For streams such as:

```text
Network state
Location updates
Sensor events
Battery state
Lifecycle events
```

a `Flow` can be an appropriate common abstraction.

Example:

```kotlin
interface NetworkMonitor {
    val status: Flow<NetworkStatus>
}
```

The platform implementation adapts its native event mechanism.

The common layer receives a consistent stream.

---

# 23. Avoid Global Platform State

Be cautious with:

```kotlin
object PlatformManager
```

holding mutable global state.

Global state can make:

- Tests harder
- Lifecycle management harder
- Memory ownership unclear
- Dependency replacement difficult

Prefer explicitly managed dependencies where practical.

---

# 24. Keep `actual` Implementations Focused

An `actual` implementation should primarily answer:

> How does this platform provide this capability?

Avoid putting business rules into:

```kotlin
actual fun ...
```

For example, this is questionable:

```kotlin
actual fun calculateDiscount(): Double {
    // platform implementation
    // business rules
}
```

The discount calculation should normally be common.

The platform implementation should deal with genuine platform differences.

---

# 25. Keep Platform Implementations Thin

A good platform implementation often looks like:

```text
actual
  ↓
small adapter
  ↓
native API
```

Rather than:

```text
actual
  ↓
business logic
  ↓
repository
  ↓
domain rules
  ↓
native API
```

The latter makes the architecture harder to reason about.

---

# 26. Use Intermediate Source Sets When Appropriate

If several targets can share an implementation, avoid unnecessary duplication.

For example:

```text
commonMain
     │
  appleMain
     │
 ┌───┼────┐
 ▼   ▼    ▼
iOS macOS tvOS
```

If a platform API is available across those targets and the implementation is genuinely compatible, an intermediate source set can host it.

This is better than copying the same code into every target.

---

# 27. Do Not Assume Platform Compatibility

An API being available on one Apple target does not automatically mean it should be placed in a shared Apple source set.

Before moving code into an intermediate source set, verify:

- API availability
- Platform behavior
- Lifecycle differences
- Version constraints
- Build configuration
- Actual implementation compatibility

Share only what is truly shared.

---

# 28. Keep Dependencies in the Correct Source Set

If a dependency is required only by Android implementation, keep it in the Android-specific source set.

Likewise for iOS.

Conceptually:

```text
commonMain
    common dependencies

androidMain
    Android dependencies

iosMain
    iOS dependencies
```

This prevents platform-specific libraries from accidentally becoming common architectural dependencies.

---

# 29. Do Not Leak Platform Dependencies Through Common APIs

A common API should not require callers to understand a platform dependency.

For example, avoid:

```kotlin
expect fun createAndroidThing(
    androidConfiguration: SomeAndroidType
)
```

Prefer a platform-neutral configuration:

```kotlin
data class FeatureConfig(
    val enabled: Boolean
)
```

Then the platform implementation can translate it internally.

---

# 30. Document Why `expect` Exists

A future developer may see:

```kotlin
expect fun appDataDirectory(): String
```

and wonder why it exists.

A short comment can explain the architectural boundary:

```kotlin
/**
 * Returns the application-owned data directory for the current platform.
 *
 * The common storage layer requires a writable application location,
 * while the actual location is platform-specific.
 */
expect fun appDataDirectory(): String
```

Document the **reason for the abstraction**, not obvious syntax.

---

# 31. Prefer Meaningful Names

Good names communicate capability.

### Good

```text
SecureStorage
Clipboard
LocationProvider
NetworkMonitor
DeviceInfo
UrlOpener
BiometricAuthenticator
```

### Weak

```text
PlatformHelper
NativeUtils
CommonManager
PlatformBridge
PlatformWrapper
```

A name should tell the reader what the component does.

---

# 32. Keep APIs Stable

Platform implementations may change frequently.

The common capability should change less often.

For example:

```kotlin
interface SecureStorage {
    fun save(key: String, value: String)
    fun read(key: String): String?
}
```

The underlying implementation can evolve without changing consumers.

This creates a stable architectural contract.

---

# 33. Design for Evolution

Native APIs change.

Operating systems change.

Libraries change.

A good capability boundary protects the common layer from unnecessary changes.

```text
Native API changes
       ↓
Platform adapter changes
       ↓
Common business logic
       │
       └── remains stable
```

This is one of the most practical benefits of a clean platform boundary.

---

# 34. Avoid Premature Abstraction

Do not create ten platform interfaces before the feature exists.

Instead:

```text
Need appears
    ↓
Understand requirement
    ↓
Define smallest useful capability
    ↓
Implement platform behavior
```

Architecture should solve a real problem.

It should not create complexity in anticipation of every possible future requirement.

---

# 35. Prefer One Responsibility per Platform Capability

For example:

```text
SecureStorage
```

should not also:

```text
Open URLs
Copy clipboard
Read location
Request permissions
```

If unrelated responsibilities accumulate, split the capability.

Small interfaces are generally easier to:

- Test
- Replace
- Document
- Review
- Reuse

---

# 36. Be Careful With `expect class`

An `expect class` can be useful:

```kotlin
expect class DeviceInfo {
    val model: String
}
```

But it should represent a meaningful platform abstraction.

If the class only contains one operation, a function may be simpler:

```kotlin
expect fun deviceModel(): String
```

There is no requirement that every platform capability must be represented as a class.

Choose the simplest API that fits the lifecycle and dependency model.

---

# 37. Prefer Interfaces for Stateful Services

If a component has behavior and state and is injected into multiple consumers, an interface may be more flexible:

```kotlin
interface NetworkMonitor {
    val status: Flow<NetworkStatus>
}
```

Then:

```text
AndroidNetworkMonitor
IosNetworkMonitor
FakeNetworkMonitor
```

can all implement the same contract.

This can be easier to test than tightly coupling consumers to an expected class.

---

# 38. Combine `expect` / `actual` With Interfaces Carefully

A useful pattern is:

```kotlin
interface DeviceInfo {
    val model: String
}
```

and:

```kotlin
expect fun createDeviceInfo(): DeviceInfo
```

This separates:

```text
Platform selection
```

from:

```text
Consumer dependency
```

The consumer only depends on:

```kotlin
DeviceInfo
```

The platform decides which implementation to create.

---

# 39. Test Common Logic With Fakes

Suppose:

```kotlin
interface SecureStorage {
    fun save(key: String, value: String)
    fun read(key: String): String?
}
```

A common fake can be:

```kotlin
class FakeSecureStorage : SecureStorage {

    private val values = mutableMapOf<String, String>()

    override fun save(key: String, value: String) {
        values[key] = value
    }

    override fun read(key: String): String? {
        return values[key]
    }
}
```

Now:

```text
Business logic
      ↓
Fake capability
      ↓
commonTest
```

The test does not need Android or iOS.

---

# 40. Test Platform Adapters Separately

Common tests should not try to prove every native API detail.

Instead:

```text
commonTest
    ↓
Does business logic behave correctly?

Android tests
    ↓
Does Android adapter integrate correctly?

iOS tests
    ↓
Does iOS adapter integrate correctly?
```

This keeps tests focused.

---

# 41. Avoid Testing Implementation Details

Do not make tests depend on:

```text
Android class names
iOS framework objects
Internal adapter structure
```

unless the test is specifically a platform integration test.

Test the capability contract.

For example:

```text
save("token", "123")
read("token")
→ "123"
```

The test does not need to know which native storage mechanism was used.

---

# 42. Security Must Remain Platform-Aware

Security is one area where abstraction should not become ignorance.

For:

```text
Passwords
Tokens
Keys
Biometric authentication
Encryption
Secure storage
```

the platform implementation must respect the platform's current security model.

Do not invent custom security mechanisms simply because the common abstraction is easier.

A common API can be platform-neutral while the security implementation remains platform-specific.

---

# 43. Do Not Hide Important Platform Constraints

Abstraction should simplify usage.

It should not hide behavior that materially affects correctness.

For example, if a location provider:

```text
requires permission
may return no location
may be unavailable
```

the common API should represent those realities.

A misleading API like:

```kotlin
expect fun currentLocation(): Location
```

may be worse than:

```kotlin
expect suspend fun currentLocation(): Location?
```

or a richer result model.

Good abstraction does not mean pretending failure cannot happen.

---

# 44. Model Platform Differences Explicitly When Necessary

Sometimes the platforms genuinely have different capabilities.

For example:

```kotlin
sealed interface BiometricAvailability {
    data object Available : BiometricAvailability
    data object Unavailable : BiometricAvailability
}
```

This is better than forcing every platform into a false assumption.

The common model can represent meaningful differences.

---

# 45. Do Not Force Symmetry

Android and iOS do not need identical implementations.

For example:

```text
Android implementation: 80 lines
iOS implementation: 45 lines
```

That is perfectly acceptable.

The goal is not:

```text
Same number of lines
```

The goal is:

```text
Same capability contract
```

Platform implementations are allowed to differ.

---

# 46. Keep Platform-Specific UI Where It Belongs

If a platform capability is strongly connected to native UI, it may be better to keep the UI interaction in the platform presentation layer.

For example:

```text
Camera picker
Share sheet
Permission dialog
Biometric prompt
```

may have platform-specific UI behavior.

Do not force every UI detail through common `expect` / `actual` APIs.

Share the business decision, not necessarily every UI interaction.

---

# 47. Separate Business Intent From UI Execution

For example:

```text
Business:
"User should authenticate before payment."

Platform:
"Show the appropriate biometric authentication UI."
```

The common layer can request:

```kotlin
authenticate(reason)
```

while the platform implementation controls the actual native prompt.

This separation is clean and practical.

---

# 48. Avoid Platform Conditionals in Common Code

Avoid code such as:

```kotlin
if (platform == "Android") {
    ...
} else if (platform == "iOS") {
    ...
}
```

when the behavior itself is platform-specific.

Instead:

```text
common capability
        ↓
platform implementation
```

This keeps platform knowledge out of common business code.

---

# 49. Avoid Platform Detection as an Architectural Shortcut

This:

```kotlin
when (platform) {
    "Android" -> ...
    "iOS" -> ...
}
```

may look simple initially.

But as targets grow:

```text
Android
iOS
Desktop
Web
Wasm
macOS
tvOS
```

the conditional logic becomes increasingly difficult to maintain.

Prefer polymorphism or platform-specific source sets.

---

# 50. Use Source Sets Instead of Runtime Branching

If behavior is known at compile time, source-set separation is usually clearer than runtime platform checks.

Instead of:

```kotlin
if (isAndroid()) {
    androidBehavior()
} else {
    iosBehavior()
}
```

use:

```text
commonMain
androidMain
iosMain
```

with:

```kotlin
expect / actual
```

or platform-specific implementations.

This makes the architecture explicit.

---

# 51. Keep Platform APIs Out of Domain Models

Domain models should remain portable.

Avoid:

```kotlin
data class Payment(
    val androidUri: Uri
)
```

Prefer:

```kotlin
data class Payment(
    val receiptUrl: String
)
```

The platform can convert the common representation into the native type when required.

---

# 52. Use Serialization-Friendly Models

When common models cross:

```text
Network
Database
Cache
UI state
```

keep them platform-neutral and predictable.

For example:

```kotlin
@Serializable
data class Product(
    val id: String,
    val name: String
)
```

Do not embed native platform objects in serializable common models.

---

# 53. Keep Platform Dependencies Visible in Build Logic

A reviewer should be able to understand:

```text
common dependencies
Android dependencies
iOS dependencies
```

from the Gradle configuration.

Avoid accidentally making a platform-only dependency part of common architecture.

A clean dependency graph helps prevent accidental coupling.

---

# 54. Review `expect` / `actual` Like an API Contract

During code review, ask:

### Contract

Does the expected declaration clearly express the capability?

### Portability

Could another supported platform implement it?

### Coupling

Does it leak native types?

### Scope

Is it smaller than necessary?

### Behavior

Does it define meaningful failure states?

### Testing

Can common consumers be tested independently?

### Ownership

Who owns the native resource?

These questions are more valuable than simply checking whether the code compiles.

---

# 55. A Practical Review Example

Suppose a pull request introduces:

```kotlin
expect class PlatformManager {
    fun getContext(): Any
    fun getLocation(): Any
    fun openUrl(url: String)
    fun share(text: String)
}
```

A strong review would ask:

```text
Why does common code need Context?

Why is Location represented as Any?

Why are unrelated capabilities grouped together?

Can the API expose a common Location model?

Can sharing and URL opening be separate capabilities?
```

A possible redesign:

```kotlin
interface LocationProvider {
    suspend fun currentLocation(): Location?
}

expect fun createLocationProvider(): LocationProvider

expect fun openUrl(url: String)

expect fun shareText(text: String)
```

The second design communicates the architecture much more clearly.

---

# 56. The Smallest Useful Abstraction

A powerful design rule is:

> **Create the smallest abstraction that allows common code to express its requirement without knowing the platform implementation.**

For example:

```text
Requirement:
"Copy this text."

Abstraction:
copyToClipboard(text)
```

Not:

```text
PlatformClipboardManager
NativeClipboardController
PlatformContextProvider
```

The smaller API is often the stronger API.

---

# 57. A Layered Mental Model

Think about platform integration in four layers:

```text
┌─────────────────────────────┐
│        Business Intent      │
│   "Authenticate the user"   │
└──────────────┬──────────────┘
               │
┌──────────────▼──────────────┐
│       Common Capability     │
│     BiometricAuthenticator  │
└──────────────┬──────────────┘
               │
┌──────────────▼──────────────┐
│     Platform Adapter        │
│ Android / iOS implementation│
└──────────────┬──────────────┘
               │
┌──────────────▼──────────────┐
│        Native API            │
└─────────────────────────────┘
```

Each layer has one responsibility.

---

# 58. Best-Practice Architecture

A mature KMP feature may look like:

```text
commonMain
│
├── domain
│   ├── models
│   └── usecases
│
├── data
│   ├── repository
│   └── storage
│
├── platform
│   ├── SecureStorage.kt
│   ├── NetworkMonitor.kt
│   └── DeviceInfo.kt
│
└── expect declarations
        │
        ├───────────────┐
        ▼               ▼
   androidMain       iosMain
        │               │
   actual adapters   actual adapters
        │               │
   Android APIs      Apple APIs
```

The important property is not the folder names.

It is the dependency direction.

---

# 59. Anti-Pattern: Platform Logic in Common Business Code

### Problem

```kotlin
class LoginUseCase {
    fun login() {
        if (isAndroid()) {
            // Android behavior
        } else {
            // iOS behavior
        }
    }
}
```

### Better

```kotlin
class LoginUseCase(
    private val secureStorage: SecureStorage
) {
    fun login() {
        // common login rules
    }
}
```

The platform difference belongs behind the dependency.

---

# 60. Anti-Pattern: Common Code Returning Native Objects

### Problem

```kotlin
expect fun createNativeObject(): Any
```

The caller still needs to understand what the object is.

### Better

```kotlin
expect fun createSecureStorage(): SecureStorage
```

Now the caller understands the capability without understanding the implementation.

---

# 61. Anti-Pattern: Duplicate Platform Business Logic

### Problem

```text
androidMain
    TokenRefreshLogic.kt

iosMain
    TokenRefreshLogic.kt
```

when the logic is identical.

### Better

```text
commonMain
    TokenRefreshLogic.kt
        │
        ▼
    SecureStorage
        │
   ┌────┴────┐
   ▼         ▼
Android     iOS
```

Only the platform capability is different.

---

# 62. Anti-Pattern: Giant `expect` API

### Problem

```kotlin
expect class PlatformServices {
    // 30+ methods
}
```

### Better

Break it into meaningful capabilities:

```text
SecureStorage
Clipboard
LocationProvider
NetworkMonitor
DeviceInfo
UrlOpener
```

This makes dependency usage explicit.

---

# 63. Anti-Pattern: Abstraction That Lies

Suppose Android supports a capability but iOS does not.

Do not expose:

```kotlin
expect fun feature(): Boolean
```

and make the iOS implementation silently return:

```kotlin
true
```

or:

```kotlin
false
```

without communicating what that means.

Represent actual capability and availability honestly.

For example:

```kotlin
sealed interface FeatureAvailability {
    data object Available : FeatureAvailability
    data object Unsupported : FeatureAvailability
}
```

---

# 64. Anti-Pattern: Ignoring Native Security

Never weaken platform security just to create identical behavior.

For example:

```text
Android secure storage
        ↓
Plain text common storage
```

is not a valid portability strategy.

Instead:

```text
Common SecureStorage contract
        ↓
Android secure implementation
        ↓
iOS secure implementation
```

The security boundary remains platform-aware.

---

# 65. Anti-Pattern: Hiding Lifecycle Requirements

A method such as:

```kotlin
expect fun startCamera()
```

may look simple but can hide:

- Permission requirements
- Lifecycle requirements
- Resource ownership
- UI requirements
- Cancellation
- Orientation
- Background behavior

A good abstraction should make important behavior explicit.

Sometimes a higher-level feature interface is better than exposing a low-level native operation.

---

# 66. When to Prefer an Interface Over `expect` / `actual`

Use an interface when:

- Dependency injection is central.
- Multiple implementations may exist.
- Testing requires easy fakes.
- The capability is conceptually independent of platform selection.
- Construction can be handled by the composition root.

Example:

```kotlin
interface Analytics {
    fun track(event: String)
}
```

The platform or application layer can provide the implementation.

---

# 67. When `expect` / `actual` Is a Better Fit

It can be a good fit when:

- The capability must exist per target.
- The implementation is inherently platform-specific.
- The common API should be very small.
- The compiler should ensure every target provides an implementation.
- There is a natural compile-time platform boundary.

Example:

```kotlin
expect fun appDataDirectory(): String
```

---

# 68. They Can Be Used Together

There is no requirement to choose only one.

For example:

```kotlin
interface DeviceInfo {
    val model: String
}

expect fun createDeviceInfo(): DeviceInfo
```

This provides:

```text
expect / actual
        ↓
platform-specific construction

interface
        ↓
stable consumer contract
```

This combination can be powerful when used intentionally.

---

# 69. Best-Practice Checklist

Before merging an `expect` / `actual` change, ask:

- [ ] Is there a real platform difference?
- [ ] Does common code need this capability?
- [ ] Is the API capability-oriented?
- [ ] Is the API as small as possible?
- [ ] Are platform types hidden?
- [ ] Is business logic in `commonMain`?
- [ ] Are platform implementations thin?
- [ ] Are native dependencies isolated?
- [ ] Are lifecycle rules respected?
- [ ] Is cancellation handled?
- [ ] Are errors represented meaningfully?
- [ ] Is resource ownership clear?
- [ ] Can common code be tested with a fake?
- [ ] Are platform adapters tested separately?
- [ ] Is dependency injection used where appropriate?
- [ ] Is an existing multiplatform library worth considering?
- [ ] Could an interface be simpler?
- [ ] Could `expect` / `actual` be avoided entirely?
- [ ] Could an intermediate source set reduce duplication?
- [ ] Does the abstraction support future targets?
- [ ] Does the API describe intent rather than implementation?
- [ ] Does the design avoid runtime platform checks?
- [ ] Are security-sensitive operations delegated to native security facilities?

---

# 70. Final Architecture Rule

The most useful rule to carry forward is:

```text
                 COMMON
                   │
                   │
            What do we need?
                   │
                   ▼
              Capability
                   │
                   │
              expect
                   │
        ┌──────────┴──────────┐
        ▼                     ▼
     Android                  iOS
      actual                 actual
        │                     │
        ▼                     ▼
   Native APIs           Native APIs
```

Do not ask:

> "How can I make Android and iOS look identical?"

Ask:

> "How can I share the application intent while allowing each platform to implement that intent correctly?"

That mindset produces better KMP architecture.

---

# Chapter Takeaways

> [!IMPORTANT]
> **The quality of `expect` / `actual` code depends more on the boundary you design than on the amount of platform code you write.**

1. Start with the capability, not the platform.
2. Keep `expect` declarations small.
3. Use interfaces when they improve dependency management and testing.
4. Use `expect` / `actual` only for genuine platform differences.
5. Keep business logic in `commonMain`.
6. Never leak unnecessary platform types into common code.
7. Prefer intent-based APIs.
8. Keep native implementation details at the platform edge.
9. Translate native exceptions into meaningful common errors when appropriate.
10. Do not mirror Android or iOS APIs unnecessarily.
11. Avoid giant platform managers.
12. Use dependency injection for platform implementations where appropriate.
13. Keep constructors platform-neutral.
14. Assemble dependencies in platform-specific composition roots.
15. Convert native objects into common models when the business layer needs them.
16. Do not force complex native APIs into artificial common abstractions.
17. Let platform code handle platform complexity.
18. Respect lifecycle differences.
19. Handle coroutine cancellation correctly.
20. Use `suspend` for appropriate one-shot asynchronous operations.
21. Use `Flow` for appropriate continuous platform events.
22. Avoid unnecessary global platform state.
23. Keep `actual` implementations focused.
24. Use intermediate source sets when implementations are genuinely shared.
25. Verify compatibility before sharing code through intermediate source sets.
26. Keep platform-only dependencies in platform source sets.
27. Document why important expected declarations exist.
28. Use meaningful capability names.
29. Keep common contracts stable.
30. Design platform boundaries for future evolution.
31. Avoid premature abstraction.
32. Prefer one responsibility per capability.
33. Use `expect class` only when a class is actually useful.
34. Prefer functions for simple stateless capabilities.
35. Prefer interfaces for injectable stateful services when appropriate.
36. Combine interfaces with `expect` / `actual` when it improves architecture.
37. Test common behavior with fakes.
38. Test native adapters separately.
39. Keep security platform-aware.
40. Do not hide important platform constraints.
41. Model genuine platform differences explicitly.
42. Do not force Android and iOS implementations to be symmetrical.
43. Keep native UI interactions close to platform UI.
44. Separate business intent from UI execution.
45. Avoid platform conditionals in common code.
46. Prefer source-set separation over runtime platform branching.
47. Keep platform objects out of domain models.
48. Keep common models serialization-friendly.
49. Keep platform dependencies visible in build configuration.
50. Review `expect` / `actual` as an API contract.
51. Choose the smallest useful abstraction.
52. Keep the dependency direction predictable.
53. Do not share implementation details merely for the sake of sharing code.
54. Do not weaken platform security for portability.
55. Do not hide lifecycle or resource ownership problems behind abstractions.
56. Use interfaces and `expect` / `actual` together when both provide value.
57. Avoid creating abstractions that only forward native calls.
58. Prefer capabilities that describe application intent.
59. Let each platform use its native strengths.
60. **Share the intent. Respect the platform. Keep the boundary clean.**

---

## Final Thought

`expect` / `actual` is not simply a syntax feature for writing two implementations.

It is an architectural boundary.

The strongest KMP projects do not try to erase every platform difference.

They identify those differences, isolate them, and keep them from contaminating the shared business architecture.

```text
        Shared Intent
             │
             ▼
       Common Contract
             │
        expect / actual
             │
      ┌──────┴──────┐
      ▼             ▼
   Android          iOS
   Native           Native
   behavior         behavior
```

The result is not:

> **"One codebase that pretends every platform is identical."**

It is:

> **"One shared architecture that knows exactly where platforms are different."**

That is the real power of `expect` / `actual` in Kotlin Multiplatform.
