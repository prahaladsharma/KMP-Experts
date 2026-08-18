# Chapter 7 — Source Sets Deep Dive

## Part 3 — `androidMain`

> **`commonMain` defines what can be shared. `androidMain` defines what Android uniquely needs.**

Kotlin Multiplatform does not mean removing Android from the architecture.

It means giving Android-specific code a clear boundary.

That boundary is typically:

```text
androidMain
```

If `commonMain` contains platform-independent implementation, `androidMain` contains the Android-specific implementation required by the shared architecture.

A simplified KMP module looks like:

```text
shared/
└── src/
    ├── commonMain/
    ├── commonTest/
    ├── androidMain/
    └── androidUnitTest/
```

The important relationship is:

```text
                 commonMain
                     │
             Shared abstractions
                     │
          ┌──────────┴──────────┐
          ▼                     ▼
     androidMain              iosMain
          │
          ▼
 Android implementation
```

The purpose of `androidMain` is not to duplicate the application.

Its purpose is to provide the Android implementation where the platform genuinely matters.

---

# 1. What Is `androidMain`?

`androidMain` is the Android-specific source set in a Kotlin Multiplatform project.

Code placed there is compiled for the Android target and can use Android-specific APIs when the module and target configuration allow them.

Typical examples include:

```text
Android Context
Android resources
Android lifecycle APIs
Android-specific storage
Android platform services
Android SDK APIs
Android-specific libraries
Android native integrations
```

The source-set structure might look like:

```text
shared/
└── src/
    ├── commonMain/
    │   └── kotlin/
    │
    ├── commonTest/
    │   └── kotlin/
    │
    ├── androidMain/
    │   └── kotlin/
    │
    └── androidUnitTest/
        └── kotlin/
```

The key idea is:

> **Android-specific code belongs at the Android boundary instead of leaking into common code.**

---

# 2. Why `androidMain` Exists

Consider a shared application that needs secure storage.

The requirement is common:

```text
Save sensitive application data.
Read sensitive application data.
Delete sensitive application data.
```

But the actual implementation may differ by platform.

A common abstraction can live in:

```text
commonMain
```

For example:

```kotlin
interface SecureStorage {

    fun save(
        key: String,
        value: String
    )

    fun read(
        key: String
    ): String?

    fun remove(
        key: String
    )
}
```

Android can provide:

```text
androidMain
    ↓
Android SecureStorage implementation
```

iOS can provide:

```text
iosMain
    ↓
iOS SecureStorage implementation
```

The architecture becomes:

```text
                 commonMain
                     │
              SecureStorage
                 interface
                     │
          ┌──────────┴──────────┐
          ▼                     ▼
     androidMain              iosMain
          │                     │
    Android storage        iOS storage
```

This is the essence of platform-specific source sets.

---

# 3. `androidMain` Is Not a Second Application

A common mistake is to think:

```text
commonMain = shared app
androidMain = Android app
```

That is not the right mental model.

Instead:

```text
commonMain
    +
androidMain
    +
iosMain
    =
complete multiplatform application
```

The common source set contains shared behavior.

The platform source set contains platform-specific behavior.

---

# 4. Common Code Depends on Abstractions

Suppose:

```kotlin
interface PlatformLogger {
    fun log(message: String)
}
```

Common code can use:

```kotlin
class UserRepository(
    private val logger: PlatformLogger
) {
    // Shared repository logic
}
```

Android can provide:

```kotlin
class AndroidLogger : PlatformLogger {

    override fun log(message: String) {
        println("Android: $message")
    }
}
```

The Android implementation belongs in:

```text
androidMain
```

The common repository does not need to know how Android logging works.

---

# 5. The Platform Boundary

A useful architectural picture is:

```text
                 COMMON LAYER
        ┌──────────────────────────┐
        │ Domain                   │
        │ Use Cases                │
        │ Repository contracts     │
        │ Business rules           │
        │ Shared state             │
        └────────────┬─────────────┘
                     │
              Platform boundary
                     │
        ┌────────────┴─────────────┐
        ▼                          ▼
   androidMain                  iosMain
        │                          │
 Android APIs                Apple APIs
```

This boundary should be intentional.

The more platform-specific code stays near the edge, the easier the shared architecture is to reason about.

---

# 6. What Belongs in `androidMain`?

Typical examples include:

### Android framework integration

```text
Context
Activity
Service
BroadcastReceiver
ContentResolver
Android lifecycle APIs
```

### Android platform services

```text
Connectivity
Bluetooth
Location
Notifications
Biometrics
Sensors
```

### Android-specific storage

```text
SharedPreferences
DataStore
Room Android integration
Files
Keystore
```

### Android-specific SDKs

```text
Android SDK
AndroidX APIs
Google Play services
Android-specific vendor SDKs
```

### Android-specific implementations

```text
PlatformLogger
SecureStorage
PlatformDatabase
NetworkMonitor
PermissionHandler
```

---

# 7. What Should Not Automatically Go Into `androidMain`?

Do not move code into `androidMain` merely because it is currently used by Android.

Ask:

> **Is this behavior actually Android-specific?**

For example:

```kotlin
fun calculateDiscount(
    price: Double,
    percentage: Double
): Double
```

does not need Android.

It should remain in:

```text
commonMain
```

Moving it to `androidMain` would unnecessarily reduce sharing.

---

# 8. The Golden Rule

A simple rule is:

```text
If it can be common → commonMain

If it must be Android-specific → androidMain
```

This sounds simple, but applying it consistently is one of the most important KMP architecture skills.

---

# 9. Android Implementations of Common Interfaces

Suppose common code defines:

```kotlin
interface NetworkMonitor {
    fun isOnline(): Boolean
}
```

Android can implement it using Android networking APIs.

Conceptually:

```text
commonMain
    │
    └── NetworkMonitor
             ▲
             │
        androidMain
             │
      AndroidNetworkMonitor
```

The shared business logic can depend on:

```kotlin
NetworkMonitor
```

rather than:

```kotlin
ConnectivityManager
```

That distinction keeps the common layer portable.

---

# 10. Example: Android Network Monitor

An Android implementation may use:

```kotlin
class AndroidNetworkMonitor(
    private val context: Context
) : NetworkMonitor {

    override fun isOnline(): Boolean {
        // Android-specific network detection.
        return true
    }
}
```

The exact implementation depends on the Android API level and application requirements.

The architectural point is more important:

```text
ConnectivityManager
        ↓
androidMain
        ↓
NetworkMonitor
        ↓
commonMain
```

Android APIs stop at the platform boundary.

---

# 11. `expect` and `actual`

Kotlin Multiplatform also provides another mechanism for platform-specific implementations:

```text
expect
actual
```

The common source set can declare an expected API:

```kotlin
expect class PlatformInfo {
    val name: String
}
```

Android can provide:

```kotlin
actual class PlatformInfo {
    actual val name: String = "Android"
}
```

The common declaration describes the API.

The Android implementation provides the platform-specific behavior.

---

# 12. `expect`/`actual` Mental Model

Think of it as:

```text
commonMain

expect
  │
  ├───────────────┐
  ▼               ▼
androidMain     iosMain
  │               │
actual          actual
```

The common layer says:

> "I need this capability."

The platform source set says:

> "Here is how my platform provides it."

---

# 13. `expect` Is Not a Replacement for Abstractions

A common misconception is:

> "Whenever I need platform-specific code, I should use `expect`/`actual`."

Not necessarily.

You can also use:

```text
interfaces
dependency injection
factory functions
platform implementations
```

For example:

```kotlin
interface SecureStorage
```

may be a cleaner abstraction than an `expect` class depending on the architecture.

The correct mechanism depends on the dependency boundary.

---

# 14. Interface vs `expect`/`actual`

A useful comparison:

| Requirement | Possible approach |
|---|---|
| Platform implementation injected into shared logic | Interface + DI |
| Small platform-specific utility | `expect` / `actual` |
| Platform service abstraction | Interface |
| Platform-specific object creation | Factory |
| Native API wrapper | Interface or `expect` / `actual` |
| Large platform implementation | Interface + platform implementation |

There is no universal rule.

Architecture should drive the choice.

---

# 15. Android Resources

Android resources are naturally platform-specific.

Examples:

```text
res/
├── drawable/
├── mipmap/
├── layout/
├── values/
└── xml/
```

Depending on the project structure, Android-specific resources remain associated with the Android application/module.

Common code should not directly assume Android resource IDs unless the architecture intentionally exposes that capability through a platform boundary.

---

# 16. Resource Abstraction

Suppose common business logic needs a user-facing message.

Avoid putting Android resource IDs directly into common domain code:

```kotlin
R.string.invalid_user
```

Instead, the common layer can expose a semantic result:

```kotlin
sealed interface ValidationResult {

    data object Valid : ValidationResult

    data object InvalidUser : ValidationResult
}
```

Android UI can map that result to:

```text
R.string.invalid_user
```

iOS UI can map it to its own localized resource.

This keeps localization and resource ownership at the correct boundary.

---

# 17. Android Context

`Context` is one of the most common examples of Android-specific functionality.

If a common class requires:

```kotlin
Context
```

ask why.

Sometimes the class genuinely needs Android functionality.

But often the real requirement is something smaller:

```text
Read a value
Open a file
Access preferences
Get a resource
Start an operation
```

Those capabilities can frequently be represented by a smaller abstraction.

---

# 18. Avoid Context Leakage

Instead of:

```kotlin
class UserRepository(
    private val context: Context
)
```

consider:

```kotlin
interface UserStorage {
    suspend fun save(user: User)
    suspend fun get(): User?
}
```

Then:

```text
commonMain
    ↓
UserStorage
```

and:

```text
androidMain
    ↓
AndroidUserStorage
```

This creates a smaller platform boundary.

---

# 19. Android Storage Example

Common:

```kotlin
interface TokenStorage {

    suspend fun save(token: String)

    suspend fun get(): String?

    suspend fun clear()
}
```

Android:

```text
androidMain
    ↓
AndroidTokenStorage
    ↓
Android-specific secure storage
```

The repository remains common:

```kotlin
class AuthRepository(
    private val tokenStorage: TokenStorage
)
```

The repository does not know how Android stores the token.

---

# 20. Android Lifecycle

Lifecycle is another area where Android-specific code may be necessary.

For example:

```text
Activity
Fragment
LifecycleOwner
Process lifecycle
```

These concepts should not automatically leak into common business logic.

A common state machine can remain platform-independent:

```text
Idle
Loading
Success
Error
```

Android can connect lifecycle events to the shared state.

---

# 21. Android UI

If the project uses Android-only UI components, those components belong at the Android boundary.

For example:

```text
Android Views
RecyclerView
Fragment
Activity
Android-specific Compose integrations
```

can remain Android-specific.

If using Compose Multiplatform, more UI may move to common source sets, but Android-specific integrations can still remain in `androidMain`.

---

# 22. Android and Compose

Jetpack Compose itself is an Android UI technology.

Compose Multiplatform extends declarative UI concepts across platforms.

Therefore distinguish:

```text
Android-specific Compose integration
        ↓
androidMain

Multiplatform UI
        ↓
commonMain
```

Do not move code to `commonMain` simply because it uses Compose syntax.

The important question is whether the actual APIs are multiplatform-compatible.

---

# 23. Android-Specific Navigation

If navigation relies on Android-specific APIs, it belongs at the Android boundary.

For example:

```text
Intent
NavController
Activity navigation
Deep-link handling
```

may require Android-specific code.

A common layer can instead express navigation intent:

```kotlin
sealed interface NavigationAction {
    data object OpenProfile : NavigationAction
    data object OpenSettings : NavigationAction
}
```

Android can interpret that action.

---

# 24. Android Permissions

Permissions are inherently platform-specific.

Examples:

```text
Camera
Location
Bluetooth
Notifications
Contacts
Microphone
```

The common layer can define a capability:

```kotlin
interface PermissionChecker {
    fun isGranted(permission: Permission): Boolean
}
```

Android provides the implementation.

The common layer should not depend directly on Android permission APIs.

---

# 25. Android Notifications

Notifications are another natural `androidMain` responsibility.

Common code can produce:

```text
NotificationRequest
```

Android can convert it into an Android notification.

Conceptually:

```text
commonMain
    ↓
NotificationRequest
    ↓
androidMain
    ↓
Android NotificationManager
```

iOS can use its own implementation.

---

# 26. Android Background Work

Android background execution is platform-specific.

Examples include:

```text
WorkManager
Foreground services
AlarmManager
BroadcastReceiver
```

The business operation itself can be common.

The scheduling mechanism can remain Android-specific.

For example:

```text
commonMain
    ↓
SyncManager
```

Android:

```text
androidMain
    ↓
WorkManager
    ↓
SyncManager
```

This keeps scheduling separate from business behavior.

---

# 27. Android Database Integration

Suppose shared code defines:

```kotlin
interface UserLocalDataSource {
    suspend fun save(user: User)
    suspend fun get(id: String): User?
}
```

Android can provide a database implementation.

Depending on the architecture, this might use an Android-specific database library or an API shared across targets.

The key principle:

> **The common layer should depend on the capability, not the Android database framework.**

---

# 28. Android Networking

Modern KMP projects can share networking code using multiplatform-compatible libraries.

Therefore:

```text
HTTP request
JSON parsing
API models
Repository logic
```

may belong in:

```text
commonMain
```

not necessarily:

```text
androidMain
```

Android-specific networking code should be limited to capabilities that genuinely require Android.

---

# 29. Android Native SDKs

Sometimes an Android SDK has no multiplatform equivalent.

For example:

```text
Android-only analytics SDK
Android payment SDK
Android hardware SDK
Android enterprise SDK
```

A wrapper can live in:

```text
androidMain
```

Common code can interact through an abstraction:

```kotlin
interface Analytics {
    fun track(event: AnalyticsEvent)
}
```

Android:

```text
AndroidAnalytics
    ↓
Android SDK
```

This preserves the shared architecture.

---

# 30. Android Logging

Logging is a simple example.

Common:

```kotlin
interface Logger {
    fun debug(message: String)
    fun error(message: String)
}
```

Android:

```text
androidMain
    ↓
AndroidLogger
    ↓
Android logging infrastructure
```

The common code does not need to know which logging implementation Android uses.

---

# 31. Android File System

File-system APIs vary across platforms.

If shared logic needs:

```text
Read file
Write file
Delete file
```

define a capability:

```kotlin
interface FileStorage {
    suspend fun read(path: String): ByteArray
    suspend fun write(path: String, data: ByteArray)
}
```

Android implements it using Android-compatible storage APIs.

iOS implements it separately.

---

# 32. Android Secure APIs

Security is another area where platform APIs often matter.

Examples:

```text
Android Keystore
BiometricPrompt
Credential Manager
Platform cryptography
```

A common security contract can be shared:

```kotlin
interface CredentialStore {
    suspend fun save(...)
    suspend fun read(...)
}
```

The Android implementation belongs in:

```text
androidMain
```

The security policy remains common.

---

# 33. Platform-Specific Dependency Injection

A common dependency graph might look like:

```text
commonMain
    │
    ├── Repository
    ├── UseCase
    └── Logger
```

Android wiring:

```text
androidMain
    │
    ├── AndroidLogger
    ├── AndroidStorage
    └── AndroidNetworkMonitor
```

The Android application can assemble these dependencies.

This is often cleaner than allowing common classes to construct Android objects directly.

---

# 34. Android Dependency Injection Frameworks

A DI framework can be used to wire Android-specific dependencies.

The important architectural distinction is:

```text
Common code
→ depends on abstractions

Android composition root
→ creates Android implementations
```

Do not make the domain layer responsible for Android dependency construction.

---

# 35. Composition Root

The Android application is often the appropriate place to assemble platform dependencies.

Conceptually:

```text
Android Application
       │
       ├── AndroidStorage
       ├── AndroidLogger
       ├── AndroidNetworkMonitor
       │
       ▼
Shared components
```

This is a useful way to keep platform construction at the edge.

---

# 36. `androidMain` and Dependency Direction

A healthy dependency direction looks like:

```text
androidMain
     │
     ▼
commonMain abstractions
```

rather than:

```text
commonMain
     │
     ▼
androidMain implementation
```

The shared layer should not directly depend on Android implementation details.

Instead:

```text
commonMain defines contract
androidMain implements contract
```

---

# 37. Android-Specific Factories

Sometimes a factory is simpler than dependency injection.

Common:

```kotlin
interface PlatformClock {
    fun now(): Instant
}
```

Android:

```kotlin
fun createPlatformClock(): PlatformClock {
    return AndroidClock()
}
```

The Android-specific factory belongs at the Android boundary.

---

# 38. `actual` Implementations

With `expect`/`actual`, the relationship is more direct.

Common:

```kotlin
expect fun platformName(): String
```

Android:

```kotlin
actual fun platformName(): String = "Android"
```

The compiler connects the platform implementation to the common declaration.

This can be useful for small, well-defined platform capabilities.

---

# 39. Keep `expect` APIs Small

A useful principle is:

> **The smaller the expected API, the easier the platform implementations are to maintain.**

Prefer:

```kotlin
expect fun platformName(): String
```

over exposing a large platform abstraction with dozens of Android-specific concepts.

The common API should represent the capability, not the platform framework.

---

# 40. `androidMain` and Build Configuration

Android-specific source code is compiled using the Android target configuration.

A simplified conceptual configuration might look like:

```kotlin
kotlin {
    androidTarget()

    sourceSets {
        commonMain.dependencies {
            // Shared dependencies
        }

        androidMain.dependencies {
            // Android-specific dependencies
        }
    }
}
```

The exact syntax depends on the Kotlin, Android Gradle Plugin, and project configuration being used.

The important point is dependency ownership:

```text
commonMain dependency
→ available to common code

androidMain dependency
→ available to Android-specific code
```

---

# 41. Dependency Placement

Suppose an Android-only library is required:

```text
Android SDK library
```

Do not place it in:

```text
commonMain
```

unless it is actually compatible with the common compilation and all intended targets.

Place it in the appropriate Android source set.

This prevents accidental platform coupling.

---

# 42. Android-Specific Test Dependencies

The same principle applies to testing.

For example:

```text
commonTest
→ common testing APIs

androidUnitTest
→ Android/JVM-specific testing APIs
```

Do not make common tests depend on Android-only testing infrastructure.

---

# 43. `androidMain` and Android Unit Tests

Android-specific implementation code can be tested separately.

Conceptually:

```text
androidMain
    │
    ▼
Android implementation
    │
    ▼
androidUnitTest
```

For example:

```text
AndroidStorage
    ↓
androidUnitTest
    ↓
Storage behavior
```

This complements:

```text
commonMain
    ↓
commonTest
```

---

# 44. Common Test + Android Test

A mature feature may therefore have:

```text
commonMain
    ↓
commonTest

androidMain
    ↓
androidUnitTest
```

The first validates the shared contract.

The second validates Android-specific behavior.

---

# 45. Example: Secure Storage

### Common contract

```kotlin
interface SecureStorage {
    suspend fun save(key: String, value: String)
    suspend fun read(key: String): String?
}
```

### Common test

```text
Repository
    ↓
FakeSecureStorage
    ↓
commonTest
```

Tests:

```text
Token is saved
Token is retrieved
Missing token returns null
Logout clears token
```

### Android implementation

```text
AndroidSecureStorage
    ↓
Android security APIs
```

### Android tests

```text
AndroidSecureStorage
    ↓
androidUnitTest
```

Now the architecture verifies both levels.

---

# 46. Avoid Duplicating Business Logic

A dangerous pattern is:

```text
commonMain
    ↓
Business rule A

androidMain
    ↓
Business rule A again
```

If the rule is genuinely common, Android should consume the common implementation.

Use `androidMain` for:

```text
Android-specific behavior
```

not for:

```text
Android copy of shared behavior
```

---

# 47. A Duplication Smell

If you find:

```text
AndroidCalculator
IOSCalculator
CommonCalculator
```

and all three contain the same business rules, ask whether the calculator belongs in:

```text
commonMain
```

The goal of KMP is not to create three versions of the same architecture.

It is to share the parts that should be shared.

---

# 48. Android-Specific Adapter Pattern

Adapters are particularly useful at the platform boundary.

Example:

```text
Common abstraction
        │
        ▼
Android adapter
        │
        ▼
Android API
```

This isolates platform details.

For example:

```text
Analytics
    ↓
AndroidAnalyticsAdapter
    ↓
Android analytics SDK
```

---

# 49. Android Platform Services

A shared application may need:

```text
Network status
Battery status
Device information
App version
Locale
Timezone
Connectivity
```

Some of these can be shared through multiplatform libraries.

Others may require platform implementations.

When Android-specific APIs are necessary:

```text
androidMain
```

is the natural boundary.

---

# 50. Device Information

Common code may need a small abstraction:

```kotlin
data class DeviceInfo(
    val model: String,
    val osVersion: String
)
```

Android can construct it from Android APIs.

iOS can construct the same domain model from Apple APIs.

The model can remain common while collection remains platform-specific.

---

# 51. Platform Capability vs Platform Identity

This distinction is useful.

Avoid exposing:

```text
AndroidContext
AndroidActivity
AndroidView
```

when the application only needs:

```text
Can open URL
Can store data
Can notify user
Can check permission
```

Model the capability rather than the platform identity.

---

# 52. Android URL Handling

Common:

```kotlin
interface UrlLauncher {
    fun open(url: String): Boolean
}
```

Android:

```text
androidMain
    ↓
Intent.ACTION_VIEW
```

iOS:

```text
iosMain
    ↓
UIApplication / platform equivalent
```

The common layer only knows:

```text
UrlLauncher
```

---

# 53. Android Share Sheet

Sharing is platform-specific.

Common code might create:

```kotlin
data class ShareRequest(
    val text: String
)
```

Android:

```text
ShareRequest
    ↓
Android share Intent
```

iOS:

```text
ShareRequest
    ↓
iOS sharing mechanism
```

This is a clean platform boundary.

---

# 54. Android Biometric Authentication

Biometrics are a classic example.

Common:

```kotlin
interface BiometricAuthenticator {
    suspend fun authenticate(): AuthenticationResult
}
```

Android:

```text
androidMain
    ↓
BiometricPrompt
```

iOS:

```text
iosMain
    ↓
LocalAuthentication
```

The business flow can remain common:

```text
Login
   ↓
Authenticate
   ↓
Continue
```

---

# 55. Android Notifications and Shared Business Logic

Suppose a shared order system determines:

```text
Order shipped
```

The common layer can produce:

```text
OrderShipped event
```

Android-specific notification code can consume that event:

```text
androidMain
    ↓
Android notification
```

The business event does not need to know about notification channels or Android APIs.

---

# 56. Android Background Processing

A common sync use case:

```kotlin
class SyncOrdersUseCase(
    private val repository: OrderRepository
) {
    suspend operator fun invoke() {
        repository.sync()
    }
}
```

Android can schedule it through its background execution mechanism.

The architecture becomes:

```text
androidMain
    ↓
Android scheduler
    ↓
SyncOrdersUseCase
    ↓
commonMain
```

The scheduling policy is Android-specific.

The synchronization behavior can remain common.

---

# 57. Android App Lifecycle Integration

The Android application can connect:

```text
Application lifecycle
```

to:

```text
Shared lifecycle-aware service
```

without making the service depend directly on Android lifecycle classes.

For example:

```text
Android lifecycle
      ↓
Platform adapter
      ↓
Shared service
```

This keeps the shared service portable.

---

# 58. Android Main Thread

Some platform APIs require Android main-thread access.

That is an Android-specific concern.

Shared code should avoid assuming:

```text
Android Main Looper
```

unless the requirement is genuinely part of the platform implementation.

Use common coroutine abstractions where possible, and isolate Android dispatcher or thread requirements at the Android boundary.

---

# 59. Dispatchers and Platform Boundaries

A common abstraction might define a dispatcher provider when the architecture needs explicit scheduling.

Android can supply:

```text
Dispatchers.Main
Dispatchers.IO
```

The common layer can depend on an appropriate abstraction rather than directly constructing Android-specific threading infrastructure.

The exact design should reflect the application's concurrency model.

---

# 60. Android-specific Serialization or Parsing

If a parser is Android-only, it belongs in:

```text
androidMain
```

But do not assume parsing itself must be platform-specific.

If the selected serialization library is multiplatform-compatible:

```text
commonMain
```

may be the better location.

Always distinguish:

```text
Capability
```

from:

```text
Library availability
```

---

# 61. Library Compatibility Is Part of the Decision

Before placing a dependency into `commonMain`, verify that it supports the intended KMP targets.

If a library only supports Android:

```text
androidMain
```

is the appropriate source set.

If it supports all intended targets:

```text
commonMain
```

may be appropriate.

Do not move a library to common simply because the package name sounds generic.

---

# 62. Android-Specific Networking Interceptors

Suppose the shared networking stack needs an Android-only interceptor.

If the interceptor depends on:

```text
Android Context
Android security APIs
Android-specific SDK
```

it belongs at the Android boundary.

But generic:

```text
Authentication
Logging
Serialization
Request headers
```

may be candidates for common implementation when supported by the networking architecture.

---

# 63. Platform Boundary and Security

Security-sensitive Android APIs should generally remain isolated.

Examples:

```text
Keystore
BiometricPrompt
Credential APIs
Device attestation APIs
```

Common code can define security policy and contracts.

Android provides implementation details.

This prevents platform-specific security APIs from spreading through the domain layer.

---

# 64. `androidMain` as an Adapter Layer

A useful mental model is:

```text
                 commonMain
                     │
               Business intent
                     │
                     ▼
                Platform API
                     │
                     ▼
                 androidMain
                     │
                     ▼
                Android SDK
```

The platform source set acts as an adapter between shared application intent and native platform capabilities.

---

# 65. AndroidMain and Architecture Boundaries

A strong KMP architecture often resembles:

```text
┌────────────────────────────────────────────┐
│                commonMain                  │
│                                            │
│ Domain → Use Cases → Shared Data           │
│                                            │
└───────────────────┬────────────────────────┘
                    │
             Platform contracts
                    │
          ┌─────────┴─────────┐
          ▼                   ▼
     androidMain            iosMain
          │                   │
     Android SDK          Apple SDK
```

This structure minimizes accidental coupling.

---

# 66. What Happens When `androidMain` Gets Too Large?

A large `androidMain` is not automatically a problem.

Android applications can legitimately contain substantial native functionality.

The concern is different:

> **Is common functionality remaining common, or is Android-specific code taking over because the architecture has not identified the right boundaries?**

Review large Android-specific modules by capability.

---

# 67. Signs of Excessive Android Coupling

Watch for:

```text
Context passed through many layers
Android types in domain models
Android resources in business logic
Android SDK calls from common repositories
Android exceptions exposed through common APIs
Android lifecycle objects in shared use cases
```

These are signs that platform concerns may have leaked inward.

---

# 68. Signs of Healthy `androidMain`

A healthy Android source set often contains:

```text
Platform implementations
Native adapters
Android dependency wiring
Android resource integration
Android lifecycle integration
Android SDK wrappers
Android-specific UI
Android-specific tests/support
```

while the common layer contains:

```text
Domain behavior
Shared state
Shared networking
Shared persistence abstractions
Shared use cases
Shared models
```

---

# 69. AndroidMain and the Dependency Inversion Principle

The dependency inversion principle fits KMP naturally.

Common:

```text
interface Analytics
```

Android:

```text
AndroidAnalytics : Analytics
```

Common business logic depends on:

```text
Analytics
```

not:

```text
AndroidAnalytics
```

The platform implementation depends inward on the common abstraction.

---

# 70. AndroidMain and Clean Architecture

A common Clean Architecture boundary can be:

```text
Domain
   ↑
Data abstractions
   ↑
Platform implementations
```

Android-specific implementation details remain outside the core business rules.

For example:

```text
Domain
    ↓
UserRepository interface

androidMain
    ↓
AndroidUserRepository implementation
```

depending on the architecture and which parts of the repository are truly shared.

---

# 71. Not Every Repository Needs to Be Platform-Specific

This is important.

If a repository implementation uses:

```text
Ktor
SQLDelight
multiplatform serialization
shared models
shared cache
```

it may belong in:

```text
commonMain
```

Only its platform-specific dependencies should move to:

```text
androidMain
```

Do not classify an entire layer as Android-specific when only one dependency is platform-specific.

---

# 72. Split the Capability, Not the Whole Feature

Suppose:

```text
UserRepository
```

needs Android-only secure storage.

Do not necessarily move the entire repository to:

```text
androidMain
```

Instead:

```text
commonMain

UserRepository
     │
     ▼
SecureStorage abstraction

androidMain

AndroidSecureStorage
```

Now the repository remains shared.

---

# 73. This Is One of the Most Important KMP Skills

The difference between:

```text
"I cannot share this feature."
```

and:

```text
"I cannot share this one platform capability."
```

is huge.

The second statement often leads to a better architecture.

For example:

```text
Feature
 ├── Business logic → commonMain
 ├── State → commonMain
 ├── Repository → commonMain
 └── Native storage → androidMain
```

That is much more reusable.

---

# 74. AndroidMain and Source Set Visibility

Code in:

```text
androidMain
```

is Android-specific.

It should not become an accidental dependency for:

```text
commonMain
```

The source-set model exists partly to enforce these boundaries during compilation.

This is one reason KMP source sets are more than organizational folders.

---

# 75. Source Sets Are Compilation Boundaries

Think of:

```text
commonMain
androidMain
iosMain
```

as different compilation contexts.

They determine:

```text
Which APIs are available
Which dependencies can be used
Which code can be compiled
Which platforms receive the implementation
```

This is why source-set placement matters architecturally.

---

# 76. `androidMain` and Platform Availability

A class that uses:

```kotlin
android.content.Context
```

cannot simply become common code.

The compiler helps reveal this boundary.

Instead of fighting the compiler:

```text
"How can I hide this Android dependency?"
```

ask:

```text
"Should this capability remain Android-specific?"
```

Sometimes the correct answer is yes.

---

# 77. Compiler Errors Can Be Architectural Feedback

KMP compilation errors can be useful signals.

If common code suddenly reports:

```text
Unresolved reference: android.*
```

the problem is not merely an import issue.

It may indicate:

```text
Platform dependency leaked into commonMain.
```

The correct fix may be architectural rather than syntactic.

---

# 78. AndroidMain and Testing Strategy

For an Android-specific implementation:

```text
androidMain
    ↓
Android-specific behavior
    ↓
androidUnitTest
```

For shared behavior:

```text
commonMain
    ↓
Shared behavior
    ↓
commonTest
```

This gives two complementary test layers.

---

# 79. Example: Android Permission Handler

Common:

```kotlin
interface CameraPermission {

    fun isGranted(): Boolean

    suspend fun request(): Boolean
}
```

Android:

```text
androidMain
    ↓
AndroidCameraPermission
    ↓
Android permission APIs
```

Common test:

```text
Business behavior when permission is granted/denied
```

Android test:

```text
Actual Android permission integration
```

The distinction keeps testing focused.

---

# 80. AndroidMain and UI Events

An Android UI event may start at:

```text
Activity / Compose UI
```

then call:

```text
Shared ViewModel / Use Case
```

The flow can be:

```text
Android UI
    ↓
Shared presentation
    ↓
Shared domain
```

The direction should generally move toward shared behavior, not expose Android UI types throughout the common layer.

---

# 81. Android Activity Results

Activity Result APIs are Android-specific.

Do not expose:

```text
ActivityResultLauncher
```

deep inside common business logic.

Instead, common code can express:

```text
RequestCamera
RequestDocument
OpenSettings
```

and Android UI handles the platform operation.

---

# 82. Android Deep Links

Deep-link parsing can sometimes be common.

For example:

```text
myapp://profile/100
```

The route parsing logic may be common.

But Android-specific intent registration and activity integration belong in:

```text
androidMain
```

This is another example of separating:

```text
Business interpretation
```

from:

```text
Platform integration
```

---

# 83. Android App Configuration

Configuration values such as:

```text
API endpoint
Build environment
Feature flags
```

may be shared conceptually.

But Android-specific build configuration can remain at the Android application/build boundary.

Do not expose Android build APIs directly to common business logic.

---

# 84. Android Build Variants

Android has concepts such as:

```text
debug
release
flavors
build types
```

These are Android-specific build concerns.

KMP shared code should not become tightly coupled to Android build variants unless the requirement genuinely concerns platform-specific behavior.

Where possible, expose a clean configuration abstraction.

---

# 85. Android Logging Example

A shared logger:

```kotlin
interface Logger {
    fun log(message: String)
}
```

Android implementation:

```kotlin
class AndroidLogger : Logger {

    override fun log(message: String) {
        // Android-specific logging implementation.
    }
}
```

Common code:

```kotlin
class SyncManager(
    private val logger: Logger
) {
    fun sync() {
        logger.log("Sync started")
    }
}
```

The shared code remains unaware of Android logging.

---

# 86. Android Analytics Example

Common event:

```kotlin
data class AnalyticsEvent(
    val name: String,
    val parameters: Map<String, String>
)
```

Common:

```kotlin
interface Analytics {
    fun track(event: AnalyticsEvent)
}
```

Android:

```text
AndroidAnalytics
    ↓
Android analytics provider
```

iOS:

```text
IosAnalytics
    ↓
iOS analytics provider
```

The event model can remain common.

---

# 87. Android Platform Factory

When using interfaces, a factory can be used to construct the platform implementation.

Conceptually:

```text
commonMain
    ↓
Analytics interface

androidMain
    ↓
createAnalytics()
    ↓
AndroidAnalytics
```

The Android application can call the factory during dependency setup.

---

# 88. Keep Platform Construction Near the Edge

A good rule:

```text
Create Android objects at the Android edge.
Use abstractions in the shared core.
```

Avoid:

```kotlin
class UserRepository {
    private val context = ...
}
```

Prefer:

```kotlin
class UserRepository(
    private val storage: UserStorage
)
```

and construct:

```text
AndroidUserStorage
```

outside the repository.

---

# 89. AndroidMain and Long-Term Portability

Code in `androidMain` is intentionally not portable.

That is okay.

The goal is not:

```text
100% common code
```

The goal is:

```text
Maximum sensible sharing
+
Clear platform boundaries
```

Some platform-specific code is necessary and often desirable.

---

# 90. Platform-Specific Code Is Not Bad

KMP does not require eliminating native code.

Native APIs often provide:

```text
Best platform integration
Best performance for specific workloads
Access to unique capabilities
Platform-standard UX
Existing SDK compatibility
```

The architectural goal is to use native code deliberately.

---

# 91. Native Where Necessary, Shared Where Valuable

A useful principle:

```text
Shared business logic
        +
Native platform capabilities
        =
Strong KMP architecture
```

Not:

```text
Everything common
```

and not:

```text
Everything native
```

---

# 92. A Practical `androidMain` Checklist

When adding code to `androidMain`, ask:

```text
[ ] Does this code require an Android API?
[ ] Is there a genuine Android-specific capability?
[ ] Could only the dependency be platform-specific?
[ ] Can the common layer depend on an abstraction instead?
[ ] Should the implementation be injected?
[ ] Would expect/actual be simpler?
[ ] Is Android-specific construction kept at the edge?
[ ] Is the shared layer free from Android types?
[ ] Does the implementation need Android-specific tests?
[ ] Is the platform boundary documented?
```

---

# 93. CommonMain vs AndroidMain

| Concern | `commonMain` | `androidMain` |
|---|:---:|:---:|
| Business rules | ✅ | ❌ |
| Domain models | ✅ | ❌ |
| Shared use cases | ✅ | ❌ |
| Shared state | ✅ | ❌ |
| Multiplatform networking | ✅ | ❌ |
| Android `Context` | ❌ | ✅ |
| Android SDK APIs | ❌ | ✅ |
| Android storage implementation | ❌ | ✅ |
| Android permissions | ❌ | ✅ |
| Android background scheduling | ❌ | ✅ |
| Android native SDKs | ❌ | ✅ |
| Android-specific resource integration | ❌ | ✅ |
| Android-specific UI | ❌ | ✅ |
| Android platform adapters | ❌ | ✅ |

---

# 94. `androidMain` Is the Edge

A strong KMP architecture can be visualized as:

```text
                  SHARED CORE
        ┌──────────────────────────┐
        │ commonMain               │
        │                          │
        │ Domain                   │
        │ Business Rules           │
        │ Use Cases                │
        │ Shared State             │
        │ Shared Data Logic        │
        └────────────┬─────────────┘
                     │
              Platform Boundary
                     │
        ┌────────────┴─────────────┐
        ▼                          ▼
   androidMain                  iosMain
        │                          │
 Android APIs                 Apple APIs
```

The center remains portable.

The edges embrace the platform.

---

# 95. A Complete Example

Consider an application that needs device information.

### Common contract

```kotlin
data class DeviceInfo(
    val model: String,
    val osVersion: String
)

interface DeviceInfoProvider {
    fun get(): DeviceInfo
}
```

### Android implementation

```kotlin
class AndroidDeviceInfoProvider : DeviceInfoProvider {

    override fun get(): DeviceInfo {
        return DeviceInfo(
            model = "Android Device",
            osVersion = "Android"
        )
    }
}
```

The real implementation would use appropriate Android APIs.

### Shared use case

```kotlin
class GetDeviceInfoUseCase(
    private val provider: DeviceInfoProvider
) {
    operator fun invoke(): DeviceInfo {
        return provider.get()
    }
}
```

The use case remains common.

---

# 96. Another Example: URL Launcher

Common:

```kotlin
interface UrlLauncher {
    fun open(url: String): Boolean
}
```

Android:

```text
AndroidUrlLauncher
       ↓
Intent
       ↓
Android browser/app
```

iOS:

```text
IosUrlLauncher
       ↓
iOS URL handling
       ↓
Browser/app
```

Common business logic does not know how the URL is opened.

---

# 97. Another Example: Connectivity

Common:

```kotlin
interface Connectivity {
    fun isOnline(): Boolean
}
```

Android:

```text
androidMain
    ↓
Android connectivity APIs
```

iOS:

```text
iosMain
    ↓
Apple networking APIs
```

The shared repository can use:

```kotlin
connectivity.isOnline()
```

without knowing either platform.

---

# 98. AndroidMain and Feature Modules

Large KMP applications may contain multiple modules.

For example:

```text
:core
:feature-auth
:feature-orders
:feature-profile
```

Each KMP module can have its own:

```text
commonMain
commonTest
androidMain
androidUnitTest
```

This allows platform boundaries to remain local to each feature.

---

# 99. Avoid One Giant `androidMain`

Do not move every Android-specific implementation into a single giant module if the project architecture naturally supports feature boundaries.

Prefer:

```text
feature-auth
    └── androidMain

feature-orders
    └── androidMain

feature-profile
    └── androidMain
```

when the platform-specific concerns belong to individual features.

---

# 100. Final Mental Model

The most useful way to remember `androidMain` is:

```text
                  commonMain
                      │
            "What the app needs"
                      │
                      ▼
             Platform abstraction
                      │
                      ▼
                  androidMain
                      │
             "How Android does it"
                      │
                      ▼
                 Android APIs
```

`androidMain` is not a failure to share.

It is the place where intentional Android-specific behavior lives.

> **Good KMP architecture does not eliminate native Android code. It puts native Android code exactly where it belongs.**

---

# Chapter Takeaways

> [!TIP]
> **`androidMain` is the Android boundary of a KMP module. Keep Android APIs, native SDKs, platform services, and Android-specific implementations here while keeping business behavior and reusable abstractions in `commonMain`.**

Remember:

1. `androidMain` is the Android-specific source set.
2. `commonMain` contains shared production behavior.
3. `androidMain` contains Android-specific production behavior.
4. `androidMain` is not a second Android application.
5. Its purpose is to provide Android-specific implementations.
6. Android framework APIs naturally belong at the Android boundary.
7. `Context` is Android-specific.
8. Android lifecycle APIs are Android-specific.
9. Android permissions are Android-specific.
10. Android background scheduling is Android-specific.
11. Android native SDK integrations belong in the Android layer.
12. Android-specific storage implementations can live in `androidMain`.
13. Android-specific security APIs can live in `androidMain`.
14. Android-specific resource integration belongs at the Android boundary.
15. Business rules should generally remain in `commonMain`.
16. Domain models should generally remain in `commonMain`.
17. Shared use cases should generally remain in `commonMain`.
18. Shared networking can remain common when the selected stack supports the intended targets.
19. Shared repositories do not automatically need to be Android-specific.
20. Split platform-specific capabilities rather than moving entire features unnecessarily.
21. Interfaces can isolate Android implementations.
22. Dependency injection can connect common abstractions to Android implementations.
23. `expect`/`actual` is another mechanism for platform-specific implementations.
24. `expect`/`actual` should not automatically replace interfaces.
25. Keep platform APIs out of shared business logic.
26. Model capabilities instead of exposing platform objects whenever possible.
27. Avoid passing `Context` deep through shared layers.
28. Keep Android dependency construction near the Android composition root.
29. Use Android-specific dependencies in the appropriate Android source set.
30. Keep Android-only test dependencies out of `commonTest`.
31. Android-specific implementations should have Android-specific tests where appropriate.
32. Common behavior should be validated through `commonTest`.
33. Platform integrations should be validated through platform tests.
34. Android UI can remain Android-specific when the application requires it.
35. Compose Multiplatform can move UI into common source sets, but Android-specific integrations can still remain in `androidMain`.
36. Android navigation APIs should not leak into common business logic.
37. Android permission APIs should not leak into common business logic.
38. Android notification APIs should remain platform-specific.
39. Android background execution mechanisms should remain platform-specific.
40. Android build variants are Android-specific build concerns.
41. Source sets are compilation boundaries, not merely folders.
42. KMP compiler errors can reveal unwanted platform coupling.
43. A large `androidMain` is not automatically bad.
44. Excessive Android types in common layers are a warning sign.
45. The objective is not 100% common code.
46. The objective is maximum sensible sharing with clear boundaries.
47. Native platform code is valuable when the platform provides unique capabilities.
48. Common abstractions should express application capabilities rather than platform frameworks.
49. `androidMain` acts as an adapter between shared application intent and Android APIs.
50. **Good KMP architecture does not remove Android. It gives Android a well-defined place to live.**

---

# Final Thought

Kotlin Multiplatform becomes much easier to reason about when the source sets are treated as architectural boundaries rather than directories.

The mental model is:

```text
                         commonMain
                             │
                    Shared application logic
                             │
                      Platform contract
                             │
                    ┌────────┴────────┐
                    ▼                 ▼
               androidMain         iosMain
                    │                 │
               Android APIs       Apple APIs
```

The common layer answers:

> **What does the application need?**

The Android layer answers:

> **How does Android provide it?**

That separation is one of the foundations of scalable KMP architecture.

> **Share the behavior. Isolate the platform. Let Android remain native where Android is the right tool.**
