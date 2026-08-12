# Chapter 4 — Inside Kotlin Multiplatform

## Part 5 — Runtime

> **The compiler gets KMP code onto a platform. The runtime is where that code actually becomes part of a running application.**

A KMP project can successfully compile for Android and iOS and still behave differently at runtime.

That distinction matters.

Compilation answers:

> **Can this code be built for the target?**

Runtime answers:

> **What actually happens when the application starts, executes shared code, talks to the platform, performs asynchronous work, accesses storage, and eventually gets destroyed?**

For an Android developer, this is where the KMP mental model becomes more interesting.

The same Kotlin source can participate in two very different application environments:

```text
                    Shared Kotlin Code
                           │
             ┌─────────────┴─────────────┐
             ▼                           ▼
       Android Runtime              iOS Runtime
             │                           │
             ▼                           ▼
          JVM / ART                  Native Runtime
             │                           │
             ▼                           ▼
       Android APIs                Apple APIs
```

KMP shares code.

It does not create one universal runtime.

---

# 1. Source Code vs Runtime

It is easy to confuse these two ideas:

```text
Same Source Code
```

and:

```text
Same Runtime
```

They are not the same.

Suppose we have:

```kotlin
class ProductService {
    suspend fun getProducts(): List<Product> {
        // shared logic
    }
}
```

That source may be compiled for Android and iOS.

At runtime:

```text
Android
ProductService
      │
      ▼
Android Application Process
```

while:

```text
iOS
ProductService
      │
      ▼
iOS Application Process
```

The implementation may be shared, but the surrounding runtime environment is different.

---

# 2. The Runtime Mental Model

A useful high-level model is:

```text
                         Application
                              │
                ┌─────────────┴─────────────┐
                ▼                           ▼
             Android                        iOS
                │                           │
                ▼                           ▼
         Native UI Layer              Native UI Layer
                │                           │
                └─────────────┬─────────────┘
                              ▼
                       Shared Kotlin Code
                              │
                ┌─────────────┼─────────────┐
                ▼             ▼             ▼
             Domain          Data        Platform
                │             │             │
                └─────────────┴─────────────┘
                              │
                              ▼
                         Runtime Services
```

The shared layer executes inside the host application.

Android hosts it on Android.

iOS hosts it on iOS.

---

# 3. Android Runtime

On Android, Kotlin code ultimately executes within the Android application runtime environment.

A simplified path is:

```text
Kotlin Source
      │
      ▼
Kotlin/JVM Compilation
      │
      ▼
Android Application
      │
      ▼
Android Runtime
      │
      ▼
Device
```

The Android application has access to Android platform capabilities such as:

```text
Context
Activity
Services
Android SDK
Storage
Network
Sensors
```

KMP shared code can interact with these capabilities through appropriate boundaries.

---

# 4. iOS Runtime

On iOS, the shared Kotlin code follows a different runtime path.

Conceptually:

```text
Kotlin Source
      │
      ▼
Kotlin/Native
      │
      ▼
Native Output
      │
      ▼
iOS Application
      │
      ▼
Apple Runtime Environment
```

The application can interact with:

```text
Foundation
UIKit
SwiftUI
Core frameworks
Device APIs
Apple services
```

The shared Kotlin code becomes part of the native iOS application rather than running inside the Android runtime.

---

# 5. Two Applications, One Shared Codebase

This is the key idea:

```text
                         Shared Code
                             │
                  ┌──────────┴──────────┐
                  ▼                     ▼
              Android                  iOS
                  │                     │
                  ▼                     ▼
             Android App           iOS App
                  │                     │
                  ▼                     ▼
           Android Runtime        Apple Runtime
```

KMP does not turn Android and iOS into one application.

Instead, it allows selected implementation logic to be reused across two applications.

---

# 6. The Runtime Boundary

A useful architecture is:

```text
┌─────────────────────────────────────────┐
│              Android App               │
│                                         │
│  Compose / Views / Android APIs         │
└────────────────────┬────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────┐
│              Shared KMP                 │
│                                         │
│ Domain • Data • Business Logic          │
└────────────────────┬────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────┐
│            Platform Layer               │
│                                         │
│ Android APIs / iOS APIs                 │
└─────────────────────────────────────────┘
```

The iOS application follows the same conceptual model:

```text
┌─────────────────────────────────────────┐
│                iOS App                 │
│                                         │
│  SwiftUI / UIKit / Apple APIs          │
└────────────────────┬────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────┐
│              Shared KMP                 │
│                                         │
│ Domain • Data • Business Logic          │
└────────────────────┬────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────┐
│            Platform Layer               │
│                                         │
│ Apple APIs / iOS APIs                   │
└─────────────────────────────────────────┘
```

---

# 7. What Actually Runs at Runtime?

A production application may execute:

```text
UI
 ↓
Presentation
 ↓
ViewModel / State
 ↓
Use Case
 ↓
Repository
 ↓
Network
 ↓
Database / Cache
 ↓
Platform Services
```

In a KMP application, some or many of these layers can be shared.

For example:

```text
Android UI ───────┐
                  ▼
            Shared State
                  │
                  ▼
            Shared Domain
                  │
                  ▼
            Shared Repository
                  │
                  ▼
               Network
                  ▲
                  │
iOS UI ───────────┘
```

The UI may differ.

The business flow can remain the same.

---

# 8. Runtime Is About Behavior

Consider an authentication flow:

```text
User taps Login
       │
       ▼
Validate input
       │
       ▼
Call authentication API
       │
       ▼
Receive response
       │
       ▼
Store token
       │
       ▼
Update state
       │
       ▼
Show authenticated screen
```

Most of this behavior may be shared.

The actual UI interaction can remain platform-specific.

```text
Android UI ──┐
             ├── Login Flow ── Shared Logic
iOS UI ──────┘
```

This is where KMP can provide substantial value.

---

# 9. Runtime State

Shared code frequently manages application state.

For example:

```kotlin
data class LoginState(
    val email: String = "",
    val password: String = "",
    val loading: Boolean = false,
    val error: String? = null
)
```

The state object can be shared.

Android may render it using:

```text
Jetpack Compose
```

iOS may render it using:

```text
SwiftUI
```

Conceptually:

```text
                LoginState
                    │
          ┌─────────┴─────────┐
          ▼                   ▼
      Android UI           iOS UI
```

The runtime representation of the state exists inside each application's process.

---

# 10. Coroutines at Runtime

KMP applications frequently use Kotlin coroutines for asynchronous work.

For example:

```kotlin
suspend fun loadProducts(): List<Product> {
    return repository.getProducts()
}
```

At runtime:

```text
UI
 │
 ▼
Coroutine
 │
 ▼
Use Case
 │
 ▼
Repository
 │
 ▼
Network
```

The coroutine model is shared at the Kotlin level.

But the underlying execution environment still depends on the target.

---

# 11. Coroutine Context Matters

Consider:

```kotlin
withContext(Dispatchers.Default) {
    calculateSomething()
}
```

The intent is:

```text
Run work away from the caller's current execution context.
```

But the actual runtime implementation of dispatching depends on the platform and Kotlin runtime.

The important architectural rule is:

> **Shared code should express concurrency intent rather than assume Android-specific threading behavior.**

Avoid putting assumptions such as:

```text
"This always runs on an Android thread."
```

inside common business logic.

---

# 12. Main Thread and UI Thread

Both Android and iOS have UI-thread requirements.

Conceptually:

```text
                 UI
                  │
                  ▼
             Main Thread
                  │
          ┌───────┴───────┐
          ▼               ▼
       Android            iOS
```

The exact platform mechanics differ.

The shared layer should therefore avoid unnecessarily controlling UI-thread behavior using platform-specific APIs.

Instead:

```text
Shared Logic
     │
     ▼
State / Result
     │
     ▼
Platform UI
```

The platform UI layer owns presentation concerns.

---

# 13. Threading Is a Runtime Concern

A common misconception is:

> "If the Kotlin code is shared, threading works exactly the same."

The better mental model is:

```text
Shared Concurrency API
          │
          ▼
Platform Runtime
          │
     ┌────┴────┐
     ▼         ▼
 Android       iOS
```

The source-level abstraction can be shared.

Runtime behavior still needs to be understood for each target.

---

# 14. Memory Management

Memory management is another important runtime difference.

Android applications operate in a managed runtime environment with garbage collection.

iOS native applications use Apple's native memory management model, while Kotlin/Native has its own memory-management implementation.

The practical lesson is:

```text
Same Kotlin Source
       ≠
Identical Memory Runtime
```

KMP developers should understand that object lifetime and runtime behavior need to be considered across targets.

---

# 15. Object Lifetime

Consider:

```kotlin
class SessionManager {
    private var token: String? = null
}
```

The object exists while something holds a reference to it.

The runtime determines when the object can be reclaimed.

This becomes particularly important for:

```text
Long-lived services
Caches
Observers
Callbacks
Coroutines
Platform references
```

A shared object that accidentally retains platform resources can create lifecycle problems.

---

# 16. Avoid Retaining Platform Objects

For example, avoid designing shared business logic around long-lived references to platform-specific objects.

Bad conceptual design:

```text
Shared Singleton
      │
      ▼
Android Context
```

or:

```text
Shared Singleton
      │
      ▼
UIViewController
```

A better design is:

```text
Shared Service
      │
      ▼
Small Abstraction
      ▲
      │
Platform Implementation
```

The shared layer should not own the lifecycle of UI objects.

---

# 17. Lifecycle Differences

Android has concepts such as:

```text
Activity
Fragment
ViewModel
Lifecycle
Process
```

iOS has concepts such as:

```text
UIViewController
SwiftUI View
Scene
Application lifecycle
```

These are not identical.

Therefore, avoid assuming:

```text
Android lifecycle = iOS lifecycle
```

Instead, model shared logic around product state and explicit lifecycle boundaries.

---

# 18. A Better Lifecycle Model

Rather than sharing:

```text
Activity lifecycle
```

share:

```text
Feature lifecycle
```

For example:

```text
Feature Created
      │
      ▼
Start Loading
      │
      ▼
Display Data
      │
      ▼
Pause / Stop Observation
      │
      ▼
Dispose
```

The Android and iOS layers can map their native lifecycle events to these shared operations.

---

# 19. Shared ViewModels

A shared ViewModel-like component can expose:

```text
State
Actions
Events
```

For example:

```text
                 FeatureViewModel
                        │
              ┌─────────┼─────────┐
              ▼         ▼         ▼
             State     Action    Event
              │
        ┌─────┴─────┐
        ▼           ▼
     Android        iOS
        UI           UI
```

This can be useful when product behavior is identical.

But the lifecycle ownership must still be clear.

---

# 20. Flow at Runtime

KMP applications often use:

```kotlin
StateFlow
SharedFlow
Flow
```

For example:

```kotlin
val state: StateFlow<LoginState>
```

At runtime:

```text
Repository
    │
    ▼
ViewModel / Presenter
    │
    ▼
StateFlow
    │
 ┌──┴──┐
 ▼     ▼
Android iOS
 UI     UI
```

The shared flow can represent the state.

Each platform consumes it according to its own UI framework.

---

# 21. Flow Collection Is a Boundary

The producer may be shared:

```text
Shared StateFlow
```

The collection mechanism can be platform-specific.

For example:

```text
Shared StateFlow
      │
      ├── Android collector
      │
      └── iOS collector
```

This is often a clean separation.

The shared layer owns:

```text
What the state means.
```

The platform UI owns:

```text
How the state is observed and rendered.
```

---

# 22. Events and State

A useful runtime architecture distinguishes:

```text
State
```

from:

```text
One-time Events
```

For example:

```text
State:
LoginState
```

and:

```text
Events:
ShowError
NavigateHome
OpenVerification
```

Conceptually:

```text
                Shared Feature
                     │
             ┌───────┴───────┐
             ▼               ▼
           State           Events
             │               │
             ▼               ▼
        Android / iOS UI
```

This reduces the temptation to put navigation and UI operations directly into shared business logic.

---

# 23. Runtime Dependencies

A shared service may depend on:

```text
NetworkClient
Storage
Logger
Analytics
Clock
UUID Generator
```

These dependencies can be provided at runtime.

Conceptually:

```text
                Shared Service
                     │
        ┌────────────┼────────────┐
        ▼            ▼            ▼
      Network      Storage      Logger
        │            │            │
    Platform      Platform     Platform
```

Dependency injection makes these runtime relationships explicit.

---

# 24. Platform Services

Suppose the application needs a clock.

Instead of:

```kotlin
System.currentTimeMillis()
```

everywhere, define a capability:

```kotlin
interface Clock {
    fun now(): Long
}
```

Then:

```text
Shared Logic
      │
      ▼
     Clock
      ▲
      │
 ┌────┴────┐
 ▼         ▼
Android    iOS
```

This makes deterministic testing easier as well.

---

# 25. Time Is a Runtime Dependency

Time is often overlooked.

Business logic such as:

```text
Token expired
Offer available
Session timeout
Subscription expired
```

depends on time.

If the shared layer directly accesses platform time everywhere, testing becomes harder.

A clock abstraction gives:

```text
Production
    │
    ▼
Real Clock

Testing
    │
    ▼
Fake Clock
```

The runtime supplies the appropriate implementation.

---

# 26. Randomness Is Another Runtime Dependency

The same principle applies to:

```text
UUID
Random numbers
Device identifiers
Cryptographic operations
```

If business logic requires randomness, abstract the capability where appropriate.

```text
Shared Logic
      │
      ▼
RandomProvider
      ▲
      │
 ┌────┴────┐
 ▼         ▼
Android    iOS
```

This improves portability and testability.

---

# 27. Networking at Runtime

A typical KMP runtime network flow looks like:

```text
UI
 │
 ▼
Use Case
 │
 ▼
Repository
 │
 ▼
HTTP Client
 │
 ▼
Network
 │
 ▼
Backend
```

The same logical flow can run on both platforms.

But the actual network engine may have target-specific behavior or implementations.

Therefore:

```text
Shared API Contract
        │
        ▼
Multiplatform HTTP Client
        │
        ▼
Platform Network Stack
```

The abstraction hides unnecessary platform details.

---

# 28. Runtime Errors

Errors are another important architectural concern.

A platform API may produce:

```text
Platform-specific error
```

The shared layer should often translate it into:

```text
Domain-level error
```

For example:

```text
Network Timeout
      │
      ▼
Repository
      │
      ▼
NetworkUnavailable
      │
      ▼
Shared State
```

The UI can then decide how to display it.

---

# 29. Don't Share Platform Error Types

Avoid exposing platform-specific errors deep into business logic.

For example:

```text
NSError
```

or:

```text
Android-specific exception
```

should not become the primary contract of shared domain logic.

Prefer:

```kotlin
sealed interface DomainError
```

with meaningful application-level states.

Conceptually:

```text
Platform Error
      │
      ▼
Error Mapper
      │
      ▼
Domain Error
      │
      ▼
Shared Logic
```

---

# 30. Runtime and Serialization

Shared data often crosses boundaries:

```text
JSON
  ↓
DTO
  ↓
Domain Model
  ↓
UI State
```

For example:

```text
Backend Response
      │
      ▼
Serialization
      │
      ▼
DTO
      │
      ▼
Mapper
      │
      ▼
Domain Model
```

If serialization is shared, the runtime representation should remain platform-neutral.

This keeps network contracts consistent.

---

# 31. Runtime Caching

Caching is another area where shared architecture can be useful.

```text
Repository
    │
 ┌──┴──┐
 ▼     ▼
Cache  Network
```

The repository decides:

```text
Should I use cached data?
Should I refresh?
What happens when the network fails?
```

The actual storage implementation may be platform-specific or use a multiplatform library.

---

# 32. Offline-First Runtime Behavior

A shared repository can implement:

```text
Read local data
      │
      ▼
Show cached state
      │
      ▼
Fetch remote data
      │
      ▼
Update cache
      │
      ▼
Emit new state
```

This behavior can often be shared.

The platform UI simply observes:

```text
Loading
Content
Error
Refreshing
```

This is a strong example of valuable code sharing.

---

# 33. Runtime Architecture for Offline Data

```text
                 UI
                  │
                  ▼
             Shared State
                  │
                  ▼
              Repository
             ┌────┴────┐
             ▼         ▼
           Cache     Network
             │         │
             └────┬────┘
                  ▼
             Domain Model
```

The platform boundary remains around the infrastructure implementation.

---

# 34. Background Work

Background execution is one area where platform differences become significant.

Android has:

```text
WorkManager
Foreground Services
Alarms
```

iOS has different mechanisms for:

```text
Background tasks
Background refresh
Push-triggered work
```

Therefore, the business requirement can be shared:

```text
Synchronize Orders
```

while scheduling remains platform-specific.

```text
             Sync Requirement
                    │
          ┌─────────┴─────────┐
          ▼                   ▼
       Android               iOS
     Scheduler            Scheduler
          │                   │
          └─────────┬─────────┘
                    ▼
              Shared Sync Logic
```

---

# 35. Push Notifications

Push notifications are another example.

Shared code may understand:

```text
NewMessageReceived
OrderStatusChanged
PaymentCompleted
```

But registration and delivery are platform responsibilities.

```text
Shared Event
      │
      ▼
Platform Notification Layer
      │
 ┌────┴────┐
 ▼         ▼
Android    iOS
```

Don't force platform notification APIs into common business logic.

---

# 36. Application Startup

Startup is also platform-specific.

Android might begin with:

```text
Application
    ↓
Activity
    ↓
Screen
```

iOS might begin with:

```text
App
    ↓
Scene
    ↓
View
```

But both applications may initialize:

```text
Network
Database
Analytics
Authentication
Configuration
```

The initialization strategy can be shared at the dependency level while the actual application entry point remains platform-specific.

---

# 37. Runtime Initialization

A useful structure is:

```text
Platform Application
       │
       ▼
Create Dependencies
       │
       ▼
Initialize Shared Module
       │
       ▼
Start Feature
```

For example:

```text
Android
  │
  ▼
Android DI
  │
  ▼
Shared Dependencies
  │
  ▼
Application

iOS
  │
  ▼
iOS DI
  │
  ▼
Shared Dependencies
  │
  ▼
Application
```

This keeps startup ownership clear.

---

# 38. Singleton Services

Singletons can be useful.

They can also create difficult runtime problems.

For example:

```text
Global Singleton
     │
     ├── Context
     ├── Activity
     ├── ViewController
     └── UI State
```

This can create:

```text
Memory Retention
Lifecycle Problems
Testing Problems
Unexpected State
```

Prefer controlled dependency ownership.

---

# 39. Runtime Ownership

For every long-lived object, ask:

> **Who owns this object?**

For example:

```text
Application
    │
    ├── NetworkClient
    ├── Database
    └── Analytics
```

while:

```text
Feature
    │
    └── FeatureState
```

and:

```text
Screen
    │
    └── UI-specific state
```

Ownership should match lifetime.

---

# 40. Runtime Lifecycle Diagram

A useful model:

```text
Application Start
       │
       ▼
Dependency Initialization
       │
       ▼
Feature Creation
       │
       ▼
State Collection
       │
       ▼
User Interaction
       │
       ▼
Business Logic
       │
       ▼
Data / Platform Services
       │
       ▼
State Update
       │
       ▼
UI Rendering
       │
       ▼
Feature Disposal
```

The same conceptual lifecycle can exist on Android and iOS even though the platform events differ.

---

# 41. Runtime Cancellation

Asynchronous work must also be cancelled correctly.

For example:

```text
Screen disappears
      │
      ▼
Feature no longer needed
      │
      ▼
Cancel work
```

A good coroutine structure allows the lifetime of asynchronous work to follow the owner.

Conceptually:

```text
Feature Scope
      │
      ├── Request A
      ├── Request B
      └── Observation
```

When the feature scope ends:

```text
Feature Scope
      │
      ▼
Cancel Children
```

This prevents unnecessary background work and potential leaks.

---

# 42. Runtime and Structured Concurrency

Structured concurrency encourages:

```text
Parent
  │
  ├── Child
  ├── Child
  └── Child
```

instead of:

```text
Global Coroutine
       │
       ▼
Unknown Lifetime
```

This becomes especially important in shared code because the same business logic may execute inside different application lifecycle models.

---

# 43. Runtime Logging

When debugging KMP, logs should identify:

```text
Platform
Feature
Operation
Result
Error
```

For example:

```text
[iOS] CheckoutRepository: loading order
[Android] CheckoutRepository: loading order
```

The implementation can remain shared while the logging backend remains platform-aware.

---

# 44. Runtime Observability

Production KMP systems should consider:

```text
Crash reporting
Analytics
Performance
Network metrics
Business events
```

But the shared layer should not become tightly coupled to a specific platform SDK.

A useful architecture is:

```text
Shared Telemetry API
         │
   ┌─────┴─────┐
   ▼           ▼
Android      iOS
Adapter      Adapter
```

---

# 45. Runtime Security Boundaries

Security-sensitive capabilities often belong at the platform boundary.

Examples:

```text
Secure Storage
Biometric Authentication
Keychain
Keystore
Device Security
```

Shared business logic can request a capability:

```text
SecureTokenStore
```

while the platform implementation uses the appropriate secure storage mechanism.

```text
             Shared
        SecureTokenStore
               │
       ┌───────┴───────┐
       ▼               ▼
    Android            iOS
    Keystore         Keychain
```

The shared layer should not pretend the platforms have identical security APIs.

---

# 46. Runtime Performance

Sharing code does not automatically mean identical performance.

For example:

```text
Same Algorithm
      │
      ▼
Different Runtime
      │
      ▼
Different Performance Characteristics
```

Performance should therefore be measured on actual targets.

Don't assume:

```text
Android performance = iOS performance
```

because the source code looks identical.

---

# 47. Runtime Profiling

When performance matters, profile the actual platform application.

Android tools can investigate:

```text
CPU
Memory
Network
Rendering
Startup
```

iOS tooling can investigate:

```text
CPU
Memory
Energy
Network
Startup
```

The shared source gives you reuse.

It does not remove the need for platform-specific performance analysis.

---

# 48. Runtime Compatibility

A shared library can compile successfully but still expose runtime problems.

Examples:

```text
Unsupported platform behavior
Incorrect lifecycle handling
Concurrency assumptions
Memory retention
Platform API differences
Dependency incompatibility
```

Therefore:

```text
Build Success
      ≠
Runtime Correctness
```

This is one of the most important lessons in multiplatform development.

---

# 49. Runtime Testing

A good KMP test strategy has multiple levels.

```text
                Tests
                  │
       ┌──────────┼──────────┐
       ▼          ▼          ▼
    Common     Android       iOS
     Tests      Tests       Tests
```

Common tests verify:

```text
Business behavior
State transitions
Mapping
Validation
Use cases
```

Platform tests verify:

```text
Platform integration
Lifecycle
Permissions
Native APIs
```

---

# 50. Architecture Determines Testability

Compare:

```text
Business Logic
      │
      ▼
Android Context
```

with:

```text
Business Logic
      │
      ▼
Platform Abstraction
      ▲
      │
Fake Implementation
```

The second architecture is much easier to test.

For example:

```text
FakeClock
FakeStorage
FakeNetwork
FakeAnalytics
```

can be injected into shared logic.

---

# 51. Runtime Dependency Graph

A useful mental model is:

```text
                    Feature
                       │
                       ▼
                    UseCase
                       │
              ┌────────┴────────┐
              ▼                 ▼
          Repository          Clock
              │                 │
       ┌──────┴──────┐          │
       ▼             ▼          ▼
     Cache         Network   Platform
```

At runtime, these dependencies form an object graph.

The object graph is created by the application.

---

# 52. The Runtime Object Graph

For Android:

```text
Android Application
        │
        ▼
Dependency Graph
        │
 ┌──────┼─────────┐
 ▼      ▼         ▼
Network Cache   Analytics
 │       │         │
 ▼       ▼         ▼
Android Platform Services
```

For iOS:

```text
iOS Application
        │
        ▼
Dependency Graph
        │
 ┌──────┼─────────┐
 ▼      ▼         ▼
Network Cache   Analytics
 │       │         │
 ▼       ▼         ▼
Apple Platform Services
```

The shared business layer can consume the same conceptual dependencies.

---

# 53. Runtime Boundaries Make Migration Easier

Suppose an existing Android application has:

```text
Android UI
Android ViewModel
Android Repository
Android Database
Android Network
```

A gradual KMP migration can look like:

```text
Phase 1
Android UI
     │
     ▼
Shared Domain
     │
     ▼
Android Data
```

Then:

```text
Phase 2
Android UI
     │
     ▼
Shared Domain
     │
     ▼
Shared Data
```

Then iOS can consume:

```text
iOS UI
  │
  ▼
Shared Domain
  │
  ▼
Shared Data
```

This is often more realistic than rewriting the entire application at once.

---

# 54. Runtime Architecture Enables Incremental Adoption

A useful migration path:

```text
Existing Android
       │
       ▼
Extract Business Logic
       │
       ▼
Move to commonMain
       │
       ▼
Share Data Layer
       │
       ▼
Add iOS Application
       │
       ▼
Expand Shared Capabilities
```

The architecture provides the seams needed for incremental migration.

---

# 55. Runtime and Feature Ownership

A mature KMP team should be able to answer:

```text
Who owns this code?
```

For example:

```text
Payment Rules
→ Shared Domain

Payment API
→ Shared Data

Android Payment UI
→ Android Team

iOS Payment UI
→ iOS Team

Apple Pay Integration
→ iOS Team

Google Pay Integration
→ Android Team
```

This avoids turning the shared module into a dumping ground.

---

# 56. The Platform Is Still Important

KMP should not be understood as:

```text
Ignore Android
Ignore iOS
```

The opposite is true.

A strong KMP developer understands:

```text
Kotlin
+
Android
+
iOS
+
Architecture
+
Runtime
```

The better you understand the platform boundaries, the better you can decide what should be shared.

---

# 57. The Most Important Runtime Principle

> [!IMPORTANT]
> **Shared code should own shared behavior. The platform should own platform behavior.**

A useful diagram is:

```text
                 Shared Behavior
                       │
          ┌────────────┴────────────┐
          ▼                         ▼
      Android Adapter            iOS Adapter
          │                         │
          ▼                         ▼
    Android Runtime          iOS Runtime
```

The shared layer should not try to become a fake Android or fake iOS runtime.

It should provide portable product behavior.

---

# 58. Runtime Is Where Architecture Becomes Real

During design, architecture looks like:

```text
Boxes
Arrows
Interfaces
Modules
```

At runtime, those decisions become:

```text
Objects
Threads
Coroutines
State
Network Calls
Memory
Lifecycle
Platform APIs
```

That is why runtime architecture matters.

A beautiful diagram is not enough.

The objects and dependencies must behave correctly when the application is actually running.

---

# 59. Complete Runtime Mental Model

The full runtime picture can be represented as:

```text
                              USER
                               │
                               ▼
                         PLATFORM UI
                    ┌──────────┴──────────┐
                    ▼                     ▼
                 Android                  iOS
                    │                     │
                    └──────────┬──────────┘
                               ▼
                       Shared Presentation
                               │
                               ▼
                           Use Cases
                               │
                               ▼
                            Domain
                               │
                               ▼
                           Repository
                               │
                    ┌──────────┴──────────┐
                    ▼                     ▼
                  Cache                Network
                    │                     │
                    └──────────┬──────────┘
                               ▼
                       Platform Services
                    ┌──────────┴──────────┐
                    ▼                     ▼
                Android APIs          Apple APIs
```

This is the runtime architecture to keep in mind when designing a production KMP application.

---

# Chapter Takeaways

> [!TIP]
> **KMP shares implementation logic, but that logic still executes inside separate platform runtimes.**

Remember:

1. Same source code does not mean the same runtime.
2. Android and iOS remain separate application environments.
3. Shared Kotlin code executes inside the host application's runtime.
4. Android and iOS have different platform capabilities and lifecycle models.
5. UI can remain platform-specific while business behavior is shared.
6. State is often easier and safer to share than UI.
7. Coroutines provide a shared concurrency model, but runtime behavior still depends on the target.
8. Threading assumptions should not leak into common business logic.
9. Object lifetime matters across platforms.
10. Avoid retaining platform UI objects from long-lived shared services.
11. Lifecycle should be modeled around feature behavior rather than copied from one platform.
12. Flow and StateFlow can provide shared state while platform layers own collection and rendering.
13. Platform capabilities should be exposed through abstractions.
14. Time, randomness, storage, logging, analytics, and device services can be treated as runtime dependencies.
15. Platform-specific errors should be translated into meaningful domain errors where appropriate.
16. Background execution and notifications usually require platform-specific integration.
17. Dependency injection makes runtime boundaries explicit.
18. Singleton ownership must be designed carefully.
19. Structured concurrency helps control asynchronous work.
20. Runtime success must be verified independently from build success.
21. Performance should be measured on real target platforms.
22. Common tests validate shared behavior; platform tests validate platform integration.
23. A good runtime architecture creates clear ownership and failure boundaries.
24. KMP works best when shared code owns shared product behavior and platform code owns platform behavior.

---

# Final Mental Model

When thinking about KMP runtime, don't think:

```text
"One Kotlin application running everywhere."
```

Think:

```text
                         Shared Kotlin Logic
                                  │
                    ┌─────────────┴─────────────┐
                    ▼                           ▼
             Android Application          iOS Application
                    │                           │
                    ▼                           ▼
             Android Runtime              iOS Runtime
                    │                           │
                    ▼                           ▼
             Android APIs                 Apple APIs
```

And at the architectural level:

```text
Platform UI
     │
     ▼
Shared Presentation
     │
     ▼
Shared Domain
     │
     ▼
Shared Data
     │
     ▼
Platform Abstractions
     │
     ▼
Platform Implementations
```

The final principle is simple:

> **Share the behavior that represents your product. Keep the behavior that represents the platform at the platform boundary.**

That is what allows the same KMP codebase to participate naturally in two different runtime worlds without pretending that Android and iOS are the same platform.
