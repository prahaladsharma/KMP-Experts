# Chapter 7 — Source Sets Deep Dive

## Part 4 — `iosMain`

> **`commonMain` defines what can be shared. `iosMain` defines what iOS uniquely needs.**

Kotlin Multiplatform is not about replacing native platform development.

It is about deciding which parts of an application should be shared and which parts should remain native.

For iOS-specific implementation, the natural source-set boundary is:

```text
iosMain
```

A typical KMP module can contain:

```text
shared/
└── src/
    ├── commonMain/
    ├── commonTest/
    ├── androidMain/
    ├── androidUnitTest/
    ├── iosMain/
    └── iosTest/
```

The relationship can be visualized as:

```text
                    commonMain
                        │
                Shared abstractions
                        │
             ┌──────────┴──────────┐
             ▼                     ▼
        androidMain             iosMain
             │                     │
       Android APIs            Apple APIs
```

The goal of `iosMain` is not to duplicate the shared application.

Its purpose is to provide the native iOS implementation whenever a capability cannot, or should not, be shared.

---

# 1. What Is `iosMain`?

`iosMain` is the iOS-specific source set in a Kotlin Multiplatform project.

It contains production code intended for iOS targets and can interact with Apple platform APIs and iOS-specific libraries supported by the project.

Typical responsibilities include:

```text
Apple platform APIs
iOS-specific SDK integrations
Keychain access
UserNotifications
Core Location
Core Bluetooth
UIKit integration
Swift/Objective-C interoperability
iOS-specific storage
iOS-specific lifecycle integration
Native iOS services
```

The important rule is:

> **If the implementation genuinely requires iOS or Apple APIs, `iosMain` is the appropriate boundary.**

---

# 2. Why `iosMain` Exists

Consider secure storage.

The application requirement is platform-independent:

```text
Store sensitive data.
Retrieve sensitive data.
Remove sensitive data.
```

The implementation can differ:

```text
commonMain
    ↓
SecureStorage
    ↓
┌─────────────────────┬─────────────────────┐
▼                     ▼
androidMain           iosMain
Android security      iOS Keychain
```

The business logic does not need to know how either platform stores the data.

---

# 3. `iosMain` Is Not a Second iOS Application

A common misunderstanding is:

```text
commonMain = shared application
iosMain = complete iOS application
```

That is not the intended model.

A better model is:

```text
commonMain
    +
iosMain
    =
KMP functionality for iOS
```

The iOS application itself may still contain native Swift/SwiftUI/UIKit code outside the shared KMP module.

The KMP shared module provides the reusable functionality.

---

# 4. The iOS Platform Boundary

A useful architecture looks like:

```text
┌─────────────────────────────────────┐
│             commonMain              │
│                                     │
│ Domain                              │
│ Business Rules                      │
│ Use Cases                           │
│ Shared State                        │
│ Shared Data Logic                   │
│ Shared Models                       │
└──────────────────┬──────────────────┘
                   │
            Platform boundary
                   │
┌──────────────────┴──────────────────┐
│               iosMain               │
│                                     │
│ Apple APIs                          │
│ iOS SDK integrations                │
│ Native adapters                     │
│ iOS-specific storage                │
│ iOS-specific services               │
└─────────────────────────────────────┘
```

The common layer should not need to understand UIKit, Foundation, Keychain, or other Apple-specific implementation details.

---

# 5. What Belongs in `iosMain`?

Typical examples include:

### Apple platform APIs

```text
Foundation
UIKit
CoreFoundation
CoreLocation
CoreBluetooth
AVFoundation
Security
UserNotifications
```

### iOS services

```text
Keychain
Notifications
Location
Bluetooth
Biometrics
Camera
Device information
Application lifecycle
```

### iOS-specific integrations

```text
Apple SDKs
iOS vendor SDKs
Objective-C frameworks
Swift interoperability
Native libraries
```

### Platform implementations

```text
IosSecureStorage
IosLogger
IosNetworkMonitor
IosPermissionHandler
IosDeviceInfoProvider
```

---

# 6. What Should Remain in `commonMain`?

Do not move code into `iosMain merely because it is used by the iOS application.

Ask:

> **Does the implementation actually depend on iOS?**

For example:

```kotlin
fun calculateTotal(
    price: Double,
    tax: Double
): Double {
    return price + tax
}
```

This has no iOS dependency.

It should remain:

```text
commonMain
```

The same implementation can be used by Android and iOS.

---

# 7. The Golden Rule

A practical rule is:

```text
If it can be shared → commonMain

If it requires iOS → iosMain
```

The goal is not to force every line of code into `commonMain`.

The goal is to keep platform-specific code at the platform boundary.

---

# 8. iOS Implementations of Common Interfaces

Suppose common code defines:

```kotlin
interface PlatformLogger {
    fun log(message: String)
}
```

The iOS source set can provide:

```kotlin
class IosLogger : PlatformLogger {

    override fun log(message: String) {
        println("iOS: $message")
    }
}
```

The common layer can depend on:

```kotlin
PlatformLogger
```

instead of knowing anything about the underlying Apple logging mechanism.

---

# 9. Interface-Based Architecture

The dependency direction becomes:

```text
commonMain
    │
    └── PlatformLogger
             ▲
             │
          iosMain
             │
        IosLogger
```

The common layer owns the abstraction.

The platform layer owns the implementation.

This is a clean application of dependency inversion.

---

# 10. Example: Secure Storage

Common:

```kotlin
interface SecureStorage {

    suspend fun save(
        key: String,
        value: String
    )

    suspend fun read(
        key: String
    ): String?

    suspend fun remove(
        key: String
    )
}
```

iOS:

```text
iosMain
    ↓
IosSecureStorage
    ↓
Apple Keychain APIs
```

Android:

```text
androidMain
    ↓
AndroidSecureStorage
    ↓
Android security APIs
```

The shared repository can simply depend on:

```text
SecureStorage
```

---

# 11. iOS Keychain

The Keychain is a classic example of a platform-specific security capability.

The application may need:

```text
Save authentication token
Retrieve authentication token
Delete authentication token
```

The business requirement is common.

The Keychain API is iOS-specific.

Therefore:

```text
Business contract
→ commonMain

Keychain implementation
→ iosMain
```

This keeps security integration close to the native platform.

---

# 12. Why Not Put Keychain Calls in `commonMain`?

Because Keychain is an Apple-specific API.

A common source set must remain compilable for all intended targets.

If common business logic directly depends on:

```text
Security framework
```

the shared architecture becomes coupled to iOS.

A platform abstraction avoids that coupling.

---

# 13. `expect` and `actual`

Kotlin Multiplatform also provides:

```text
expect
actual
```

for platform-specific implementations.

Common:

```kotlin
expect class PlatformInfo {
    val name: String
}
```

iOS:

```kotlin
actual class PlatformInfo {
    actual val name: String = "iOS"
}
```

Android can provide its own `actual` implementation.

Conceptually:

```text
                  commonMain
                      │
                    expect
                      │
             ┌────────┴────────┐
             ▼                 ▼
        androidMain          iosMain
             │                 │
           actual             actual
```

---

# 14. `expect`/`actual` Is Not the Only Option

Platform-specific requirements can be modeled using:

```text
Interfaces
Dependency Injection
Factories
expect/actual
Platform adapters
```

For example:

```kotlin
interface DeviceInfoProvider
```

may be more flexible than:

```kotlin
expect class DeviceInfoProvider
```

depending on the architecture.

The important thing is the boundary, not the mechanism itself.

---

# 15. When `expect`/`actual` Works Well

It can be useful for small, well-defined platform APIs.

For example:

```kotlin
expect fun platformName(): String
```

iOS:

```kotlin
actual fun platformName(): String = "iOS"
```

This is concise and easy to understand.

---

# 16. Keep `expect` APIs Small

Avoid creating huge expected APIs.

Prefer:

```kotlin
expect fun currentPlatformName(): String
```

over exposing dozens of platform-specific concepts.

A small API is easier to implement across:

```text
Android
iOS
Desktop
Web
```

if additional targets are introduced later.

---

# 17. iOS Foundation APIs

Foundation is widely used by iOS applications.

Depending on the KMP setup, `iosMain` can interact with appropriate Foundation APIs.

Examples include:

```text
NSDate / date-related APIs
NSURL
File management
Data
UserDefaults
Locale
Notification-related Foundation APIs
```

The important architectural question remains:

> Does the common layer need Foundation itself, or only a capability provided by Foundation?

If it only needs the capability, consider exposing an abstraction.

---

# 18. UserDefaults

Suppose the application needs lightweight preferences:

```text
Theme
First-launch state
Simple settings
Feature preferences
```

The common layer can define:

```kotlin
interface PreferencesStorage {
    fun put(key: String, value: String)
    fun get(key: String): String?
}
```

The iOS implementation can use:

```text
UserDefaults
```

The implementation belongs in:

```text
iosMain
```

---

# 19. iOS File Storage

File access can also be platform-specific.

Common:

```kotlin
interface FileStorage {
    suspend fun read(path: String): ByteArray
    suspend fun write(path: String, data: ByteArray)
}
```

iOS:

```text
iosMain
    ↓
Foundation file APIs
```

The common layer remains independent of the underlying file system.

---

# 20. iOS Device Information

Device information is often platform-specific.

Common:

```kotlin
data class DeviceInfo(
    val model: String,
    val osVersion: String
)

interface DeviceInfoProvider {
    fun get(): DeviceInfo
}
```

iOS:

```text
iosMain
    ↓
UIDevice / appropriate Apple APIs
```

Android:

```text
androidMain
    ↓
Android device APIs
```

The same domain model can be shared.

---

# 21. iOS Network Monitoring

Connectivity monitoring can be platform-specific.

Common:

```kotlin
interface NetworkMonitor {
    fun isOnline(): Boolean
}
```

iOS can use Apple networking APIs.

Android can use Android connectivity APIs.

The business layer sees only:

```text
NetworkMonitor
```

This is a clean example of capability-based design.

---

# 22. iOS Notifications

Push and local notifications require native platform integration.

Common business logic may produce:

```text
NotificationRequest
```

iOS can convert it into:

```text
UserNotifications
```

Android can convert the same intent into Android notifications.

The architecture becomes:

```text
commonMain
    ↓
NotificationRequest
    ↓
iosMain
    ↓
Apple notification APIs
```

---

# 23. Notification Permission

Permission requests are platform-specific.

Common code can express:

```kotlin
interface NotificationPermission {
    suspend fun request(): Boolean
}
```

iOS:

```text
iosMain
    ↓
UNUserNotificationCenter
```

Android:

```text
androidMain
    ↓
Android permission APIs
```

The application flow remains common.

---

# 24. Location Services

Location is another classic platform boundary.

Common:

```kotlin
interface LocationProvider {
    suspend fun currentLocation(): Location
}
```

iOS:

```text
iosMain
    ↓
Core Location
```

Android:

```text
androidMain
    ↓
Android location APIs
```

The shared business logic should not depend directly on either platform's location framework.

---

# 25. Bluetooth

Bluetooth implementations are platform-specific.

iOS:

```text
iosMain
    ↓
Core Bluetooth
```

Android:

```text
androidMain
    ↓
Android Bluetooth APIs
```

A common abstraction can expose only the operations the application actually needs.

For example:

```kotlin
interface BluetoothManager {
    suspend fun scan(): List<BluetoothDevice>
}
```

---

# 26. Camera

Camera APIs are also platform-specific.

Common:

```text
CaptureImage
```

iOS:

```text
iosMain
    ↓
AVFoundation / appropriate iOS camera APIs
```

Android:

```text
androidMain
    ↓
CameraX / Android camera APIs
```

The exact API choice belongs to the platform implementation.

---

# 27. Biometrics

Biometric authentication is an important example.

Common:

```kotlin
interface BiometricAuthenticator {
    suspend fun authenticate(): AuthenticationResult
}
```

iOS:

```text
iosMain
    ↓
LocalAuthentication
```

Android:

```text
androidMain
    ↓
BiometricPrompt
```

The authentication business flow remains common.

---

# 28. iOS Lifecycle

iOS has platform-specific application lifecycle concepts.

Depending on the application architecture, these may involve:

```text
UIApplication
UIScene
SceneDelegate
SwiftUI lifecycle
App lifecycle callbacks
```

These concerns should stay at the iOS boundary.

Shared services can expose platform-independent lifecycle operations where necessary.

---

# 29. iOS UI Integration

KMP can share business logic independently of whether the iOS UI is built using:

```text
SwiftUI
UIKit
Compose Multiplatform
```

If the UI is native SwiftUI:

```text
SwiftUI
    ↓
iOS application
    ↓
KMP shared module
```

If the UI uses Compose Multiplatform:

```text
Compose UI
    ↓
commonMain / platform-specific integrations
```

The source-set decision depends on the actual APIs being used.

---

# 30. SwiftUI Does Not Automatically Belong in `commonMain`

SwiftUI is an Apple framework.

Therefore SwiftUI-specific integration remains on the iOS side.

Common code can expose:

```text
State
Models
Use Cases
Events
UI state
```

SwiftUI can consume that state.

For example:

```text
commonMain
    ↓
UiState
    ↓
SwiftUI
```

The shared state remains multiplatform.

---

# 31. UIKit Integration

UIKit is also platform-specific.

Examples:

```text
UIViewController
UIView
UIApplication
UIPasteboard
UIDevice
```

If KMP code needs to interact with these APIs, that implementation belongs in:

```text
iosMain
```

Avoid leaking UIKit objects into domain models.

---

# 32. Swift Interoperability

One of KMP's important capabilities is interoperability with the Apple ecosystem.

The Kotlin/Native toolchain allows Kotlin code to interact with Objective-C and supported Apple frameworks.

This makes:

```text
iosMain
```

the natural place for native Apple integrations.

The goal is not to pretend Apple APIs do not exist.

The goal is to use them deliberately.

---

# 33. Kotlin/Native and iOS

iOS KMP code is compiled for Kotlin/Native targets.

Conceptually:

```text
Kotlin source
      ↓
Kotlin/Native compiler
      ↓
iOS native binary integration
```

This differs from Android's JVM-based execution model.

That difference is important when designing platform-specific code.

---

# 34. Memory and Native Interoperability

Kotlin/Native has its own runtime and interoperability model.

Modern Kotlin/Native uses a garbage-collected memory management model rather than the historical strict freezing model.

When working with Apple APIs, developers should still understand:

```text
Object lifetime
Concurrency
Interop types
Callbacks
Native references
Threading
```

The exact behavior depends on the Kotlin version and APIs involved.

---

# 35. Objective-C Interoperability

Many Apple frameworks expose Objective-C-compatible APIs.

Kotlin/Native can interoperate with those APIs.

This enables KMP code in `iosMain` to use platform functionality such as:

```text
Foundation
UIKit
Security
CoreLocation
UserNotifications
```

where supported by the target and project configuration.

---

# 36. Swift Interoperability

KMP projects often expose shared Kotlin APIs to Swift.

However, not every Kotlin construct maps equally naturally into Swift.

API design matters.

For example:

```kotlin
data class User(
    val id: String,
    val name: String
)
```

may be straightforward to consume, while more complex Kotlin-specific constructs may require additional consideration.

This is one reason KMP API design should consider the iOS consumer.

---

# 37. Design APIs for Both Platforms

A shared API should be:

```text
Kotlin-friendly
+
Android-friendly
+
iOS-friendly
```

Avoid unnecessarily exposing complicated Kotlin implementation details if the API will be consumed directly from Swift.

This becomes especially important for:

```text
Flows
Suspend functions
Sealed hierarchies
Generics
Exceptions
Callbacks
```

---

# 38. `Flow` and iOS

A common KMP application may expose:

```kotlin
val state: StateFlow<UiState>
```

Android can collect it naturally using Kotlin/Android tools.

iOS may need an adapter or lifecycle-aware bridge depending on the architecture.

The important point is:

```text
State ownership → commonMain
iOS observation mechanism → iOS boundary
```

Do not move the entire state-management logic into `iosMain simply because Swift needs a bridge.

---

# 39. iOS-Specific Flow Adapters

A common architecture can use:

```text
commonMain
    ↓
StateFlow
    ↓
iOS adapter
    ↓
SwiftUI / UIKit
```

The adapter converts shared state into a form convenient for the iOS UI.

This keeps the source of truth in shared code.

---

# 40. Coroutines and iOS

Shared suspend functions can be consumed from iOS through appropriate interoperability patterns.

However, the UI integration layer must respect:

```text
Main-thread UI requirements
Lifecycle
Cancellation
Swift concurrency expectations
```

The platform adapter is often the right place to handle those concerns.

---

# 41. Avoid Moving Business Logic to Swift Just Because UI Is Swift

Suppose:

```text
Order validation
Payment rules
Discount calculation
Authentication state
```

are already implemented in common Kotlin.

Do not recreate them in Swift simply because the UI is native.

Instead:

```text
SwiftUI
    ↓
Shared Kotlin API
    ↓
commonMain business logic
```

This preserves the value of KMP.

---

# 42. iOS-Specific Dependency Placement

If a library is Apple/iOS-only:

```text
iosMain
```

is the natural location.

If a dependency supports all intended KMP targets:

```text
commonMain
```

may be appropriate.

For example:

```text
Apple-only SDK
→ iosMain

Multiplatform HTTP client
→ commonMain
```

Always verify actual target support before choosing the source set.

---

# 43. Do Not Assume a Library Is Multiplatform

A Kotlin library is not automatically a KMP library.

Before putting a dependency into:

```text
commonMain
```

verify that it supports the required targets.

If it only supports iOS:

```text
iosMain
```

is the correct location.

This prevents platform-specific dependencies from leaking into common compilation.

---

# 44. iOS SDK Wrappers

A wrapper is often useful for native SDK integration.

For example:

```text
commonMain
    ↓
PaymentService
```

iOS:

```text
iosMain
    ↓
IosPaymentService
    ↓
Apple/payment SDK
```

The shared feature remains independent of the native SDK.

---

# 45. iOS Analytics

Common:

```kotlin
interface Analytics {
    fun track(event: AnalyticsEvent)
}
```

iOS:

```text
IosAnalytics
    ↓
iOS analytics SDK
```

Android:

```text
AndroidAnalytics
    ↓
Android analytics SDK
```

The analytics contract remains common.

---

# 46. iOS URL Launcher

Common:

```kotlin
interface UrlLauncher {
    fun open(url: String): Boolean
}
```

iOS:

```text
iosMain
    ↓
Apple URL handling
```

Android:

```text
androidMain
    ↓
Android Intent
```

This is a simple example of platform capability abstraction.

---

# 47. iOS Share Sheet

Common:

```kotlin
data class ShareRequest(
    val text: String
)
```

iOS:

```text
iosMain
    ↓
UIActivityViewController
```

Android:

```text
androidMain
    ↓
Android share Intent
```

The business layer only expresses the intent to share.

---

# 48. iOS Clipboard

Clipboard access is platform-specific.

Common:

```kotlin
interface Clipboard {
    fun copy(text: String)
    fun paste(): String?
}
```

iOS:

```text
iosMain
    ↓
UIPasteboard
```

Android:

```text
androidMain
    ↓
Android ClipboardManager
```

Again, the shared layer depends on the capability.

---

# 49. iOS Background Work

Background execution on iOS has platform-specific rules and APIs.

Unlike Android, iOS places strong constraints on when and how background work can execute.

Therefore:

```text
Business synchronization
→ commonMain

iOS scheduling/background integration
→ iosMain
```

The platform decides how and when the work is scheduled.

---

# 50. iOS Permissions

Permissions can include:

```text
Camera
Location
Notifications
Bluetooth
Microphone
Contacts
Photos
```

The permission mechanism belongs to the iOS platform layer.

Common code can define the required capability.

iOS implements the platform interaction.

---

# 51. iOS Resource Handling

iOS resources may be managed through:

```text
Asset catalogs
Localized strings
Bundle resources
SwiftUI resources
```

The exact resource mechanism belongs to the iOS application layer.

Common code should ideally expose semantic information rather than Apple-specific resource identifiers.

For example:

```text
ValidationResult.InvalidUser
```

is preferable to:

```text
iOS resource identifier
```

inside domain logic.

---

# 52. iOS Localization

Localization is often platform-specific at the presentation layer.

Common code can expose:

```text
MessageKey.InvalidUser
```

The iOS UI can resolve it to an appropriate localized string.

Android can resolve the same semantic key to its Android resource.

This keeps business logic independent of resource systems.

---

# 53. iOS Security Boundary

Security-sensitive platform APIs should remain isolated.

Examples:

```text
Keychain
LocalAuthentication
Secure Enclave integrations
Device attestation
Credential APIs
```

Common code can define:

```text
Security policy
Authentication state
Credential contracts
```

The iOS layer provides the native implementation.

---

# 54. iOS Biometrics

A common abstraction:

```kotlin
interface BiometricAuthenticator {
    suspend fun authenticate(): AuthenticationResult
}
```

iOS:

```text
iosMain
    ↓
LocalAuthentication
```

Android:

```text
androidMain
    ↓
BiometricPrompt
```

This allows the same authentication flow to work across platforms.

---

# 55. iOS Device Authentication

Do not let common business logic know:

```text
LAContext
```

Instead, expose:

```text
BiometricAuthenticator
```

This prevents Apple-specific classes from becoming part of the shared domain API.

---

# 56. iOS Platform Factory

A platform factory can create iOS implementations.

Conceptually:

```text
iosMain
    ↓
createPlatformServices()
    ↓
IosSecureStorage
IosLogger
IosNetworkMonitor
IosDeviceInfoProvider
```

The iOS application can then provide these services to shared components.

---

# 57. Composition Root on iOS

The iOS composition root may be:

```text
SwiftUI App
UIKit AppDelegate
Dependency container
Platform bootstrap
```

Its responsibility is to connect:

```text
iOS implementations
```

to:

```text
shared abstractions
```

Conceptually:

```text
iOS App
   │
   ├── IosLogger
   ├── IosSecureStorage
   ├── IosNetworkMonitor
   │
   ▼
Shared KMP components
```

---

# 58. Keep Platform Construction at the Edge

A useful rule is:

> **Create Apple-specific objects at the iOS edge, not deep inside shared business logic.**

Avoid:

```kotlin
class UserRepository {
    // Direct native iOS dependency creation
}
```

Prefer:

```kotlin
class UserRepository(
    private val storage: SecureStorage
)
```

and provide:

```text
IosSecureStorage
```

from the platform layer.

---

# 59. iOS and Clean Architecture

A Clean Architecture-inspired structure might be:

```text
commonMain
├── domain
├── usecase
├── repository contracts
└── shared data logic

iosMain
├── native adapters
├── iOS implementations
└── Apple SDK integrations
```

This keeps the business core independent from Apple frameworks.

---

# 60. iOS Repository Implementations

Not every repository needs to be in `iosMain`.

Suppose a repository uses only:

```text
KMP HTTP client
Shared serialization
Shared database
Shared models
```

It may remain in:

```text
commonMain
```

If only a specific data source needs an iOS API:

```text
iosMain
```

may contain that implementation.

Again:

> **Split the platform-specific capability, not necessarily the entire feature.**

---

# 61. Example: Authentication

A shared authentication flow:

```text
Login
   ↓
Validate credentials
   ↓
Call API
   ↓
Store token
   ↓
Authenticated
```

can live in:

```text
commonMain
```

Only secure storage may be platform-specific:

```text
commonMain
    ↓
SecureStorage
    ↓
iosMain → Keychain
androidMain → Android security implementation
```

This produces much more shared code.

---

# 62. Example: User Profile

The profile feature may contain:

```text
User model
Profile repository
Fetch profile use case
Validation
State management
```

all in:

```text
commonMain
```

Only platform-specific operations such as:

```text
Photo picker
Camera
Share
```

need platform implementations.

---

# 63. Example: File Picker

Common:

```text
SelectFile
```

iOS:

```text
iosMain
    ↓
UIDocumentPickerViewController
```

Android:

```text
androidMain
    ↓
Android document picker
```

The business workflow remains shared.

---

# 64. Example: Camera Capture

Common:

```kotlin
interface Camera {
    suspend fun capture(): ByteArray
}
```

iOS:

```text
iosMain
    ↓
AVFoundation / iOS camera APIs
```

Android:

```text
androidMain
    ↓
CameraX / Android APIs
```

The platform implementations are separate.

The business feature remains common.

---

# 65. Platform Capability Pattern

A recurring architecture pattern is:

```text
                commonMain
                    │
             Capability API
                    │
        ┌───────────┴───────────┐
        ▼                       ▼
   androidMain               iosMain
        │                       │
 Android implementation    iOS implementation
```

This pattern works well for:

```text
Storage
Logging
Analytics
Permissions
Notifications
Biometrics
Location
Bluetooth
Camera
Clipboard
URL handling
```

---

# 66. Avoid Platform Leakage

Platform leakage occurs when common code starts exposing:

```text
UIViewController
UIView
UIApplication
UIDevice
LAContext
UNUserNotificationCenter
```

as part of its business API.

That makes the shared code less reusable.

Instead, expose a business capability.

For example:

```text
BiometricAuthenticator
```

rather than:

```text
LAContext
```

---

# 67. Platform Types Should Stop at the Boundary

A healthy boundary looks like:

```text
iosMain
    │
    ├── Apple type
    │
    ▼
Common abstraction
    │
    ▼
commonMain
```

The Apple type is translated into a platform-neutral representation.

For example:

```text
CLLocation
    ↓
Location
```

or:

```text
UIDevice
    ↓
DeviceInfo
```

---

# 68. Translation Is an Architectural Tool

Platform adapters often translate:

```text
Apple API
    ↓
Kotlin abstraction
```

For example:

```text
UNAuthorizationStatus
    ↓
NotificationPermissionStatus
```

This allows the common layer to work with a stable model.

---

# 69. iOS Errors

Apple APIs may expose platform-specific errors.

Avoid exposing those directly through domain APIs when possible.

Instead:

```text
Apple error
    ↓
iosMain adapter
    ↓
Domain error
```

For example:

```kotlin
sealed interface AuthenticationResult {
    data object Success : AuthenticationResult
    data object Cancelled : AuthenticationResult
    data object Failed : AuthenticationResult
}
```

The common layer can work with semantic outcomes.

---

# 70. iOS Callbacks

Many Apple APIs use callback-based patterns.

The platform layer can adapt callbacks into:

```text
suspend functions
Flow
StateFlow
channels
callbacks
```

depending on the shared architecture.

For example:

```text
Apple callback
    ↓
iosMain adapter
    ↓
suspend/Flow abstraction
    ↓
commonMain
```

This keeps platform callback complexity localized.

---

# 71. iOS Delegates

Apple frameworks frequently use delegates.

Examples:

```text
CLLocationManagerDelegate
CBCentralManagerDelegate
UNUserNotificationCenterDelegate
```

These delegate implementations naturally belong to:

```text
iosMain
```

The delegate can forward meaningful events into a shared abstraction.

---

# 72. iOS Notifications as Flow

An iOS native event source could conceptually become:

```text
Apple notification callback
        ↓
iosMain adapter
        ↓
Flow<PlatformEvent>
        ↓
commonMain
```

The common layer then works with a multiplatform stream rather than an Apple delegate.

---

# 73. iOS Lifecycle Events

Similarly:

```text
iOS lifecycle callback
        ↓
iosMain
        ↓
PlatformLifecycleEvent
        ↓
commonMain
```

This approach prevents UIKit or SwiftUI lifecycle objects from leaking into shared business logic.

---

# 74. iOS Main Thread

UI updates on iOS generally need to respect the main thread.

If an iOS adapter interacts with:

```text
UIKit
SwiftUI
Apple UI APIs
```

the platform layer should handle the appropriate execution context.

Shared business logic should remain independent from UIKit's threading model where possible.

---

# 75. iOS Background Execution Constraints

A major platform difference is background execution.

Android and iOS have different scheduling and lifecycle models.

Therefore, avoid designing common code around:

```text
Android WorkManager semantics
```

or:

```text
iOS background task semantics
```

Instead, common code should expose the work:

```text
SyncOrdersUseCase
```

and each platform decides how to schedule it.

---

# 76. Shared Work vs Platform Scheduling

The architecture becomes:

```text
commonMain
    ↓
SyncOrdersUseCase
```

Android:

```text
androidMain
    ↓
Android scheduler
    ↓
SyncOrdersUseCase
```

iOS:

```text
iosMain
    ↓
iOS background mechanism
    ↓
SyncOrdersUseCase
```

This is a strong example of separating behavior from execution strategy.

---

# 77. iOS and Networking

Many KMP networking stacks can be shared.

Therefore:

```text
API client
Serialization
Repository
Error mapping
Authentication
```

may remain in:

```text
commonMain
```

The underlying networking engine may use platform-specific implementations internally.

Do not move the entire network layer to `iosMain` unless the architecture actually requires iOS-only networking behavior.

---

# 78. Platform Engine vs Shared API

A library may provide:

```text
Common API
```

with:

```text
iOS engine
Android engine
```

internally.

That does not mean your application networking code needs separate source sets.

Distinguish:

```text
Library's platform implementation
```

from:

```text
Your application's platform-specific code.
```

---

# 79. iOS Database Integration

Similarly, a KMP-compatible database can allow shared persistence logic.

If the database API supports iOS:

```text
commonMain
```

may contain:

```text
Database
DAO
Queries
Repository
```

while truly native database integrations can remain in:

```text
iosMain
```

when necessary.

---

# 80. iOS-Specific Database Driver

Some database libraries use platform-specific drivers.

Conceptually:

```text
commonMain
    ↓
Database API
    ↓
iosMain
    ↓
iOS driver
```

The shared persistence layer can remain common if the library architecture supports it.

---

# 81. iOS Build Dependencies

iOS-specific dependencies should be kept within the appropriate source set or native framework configuration.

The principle is:

```text
Apple-only dependency
→ iOS boundary

Multiplatform dependency
→ common boundary
```

This keeps dependency graphs understandable.

---

# 82. Source Sets Are More Than Folders

`iosMain` is not simply:

```text
A folder named iosMain
```

It represents a compilation and dependency boundary.

It determines:

```text
Which APIs are available
Which libraries can be used
Which target compiles the code
Which platform receives the implementation
```

This is why correct source-set placement matters.

---

# 83. Compiler Feedback

If common code tries to use:

```text
UIKit
```

or another Apple-specific API, compilation should expose the mismatch.

That is useful feedback.

Instead of trying to bypass the boundary, ask:

```text
Should this API remain in iosMain?
```

or:

```text
Should commonMain depend on an abstraction?
```

---

# 84. `iosMain` and Testing

iOS-specific implementations need platform-aware testing.

The general relationship is:

```text
commonMain
    ↓
commonTest

iosMain
    ↓
iosTest
```

Common tests verify:

```text
Shared business behavior
```

iOS tests verify:

```text
iOS-specific implementation behavior
```

---

# 85. Common Tests vs iOS Tests

For secure storage:

```text
commonTest
    ↓
Repository behavior
    ↓
Fake SecureStorage
```

iOS-specific tests:

```text
iosTest
    ↓
IosSecureStorage
    ↓
Keychain integration
```

This gives confidence at both architectural layers.

---

# 86. Test the Boundary

A particularly useful testing strategy is to test the contract.

For example:

```text
SecureStorage contract
```

should have expected behavior.

Then:

```text
AndroidSecureStorage
IosSecureStorage
```

can each be validated against that behavior.

This makes platform implementations interchangeable from the common layer's perspective.

---

# 87. iOS Test Considerations

Native iOS APIs may have requirements around:

```text
Simulator
Device
Entitlements
Permissions
Keychain access
UI lifecycle
Main thread
```

Some integration tests may therefore require an actual iOS runtime environment rather than only common unit tests.

This is another reason to keep platform-specific tests separate from common tests.

---

# 88. iOS and Entitlements

Some Apple capabilities require project configuration and entitlements.

Examples may include:

```text
Keychain-related capabilities
Push notifications
Background modes
Associated domains
Bluetooth
Location
```

These are platform/application configuration concerns.

They should not leak into common business logic.

---

# 89. iOS Application Configuration

The iOS application may configure:

```text
Info.plist
Entitlements
Capabilities
URL schemes
Background modes
App lifecycle
```

These belong to the iOS application boundary.

The shared KMP module should expose only the functionality required by the application.

---

# 90. iOS Deep Links

Deep linking can be split into two responsibilities.

### Common

```text
Parse route
Determine destination
Validate parameters
```

### iOS

```text
Register URL scheme
Receive URL
Connect to application lifecycle
```

This keeps platform integration separate from route logic.

---

# 91. iOS App Lifecycle and Shared State

Shared state can remain in:

```text
commonMain
```

while iOS lifecycle events are handled by:

```text
iosMain / iOS application
```

For example:

```text
App enters foreground
    ↓
iOS lifecycle
    ↓
Shared refresh capability
```

The shared code does not need to know whether the event came from UIKit or SwiftUI.

---

# 92. iOS Push Notifications

Push notification registration is platform-specific.

A common application may define:

```text
RegisterForNotifications
```

but iOS handles:

```text
APNs registration
Device token
Notification permissions
Notification callbacks
```

at the platform boundary.

---

# 93. Device Token Handling

The iOS device token is platform-specific.

The platform layer can translate it into:

```text
String / ByteArray / domain representation
```

before passing it to shared code.

For example:

```text
APNs token
    ↓
iosMain
    ↓
RegisterDeviceToken
    ↓
commonMain
```

---

# 94. iOS Share and Open Operations

The same principle applies to:

```text
Open URL
Share content
Copy text
Open settings
Pick file
Capture image
```

Common code expresses intent.

iOS implements the native operation.

---

# 95. Platform Capability Interfaces

A mature shared architecture may define:

```kotlin
interface PlatformServices {
    val logger: Logger
    val storage: SecureStorage
    val connectivity: NetworkMonitor
    val deviceInfo: DeviceInfoProvider
}
```

The iOS implementation can provide:

```text
IosPlatformServices
```

The Android implementation can provide:

```text
AndroidPlatformServices
```

Use this pattern carefully; smaller focused interfaces are often easier to test and evolve.

---

# 96. Prefer Focused Interfaces

Instead of:

```kotlin
interface EverythingIosNeeds
```

prefer:

```text
Logger
Storage
NetworkMonitor
DeviceInfoProvider
PermissionHandler
```

This follows the interface segregation principle and reduces coupling.

---

# 97. iOS Adapter Example

Conceptually:

```text
                 commonMain
                     │
              DeviceInfoProvider
                     ▲
                     │
                  iosMain
                     │
          IosDeviceInfoProvider
                     │
               UIDevice
```

The Apple object remains isolated.

The shared layer receives:

```text
DeviceInfo
```

---

# 98. Avoid Returning Native Objects

Avoid exposing:

```text
UIView
UIViewController
UIImage
NSURLSession
CLLocation
```

from shared business APIs unless there is a very specific interoperability requirement.

Prefer platform-neutral representations:

```text
ImageReference
Location
Url
FileData
```

when appropriate.

---

# 99. The Platform Adapter Pattern

A powerful pattern is:

```text
Native API
    ↓
Platform Adapter
    ↓
Common abstraction
    ↓
Shared business logic
```

For iOS:

```text
Apple API
    ↓
iosMain adapter
    ↓
KMP abstraction
    ↓
commonMain
```

This pattern keeps the architecture clean as the application grows.

---

# 100. iOSMain and the KMP Mental Model

The most useful way to remember `iosMain` is:

```text
                  commonMain
                      │
             "What the app needs"
                      │
                      ▼
             Platform abstraction
                      │
                      ▼
                   iosMain
                      │
              "How iOS does it"
                      │
                      ▼
                Apple APIs
```

The common layer answers:

> **What capability does the application need?**

The iOS layer answers:

> **How should Apple provide that capability?**

That separation is one of the most important skills in KMP architecture.

---

# Chapter Takeaways

> [!TIP]
> **`iosMain` is the iOS boundary of a KMP module. Keep Apple APIs, iOS-specific SDKs, native integrations, and platform implementations here while keeping business behavior and reusable abstractions in `commonMain`.**

Remember:

1. `iosMain` is the iOS-specific production source set.
2. `commonMain` contains shared production behavior.
3. `iosMain` contains iOS-specific production behavior.
4. `iosMain` is not a complete second iOS application.
5. Its purpose is to provide iOS-specific implementations.
6. Apple framework integrations naturally belong at the iOS boundary.
7. UIKit belongs at the iOS boundary.
8. SwiftUI-specific integration belongs at the iOS boundary.
9. Keychain integration belongs in the iOS platform layer.
10. UserNotifications integration belongs in the iOS platform layer.
11. Core Location integration belongs in the iOS platform layer.
12. Core Bluetooth integration belongs in the iOS platform layer.
13. LocalAuthentication integration belongs in the iOS platform layer.
14. iOS-specific storage belongs in the iOS platform layer.
15. iOS lifecycle integration belongs at the iOS boundary.
16. iOS background execution belongs at the iOS boundary.
17. Apple SDK integrations should not leak into common business logic.
18. Business rules should generally remain in `commonMain`.
19. Shared use cases should generally remain in `commonMain`.
20. Shared models should generally remain in `commonMain`.
21. Shared networking can remain common when the selected stack supports the intended targets.
22. Shared persistence can remain common when the selected database supports the intended targets.
23. Interfaces can isolate iOS implementations.
24. Dependency injection can connect common abstractions to iOS implementations.
25. `expect`/`actual` is another mechanism for platform-specific implementations.
26. `expect`/`actual` should not automatically replace interfaces.
27. Keep `expect` APIs small and capability-focused.
28. Model capabilities rather than Apple framework objects.
29. Avoid passing UIKit objects into shared domain logic.
30. Avoid exposing Apple-specific errors through shared business APIs.
31. Translate native API results into platform-neutral models where appropriate.
32. Keep iOS dependency construction near the iOS composition root.
33. Keep Apple-only dependencies out of `commonMain`.
34. Verify library target support before placing dependencies in `commonMain`.
35. Android-only and iOS-only libraries belong in their respective platform boundaries.
36. iOS-specific tests can live in `iosTest`.
37. Shared behavior should be tested through `commonTest`.
38. Platform integrations should receive platform-specific tests.
39. Test the contracts shared between platform implementations.
40. iOS permissions are platform-specific.
41. iOS notification registration is platform-specific.
42. APNs integration is platform-specific.
43. iOS URL handling is platform-specific.
44. iOS share-sheet integration is platform-specific.
45. iOS clipboard integration is platform-specific.
46. iOS camera integration is platform-specific.
47. iOS file picker integration is platform-specific.
48. iOS device information collection is platform-specific.
49. SwiftUI does not make business logic iOS-specific.
50. Native iOS UI can consume shared state and business logic.
51. Do not duplicate shared business rules in Swift merely because the UI is SwiftUI.
52. Shared state can remain in `commonMain`.
53. iOS adapters can bridge shared state into SwiftUI or UIKit.
54. Native callbacks and delegates can be adapted at the iOS boundary.
55. Apple-specific lifecycle events can be translated into shared events.
56. iOS background scheduling should invoke shared work rather than duplicate it.
57. Platform scheduling should remain platform-specific.
58. A repository does not automatically become iOS-specific because one dependency is native.
59. Split the platform-specific capability rather than moving the entire feature.
60. Source sets are compilation and dependency boundaries, not merely folders.
61. Compiler errors can reveal accidental Apple platform coupling.
62. A large `iosMain` is not automatically bad.
63. The objective is not 100% common code.
64. The objective is maximum sensible sharing with clear platform boundaries.
65. Native iOS code is valuable when Apple provides unique capabilities.
66. `iosMain` acts as an adapter between shared application intent and Apple APIs.
67. The common layer defines what the application needs.
68. The iOS layer defines how iOS provides it.
69. **Good KMP architecture does not remove iOS. It gives iOS a well-defined place to live.**

---

# Final Thought

Kotlin Multiplatform becomes powerful when native platform capabilities are treated as partners rather than problems.

The architecture can be summarized as:

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

`commonMain` should contain the behavior worth sharing.

`iosMain` should contain the behavior that genuinely belongs to Apple platforms.

That does not weaken KMP.

It is what makes KMP practical.

> **Share the business logic. Respect the platform. Keep Apple-specific implementation at the edge.**
