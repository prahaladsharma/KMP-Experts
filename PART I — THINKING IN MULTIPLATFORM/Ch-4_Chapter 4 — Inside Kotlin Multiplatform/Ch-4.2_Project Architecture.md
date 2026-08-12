# Chapter 4 — Inside Kotlin Multiplatform

## Part 2 — Project Architecture

> **A KMP project becomes much easier to understand when you stop looking at it as a collection of Gradle folders and start looking at it as a map of responsibilities.**

The previous part established the mental model:

```text
Android Application
        │
        ├──────────────┐
        │              │
        ▼              ▼
   Shared KMP Code   Platform Code
        ▲              ▲
        │              │
        └──────────────┘
               iOS
```

Now we need to turn that mental model into a real project.

This is where many developers encounter their first confusion with Kotlin Multiplatform.

You open a KMP project and suddenly see:

```text
commonMain
commonTest
androidMain
androidUnitTest
iosMain
iosTest
```

Then there is:

```text
build.gradle.kts
settings.gradle.kts
gradle.properties
libs.versions.toml
```

And depending on the project:

```text
androidApp
iosApp
shared
composeApp
```

At first, it can look like a lot of infrastructure.

The good news is that the structure becomes much easier once you understand **why each piece exists**.

---

# 1. Start With the Big Picture

A typical multiplatform application can be visualized like this:

```text
                         KMP PROJECT
                              │
             ┌────────────────┴────────────────┐
             │                                 │
             ▼                                 ▼
       Shared Module                     Platform Apps
             │                                 │
      ┌──────┴──────┐                  ┌───────┴───────┐
      ▼             ▼                  ▼               ▼
 commonMain    Platform Source      Android           iOS
      │             Sets               │               │
      ▼                                ▼               ▼
 Shared Logic                    Native Entry     Native Entry
```

There are two major concepts here:

### Shared module

Contains code that can be reused.

### Platform applications

Contain the platform-specific application entry points and integrations.

This distinction is the foundation of KMP project architecture.

---

# 2. A Typical Project Structure

A simple KMP project might look like:

```text
MyKMPApp/
│
├── androidApp/
│
├── iosApp/
│
├── shared/
│   └── src/
│       ├── commonMain/
│       ├── commonTest/
│       ├── androidMain/
│       ├── androidUnitTest/
│       ├── iosMain/
│       └── iosTest/
│
├── gradle/
│   └── libs.versions.toml
│
├── build.gradle.kts
├── settings.gradle.kts
├── gradle.properties
└── gradlew
```

This structure is only one possible arrangement.

A project using Compose Multiplatform may look different:

```text
MyKMPApp/
│
├── composeApp/
│   └── src/
│       ├── commonMain/
│       ├── androidMain/
│       ├── iosMain/
│       └── commonTest/
│
├── iosApp/
│
├── gradle/
│
├── build.gradle.kts
└── settings.gradle.kts
```

The names can change.

The underlying concepts do not.

---

# 3. The Four Questions Behind the Structure

Whenever you see a KMP project, ask four questions:

```text
1. What code is shared?

2. What code is platform-specific?

3. What application starts on each platform?

4. Which target is this code compiled for?
```

Almost every folder and Gradle configuration decision can be understood through these questions.

---

# 4. The Shared Module

The shared module is the heart of a typical KMP project.

For example:

```text
shared/
```

It might contain:

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

The module itself is not necessarily an application.

It is a multiplatform library/module that produces platform-specific outputs consumed by the applications.

Think of it as:

```text
                    Shared Module
                         │
             ┌───────────┴───────────┐
             ▼                       ▼
         Android                   iOS
         Output                    Output
             │                       │
             ▼                       ▼
      Android App              iOS App
```

This is an important distinction.

The shared module contains reusable product logic.

The platform applications own the actual application lifecycle.

---

# 5. `commonMain`

`commonMain` is usually the most important source set in a KMP project.

It contains code intended to be shared across the configured targets.

For example:

```text
commonMain/
├── domain/
├── data/
├── networking/
├── models/
├── validation/
└── synchronization/
```

A simplified example:

```kotlin
class CartCalculator {

    fun total(
        price: Double,
        quantity: Int
    ): Double {
        return price * quantity
    }
}
```

There is no Android API here.

There is no iOS API here.

The calculation represents product behavior.

Therefore it is a natural candidate for `commonMain`.

---

# 6. `commonMain` Is a Contract of Portability

A useful way to think about `commonMain` is:

> **Everything placed here is making a portability statement.**

When you write:

```text
commonMain/
    PaymentCalculator.kt
```

you are effectively saying:

> "This logic should be usable by all of the targets supported by this source set."

That is a powerful architectural constraint.

It prevents accidental coupling to one operating system.

---

# 7. `commonTest`

Shared code needs shared tests.

That is where:

```text
commonTest/
```

comes in.

For example:

```text
shared/
└── src/
    ├── commonMain/
    │   └── domain/
    │       └── DiscountCalculator.kt
    │
    └── commonTest/
        └── domain/
            └── DiscountCalculatorTest.kt
```

The test can validate the business rule once.

Conceptually:

```text
             Shared Business Rule
                     │
                     ▼
                commonTest
                     │
             ┌───────┴───────┐
             ▼               ▼
          Android           iOS
```

This is one of the strongest practical benefits of moving genuine business logic into common code.

---

# 8. `androidMain`

Now we cross the platform boundary.

```text
androidMain/
```

contains Android-specific implementations.

For example:

```text
androidMain/
├── database/
├── platform/
├── notifications/
└── security/
```

A class in `androidMain` can use Android-specific APIs.

For example:

```kotlin
class AndroidNotificationManager {
    // Android-specific implementation
}
```

This code does not need to pretend that iOS exists.

It belongs to Android.

That is exactly what the source-set architecture is designed to express.

---

# 9. `iosMain`

Similarly:

```text
iosMain/
```

contains iOS-specific Kotlin implementations.

For example:

```text
iosMain/
├── platform/
├── security/
├── notifications/
└── storage/
```

A common interface can then have an iOS implementation.

Conceptually:

```text
                 commonMain
                     │
                     ▼
              Common Contract
                     │
          ┌──────────┴──────────┐
          ▼                     ▼
     androidMain             iosMain
          │                     │
          ▼                     ▼
   Android implementation   iOS implementation
```

This pattern is fundamental to KMP.

---

# 10. Source Sets Are More Than Folders

A beginner may think:

> "`commonMain` is just a folder."

Technically, it is represented in the project structure as a source-set directory.

Architecturally, however, it is much more.

A source set defines:

```text
Which code belongs together
+
Which dependencies are available
+
Which targets can consume it
```

So:

```text
commonMain
```

is not merely a directory.

It represents a **set of source files compiled for common targets under the multiplatform model**.

---

# 11. Dependency Direction

One of the most important rules in KMP architecture is dependency direction.

A healthy structure looks like:

```text
                 Platform App
                      │
                      ▼
                 Shared Module
                      │
                      ▼
                Shared Domain
```

The platform application consumes the shared module.

You generally don't want the common layer to depend directly on an Android application module.

Conceptually:

```text
          Android App       iOS App
                │             │
                └──────┬──────┘
                       ▼
                 Shared Module
```

The shared module should remain independently meaningful.

---

# 12. A Layered KMP Architecture

As the project grows, the shared module can be organized into architectural layers.

For example:

```text
shared/
└── src/
    └── commonMain/
        ├── domain/
        ├── data/
        ├── networking/
        ├── presentation/
        └── core/
```

One possible dependency flow:

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
 Platform Implementations
```

The exact layering is not mandated by KMP.

KMP provides the multiplatform boundary.

Your architecture provides the internal organization.

---

# 13. KMP Does Not Force Clean Architecture

This distinction matters.

You can build KMP using:

```text
Clean Architecture
```

or:

```text
MVVM
```

or:

```text
MVI
```

or:

```text
Repository-based architecture
```

or another architecture entirely.

KMP doesn't tell you how to structure your business logic.

It provides the ability to compile and share Kotlin code across targets.

Architecture remains your responsibility.

---

# 14. The Role of the Android Application

Let's look at:

```text
androidApp/
```

This is typically a real Android application module.

It may contain:

```text
androidApp/
├── src/
│   └── main/
│       ├── AndroidManifest.xml
│       └── kotlin/
│
└── build.gradle.kts
```

This module owns Android-specific concerns such as:

* Application lifecycle
* Android manifest
* Activities
* Android resources
* Android permissions
* Android-specific configuration
* Android dependency integration

It can consume the shared KMP module.

```text
androidApp
     │
     ▼
shared
```

---

# 15. The Role of the iOS Application

The iOS application is slightly different from what Android developers may expect.

A common project can have:

```text
iosApp/
```

which is managed using Apple's tooling.

It may contain:

```text
iosApp/
├── iOSApp.swift
├── ContentView.swift
├── Assets.xcassets
└── ...
```

The exact structure depends on the project.

The important idea is:

```text
                  iOS Application
                         │
                         ▼
                 Shared KMP Framework
```

The iOS application remains a real iOS application.

KMP provides the shared Kotlin functionality it consumes.

---

# 16. Why iOS Has a Different Shape

This can surprise Android developers.

Android projects are typically managed heavily through Gradle and Android Studio.

iOS applications rely on Apple's ecosystem:

```text
Xcode
Swift / SwiftUI
Apple SDKs
Signing
Provisioning
App Store tooling
```

KMP doesn't erase that ecosystem.

Instead, it creates a bridge between Kotlin shared code and the iOS application.

This is why a multiplatform developer needs to understand both sides of the project.

---

# 17. The Framework Boundary

A simplified project architecture looks like:

```text
                     Git Repository
                           │
          ┌────────────────┴────────────────┐
          ▼                                 ▼
     Android Project                    iOS Project
          │                                 │
          │                                 │
          └──────────────┬──────────────────┘
                         ▼
                   KMP Shared Module
                         │
              ┌──────────┴──────────┐
              ▼                     ▼
         commonMain             Platform Source
                                    Sets
```

This is the architecture we will use repeatedly throughout the book.

---

# 18. Platform Source Sets

A KMP project can have more than:

```text
commonMain
androidMain
iosMain
```

As the number of targets grows, source sets can become hierarchical.

For example:

```text
commonMain
    │
    ├── androidMain
    │
    ├── iosMain
    │      ├── iosArm64
    │      └── iosSimulatorArm64
    │
    └── jvmMain
```

The important concept is **shared source-set hierarchy**.

A source set can provide code to multiple more-specific targets.

This allows common code to be shared at the appropriate level instead of duplicating it for every target.

---

# 19. The Hierarchical Source-Set Model

Think about it like a tree:

```text
                    commonMain
                        │
            ┌───────────┼───────────┐
            ▼           ▼           ▼
        androidMain   iosMain    jvmMain
                        │
                ┌───────┴───────┐
                ▼               ▼
             iosArm64     iosSimulatorArm64
```

The higher-level source set contains code shared by the targets underneath it.

The lower-level source sets contain more specific implementations.

This is one of the key ideas behind modern KMP source-set architecture.

---

# 20. Why Hierarchy Matters

Imagine you have:

```text
Android
iOS
Desktop
```

Some code is common to all three.

Other code is common only to:

```text
JVM-based targets
```

Other code is specific to:

```text
iOS
```

A flat structure would encourage duplication.

A hierarchical structure allows:

```text
                  commonMain
                 /    |     \
                /     |      \
          Android   JVM      iOS
                     |
                 Desktop
```

This gives the project more precise sharing boundaries.

---

# 21. Gradle Is the Orchestrator

Now we reach one of the most important pieces:

```text
build.gradle.kts
```

For Android developers, this is familiar.

But in KMP, Gradle has a broader responsibility.

It defines:

```text
Targets
+
Source Sets
+
Dependencies
+
Compilation
+
Framework Output
+
Tests
```

Conceptually:

```text
                 Gradle
                   │
        ┌──────────┼──────────┐
        ▼          ▼          ▼
     Android      iOS        JVM
        │          │          │
        ▼          ▼          ▼
     Compile    Compile    Compile
```

The Kotlin Multiplatform Gradle plugin coordinates these targets and source sets.

---

# 22. A Simplified KMP Gradle Configuration

A conceptual configuration may look like:

```kotlin
plugins {
    kotlin("multiplatform")
}

kotlin {
    androidTarget()

    iosArm64()
    iosSimulatorArm64()

    sourceSets {
        commonMain.dependencies {
            // Shared dependencies
        }

        commonTest.dependencies {
            // Shared test dependencies
        }
    }
}
```

This is intentionally simplified.

The exact configuration depends on the Kotlin version, Android Gradle Plugin version, targets, plugins, and project requirements.

The important part is the relationship:

```text
Kotlin Multiplatform Plugin
          │
          ▼
        Targets
          │
          ▼
      Source Sets
          │
          ▼
     Dependencies
```

---

# 23. Targets vs Source Sets

These two terms are easy to confuse.

### Target

A target represents a platform/compiler destination.

Examples include:

```text
Android
iOS ARM64
iOS Simulator
JVM
```

### Source Set

A source set represents a collection of code that can be compiled for one or more targets.

For example:

```text
commonMain
```

can contribute to multiple targets.

Think:

```text
Target = Where code goes

Source Set = Which code is shared
```

This mental model is extremely useful.

---

# 24. Dependency Resolution

Dependencies also follow source-set boundaries.

For example:

```text
commonMain
   │
   ├── Kotlinx Coroutines
   ├── Serialization
   └── Shared Networking
```

Android-specific:

```text
androidMain
   │
   └── Android-specific library
```

iOS-specific:

```text
iosMain
   │
   └── iOS-specific library
```

Conceptually:

```text
                    Dependencies
                         │
          ┌──────────────┼──────────────┐
          ▼              ▼              ▼
      commonMain     androidMain      iosMain
          │              │              │
          ▼              ▼              ▼
      Shared libs    Android libs    iOS libs
```

This prevents platform dependencies from accidentally leaking into common code.

---

# 25. The Dependency Leakage Problem

Suppose someone adds an Android-only dependency to common code.

Conceptually:

```text
commonMain
     │
     ▼
Android-only library
```

Now the common code is no longer genuinely common.

The source set model helps expose this problem.

A healthier design is:

```text
commonMain
     │
     ▼
Common abstraction
     │
 ┌───┴────┐
 ▼        ▼
Android   iOS
```

Platform implementations can use platform-specific libraries.

---

# 26. Example: Storage

Imagine we need:

```text
Key-Value Storage
```

The application requires the same conceptual operation:

```text
save(key, value)
read(key)
remove(key)
```

The interface can be shared:

```kotlin
interface KeyValueStorage {
    fun save(key: String, value: String)
    fun read(key: String): String?
    fun remove(key: String)
}
```

The implementations can differ.

```text
commonMain
    │
    ▼
KeyValueStorage
    │
    ├──────────────┐
    ▼              ▼
androidMain      iosMain
    │              │
    ▼              ▼
Android storage  iOS storage
```

The business layer doesn't need to know the storage implementation.

This is normal dependency inversion applied to a multiplatform project.

---

# 27. `expect` and `actual`

KMP also provides a mechanism called:

```text
expect / actual
```

At a high level:

```text
commonMain
    │
    ▼
expect
    │
 ┌──┴──┐
 ▼     ▼
Android iOS
 │      │
 ▼      ▼
actual actual
```

For example:

```kotlin
expect fun platformName(): String
```

Android:

```kotlin
actual fun platformName(): String {
    return "Android"
}
```

iOS:

```kotlin
actual fun platformName(): String {
    return "iOS"
}
```

The common code knows the contract.

Each platform supplies its implementation.

Kotlin's official documentation describes `expect` and `actual` declarations as a mechanism for defining platform-specific implementations from common code.

---

# 28. Interfaces vs `expect` / `actual`

You should not automatically use `expect` / `actual` for every platform-specific dependency.

Sometimes a normal interface is cleaner:

```text
commonMain
    │
    ▼
Interface
    │
 ┌──┴──┐
 ▼     ▼
Android iOS
```

For example:

```kotlin
interface Analytics {
    fun track(event: String)
}
```

Then Android and iOS can provide implementations.

The choice depends on the abstraction.

A useful rule:

> **Use the simplest mechanism that clearly represents the dependency boundary.**

---

# 29. Platform Code Is an Escape Hatch

This is one of KMP's strengths.

Suppose 90% of your authentication flow is common.

But Android needs:

```text
BiometricPrompt
```

and iOS needs:

```text
LocalAuthentication
```

You don't have to choose between:

```text
Everything shared
```

and:

```text
Everything native
```

Instead:

```text
Shared Authentication
         │
         ▼
Platform Authentication
    ┌────┴────┐
    ▼         ▼
 Android     iOS
```

This is the selective-sharing model in practice.

---

# 30. Compose Multiplatform Changes the Shape

If you use Compose Multiplatform, the architecture can become:

```text
shared/
└── src/
    ├── commonMain/
    │   ├── ui/
    │   ├── domain/
    │   ├── data/
    │   └── ...
    │
    ├── androidMain/
    └── iosMain/
```

Now `commonMain` may contain both:

```text
Shared Logic
+
Shared UI
```

The application structure becomes:

```text
             Compose Multiplatform
                      │
               ┌──────┴──────┐
               ▼             ▼
            Android          iOS
```

But remember:

> **Compose Multiplatform is optional.**

KMP architecture does not require shared UI.

---

# 31. Native UI + KMP Architecture

If you keep native UI:

```text
Android
│
└── Jetpack Compose
       │
       ▼
    Shared KMP
       │
       ▼
iOS
│
└── SwiftUI
```

The shared module might contain:

```text
Domain
Data
Networking
Business Logic
State
```

while the UI remains native.

This is often a very clean architecture for teams that want platform-specific experiences.

---

# 32. Shared UI + KMP Architecture

If you share UI:

```text
                 shared
                   │
          ┌────────┴────────┐
          ▼                 ▼
      Compose UI       Business Logic
          │                 │
          └────────┬────────┘
                   ▼
              Android + iOS
```

This can reduce UI duplication but introduces a different set of design considerations.

The correct choice depends on the product.

---

# 33. A Realistic Enterprise Project

Imagine an enterprise application:

```text
warehouse-app/
│
├── androidApp/
│
├── iosApp/
│
├── shared/
│   │
│   └── src/
│       ├── commonMain/
│       │   ├── domain/
│       │   ├── data/
│       │   ├── networking/
│       │   ├── sync/
│       │   └── validation/
│       │
│       ├── commonTest/
│       │
│       ├── androidMain/
│       │   ├── scanner/
│       │   ├── bluetooth/
│       │   └── printing/
│       │
│       └── iosMain/
│           └── platform/
│
├── gradle/
│
├── build.gradle.kts
└── settings.gradle.kts
```

Notice what happened.

The shared layer contains:

```text
Business
Data
Network
Sync
Validation
```

Android-specific hardware stays:

```text
androidMain
```

The architecture reflects the product.

That is what we want.

---

# 34. Modules Can Grow Beyond One Shared Module

A small application might have:

```text
shared/
```

A large enterprise application may eventually need:

```text
core/
domain/
data/
network/
feature-auth/
feature-orders/
feature-profile/
platform/
```

For example:

```text
MyKMPApp/
│
├── androidApp/
├── iosApp/
│
├── core/
├── domain/
├── data/
├── networking/
├── feature-auth/
└── feature-orders/
```

Each module can itself be configured for the relevant multiplatform targets.

This allows large projects to maintain clear ownership boundaries.

---

# 35. Don't Create Modules Just Because You Can

More modules don't automatically mean better architecture.

A project with:

```text
40 tiny modules
```

can be harder to understand than:

```text
5 well-defined modules
```

Module boundaries should represent meaningful responsibilities.

Good reasons include:

* Independent ownership
* Clear dependency boundaries
* Reusable functionality
* Build isolation
* Feature isolation
* Platform boundaries

Avoid creating modules simply to make the project look "enterprise."

---

# 36. A Useful Dependency Graph

A mature architecture might look like:

```text
                   Android App
                        │
                        ▼
                 Feature Modules
                        │
                        ▼
                      Domain
                        │
                        ▼
                       Data
                        │
                ┌───────┴───────┐
                ▼               ▼
             Network         Storage
                │               │
                └───────┬───────┘
                        ▼
                Platform APIs
```

iOS consumes the same shared layers where applicable.

This keeps the business core relatively independent from application entry points.

---

# 37. The Direction of Dependency

One rule is worth putting in bold:

> [!IMPORTANT]
> **Platform-specific code should not accidentally become the foundation of your shared business logic.**

A healthy direction is:

```text
Platform
   │
   ▼
Shared abstractions
   │
   ▼
Domain
```

Not:

```text
Domain
   │
   ▼
Android Activity
   │
   ▼
Android SDK
```

The second structure destroys portability.

---

# 38. Architecture Is About Dependency Control

The real goal of the project structure is not to make folders look organized.

It is to control dependencies.

For example:

```text
Domain
  │
  ├── Kotlin standard/common APIs
  │
  └── Business rules
```

while:

```text
Android implementation
  │
  ├── Android SDK
  ├── AndroidX
  └── Device APIs
```

and:

```text
iOS implementation
  │
  ├── Apple APIs
  └── iOS frameworks
```

This separation protects the shared core.

---

# 39. The Architecture Test

A simple test can tell you whether your boundaries are healthy.

Ask:

> **"If I removed Android from this module, would the remaining code still make sense?"**

For `commonMain`, the answer should generally be:

```text
Yes.
```

Ask:

> **"If I removed iOS from this module, would the common business logic still make sense?"**

Again:

```text
Yes.
```

If the answer is no, platform-specific assumptions may have leaked into the shared layer.

---

# 40. A Second Architecture Test

Ask:

> **"Can I explain why this class belongs in `commonMain`?"**

A good answer sounds like:

> "It implements order validation, which is a business rule shared by every client."

A weak answer sounds like:

> "It works on both platforms."

Compilation compatibility alone isn't enough.

Architectural responsibility matters.

---

# 41. Common Architecture Mistakes

### Mistake 1 — Put Everything in `commonMain`

```text
commonMain
└── Everything
```

Result:

```text
Hard to maintain
Hard to understand
Hard to evolve
```

---

### Mistake 2 — Treat Platform Code as Failure

```text
iosMain
androidMain
```

are not signs that KMP failed.

They are part of the architecture.

---

### Mistake 3 — Share UI Automatically

Not every application needs shared UI.

---

### Mistake 4 — Ignore Native Developers

KMP still requires platform knowledge.

---

### Mistake 5 — Create Giant Shared Modules

A single:

```text
shared/
```

module containing thousands of unrelated classes can eventually become difficult to maintain.

---

### Mistake 6 — Leak Platform Dependencies

If `commonMain` depends directly on Android-specific concepts, the portability boundary has already weakened.

---

# 42. The Architecture We Want

A healthy KMP project should make the following relationship obvious:

```text
                 PLATFORM APPS
               /              \
              ▼                ▼
          Android              iOS
              \                /
               \              /
                ▼              ▼
                  SHARED CORE
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
        Domain        Data       Network
          │            │            │
          └────────────┼────────────┘
                       ▼
               Platform-specific
                implementations
```

The architecture should answer:

```text
What is shared?
What is native?
Why?
```

without requiring a 30-minute explanation.

---

# 43. The Mental Model Becomes Concrete

We started this chapter with:

```text
Android
   +
iOS
   +
Shared KMP
```

Now we can expand it:

```text
                    Android App
                         │
                         ▼
                   Android APIs
                         │
                         │
             ┌───────────┴───────────┐
             │                       │
             ▼                       ▼
        androidMain             commonMain
                                     │
                           ┌─────────┼─────────┐
                           ▼         ▼         ▼
                        Domain     Data      Network
                           │         │         │
                           └─────────┼─────────┘
                                     │
                                     ▼
                                  iosMain
                                     │
                                     ▼
                                iOS APIs
```

This is the project architecture in action.

---

# 44. The Most Important Vocabulary

Before moving forward, make sure these terms are clear.

| Term                     | Meaning                                                  |
| ------------------------ | -------------------------------------------------------- |
| **Kotlin Multiplatform** | Kotlin technology for sharing code across platforms      |
| **Target**               | A platform/compiler destination                          |
| **Source Set**           | A group of source files compiled for one or more targets |
| **`commonMain`**         | Shared production code                                   |
| **`commonTest`**         | Shared test code                                         |
| **`androidMain`**        | Android-specific source set                              |
| **`iosMain`**            | iOS-specific source set                                  |
| **Platform App**         | The native application consuming shared code             |
| **Shared Module**        | Multiplatform module containing reusable code            |
| **`expect` / `actual`**  | Mechanism for platform-specific implementations          |

These terms will appear throughout the rest of the book.

---

# 45. The Project Architecture in One Picture

If you are an Android developer coming into KMP, keep this diagram nearby:

```text
                         KMP PROJECT
                              │
          ┌───────────────────┴───────────────────┐
          │                                       │
          ▼                                       ▼
     Android App                              iOS App
          │                                       │
          │                                       │
          └───────────────────┬───────────────────┘
                              ▼
                       SHARED MODULE
                              │
                  ┌───────────┴───────────┐
                  ▼                       ▼
             commonMain             commonTest
                  │
        ┌─────────┼─────────┐
        ▼         ▼         ▼
     Domain      Data     Network
        │         │         │
        └─────────┼─────────┘
                  │
        ┌─────────┴─────────┐
        ▼                   ▼
  androidMain            iosMain
        │                   │
        ▼                   ▼
 Android APIs           iOS APIs
```

Once this picture becomes familiar, KMP projects stop looking mysterious.

They become a collection of clearly defined boundaries.

---

# 46. The Deeper Lesson

KMP project architecture is not primarily about folders.

It is about answering one question repeatedly:

> **Where does this responsibility belong?**

If the answer is:

```text
Every platform
```

put it in shared code.

If the answer is:

```text
Android only
```

put it behind the Android boundary.

If the answer is:

```text
iOS only
```

put it behind the iOS boundary.

If the answer is:

```text
Some platforms
```

use the appropriate hierarchical source-set or abstraction strategy.

That is the architecture.

---

# 47. Chapter Takeaways

> [!TIP]
> **A KMP project is easiest to understand when you view source sets and modules as dependency boundaries rather than directories.**

Remember:

1. A KMP project can contain native Android and iOS applications.
2. A shared module contains reusable multiplatform code.
3. `commonMain` is the primary shared production source set.
4. `commonTest` contains shared tests.
5. `androidMain` contains Android-specific implementations.
6. `iosMain` contains iOS-specific implementations.
7. Source sets can form a hierarchy for more precise sharing.
8. Gradle coordinates targets, source sets, dependencies, and compilation.
9. A target describes where code is compiled.
10. A source set describes which code is compiled for those targets.
11. Platform dependencies should not leak into common business logic.
12. `expect` / `actual` is one mechanism for platform-specific implementations.
13. Interfaces and dependency inversion can also be used for platform boundaries.
14. KMP does not require shared UI.
15. Compose Multiplatform can be introduced when shared UI is valuable.
16. Large projects may benefit from multiple well-defined shared modules.
17. The goal of the architecture is controlled dependency direction.
18. The best shared code represents genuinely common product behavior.

---

# Closing Thought

At first glance, a KMP project can look like this:

```text
commonMain
androidMain
iosMain
Gradle
Xcode
Targets
Source Sets
```

It can feel like a lot of new terminology.

But underneath all of it is a simple idea:

```text
                    WHAT IS COMMON?
                           │
                           ▼
                      commonMain
                           │
                           ▼
                  Shared Product Logic
                           │
                ┌──────────┴──────────┐
                ▼                     ▼
           Android-specific      iOS-specific
                │                     │
                ▼                     ▼
           androidMain             iosMain
```

Once you understand that boundary, the project structure becomes much easier to reason about.

The next step is to understand what actually happens when the code in those source sets is compiled.

Because `commonMain` is not magic.

Kotlin Multiplatform has a real compilation model underneath it—and understanding that model is what turns KMP from a collection of Gradle configurations into something you can confidently architect.
