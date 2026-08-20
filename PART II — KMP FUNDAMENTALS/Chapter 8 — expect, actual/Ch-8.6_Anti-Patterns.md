# Chapter 8 — `expect` / `actual`

## Part 6 — Anti-Patterns

> **`expect` / `actual` is powerful when it creates a clean platform boundary. It becomes expensive when it is used to hide poor architecture, duplicate business logic, or expose native implementation details.**

The biggest problems in KMP are rarely caused by the syntax itself.

They usually appear when the platform boundary is designed incorrectly.

This part explores the most common `expect` / `actual` anti-patterns, why they happen, and how to replace them with cleaner designs.

---

## 1. Anti-Pattern: Using `expect` / `actual for Everything`

The first mistake is treating `expect` / `actual` as the default mechanism for every platform difference.

For example:

```kotlin
expect fun calculateTotal(
    price: Double,
    tax: Double
): Double
```

If the calculation is identical on Android and iOS, there is no reason for it to be platform-specific.

Prefer:

```kotlin
fun calculateTotal(
    price: Double,
    tax: Double
): Double {
    return price + tax
}
```

### Why this is a problem

Unnecessary `expect` / `actual` creates:

- Duplicate implementations
- More files
- More testing effort
- More maintenance
- More opportunities for behavior drift

### Better rule

> **Use `expect` / `actual` only when the implementation genuinely depends on the platform.**

---

# 2. Anti-Pattern: Duplicating Business Logic

One of the worst mistakes is putting business rules inside `actual` implementations.

For example:

```kotlin
// commonMain
expect fun calculateDiscount(
    customerType: String,
    amount: Double
): Double
```

Then:

```kotlin
// androidMain
actual fun calculateDiscount(
    customerType: String,
    amount: Double
): Double {
    // Android discount rules
}
```

And:

```kotlin
// iosMain
actual fun calculateDiscount(
    customerType: String,
    amount: Double
): Double {
    // iOS discount rules
}
```

Now the same business rule exists twice.

Eventually:

```text
Android → 10%
iOS     → 15%
```

The applications no longer behave consistently.

### Better

Keep the business rule in:

```text
commonMain
```

and use `expect` / `actual` only for the capability the rule needs.

---

# 3. Anti-Pattern: Giant `PlatformManager`

A common design starts small:

```kotlin
expect class PlatformManager {
    fun openUrl(url: String)
}
```

Then more requirements arrive:

```kotlin
expect class PlatformManager {
    fun openUrl(url: String)
    fun share(text: String)
    fun copy(text: String)
    fun getLocation(): Location?
    fun getDeviceModel(): String
    fun getBatteryLevel(): Int
    fun vibrate()
    fun requestPermission(): Boolean
    fun authenticate(): Boolean
}
```

Eventually it becomes a platform-specific God object.

### Problems

- Too many responsibilities
- Difficult testing
- Large dependency surface
- Harder code reviews
- Strong coupling
- Poor discoverability

### Better

Split capabilities:

```text
UrlOpener
Clipboard
ShareService
LocationProvider
DeviceInfo
BatteryInfo
HapticFeedback
PermissionManager
BiometricAuthenticator
```

Each component should have a clear responsibility.

---

# 4. Anti-Pattern: Returning `Any`

This is a major warning sign:

```kotlin
expect fun getNativeObject(): Any
```

The common layer now has no meaningful contract.

The caller must know:

```text
What is this object?
Which platform is it from?
How should it be used?
What type should I cast it to?
```

This defeats the purpose of a common abstraction.

### Better

Expose a capability:

```kotlin
expect fun createSecureStorage(): SecureStorage
```

Now the consumer knows what it can do without knowing the native type.

---

# 5. Anti-Pattern: Leaking Android Types Into `commonMain`

Avoid:

```kotlin
class UserRepository(
    private val context: Context
)
```

The repository is now coupled to Android.

This prevents the repository from being genuinely common.

### Better

```kotlin
class UserRepository(
    private val secureStorage: SecureStorage
)
```

The repository depends on behavior rather than Android infrastructure.

---

# 6. Anti-Pattern: Leaking iOS Types Into Common Code

The same problem exists on iOS.

Avoid common APIs containing types such as:

```text
UIViewController
UIApplication
NSObject
NSURL
CLLocation
```

unless the abstraction is deliberately designed around a platform boundary.

Prefer common models:

```kotlin
data class Location(
    val latitude: Double,
    val longitude: Double
)
```

and keep native conversion inside `iosMain`.

---

# 7. Anti-Pattern: Wrapping Every Native API

A common mistake is creating a one-to-one wrapper around an entire native framework.

For example:

```kotlin
expect class NativeCamera {
    fun startPreview()
    fun stopPreview()
    fun setFlash()
    fun setZoom()
    fun setFocus()
    fun setExposure()
    fun switchCamera()
    fun capture()
}
```

If common code only needs:

```kotlin
suspend fun capturePhoto(): ByteArray
```

then the larger abstraction is unnecessary.

### Better

Expose the smallest application-level capability.

---

# 8. Anti-Pattern: `PlatformHelper`

Names such as:

```text
PlatformHelper
PlatformUtils
NativeUtils
PlatformManager
CommonUtils
NativeBridge
```

often indicate an unclear boundary.

A name like:

```kotlin
interface SecureStorage
```

immediately tells the reader what the component does.

### Rule

> **If you cannot give the abstraction a meaningful responsibility-based name, the abstraction may be too broad.**

---

# 9. Anti-Pattern: Runtime Platform Checks

Avoid:

```kotlin
if (platform == "Android") {
    ...
} else if (platform == "iOS") {
    ...
}
```

inside common business code.

This creates runtime platform branching where compile-time source-set separation could be used.

### Better

```text
commonMain
    common behavior

androidMain
    Android implementation

iosMain
    iOS implementation
```

Use `expect` / `actual` or platform-specific implementations.

---

# 10. Anti-Pattern: Platform Detection APIs

This is another warning sign:

```kotlin
expect fun platformName(): String
```

followed by:

```kotlin
when (platformName()) {
    "Android" -> ...
    "iOS" -> ...
}
```

The common code is now explicitly aware of the platforms.

That defeats one of the major benefits of abstraction.

### Better

Ask for the capability:

```kotlin
expect fun openApplicationSettings()
```

instead of:

```kotlin
expect fun platformName(): String
```

---

# 11. Anti-Pattern: Using `expect` for Business Configuration

Consider:

```kotlin
expect val maximumRetryCount: Int
```

If retry policy is a business decision, it should not vary simply because the application runs on Android or iOS.

Prefer:

```kotlin
const val MAX_RETRY_COUNT = 3
```

or a configurable common policy.

Use `expect` only when the value genuinely depends on platform behavior.

---

# 12. Anti-Pattern: Platform-Specific Business Decisions

Avoid:

```kotlin
actual fun shouldShowDiscount(): Boolean {
    return ...
}
```

when the decision belongs to the domain.

Business rules should be expressed in common code.

The platform implementation should provide the environmental information required by that rule.

---

# 13. Anti-Pattern: Hiding Native Lifecycle Requirements

Consider:

```kotlin
expect fun startCamera()
```

This looks simple.

But the actual operation may require:

- Permission
- Activity lifecycle
- UIViewController
- Camera session
- Resource cleanup
- Cancellation
- Foreground state

A simplistic abstraction can hide important lifecycle constraints.

### Better

Design the capability around the actual application requirement and make lifecycle ownership explicit.

---

# 14. Anti-Pattern: Ignoring Resource Ownership

Native resources may have lifecycle requirements.

Examples:

```text
Camera
Bluetooth connection
Location listener
Audio session
Sensor subscription
WebSocket
Native view
```

An `actual` implementation should make ownership clear.

Ask:

```text
Who creates it?
Who owns it?
Who stops it?
Who releases it?
What happens when the coroutine is cancelled?
What happens when the screen disappears?
```

If those questions cannot be answered, the abstraction is probably incomplete.

---

# 15. Anti-Pattern: Ignoring Cancellation

Suppose common code calls:

```kotlin
val location = currentLocation()
```

and the native implementation starts a callback-based request.

If the coroutine is cancelled but the native request continues forever, the adapter may leak work.

A correct asynchronous bridge should consider:

```text
Coroutine cancellation
        ↓
Native operation cancellation
        ↓
Resource cleanup
```

Do not treat cancellation as an optional detail.

---

# 16. Anti-Pattern: Swallowing Native Errors

Avoid:

```kotlin
actual suspend fun authenticate(): Boolean {
    return try {
        nativeAuthentication()
    } catch (e: Exception) {
        false
    }
}
```

This may hide important differences between:

```text
User cancelled
Not available
Permission denied
Authentication failed
System error
```

### Better

Represent meaningful outcomes:

```kotlin
sealed interface AuthenticationResult {
    data object Success : AuthenticationResult
    data object Cancelled : AuthenticationResult
    data object Unavailable : AuthenticationResult
    data object Failed : AuthenticationResult
}
```

The common layer can then make an informed decision.

---

# 17. Anti-Pattern: Leaking Native Exceptions

Avoid exposing platform-specific exceptions through common APIs.

For example:

```kotlin
expect suspend fun authenticate(): Result<Unit, LAError>
```

The common layer should not need to know Apple's native error type.

Translate it at the platform boundary.

```text
Native error
    ↓
Platform adapter
    ↓
Common error model
    ↓
Business logic
```

---

# 18. Anti-Pattern: Fake Symmetry

Android and iOS do not need identical implementations.

Do not force both platforms into artificial behavior simply to make the source code look symmetrical.

For example:

```text
Android implementation → 100 lines
iOS implementation     → 40 lines
```

There is nothing wrong with that.

The important requirement is:

```text
Same capability contract
```

not:

```text
Same implementation
```

---

# 19. Anti-Pattern: Making Unsupported Platforms Pretend

Suppose a feature does not exist on a particular platform.

Do not implement:

```kotlin
actual fun featureAvailable(): Boolean = true
```

just to satisfy the compiler.

Do not silently do nothing:

```kotlin
actual fun performFeature() {
    // nothing
}
```

unless the API explicitly defines no-op behavior.

### Better

Represent support honestly:

```kotlin
sealed interface FeatureAvailability {
    data object Available : FeatureAvailability
    data object Unsupported : FeatureAvailability
}
```

---

# 20. Anti-Pattern: Forcing Unsupported APIs Into Common Code

Not every native capability needs to be shared.

If a feature is:

```text
Android-only
```

and common code does not require it, keep it in:

```text
androidMain
```

Do not create:

```kotlin
expect fun androidOnlyFeature()
```

just for architectural symmetry.

---

# 21. Anti-Pattern: Premature Abstraction

A developer may create:

```text
SecureStorage
LocationProvider
CameraManager
NotificationManager
DeviceManager
AnalyticsManager
PermissionManager
```

before any real requirement exists.

This can create a large abstraction layer with little value.

### Better process

```text
Real requirement
      ↓
Understand platform difference
      ↓
Define smallest useful capability
      ↓
Implement it
```

Architecture should respond to real requirements.

---

# 22. Anti-Pattern: Over-Generalizing the API

Avoid an abstraction such as:

```kotlin
expect fun executePlatformOperation(
    operation: String,
    parameters: Map<String, Any>
): Any?
```

This looks flexible but creates an untyped protocol inside the application.

Problems include:

- No compiler safety
- Poor discoverability
- Runtime errors
- Hard testing
- Weak documentation

Prefer strongly typed APIs.

---

# 23. Anti-Pattern: String-Based Platform Commands

Avoid:

```kotlin
execute("OPEN_URL", url)
execute("COPY_TEXT", text)
execute("SHARE_TEXT", text)
```

This is essentially building a mini runtime protocol.

Prefer:

```kotlin
openUrl(url)
copyToClipboard(text)
shareText(text)
```

The compiler can now help enforce correctness.

---

# 24. Anti-Pattern: Using `Map<String, Any>`

This:

```kotlin
fun platformAction(
    parameters: Map<String, Any>
)
```

is usually a sign that the abstraction is too generic.

Instead of:

```kotlin
parameters["url"] as String
```

use:

```kotlin
openUrl(url)
```

Strong types make the architecture easier to understand.

---

# 25. Anti-Pattern: Common Code Knows the Native Implementation

Avoid comments or APIs like:

```kotlin
// Android uses Context
// iOS uses UIApplication
```

inside domain classes.

The domain should not need to understand how the platform provides the capability.

Keep that knowledge in the adapter layer.

---

# 26. Anti-Pattern: Platform Dependencies in Domain Modules

If the domain module directly depends on:

```text
Android SDK
UIKit
CoreLocation
AVFoundation
```

the domain is no longer truly platform-neutral.

Keep platform dependencies at the appropriate edge.

---

# 27. Anti-Pattern: Native Types in Domain Models

Avoid:

```kotlin
data class UserProfile(
    val avatar: UIImage
)
```

or:

```kotlin
data class UserProfile(
    val uri: Uri
)
```

Prefer:

```kotlin
data class UserProfile(
    val avatarUrl: String
)
```

Then convert to native UI types where needed.

---

# 28. Anti-Pattern: Platform Logic Inside View Models

A ViewModel should not become:

```kotlin
class PaymentViewModel {
    fun pay() {
        if (isAndroid()) {
            // Android logic
        } else {
            // iOS logic
        }
    }
}
```

The ViewModel should depend on common capabilities:

```kotlin
class PaymentViewModel(
    private val paymentAuthenticator: PaymentAuthenticator
)
```

The platform implementation stays behind the dependency.

---

# 29. Anti-Pattern: `expect` / `actual` in Every Layer

A project may eventually look like:

```text
common
    expect repository

android
    actual repository

ios
    actual repository
```

Then:

```text
common
    expect service

android
    actual service

ios
    actual service
```

and so on.

This can indicate that too much architecture has been pushed into platform code.

If the repository and service logic are identical, keep them common and abstract only their platform dependencies.

---

# 30. Anti-Pattern: Duplicating Repositories

Avoid:

```text
androidMain
    UserRepository.kt

iosMain
    UserRepository.kt
```

when the repository logic is the same.

Prefer:

```text
commonMain
    UserRepository.kt
        ↓
    SecureStorage
    ApiClient
    Database
```

Only the platform-specific dependencies should differ.

---

# 31. Anti-Pattern: Duplicating Use Cases

Similarly:

```text
androidMain
    LoginUseCase.kt

iosMain
    LoginUseCase.kt
```

is usually unnecessary.

Business use cases should generally remain in `commonMain`.

If a use case needs a platform capability, inject that capability.

---

# 32. Anti-Pattern: Copy-Paste `actual` Implementations

Suppose:

```text
androidMain
    actual fun formatSomething()

iosMain
    actual fun formatSomething()
```

and both contain identical logic.

That is a strong signal that the function should probably not be `expect`.

Move shared behavior to:

```text
commonMain
```

and isolate only the truly different portion.

---

# 33. Anti-Pattern: Using `expect` to Avoid Dependency Injection

A developer may write:

```kotlin
expect object AppServices {
    val storage: SecureStorage
}
```

This can turn global platform objects into hidden dependencies.

Prefer explicit dependency injection:

```kotlin
class UserRepository(
    private val storage: SecureStorage
)
```

Explicit dependencies are easier to reason about and test.

---

# 34. Anti-Pattern: Global Mutable `actual` Objects

Be cautious with:

```kotlin
actual object PlatformState {
    var currentUser: User? = null
}
```

Global mutable state can cause:

- Test pollution
- Lifecycle issues
- Threading problems
- Unexpected coupling
- Difficult debugging

Prefer scoped state ownership.

---

# 35. Anti-Pattern: Treating Platform APIs as Business APIs

For example:

```kotlin
expect fun showToast(message: String)
```

may be useful for UI infrastructure, but it should not be called directly from a domain use case.

Instead:

```text
Domain
    ↓
UI state / event
    ↓
Platform/UI layer
    ↓
Toast / native notification
```

The domain should express intent, not UI implementation.

---

# 36. Anti-Pattern: `expect` APIs With UI Side Effects in Domain Logic

Avoid:

```kotlin
class CheckoutUseCase {
    fun checkout() {
        // business logic
        showToast("Payment successful")
    }
}
```

The use case should produce a result or event.

The UI layer decides how to present it.

---

# 37. Anti-Pattern: Hiding Permissions

Platform permissions are real application states.

Avoid an API that makes permission handling invisible:

```kotlin
expect suspend fun getLocation(): Location
```

when permission may not exist.

Prefer an explicit result:

```kotlin
sealed interface LocationResult {
    data class Success(val location: Location) : LocationResult
    data object PermissionDenied : LocationResult
    data object Unavailable : LocationResult
}
```

The common layer can then respond appropriately.

---

# 38. Anti-Pattern: Assuming Native APIs Behave the Same

Two platforms may provide a capability but with different semantics.

For example:

```text
Background execution
Notifications
Location updates
Biometric authentication
File access
Bluetooth
```

Do not assume:

```text
Same API concept = Same behavior
```

The common contract should represent the application's actual requirement.

---

# 39. Anti-Pattern: Over-Simplifying Platform Differences

A poor abstraction may say:

```kotlin
expect fun requestPermission(): Boolean
```

But platform permissions may have several meaningful states.

A richer model may be more accurate:

```kotlin
enum class PermissionStatus {
    Granted,
    Denied,
    Restricted,
    NotDetermined
}
```

The right model depends on what the application needs to know.

---

# 40. Anti-Pattern: Hiding Threading Requirements

Native APIs may have specific threading or main-thread requirements.

Do not assume that:

```kotlin
expect suspend fun performNativeOperation()
```

automatically solves threading.

The platform adapter should respect native requirements and document important constraints.

---

# 41. Anti-Pattern: Ignoring Memory Ownership

This is especially important when bridging native APIs.

For example:

```text
Kotlin object
     ↕
Native object
```

Ask:

- Who owns the native object?
- How long should it live?
- Can callbacks retain it?
- Can it retain Kotlin objects?
- What happens when the consumer is destroyed?

Poor ownership decisions can lead to memory leaks and lifecycle bugs.

---

# 42. Anti-Pattern: Creating Cyclic Dependencies

Avoid:

```text
Common service
    ↓
Platform implementation
    ↓
Common service
```

A platform adapter should normally depend on the common contract, not recreate or own the business layer.

Keep dependency direction clear.

---

# 43. Anti-Pattern: Putting Platform Work in `commonMain` Through Reflection

Reflection or dynamic mechanisms should not be used simply to avoid proper platform separation.

For example:

```kotlin
Class.forName(...)
```

to find a platform class creates runtime coupling and weakens compile-time guarantees.

KMP's source sets and compiler are designed to provide a safer boundary.

---

# 44. Anti-Pattern: Treating `actual` as a Back Door

An `actual` implementation should not become a place where architecture rules are bypassed.

Avoid:

```text
common architecture
      ↓
actual implementation
      ↓
direct access to everything
```

The platform layer still needs clear boundaries.

---

# 45. Anti-Pattern: One Abstraction for Unrelated Platforms

Sometimes Android and iOS are not the only targets.

For example:

```text
Android
iOS
Desktop
Web
Wasm
```

An abstraction designed only around Android and iOS may become difficult to extend.

Before designing the contract, ask:

> **What is the stable capability, independent of today's targets?**

---

# 46. Anti-Pattern: Designing Around Current Targets Only

Avoid:

```kotlin
expect fun iosOrAndroidStorage(): Storage
```

The API should not encode today's target list.

Prefer:

```kotlin
expect fun createSecureStorage(): SecureStorage
```

Now adding another target does not require changing the common API.

---

# 47. Anti-Pattern: Ignoring Existing Multiplatform Libraries

Before implementing:

```text
HTTP
Serialization
Database
Logging
Coroutines
Date/time
Settings
```

from scratch, check whether a mature multiplatform library already exists and meets the project's requirements.

Custom `expect` / `actual` code should solve a meaningful problem rather than duplicate existing ecosystem capabilities.

---

# 48. Anti-Pattern: Reimplementing Mature Libraries

For example, creating your own:

```text
HTTP abstraction
JSON serializer
date/time system
logging framework
```

using `expect` / `actual` can create significant maintenance cost.

Evaluate:

- Stability
- Platform coverage
- API quality
- Performance
- Community adoption
- Security
- Maintenance status

before introducing custom infrastructure.

---

# 49. Anti-Pattern: Ignoring Version and Platform Constraints

A native API may require:

```text
Specific Android API level
Specific iOS version
Specific compiler behavior
Specific framework version
```

Do not expose a common capability without considering these constraints.

The common API may need to represent availability explicitly.

---

# 50. Anti-Pattern: No Documentation for Non-Obvious Contracts

A small API can still have important semantics.

For example:

```kotlin
expect suspend fun authenticate(reason: String): Boolean
```

Does `false` mean:

```text
Cancelled?
Denied?
Unavailable?
Failed?
```

Document the contract or use a richer result model.

---

# 51. Anti-Pattern: Ignoring Testability

If the only way to test:

```kotlin
class PaymentService
```

is to run a real Android or iOS environment, the abstraction may be too tightly coupled.

Prefer:

```text
PaymentService
    ↓
PaymentAuthenticator interface
    ↓
FakePaymentAuthenticator
```

Common business logic should be testable without a device whenever practical.

---

# 52. Anti-Pattern: Testing Only the Happy Path

Platform APIs have many failure modes.

For example:

```text
Permission denied
User cancelled
Network unavailable
Native API unavailable
Background restriction
Unsupported device
```

Test the common behavior around those meaningful states.

---

# 53. Anti-Pattern: Treating Compile Success as Architecture Success

This code can compile:

```kotlin
expect fun platform(): Any
```

and:

```kotlin
actual fun platform(): Any = ...
```

But compilation does not prove that the boundary is good.

Review:

```text
Coupling
Testability
Responsibility
Lifecycle
Error handling
Future targets
```

Architecture quality goes beyond compilation.

---

# 54. Anti-Pattern: Too Many Tiny `expect` Functions

The opposite problem also exists.

For example:

```kotlin
expect fun getWidth(): Int
expect fun getHeight(): Int
expect fun getDensity(): Float
expect fun getScale(): Float
expect fun getOrientation(): String
expect fun getInsets(): Insets
```

If these are part of one coherent capability, a focused abstraction may be better:

```kotlin
interface ScreenInfo {
    val width: Int
    val height: Int
    val density: Float
}
```

Do not split APIs mechanically.

Split them according to responsibility.

---

# 55. Anti-Pattern: Too Many Intermediate Source Sets

Intermediate source sets are useful, but creating a complicated hierarchy without real sharing can make builds harder to understand.

For example:

```text
commonMain
    appleMain
        mobileAppleMain
            iosMain
```

should only exist when each layer has a meaningful shared responsibility.

Keep the hierarchy as simple as the project allows.

---

# 56. Anti-Pattern: Sharing Code That Is Not Actually Compatible

Code should not be moved into:

```text
appleMain
```

just because all targets are Apple targets.

Verify that the APIs and semantics truly match.

Shared code should represent genuine compatibility.

---

# 57. Anti-Pattern: Using `expect` / `actual` as a Replacement for Architecture

KMP does not automatically create:

```text
Clean Architecture
MVVM
MVI
Domain-driven design
Dependency injection
```

The compiler can guarantee platform implementations exist.

It cannot guarantee that your architecture is good.

`expect` / `actual` is a tool inside the architecture.

It is not the architecture itself.

---

# 58. Anti-Pattern: Hiding Too Much Behind the Abstraction

An abstraction should simplify complexity.

It should not make important behavior impossible to understand.

For example:

```kotlin
expect fun doEverything()
```

is technically simple but architecturally useless.

Good abstraction hides implementation details while preserving meaningful behavior.

---

# 59. Anti-Pattern: Exposing Too Much Through the Abstraction

The opposite is:

```kotlin
expect class PlatformController {
    // 50 methods
}
```

Now common code knows too much.

A good boundary balances:

```text
Enough abstraction to hide implementation
+
Enough information to express behavior correctly
```

---

# 60. Anti-Pattern: Copying Android Architecture Onto iOS

KMP is not:

```text
Android architecture
        +
iOS implementation
```

It is a multiplatform architecture.

Do not force Android-specific concepts into the common layer simply because the project started as an Android application.

Examples to question:

```text
Context everywhere
Activity-based lifecycle assumptions
Android-specific navigation concepts
Android-specific UI types
```

Design common APIs around application capabilities.

---

# 61. Anti-Pattern: Assuming iOS Is Just Another Android Target

The reverse is also problematic.

iOS has its own:

```text
Lifecycle
Memory model
UI framework
Permission behavior
Background execution model
Native concurrency considerations
```

Respect those differences.

KMP allows shared business logic without requiring identical platform architecture.

---

# 62. Anti-Pattern: Ignoring Platform Ownership

A common abstraction may say:

```kotlin
expect fun createNativeView(): Any
```

but who owns that view?

```text
Common layer?
Android Activity?
Compose UI?
SwiftUI?
UIViewController?
```

If ownership is unclear, lifecycle problems are likely.

Keep UI ownership in the UI/platform layer unless there is a strong reason otherwise.

---

# 63. Anti-Pattern: Mixing UI and Infrastructure

Avoid:

```kotlin
expect fun showBiometricPromptAndSaveToken(token: String): Boolean
```

This combines:

```text
UI
Authentication
Storage
Business operation
```

into one API.

Prefer separate capabilities:

```text
BiometricAuthenticator
SecureStorage
```

and let the common use case coordinate them.

---

# 64. Anti-Pattern: Platform Adapter Becoming a Service Layer

A platform adapter should not gradually become:

```text
Repository
Use case
Business service
Analytics service
Navigation service
UI coordinator
```

Keep platform adapters focused on translating between:

```text
Common capability
        ↕
Native API
```

---

# 65. Anti-Pattern: No Clear Boundary Between Common and Native Code

A healthy project should make it easy to answer:

> "Which part of this feature is truly platform-specific?"

If the answer is:

```text
Almost everything
```

the common architecture may not be providing enough value.

If the answer is:

```text
Nothing, but we still have expect/actual everywhere
```

the project may be overusing the mechanism.

---

# 66. A Simple Diagnostic Model

When reviewing an `expect` / `actual` API, classify it:

```text
                    Is behavior platform-specific?
                              │
                    ┌─────────┴─────────┐
                    │                   │
                   No                  Yes
                    │                   │
                    ▼                   ▼
              Keep common        Does common code
                                  need the capability?
                                      │
                               ┌──────┴──────┐
                               │             │
                              No            Yes
                               │             │
                               ▼             ▼
                         Keep platform    Define small
                            specific       capability
                                             │
                                             ▼
                                      expect / actual
```

This simple decision tree prevents many unnecessary abstractions.

---

# 67. Before and After

## Before

```kotlin
expect class PlatformManager {

    fun getContext(): Any

    fun getPlatformName(): String

    fun saveToken(token: String)

    fun calculateDiscount(
        price: Double
    ): Double

    fun showPaymentMessage(
        message: String
    )
}
```

Problems:

- Platform leakage
- Business logic
- UI logic
- Storage
- Platform detection
- Too many responsibilities

---

## After

```kotlin
interface SecureStorage {
    fun save(key: String, value: String)
    fun read(key: String): String?
}

expect fun createSecureStorage(): SecureStorage
```

And:

```kotlin
class PaymentUseCase(
    private val storage: SecureStorage
) {
    fun execute() {
        // common business rules
    }
}
```

The platform boundary is now focused.

---

# 68. Anti-Pattern Checklist

Before merging `expect` / `actual` code, check for these warning signs:

- [ ] `Any` appears in the common API.
- [ ] Native Android types appear in common code.
- [ ] Native iOS types appear in common code.
- [ ] Business rules exist in `actual` implementations.
- [ ] Repositories are duplicated per platform.
- [ ] Use cases are duplicated per platform.
- [ ] Platform names are checked at runtime.
- [ ] A giant `PlatformManager` exists.
- [ ] A generic `Map<String, Any>` API is used.
- [ ] String-based platform commands are used.
- [ ] Native exceptions leak into common code.
- [ ] Native lifecycle requirements are hidden.
- [ ] Cancellation is ignored.
- [ ] Resource ownership is unclear.
- [ ] Permissions are hidden.
- [ ] Unsupported platforms silently do nothing.
- [ ] `expect` is used for identical logic.
- [ ] Platform-only dependencies leak into domain code.
- [ ] Native objects are stored in domain models.
- [ ] Global mutable platform state is introduced.
- [ ] `expect` / `actual` is being used to avoid dependency injection.
- [ ] Existing multiplatform libraries were ignored without evaluation.
- [ ] Intermediate source sets are created without real sharing.
- [ ] Platform implementations are forced to be symmetrical.
- [ ] Common APIs are designed around today's target list.
- [ ] UI side effects are triggered from domain logic.
- [ ] Common code depends on platform detection.
- [ ] The abstraction is either too broad or too generic.
- [ ] Tests require a real platform when they could use a fake.
- [ ] The platform boundary is difficult to explain in one sentence.

---

# 69. The Golden Rules

Keep these rules in mind when working with `expect` / `actual`:

> ### Rule 1
> **Do not use `expect` / `actual` for code that can simply live in `commonMain`.**

> ### Rule 2
> **Do not put business logic inside platform implementations.**

> ### Rule 3
> **Do not leak native types into common business code.**

> ### Rule 4
> **Expose capabilities, not platform objects.**

> ### Rule 5
> **Keep platform adapters small and focused.**

> ### Rule 6
> **Do not force Android and iOS to behave identically when the platforms are genuinely different.**

> ### Rule 7
> **Represent important platform limitations honestly.**

> ### Rule 8
> **Prefer explicit dependencies over hidden global platform state.**

> ### Rule 9
> **Use strong types instead of generic platform command APIs.**

> ### Rule 10
> **Treat `expect` / `actual` as a boundary, not as a replacement for architecture.**

---

# Final Takeaway

The biggest `expect` / `actual` anti-pattern is not a specific piece of syntax.

It is **putting the wrong responsibility on the platform boundary**.

A clean KMP architecture usually looks like:

```text
                COMMON
                   │
           Business rules
                   │
             Use cases
                   │
             Capability
                   │
          expect / actual
             /         \
            /           \
       Android          iOS
          │               │
     Native APIs      Native APIs
```

The common layer answers:

> **What does the application need?**

The platform layer answers:

> **How does this platform provide it?**

When those responsibilities are mixed, `expect` / `actual` becomes a source of duplication and coupling.

When they are separated, it becomes a precise architectural tool.

> **Share the business intent. Isolate the platform mechanics. Keep the boundary small.**

That is how `expect` / `actual` remains an advantage instead of becoming another layer of technical debt.
