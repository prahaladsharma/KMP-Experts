# Chapter 8 — `expect` / `actual`

## Part 3 — Real Examples

> **The best way to understand `expect` / `actual` is to see where a real multiplatform application actually needs platform-specific behavior.**

The previous parts covered why `expect` / `actual` exists and how the compiler resolves the relationship.

This part focuses on practical examples.

The goal is not to use `expect` / `actual` everywhere. The goal is to recognize situations where a shared API makes sense while the implementation must remain platform-specific.

---

## 1. Example — Identifying the Platform

A simple first example is exposing the current platform.

### `commonMain`

```kotlin
expect fun platformName(): String
```

### `androidMain`

```kotlin
actual fun platformName(): String = "Android"
```

### `iosMain`

```kotlin
actual fun platformName(): String = "iOS"
```

Common code:

```kotlin
fun greeting(): String {
    return "Running on ${platformName()}"
}
```

The business logic remains common.

The platform-specific implementation stays inside the platform source set.

---

## 2. Example — Platform Information Class

The same idea can be represented using a class.

### `commonMain`

```kotlin
expect class Platform {
    val name: String
}
```

### `androidMain`

```kotlin
actual class Platform {
    actual val name: String = "Android"
}
```

### `iosMain`

```kotlin
actual class Platform {
    actual val name: String = "iOS"
}
```

Usage:

```kotlin
class PlatformInfoUseCase(
    private val platform: Platform
) {
    fun execute(): String {
        return platform.name
    }
}
```

This is useful when the platform capability grows beyond a single function.

---

## 3. Example — Current Time

Time is a common source of platform-specific concerns.

A shared feature might need:

```text
Current timestamp
```

The business logic should not need to know how each platform obtains it.

### `commonMain`

```kotlin
expect fun currentTimeMillis(): Long
```

### `androidMain`

```kotlin
actual fun currentTimeMillis(): Long {
    return System.currentTimeMillis()
}
```

### `iosMain`

```kotlin
import platform.Foundation.NSDate

actual fun currentTimeMillis(): Long {
    return (NSDate().timeIntervalSince1970 * 1000).toLong()
}
```

Common code:

```kotlin
fun createTimestamp(): Long {
    return currentTimeMillis()
}
```

The common layer only cares about the capability:

```text
Give me the current time.
```

---

## 4. Example — Opening a URL

Opening a web page is another platform-dependent operation.

The common application may only need:

```text
Open this URL.
```

### `commonMain`

```kotlin
expect fun openUrl(url: String)
```

### `androidMain`

```kotlin
actual fun openUrl(url: String) {
    // Android-specific implementation
}
```

### `iosMain`

```kotlin
actual fun openUrl(url: String) {
    // iOS-specific implementation
}
```

The important architectural boundary is:

```text
commonMain
    │
    └── openUrl(url)
            │
            ├── Android actual
            └── iOS actual
```

The shared code does not need to know about:

```text
Android Intent
UIApplication
UIViewController
```

unless those details are required by the actual implementations.

---

## 5. Example — Secure Storage

Security-sensitive storage is a strong example of a platform boundary.

A shared application may need:

```text
Save token
Read token
Remove token
```

But the underlying secure-storage mechanisms differ by platform.

### `commonMain`

```kotlin
expect class SecureStorage {
    fun put(key: String, value: String)
    fun get(key: String): String?
    fun remove(key: String)
}
```

### Android

```kotlin
actual class SecureStorage {
    actual fun put(key: String, value: String) {
        // Android secure-storage implementation
    }

    actual fun get(key: String): String? {
        // Android secure-storage implementation
        return null
    }

    actual fun remove(key: String) {
        // Android secure-storage implementation
    }
}
```

### iOS

```kotlin
actual class SecureStorage {
    actual fun put(key: String, value: String) {
        // iOS secure-storage implementation
    }

    actual fun get(key: String): String? {
        // iOS secure-storage implementation
        return null
    }

    actual fun remove(key: String) {
        // iOS secure-storage implementation
    }
}
```

The shared authentication flow can then depend on the capability instead of platform APIs.

> [!IMPORTANT]
> For security-sensitive functionality, use the platform's supported security mechanisms and follow current platform security guidance. Do not invent custom cryptography or insecure storage simply to make an example compile.

---

## 6. Example — Device Information

A KMP application might need a small amount of device information:

```text
Platform name
Operating-system version
Device model
```

A common abstraction could be:

```kotlin
expect class DeviceInfo {
    val platform: String
    val osVersion: String
}
```

Android:

```kotlin
actual class DeviceInfo {
    actual val platform: String = "Android"
    actual val osVersion: String = "Android version"
}
```

iOS:

```kotlin
actual class DeviceInfo {
    actual val platform: String = "iOS"
    actual val osVersion: String = "iOS version"
}
```

The important point is not the exact API used to obtain the values.

The important point is that common code depends on a small capability.

---

## 7. Example — UUID Generation

Suppose a shared repository needs a unique identifier.

### `commonMain`

```kotlin
expect fun generateUuid(): String
```

Platform implementations can use appropriate platform-supported UUID facilities.

### Android

```kotlin
actual fun generateUuid(): String {
    return java.util.UUID.randomUUID().toString()
}
```

### iOS

```kotlin
import platform.Foundation.NSUUID

actual fun generateUuid(): String {
    return NSUUID().UUIDString
}
```

Common code:

```kotlin
data class Request(
    val id: String = generateUuid()
)
```

The domain model does not need to know which platform generated the identifier.

---

## 8. Example — Logging

Logging is another practical case.

The common application may want:

```kotlin
expect object AppLogger {
    fun debug(message: String)
    fun error(message: String, throwable: Throwable? = null)
}
```

Android can connect this to its preferred logging mechanism.

iOS can connect it to its appropriate logging mechanism.

Common code can then write:

```kotlin
AppLogger.debug("Loading user profile")
```

This gives the shared layer a consistent logging API while keeping the output mechanism platform-specific.

---

## 9. Example — File Storage

Suppose a shared feature needs to cache a file.

The common layer might define:

```kotlin
expect class FileStorage {
    fun write(name: String, content: ByteArray)
    fun read(name: String): ByteArray?
}
```

Android can use an appropriate Android storage location.

iOS can use an appropriate location provided by the Apple platform.

The common repository can simply call:

```kotlin
fileStorage.write("profile.json", bytes)
```

The repository should not need to understand:

```text
Android Context
iOS Foundation APIs
platform-specific paths
```

---

## 10. Example — Network Connectivity

A shared application may need to know whether network connectivity is available.

### Common API

```kotlin
expect class NetworkMonitor {
    fun isConnected(): Boolean
}
```

Android can use Android networking APIs.

iOS can use Apple's networking APIs.

The shared application can then make decisions such as:

```kotlin
if (networkMonitor.isConnected()) {
    repository.refresh()
}
```

The repository does not need to know how connectivity was detected.

---

## 11. Example — Clipboard

Clipboard access is naturally platform-specific.

Common code might request:

```kotlin
expect class Clipboard {
    fun copy(text: String)
    fun paste(): String?
}
```

Android provides its implementation.

iOS provides its implementation.

The common presentation logic can remain:

```kotlin
fun copyInviteCode(code: String) {
    clipboard.copy(code)
}
```

The UI layer does not need to know which platform clipboard API is being used.

---

## 12. Example — Secure Authentication Flow

Consider a shared authentication use case.

```kotlin
class LoginUseCase(
    private val repository: AuthRepository,
    private val secureStorage: SecureStorage
) {
    suspend fun login(
        username: String,
        password: String
    ): Boolean {
        val token = repository.login(username, password)

        secureStorage.put("auth_token", token)

        return true
    }
}
```

The use case is common.

The secure-storage implementation is platform-specific.

```text
                 LoginUseCase
                      │
          ┌───────────┴───────────┐
          ▼                       ▼
     AuthRepository          SecureStorage
          │                       │
      common logic          expect / actual
                                  │
                         ┌────────┴────────┐
                         ▼                 ▼
                      Android             iOS
                       actual             actual
```

This is a realistic architectural use of platform boundaries.

---

## 13. Example — Repository With Platform Capability

Imagine an offline-first repository.

```kotlin
class ProductRepository(
    private val api: ProductApi,
    private val storage: ProductStorage,
    private val networkMonitor: NetworkMonitor
) {

    suspend fun getProducts(): List<Product> {
        return if (networkMonitor.isConnected()) {
            val products = api.getProducts()
            storage.save(products)
            products
        } else {
            storage.getProducts()
        }
    }
}
```

The repository remains common.

Only:

```text
NetworkMonitor
```

is platform-specific.

This is a strong KMP pattern:

```text
Business decision → commonMain
Platform capability → expect / actual
```

---

## 14. Example — Application Configuration

Some applications need access to platform-specific configuration.

For example:

```kotlin
expect object AppEnvironment {
    val versionName: String
    val buildType: String
}
```

The Android implementation can obtain values from the Android application configuration.

The iOS implementation can obtain corresponding values from the application bundle.

Common analytics or diagnostics code can then use:

```kotlin
AppEnvironment.versionName
```

without importing platform APIs.

---

## 15. Example — Background Work

Background execution is heavily platform-dependent.

Instead of exposing platform scheduling APIs directly to common code, define a small capability.

For example:

```kotlin
expect class BackgroundTaskScheduler {
    fun schedule()
}
```

Android and iOS can implement the capability according to their own lifecycle and background-execution rules.

The common layer expresses:

```text
Schedule this work.
```

The platform decides:

```text
How and when this is legally and technically possible.
```

This is an important distinction.

> [!NOTE]
> Platform-specific background execution policies can change over time. Treat the `actual` implementation as the place where current platform rules are respected.

---

## 16. Example — Biometric Authentication

Biometric authentication is another clear platform boundary.

The shared application may need:

```text
Authenticate the user.
```

A small abstraction could be:

```kotlin
expect class BiometricAuthenticator {
    suspend fun authenticate(): Boolean
}
```

Android and iOS can provide their respective implementations.

The shared authentication flow can remain:

```kotlin
suspend fun confirmSensitiveAction(): Boolean {
    return biometricAuthenticator.authenticate()
}
```

The common layer does not need to know how biometric APIs work.

---

## 17. Example — Camera Capability

A camera is fundamentally platform-specific.

Common code might define a small capability:

```kotlin
expect class CameraController {
    fun start()
    fun stop()
}
```

The Android implementation can integrate with Android camera APIs.

The iOS implementation can integrate with Apple's camera frameworks.

The shared business flow can decide:

```text
Start document scanning.
```

while the platform layer handles:

```text
Camera permissions
Camera lifecycle
Preview
Capture APIs
```

This prevents platform UI and hardware APIs from leaking into business logic.

---

## 18. Example — Platform Permissions

Permissions are another area where platform behavior differs.

Instead of putting platform permission APIs into common business code, define an abstraction around the capability you actually need.

For example:

```kotlin
expect class PermissionManager {
    suspend fun requestCameraPermission(): Boolean
}
```

Android implements it using Android permission APIs.

iOS implements it using the appropriate Apple authorization mechanism.

The shared workflow can react to the result.

---

## 19. Example — Platform Notifications

A shared application may want to request:

```text
Register for notifications.
```

A platform abstraction could expose:

```kotlin
expect class NotificationManager {
    fun requestPermission()
}
```

The platform implementation owns:

```text
Permission APIs
Registration APIs
Platform-specific configuration
```

The common application owns:

```text
Business rules
Notification-related state
User preferences
```

Again:

```text
Common intent
      ↓
Platform capability
```

---

## 20. Example — Analytics Metadata

Suppose the analytics system needs:

```text
Platform
App version
Device information
```

Instead of making analytics code depend on Android and iOS APIs:

```kotlin
expect class AnalyticsContext {
    val platform: String
    val appVersion: String
}
```

Then:

```kotlin
class AnalyticsService(
    private val context: AnalyticsContext
) {
    fun track(event: String) {
        // common analytics behavior
    }
}
```

The platform implementation supplies the environment-specific metadata.

---

# 21. Example — Time Abstraction for Testing

The time example becomes even more useful when testing is considered.

Suppose:

```kotlin
expect fun currentTimeMillis(): Long
```

is used directly everywhere.

Testing time-dependent business logic may become difficult.

A better architecture may be:

```text
Clock interface
      │
      ├── Production implementation
      └── Test implementation
```

This illustrates an important point:

> **Not every platform dependency should automatically become an `expect` function.**

If dependency injection gives you better testability and architecture, an interface may be the better abstraction.

---

# 22. Example — Interface + `actual`

The two approaches can also be combined.

For example:

### Common

```kotlin
interface SecureStorage {
    fun put(key: String, value: String)
    fun get(key: String): String?
}
```

Then an expected factory can provide the platform implementation:

```kotlin
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

This can provide:

```text
expect / actual
        +
interface
        +
dependency injection
```

when the architecture benefits from all three.

---

# 23. Example — Factory Pattern

Another useful pattern is:

```kotlin
expect object PlatformFactory {
    fun createStorage(): Storage
}
```

The common layer uses:

```kotlin
val storage = PlatformFactory.createStorage()
```

Android supplies:

```kotlin
actual object PlatformFactory {
    actual fun createStorage(): Storage {
        return AndroidStorage()
    }
}
```

iOS supplies:

```kotlin
actual object PlatformFactory {
    actual fun createStorage(): Storage {
        return IosStorage()
    }
}
```

This keeps platform construction out of common business logic.

---

# 24. Example — Platform-Specific Database Driver

A shared database layer may use a common database abstraction while the driver or low-level integration differs by target.

A typical architecture might look like:

```text
commonMain
    │
    ├── Database interface
    ├── Repository
    └── Data models
             │
             ▼
      platform-specific driver
             │
       ┌─────┴─────┐
       ▼           ▼
    Android       iOS
```

Depending on the library and architecture, the platform-specific piece may be provided through library configuration, source sets, interfaces, or `expect` / `actual`.

The important lesson is:

> Use the smallest mechanism required by the dependency and architecture.

---

# 25. Example — Platform-Specific Serialization Configuration

Most serialization logic can remain common.

For example:

```kotlin
@Serializable
data class User(
    val id: String,
    val name: String
)
```

No `expect` / `actual` is needed simply because the application targets multiple platforms.

Only introduce a platform boundary if some serialization-related capability genuinely depends on the platform.

This is a critical distinction.

---

# 26. Example — Platform-Specific HTTP Behavior

Most networking code can remain common:

```kotlin
class UserRepository(
    private val api: UserApi
)
```

If the HTTP client library already provides multiplatform implementations, you may not need your own `expect` / `actual`.

For example:

```text
common networking API
        ↓
multiplatform library
        ↓
platform-specific engine
```

In such a case, the library already solved the platform boundary.

You should avoid creating another abstraction unless your application actually needs it.

---

# 27. Example — Platform-Specific UI

UI is an area where developers often overuse `expect` / `actual`.

Suppose an Android screen and an iOS screen have different UI frameworks.

You do not necessarily need:

```kotlin
expect class LoginScreen
```

A better architecture may be:

```text
common
 ├── LoginState
 ├── LoginAction
 ├── LoginViewModel / presentation logic
 └── LoginUseCase

Android
 └── Android UI

iOS
 └── iOS UI
```

The shared state and business logic remain common.

The platform UI remains native where appropriate.

---

# 28. Example — Shared ViewModel Logic

Imagine:

```kotlin
class LoginViewModel(
    private val loginUseCase: LoginUseCase
) {
    // shared state and actions
}
```

The Android UI can observe the shared state.

The iOS UI can consume the same business state through an appropriate integration layer.

No `expect` / `actual` is required simply because the UI is different.

This demonstrates an important rule:

> **Different UI does not automatically mean `expect` / `actual`.**

---

# 29. Example — Platform Lifecycle

Android and iOS have different lifecycle models.

Avoid trying to create a giant abstraction such as:

```kotlin
expect class PlatformLifecycle {
    fun onCreate()
    fun onStart()
    fun onResume()
    fun onPause()
    fun onStop()
    fun onDestroy()
}
```

This can force one platform's lifecycle model onto another.

Instead, share the business state and lifecycle-independent logic where possible.

Use platform-specific lifecycle integration at the edges.

---

# 30. Example — Push Notifications

Push notification infrastructure differs significantly across platforms.

The common layer may manage:

```text
Notification token
User preference
Notification-related domain state
```

while platform layers manage:

```text
Registration
Permission
Platform callbacks
Token retrieval
```

A small expected capability can be useful if the common application genuinely needs to request or retrieve something platform-specific.

---

# 31. Example — Secure Authentication Token Flow

A realistic architecture might be:

```text
                    Login Screen
                         │
                         ▼
                  LoginViewModel
                         │
                         ▼
                    LoginUseCase
                         │
                 ┌───────┴────────┐
                 ▼                ▼
          AuthRepository     SecureStorage
                 │                │
              common          expect/actual
                                  │
                         ┌────────┴────────┐
                         ▼                 ▼
                      Android             iOS
                       actual             actual
```

This is a strong example because:

- UI can remain platform-specific.
- Authentication business logic can remain common.
- Repository logic can remain common.
- Secure storage remains platform-specific.
- The boundary is explicit.

---

# 32. Example — Platform Date Formatting

Date and time formatting can sometimes be platform-dependent.

A common application might define:

```kotlin
expect fun formatDate(
    timestamp: Long
): String
```

Android and iOS can provide their implementations.

However, before creating this abstraction, check whether a multiplatform date/time library already provides the required functionality.

This leads to an important engineering principle:

> **Do not create `expect` / `actual` merely because something sounds platform-specific. First check whether the shared ecosystem already provides a multiplatform abstraction.**

---

# 33. Example — Platform File Paths

Suppose an application needs an application-specific file location.

Common code may define:

```kotlin
expect fun appDataDirectory(): String
```

Android and iOS can resolve the appropriate location.

The common file-storage layer can then use that capability.

However, if a multiplatform library already provides a portable filesystem abstraction, prefer the library when it fits the project.

---

# 34. Example — Platform Logging With Common API

A useful pattern is:

```kotlin
expect object Logger {
    fun info(message: String)
    fun warning(message: String)
    fun error(message: String, throwable: Throwable? = null)
}
```

Common code:

```kotlin
class SyncService {
    suspend fun sync() {
        Logger.info("Sync started")

        try {
            // shared work
        } catch (t: Throwable) {
            Logger.error("Sync failed", t)
        }
    }
}
```

The platform implementation decides how logs are emitted.

This keeps logging calls consistent across shared code.

---

# 35. Example — Feature Flags

Feature-flag evaluation itself should usually remain common.

For example:

```kotlin
class FeatureFlagService {
    fun isEnabled(flag: String): Boolean {
        // common evaluation logic
    }
}
```

But if retrieving some platform-specific environment information is required, that small capability could be isolated behind an expected declaration.

Again:

```text
Business rule → common
Platform environment → platform boundary
```

---

# 36. Example — Device Locale

A shared application might need the device's current locale.

### Common

```kotlin
expect fun currentLocale(): String
```

Android and iOS provide their respective implementations.

Common code can then use:

```kotlin
val locale = currentLocale()
```

But if the application requires sophisticated locale handling, first consider a multiplatform library instead of implementing locale behavior separately.

---

# 37. Example — Platform Battery Information

A monitoring feature may need:

```kotlin
expect fun batteryLevel(): Int
```

Android can retrieve the value through Android APIs.

iOS can retrieve it through iOS APIs.

Common monitoring logic can remain:

```kotlin
fun shouldShowLowBatteryWarning(): Boolean {
    return batteryLevel() < 20
}
```

The business rule is common.

The device API is platform-specific.

---

# 38. Example — Platform Vibration

A shared interaction layer might request:

```kotlin
expect fun vibrate(durationMillis: Long)
```

Android and iOS can implement the behavior using their respective APIs.

The common code only expresses:

```text
Provide haptic feedback.
```

This can be useful for a shared interaction or feedback system.

---

# 39. Example — Platform Sharing

The common application may want:

```kotlin
expect fun shareText(text: String)
```

Android:

```text
Platform sharing mechanism
```

iOS:

```text
Platform sharing mechanism
```

Common feature:

```kotlin
fun shareInvite(code: String) {
    shareText("Join using code: $code")
}
```

Again, the common layer owns the intent.

---

# 40. Choosing the Right Abstraction

When you encounter a platform difference, use this decision path:

```text
Is the behavior completely common?
        │
       Yes
        ▼
   Keep it in commonMain

        No
        │
        ▼
Does a multiplatform library already solve it?
        │
       Yes
        ▼
     Use the library

        No
        │
        ▼
Can an interface + DI express it better?
        │
       Yes
        ▼
   Use interface + DI

        No
        │
        ▼
Is it naturally a target-specific declaration?
        │
       Yes
        ▼
    expect / actual
```

This prevents unnecessary use of language-level platform declarations.

---

# 41. Real-World Architecture Pattern

A production KMP application may look like:

```text
┌──────────────────────────────────────┐
│             commonMain               │
│                                      │
│  Domain                              │
│  Use Cases                           │
│  Repositories                        │
│  Network APIs                        │
│  Database abstractions               │
│  State management                    │
│                                      │
│  Platform capabilities               │
│       ↓                              │
│    expect                            │
└──────────────────┬───────────────────┘
                   │
          ┌────────┴─────────┐
          ▼                  ▼
┌─────────────────┐  ┌─────────────────┐
│   androidMain   │  │     iosMain     │
│                 │  │                 │
│ actual services │  │ actual services │
│ Android APIs    │  │ iOS APIs        │
└─────────────────┘  └─────────────────┘
```

This is the practical value of the mechanism.

---

# 42. What Should Stay Common?

Good candidates include:

```text
Business rules
Use cases
Validation
Data models
Repository interfaces
State management
Mapping
Serialization
Most networking logic
Most persistence logic
Error models
Feature rules
```

If a library already provides multiplatform support, use it rather than recreating the abstraction.

---

# 43. What Often Belongs to `actual`?

Common examples include:

```text
Platform storage
Platform permissions
Device information
Clipboard
Platform logging
Secure platform facilities
Hardware APIs
Platform notifications
Platform-specific lifecycle integration
Platform-specific OS services
```

The exact boundary depends on the application.

---

# 44. What Should Not Be Forced Into `expect` / `actual`?

Avoid using it merely because:

```text
Android UI is different from iOS UI
```

or:

```text
A library has different internal implementations
```

or:

```text
A class has platform-specific dependencies somewhere deep inside it
```

The first question should always be:

> **Can the common architecture remain platform-neutral without creating an artificial abstraction?**

---

# 45. Common Mistakes in Real Projects

### ❌ Creating a giant `PlatformManager`

```kotlin
expect class PlatformManager {
    // 30 unrelated methods
}
```

This usually indicates too many responsibilities.

### ❌ Exposing Android types

```kotlin
expect fun getContext(): Context
```

This leaks Android into the common API.

### ❌ Creating abstractions already provided by libraries

This increases maintenance without adding value.

### ❌ Using `expect` for business logic

Business rules should generally remain common.

### ❌ Hiding platform behavior behind boolean flags

```kotlin
if (isAndroid) ...
```

This can become difficult to maintain as targets increase.

### ❌ Ignoring testability

A platform abstraction should still be testable.

---

# 46. A Better Pattern: Capability Interfaces

Instead of exposing platform details:

```kotlin
expect fun getAndroidContext(): Any
```

prefer a capability:

```kotlin
interface SecureStorage {
    fun save(key: String, value: String)
    fun read(key: String): String?
}
```

Then use:

```kotlin
expect fun createSecureStorage(): SecureStorage
```

Now the common layer knows:

```text
SecureStorage
```

rather than:

```text
Android implementation details
```

This often produces a cleaner architecture.

---

# 47. `expect` / `actual` With MVI

KMP applications frequently use shared state management.

For example:

```kotlin
sealed interface LoginAction {
    data class Submit(
        val username: String,
        val password: String
    ) : LoginAction
}
```

The reducer or state machine can remain common.

If the login flow needs secure storage:

```text
MVI state
   ↓
Use case
   ↓
Repository
   ↓
SecureStorage
   ↓
expect / actual
```

This keeps the platform boundary away from the state-management logic.

---

# 48. `expect` / `actual` With Clean Architecture

A useful layer arrangement is:

```text
Presentation
     │
     ▼
Domain
     │
     ▼
Data
     │
     ▼
Platform capability
     │
     ▼
expect / actual
```

For example:

```text
LoginUseCase
     ↓
AuthRepository
     ↓
SecureStorage
     ↓
expect
     ↓
actual
     ├── Android
     └── iOS
```

This is much cleaner than:

```text
LoginUseCase
     ↓
Android Context
```

---

# 49. The Most Important Real-World Lesson

The syntax is simple:

```kotlin
expect ...
```

and:

```kotlin
actual ...
```

The difficult part is deciding:

```text
What belongs behind the boundary?
```

A good platform abstraction is:

- Small
- Focused
- Capability-based
- Testable
- Independent of platform types
- Easy to replace
- Easy to understand
- Appropriate for the source-set hierarchy

---

# 50. Practical Checklist

Before adding `expect` / `actual`, ask:

- [ ] Is the behavior genuinely platform-specific?
- [ ] Can the functionality remain common?
- [ ] Does a multiplatform library already provide it?
- [ ] Would an interface be cleaner?
- [ ] Is the abstraction capability-based?
- [ ] Am I exposing a platform type?
- [ ] Is the expected API small?
- [ ] Is the `actual` implementation in the correct source set?
- [ ] Can the implementation be tested?
- [ ] Would the design still work with another target?
- [ ] Am I accidentally sharing platform lifecycle assumptions?
- [ ] Am I introducing unnecessary duplication?

---

# 51. Chapter Takeaways

> [!IMPORTANT]
> **Real-world `expect` / `actual` usage is about isolating genuinely platform-dependent capabilities while keeping business logic shared.**

Remember:

1. Platform identification is a simple `expect` / `actual` example.
2. Current time can be exposed through a platform-specific capability.
3. URL opening can be isolated behind a common API.
4. Secure storage is a strong platform-boundary example.
5. Device information can be exposed through a small expected API.
6. UUID generation can use platform-native facilities.
7. Logging can have a common API and platform-specific implementation.
8. File storage can isolate platform filesystem details.
9. Network connectivity can be represented as a platform capability.
10. Clipboard access is naturally platform-specific.
11. Authentication can remain common while secure token storage stays platform-specific.
12. Repository logic should generally remain common.
13. Platform services can be injected into common repositories and use cases.
14. Application configuration can expose a small platform abstraction.
15. Background execution should respect each platform's rules.
16. Biometric authentication is naturally platform-specific.
17. Camera and hardware capabilities belong at the platform boundary.
18. Permissions should be handled close to the platform.
19. Notification infrastructure can be isolated from common business logic.
20. Analytics metadata can be provided by a small platform capability.
21. Platform lifecycle models should not be forced into a universal abstraction.
22. UI differences do not automatically require `expect` / `actual`.
23. Shared ViewModel or presentation logic can remain common.
24. Native platform UI can remain platform-specific.
25. Networking does not automatically require custom `expect` / `actual`.
26. A multiplatform library may already solve a platform boundary.
27. Database drivers may already be handled by multiplatform libraries.
28. Serialization models normally belong in common code.
29. Feature-flag evaluation should generally remain common.
30. Locale and date functionality should first be evaluated against available multiplatform libraries.
31. File paths can be isolated when no suitable common abstraction exists.
32. Platform sharing can be exposed through a small capability.
33. Interfaces and `expect` / `actual` can be combined.
34. `expect` can provide a factory for an interface implementation.
35. Dependency injection can compose platform services.
36. A factory can hide platform construction.
37. The expected API should expose intent, not implementation details.
38. Avoid returning Android or iOS framework objects from common APIs.
39. Avoid giant `PlatformManager` classes.
40. Avoid using `expect` / `actual` for ordinary business abstractions.
41. Avoid duplicating functionality already provided by multiplatform libraries.
42. Keep common business decisions in `commonMain`.
43. Keep platform APIs in platform source sets.
44. Use intermediate source sets when implementations can genuinely be shared.
45. Design platform capabilities around what common code actually needs.
46. Test common business logic independently.
47. Test platform implementations independently where necessary.
48. Keep security-sensitive implementations aligned with platform security guidance.
49. Do not create custom cryptography merely to support a multiplatform example.
50. Platform-specific implementation should remain replaceable.
51. The source-set hierarchy should support the abstraction naturally.
52. Adding a new platform should not require rewriting common business logic.
53. A good abstraction reduces platform conditionals.
54. A good abstraction does not hide important platform differences.
55. `expect` / `actual` should be used intentionally rather than automatically.
56. The best platform boundary is usually the smallest useful capability.
57. **Real KMP architecture is not about making every API common; it is about making the right APIs common.**

---

## Final Thought

The strongest KMP applications do not try to pretend that every platform behaves identically.

They share what should be shared:

```text
Business logic
Domain models
Use cases
State
Repositories
Validation
```

And they isolate what should remain platform-specific:

```text
Secure storage
Permissions
Hardware
OS services
Platform lifecycle
Platform UI
Device APIs
```

The architecture becomes:

```text
                 SHARED INTENT
                      │
                      ▼
              ┌──────────────┐
              │ commonMain   │
              │              │
              │ Business     │
              │ Logic        │
              └──────┬───────┘
                     │
              Platform capability
                     │
                 expect
                     │
          ┌──────────┴──────────┐
          ▼                     ▼
       Android                  iOS
        actual                 actual
          │                     │
          ▼                     ▼
    Native Android          Native iOS
        APIs                    APIs
```

That is where `expect` / `actual` becomes valuable.

> **Share the intent. Isolate the implementation. Let each platform remain good at being itself.**
