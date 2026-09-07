# Chapter 9 — Kotlin Compiler Deep Dive

## Part 4 — Kotlin/Native

> **Kotlin/Native is Kotlin's native compilation technology. It allows Kotlin code to be compiled into native binaries without requiring a JVM at runtime.**

For Kotlin Multiplatform, Kotlin/Native is particularly important because it provides the compilation path used by platforms such as iOS, macOS, Linux, and other supported native targets.

A useful high-level model is:

```text
Kotlin Source
     │
     ▼
Kotlin Compiler
     │
     ▼
Kotlin IR
     │
     ▼
Native-specific processing
     │
     ▼
Kotlin/Native Backend
     │
     ▼
Native Artifact
     │
     ▼
Platform
```

The exact compiler phases and implementation details evolve with Kotlin versions, so the diagrams in this chapter are intentionally conceptual.

---

## 1. Why Kotlin/Native Exists

Kotlin was originally associated strongly with the JVM ecosystem.

However, Kotlin Multiplatform needs to support platforms where a JVM is not the natural execution environment.

For example:

```text
Android
   ↓
JVM / Android runtime

iOS
   ↓
Native platform environment

macOS
   ↓
Native platform environment

Linux
   ↓
Native platform environment
```

Kotlin/Native provides a way to compile Kotlin into native code suitable for these environments.

The important idea is:

> **Kotlin/Native brings Kotlin to platforms where JVM bytecode is not the target execution format.**

---

## 2. Kotlin/Native in KMP

Consider a typical KMP project:

```text
shared/
├── commonMain/
├── androidMain/
└── iosMain/
```

The same shared Kotlin source can participate in different target compilations.

```text
                 commonMain
                     │
                     ▼
                Kotlin compiler
                     │
                     ▼
                    IR
              ┌──────┴──────┐
              ▼             ▼
          JVM backend    Native backend
              │             │
              ▼             ▼
          Android          iOS
```

For Android, the JVM-oriented path is used.

For iOS, Kotlin/Native provides the native compilation path.

---

## 3. Kotlin/Native Is Not "Kotlin Running on a JVM"

This distinction is fundamental.

### JVM model

```text
Kotlin
  ↓
JVM-oriented compilation
  ↓
JVM bytecode
  ↓
JVM / Android runtime
```

### Native model

```text
Kotlin
  ↓
Kotlin/Native compilation
  ↓
Native-oriented output
  ↓
Platform-native execution
```

There is no JVM requirement for the Kotlin/Native execution model.

This is one of the reasons Kotlin can be used for shared business logic in iOS applications.

---

## 4. A Simplified Native Compilation Pipeline

A useful mental model is:

```text
               Kotlin Source
                    │
                    ▼
             Frontend / Analysis
                    │
                    ▼
                 Kotlin IR
                    │
                    ▼
             IR Transformations
                    │
                    ▼
            Native-specific Lowering
                    │
                    ▼
             Native Code Generation
                    │
                    ▼
             Native Toolchain
                    │
                    ▼
               Native Artifact
```

This is not a complete implementation diagram.

It is a developer-friendly model for understanding the overall direction of compilation.

---

## 5. Kotlin IR and Native

The previous section introduced Kotlin IR.

For Native compilation:

```text
Kotlin
  ↓
IR
  ↓
Native lowerings
  ↓
Native backend
  ↓
Native output
```

This means Kotlin/Native is not a separate Kotlin language.

It is a compilation target and backend path for Kotlin.

The same Kotlin language concepts can therefore be used while the compiler produces output appropriate for a native platform.

---

## 6. Native Targets

Kotlin/Native supports multiple native targets.

Depending on the Kotlin version and supported target matrix, examples include:

```text
Apple targets
├── iOS
├── macOS
├── tvOS
└── watchOS

Linux
├── x64
└── ARM64

Other supported native targets
```

The exact list of supported targets can change over time.

For KMP developers, the important distinction is:

```text
Kotlin/Native
      │
      ├── Apple targets
      ├── Linux targets
      └── other supported native targets
```

---

## 7. iOS and Kotlin/Native

One of the most common KMP scenarios is:

```text
Android application
      +
iOS application
      +
Shared Kotlin code
```

The shared Kotlin module can contain:

```text
Business rules
Networking
Serialization
Persistence abstractions
Validation
Domain models
Use cases
```

The iOS target compiles the relevant Kotlin code through Kotlin/Native.

Conceptually:

```text
commonMain
    │
    ▼
Kotlin/Native
    │
    ▼
iOS-compatible native artifact
    │
    ▼
Swift / Objective-C interoperability
```

---

## 8. Why iOS Does Not Need a JVM

An iOS application normally does not execute application Kotlin code through a JVM.

Instead:

```text
Kotlin shared code
        ↓
Kotlin/Native
        ↓
Native code
        ↓
iOS application
```

This is a major architectural difference from Android.

It also explains why KMP can share Kotlin business logic while still allowing the iOS application to use native Apple technologies.

---

## 9. Kotlin/Native and Apple Frameworks

A common KMP architecture is:

```text
                    Shared Kotlin
                         │
             ┌───────────┴───────────┐
             ▼                       ▼
          Android                    iOS
             │                       │
       Android APIs             Apple APIs
```

Shared Kotlin can remain platform-neutral.

When platform functionality is required, the iOS side can interact with Apple APIs through Kotlin/Native interoperability.

Examples include:

```text
Foundation
UIKit
CoreLocation
Security
UserNotifications
```

The exact APIs available depend on the target and platform SDK.

---

## 10. Kotlin/Native Interoperability

Kotlin/Native provides interoperability with native platform libraries.

For Apple platforms, this enables Kotlin code to interact with Objective-C and platform APIs.

Conceptually:

```text
Kotlin
  │
  ▼
Kotlin/Native
  │
  ▼
Apple interop layer
  │
  ▼
Objective-C / Apple SDK
```

This is one of the mechanisms that makes native platform integration possible.

---

## 11. Example: Foundation API

A Kotlin/Native target can use Apple platform APIs exposed through interoperability.

Conceptually:

```kotlin
import platform.Foundation.NSUUID

fun createIdentifier(): String {
    return NSUUID().UUIDString
}
```

The exact API surface and naming can vary with SDK and Kotlin tooling versions.

The important architectural point is:

```text
commonMain
    → platform-independent code

iosMain
    → Apple-specific integration
```

---

## 12. `iosMain` and Native Code

A typical source-set hierarchy might look like:

```text
commonMain
    │
    └── iosMain
          │
          ├── iosX64Main
          ├── iosArm64Main
          └── iosSimulatorArm64Main
```

This allows developers to share implementation across Apple targets where appropriate.

For example:

```text
iosMain
    │
    ├── shared Apple implementation
    │
    ├── iOS device
    ├── iOS simulator
    └── other Apple targets where applicable
```

The exact hierarchy depends on the project's target configuration.

---

## 13. Native Compilation and Source Sets

The compiler does not simply compile every file in a project for every target.

Instead, the source-set hierarchy determines which source files belong to each compilation.

Conceptually:

```text
commonMain
   │
   ├── Android compilation
   │
   └── iOS compilation
           │
           └── iosMain
```

This is one reason source-set architecture is so important in KMP.

---

## 14. Native Compilation and `expect` / `actual`

Suppose common code declares:

```kotlin
expect fun platformName(): String
```

The iOS source set can provide:

```kotlin
actual fun platformName(): String = "iOS"
```

The compiler resolves the appropriate declaration for the target compilation.

Conceptually:

```text
commonMain
    │
    └── expect platformName()
             │
             ▼
        Native compilation
             │
             ▼
        iosMain actual
```

This allows common code to define a platform contract while the native target supplies the implementation.

---

## 15. Native Compilation Is Target-Aware

Consider:

```text
iosArm64
iosSimulatorArm64
iosX64
```

These are not simply three names for the same output.

They represent different target environments.

The compiler and build system need to know:

```text
Target architecture
Platform
ABI
SDK
Linker requirements
Native libraries
```

This is why KMP builds can have target-specific configuration.

---

## 16. Device vs Simulator

A common Apple development setup contains:

```text
iOS device
    ↓
ARM64

Apple Silicon simulator
    ↓
ARM64

Intel simulator
    ↓
x64
```

Therefore, a KMP project may have multiple iOS targets.

A simplified picture:

```text
                iosMain
                   │
        ┌──────────┼──────────┐
        ▼          ▼          ▼
     iosArm64  simulator   iosX64
                  Arm64
```

Modern Apple development increasingly uses ARM64 simulators on Apple Silicon Macs.

---

## 17. Native Artifact Types

The output consumed by an iOS application depends on how the shared framework is configured and integrated.

Common KMP/iOS integration concepts include:

```text
Framework
XCFramework
Native library
```

An XCFramework can package framework variants for multiple architectures/platform combinations.

The exact artifact structure depends on the build configuration.

---

## 18. Framework Integration

A common KMP setup exposes shared Kotlin code as a framework that can be consumed by an iOS application.

Conceptually:

```text
KMP shared module
       │
       ▼
Kotlin/Native
       │
       ▼
Apple framework
       │
       ▼
iOS application
       │
       ▼
Swift / Objective-C
```

The framework acts as the boundary between the Kotlin/Native output and the iOS application.

---

## 19. Swift Does Not Execute Kotlin Source

When an iOS application uses a KMP framework, Swift does not simply read and execute the Kotlin source files.

Instead:

```text
Kotlin source
    ↓
Kotlin/Native compiler
    ↓
Native framework
    ↓
Swift application
```

The Swift application interacts with the generated native interface.

This distinction is useful when reasoning about performance, APIs, debugging, and interoperability.

---

## 20. Objective-C and Swift Interoperability

Kotlin/Native's Apple interoperability model is strongly connected to Objective-C interoperability.

A simplified path is:

```text
Kotlin declaration
       ↓
Kotlin/Native
       ↓
Objective-C-compatible boundary
       ↓
Swift
```

Modern tooling can make the Swift-facing experience more natural in some scenarios, but the underlying interoperability model remains an important concept for KMP developers.

---

## 21. API Design for Native Consumers

Not every Kotlin API produces an equally pleasant native API.

For example, deeply Kotlin-specific constructs may be awkward at an interop boundary.

When designing shared APIs consumed by Swift, consider:

```text
Naming
Nullability
Generics
Collections
Exceptions
Suspend functions
Sealed hierarchies
Flows
Value classes
```

The goal is not merely:

> "Can Kotlin compile?"

The better question is:

> **"Is the resulting API natural for the native consumer?"**

---

## 22. Kotlin/Native Memory Model

Kotlin/Native has its own runtime and memory-management model.

Modern Kotlin/Native uses a garbage-collected memory management approach.

A useful high-level view is:

```text
Kotlin objects
      ↓
Kotlin/Native runtime
      ↓
Garbage collection
```

This is different from the historical Kotlin/Native freezing model that developers may encounter in older documentation or projects.

Therefore, when reading older KMP material, be careful to distinguish historical behavior from the current memory model.

---

## 23. The Old Freezing Model

Older Kotlin/Native versions used a model involving object freezing and strict concurrency restrictions.

Conceptually:

```text
Mutable object
      ↓
freeze()
      ↓
immutable object
```

This created significant constraints around shared mutable state.

Modern Kotlin/Native has moved away from the old freezing model.

For current development, do not automatically apply historical freezing rules to a modern KMP project.

---

## 24. Modern Concurrency Thinking

With modern Kotlin/Native, concurrency should be understood using current Kotlin concurrency primitives and the current memory model.

For example:

```kotlin
suspend fun loadData(): Data {
    return repository.load()
}
```

The key concern is safe concurrent access and correct coroutine usage, rather than manually freezing every object.

This makes modern Kotlin/Native development considerably closer to the concurrency model developers already know from Kotlin.

---

## 25. Native Runtime

Native-compiled Kotlin still needs runtime support.

A simplified model is:

```text
Native executable
      +
Kotlin/Native runtime
      +
Platform libraries
      ↓
Running application
```

"Native" does not mean "zero runtime."

It means the Kotlin application is compiled for native execution rather than requiring a JVM.

---

## 26. Kotlin/Native and Garbage Collection

Modern Kotlin/Native includes a garbage collector.

Conceptually:

```text
Application objects
       │
       ▼
Kotlin/Native runtime
       │
       ▼
Garbage collector
       │
       ▼
Unused objects reclaimed
```

The exact implementation and performance characteristics depend on Kotlin versions and runtime behavior.

As an application developer, the practical goal remains:

```text
Avoid unnecessary allocations
Use appropriate data structures
Manage lifetimes of resources
Avoid accidental retention
```

---

## 27. Native Resource Management

Garbage collection does not eliminate the need to manage external resources.

Examples:

```text
File handles
Network resources
Database connections
Native resources
Platform objects
Observers
Callbacks
```

A useful distinction is:

```text
Memory lifecycle
        ≠
External resource lifecycle
```

A garbage collector manages memory.

It does not automatically mean that every external resource should be left unmanaged.

---

## 28. Native and Coroutines

Kotlin coroutines can be used in Kotlin/Native applications.

A typical shared API might be:

```kotlin
class UserRepository {
    suspend fun getUser(): User {
        // ...
    }
}
```

The same common API can be used from different targets.

However, the actual dispatching and platform integration still depend on the target environment.

---

## 29. Dispatchers Are Platform-Aware

Consider:

```kotlin
withContext(Dispatchers.Default) {
    calculate()
}
```

The developer uses a common Kotlin API.

The underlying execution is provided by the Kotlin runtime and target implementation.

This is another example of:

```text
Common API
    ↓
Platform-specific implementation
```

KMP allows the API surface to remain common while the underlying platform behavior differs.

---

## 30. Native and Networking

Networking is often placed in `commonMain` using a multiplatform library.

For example:

```text
commonMain
    ↓
HTTP client API
    ↓
Native engine
    ↓
iOS networking stack
```

The exact engine depends on the library and configuration.

The important architectural pattern is:

```text
Shared abstraction
       ↓
Platform-specific engine
```

This lets application logic remain shared without pretending that all platform networking stacks are identical.

---

## 31. Native and Serialization

Serialization can also remain common:

```kotlin
@Serializable
data class User(
    val id: String,
    val name: String
)
```

The compiler and serialization tooling produce the required implementation for the target compilation.

Conceptually:

```text
common Kotlin
      ↓
compiler processing
      ↓
Native compilation
      ↓
iOS artifact
```

The developer does not need to manually rewrite the model in Swift.

---

## 32. Native and Databases

A multiplatform database library can expose a common API:

```text
commonMain
   ↓
Database abstraction
   ↓
Native implementation
   ↓
iOS artifact
```

The exact storage engine and integration are library-specific.

This is an important KMP architectural pattern:

> **Share the application-level abstraction and use the appropriate native implementation underneath it.**

---

## 33. Native and UI

Kotlin/Native itself does not mean that the entire iOS UI must be written in Kotlin.

A KMP architecture can look like:

```text
                 Shared Kotlin
                      │
          ┌───────────┴───────────┐
          ▼                       ▼
       Android                    iOS
          │                       │
     Android UI              SwiftUI / UIKit
```

The shared native code can contain:

```text
Domain
Data
Networking
Persistence
State
Business logic
```

while the iOS UI remains fully native.

This is a common and powerful KMP architecture.

---

## 34. Native vs Compose Multiplatform

Do not confuse Kotlin/Native with Compose Multiplatform.

They solve different problems.

### Kotlin/Native

Focuses on:

```text
Kotlin → native compilation
```

### Compose Multiplatform

Focuses on:

```text
Shared declarative UI
```

A project can use both:

```text
Compose Multiplatform
        +
Kotlin/Native
```

For example, shared Compose UI can ultimately be compiled for native Apple targets.

---

## 35. Native vs Android

The same Kotlin code can compile for both:

```text
Android
   ↓
JVM-oriented target

iOS
   ↓
Kotlin/Native target
```

But this does not mean that runtime behavior is identical.

Differences can appear in:

```text
Memory behavior
Threading
Interop
Available APIs
Binary format
Debugging
Toolchain
Performance characteristics
```

A good KMP architect understands these differences rather than treating all targets as interchangeable.

---

## 36. Native Performance

Native compilation can provide excellent performance, but "native" should not automatically be interpreted as:

> "Every operation is faster than JVM."

Performance depends on:

```text
Algorithm
Allocation patterns
Compiler optimizations
Runtime behavior
Interoperability overhead
I/O
Concurrency
Platform APIs
```

A well-designed application should be measured rather than judged by the word "native."

---

## 37. Native Interop Has a Cost

Calling platform APIs through an interoperability boundary can introduce considerations that do not exist in purely common Kotlin code.

For example:

```text
Kotlin
  ↓
Interop boundary
  ↓
Apple API
```

The boundary may involve:

```text
Object conversion
Type mapping
Memory management
Callback handling
Threading considerations
```

The correct approach is not to avoid interop.

It is to place interop intentionally.

---

## 38. Keep Platform Interop at the Edge

A strong architecture usually keeps platform-specific APIs near platform-specific layers.

Instead of:

```text
commonMain
   ↓
direct Apple API usage everywhere
```

prefer:

```text
commonMain
   ↓
platform abstraction
   ↓
iosMain
   ↓
Apple API
```

For example:

```text
commonMain
    PlatformLogger.log()

iosMain
    AppleLogger → OSLog
```

This keeps business logic portable.

---

## 39. Native and Dependency Resolution

A dependency used by common code must support the target compilation.

For example:

```text
commonMain
    │
    ▼
Multiplatform library
    │
    ├── JVM variant
    └── Native variant
```

When compiling iOS:

```text
iOS compilation
      ↓
dependency resolution
      ↓
Native-compatible variant
```

If a library does not provide a compatible native artifact, it cannot simply be used in common code for iOS.

---

## 40. Why Some Libraries Cannot Be Shared

Suppose a library is JVM-only:

```text
Library
  └── JVM artifact
```

It cannot automatically become an iOS-compatible dependency.

For common KMP code, you need a library with appropriate target support.

Conceptually:

```text
commonMain dependency
       ↓
Does it support iOS?
       │
   ┌───┴───┐
   │       │
  Yes      No
   │       │
   ▼       ▼
Compile   Cannot use
```

This is one of the most common practical KMP dependency questions.

---

## 41. Native and Build Time

Native compilation can involve substantial work.

A simplified build:

```text
Gradle
  ↓
Configure target
  ↓
Resolve dependencies
  ↓
Compile Kotlin
  ↓
Generate IR
  ↓
Native lowerings
  ↓
Native compilation
  ↓
Link
  ↓
Framework / binary
```

Large KMP projects may therefore benefit from careful build optimization.

---

## 42. Native Linking

Native applications ultimately need linked binaries.

Conceptually:

```text
Kotlin object code
      +
Native libraries
      +
Platform frameworks
      ↓
Linker
      ↓
Native binary / framework
```

This differs from a JVM application where classes can be packaged into JVM-oriented artifacts and executed by the JVM.

Native compilation therefore has an explicit linking stage that becomes important when diagnosing build failures.

---

## 43. Common Native Build Problems

Native builds can fail for different reasons.

Examples:

```text
Unsupported target
Missing native dependency
SDK mismatch
Linker error
Interop problem
Architecture mismatch
Framework integration problem
Duplicate symbols
```

When a build fails, identify the layer:

```text
Gradle?
Dependency resolution?
Kotlin compiler?
IR transformation?
Native backend?
Interop?
Linker?
Apple build system?
```

This dramatically narrows debugging.

---

## 44. Linker Errors vs Compiler Errors

These are different categories.

### Compiler error

```text
Kotlin source
   ↓
Compiler
   ✗
```

### Linker error

```text
Kotlin compilation
   ↓
Native object files
   ↓
Linker
   ✗
```

For example, a missing native symbol may not be a Kotlin syntax or type problem.

It may be a linking or native dependency issue.

---

## 45. Architecture Mismatch

Apple platforms can involve multiple architectures.

For example:

```text
Device
  → ARM64

Apple Silicon simulator
  → ARM64

Intel simulator
  → x64
```

If an artifact is built only for one architecture, it may fail when used by another target.

This is one reason XCFramework-style packaging is useful.

---

## 46. XCFramework Concept

An XCFramework can bundle framework variants for multiple Apple platform/architecture combinations.

Conceptually:

```text
                XCFramework
                     │
        ┌────────────┼────────────┐
        ▼            ▼            ▼
     device        simulator    other
      ARM64          ARM64       variants
```

The Xcode build system selects the appropriate slice for the current target.

The exact contents depend on the project's configuration.

---

## 47. Native Debugging

Debugging native code can involve multiple layers:

```text
Kotlin source
      ↓
Kotlin/Native compiler
      ↓
Native binary
      ↓
Xcode / LLDB
```

You may therefore encounter:

```text
Kotlin stack frames
Native stack frames
Apple framework frames
```

A good debugging workflow understands that the application crosses several toolchain boundaries.

---

## 48. Native Exceptions

Kotlin exceptions can be used in shared code:

```kotlin
throw IllegalStateException("Invalid state")
```

However, exposing exceptions directly through a Swift-facing API requires care.

An API designed for Kotlin consumers may not automatically be an ideal Swift API.

For cross-platform APIs, consider explicit result or state models where appropriate.

---

## 49. Native API Design Example

Instead of exposing platform-specific details:

```kotlin
fun loadUser(): AppleSpecificResult
```

prefer a common model:

```kotlin
data class User(
    val id: String,
    val name: String
)
```

and isolate platform-specific conversion at the edge.

```text
commonMain
    User
    Repository
    UseCase

iosMain
    Apple-specific integration
```

This keeps the shared domain model portable.

---

## 50. Native and Platform Services

Platform services should usually be abstracted.

For example:

```kotlin
interface SecureStorage {
    fun put(key: String, value: String)
    fun get(key: String): String?
}
```

Then:

```text
commonMain
    SecureStorage
          │
          ├── Android implementation
          │
          └── iOS implementation
```

The iOS implementation can use the appropriate Apple security APIs.

This keeps the domain layer unaware of platform details.

---

## 51. Native and Dependency Injection

Dependency injection can remain common if the DI library supports the required targets.

A conceptual architecture:

```text
commonMain
    Service interfaces
        ↓
    Dependency graph
        ↓
androidMain / iosMain
    Platform implementations
```

The important point is target compatibility.

A JVM-only DI library cannot automatically become a Native-compatible dependency.

---

## 52. Native and Testing

Common tests can be written in:

```text
commonTest
```

and executed for supported target compilations.

Platform-specific tests can live in:

```text
androidTest
iosTest
```

or target-specific source sets depending on project configuration.

Conceptually:

```text
commonTest
    ↓
Shared behavior tests

ios-specific tests
    ↓
Native integration tests
```

This allows business logic to be tested once while platform integration remains separately testable.

---

## 53. Native and CI

A KMP CI pipeline may contain:

```text
Common tests
     ↓
Android build/tests
     ↓
Native compilation
     ↓
iOS framework/build
```

Apple-target builds generally require Apple tooling and appropriate macOS infrastructure.

This becomes an important consideration when designing CI/CD for a multiplatform project.

---

## 54. Native and Apple Tooling

Kotlin/Native does not replace Apple's development toolchain.

A typical iOS workflow can involve:

```text
Android Studio / IntelliJ
        +
Gradle
        +
Kotlin compiler
        +
Kotlin/Native
        +
Xcode
        +
Apple SDKs
```

Each tool has a different responsibility.

---

## 55. Kotlin/Native vs Swift

Kotlin/Native does not mean Kotlin replaces Swift.

A KMP application can use:

```text
Kotlin
    → shared business logic

Swift
    → iOS-specific UI and integration
```

This hybrid architecture is one of KMP's strongest use cases.

It allows teams to preserve native platform development while sharing the code that benefits most from reuse.

---

## 56. A Practical KMP Architecture

A mature KMP application might look like:

```text
                    Application
                         │
             ┌───────────┴───────────┐
             ▼                       ▼
         Android UI               iOS UI
             │                       │
             └───────────┬───────────┘
                         ▼
                    Shared KMP
                         │
              ┌──────────┼──────────┐
              ▼          ▼          ▼
           Domain      Data       Network
              │          │          │
              └──────────┼──────────┘
                         ▼
                  Platform APIs
                  /            \
             Android          iOS
```

Kotlin/Native provides the compilation path for the iOS/native side of this architecture.

---

## 57. What Kotlin/Native Actually Shares

KMP does not require everything to be shared.

A useful rule is:

```text
Share what represents business behavior.
Keep platform-specific what represents platform behavior.
```

Good candidates for sharing:

```text
Domain models
Validation
Business rules
Use cases
Repositories
Networking abstractions
Serialization
State management
```

Potential platform-specific areas:

```text
UI
Notifications
Permissions
Biometrics
Platform storage
Platform lifecycle
Platform sensors
```

---

## 58. Native and Architecture Boundaries

A clean architecture often has:

```text
commonMain
├── domain
├── data
├── networking
└── shared state

androidMain
└── Android integrations

iosMain
└── Apple integrations
```

The compiler then creates target-specific artifacts from the appropriate source sets.

This is not merely an organizational convention.

It directly affects what can be compiled for each target.

---

## 59. Native Compilation Mental Model

When you see:

```text
iosMain
```

think:

```text
Apple-specific Kotlin
        ↓
Kotlin/Native
        ↓
Native compiler pipeline
        ↓
Apple-compatible artifact
```

When you see:

```text
commonMain
```

think:

```text
Shared Kotlin
        ↓
Target-specific compilation
        ↓
JVM / Native / other backend
```

This mental model prevents a common misconception that `commonMain` itself is a runtime environment.

It is a source-set concept.

---

## 60. The Complete KMP Native Path

Putting the major concepts together:

```text
                       commonMain
                           │
                           ▼
                    Kotlin Compiler
                           │
                           ▼
                       Kotlin IR
                           │
                  ┌────────┴────────┐
                  ▼                 ▼
             JVM backend       Native backend
                  │                 │
                  ▼                 ▼
              Android          Kotlin/Native
                                  │
                                  ▼
                         Native code generation
                                  │
                                  ▼
                           Apple framework
                                  │
                                  ▼
                             Swift / iOS
```

This is the core compiler mental model for KMP developers.

---

## 61. Common Misconceptions

### Misconception 1 — Kotlin/Native means no runtime

**Incorrect.**

Kotlin/Native applications still use runtime support.

The key difference is that they do not require a JVM to execute Kotlin application code.

---

### Misconception 2 — Native means everything is automatically faster

**Incorrect.**

Performance depends on the workload, compiler, runtime, allocations, interoperability, and algorithms.

---

### Misconception 3 — iOS runs Kotlin source directly

**Incorrect.**

Kotlin source is compiled through Kotlin/Native into native-oriented output.

---

### Misconception 4 — Kotlin/Native is only for iOS

**Incorrect.**

Kotlin/Native supports multiple native targets.

iOS is simply one of its most important KMP use cases.

---

### Misconception 5 — Kotlin/Native replaces Swift

**Incorrect.**

KMP can coexist with Swift and native Apple development.

---

### Misconception 6 — Every JVM library works in commonMain

**Incorrect.**

A dependency must support the relevant KMP/native target.

---

## 62. Architect's Checklist

When designing a KMP feature that targets iOS, ask:

```text
□ Is the shared code platform-independent?
□ Does every dependency support Native?
□ Is Apple-specific code isolated?
□ Is the Swift-facing API reasonable?
□ Are device and simulator targets supported?
□ Are native resources managed correctly?
□ Are common tests available?
□ Are platform integration tests needed?
□ Is the CI environment capable of building Apple targets?
□ Have performance assumptions been measured?
```

This checklist catches many problems before they become expensive architectural changes.

---

## 63. Key Takeaways

> **1. Kotlin/Native compiles Kotlin for native targets.**

It allows Kotlin code to run without requiring a JVM.

> **2. Kotlin/Native is a compiler/backend path, not a separate language.**

It uses Kotlin language concepts and the broader Kotlin compiler infrastructure.

> **3. Kotlin IR is an important stage.**

The Kotlin compiler transforms source into IR before target-specific processing and code generation.

> **4. iOS is a major KMP Native target.**

Shared Kotlin code can be compiled into native artifacts consumed by an iOS application.

> **5. Native does not mean platform-independent.**

The final artifact is still target-specific.

> **6. Swift and Kotlin can coexist.**

KMP can share business logic while keeping the iOS UI native.

> **7. Interoperability matters.**

Kotlin/Native provides access to native platform APIs, but API boundaries should be designed carefully.

> **8. Modern Kotlin/Native uses a garbage-collected memory model.**

Older freezing-based guidance should not automatically be applied to modern Kotlin/Native projects.

> **9. Native dependencies must support the target.**

A JVM-only dependency cannot simply be used from iOS common code.

> **10. Native compilation has its own toolchain concerns.**

SDKs, architectures, linking, frameworks, and Apple tooling all matter.

---

## Final Perspective

Kotlin/Native is one of the foundations that makes Kotlin Multiplatform practical for Apple and other native platforms.

The most useful mental model is:

```text
                     Kotlin
                       │
                       ▼
                 Compiler Analysis
                       │
                       ▼
                    Kotlin IR
                       │
                       ▼
               Native Lowerings
                       │
                       ▼
                Native Backend
                       │
                       ▼
                Native Artifact
                       │
             ┌─────────┴─────────┐
             ▼                   ▼
            iOS               macOS
```

For a KMP architect, the most important insight is not simply that Kotlin can compile to native code.

It is that **shared Kotlin remains part of a target-aware compilation system**.

The architecture can therefore preserve:

```text
Shared business logic
        +
Platform-native capabilities
        +
Target-specific compilation
```

rather than forcing Android and iOS into the same runtime model.

A mature KMP application embraces this boundary:

```text
                Shared Kotlin
                     │
        ┌────────────┴────────────┐
        ▼                         ▼
   Android world              Native world
        │                         │
      JVM/ART                 Kotlin/Native
        │                         │
        ▼                         ▼
   Android APIs              Apple APIs
```

That is the real strength of Kotlin Multiplatform: **share the code that benefits from sharing, while allowing each platform to remain native where it matters.**

---

### Quick Reference

| Term | Meaning |
|---|---|
| **Kotlin/Native** | Kotlin technology for compiling to native targets |
| **Native target** | A platform/architecture compiled through the Native toolchain |
| **Kotlin IR** | Intermediate compiler representation |
| **Native backend** | Compiler backend responsible for native-oriented output |
| **iosMain** | Source set for shared iOS-specific Kotlin code |
| **iosArm64** | iOS device target for ARM64 |
| **iosSimulatorArm64** | iOS simulator target for ARM64 |
| **iosX64** | iOS simulator target for x64 |
| **Interop** | Integration between Kotlin and native platform APIs |
| **Framework** | Native package that can be consumed by an iOS application |
| **XCFramework** | Apple framework packaging format supporting multiple variants |
| **Native runtime** | Runtime support used by Kotlin/Native applications |
| **Linking** | Combining compiled code and native libraries into a final artifact |

> **Kotlin/Native is the bridge between Kotlin's multiplatform source model and the native execution environments where KMP applications ultimately run.**
