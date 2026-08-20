# Chapter 8 — `expect` / `actual`

## Part 4 — Platform APIs

> **Kotlin Multiplatform does not remove platform APIs. It gives you a disciplined way to decide where those APIs belong.**

A KMP application can share a large amount of business logic while still using native Android, iOS, desktop, or web capabilities where they provide the best solution.

This is one of the most important ideas behind `expect` / `actual`:

```text
Common code defines what it needs.
Platform code decides how to provide it.
```

The goal is not to hide the platform.

The goal is to keep platform-specific details at a clear architectural boundary.

---

## 1. Why Platform APIs Matter

Every mobile platform provides capabilities that cannot be completely identical.

Examples include:

- Secure storage
- Permissions
- Notifications
- Camera
- Bluetooth
- Location
- Clipboard
- Sensors
- Biometrics
- Background execution
- Application lifecycle
- File system
- Device information
- Audio
- Haptics
- Platform sharing
- OS-level settings

A KMP application may share the business logic around these capabilities while using native APIs underneath.

For example:

```text
                    Common Application
                           │
                           ▼
                   SecureStorage API
                           │
                    expect / actual
                     ┌─────┴─────┐
                     ▼           ▼
                  Android       iOS
                     │           │
                     ▼           ▼
              Android APIs   Apple APIs
```

This separation is the foundation of a maintainable multiplatform architecture.

---

# 2. Platform APIs vs Common APIs

A platform API is an API supplied by the operating system or platform ecosystem.

Examples:

```text
Android
├── Context
├── Intent
├── Activity
├── Notification APIs
├── Camera APIs
└── Android Keystore

iOS
├── UIApplication
├── Foundation
├── UserNotifications
├── AVFoundation
└── Keychain Services
```

These APIs are valuable.

KMP does not require you to avoid them.

Instead, it encourages you to keep them out of places where they do not belong.

A common business layer should ideally not contain:

```kotlin
Context
Activity
UIApplication
UIViewController
```

unless there is a deliberate architectural reason.

---

# 3. The Platform Boundary

Consider an authentication feature.

A poor architecture might look like:

```text
LoginUseCase
     │
     ▼
Android Context
     │
     ▼
Android storage API
```

Now the use case is no longer truly common.

A better architecture is:

```text
LoginUseCase
     │
     ▼
SecureStorage
     │
     ▼
expect / actual
     │
 ┌───┴────┐
 ▼        ▼
Android   iOS
```

The business layer knows:

```text
SecureStorage
```

It does not know:

```text
Android Keystore implementation
iOS Keychain implementation
```

This is the platform boundary.

---

# 4. `expect` as a Platform Contract

An expected declaration describes a capability that the common source set requires.

For example:

```kotlin
expect class DeviceInfo {
    val model: String
    val osVersion: String
}
```

The common code says:

> Every supported platform must provide `DeviceInfo`.

The platform source set provides the implementation:

```kotlin
actual class DeviceInfo {
    actual val model: String = ...
    actual val osVersion: String = ...
}
```

The expected declaration is therefore a contract.

```text
commonMain
     │
     │ expects
     ▼
DeviceInfo
     │
     ├───────────────┐
     ▼               ▼
androidMain       iosMain
 actual             actual
```

---

# 5. Platform APIs Are Not All the Same

A common mistake is assuming that an Android API must have an identical iOS equivalent.

Usually it does not.

For example:

```text
Android Activity
```

does not have a perfect one-to-one replacement on iOS.

Similarly:

```text
Android Intent
```

and:

```text
iOS URL / navigation mechanisms
```

represent different platform concepts.

Therefore, a good abstraction should describe the **application capability**, not the underlying platform API.

Instead of:

```kotlin
expect fun getActivity(): Any
```

prefer:

```kotlin
expect fun openUrl(url: String)
```

The second API communicates intent.

---

# 6. Example — Opening a URL

### Common

```kotlin
expect fun openUrl(url: String)
```

Common business code:

```kotlin
class HelpUseCase {
    fun openHelp() {
        openUrl("https://example.com/help")
    }
}
```

Android can use its appropriate platform mechanism.

iOS can use its appropriate platform mechanism.

The use case remains common.

The platform implementation owns the platform details.

---

# 7. Example — Secure Storage

Secure storage is a classic platform boundary.

### Common

```kotlin
expect class SecureStorage {
    fun save(key: String, value: String)
    fun read(key: String): String?
    fun delete(key: String)
}
```

The Android implementation can integrate with Android's supported secure-storage mechanisms.

The iOS implementation can integrate with Apple's supported secure-storage mechanisms.

The common code remains:

```kotlin
class TokenRepository(
    private val storage: SecureStorage
) {

    fun saveToken(token: String) {
        storage.save("token", token)
    }

    fun getToken(): String? {
        return storage.read("token")
    }
}
```

> [!IMPORTANT]
> Security-sensitive code should use the platform's supported security facilities and current security guidance. Do not create custom cryptography simply to make a multiplatform example work.

---

# 8. Example — Clipboard

The common application may need:

```kotlin
expect class Clipboard {
    fun copy(text: String)
    fun paste(): String?
}
```

Common code:

```kotlin
class CopyInviteCodeUseCase(
    private val clipboard: Clipboard
) {
    fun execute(code: String) {
        clipboard.copy(code)
    }
}
```

The Android implementation uses Android's clipboard APIs.

The iOS implementation uses Apple's clipboard APIs.

The use case does not know either implementation.

---

# 9. Example — Device Information

A common diagnostics feature might need:

```kotlin
expect class DeviceInfo {
    val platform: String
    val model: String
    val osVersion: String
}
```

Common analytics code:

```kotlin
class DiagnosticsService(
    private val deviceInfo: DeviceInfo
) {
    fun metadata(): Map<String, String> {
        return mapOf(
            "platform" to deviceInfo.platform,
            "model" to deviceInfo.model,
            "osVersion" to deviceInfo.osVersion
        )
    }
}
```

Platform APIs provide the actual values.

The common service consumes only the abstraction.

---

# 10. Example — Permissions

Permissions are strongly platform-specific.

For example:

```kotlin
expect class CameraPermission {
    suspend fun request(): Boolean
}
```

Common code:

```kotlin
suspend fun startScanning(): Boolean {
    return cameraPermission.request()
}
```

The Android implementation follows Android's permission model.

The iOS implementation follows Apple's authorization model.

The common layer receives a result:

```text
Granted
or
Denied
```

It does not need to understand every platform-specific permission API.

---

# 11. Why Permission APIs Should Stay at the Edge

Permission handling often involves:

- Lifecycle
- UI
- OS callbacks
- User settings
- Permission states
- Platform-specific restrictions

Trying to reproduce the complete platform permission model in common code can create unnecessary complexity.

A better boundary is:

```text
Platform UI
     │
     ▼
Permission implementation
     │
     ▼
Common state / business decision
```

The common layer should usually consume a meaningful result rather than raw platform objects.

---

# 12. Example — Notifications

Notification registration is platform-specific.

A common application may define:

```kotlin
expect class NotificationPermission {
    suspend fun request(): Boolean
}
```

Then the platform layer manages:

```text
Permission request
Registration
Platform callback
Token retrieval
```

The common layer can manage:

```text
User preference
Notification state
Business rules
```

This creates a clean separation.

---

# 13. Example — Push Token

A common backend may need a device push token.

The platform capability could be:

```kotlin
expect suspend fun getPushToken(): String?
```

The Android implementation uses the configured Android push infrastructure.

The iOS implementation uses the configured Apple push infrastructure.

The common repository can then send the token to the backend.

```text
Platform
   │
   ▼
Push Token
   │
   ▼
Common Repository
   │
   ▼
Backend
```

The backend contract stays common even though token acquisition is platform-specific.

---

# 14. Example — Biometrics

Biometric authentication is another strong example.

```kotlin
expect class BiometricAuthenticator {
    suspend fun authenticate(reason: String): Boolean
}
```

Common code:

```kotlin
class ConfirmPaymentUseCase(
    private val biometricAuthenticator: BiometricAuthenticator
) {
    suspend fun execute(): Boolean {
        return biometricAuthenticator.authenticate(
            reason = "Confirm payment"
        )
    }
}
```

Android and iOS implement the capability using their respective native security frameworks.

The use case remains common.

---

# 15. Example — Camera

Camera APIs are highly platform-specific.

A common feature might expose:

```kotlin
expect class CameraController {
    fun start()
    fun stop()
}
```

However, this is only a simplified abstraction.

A real camera system may also require:

```text
Permissions
Preview
Capture
Lifecycle
Orientation
Resolution
Focus
Flash
Camera selection
Error handling
```

These details may be better handled largely in platform-specific UI and camera modules.

The key lesson is:

> Do not force a complex native API into a tiny common abstraction if doing so makes the design harder to understand.

---

# 16. Example — Location

A location feature might need:

```kotlin
data class Location(
    val latitude: Double,
    val longitude: Double
)
```

Common capability:

```kotlin
expect class LocationProvider {
    suspend fun currentLocation(): Location?
}
```

Android and iOS handle:

```text
Permission
Location service
Lifecycle
Provider configuration
Platform callbacks
```

The common application receives:

```text
Location?
```

rather than a platform-specific location object.

---

# 17. Example — Network Connectivity

A shared repository may need to know whether the device appears connected.

```kotlin
expect class NetworkMonitor {
    fun isConnected(): Boolean
}
```

Common code:

```kotlin
if (networkMonitor.isConnected()) {
    repository.refresh()
}
```

The platform implementation deals with the native connectivity APIs.

> [!NOTE]
> Connectivity status is not the same as guaranteed Internet reachability. Application logic should still handle actual network failures.

---

# 18. Example — File System

A shared cache may need a writable application directory.

```kotlin
expect fun appDataDirectory(): String
```

Common storage code can work with the returned capability.

The platform implementation resolves the appropriate location.

But if the project uses a mature multiplatform filesystem library, prefer that library when it satisfies the requirements.

---

# 19. Example — Logging

A common logging abstraction can be small:

```kotlin
expect object AppLogger {
    fun debug(message: String)
    fun info(message: String)
    fun error(message: String, throwable: Throwable? = null)
}
```

Common code:

```kotlin
AppLogger.info("Starting synchronization")
```

The Android implementation can connect to the Android logging ecosystem.

The iOS implementation can connect to Apple's logging facilities.

The common code remains clean.

---

# 20. Example — Haptics

Haptic feedback is platform-specific.

A common capability might be:

```kotlin
expect fun performHapticFeedback()
```

The platform implementation chooses the appropriate native mechanism.

Common UI-related business logic can request:

```text
Success feedback
Error feedback
Selection feedback
```

without importing platform APIs into common code.

---

# 21. Example — Sharing Content

A shared feature may need:

```kotlin
expect fun shareText(text: String)
```

Common code:

```kotlin
fun shareInvite(code: String) {
    shareText("Join with code: $code")
}
```

Android and iOS provide their respective sharing mechanisms.

The common code owns the message.

The platform owns the sharing UI.

---

# 22. Example — Application Settings

Suppose a feature needs to open the application's settings page.

Common API:

```kotlin
expect fun openApplicationSettings()
```

Android and iOS can implement this according to their platform capabilities.

The common feature can respond to:

```text
Permission denied
```

by offering:

```text
Open Settings
```

without knowing the exact native navigation API.

---

# 23. Example — Background Work

Background execution is one of the areas where platform differences matter most.

A simplistic abstraction might be:

```kotlin
expect class BackgroundScheduler {
    fun schedule()
}
```

But real platforms impose different restrictions.

Therefore:

```text
commonMain
    │
    └── decides WHAT work is required
              │
              ▼
       platform boundary
              │
       ┌──────┴──────┐
       ▼             ▼
    Android          iOS
    scheduler       scheduler
```

Each platform implementation must respect its own lifecycle and background-execution rules.

---

# 24. Example — Application Lifecycle

Do not attempt to make Android and iOS lifecycle models identical.

Instead, isolate the specific event or capability required.

For example, if a shared service needs to know that the application became active:

```kotlin
interface AppActivityObserver {
    fun onActive()
}
```

The platform UI/lifecycle layer can call the common observer.

This may be cleaner than creating a huge:

```kotlin
expect class PlatformLifecycle
```

abstraction.

---

# 25. Example — Application Context

A frequent anti-pattern is:

```kotlin
expect fun getContext(): Context
```

This is problematic because `Context` is an Android type.

It forces the common layer to understand Android.

Instead, identify what the code actually needs.

If it needs:

```text
Application files directory
```

expose:

```kotlin
expect fun appDataDirectory(): String
```

If it needs:

```text
Secure storage
```

expose:

```kotlin
expect fun createSecureStorage(): SecureStorage
```

If it needs:

```text
Open a URL
```

expose:

```kotlin
expect fun openUrl(url: String)
```

The capability should replace the platform object.

---

# 26. The Capability Principle

A useful rule is:

> **Expose the capability, not the platform.**

Avoid:

```kotlin
expect fun getContext(): Any
```

Prefer:

```kotlin
expect fun getAppDataDirectory(): String
```

Avoid:

```kotlin
expect fun getActivity(): Any
```

Prefer:

```kotlin
expect fun openUrl(url: String)
```

Avoid:

```kotlin
expect fun getUIApplication(): Any
```

Prefer:

```kotlin
expect fun shareText(text: String)
```

The second version is easier to test, easier to replace, and easier to understand.

---

# 27. Platform API Adapter Pattern

A common architecture is:

```text
commonMain
     │
     ▼
Platform Capability
     │
     ▼
Adapter
     │
     ▼
Native API
```

For example:

```text
SecureStorage
     │
     ▼
AndroidSecureStorage
     │
     ▼
Android security APIs
```

and:

```text
SecureStorage
     │
     ▼
IosSecureStorage
     │
     ▼
Apple security APIs
```

The adapter isolates the native API.

---

# 28. Platform APIs and Dependency Injection

`expect` / `actual` works well with dependency injection.

For example:

```kotlin
class UserRepository(
    private val secureStorage: SecureStorage
)
```

The repository does not construct the platform implementation.

The platform-specific composition layer provides it.

Conceptually:

```text
Android DI
    │
    └── AndroidSecureStorage

iOS DI
    │
    └── IosSecureStorage

             ↓

       UserRepository
```

This keeps construction separate from business logic.

---

# 29. `expect` Factory + Interface

A useful pattern is:

### Common

```kotlin
interface SecureStorage {
    fun save(key: String, value: String)
    fun read(key: String): String?
}

expect fun createSecureStorage(): SecureStorage
```

Android:

```kotlin
actual fun createSecureStorage(): SecureStorage {
    return AndroidSecureStorage()
}
```

iOS:

```kotlin
actual fun createSecureStorage(): SecureStorage {
    return IosSecureStorage()
}
```

This gives you:

```text
expect / actual
        +
interface
        +
platform implementation
```

without leaking platform APIs into common code.

---

# 30. Platform APIs and Source Sets

The source-set hierarchy helps organize platform dependencies.

A simplified structure:

```text
commonMain
   │
   ├── common API
   └── expect declarations
        │
        ├── androidMain
        │
        └── iosMain
```

If multiple targets can share an implementation, an intermediate source set may be better.

For example:

```text
commonMain
     │
     ▼
appleMain
   ├── iosMain
   ├── macosMain
   └── tvosMain
```

This prevents duplication when Apple targets can genuinely share the same implementation.

---

# 31. Platform APIs and Intermediate Source Sets

Suppose several Apple targets use the same Foundation capability.

Instead of:

```text
iosMain
macosMain
tvosMain
```

each implementing the same code independently, an intermediate source set may contain the shared implementation.

Conceptually:

```text
             commonMain
                 │
              appleMain
          ┌──────┼──────┐
          ▼      ▼      ▼
        iOS    macOS   tvOS
```

This is one of the reasons source-set hierarchy matters.

The platform boundary does not always mean:

```text
one implementation per target
```

It can mean:

```text
one implementation per compatible group of targets
```

---

# 32. Platform APIs and Testing

Platform code should not become untestable.

For example:

```kotlin
interface SecureStorage {
    fun save(key: String, value: String)
    fun read(key: String): String?
}
```

Common tests can use:

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

Now common business logic can be tested without Android or iOS.

This is a major benefit of capability-based design.

---

# 33. Testing the `actual` Implementation

The real platform implementation should also be tested where practical.

For example:

```text
commonTest
    ↓
Business behavior

android-specific tests
    ↓
Android implementation

iOS-specific tests
    ↓
iOS implementation
```

The exact test setup depends on the platform and project architecture.

The important distinction is:

```text
Common tests verify common behavior.
Platform tests verify platform integration.
```

---

# 34. Platform APIs and Error Handling

Native APIs may fail differently.

For example:

```text
Android permission denied
iOS permission denied
```

may expose different underlying states.

The common layer can map them into a domain-level result:

```kotlin
sealed interface PermissionResult {
    data object Granted : PermissionResult
    data object Denied : PermissionResult
}
```

Platform implementations translate native states into the common result.

This keeps platform error details at the boundary.

---

# 35. Platform Exceptions

Avoid exposing platform-specific exceptions to common business logic.

Instead of:

```kotlin
catch (AndroidSpecificException) { ... }
```

in common code, map the error at the platform boundary when appropriate.

For example:

```text
Native exception
      ↓
Platform adapter
      ↓
Common error model
      ↓
Business logic
```

This makes common code portable.

---

# 36. Platform APIs and Domain Models

Do not put platform objects directly into domain models.

Avoid:

```kotlin
data class UserLocation(
    val nativeLocation: Any
)
```

Prefer:

```kotlin
data class UserLocation(
    val latitude: Double,
    val longitude: Double
)
```

The domain model should describe business data.

The platform adapter converts native objects into the domain representation.

---

# 37. Platform APIs and Repository Boundaries

A repository should normally expose application-level data.

For example:

```kotlin
interface UserRepository {
    suspend fun getUser(): User
}
```

Not:

```kotlin
interface UserRepository {
    fun getAndroidCursor(): Any
}
```

or:

```kotlin
interface UserRepository {
    fun getIosNativeObject(): Any
}
```

The repository should hide platform infrastructure.

---

# 38. Platform APIs and Business Logic

A useful mental model is:

```text
Business logic asks:
"What do I need?"

Platform code answers:
"How can this platform provide it?"
```

For example:

```text
Business:
"Store the authentication token securely."

Android:
"Use the supported Android security mechanism."

iOS:
"Use the supported Apple security mechanism."
```

This is exactly the kind of boundary KMP can represent well.

---

# 39. When a Platform API Should Stay Platform-Specific

Not every native API needs a common wrapper.

If a feature exists only in Android UI:

```text
Android-specific screen
```

keep it in Android.

If a feature exists only in iOS:

```text
iOS-specific UI
```

keep it in iOS.

Creating a common abstraction for something that will never be used by common code adds unnecessary complexity.

A simple rule:

> **If common code does not need the capability, do not force it into common code.**

---

# 40. Platform APIs and Native UI

KMP does not require a single UI technology.

You can have:

```text
Shared business logic
        │
 ┌──────┴───────┐
 ▼              ▼
Android UI     iOS UI
```

The platform UI can use native platform APIs naturally.

The shared layer can provide:

```text
State
Actions
Use cases
Models
Validation
```

This often provides an excellent balance between code sharing and native experience.

---

# 41. Platform APIs With Compose Multiplatform

If Compose Multiplatform is used, some UI can also be shared.

Still, platform APIs remain relevant.

For example:

```text
Shared Compose UI
       │
       ▼
Common capability
       │
       ▼
Platform implementation
```

The UI can remain common while a specific device capability remains platform-specific.

This demonstrates that:

```text
Shared UI
```

and:

```text
Shared platform implementation
```

are separate decisions.

---

# 42. Platform APIs With Native UI

A native UI architecture can look like:

```text
             commonMain
                 │
        ┌────────┴────────┐
        ▼                 ▼
 Android presentation   iOS presentation
        │                 │
        └────────┬────────┘
                 ▼
          Shared domain/data
```

The platform UI can use native APIs directly.

The shared layers remain independent.

This is often useful when an application has strong platform-specific UI requirements.

---

# 43. Platform API Wrapper vs Library

Before writing:

```kotlin
expect class SomeService
```

ask:

```text
Does a maintained multiplatform library already provide this?
```

If yes, evaluate:

- API quality
- Platform coverage
- Maintenance
- Version compatibility
- Performance
- Security
- Community adoption
- Project requirements

A good library can remove a large amount of platform-specific maintenance.

But the library should still be evaluated rather than adopted automatically.

---

# 44. Avoiding Abstraction for Abstraction's Sake

This is an important architectural warning.

Bad:

```text
Native API
    ↓
PlatformWrapper
    ↓
PlatformService
    ↓
CommonService
    ↓
Repository
```

when every layer simply forwards the same method.

Better:

```text
Repository
    ↓
Meaningful capability
    ↓
Native implementation
```

The abstraction should provide architectural value.

---

# 45. Platform APIs and Performance

Platform APIs can be performance-sensitive.

Examples:

```text
Camera
Location
Bluetooth
Audio
Database
File I/O
Sensors
Image processing
```

Do not assume that wrapping an API automatically makes it efficient.

Consider:

- Threading
- Memory
- Lifecycle
- Resource ownership
- Callback frequency
- Cancellation
- Battery impact
- Main-thread restrictions

The platform implementation is responsible for respecting platform performance requirements.

---

# 46. Platform APIs and Coroutines

A platform API may expose callbacks.

The common layer may prefer:

```kotlin
suspend fun currentLocation(): Location?
```

The platform implementation can adapt the native callback mechanism to the common suspend API.

Conceptually:

```text
Native callback API
        ↓
actual implementation
        ↓
suspend function
        ↓
common business logic
```

This can make the shared layer much easier to reason about.

---

# 47. Platform APIs and Flow

Some platform capabilities are continuous streams.

Examples:

```text
Network status
Location updates
Battery state
Sensor data
Lifecycle events
```

A common abstraction could expose:

```kotlin
expect class NetworkMonitor {
    val status: Flow<NetworkStatus>
}
```

The platform implementation adapts the native event source to a common `Flow`.

The common layer can then react to:

```kotlin
networkMonitor.status.collect { status ->
    // common behavior
}
```

The native callback mechanism stays platform-specific.

---

# 48. Platform APIs and Cancellation

When wrapping asynchronous native APIs, cancellation matters.

A platform implementation should avoid:

```text
start native operation
    ↓
coroutine cancelled
    ↓
native operation continues forever
```

A good adapter should understand how the native operation can be stopped or ignored safely.

This is especially important for:

- Location
- Camera
- Bluetooth
- Network calls
- Background work
- Sensor streams

---

# 49. Platform APIs and Lifecycle Ownership

Native APIs often have lifecycle requirements.

For example:

```text
Camera
Location
Observers
Listeners
UI controllers
```

The platform implementation should own lifecycle-sensitive resources.

Do not force common business logic to manage native lifecycle objects.

A healthy separation is:

```text
Common:
"What should happen?"

Platform:
"When is it safe to interact with the native API?"
```

---

# 50. Platform APIs and Resource Cleanup

Native resources often require cleanup.

Examples:

```text
Listener registration
Camera session
Location updates
Observers
Streams
Files
Connections
```

A platform adapter should have a clear ownership model.

For example:

```kotlin
interface CloseableResource {
    fun close()
}
```

or a lifecycle-aware API appropriate to the feature.

The common abstraction should not leak native resource-management details unnecessarily.

---

# 51. Platform API Design for Future Targets

A good capability abstraction should not assume only two platforms.

Avoid:

```kotlin
fun doAndroidOrIosThing()
```

Prefer:

```kotlin
fun shareText(text: String)
```

Then a future target can provide its own implementation if it supports the capability.

This makes the API target-neutral.

---

# 52. Adding a New Platform

Suppose a project initially supports:

```text
Android
iOS
```

and later adds:

```text
Desktop
```

A well-designed platform boundary should make the work obvious.

For example:

```text
expect fun openUrl(url: String)
```

The new target provides its implementation.

The common business logic remains unchanged.

That is one of the strongest benefits of a capability-based abstraction.

---

# 53. Platform API Decision Framework

When you encounter a native API, ask:

### Question 1

Does common code need this capability?

```text
No → Keep it platform-specific.
Yes → Continue.
```

### Question 2

Does a multiplatform library already provide it?

```text
Yes → Evaluate and potentially use it.
No → Continue.
```

### Question 3

Can an interface express the capability?

```text
Yes → Prefer interface + DI when appropriate.
No → Continue.
```

### Question 4

Is the platform difference naturally represented by an expected declaration?

```text
Yes → Consider expect / actual.
```

### Question 5

Is the abstraction small and meaningful?

```text
Yes → Good boundary.
No → Redesign it.
```

---

# 54. A Complete Example

Consider an offline-first shopping application.

The application needs:

```text
Authentication
Product data
Offline storage
Network status
Secure token storage
Device analytics
```

A possible architecture:

```text
                    commonMain
                        │
          ┌─────────────┼─────────────┐
          ▼             ▼             ▼
       Domain         Data        Platform APIs
          │             │             │
          │             │          expect
          │             │             │
          │             │      ┌──────┴──────┐
          │             │      ▼             ▼
          │             │   Android          iOS
          │             │    actual         actual
          │             │
          └─────────────┴──────────────┐
                                       ▼
                                Business Logic
```

The shared repository might look conceptually like:

```kotlin
class ProductRepository(
    private val api: ProductApi,
    private val storage: ProductStorage,
    private val networkMonitor: NetworkMonitor
) {

    suspend fun products(): List<Product> {
        return if (networkMonitor.isConnected()) {
            val products = api.fetchProducts()
            storage.save(products)
            products
        } else {
            storage.getProducts()
        }
    }
}
```

The platform layer provides only what is required.

---

# 55. Common Architecture Smell

Consider:

```kotlin
class ProductRepository(
    private val context: Context
)
```

This should immediately raise a question:

> Why does the repository need an Android `Context`?

If the answer is:

```text
To access a file directory
```

then expose:

```kotlin
Storage
```

If the answer is:

```text
To access secure credentials
```

then expose:

```kotlin
SecureStorage
```

If the answer is:

```text
To open another screen
```

then that responsibility probably belongs closer to the presentation/platform layer.

The platform object is often a symptom that the capability boundary has not been defined yet.

---

# 56. The Golden Rule

A useful KMP rule is:

> **Do not share platform objects. Share platform capabilities.**

Instead of:

```text
Context
Activity
UIViewController
UIApplication
NSUrl
Android Cursor
```

common code should preferably work with:

```text
Storage
Navigator
UrlOpener
Clipboard
Location
User
Result
State
```

This makes the common layer meaningful rather than artificially platform-neutral.

---

# 57. Platform APIs and Clean Architecture

The dependency direction should remain:

```text
Presentation
     ↓
Domain
     ↓
Data
     ↓
Platform capability
```

Platform implementation details should not point upward into domain rules.

For example:

```text
Android API
    ↓
Android adapter
    ↓
SecureStorage
    ↓
Repository
    ↓
Use Case
```

This keeps the dependency direction predictable.

---

# 58. Platform APIs and SOLID Principles

Good platform boundaries naturally support several design principles.

### Single Responsibility

A platform adapter should handle one capability.

### Dependency Inversion

Common business logic depends on an abstraction.

### Interface Segregation

Small capabilities are better than a giant platform interface.

### Open/Closed

New platform implementations can be added without rewriting common business logic.

The goal is not to apply principles mechanically.

The goal is to create boundaries that make the architecture easier to evolve.

---

# 59. Common Mistakes

### ❌ Leaking Android types

```kotlin
expect fun getContext(): Context
```

### ❌ Creating giant platform abstractions

```kotlin
expect class PlatformManager
```

with dozens of unrelated APIs.

### ❌ Duplicating common business logic

Platform code should implement the capability, not duplicate the entire feature.

### ❌ Ignoring platform lifecycle rules

Native APIs must be used according to their platform lifecycle.

### ❌ Ignoring cancellation

Asynchronous native work must be stopped or handled safely when the common operation is cancelled.

### ❌ Recreating existing multiplatform libraries

Check the ecosystem first.

### ❌ Sharing the wrong abstraction

If the common API is designed around Android terminology, it may not represent iOS or future targets well.

---

# 60. Platform API Checklist

Before exposing a platform API through `expect` / `actual`, verify:

- [ ] Common code genuinely needs the capability.
- [ ] The API describes intent rather than implementation.
- [ ] No Android or iOS framework types leak into `commonMain`.
- [ ] The expected declaration is small.
- [ ] The platform implementation is isolated.
- [ ] Error handling is mapped appropriately.
- [ ] Lifecycle requirements are respected.
- [ ] Cancellation is handled.
- [ ] Resource ownership is clear.
- [ ] Security requirements are respected.
- [ ] Testing is possible.
- [ ] A multiplatform library has been considered.
- [ ] The abstraction can support future targets.
- [ ] The platform implementation does not duplicate business logic.
- [ ] The source-set hierarchy is appropriate.

---

# 61. Chapter Takeaways

> [!IMPORTANT]
> **Platform APIs are not the enemy of KMP. Uncontrolled platform dependencies are.**

Remember:

1. KMP does not eliminate native platform APIs.
2. Native APIs remain essential for many device capabilities.
3. `expect` / `actual` provides an explicit platform boundary.
4. The common layer should define the capability it needs.
5. The platform layer should decide how that capability is implemented.
6. Expose capabilities instead of platform objects.
7. Avoid leaking `Context`, `Activity`, or `UIViewController` into common code.
8. Secure storage is a strong example of a platform capability.
9. Clipboard access is naturally platform-specific.
10. Device information can be exposed through a small abstraction.
11. Permissions should be handled close to the platform.
12. Notification registration is platform-specific.
13. Push-token acquisition can be isolated behind a common capability.
14. Biometrics should use native security facilities.
15. Camera APIs should remain close to platform-specific code.
16. Location APIs can be adapted into common data models.
17. Connectivity monitoring can expose common state.
18. File-system access should use the smallest required capability.
19. Logging can have a common API and platform implementation.
20. Haptics can be isolated behind a capability.
21. Sharing content can use a platform-specific implementation.
22. Application settings navigation can be represented by intent.
23. Background execution must respect platform rules.
24. Lifecycle models should not be artificially unified.
25. `expect` should not expose Android or iOS framework objects.
26. Interfaces and dependency injection can complement `expect` / `actual`.
27. Factories can be provided through expected declarations.
28. Intermediate source sets can share implementations across compatible targets.
29. Platform implementations should be testable.
30. Common tests should focus on common behavior.
31. Platform tests should verify platform integration.
32. Native errors can be translated into common error models.
33. Domain models should not contain native platform objects.
34. Repositories should expose application-level data.
35. Platform-specific behavior should remain near the platform edge.
36. Shared UI does not automatically mean shared platform implementation.
37. Native UI does not prevent sharing business logic.
38. Compose Multiplatform can coexist with platform-specific APIs.
39. Multiplatform libraries should be evaluated before creating custom wrappers.
40. Avoid abstractions that only forward calls without adding value.
41. Performance-sensitive platform APIs need careful lifecycle and threading design.
42. Callback-based APIs can be adapted to suspending APIs.
43. Event-based APIs can be adapted to `Flow`.
44. Cancellation must be considered when wrapping native asynchronous APIs.
45. Native resource ownership must be explicit.
46. Platform abstractions should not assume only Android and iOS.
47. A good capability can often support additional targets naturally.
48. Common business logic should remain independent of native APIs.
49. Platform adapters should implement capabilities rather than business features.
50. A platform object appearing in common code is often an architectural warning sign.
51. The best abstraction represents application intent.
52. The smallest useful capability is usually the safest boundary.
53. `expect` / `actual` should be intentional, not automatic.
54. The goal is not to hide the platform.
55. The goal is to control where platform knowledge exists.
56. **Good KMP architecture shares intent while respecting platform differences.**

---

## Final Thought

Kotlin Multiplatform is strongest when it does not pretend that Android and iOS are the same.

They are not.

They have different:

```text
APIs
Lifecycle models
Security systems
Permission models
UI frameworks
Background execution rules
Hardware APIs
Application environments
```

A mature KMP architecture acknowledges those differences.

It creates a clean boundary:

```text
                 COMMON INTENT
                      │
                      ▼
               commonMain
                      │
             capability API
                      │
                 expect
                      │
          ┌───────────┴───────────┐
          ▼                       ▼
      Android                     iOS
       actual                    actual
          │                       │
          ▼                       ▼
    Native Android           Native Apple
        APIs                    APIs
```

The common code says:

> **What does the application need?**

The platform implementation says:

> **How does this platform provide it?**

That separation is the real value of `expect` / `actual`.

> **KMP is not about replacing native APIs. It is about using them deliberately, behind boundaries that keep shared architecture clean.**
