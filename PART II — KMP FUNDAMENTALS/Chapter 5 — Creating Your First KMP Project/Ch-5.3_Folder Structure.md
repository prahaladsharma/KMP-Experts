# Chapter 5 — Creating Your First KMP Project

## Part 3 — Folder Structure

> **A KMP project becomes easier to scale when every folder has a clear reason to exist.**

The generated project gives us the build configuration.

Now we need to understand the **physical structure of the codebase**.

For an Android developer, the first instinct may be to look for:

```text
app/
  └── src/main/
```

A KMP project introduces another level of organization:

```text
Project
│
├── Android Application
├── iOS Application
└── Shared Multiplatform Module
```

The most important question is no longer:

> "Where should I put this Kotlin file?"

It becomes:

> **"Who owns this code, which platforms need it, and what should its lifetime and dependency boundary be?"**

---

# 1. The Big Picture

A typical Android + iOS KMP project can be visualized as:

```text
KmpFirstProject/
│
├── androidApp/
│
├── shared/
│
├── iosApp/
│
├── gradle/
│
├── build.gradle.kts
├── settings.gradle.kts
└── gradle.properties
```

At the highest level:

```text
                    KMP PROJECT
                         │
          ┌──────────────┼──────────────┐
          ▼              ▼              ▼
      androidApp       shared         iosApp
          │              │              │
          ▼              ▼              ▼
       Android         Shared           iOS
       Application      Logic        Application
```

This is the first structure to understand.

---

# 2. Three Worlds in One Project

A KMP project combines three different concerns:

```text
┌─────────────────────────────────────────────────┐
│                 Android World                    │
│                                                 │
│  Android UI • Resources • Manifest • APIs      │
└───────────────────────┬─────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────┐
│                 Shared World                    │
│                                                 │
│  Domain • Data • Business Logic • State        │
└───────────────────────┬─────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────┐
│                    iOS World                    │
│                                                 │
│  SwiftUI • UIKit • Resources • Apple APIs      │
└─────────────────────────────────────────────────┘
```

The folder structure should make these boundaries obvious.

---

# 3. Root Folder

The root folder represents the entire application repository.

For example:

```text
KmpFirstProject/
```

It contains:

```text
KmpFirstProject/
│
├── androidApp/
├── shared/
├── iosApp/
├── gradle/
├── build.gradle.kts
├── settings.gradle.kts
└── gradle.properties
```

The root is not where application business logic should normally live.

It is the place where the modules and build are coordinated.

---

# 4. `androidApp`

The Android application belongs under:

```text
androidApp/
```

Conceptually:

```text
androidApp/
│
├── src/
├── build.gradle.kts
└── ...
```

Its responsibility is:

```text
Android Application
```

This can include:

```text
Android UI
Android resources
Android manifest
Android navigation
Android lifecycle integration
Android-specific services
Android-only dependencies
```

---

# 5. What Should Not Go Into `androidApp`

If the logic is genuinely platform-independent, don't automatically put it in:

```text
androidApp/
```

For example:

```text
CalculateTax
ValidateEmail
CalculateCartTotal
DetermineOrderStatus
```

should not be Android-only just because the first implementation was written by an Android developer.

If iOS needs the same behavior, it is a candidate for:

```text
shared/
```

---

# 6. `iosApp`

The iOS application belongs under:

```text
iosApp/
```

Conceptually:

```text
iosApp/
│
├── Swift / SwiftUI sources
├── Xcode project
├── Resources
└── ...
```

Its responsibility is:

```text
iOS Application
```

This can include:

```text
SwiftUI
UIKit
Navigation
iOS lifecycle
Apple services
iOS resources
Apple-specific integrations
```

---

# 7. `shared`

The most important directory for KMP is:

```text
shared/
```

Conceptually:

```text
shared/
│
├── src/
└── build.gradle.kts
```

Inside it:

```text
shared/
└── src/
    ├── commonMain/
    ├── commonTest/
    ├── androidMain/
    └── iosMain/
```

This is where the multiplatform architecture starts becoming visible.

---

# 8. The Shared Module Is Not One Folder

One common mistake is to think:

```text
shared/
└── Everything
```

That is not the purpose of the shared module.

The shared module contains **multiple source sets**.

```text
shared/
└── src/
    ├── commonMain/
    ├── commonTest/
    ├── androidMain/
    └── iosMain/
```

Each source set answers a different question:

| Source Set | Responsibility |
|---|---|
| `commonMain` | Shared production code |
| `commonTest` | Shared tests |
| `androidMain` | Android-specific implementation |
| `iosMain` | iOS-specific implementation |

---

# 9. `commonMain`

This is the primary shared production source set:

```text
shared/src/commonMain/
```

It is where platform-independent logic belongs.

Typical examples:

```text
Domain models
Use cases
Business rules
Repository contracts
Repository implementations
Networking
Serialization
Shared state
Validation
Mapping
```

A possible structure is:

```text
commonMain/
└── kotlin/
    └── com.example.app/
        ├── domain/
        ├── data/
        ├── networking/
        ├── presentation/
        └── platform/
```

The exact package structure depends on the architecture you choose.

---

# 10. `androidMain`

Android-specific shared-module code belongs here:

```text
shared/src/androidMain/
```

For example:

```text
androidMain/
└── kotlin/
    └── com.example.app/
        └── platform/
```

Possible responsibilities:

```text
Android storage implementation
Android API integration
Android-specific logging
Android-specific services
Android platform adapters
```

This code can use:

```text
Android SDK
```

because it is compiled only for Android.

---

# 11. `iosMain`

The equivalent iOS-specific source set is:

```text
shared/src/iosMain/
```

A possible structure:

```text
iosMain/
└── kotlin/
    └── com.example.app/
        └── platform/
```

It can contain:

```text
Apple API integration
iOS storage implementation
iOS-specific services
iOS platform adapters
Native interoperability code
```

The important point is that this code is not pretending to be common.

It explicitly belongs to the iOS side.

---

# 12. `commonTest`

Shared tests belong under:

```text
shared/src/commonTest/
```

A clean structure might be:

```text
commonTest/
└── kotlin/
    └── com.example.app/
        ├── domain/
        ├── data/
        └── networking/
```

Typical tests include:

```text
Business rules
Validation
Mapping
Use cases
Repository behavior
State transitions
```

---

# 13. The Source Set Tree

Put everything together:

```text
shared/
└── src/
    │
    ├── commonMain/
    │   └── kotlin/
    │       └── com.example.app/
    │
    ├── commonTest/
    │   └── kotlin/
    │       └── com.example.app/
    │
    ├── androidMain/
    │   └── kotlin/
    │       └── com.example.app/
    │
    └── iosMain/
        └── kotlin/
            └── com.example.app/
```

This is the structure you should become comfortable reading.

---

# 14. The Most Important Boundary

The core relationship is:

```text
                    commonMain
                        │
          ┌─────────────┴─────────────┐
          ▼                           ▼
     androidMain                   iosMain
          │                           │
          ▼                           ▼
     Android APIs                Apple APIs
```

Think of `commonMain` as the portable center.

Think of `androidMain` and `iosMain` as platform extension points.

---

# 15. Folder Structure Is an Architectural Boundary

Folders are not architecture by themselves.

But in KMP, source-set folders represent real compilation boundaries.

For example:

```text
commonMain/
```

means:

```text
This code must be compatible with all configured common targets.
```

while:

```text
androidMain/
```

means:

```text
This code is allowed to depend on Android.
```

and:

```text
iosMain/
```

means:

```text
This code is allowed to depend on iOS APIs.
```

That makes the folder structure meaningful.

---

# 16. A Simple Feature

Suppose we build:

```text
Product
```

The shared implementation might look like:

```text
commonMain/
└── product/
    ├── Product.kt
    ├── ProductRepository.kt
    ├── GetProductsUseCase.kt
    └── ProductMapper.kt
```

Android-specific implementation:

```text
androidMain/
└── product/
    └── AndroidProductIntegration.kt
```

iOS-specific implementation:

```text
iosMain/
└── product/
    └── IOSProductIntegration.kt
```

This makes ownership visible.

---

# 17. Feature-Based Structure

As the application grows, a feature-oriented structure can be useful.

For example:

```text
commonMain/
└── kotlin/
    └── com.example.app/
        ├── authentication/
        ├── products/
        ├── cart/
        ├── checkout/
        └── profile/
```

Inside a feature:

```text
products/
├── Product.kt
├── ProductRepository.kt
├── GetProductsUseCase.kt
└── ProductState.kt
```

This can make large applications easier to navigate.

---

# 18. Layer-Based Structure

Another approach is layer-oriented:

```text
commonMain/
└── kotlin/
    └── com.example.app/
        ├── data/
        ├── domain/
        ├── presentation/
        └── platform/
```

For example:

```text
domain/
├── Product.kt
├── ProductRepository.kt
└── GetProductsUseCase.kt

data/
├── ProductRepositoryImpl.kt
├── ProductApi.kt
└── ProductMapper.kt
```

Both feature-based and layer-based structures can work.

The important thing is that the dependency direction remains clear.

---

# 19. Don't Mix the Two Without a Reason

A confusing structure can become:

```text
commonMain/
├── data/
│   └── products/
├── products/
│   └── domain/
├── feature/
│   └── product/
└── product/
```

Now a developer has to guess:

```text
Where does product logic belong?
```

Choose a clear organizational model and apply it consistently.

---

# 20. A Practical Feature Structure

For a growing KMP application, one reasonable approach is:

```text
commonMain/
└── kotlin/
    └── com.example.app/
        │
        ├── core/
        │   ├── network/
        │   ├── database/
        │   ├── result/
        │   └── logging/
        │
        ├── product/
        │   ├── data/
        │   ├── domain/
        │   └── presentation/
        │
        ├── cart/
        │   ├── data/
        │   ├── domain/
        │   └── presentation/
        │
        └── checkout/
            ├── data/
            ├── domain/
            └── presentation/
```

This combines:

```text
Feature ownership
+
Architectural layers
```

without making the root package excessively large.

---

# 21. `core` Should Stay Small

A `core` package can be useful.

But it can also become:

```text
core/
└── Everything
```

Avoid that.

Good candidates:

```text
Result
Error
Network abstraction
Common logging abstraction
Date/time abstraction
Shared utilities with real cross-feature value
```

Bad candidates:

```text
Random feature logic
Feature-specific helpers
Temporary code
"Stuff we don't know where to put"
```

If `core` keeps growing indefinitely, your boundaries probably need another look.

---

# 22. Platform Folder

A common pattern is:

```text
commonMain/
└── platform/
```

This can contain platform abstractions.

For example:

```kotlin
interface SecureStorage {
    suspend fun save(key: String, value: String)
    suspend fun read(key: String): String?
}
```

Then:

```text
commonMain
      │
      ▼
SecureStorage
      ▲
      │
 ┌────┴────┐
 ▼         ▼
Android    iOS
```

The implementations can live in their respective source sets.

---

# 23. Platform Implementations

For example:

```text
androidMain/
└── platform/
    └── AndroidSecureStorage.kt
```

and:

```text
iosMain/
└── platform/
    └── IOSSecureStorage.kt
```

The shared business logic depends on:

```text
SecureStorage
```

not:

```text
AndroidSecureStorage
```

or:

```text
IOSSecureStorage
```

This is a clean dependency boundary.

---

# 24. `expect` / `actual` and Folder Structure

KMP also provides the `expect` / `actual` mechanism for certain platform-specific implementations.

Conceptually:

```text
commonMain
    │
    └── expect
          │
          ├───────────────┐
          ▼               ▼
    androidMain        iosMain
       actual             actual
```

For example:

```kotlin
// commonMain
expect fun platformName(): String
```

Android:

```kotlin
// androidMain
actual fun platformName(): String = "Android"
```

iOS:

```kotlin
// iosMain
actual fun platformName(): String = "iOS"
```

The exact choice between `expect` / `actual` and interface-based dependency injection depends on the problem.

---

# 25. Don't Use `expect` / `actual` Everywhere

The existence of `expect` / `actual` does not mean every platform difference should use it.

For larger business capabilities, an interface can often communicate intent more clearly:

```text
Interface
   │
   ├── Android implementation
   └── iOS implementation
```

Use the mechanism that creates the clearest boundary for the problem.

---

# 26. Android UI Structure

The Android application may have a structure such as:

```text
androidApp/
└── src/
    └── main/
        ├── kotlin/
        │   └── com.example.app/
        │       ├── MainActivity.kt
        │       └── ui/
        │
        ├── res/
        └── AndroidManifest.xml
```

If using Jetpack Compose:

```text
ui/
├── App.kt
├── navigation/
└── screens/
```

The UI remains Android-owned in this architecture.

---

# 27. iOS UI Structure

The iOS application may look conceptually like:

```text
iosApp/
├── App/
├── Views/
├── Resources/
└── ...
```

For SwiftUI:

```text
Views/
├── ProductView.swift
├── CartView.swift
└── CheckoutView.swift
```

The exact Xcode organization can differ.

The principle remains:

```text
Android UI → Android
iOS UI     → iOS
```

while shared product behavior can live in KMP.

---

# 28. Native UI, Shared Logic

With native UI, the structure becomes:

```text
                  Shared KMP
                     │
        ┌────────────┴────────────┐
        ▼                         ▼
   Android UI                  iOS UI
        │                         │
        ▼                         ▼
   Android App                 iOS App
```

This is a powerful architecture because each platform can use its native UI conventions while sharing the logic that genuinely needs to be shared.

---

# 29. Shared Presentation

Some teams may also share presentation logic.

For example:

```text
commonMain/
└── product/
    └── ProductPresenter.kt
```

or a shared state-holder / ViewModel-style abstraction.

Then:

```text
Android UI
     │
     ▼
Shared State
     ▲
     │
iOS UI
```

The folder structure should make it clear whether presentation code is:

```text
Shared
```

or:

```text
Platform-specific
```

---

# 30. Avoid Accidental Sharing

A class should not move into:

```text
commonMain
```

just because it can technically compile there.

Ask:

```text
Does both platforms need it?
Is the behavior truly the same?
Does it have platform assumptions?
Would sharing reduce duplication?
Does sharing create coupling?
```

Compilation compatibility is not the same as architectural suitability.

---

# 31. The "Can Compile" vs "Should Share" Test

Consider:

```text
Class A
```

It compiles on Android and iOS.

That answers:

```text
Can it be shared?
```

But we still need to ask:

```text
Should it be shared?
```

For example:

```text
TaxCalculator
```

probably yes.

But:

```text
AndroidPermissionController
```

probably no.

This distinction should guide the folder structure.

---

# 32. A Dependency Direction

A healthy structure generally looks like:

```text
UI
 │
 ▼
Presentation
 │
 ▼
Domain
 │
 ▼
Data
 │
 ▼
Platform Abstractions
 │
 ▼
Platform Implementations
```

In KMP:

```text
Android UI ───────┐
                  ▼
              commonMain
                  ▲
                  │
iOS UI ───────────┘
```

The shared module should not depend upward on platform UI.

---

# 33. Dependency Direction Diagram

```text
                 Android UI
                      │
                      ▼
                Shared Logic
                      ▲
                      │
                   iOS UI
```

For platform services:

```text
                 Shared Logic
                      │
                      ▼
               Platform Interface
                 ▲            ▲
                 │            │
                 │            │
          Android Impl     iOS Impl
```

This keeps the dependency flow predictable.

---

# 34. What About Resources?

Resources are another important folder decision.

Android may own:

```text
androidApp/src/main/res/
```

while iOS owns its resources through Xcode.

If your architecture uses a multiplatform resource solution, resources may also exist in a shared module depending on the chosen tooling.

The important rule is:

> **Resource ownership should follow the UI and platform strategy you have chosen.**

Do not move resources into shared simply because they are duplicated.

---

# 35. Shared vs Platform Resources

For native UI:

```text
Android Resources
        │
        ▼
Android UI

iOS Resources
        │
        ▼
iOS UI
```

For genuinely shared resources:

```text
Shared Resources
        │
   ┌────┴────┐
   ▼         ▼
Android     iOS
```

The decision should be intentional.

---

# 36. Testing Folder Structure

A clean testing structure mirrors the production boundaries.

```text
shared/
└── src/
    ├── commonMain/
    └── commonTest/
```

For Android-specific integration tests:

```text
androidApp/
└── src/
    └── androidTest/
```

For iOS-specific tests, use the appropriate Xcode test targets.

Conceptually:

```text
Shared Behavior
      │
      ▼
commonTest

Platform Integration
      │
 ┌────┴────┐
 ▼         ▼
Android    iOS
Tests      Tests
```

---

# 37. Feature Testing

Suppose we have:

```text
product/
├── data/
├── domain/
└── presentation/
```

Tests can follow the same feature boundary:

```text
commonTest/
└── product/
    ├── data/
    ├── domain/
    └── presentation/
```

This makes it easy to locate tests for a feature.

---

# 38. Large Project Structure

As the application grows, one shared module can become large.

For example:

```text
shared/
└── src/
    └── commonMain/
        └── kotlin/
            └── com.example.app/
                ├── authentication/
                ├── catalog/
                ├── cart/
                ├── checkout/
                ├── profile/
                ├── orders/
                └── notifications/
```

At some point, module boundaries may become more useful than package boundaries.

---

# 39. When to Create More Modules

Don't create ten modules on day one.

Start with:

```text
androidApp
shared
iosApp
```

As the codebase grows, consider modules when you have clear reasons such as:

```text
Independent ownership
Large build cost
Clear feature boundaries
Reusable libraries
Different dependency requirements
Separate release lifecycle
```

Modules should solve a problem.

They should not exist just to make the project tree look sophisticated.

---

# 40. A Possible Scaled Structure

A mature KMP project might eventually look like:

```text
KmpApplication/
│
├── androidApp/
├── iosApp/
│
├── shared/
│
├── core/
│   ├── network/
│   ├── database/
│   └── common/
│
├── feature-auth/
├── feature-products/
├── feature-cart/
└── feature-checkout/
```

This is an evolution, not a requirement for the first project.

---

# 41. Folder Structure Should Follow Ownership

A useful question for every folder is:

> **Who owns this code?**

For example:

```text
Android UI
→ Android team / Android layer

iOS UI
→ iOS team / iOS layer

Business rules
→ Shared domain

Network implementation
→ Shared data

Apple Pay integration
→ iOS

Google Pay integration
→ Android
```

Ownership makes architecture easier to maintain.

---

# 42. Folder Structure Should Follow Change

Another useful question:

> **What is likely to change together?**

If Android UI changes frequently:

```text
Keep it in Android.
```

If business rules must stay identical:

```text
Keep them shared.
```

If iOS integration changes independently:

```text
Keep it in iOS.
```

Good structure follows change boundaries.

---

# 43. Folder Structure and Team Collaboration

Imagine:

```text
Android Developer
iOS Developer
KMP Developer
Backend Developer
```

If all code is mixed together:

```text
commonMain/
└── everything
```

ownership becomes unclear.

A clear structure communicates:

```text
commonMain → shared responsibility
androidApp → Android responsibility
iosApp    → iOS responsibility
```

This reduces accidental changes.

---

# 44. Folder Structure and Git

The folder structure also affects Git workflows.

A developer working on Android UI should usually be able to modify:

```text
androidApp/
```

without touching:

```text
shared/
```

unless the feature requires shared behavior changes.

Similarly, an iOS UI change should not require unrelated Android modifications.

Clear boundaries reduce merge conflicts.

---

# 45. Folder Structure and Code Review

A reviewer should be able to infer intent from a changed path.

For example:

```text
shared/src/commonMain/.../PaymentValidator.kt
```

immediately suggests:

```text
Shared business logic
```

while:

```text
iosApp/.../ApplePayView.swift
```

suggests:

```text
iOS-specific UI/integration
```

Folder names become part of the project's communication system.

---

# 46. A Feature Across Platforms

Consider a checkout feature.

The structure could be:

```text
commonMain/
└── checkout/
    ├── domain/
    │   ├── Checkout.kt
    │   └── CheckoutUseCase.kt
    │
    ├── data/
    │   ├── CheckoutRepository.kt
    │   └── CheckoutApi.kt
    │
    └── presentation/
        └── CheckoutState.kt
```

Android:

```text
androidApp/
└── checkout/
    └── CheckoutScreen.kt
```

iOS:

```text
iosApp/
└── Checkout/
    └── CheckoutView.swift
```

Now the ownership is obvious.

---

# 47. Complete Feature Flow

```text
                     Checkout Feature
                            │
                ┌───────────┴───────────┐
                ▼                       ▼
          Shared KMP Logic          Platform UI
                │                 ┌─────┴─────┐
                │                 ▼           ▼
          ┌─────┴─────┐       Android       iOS
          ▼           ▼
       Domain       Data
          │           │
          └─────┬─────┘
                ▼
          Business Result
                │
          ┌─────┴─────┐
          ▼           ▼
      Android UI    iOS UI
```

This is what a good folder structure should make easy to understand.

---

# 48. The Folder Structure Is a Dependency Graph

The directory tree may look physical:

```text
folders
files
packages
```

but underneath it represents dependencies:

```text
UI
 │
 ▼
Shared Presentation
 │
 ▼
Domain
 │
 ▼
Data
 │
 ▼
Platform
```

If the physical structure contradicts the dependency structure, the project becomes harder to maintain.

---

# 49. A Warning Sign

If you find yourself creating:

```text
commonMain/
└── android/
```

and:

```text
commonMain/
└── ios/
```

for large amounts of platform-specific code, stop and ask whether that code belongs in:

```text
androidMain/
```

or:

```text
iosMain/
```

Source sets exist specifically to express this distinction.

---

# 50. Another Warning Sign

If `commonMain` starts containing:

```text
Context
Activity
UIViewController
UIApplication
Android SDK
UIKit
```

you have probably crossed the platform boundary.

The solution is usually to introduce:

```text
Abstraction
+
Platform Implementation
```

rather than hiding the platform dependency inside common code.

---

# 51. A Clean First Project

For the first project in this book, keep it simple:

```text
KmpFirstProject/
│
├── androidApp/
│
├── shared/
│   └── src/
│       ├── commonMain/
│       ├── commonTest/
│       ├── androidMain/
│       └── iosMain/
│
├── iosApp/
│
├── gradle/
│
├── build.gradle.kts
├── settings.gradle.kts
└── gradle.properties
```

Don't introduce unnecessary modules yet.

---

# 52. A Clean First `commonMain`

Start with something small:

```text
commonMain/
└── kotlin/
    └── com.example.kmp/
        ├── domain/
        └── data/
```

As features appear:

```text
commonMain/
└── kotlin/
    └── com.example.kmp/
        ├── core/
        ├── authentication/
        ├── products/
        └── checkout/
```

Let the structure grow with the product.

---

# 53. A Clean First Platform Structure

Android:

```text
androidApp/
└── src/
    └── main/
        ├── kotlin/
        │   └── com.example.kmp/
        │       ├── MainActivity.kt
        │       └── ui/
        └── res/
```

iOS:

```text
iosApp/
├── App/
├── Views/
└── Resources/
```

Shared:

```text
shared/
└── src/
    ├── commonMain/
    ├── commonTest/
    ├── androidMain/
    └── iosMain/
```

This gives us a clean separation from the beginning.

---

# 54. Folder Structure Checklist

Before adding more code, verify:

```text
[ ] Android application is clearly separated
[ ] iOS application is clearly separated
[ ] Shared module is clearly separated
[ ] commonMain contains platform-independent code
[ ] androidMain contains Android-specific code
[ ] iosMain contains iOS-specific code
[ ] commonTest contains shared tests
[ ] Feature boundaries are understandable
[ ] Platform APIs are not leaking into common code
[ ] Dependencies flow in one clear direction
[ ] No unnecessary modules exist
[ ] No "everything" or "utils" dumping ground exists
```

---

# 55. Folder Structure and the Next Step

Once you understand:

```text
Project
   │
   ├── androidApp
   ├── shared
   └── iosApp
```

and:

```text
shared
   │
   ├── commonMain
   ├── commonTest
   ├── androidMain
   └── iosMain
```

the next important question becomes:

> **How does code actually move between these source sets?**

That is where concepts such as:

```text
expect / actual
interfaces
dependency injection
platform implementations
```

become important.

---

# Chapter Takeaways

> [!TIP]
> **A good KMP folder structure makes platform boundaries visible before you read a single line of code.**

Remember:

1. `androidApp` owns the Android application.
2. `iosApp` owns the iOS application.
3. `shared` owns the multiplatform module.
4. `commonMain` contains platform-independent production code.
5. `androidMain` contains Android-specific implementation.
6. `iosMain` contains iOS-specific implementation.
7. `commonTest` contains shared tests.
8. A folder boundary in KMP can represent a real compilation boundary.
9. `commonMain` should not become a dumping ground.
10. Code that depends on Android APIs belongs on the Android side.
11. Code that depends on Apple APIs belongs on the iOS side.
12. Shared business logic should depend on abstractions rather than platform implementations.
13. Feature-based organization can help large applications.
14. Layer-based organization can also work; consistency matters more than a specific style.
15. `core` should contain only genuinely cross-cutting functionality.
16. Don't create many Gradle modules before you have a reason to do so.
17. Folder structure should follow ownership and change boundaries.
18. A good structure makes code review and team collaboration easier.
19. Resource ownership should follow the UI and platform strategy.
20. The physical project tree should reflect the logical dependency graph.

---

# Final Mental Model

When you look at a KMP project, don't see:

```text
Folders
Files
Packages
Modules
```

See:

```text
                         PRODUCT
                            │
              ┌─────────────┼─────────────┐
              ▼             ▼             ▼
          Android         Shared           iOS
          Application      Logic        Application
              │             │               │
              │       ┌─────┴─────┐         │
              │       ▼           ▼         │
              │    commonMain   Platform     │
              │       │        Boundaries    │
              │       │           │          │
              │       └─────┬─────┘          │
              │             ▼                │
              └─────── Shared Behavior ──────┘
```

And inside the shared module:

```text
shared/
└── src/
    │
    ├── commonMain
    │      │
    │      ▼
    │  Shared Product Behavior
    │
    ├── commonTest
    │      │
    │      ▼
    │  Shared Verification
    │
    ├── androidMain
    │      │
    │      ▼
    │  Android Implementation
    │
    └── iosMain
           │
           ▼
       iOS Implementation
```

The key idea is simple:

> **The best KMP folder structure does not try to hide platform differences. It makes them obvious, keeps shared behavior in the center, and gives each platform a clear place to live.**
