# Chapter 4 --- Inside Kotlin Multiplatform

## Part 3 --- Compiler Overview

> **KMP does not make Android and iOS run the same binary. It lets the
> same Kotlin source participate in different platform-specific
> compilation processes.**

If the previous part explained **where KMP code lives**, this part
explains **what happens to that code when you build the application**.

For an Android developer, this is an important mental shift.

In a traditional Android application, the compilation path is relatively
familiar:

``` text
Kotlin
   │
   ▼
Kotlin Compiler
   │
   ▼
JVM Bytecode
   │
   ▼
Android Build Toolchain
   │
   ▼
DEX
   │
   ▼
Android Runtime
```

Kotlin Multiplatform introduces another dimension.

The same shared Kotlin source can participate in different target
compilations:

``` text
                         Kotlin Source
                              │
                              ▼
                         Source Sets
                              │
               ┌──────────────┴──────────────┐
               ▼                             ▼
          commonMain                  Platform Source Sets
                                             │
                                  ┌──────────┴──────────┐
                                  ▼                     ▼
                             androidMain             iosMain
                                  │                     │
                                  ▼                     ▼
                              Android                  iOS
                               Target                 Target
                                  │                     │
                                  ▼                     ▼
                           Kotlin/JVM             Kotlin/Native
                                  │                     │
                                  ▼                     ▼
                            Android Output         Native Output
```

That is the foundation of the KMP compiler model.

------------------------------------------------------------------------

# 1. Why Understanding the Compiler Matters

You can use KMP without knowing how the compiler works.

But if you want to **design**, **debug**, and **scale** KMP
applications, the compiler model becomes important.

It helps answer questions such as:

-   Why can this class be placed in `commonMain`?
-   Why can't I import an Android API there?
-   Why does the same Kotlin class work on Android and iOS?
-   Why does iOS need native output?
-   Why are there multiple iOS targets?
-   Why does a common dependency need multiplatform support?
-   What exactly does `expect` / `actual` do?
-   Why does the same source produce different artifacts?

Without this knowledge, KMP can feel like Gradle magic.

With it, the project starts to make sense.

------------------------------------------------------------------------

# 2. KMP Is Not a Runtime

One of the first misconceptions to remove is:

> "KMP is a runtime that executes Kotlin everywhere."

That is not the right mental model.

Think instead:

``` text
                 Kotlin Multiplatform
                         │
                         ▼
                Shared Source Code
                         │
              ┌──────────┴──────────┐
              ▼                     ▼
           Android                  iOS
              │                     │
              ▼                     ▼
         Target-specific       Target-specific
          compilation           compilation
```

KMP is primarily a **compile-time code-sharing technology**.

The application ultimately runs using the mechanisms appropriate to its
target.

------------------------------------------------------------------------

# 3. One Source Does Not Mean One Binary

This is one of the most important sentences in this chapter:

> \[!IMPORTANT\] **One shared source codebase does not mean one
> universal binary.**

Suppose we have:

``` text
commonMain/
└── UserRepository.kt
```

The source exists once.

But it may participate in:

``` text
Android compilation
```

and:

``` text
iOS compilation
```

Conceptually:

``` text
                 UserRepository.kt
                        │
             ┌──────────┴──────────┐
             ▼                     ▼
        Android Build          iOS Build
             │                     │
             ▼                     ▼
       Android Output          iOS Output
```

The source is shared.

The resulting output is target-specific.

------------------------------------------------------------------------

# 4. Kotlin Has Multiple Compilation Targets

Kotlin is not limited to the JVM.

The Kotlin ecosystem includes multiple compilation technologies,
including:

-   Kotlin/JVM
-   Kotlin/Native
-   Kotlin/JS
-   Kotlin/Wasm

For the mobile architecture discussed in this book, the most important
are:

``` text
Android → Kotlin/JVM
iOS     → Kotlin/Native
```

A simplified view:

``` text
                       Kotlin
                         │
              ┌──────────┴──────────┐
              ▼                     ▼
          Kotlin/JVM           Kotlin/Native
              │                     │
              ▼                     ▼
          JVM-based              Native
           output                output
              │                     │
              ▼                     ▼
          Android                 iOS
```

The actual build pipeline contains more stages, but this model is enough
to understand the fundamental difference.

------------------------------------------------------------------------

# 5. Android: The JVM-Based Path

Android developers are already familiar with the JVM model.

A simplified Kotlin-to-Android pipeline looks like:

``` text
Kotlin Source
     │
     ▼
Kotlin/JVM
     │
     ▼
JVM Bytecode
     │
     ▼
Android Build Toolchain
     │
     ▼
DEX
     │
     ▼
Android Application
```

The Android build system then handles the rest of the application
packaging process.

For KMP, shared Kotlin code targeting Android ultimately participates in
this Android-oriented compilation path.

------------------------------------------------------------------------

# 6. iOS: The Native Path

iOS follows a different model.

There is no Android-style ART runtime executing JVM bytecode.

Instead, Kotlin/Native compiles Kotlin code into native output.

Conceptually:

``` text
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
```

For shared KMP code:

``` text
commonMain
     │
     ▼
Kotlin/Native
     │
     ▼
Native iOS-compatible output
     │
     ▼
iOS Application
```

This difference explains why KMP can share Kotlin source while still
producing platform-appropriate applications.

------------------------------------------------------------------------

# 7. The Two Mobile Paths

Put Android and iOS next to each other:

``` text
                 Shared Kotlin Source
                         │
             ┌───────────┴───────────┐
             ▼                       ▼
          Android                    iOS
             │                       │
             ▼                       ▼
        Kotlin/JVM              Kotlin/Native
             │                       │
             ▼                       ▼
        JVM-oriented              Native
           output                 output
             │                       │
             ▼                       ▼
         Android                   iOS
```

This is the core compiler mental model.

------------------------------------------------------------------------

# 8. What Happens to `commonMain`?

Consider:

``` text
shared/
└── src/
    └── commonMain/
        └── kotlin/
            └── Product.kt
```

The code in `commonMain` is not compiled only once and then copied
everywhere.

Instead, it becomes part of the appropriate target compilation.

Conceptually:

``` text
                       Product.kt
                           │
                  ┌────────┴────────┐
                  ▼                 ▼
              Android               iOS
                  │                 │
                  ▼                 ▼
          Android compilation  Native compilation
                  │                 │
                  ▼                 ▼
           Android output       iOS output
```

This is why `commonMain` is better understood as a **shared source
set**, not a universal runtime.

------------------------------------------------------------------------

# 9. Source Sets Define the Compilation Inputs

A simplified project might contain:

``` text
shared/
└── src/
    ├── commonMain/
    ├── commonTest/
    ├── androidMain/
    └── iosMain/
```

For an Android build, conceptually:

``` text
Android Compilation
    │
    ├── commonMain
    └── androidMain
```

For an iOS build:

``` text
iOS Compilation
    │
    ├── commonMain
    └── iosMain
```

So:

``` text
Target
  │
  ▼
Relevant Source Sets
  │
  ▼
Compiler
  │
  ▼
Target Output
```

This is one of the most useful diagrams to remember.

------------------------------------------------------------------------

# 10. Source Set Hierarchy

The source-set model becomes more interesting when there are multiple
targets.

For example:

``` text
                    commonMain
                         │
             ┌───────────┼───────────┐
             ▼           ▼           ▼
         Android        iOS          JVM
                         │
                 ┌───────┴───────┐
                 ▼               ▼
             iosArm64     iosSimulatorArm64
```

The more general source set sits higher in the hierarchy.

More specific source sets sit below it.

This allows code to be shared at different levels.

------------------------------------------------------------------------

# 11. Why Hierarchical Source Sets Matter

Imagine an application targeting:

``` text
Android
iOS
Desktop
```

Some code is common everywhere:

``` text
Business Rules
Models
Validation
```

Some code is common only to JVM targets.

Other code is iOS-specific.

A hierarchical model allows the project to represent those differences:

``` text
                    commonMain
                   /    |     \
                  /     |      \
             Android    JVM     iOS
                         │
                      Desktop
```

The result is more precise sharing.

Instead of choosing between:

``` text
Everything shared
```

or:

``` text
Everything duplicated
```

the source-set hierarchy allows intermediate boundaries.

------------------------------------------------------------------------

# 12. The Compiler Does Not Compile Every File Everywhere

This is important.

Suppose we have:

``` text
commonMain/
    User.kt

androidMain/
    AndroidUserStorage.kt

iosMain/
    IOSUserStorage.kt
```

The iOS compiler should not try to compile:

``` text
AndroidUserStorage.kt
```

And the Android compiler should not try to compile:

``` text
IOSUserStorage.kt
```

The source-set model tells the build which files belong to which target.

Conceptually:

``` text
Android Build
    │
    ├── commonMain
    └── androidMain


iOS Build
    │
    ├── commonMain
    └── iosMain
```

------------------------------------------------------------------------

# 13. Platform-Specific Code Is Not a Failure

This is worth emphasizing.

A KMP project can contain:

``` text
commonMain
androidMain
iosMain
```

and still be an excellent multiplatform architecture.

In fact, that is often exactly what you want.

``` text
                    Shared Logic
                         │
              ┌──────────┴──────────┐
              ▼                     ▼
         Android-specific       iOS-specific
             code                    code
```

The goal isn't to eliminate platform code.

The goal is to eliminate **unnecessary duplication**.

------------------------------------------------------------------------

# 14. A Simple Example

Imagine an application needs to display the current platform.

Common code:

``` kotlin
expect fun platformName(): String
```

Android implementation:

``` kotlin
actual fun platformName(): String {
    return "Android"
}
```

iOS implementation:

``` kotlin
actual fun platformName(): String {
    return "iOS"
}
```

The architecture is:

``` text
                       commonMain
                           │
                           ▼
                         expect
                           │
                  ┌────────┴────────┐
                  ▼                 ▼
             androidMain         iosMain
                  │                 │
                  ▼                 ▼
               actual             actual
```

The compiler understands this relationship during the target
compilation.

------------------------------------------------------------------------

# 15. `expect` / `actual` Is a Compile-Time Relationship

The important idea is not the syntax.

It is the relationship:

``` text
Common Contract
      │
      ├── Android Implementation
      │
      └── iOS Implementation
```

The common code says:

> "A platform implementation must exist."

The target-specific source set supplies that implementation.

This is different from writing:

``` kotlin
if (platform == "Android") {
    ...
}
```

inside shared runtime code.

The platform distinction is represented structurally in the source sets.

------------------------------------------------------------------------

# 16. Interfaces Are Another Option

`expect` / `actual` is not the only way to handle platform-specific
behavior.

You can also use interfaces:

``` kotlin
interface SecureStorage {
    fun save(key: String, value: String)
    fun read(key: String): String?
}
```

Then:

``` text
                 SecureStorage
                       │
                ┌──────┴──────┐
                ▼             ▼
             Android         iOS
          implementation  implementation
```

This can be especially useful when dependency injection is already part
of your architecture.

The important lesson is:

> **KMP gives you mechanisms. Architecture determines how you use
> them.**

------------------------------------------------------------------------

# 17. Where Gradle Fits

KMP compilation is heavily coordinated by Gradle.

For an Android developer, this part will feel familiar.

Gradle manages concepts such as:

``` text
Targets
Source Sets
Dependencies
Compilation Tasks
Tests
Framework Outputs
```

Conceptually:

``` text
                      Gradle
                        │
             ┌──────────┴──────────┐
             ▼                     ▼
        Android Target         iOS Target
             │                     │
             ▼                     ▼
       Kotlin/JVM             Kotlin/Native
             │                     │
             ▼                     ▼
       Android Output          iOS Output
```

Gradle is the orchestrator.

The compiler is responsible for compiling Kotlin.

------------------------------------------------------------------------

# 18. A Simplified Gradle Configuration

A KMP module may contain something conceptually similar to:

``` kotlin
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

The exact syntax and supported configuration evolve with Kotlin and the
Gradle plugins.

The important relationship is:

``` text
KMP Plugin
    │
    ▼
Targets
    │
    ▼
Source Sets
    │
    ▼
Dependencies
    │
    ▼
Compilation
```

------------------------------------------------------------------------

# 19. Target vs Source Set

These terms are easy to mix up.

## Target

A target represents a compilation destination.

Examples:

``` text
Android
iOS ARM64
iOS Simulator ARM64
JVM
```

## Source Set

A source set represents code that can be compiled for one or more
targets.

Examples:

``` text
commonMain
androidMain
iosMain
```

A useful mental shortcut is:

> **Target = where the code goes.**

> **Source set = which code participates.**

------------------------------------------------------------------------

# 20. Dependencies Follow the Same Model

Suppose `commonMain` uses:

``` text
Coroutines
Serialization
Multiplatform networking
```

Those dependencies must support the targets consuming the source set.

Conceptually:

``` text
                  commonMain
                      │
                      ▼
             Multiplatform Library
                      │
             ┌────────┴────────┐
             ▼                 ▼
          Android              iOS
```

But if you place an Android-only library into shared code:

``` text
commonMain
    │
    ▼
Android-only library
```

you have broken the portability assumption.

The build system should expose that incompatibility.

------------------------------------------------------------------------

# 21. The Compiler as an Architectural Guardrail

This is one of the deeper benefits of KMP.

Architecture usually depends on:

``` text
Code Review
+
Documentation
+
Developer Discipline
```

KMP adds:

``` text
Source Set Boundaries
+
Dependency Constraints
+
Compiler Checks
```

So:

``` text
Architectural Boundary
          +
Compiler Enforcement
          =
Stronger Boundary
```

For example, `commonMain` shouldn't casually depend on Android-only
APIs.

The compiler and build configuration help enforce that assumption.

------------------------------------------------------------------------

# 22. Android Build Flow

A simplified Android KMP build looks like:

``` text
                     Gradle
                       │
                       ▼
               Android Target
                       │
             ┌─────────┴─────────┐
             ▼                   ▼
        commonMain          androidMain
             │                   │
             └─────────┬─────────┘
                       ▼
                  Kotlin/JVM
                       │
                       ▼
               Android Toolchain
                       │
                       ▼
                    DEX
                       │
                       ▼
                  APK / AAB
```

This is simplified because the real Android build includes additional
processing for resources, manifests, shrinking, packaging, signing, and
other tasks.

But the compiler model remains the same.

------------------------------------------------------------------------

# 23. iOS Build Flow

The iOS path is different:

``` text
                     Gradle
                       │
                       ▼
                   iOS Target
                       │
             ┌─────────┴─────────┐
             ▼                   ▼
        commonMain             iosMain
             │                   │
             └─────────┬─────────┘
                       ▼
                 Kotlin/Native
                       │
                       ▼
                Native Output
                       │
                       ▼
                     Xcode
                       │
                       ▼
                iOS Application
```

This is where Android developers need to adjust their mental model.

KMP doesn't replace Xcode.

It integrates shared Kotlin code into the iOS application ecosystem.

------------------------------------------------------------------------

# 24. Why Xcode Still Matters

A KMP project targeting iOS still deals with Apple's world:

``` text
Xcode
Swift
SwiftUI
Apple SDKs
Signing
Provisioning
Simulators
Devices
App Store tooling
```

KMP reduces duplicated application logic.

It does not remove the iOS platform.

A multiplatform developer therefore needs to understand both:

``` text
Kotlin / Gradle
```

and:

``` text
Swift / Xcode
```

at an appropriate level.

------------------------------------------------------------------------

# 25. Kotlin/Native

Kotlin/Native is the Kotlin technology that enables Kotlin code to
compile to native binaries without requiring a JVM runtime.

Conceptually:

``` text
Kotlin
  │
  ▼
Kotlin/Native
  │
  ▼
Native Code
  │
  ▼
Platform Binary
```

For iOS:

``` text
Kotlin
  │
  ▼
Kotlin/Native
  │
  ▼
iOS-compatible native output
```

This is why KMP can share Kotlin code with iOS while still producing
native output.

------------------------------------------------------------------------

# 26. Native Does Not Mean "No Compiler"

Sometimes the phrase "native" creates another misconception.

Native does not mean:

``` text
Kotlin magically becomes Swift.
```

Nor does it mean:

``` text
Kotlin source is copied into an iOS project.
```

Instead:

``` text
Kotlin Source
     │
     ▼
Kotlin/Native Compiler
     │
     ▼
Native Output
```

The code remains Kotlin.

The compiler produces target-appropriate native output.

------------------------------------------------------------------------

# 27. Kotlin/JVM vs Kotlin/Native

The difference can be summarized:

                           Kotlin/JVM     Kotlin/Native
  ------------------------ -------------- -------------------------
  Primary role             JVM targets    Native targets
  Android                  ✅             ---
  iOS                      ---            ✅
  JVM runtime              Yes            No JVM runtime required
  Output model             JVM-oriented   Native
  Typical KMP mobile use   Android        iOS

This is a conceptual comparison rather than a complete description of
either compiler.

------------------------------------------------------------------------

# 28. Intermediate Representation

Now we can go one level deeper.

Modern compilers generally do not translate source code directly into
final machine output in a single step.

A simplified compiler pipeline looks like:

``` text
Kotlin Source
     │
     ▼
Frontend
     │
     ▼
Intermediate Representation
     │
     ▼
Backend
     │
     ▼
Target Output
```

Kotlin uses intermediate representations as part of its compiler
architecture.

For multiplatform development, this is important because the
target-specific backend can produce appropriate output from the program
representation.

------------------------------------------------------------------------

# 29. FIR --- Frontend Intermediate Representation

Modern Kotlin compiler architecture includes:

> **FIR --- Frontend Intermediate Representation**

A highly simplified view:

``` text
Kotlin Source
     │
     ▼
Parsing
     │
     ▼
FIR
     │
     ▼
Analysis
     │
     ▼
IR
     │
     ▼
Backend
```

FIR belongs to the compiler's frontend architecture.

It helps the compiler understand Kotlin code and perform semantic
analysis before later compilation stages.

You don't need to understand every FIR implementation detail to build
KMP applications.

But knowing that the compiler has this structured frontend helps explain
why Kotlin can support multiple targets from a common language model.

------------------------------------------------------------------------

# 30. IR --- Intermediate Representation

Another important term is:

> **IR --- Intermediate Representation**

Conceptually:

``` text
Kotlin
  │
  ▼
Frontend
  │
  ▼
IR
  │
  ▼
Target Backend
  │
  ▼
Platform Output
```

The IR acts as an intermediate representation between Kotlin source
semantics and target-specific code generation.

This is especially useful in a multiplatform compiler because the final
output differs by target.

------------------------------------------------------------------------

# 31. One Kotlin Program, Multiple Backends

Think about a simple class:

``` kotlin
class PriceCalculator {

    fun calculate(price: Double): Double {
        return price * 0.9
    }
}
```

The business meaning is the same.

But the target output differs:

``` text
                    PriceCalculator
                           │
                           ▼
                          IR
                           │
                  ┌────────┴────────┐
                  ▼                 ▼
              JVM Backend       Native Backend
                  │                 │
                  ▼                 ▼
             Android             iOS
```

The source stays Kotlin.

The backend determines the target-specific output.

------------------------------------------------------------------------

# 32. Common Code Is Not Converted at Runtime

This is another misconception worth eliminating.

KMP does not wait until application startup and ask:

``` text
"Which platform am I running on?"
```

and then dynamically convert Kotlin code.

Instead:

``` text
                         Build Time
                             │
                             ▼
                      Kotlin Source
                             │
                  ┌──────────┴──────────┐
                  ▼                     ▼
             Android Build          iOS Build
                  │                     │
                  ▼                     ▼
            Target Output          Target Output
```

The important work happens during compilation.

At runtime, the application executes the output appropriate to its
platform.

------------------------------------------------------------------------

# 33. Build Time vs Runtime

Keep these two concepts separate.

### Build Time

``` text
Source
  ↓
Source Set Resolution
  ↓
Compiler
  ↓
Target Output
```

### Runtime

``` text
Target Output
  ↓
Platform Runtime
  ↓
Application
```

KMP's primary sharing mechanism operates at build time.

------------------------------------------------------------------------

# 34. Common APIs and Platform APIs

Now the compiler model explains another important KMP rule.

Code in `commonMain` can only use APIs available to the supported common
targets.

For example:

``` kotlin
fun calculateTotal(
    price: Double,
    quantity: Int
): Double {
    return price * quantity
}
```

This is naturally portable.

But:

``` kotlin
import android.content.Context
```

is Android-specific.

It doesn't belong directly in genuinely shared code.

Conceptually:

``` text
commonMain
    │
    ├── Kotlin/common APIs       ✅
    ├── Multiplatform APIs       ✅
    └── Android-only API         ❌
```

This is not an arbitrary rule.

The compiler needs to be able to compile the source for every target
that consumes it.

------------------------------------------------------------------------

# 35. The Library Problem

Suppose your common code depends on:

``` text
Library A
```

For the code to remain common, Library A needs compatible
implementations for the relevant targets.

Conceptually:

``` text
                 commonMain
                     │
                     ▼
                Library A
                     │
              ┌──────┴──────┐
              ▼             ▼
           Android          iOS
```

If Library A supports only Android:

``` text
commonMain
    │
    ▼
Android-only Library
    │
    └── ❌ iOS
```

The architecture cannot honestly claim that the code is multiplatform.

This is why dependency selection is a major part of KMP design.

------------------------------------------------------------------------

# 36. Platform Libraries Are Still Valuable

The answer isn't:

> "Never use platform libraries."

Quite the opposite.

Android-specific functionality can use Android libraries.

iOS-specific functionality can use Apple frameworks.

The important thing is **where that dependency enters the
architecture**.

``` text
                 Shared Abstraction
                        │
              ┌─────────┴─────────┐
              ▼                   ▼
        Android implementation  iOS implementation
              │                   │
              ▼                   ▼
        Android libraries      Apple frameworks
```

Platform-specific dependencies are perfectly valid when they remain
behind the platform boundary.

------------------------------------------------------------------------

# 37. Kotlin/Native and Apple Frameworks

On iOS, shared Kotlin code can interact with Apple APIs through
Kotlin/Native interoperability.

Conceptually:

``` text
Kotlin Shared Code
       │
       ▼
Kotlin/Native
       │
       ▼
Apple Frameworks
       │
       ▼
iOS Application
```

This is one of the reasons KMP is not simply a replacement for native
iOS development.

Kotlin can reach into the platform when necessary.

------------------------------------------------------------------------

# 38. Kotlin-to-Swift Interoperability

A KMP module may expose functionality that an iOS application consumes.

Conceptually:

``` text
Kotlin
  │
  ▼
Kotlin/Native
  │
  ▼
iOS Framework Boundary
  │
  ▼
Swift / SwiftUI
```

This introduces another architectural consideration:

> **The API needs to be understandable on the consuming platform.**

An API that looks beautiful in Kotlin isn't automatically a beautiful
Swift API.

------------------------------------------------------------------------

# 39. Shared Internal APIs vs Platform-Facing APIs

A useful pattern is to distinguish between:

``` text
Internal Kotlin API
```

and:

``` text
Platform-facing API
```

For example:

``` text
                  Shared Module
                       │
              ┌────────┴────────┐
              ▼                 ▼
       Internal Kotlin      iOS-facing
          architecture         API
              │                 │
              ▼                 ▼
        Kotlin developers     Swift
```

Inside the shared module, use idiomatic Kotlin.

At the boundary, design APIs carefully for the consuming platform.

This becomes especially important for:

-   Coroutines
-   Flow
-   Exceptions
-   Sealed hierarchies
-   Generics
-   Complex Kotlin types

------------------------------------------------------------------------

# 40. Compilation Does Not Guarantee Good API Design

A project can successfully compile and still expose a poor API to iOS.

For example:

``` text
BUILD SUCCESSFUL
```

doesn't automatically mean:

``` text
Swift API
    =
Beautiful API
```

The compiler checks technical correctness.

The architecture and API design still require human judgment.

------------------------------------------------------------------------

# 41. Testing Follows the Same Model

The compiler model also affects testing.

Shared logic:

``` text
commonTest
```

can test:

``` text
Business Rules
Validation
Data Transformations
State Logic
```

Platform-specific tests can cover:

``` text
Android APIs
iOS APIs
Hardware
Platform Integration
```

Conceptually:

``` text
                    Tests
                      │
             ┌────────┴────────┐
             ▼                 ▼
        Shared Tests      Platform Tests
             │             ┌────┴────┐
             ▼             ▼         ▼
        commonTest      Android      iOS
```

The same principle applies:

> **Share tests where behavior is shared; keep platform tests where
> behavior is platform-specific.**

------------------------------------------------------------------------

# 42. Multiple iOS Targets

An iOS application can involve multiple architecture targets.

For example:

``` text
iosArm64
iosSimulatorArm64
```

Conceptually:

``` text
                       iOS
                        │
              ┌─────────┴─────────┐
              ▼                   ▼
         Physical Device       Simulator
              │                   │
          iosArm64        iosSimulatorArm64
```

This is why a project may appear to have several iOS targets even though
the product is simply "an iOS application."

The compiler needs to produce output appropriate for the architecture
and environment.

------------------------------------------------------------------------

# 43. One Platform Can Have Multiple Targets

This is an important general lesson.

We previously simplified:

``` text
Android
iOS
```

But the real model can be:

``` text
Android
iOS device
iOS simulator
Desktop JVM
Web
```

Therefore:

``` text
Platform
   ≠
Target
```

A platform can contain multiple compilation targets.

------------------------------------------------------------------------

# 44. The Full Build Graph

We can now draw a more complete picture:

``` text
                         KMP Project
                              │
                              ▼
                            Gradle
                              │
                   ┌──────────┴──────────┐
                   ▼                     ▼
              Android Target        iOS Targets
                   │                ┌────┴─────┐
                   │                ▼          ▼
                   │            iosArm64   iosSimulatorArm64
                   │                │          │
                   ▼                └────┬─────┘
              Source Sets               │
                   │                    │
            ┌──────┴──────┐             │
            ▼             ▼             ▼
       commonMain    androidMain     iosMain
            │             │             │
            └──────┬──────┴─────────────┘
                   ▼
              Compilation
                   │
          ┌────────┴────────┐
          ▼                 ▼
       JVM-based          Native
        output            output
          │                 │
          ▼                 ▼
       Android             iOS
```

This is much closer to how you should mentally visualize a KMP build.

------------------------------------------------------------------------

# 45. What the Compiler Actually Gives You

The compiler provides several important guarantees.

### Source compatibility

``` text
Can this common source compile for this target?
```

### Dependency compatibility

``` text
Are the required APIs available for this target?
```

### Platform implementation completeness

``` text
Does the required platform implementation exist?
```

### Type correctness

``` text
Are the types and expressions valid?
```

### Target output

``` text
Can the source be transformed into target-specific output?
```

But it does not answer:

``` text
Should this code be shared?
```

That remains an architectural decision.

------------------------------------------------------------------------

# 46. The Compiler Does Not Know Your Business

Suppose you have:

``` text
OrderValidator
```

The compiler doesn't know whether:

``` text
OrderValidator
```

should be:

``` text
commonMain
```

or:

``` text
androidMain
```

You decide based on product behavior.

If order validation is identical on Android and iOS:

``` text
commonMain
```

is a strong candidate.

If the logic is specifically tied to Android hardware:

``` text
androidMain
```

may be appropriate.

The compiler executes your architecture.

It doesn't invent it.

------------------------------------------------------------------------

# 47. The Architect's View

From an architect's perspective, the entire process can be summarized:

``` text
                   Product Requirement
                          │
                          ▼
                  Architecture Decision
                          │
                          ▼
                   Sharing Boundary
                          │
                          ▼
                     Source Sets
                          │
                          ▼
                       Gradle
                          │
                          ▼
                      Compiler
                          │
             ┌────────────┴────────────┐
             ▼                         ▼
          Android                      iOS
             │                         │
             ▼                         ▼
         JVM-based                  Native
          output                    output
```

Architecture comes first.

Compilation follows the architecture.

------------------------------------------------------------------------

# 48. A Real Example

Consider an e-commerce application.

The shared module contains:

``` text
commonMain/
├── domain/
│   ├── Cart.kt
│   ├── Product.kt
│   └── CalculateDiscount.kt
│
├── data/
│   └── ProductRepository.kt
│
└── networking/
    └── ProductApi.kt
```

Android-specific:

``` text
androidMain/
└── platform/
    └── AndroidSecureStorage.kt
```

iOS-specific:

``` text
iosMain/
└── platform/
    └── IOSSecureStorage.kt
```

The build graph becomes:

``` text
                    E-Commerce App
                          │
                ┌─────────┴─────────┐
                ▼                   ▼
             Android                iOS
                │                   │
                └─────────┬─────────┘
                          ▼
                    commonMain
                          │
             ┌────────────┼────────────┐
             ▼            ▼            ▼
           Domain        Data       Networking
                          │
                   Platform boundary
                    ┌─────┴─────┐
                    ▼           ▼
                 Android       iOS
```

This is where the compiler model meets real architecture.

------------------------------------------------------------------------

# 49. Why This Is Different From "Write Once, Run Anywhere"

The phrase:

> **Write once, run anywhere**

can be useful as a high-level marketing concept.

But it is not the best architectural model for KMP.

A more accurate mental model is:

> **Write common logic once, compile it for the platforms that need it,
> and keep platform-specific responsibilities where they belong.**

That is much closer to how KMP actually works.

------------------------------------------------------------------------

# 50. The KMP Compilation Equation

You can summarize the entire chapter with this:

``` text
                 Shared Kotlin Source
                          +
                 Platform Source Sets
                          +
                    Target Config
                          │
                          ▼
                     Compiler
                          │
              ┌───────────┴───────────┐
              ▼                       ▼
        Android Output            iOS Output
```

Or even more simply:

``` text
One Source
   +
Multiple Targets
   =
Multiple Platform Outputs
```

------------------------------------------------------------------------

# 51. What You Should Remember

> \[!IMPORTANT\] **KMP shares source code at the language level while
> allowing each target to have its own compilation path and
> platform-specific implementation.**

Keep these concepts clear:

  Concept         Mental Model
  --------------- -----------------------------------------
  `commonMain`    Shared source
  `androidMain`   Android-specific source
  `iosMain`       iOS-specific source
  Target          Compilation destination
  Kotlin/JVM      JVM compilation technology
  Kotlin/Native   Native compilation technology
  FIR             Kotlin compiler frontend representation
  IR              Intermediate compiler representation
  Gradle          Build orchestration
  Xcode           Apple application/build ecosystem
  Artifact        Target-specific compiled output

------------------------------------------------------------------------

# 52. The One Diagram to Remember

If you remember only one diagram from this part, use this:

``` text
                         SHARED KOTLIN
                              │
                              ▼
                         commonMain
                              │
                 ┌────────────┴────────────┐
                 │                         │
                 ▼                         ▼
           Android Build              iOS Build
                 │                         │
          commonMain +               commonMain +
          androidMain                 iosMain
                 │                         │
                 ▼                         ▼
            Kotlin/JVM               Kotlin/Native
                 │                         │
                 ▼                         ▼
         Android-compatible          Native iOS
              output                   output
                 │                         │
                 ▼                         ▼
             Android                    iOS
```

The source is shared.

The compilation is target-specific.

The output is platform-specific.

That is KMP.

------------------------------------------------------------------------

# 53. Chapter Takeaways

> \[!TIP\] **The compiler is the bridge between shared Kotlin source and
> platform-specific applications.**

Remember:

1.  KMP is not a universal runtime.
2.  KMP primarily enables source sharing through multiplatform
    compilation.
3.  Shared source does not mean shared binary.
4.  Android commonly uses the Kotlin/JVM compilation path.
5.  iOS uses Kotlin/Native.
6.  `commonMain` participates in compatible target compilations.
7.  Platform source sets provide target-specific implementations.
8.  Source-set hierarchies allow sharing at different levels.
9.  A platform can contain multiple compilation targets.
10. Gradle orchestrates the build configuration and tasks.
11. The Kotlin compiler performs the language compilation.
12. FIR and IR are important parts of Kotlin's modern compiler
    architecture.
13. Platform-specific APIs should remain behind appropriate boundaries.
14. Multiplatform dependencies need compatible target implementations.
15. `expect` / `actual` provides one mechanism for platform-specific
    implementations.
16. Interfaces and dependency injection can provide another.
17. Xcode remains important for iOS development.
18. KMP does not eliminate native platform tooling.
19. Compilation can enforce some architectural boundaries.
20. The compiler cannot decide what your application should share---that
    remains an architectural decision.

------------------------------------------------------------------------

# Final Mental Model

When you see:

``` text
commonMain/
```

don't think:

> "This code somehow runs everywhere."

Think:

> **"This Kotlin source is eligible to participate in the compilation of
> multiple supported targets."**

Then visualize:

``` text
                    commonMain
                        │
             ┌──────────┴──────────┐
             ▼                     ▼
        Android Build          iOS Build
             │                     │
             ▼                     ▼
        Kotlin/JVM            Kotlin/Native
             │                     │
             ▼                     ▼
       Android Output         Native Output
             │                     │
             ▼                     ▼
          Android                  iOS
```

Once this model is clear, KMP stops looking like magic.

It becomes a predictable relationship between:

``` text
Source
  ↓
Source Sets
  ↓
Targets
  ↓
Compiler
  ↓
Platform Output
```

And that leads to the next important question:

> **If `commonMain` has to compile for multiple platforms, what can we
> actually put inside it?**

That question takes us to the next layer: **common APIs, platform APIs,
dependencies, and the rules that determine whether a piece of Kotlin
code is truly multiplatform.**
