# Chapter 7 — Source Sets Deep Dive

## Part 1 — `commonMain`

> **`commonMain` is where Kotlin Multiplatform starts to become an architectural tool rather than simply a way to share code. It is the source set that defines production code intended to be compiled for multiple supported platforms.**

A KMP project often begins with a simple question:

> **What code can I write once and use on multiple platforms?**

The answer is not:

```text
Put everything into one folder.
```

The better answer is:

```text
Put platform-independent code in common source sets.
Put platform-specific behavior in platform source sets.
```

At the center of that model is:

```text
commonMain
```

A simplified project may look like:

```text
shared/
└── src/
    ├── commonMain/
    │   └── kotlin/
    ├── androidMain/
    │   └── kotlin/
    └── iosMain/
        └── kotlin/
```

The important idea is:

```text
commonMain
    │
    ├── Android
    ├── iOS
    └── other configured targets
```

The code in `commonMain` is written against APIs available to the common compilation.

---

# 1. What Is a Source Set?

A source set is a logical collection of source code and related configuration.

Examples include:

```text
commonMain
commonTest
androidMain
iosMain
```

A source set can have:

```text
Source files
Dependencies
Resources
Language settings
Relationships with other source sets
```

Conceptually:

```text
Source Set
   │
   ├── Kotlin source
   ├── Dependencies
   ├── Resources
   └── Compilation relationships
```

KMP uses source sets to express which code belongs to which compilation context.

---

# 2. Why Source Sets Exist

Consider an application with:

```text
Business rules
Networking
Data models
Repositories
Caching
Android UI
iOS UI
```

Not all of these concerns have the same portability.

A useful separation is:

```text
                    Application
                        │
          ┌─────────────┴─────────────┐
          ▼                           ▼
    Shared Concerns              Platform Concerns
          │                           │
          ▼                           ▼
     commonMain              androidMain / iosMain
```

`commonMain` gives the shared side of this boundary a formal place in the project.

---

# 3. The Meaning of `commonMain`

The name can be understood as:

```text
common + main
```

### `common`

The source set belongs to the common multiplatform compilation model.

### `main`

It contains production code rather than test code.

Therefore:

> **`commonMain` is the production source set for code intended to be shared across the configured targets.**

This does not mean every Kotlin or platform API is automatically available there.

That distinction is critical.

---

# 4. `commonMain` Is Not Android Code That Happens to Run on iOS

An Android developer may naturally reach for:

```kotlin
android.content.Context
android.net.Uri
android.os.Bundle
android.view.View
```

These APIs are Android-specific.

They cannot simply become `commonMain` APIs because the file was moved into a shared directory.

The source-set location does not magically make an API multiplatform.

Instead:

```text
commonMain
      │
      ▼
Commonly available APIs
```

must be respected by the common compilation.

---

# 5. A Simple `commonMain` Example

```kotlin
class UserNameFormatter {

    fun format(firstName: String, lastName: String): String {
        return "$firstName $lastName"
    }
}
```

This is a good candidate for `commonMain`.

It depends only on ordinary Kotlin types and language features.

The same applies to:

```kotlin
class OrderCalculator {

    fun calculateTotal(
        subtotal: Double,
        tax: Double
    ): Double {
        return subtotal + tax
    }
}
```

Both Android and iOS can use the same implementation.

---

# 6. What Belongs in `commonMain`?

Typical candidates include:

```text
Domain models
Business rules
Validation
Use cases
Repository contracts
State models
Serialization models
Shared networking abstractions
Cross-platform data transformations
Pure Kotlin utilities
Shared application state
```

The exact boundary depends on the application architecture.

A useful question is:

> **Does this code require a platform-specific API to perform its job?**

If the answer is no, it is often a candidate for `commonMain`.

---

# 7. What Does Not Belong in `commonMain`?

Typical platform-specific concerns include:

```text
Android Activity
Android Context
Android Views
iOS UIKit types
Apple-specific frameworks
Android-specific storage APIs
Platform-specific lifecycle APIs
Platform-specific sensors
Platform-specific permissions
```

These belong in platform-specific source sets or behind an abstraction.

For example:

```text
commonMain
    │
    └── interface PlatformStorage
                 ▲
                 │
        ┌────────┴────────┐
        │                 │
   androidMain         iosMain
```

---

# 8. `commonMain` and the Compiler

`commonMain` is compiled in the context of the Kotlin Multiplatform model.

Therefore:

```kotlin
import android.content.Context
```

inside `commonMain` is not valid merely because Android is one of the targets.

Common code must remain compatible with the common compilation model.

---

# 9. Common Code Is Compiled for Targets

A simplified model is:

```text
                    commonMain
                        │
            ┌───────────┼───────────┐
            ▼           ▼           ▼
         Android       iOS       Other Targets
            │           │
            ▼           ▼
       Platform Compilation
```

The exact compiler and source-set dependency model is more detailed, but the key idea is:

> **Common source participates in the compilations of the configured targets.**

---

# 10. `commonMain` Is a Compilation Input

Think of `commonMain` as more than a directory.

It is a compilation input:

```text
commonMain
     │
     ▼
Common compilation model
     │
     ▼
Target-specific compilations
```

That is why moving a file from:

```text
androidMain
```

to:

```text
commonMain
```

can immediately produce compilation errors.

The available API surface has changed.

---

# 11. The `src/commonMain` Convention

A conventional KMP layout is:

```text
shared/
└── src/
    └── commonMain/
        └── kotlin/
            └── com/example/shared/
                ├── User.kt
                ├── Order.kt
                └── OrderRepository.kt
```

The Kotlin Multiplatform Gradle plugin understands this source-set structure.

---

# 12. Package Names vs Source Sets

A package and a source set solve different problems.

```text
Package
→ Logical organization

Source Set
→ Compilation context
```

For example:

```kotlin
package com.example.shared.domain

data class User(
    val id: String,
    val name: String
)
```

The package says where the class logically belongs.

`commonMain` says where the class participates in the build.

---

# 13. `commonMain` and Architecture

A KMP project can place the core of its architecture in `commonMain`:

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
├── presentation
│   └── state
│
└── core
    └── utility
```

Platform-specific source sets then provide platform integrations where required.

---

# 14. Common Domain Models

Data models are often excellent `commonMain` candidates.

```kotlin
data class Product(
    val id: String,
    val name: String,
    val price: Double
)
```

Both Android and iOS can consume the same model.

```text
                    Product
                       │
              ┌────────┴────────┐
              ▼                 ▼
           Android              iOS
```

There is usually no reason to create separate platform models when the concept itself is platform-neutral.

---

# 15. Common Validation

Validation is another strong candidate:

```kotlin
class EmailValidator {

    fun isValid(value: String): Boolean {
        return value.contains("@")
    }
}
```

The implementation can be shared while each platform decides how to display the result.

---

# 16. Common Use Cases

A use case can live in `commonMain`:

```kotlin
class GetUserUseCase(
    private val repository: UserRepository
) {
    suspend operator fun invoke(id: String): User {
        return repository.getUser(id)
    }
}
```

The use case does not need to know whether the caller is:

```text
Android UI
```

or:

```text
iOS UI
```

---

# 17. Repository Interfaces

A repository contract is commonly appropriate for `commonMain`:

```kotlin
interface UserRepository {

    suspend fun getUser(id: String): User
}
```

If the implementation is platform-independent, it can also live in `commonMain`.

If it requires platform services:

```text
commonMain
    │
    └── UserRepository
            ▲
            │
      ┌─────┴─────┐
      │           │
androidMain    iosMain
```

can provide the implementations.

---

# 18. `commonMain` Does Not Mean Everything Must Be Shared

KMP is not:

```text
Share 100% of the code.
```

It is:

```text
Share what makes sense.
Keep platform-specific behavior where it belongs.
```

For example:

```text
Business rules        → commonMain
Data models           → commonMain
Repository contract   → commonMain
Platform storage      → platform source set
Platform UI           → platform layer
```

The goal is useful sharing, not maximum sharing.

---

# 19. Common Dependencies

Dependencies can be declared for `commonMain`:

```kotlin
kotlin {
    sourceSets {
        commonMain.dependencies {
            implementation(libs.kotlinx.coroutines.core)
        }
    }
}
```

The dependency must itself support the common source set and the intended targets.

An Android-only library should not be placed in `commonMain`.

Instead:

```kotlin
androidMain.dependencies {
    implementation(libs.android.specific.library)
}
```

keeps the dependency on the correct side of the boundary.

---

# 20. Common Coroutines

Multiplatform-compatible coroutine APIs are a common example of shared functionality.

A common class can use:

```kotlin
suspend fun loadUser(): User
```

or:

```kotlin
Flow<User>
```

when the project uses a compatible coroutine dependency.

Android and iOS can then consume the same asynchronous behavior.

---

# 21. Common Serialization

Shared models may use a multiplatform-compatible serialization library:

```kotlin
@Serializable
data class User(
    val id: String,
    val name: String
)
```

The important requirement is:

```text
The serialization stack must support the common source set and intended targets.
```

---

# 22. Common Networking

A networking implementation can be shared when the selected networking stack supports the required KMP targets.

Conceptually:

```text
commonMain
    │
    ├── API service
    ├── DTOs
    ├── Mappers
    └── Repository
```

The HTTP engine can vary by platform:

```text
common networking API
        │
        ├── Android engine
        └── iOS/Darwin engine
```

This keeps high-level networking logic common.

---

# 23. The Abstraction Boundary

When common code needs a platform-specific capability, introduce a boundary.

```kotlin
interface DeviceInfo {
    val model: String
    val osVersion: String
}
```

Then:

```text
commonMain
    │
    └── DeviceInfo
           ▲
           │
      implementations
```

The common layer depends on the capability rather than the platform API.

---

# 24. `expect` and `actual`

KMP also provides:

```text
expect
actual
```

A common declaration can define what is required:

```kotlin
expect class PlatformInfo {
    val name: String
}
```

A platform source set provides the corresponding implementation:

```kotlin
actual class PlatformInfo {
    actual val name: String = "Android"
}
```

Another target can provide its own `actual` implementation.

The mental model is:

```text
commonMain
     │
     └── expect
           │
           ▼
     Platform contract
           │
      ┌────┴────┐
      ▼         ▼
   actual     actual
 Android       iOS
```

---

# 25. Do Not Use `expect/actual` for Everything

`expect/actual` is powerful, but it should not automatically be used for every platform difference.

Sometimes a normal interface is simpler:

```kotlin
interface Clock {
    fun now(): Instant
}
```

Dependency injection can provide the implementation.

Choose the mechanism that best expresses the architectural boundary.

---

# 26. `commonMain` and Dependency Injection

Dependency injection works naturally with common code:

```kotlin
class UserRepository(
    private val api: UserApi
)
```

The common class depends on an abstraction.

Platform-specific composition can provide the implementation:

```text
Android App
    │
    ▼
Dependency graph
    │
    ▼
commonMain

iOS App
    │
    ▼
Dependency graph
    │
    ▼
commonMain
```

---

# 27. Common State Models

State models are often excellent `commonMain` candidates:

```kotlin
sealed interface UserState {

    data object Loading : UserState

    data class Success(
        val user: User
    ) : UserState

    data class Error(
        val message: String
    ) : UserState
}
```

The state can be consumed by Android, iOS, or a shared UI layer.

---

# 28. Common Presentation Logic

Some presentation logic can be shared:

```text
State
Events
Reducers
Validators
UI-independent presentation models
```

But platform UI APIs should not be pulled into common code merely to force sharing.

---

# 29. `commonMain` and UI

Whether UI belongs in `commonMain` depends on the UI technology.

With native Android and native iOS UI:

```text
Android UI → Android layer
iOS UI     → iOS/native layer
```

With Compose Multiplatform, shared UI can also live in common source sets.

The important distinction is:

```text
KMP shared business/data code
```

versus:

```text
Shared UI through a multiplatform UI framework
```

They are related but not identical concepts.

---

# 30. Common File Storage

File access often crosses a platform boundary because Android and iOS have different storage models.

A common abstraction can express the requirement:

```kotlin
interface FileStorage {
    suspend fun save(name: String, data: ByteArray)
    suspend fun load(name: String): ByteArray?
}
```

Platform implementations can handle the actual filesystem APIs.

---

# 31. Secure Storage

Secure storage is another clear boundary.

The common layer might define:

```kotlin
interface SecureStorage {

    suspend fun save(key: String, value: String)

    suspend fun read(key: String): String?
}
```

Platform-specific implementations can use the appropriate secure storage mechanism.

The common layer should not depend directly on Android or Apple security APIs.

---

# 32. Permissions and Hardware

Permissions and hardware are usually platform-specific:

```text
Camera
Location
Bluetooth
NFC
Biometrics
Sensors
Notifications
```

Common code can express the application capability:

```text
"Location is required."
```

but platform code should handle the actual OS APIs.

---

# 33. Source Set Hierarchy

A simplified hierarchy is:

```text
commonMain
    │
    ├── androidMain
    │
    └── iosMain
```

The child source sets can access code from their parents.

The reverse is not true.

Therefore:

```text
androidMain → commonMain
iosMain     → commonMain
```

is valid.

But:

```text
commonMain → androidMain
```

would violate the intended dependency direction.

---

# 34. Intermediate Source Sets

Real KMP projects can contain intermediate source sets:

```text
commonMain
    │
    └── appleMain
          │
          ├── iosMain
          ├── macosMain
          └── other Apple targets
```

The exact hierarchy depends on the configured targets and compatibility.

This allows code shared by a family of targets to live at the appropriate level.

---

# 35. Why Intermediate Source Sets Matter

Suppose several Apple targets share the same implementation.

Without an intermediate source set:

```text
iosMain
macosMain
tvosMain
```

may duplicate code.

With an appropriate intermediate source set:

```text
             commonMain
                  │
              appleMain
            /      |                 ▼       ▼       ▼
          iOS    macOS    tvOS
```

the shared Apple behavior has one source of truth.

---

# 36. The Widest Compatible Source Set

A useful rule is:

> **Place code in the widest source set that can correctly support it.**

For example:

```text
All targets
→ commonMain

Apple targets
→ appropriate Apple intermediate source set

One platform
→ platform source set
```

This avoids both duplication and incorrect abstractions.

---

# 37. Source Sets as Capability Boundaries

Think of source sets as capability levels:

```text
commonMain
→ widest portability

Intermediate source set
→ group-specific capabilities

Platform source set
→ platform-specific capabilities
```

The deeper the source set, the more platform-specific assumptions it can safely make.

---

# 38. CommonMain and Clean Architecture

A typical architecture can look like:

```text
                 commonMain
                     │
        ┌────────────┼────────────┐
        ▼            ▼            ▼
      Domain        Data       Presentation
        │            │            │
        └────────────┼────────────┘
                     ▼
                Abstractions
                     ▲
                     │
             Platform Integration
```

The common layer defines portable behavior.

Platform layers provide platform-specific capabilities.

---

# 39. Capability, Not Platform

Do not ask:

```text
"How do I expose Android to commonMain?"
```

Ask:

```text
"What capability does commonMain actually need?"
```

For example:

```text
Needs storage
→ Storage abstraction

Needs current time
→ Clock abstraction

Needs logging
→ Logger abstraction

Needs device information
→ DeviceInfo abstraction
```

This usually produces cleaner architecture.

---

# 40. Common Error Models

Errors can also be represented in common code:

```kotlin
sealed interface AppError {

    data object Network : AppError

    data object Unauthorized : AppError

    data object NotFound : AppError

    data class Unknown(
        val message: String
    ) : AppError
}
```

The platform UI can translate these common errors into platform-specific presentation.

---

# 41. Common Logging

Instead of directly depending on a platform logger:

```kotlin
interface Logger {
    fun debug(message: String)
    fun error(message: String, throwable: Throwable? = null)
}
```

The application composition layer can provide the implementation.

This keeps business logic portable.

---

# 42. Common Analytics

Similarly:

```kotlin
interface Analytics {
    fun track(event: AnalyticsEvent)
}
```

The common layer decides:

```text
What event occurred?
```

The platform or vendor-specific layer decides:

```text
How is it recorded?
```

---

# 43. Common Navigation Intent

Navigation can sometimes be modeled without embedding UI framework types:

```kotlin
sealed interface NavigationTarget {

    data object Home : NavigationTarget

    data class Details(
        val id: String
    ) : NavigationTarget
}
```

Android or iOS can translate this into their own navigation mechanism.

---

# 44. Common Configuration

Business configuration can be represented in common code:

```kotlin
interface AppConfig {
    val environment: Environment
    val apiBaseUrl: String
}
```

The source of the configuration can differ between platforms.

Platform-specific build settings should remain outside the common domain model.

---

# 45. Common Security Logic

Security-sensitive business rules can often be shared:

```text
Token validation rules
Session state
Authorization decisions
Input validation
Security policies
```

But secure storage, biometrics, and OS security frameworks should remain behind platform boundaries.

---

# 46. Common Tests

Production common code belongs in:

```text
commonMain
```

Common tests belong in:

```text
commonTest
```

The relationship is:

```text
commonMain
    │
    ▼
commonTest
```

For shared business behavior, this allows one common test suite to verify the common implementation.

---

# 47. Why Common Tests Matter

Suppose:

```kotlin
class PriceCalculator {
    fun total(price: Double, tax: Double): Double {
        return price + tax
    }
}
```

You do not need duplicate business-rule tests simply because the same class is used by Android and iOS.

Test the shared behavior in `commonTest`.

Platform-specific behavior can have platform-specific tests where needed.

---

# 48. CommonMain and Code Duplication

Without KMP:

```text
Android business rule
        │
        └── copy
             │
             ▼
        iOS business rule
```

With KMP:

```text
             commonMain
                 │
          Shared business rule
             /                   ▼         ▼
        Android       iOS
```

The benefit is not just fewer lines.

It creates one source of truth.

---

# 49. Sharing Logic vs Sharing UI

KMP can provide substantial sharing without shared UI.

You can have:

```text
Android UI
iOS UI
     │
     ▼
commonMain
```

and share:

```text
Domain
Data
Networking
State
Business Logic
```

This is a practical KMP architecture for teams that want native platform UI.

---

# 50. CommonMain and Compose Multiplatform

Another architecture may use Compose Multiplatform:

```text
commonMain
    │
    ├── Business Logic
    ├── Data
    ├── State
    └── Shared UI
```

The source-set model remains the same.

What changes is the amount of the application that can be shared.

---

# 51. There Is No Magic Percentage

There is no universal rule such as:

```text
"commonMain must contain 80% of the app."
```

The appropriate level of sharing depends on:

```text
Product requirements
Platform differences
UI strategy
Hardware integration
Existing architecture
Team structure
```

The right amount of sharing is an architectural decision.

---

# 52. CommonMain and Future Targets

Suppose a project supports:

```text
Android
iOS
```

and later adds another target.

A clean common layer gives that target a starting point:

```text
commonMain
     │
 ┌───┼────┐
 ▼   ▼    ▼
Android iOS New Target
```

The less platform-specific the common layer is, the easier this expansion can become.

---

# 53. CommonMain as a Stable Core

A mature KMP application can treat `commonMain` as a stable core:

```text
             Platform UI
              /                    /                 Android       iOS
             \         /
              \       /
              commonMain
```

The common core should have clear APIs and limited platform leakage.

---

# 54. Source Sets Are More Than Folders

This distinction is essential.

A source set is not merely:

```text
A directory.
```

It represents a Kotlin/Gradle build concept with:

```text
Source files
Dependencies
Compilation relationships
Target participation
Test relationships
```

That is why moving files between source sets can change compilation behavior.

---

# 55. Migrating Android Code to `commonMain`

Do not simply move files.

A safer process is:

```text
Android implementation
        │
        ▼
Identify platform dependencies
        │
        ▼
Separate common behavior
        │
        ▼
Extract abstractions
        │
        ▼
Move portable code
        │
        ▼
Implement platform pieces
```

For example, this is not common:

```kotlin
class UserRepository(
    private val context: Context,
    private val api: UserApi
)
```

because `Context` is Android-specific.

Instead, extract the capability that the repository actually needs.

---

# 56. Extract Capabilities

Common:

```kotlin
interface UserStorage {
    suspend fun save(user: User)
    suspend fun load(): User?
}
```

Common repository:

```kotlin
class UserRepository(
    private val api: UserApi,
    private val storage: UserStorage
) {
    // shared logic
}
```

Platform composition provides:

```text
AndroidUserStorage
IOSUserStorage
```

The repository remains common.

---

# 57. A Practical Classification

| Concern | Typical Location |
|---|---|
| Domain model | `commonMain` |
| Business rule | `commonMain` |
| Use case | `commonMain` |
| Repository interface | `commonMain` |
| Common repository implementation | `commonMain` when dependencies permit |
| Shared networking | `commonMain` when dependencies support it |
| Android storage | `androidMain` |
| iOS secure storage | iOS/Apple-specific source set |
| Android UI | Android layer |
| Native iOS UI | iOS/native layer |
| Shared Compose UI | `commonMain` when using Compose Multiplatform |
| Common tests | `commonTest` |

The exact placement depends on the target configuration and architecture.

---

# 58. A Simple Decision Tree

```text
                 New Class
                     │
                     ▼
        Does it depend on a platform API?
                /                         Yes            No
              │               │
              ▼               ▼
       Platform source     commonMain
              │
              ▼
      Can the dependency
      be abstracted?
          /               Yes        No
        │           │
        ▼           ▼
    Common API   Keep platform-specific
```

Then ask:

```text
Is it shared by a group of platforms?
```

If yes, an intermediate source set may be more appropriate.

---

# 59. CommonMain Review Checklist

Before merging a change to `commonMain`:

```text
[ ] Does the code compile for every intended target?
[ ] Does it use only common-compatible APIs?
[ ] Are all dependencies available to commonMain?
[ ] Is the business logic genuinely shared?
[ ] Has platform-specific behavior been isolated?
[ ] Is an intermediate source set more appropriate?
[ ] Are public APIs platform-neutral?
[ ] Are tests included in commonTest where appropriate?
[ ] Is the dependency direction correct?
[ ] Could the abstraction be simpler?
```

---

# 60. Final Mental Model

Keep this diagram in mind:

```text
                         KMP MODULE
                             │
                             ▼
                       commonMain
                             │
          ┌──────────────────┼──────────────────┐
          ▼                  ▼                  ▼
       Android            Apple             Other
       Sources            Sources            Targets
          │                  │
          ▼                  ▼
     Platform APIs      Platform APIs
          │                  │
          └──────────┬───────┘
                     ▼
                Application
```

The central rule is:

```text
Common code depends on common concepts.
Platform code handles platform capabilities.
```

---

# Chapter Takeaways

> [!TIP]
> **`commonMain` is not simply a shared folder. It is the common production source set that forms the portable core of a KMP module.**

Remember:

1. A source set is a logical collection of source code and related build configuration.
2. `commonMain` represents common production code.
3. `commonTest` represents common test code.
4. `commonMain` participates in the compilations of configured targets.
5. Putting a file in `commonMain` does not make platform-specific APIs portable.
6. Common code must use APIs and dependencies supported by the common compilation.
7. Domain models are usually strong candidates for `commonMain`.
8. Business rules are usually strong candidates for `commonMain`.
9. Use cases are often appropriate for `commonMain`.
10. Repository interfaces commonly belong in `commonMain`.
11. Repository implementations can also be common when their dependencies are multiplatform-compatible.
12. Platform-specific storage, hardware, permissions, and OS APIs should remain behind platform boundaries.
13. Common code should depend on capabilities rather than platform implementations.
14. `expect/actual` can model platform-specific implementations when appropriate.
15. Interfaces and dependency injection are often useful alternatives to `expect/actual`.
16. `commonMain` should not become a dumping ground for everything that appears to be shared.
17. KMP does not require 100% code sharing.
18. The goal is useful and maintainable sharing.
19. Platform source sets can build upon common source sets.
20. Common code should not depend directly on platform-specific source sets.
21. Source-set hierarchy expresses compatibility and capability.
22. Intermediate source sets can share code among groups of targets.
23. The widest compatible source set is generally the best place for a piece of code.
24. Common dependencies must be compatible with the common source set.
25. Android-only dependencies belong in Android-specific source sets.
26. Shared networking can live in `commonMain` when the networking stack supports the intended targets.
27. Common state models can be consumed by different platform UIs.
28. Shared business logic does not require shared UI.
29. Compose Multiplatform can extend sharing into UI, but that is separate from the basic KMP source-set concept.
30. Common code is often easier to test because it can avoid platform runtime dependencies.
31. `commonMain` changes can affect multiple platforms and therefore have a wider impact surface.
32. Common APIs should speak in domain concepts rather than Android or iOS concepts.
33. `Context`, `Activity`, UIKit types, and other platform types should not leak into common APIs.
34. A common API should expose common types.
35. Platform entry points can construct and inject platform-specific implementations into common code.
36. Source-set boundaries are architectural boundaries as well as build-system concepts.
37. Migrating Android code to `commonMain` requires identifying and isolating platform dependencies rather than simply moving files.
38. Capability extraction is often more useful than exposing an entire platform.
39. `commonMain` can form the stable core of an application shared by Android, iOS, and other targets.
40. There is no universal percentage of code that should be placed in `commonMain`.
41. The right amount of sharing depends on product, architecture, platform differences, and technology choices.
42. The most useful question is not "Can this be shared?" but "Should this be shared at this source-set level?"
43. The best `commonMain` code is portable, focused, testable, and expressed in platform-neutral concepts.
44. **`commonMain` is the center of KMP's shared-code model: share the behavior that is genuinely common, and isolate everything that belongs to a platform.**

---

# Final Thought

The real power of `commonMain` is not that it allows an Android developer to move Kotlin files into a shared directory.

Its real value is architectural.

Instead of building:

```text
Android Business Logic
        +
iOS Business Logic
```

you can build:

```text
              commonMain
                  │
        Shared business behavior
                  │
          ┌───────┴───────┐
          ▼               ▼
       Android            iOS
```

The platform layers remain free to use the APIs and UI technologies that make sense for them.

The common layer remains focused on what the application actually knows independently of the operating system:

```text
Users
Orders
Payments
Authentication
Business Rules
State
Data
Policies
```

That is the deeper idea behind `commonMain`.

> **Do not think of `commonMain` as the place where code is shared. Think of it as the place where platform-independent knowledge of your application lives.**
