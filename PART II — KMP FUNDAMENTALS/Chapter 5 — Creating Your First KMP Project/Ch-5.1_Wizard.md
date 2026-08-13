# Chapter 5 — Creating Your First KMP Project

## Part 1 — Wizard

> **The first KMP project should not feel magical. It should feel understandable.**

Creating a Kotlin Multiplatform project is often the moment when the concepts from the previous chapters become concrete.

Until now, we have talked about:

- why multiplatform development exists,
- how KMP differs from other approaches,
- how the compiler works,
- how the build pipeline works,
- and how shared code participates in different runtimes.

Now it is time to create a project.

The goal of this part is not to blindly click through a wizard.

The goal is to understand:

> **What is the wizard actually creating for us?**

---

# 1. Before Creating the Project

A KMP project depends on more than Kotlin source code.

Your development environment needs to support the targets you intend to build.

For a typical Android + iOS KMP project, the development environment includes:

```text
                    Development Machine
                           │
             ┌─────────────┴─────────────┐
             ▼                           ▼
        Android Toolchain            Apple Toolchain
             │                           │
             ▼                           ▼
        Android Studio                 Xcode
             │                           │
             └─────────────┬─────────────┘
                           ▼
                    Kotlin Multiplatform
```

The exact versions of Android Studio, Kotlin, Gradle, Xcode, and related tooling change over time, so always verify the versions recommended by the current Kotlin Multiplatform documentation before starting a new project.

The important architectural point is:

```text
KMP
 │
 ├── Android Toolchain
 │
 └── Apple Toolchain
```

KMP does not remove the need to understand the native platforms.

---

# 2. What the Project Wizard Actually Does

A project wizard is essentially a project generator.

Instead of manually creating:

```text
Gradle files
Source sets
Targets
Android module
iOS integration
Build configuration
```

the wizard creates a starting structure for you.

Conceptually:

```text
Your Choices
     │
     ▼
Project Wizard
     │
     ▼
Project Configuration
     │
     ▼
Gradle / Source Set Structure
     │
     ▼
First KMP Project
```

The wizard is therefore not the architecture.

It is a convenient way to generate an initial architecture.

That distinction matters.

---

# 3. Start With the Project Goal

Before opening the wizard, decide what you are actually trying to build.

For the examples in this book, assume:

```text
Target Platforms:
Android + iOS
```

and:

```text
Shared:
Domain
Data
Networking
Business Logic
```

while initially keeping:

```text
UI:
Platform-specific
```

That gives us a clear target:

```text
                 KMP Application
                       │
             ┌─────────┴─────────┐
             ▼                   ▼
        Android App           iOS App
             │                   │
             └─────────┬─────────┘
                       ▼
                  Shared Module
                       │
             ┌─────────┼─────────┐
             ▼         ▼         ▼
          Domain      Data     Network
```

This is intentionally simple.

The first project should teach the structure before introducing unnecessary complexity.

---

# 4. Opening the Wizard

Depending on the current IDE and installed Kotlin Multiplatform tooling, the exact UI can change.

The names of buttons and screens may evolve.

The concepts remain the same.

You are generally looking for an option to create a new Kotlin Multiplatform project.

The important thing is not to memorize the exact button position.

Instead, understand the decisions the wizard is asking you to make.

---

# 5. Project Name

The first obvious choice is the project name.

For this book, we can use:

```text
KMPBookSample
```

or:

```text
KmpFirstProject
```

The name itself has no special KMP meaning.

It becomes part of the Gradle project and may influence package/module naming depending on the project template.

A clean project name makes examples easier to follow.

---

# 6. Project Location

Choose a directory where the project can live comfortably.

For example:

```text
~/Projects/KmpFirstProject
```

On Windows:

```text
C:\Projects\KmpFirstProject
```

Avoid deeply nested paths or unusual characters when possible.

This isn't specifically a KMP requirement, but simple project paths can eliminate unnecessary tooling problems.

---

# 7. Package Name

A package identifier might look like:

```text
com.example.kmpfirstproject
```

For a real application, use your organization's package naming convention.

For example:

```text
com.company.product
```

The package name is especially important on Android because it participates in application identity.

For iOS, the application identifier is handled through Apple's project and signing configuration.

---

# 8. Choosing the Platforms

This is one of the most important wizard decisions.

You may see options representing platforms such as:

```text
Android
iOS
Desktop
Web
Wasm
```

For our first project, select:

```text
Android
+
iOS
```

Conceptually:

```text
             KMP Project
                  │
          ┌───────┴───────┐
          ▼               ▼
       Android            iOS
```

Don't select every available platform simply because the wizard offers it.

Every additional target creates:

```text
More compilation
More testing
More dependencies
More CI work
More platform-specific decisions
```

Start with the platforms your product actually needs.

---

# 9. Android Target

The Android target is usually the most familiar part for an Android developer.

The project may contain an Android application module or Android target configuration.

Conceptually:

```text
Android Target
      │
      ├── Android source
      ├── Android resources
      └── Shared KMP code
```

The Android application eventually becomes:

```text
APK / AAB
```

through the Android build pipeline.

---

# 10. iOS Target

The iOS target is where Android developers often need to slow down.

The KMP project may generate or configure an iOS integration path.

Conceptually:

```text
KMP Shared Module
       │
       ▼
Kotlin/Native
       │
       ▼
iOS Framework / Native Output
       │
       ▼
Xcode
       │
       ▼
iOS Application
```

The iOS application is still an Apple application.

KMP provides the shared Kotlin implementation.

---

# 11. The iOS Requirement

If you want to build and run the iOS target, you need Apple's development environment.

In practice, this means:

```text
macOS
+
Xcode
```

are part of the iOS development workflow.

This is an important distinction for teams.

You can write and reason about shared KMP code independently of iOS UI development, but building and validating the Apple target requires the appropriate Apple tooling and environment.

---

# 12. Choosing the UI Strategy

Some KMP project templates may ask whether you want to share UI.

This is an architectural choice.

You can think about it as:

```text
Option A
Native UI
│
├── Android → Compose / Views
└── iOS     → SwiftUI / UIKit
```

or:

```text
Option B
Shared UI
│
├── Android
└── iOS
```

For this book's initial architecture, we will use the first model:

```text
              Shared KMP Logic
                     │
          ┌──────────┴──────────┐
          ▼                     ▼
      Android UI             iOS UI
```

This keeps the first project focused on KMP fundamentals rather than introducing multiple architectural decisions simultaneously.

---

# 13. The Wizard Is Not the Architecture

This is worth repeating.

A wizard may produce:

```text
shared/
androidApp/
iosApp/
```

But that doesn't mean these are automatically the perfect modules for your production application.

The wizard gives you:

```text
A Starting Point
```

You still need to decide:

```text
What belongs in shared?
What belongs in Android?
What belongs in iOS?
How should dependencies flow?
How should features be modularized?
```

Architecture comes after understanding the generated project.

---

# 14. Understanding the Generated Structure

After creating the project, you may see something similar to:

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

The exact structure can differ between project templates and tooling versions.

The important idea is:

```text
Application Modules
        +
Shared Module
        +
Gradle Build
```

---

# 15. The Three Important Areas

For a first project, focus on three areas:

```text
1. androidApp
2. shared
3. iosApp
```

Conceptually:

```text
                    Project
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
      androidApp     shared       iosApp
          │            │            │
          ▼            ▼            ▼
       Android      Shared         iOS
       Application   Logic      Application
```

Everything else supports the build.

---

# 16. `androidApp`

The Android application module represents the Android application.

It may contain:

```text
Android UI
Android resources
Android manifest
Android-specific configuration
```

Conceptually:

```text
androidApp
    │
    ├── UI
    ├── Resources
    └── Platform Integration
```

It consumes the shared module.

```text
androidApp
     │
     ▼
shared
```

---

# 17. `iosApp`

The iOS application represents the Apple application.

It can contain:

```text
Swift
SwiftUI
UIKit
iOS resources
Apple-specific configuration
```

Conceptually:

```text
iosApp
    │
    ├── Swift / SwiftUI
    ├── Resources
    └── Platform Integration
```

It also consumes the shared module.

```text
iosApp
    │
    ▼
shared
```

---

# 18. `shared`

This is where the KMP concept becomes visible.

The shared module commonly contains:

```text
commonMain
androidMain
iosMain
commonTest
```

Conceptually:

```text
shared
  │
  ├── commonMain
  │
  ├── androidMain
  │
  ├── iosMain
  │
  └── commonTest
```

Each source set has a specific role.

---

# 19. `commonMain`

`commonMain` is the heart of shared Kotlin code.

Typical contents include:

```text
Domain
Business Logic
Repository Interfaces
Networking
Models
Serialization
Shared State
Use Cases
```

For example:

```text
commonMain
    │
    ├── domain/
    ├── data/
    ├── networking/
    ├── presentation/
    └── platform/
```

The exact package organization is up to you.

---

# 20. `androidMain`

`androidMain` contains Android-specific implementation.

For example:

```text
androidMain
    │
    ├── Android Storage
    ├── Android APIs
    ├── Android-specific implementations
    └── Android integrations
```

This source set is compiled for Android.

---

# 21. `iosMain`

`iosMain` contains iOS-specific implementation.

For example:

```text
iosMain
    │
    ├── iOS Storage
    ├── Apple APIs
    ├── iOS-specific implementations
    └── iOS integrations
```

This source set is compiled for iOS targets.

---

# 22. `commonTest`

Shared tests belong here when the behavior is platform-independent.

For example:

```text
commonTest
    │
    ├── Domain Tests
    ├── Repository Tests
    ├── Validation Tests
    └── Business Rule Tests
```

This is one of the major benefits of KMP.

Instead of maintaining two copies of identical business-rule tests:

```text
Android tests
+
iOS tests
```

you can test shared behavior from common code where appropriate.

---

# 23. The Source Set Relationship

The most important diagram to remember is:

```text
                         shared
                           │
              ┌────────────┼────────────┐
              ▼            ▼            ▼
         commonMain   androidMain    iosMain
              │            │            │
              │            ▼            ▼
              │         Android         iOS
              │
              ▼
       Shared Business Logic
```

The platform-specific source sets can depend on common code.

Conceptually:

```text
androidMain ──► commonMain
iosMain ──────► commonMain
```

The reverse relationship is not the normal architecture.

---

# 24. Common Code Should Stay Platform-Neutral

For example, this belongs naturally in `commonMain`:

```kotlin
class PriceCalculator {

    fun calculate(
        price: Double,
        quantity: Int
    ): Double {
        return price * quantity
    }
}
```

It does not need:

```text
Context
Activity
UIViewController
UIApplication
```

Therefore:

```text
PriceCalculator
       │
       ▼
commonMain
```

---

# 25. Platform Code Should Stay Platform-Specific

For example:

```text
Android Context
```

belongs in Android-specific code.

And:

```text
UIWindow
UIViewController
```

belongs in iOS-specific code.

Conceptually:

```text
commonMain
   │
   ├── Product Rules
   ├── Network Models
   └── Repository

androidMain
   │
   └── Android APIs

iosMain
   │
   └── Apple APIs
```

This boundary is more important than the folder names themselves.

---

# 26. What the Wizard Has Really Given You

After the wizard finishes, don't think:

> "My KMP architecture is complete."

Think:

```text
The wizard has created:
        │
        ▼
A buildable multiplatform skeleton
        │
        ▼
Now I can design the real architecture
```

That mindset prevents many early mistakes.

---

# 27. First Build

The first thing to do after project creation is simple:

> **Build the project before changing anything.**

Why?

Because you want to establish a known-good baseline.

```text
Fresh Project
      │
      ▼
Build
      │
      ▼
Success
      │
      ▼
Known-Good Baseline
```

If the initial project doesn't build, fix the environment before introducing your own code.

---

# 28. Why the First Build Matters

Suppose you immediately add:

```text
Database
Networking
DI
Compose
Navigation
Serialization
```

and then the build fails.

Now you don't know whether the problem came from:

```text
Environment
+
Generated project
+
Your architecture
+
Your dependency
+
Your code
```

A clean first build gives you a reference point.

---

# 29. First Build Mental Model

The first build validates several layers:

```text
IDE
 │
 ▼
Gradle
 │
 ▼
Kotlin
 │
 ▼
KMP Targets
 │
 ├── Android
 │
 └── iOS
```

If this works, your foundation is healthy.

---

# 30. Don't Add Libraries Yet

For the first project:

```text
No database
No networking library
No DI framework
No analytics SDK
No complex architecture
```

Not because these tools are bad.

Because the first goal is to understand:

```text
Project
   ↓
Source Sets
   ↓
Targets
   ↓
Build
   ↓
Runtime
```

Complex dependencies hide the fundamentals.

---

# 31. Explore the Project Before Coding

Open:

```text
shared
```

and inspect:

```text
commonMain
androidMain
iosMain
commonTest
```

Then inspect:

```text
androidApp
iosApp
```

Ask:

```text
Which module owns the UI?
Which module owns shared logic?
Which source set is common?
Which source sets are platform-specific?
```

This exercise is more valuable than immediately writing code.

---

# 32. Read the Gradle Configuration

Open the relevant Gradle configuration.

You will find target declarations and source-set configuration.

Conceptually, you may see something similar to:

```kotlin
kotlin {
    androidTarget()

    iosArm64()
    iosSimulatorArm64()
}
```

The exact syntax can evolve with Kotlin Multiplatform tooling.

The important part is the concept:

```text
This project has:
Android
+
iOS Device
+
iOS Simulator
```

---

# 33. Targets vs Source Sets

These are different concepts.

A **target** describes where code is compiled.

A **source set** describes which source code participates in a compilation.

Think:

```text
Target
  │
  ▼
Compilation Environment
```

and:

```text
Source Set
  │
  ▼
Set of Source Files
```

Together:

```text
Target + Source Set
        │
        ▼
Compilation
```

---

# 34. A Simple Example

Suppose:

```text
commonMain
```

contains:

```kotlin
class Greeting {
    fun message() = "Hello"
}
```

Both Android and iOS targets can compile the shared implementation.

Conceptually:

```text
                    commonMain
                        │
             ┌──────────┴──────────┐
             ▼                     ▼
        Android Target         iOS Target
             │                     │
             ▼                     ▼
        Android Output         iOS Output
```

One source.

Two target-specific outputs.

---

# 35. Why the Wizard Creates Platform Source Sets

The wizard knows that not all code can be shared.

Therefore it provides places for:

```text
commonMain
androidMain
iosMain
```

This gives you an explicit architecture for platform differences.

```text
                    Shared
                      │
              ┌───────┴───────┐
              ▼               ▼
          Android            iOS
```

Instead of scattering platform checks throughout common code.

---

# 36. Avoid Platform Checks Everywhere

A dangerous architecture looks like:

```kotlin
if (isAndroid) {
    // Android
} else {
    // iOS
}
```

repeated throughout the codebase.

Eventually:

```text
Business Logic
   │
   ├── Android condition
   ├── iOS condition
   ├── Android condition
   └── iOS condition
```

The code becomes harder to maintain.

Prefer source-set boundaries or well-defined abstractions.

---

# 37. The Wizard and Package Structure

The wizard may create sample code.

Treat it as:

```text
Example
```

not:

```text
Mandatory Architecture
```

You can reorganize the project later.

For example:

```text
commonMain/
└── kotlin/
    └── com.example/
        ├── domain/
        ├── data/
        ├── networking/
        └── platform/
```

The package structure should evolve with the application.

---

# 38. Naming Matters

Use names that describe responsibility.

Prefer:

```text
ProductRepository
AuthenticationService
PaymentUseCase
SecureStorage
NetworkClient
```

over:

```text
CommonManager
SharedHelper
Utils
GenericService
```

Good names make a multiplatform architecture easier to understand.

---

# 39. Create a Small First Feature

After confirming the project builds, create something tiny.

For example:

```text
Greeting
```

or:

```text
Product Price Calculator
```

A small feature lets us prove:

```text
Shared code
      ↓
Android
      +
iOS
```

without introducing unnecessary infrastructure.

---

# 40. First Shared Class

For example:

```kotlin
class Greeting {

    fun message(): String {
        return "Hello from Kotlin Multiplatform"
    }
}
```

The class has no Android dependency.

It has no iOS dependency.

Therefore:

```text
Greeting
   │
   ▼
commonMain
```

---

# 41. Android Consumes Shared Code

The Android application can call the shared class:

```text
Android UI
     │
     ▼
Greeting
     │
     ▼
commonMain
```

At runtime:

```text
Android
  │
  ▼
Shared Kotlin
  │
  ▼
Result
  │
  ▼
Android UI
```

---

# 42. iOS Consumes Shared Code

The iOS application can also consume the shared implementation:

```text
iOS UI
   │
   ▼
KMP Shared Code
   │
   ▼
Result
   │
   ▼
SwiftUI / UIKit
```

Now we have proven the basic KMP model:

```text
                  Shared Logic
                  /          \
                 /            \
                ▼              ▼
           Android App      iOS App
```

---

# 43. The First Successful Milestone

The first meaningful milestone is not:

```text
100% shared code
```

It is:

```text
One shared piece of behavior
        │
        ├── Android works
        │
        └── iOS works
```

That proves the architecture is connected correctly.

---

# 44. What You Should Inspect After the Wizard

Before moving to the next step, inspect these areas:

```text
✓ settings.gradle.kts
✓ root Gradle configuration
✓ shared module
✓ commonMain
✓ androidMain
✓ iosMain
✓ commonTest
✓ Android application
✓ iOS application
```

You don't need to understand every Gradle line yet.

Focus on relationships.

---

# 45. The Project Graph

After the wizard, visualize the project like this:

```text
                         Root Project
                              │
                ┌─────────────┼─────────────┐
                ▼             ▼             ▼
            androidApp      shared        iosApp
                │             │             │
                │       ┌─────┼─────┐       │
                │       ▼     ▼     ▼       │
                │   common  android  ios     │
                │     Main    Main   Main    │
                │       │      │      │      │
                └───────┴──────┴──────┴──────┘
```

This is the structure you should understand before adding real application logic.

---

# 46. Common Beginner Mistakes

### Mistake 1 — Treating the wizard as architecture

The wizard generates a starting point.

It does not make architectural decisions for your product.

---

### Mistake 2 — Sharing everything

```text
"Everything should go into commonMain."
```

This usually creates unnecessary coupling.

---

### Mistake 3 — Adding too many dependencies

Start small.

Understand the generated build first.

---

### Mistake 4 — Ignoring iOS

If iOS is a target, validate it early.

Don't wait until the end of the project to discover that your architecture doesn't work well on iOS.

---

### Mistake 5 — Modifying Gradle without understanding it

Gradle configuration controls the build graph.

A small change can affect:

```text
Targets
Source Sets
Dependencies
Compilation
Packaging
```

Understand the purpose before changing configuration.

---

# 47. The First Project Checklist

Use this checklist after creating the project:

```text
[ ] Project created successfully
[ ] Android target configured
[ ] iOS target configured
[ ] Shared module exists
[ ] commonMain exists
[ ] androidMain exists
[ ] iosMain exists
[ ] commonTest exists
[ ] Android project builds
[ ] iOS project builds
[ ] Shared sample code compiles
[ ] Android can consume shared code
[ ] iOS can consume shared code
```

Once these are true, you have a working foundation.

---

# 48. What the Wizard Cannot Decide

The wizard cannot know:

```text
Your business domain
Your team structure
Your backend architecture
Your feature boundaries
Your testing strategy
Your deployment process
Your security requirements
Your product roadmap
```

Therefore, after project generation, architecture becomes your responsibility.

---

# 49. The Wizard Is the Starting Line

Think of the wizard as:

```text
                Project Wizard
                      │
                      ▼
                Initial Setup
                      │
                      ▼
              Buildable Project
                      │
                      ▼
              Architecture Work
                      │
                      ▼
             Production Application
```

The wizard gets you to the starting line.

It does not run the race for you.

---

# 50. Final Project Structure

A clean first KMP project can conceptually look like:

```text
KmpFirstProject/
│
├── androidApp/
│   ├── src/
│   └── build.gradle.kts
│
├── shared/
│   ├── src/
│   │   ├── commonMain/
│   │   ├── commonTest/
│   │   ├── androidMain/
│   │   └── iosMain/
│   │
│   └── build.gradle.kts
│
├── iosApp/
│   └── ...
│
├── gradle/
│
├── build.gradle.kts
├── settings.gradle.kts
└── gradle.properties
```

The exact generated files may differ with tooling versions.

The architectural relationship remains:

```text
             Android App
                  │
                  ▼
             Shared KMP
                  ▲
                  │
               iOS App
```

---

# Chapter Takeaways

> [!TIP]
> **The KMP wizard creates a buildable starting point. Your job is to understand and evolve the architecture it creates.**

Remember:

1. Start by defining the platforms your product actually needs.
2. For the first project, Android + iOS is enough to understand the core KMP model.
3. The project wizard generates a starting structure; it does not define your final architecture.
4. `androidApp` represents the Android application.
5. `iosApp` represents the iOS application.
6. `shared` contains the multiplatform implementation.
7. `commonMain` is the primary location for platform-independent Kotlin code.
8. `androidMain` is for Android-specific implementation.
9. `iosMain` is for iOS-specific implementation.
10. `commonTest` is for shared, platform-independent tests.
11. Targets and source sets are related but different concepts.
12. Build the untouched project before adding dependencies.
13. Establish a known-good baseline before changing the project.
14. Don't add libraries before understanding the generated structure.
15. Don't move everything into `commonMain`.
16. Keep platform APIs at platform boundaries.
17. Validate the iOS target early if iOS is part of the product.
18. Treat generated sample code as a starting example, not a mandatory architecture.
19. The first milestone is proving that one piece of shared behavior works on both platforms.
20. The wizard gets the project started; architecture determines whether the project can scale.

---

# Final Mental Model

When you create your first KMP project, don't think:

```text
"I selected KMP in the wizard, so I now have a multiplatform architecture."
```

Think:

```text
                    Project Wizard
                           │
                           ▼
                  Generated Structure
                           │
            ┌──────────────┼──────────────┐
            ▼              ▼              ▼
        Android          Shared           iOS
        Application      Module        Application
            │              │              │
            │        ┌─────┼─────┐        │
            │        ▼     ▼     ▼        │
            │     common android  ios     │
            │      Main   Main   Main     │
            │        │      │      │       │
            └────────┴──────┴──────┴───────┘
                           │
                           ▼
                   Your Architecture
                           │
                           ▼
                  Production KMP App
```

The most important thing to understand at this stage is simple:

> **The wizard creates the skeleton. Understanding the source sets, targets, and module boundaries is what turns that skeleton into a real KMP application.**
