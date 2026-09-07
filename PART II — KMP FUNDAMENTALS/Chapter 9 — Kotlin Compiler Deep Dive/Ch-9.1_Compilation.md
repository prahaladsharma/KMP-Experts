# Chapter 9 — Kotlin Compiler Deep Dive

## Part 1 — Compilation

> **Kotlin Multiplatform is not magic. Shared Kotlin code becomes platform-specific artifacts through a compilation pipeline designed around Kotlin, source sets, target backends, and platform toolchains.**

Understanding compilation is important because many KMP problems that look like dependency, source-set, or runtime issues are actually compilation problems.

The useful mental model is:

```text
Kotlin source
     ↓
Gradle configuration
     ↓
Kotlin compilation
     ↓
Target-specific backend
     ↓
Platform artifact
```

---

## 1. What Does "Compilation" Mean?

Compilation transforms source code into a form that a target platform can execute or consume.

For Kotlin Multiplatform:

```text
Kotlin source
     ↓
Target-specific compilation
     ↓
Platform-specific output
```

The same common source can participate in Android, iOS, desktop, WebAssembly, or other supported target compilations.

The compiler therefore needs to understand:

- Which source files belong to the compilation
- Which dependencies are available
- Which declarations are visible
- Which `expect` declarations require `actual` implementations
- Which target is being compiled
- Which backend and toolchain are required

---

## 2. Compilation Starts With Source Sets

A KMP project organizes code into source sets.

For example:

```text
commonMain
commonTest

androidMain
androidUnitTest

iosMain
iosTest
```

For a particular target, Gradle and the Kotlin Multiplatform plugin determine the relevant source sets and dependencies.

Conceptually:

```text
Android compilation

commonMain
    +
androidMain
    +
target dependencies
    ↓
Android compilation
```

For iOS:

```text
iOS compilation

commonMain
    +
iosMain
    +
target dependencies
    ↓
Native compilation
```

The source-set model is one of the foundations of KMP.

---

## 3. Gradle Configures the Compilation

Gradle orchestrates the build. The Kotlin compiler performs Kotlin compilation.

A KMP configuration might look like:

```kotlin
kotlin {
    androidTarget()

    iosArm64()
    iosSimulatorArm64()
}
```

Dependencies are associated with source sets:

```kotlin
kotlin {
    sourceSets {
        commonMain.dependencies {
            implementation("...")
        }

        androidMain.dependencies {
            implementation("...")
        }
    }
}
```

The mental model is:

```text
Gradle configuration
        ↓
Kotlin compilations
        ↓
Compiler inputs
```

Gradle determines what should be built and invokes the relevant compiler tasks.

---

## 4. A Compilation Is Target-Specific

KMP does not create one universal binary containing every platform implementation.

Each target has its own compilation.

```text
                     commonMain
                         │
          ┌──────────────┼──────────────┐
          │              │              │
       Android          iOS          Desktop
          │              │              │
       JVM/            Native          JVM/
       Android         backend         Native
```

The common source participates in multiple target compilations.

That is why the same business logic can be reused while platform-specific code remains isolated.

---

## 5. The Compiler Sees a Compilation Context

A target compilation has a context containing information such as:

```text
Source files
Dependencies
Source-set hierarchy
Expected declarations
Actual declarations
Target information
Compiler options
Language/API settings
```

So compilation is more than:

```text
.kt → binary
```

A better model is:

```text
Project model
      +
Source-set graph
      +
Dependencies
      +
Target
      +
Compiler configuration
      ↓
Compilation
```

---

## 6. Common Code Is Not Compiled "Once"

A common misconception is:

> "`commonMain` is compiled once and then copied to Android and iOS."

A more useful model is that common source participates in each relevant target compilation.

```text
commonMain
   │
   ├── Android compilation
   │
   ├── iOS compilation
   │
   ├── Desktop compilation
   │
   └── WebAssembly compilation
```

This means common code is validated in the context of each target that consumes it.

---

## 7. `expect` / `actual` Participates in Compilation

Consider:

```kotlin
// commonMain
expect fun platformName(): String
```

And:

```kotlin
// androidMain
actual fun platformName(): String = "Android"
```

The common declaration defines the expected contract, while the platform source set provides the implementation.

Conceptually:

```text
commonMain
    expect platformName()
           │
           ▼
    target compilation
           │
           ├── Android → actual platformName()
           └── iOS     → actual platformName()
```

The compiler verifies that the expected declaration can be satisfied for the target.

This is different from runtime reflection or string-based platform dispatch.

---

## 8. Compilation-Time Platform Selection

Consider:

```kotlin
expect class Platform {
    val name: String
}
```

Android:

```kotlin
actual class Platform {
    actual val name: String = "Android"
}
```

iOS:

```kotlin
actual class Platform {
    actual val name: String = "iOS"
}
```

Common code does not need to contain:

```kotlin
if (platform == "Android") {
    ...
}
```

The target compilation has the appropriate platform implementation available.

---

## 9. Kotlin Compiler Frontend

At a high level, the compiler first needs to understand the Kotlin program.

The frontend performs activities such as:

```text
Parsing
    ↓
Syntax analysis
    ↓
Symbol resolution
    ↓
Type checking
    ↓
Semantic analysis
```

For example:

```kotlin
val user: User
```

requires the compiler to determine:

```text
What is User?
Is User visible?
Is the type valid?
Is the declaration available in this compilation?
```

Modern Kotlin compiler architecture uses a shared frontend approach, while target-specific backends handle output generation.

---

## 10. Parsing Kotlin Source

The compiler reads Kotlin source such as:

```kotlin
class User(
    val id: String,
    val name: String
)
```

It must understand:

```text
Class declaration
Constructor
Properties
Types
Identifiers
Syntax
```

The source is transformed into compiler-internal representations for subsequent analysis.

---

## 11. Symbol Resolution

Suppose the source contains:

```kotlin
val repository = UserRepository()
```

The compiler must resolve:

```text
UserRepository
```

to the correct declaration.

It considers:

- Imports
- Package declarations
- Source-set visibility
- Dependencies
- Overloads
- Generic types
- Platform-specific declarations

This is why dependency and source-set configuration directly affect compilation.

---

## 12. Type Checking

Kotlin's type system is a major part of compilation.

For example:

```kotlin
val count: Int = "10"
```

is rejected because:

```text
String
  ≠
Int
```

KMP compilation also validates whether declarations used by common code are valid in the selected target context.

---

## 13. Semantic Analysis

The compiler checks whether the program follows Kotlin's language rules.

Examples include:

```text
Unresolved references
Invalid overrides
Incorrect visibility
Type mismatches
Generic constraints
Nullability violations
Incorrect declaration usage
```

Many familiar compiler errors originate from these checks.

---

## 14. Intermediate Representation

After frontend analysis, the compiler needs an internal representation suitable for backend processing.

Kotlin uses an intermediate representation, commonly referred to as **IR**.

A simplified model is:

```text
Kotlin source
      ↓
Frontend analysis
      ↓
Kotlin IR
      ↓
Target backend
      ↓
Target artifact
```

IR provides a common representation that can be processed by different target backends.

This is important because Android/JVM and Kotlin/Native have different output environments.

---

## 15. Why Intermediate Representation Matters

Without a shared intermediate representation, each backend would need to independently understand much more of the Kotlin language.

A useful conceptual model is:

```text
Kotlin source
      ↓
Shared frontend
      ↓
Intermediate representation
      ↓
Target backend
```

This separates:

```text
Understanding Kotlin
```

from:

```text
Generating target-specific output
```

---

## 16. Backend Compilation

The backend takes the analyzed program representation and generates target-specific output.

A simplified view:

```text
                 Kotlin IR
                    │
        ┌───────────┼───────────┐
        │           │           │
       JVM        Native       Web
        │           │           │
        ▼           ▼           ▼
      JVM/       Native       JS/Wasm
     Android     artifact      output
```

The actual build pipeline contains additional stages and platform tools, but this model is useful for understanding the architecture.

---

## 17. Android Compilation

For Android, Kotlin code is compiled for the JVM/Android environment and participates in the Android build pipeline.

A simplified path is:

```text
Kotlin source
      ↓
Kotlin compiler
      ↓
JVM-oriented output
      ↓
Android build pipeline
      ↓
APK / AAB
```

The Android build may additionally involve:

```text
Resources
Manifest processing
Java compilation
Bytecode transformation
DEX generation
Packaging
Signing
```

KMP integrates with the Android build system rather than replacing it.

---

## 18. iOS Compilation

iOS follows a different path because iOS applications use native Apple platform artifacts.

Conceptually:

```text
Kotlin source
      ↓
Kotlin compiler
      ↓
Kotlin/Native compilation
      ↓
Native framework/library artifact
      ↓
Apple build pipeline
      ↓
iOS application
```

The resulting artifact can be consumed by the Apple application.

---

## 19. Kotlin/Native Compilation

Kotlin/Native compiles Kotlin code for native targets.

A simplified model is:

```text
Kotlin
  ↓
Frontend
  ↓
IR
  ↓
Native backend
  ↓
Native artifact
```

The output is intended for the selected native platform and architecture.

For example, an iOS build can target device or simulator architectures depending on the configured target.

---

## 20. Compilation Is Architecture-Aware

A KMP project can declare multiple targets:

```kotlin
kotlin {
    androidTarget()

    iosArm64()
    iosSimulatorArm64()

    jvm()
}
```

These targets can have different:

```text
Toolchains
Dependencies
Native APIs
Architectures
Output artifacts
```

The common layer provides shared source, while target-specific compilation supplies the environment.

---

## 21. Source-Set Hierarchy Influences Compilation

Consider:

```text
commonMain
    │
    ├── appleMain
    │      ├── iosMain
    │      └── macosMain
    │
    └── androidMain
```

An iOS compilation can consume:

```text
commonMain
+
appleMain
+
iosMain
```

This allows code to be shared at the correct level.

The compiler therefore works with a source-set graph rather than a flat directory structure.

---

## 22. Dependencies Are Part of Compilation Inputs

Consider:

```kotlin
commonMain.dependencies {
    implementation("some.library:core:1.0.0")
}
```

That dependency becomes part of the compilation environment for the relevant common source.

If a dependency is added only to:

```kotlin
androidMain
```

common code cannot assume it exists.

The key relationship is:

```text
Dependency declared in source set
             ↓
Available to that source set's compilation
```

---

## 23. Platform Dependencies

Platform-specific dependencies belong in the appropriate platform source set.

For example:

```kotlin
androidMain.dependencies {
    implementation("android-specific-library")
}
```

This keeps Android-only APIs out of common compilation.

Conceptually:

```text
commonMain
    ↓
Portable dependencies

androidMain
    ↓
Android dependencies

iosMain
    ↓
Apple/native dependencies
```

---

## 24. Compilation Errors vs Runtime Errors

One major benefit of KMP is that certain problems can be detected at compile time.

Runtime-oriented design:

```text
Start application
     ↓
Detect platform
     ↓
Load implementation
     ↓
Maybe fail
```

Compile-time-oriented design:

```text
Configure target
     ↓
Resolve source sets
     ↓
Resolve declarations
     ↓
Compile
     ↓
Build artifact
```

If a required `actual` implementation is missing, the problem can be detected before the application reaches a user.

---

## 25. Example: Missing `actual`

Suppose:

```kotlin
// commonMain
expect fun currentTimeMillis(): Long
```

but a target does not provide the required actual implementation.

The target compilation cannot satisfy the contract.

The problem is therefore discovered during the build rather than through a runtime platform branch.

---

## 26. Example: Wrong Dependency Scope

Suppose:

```kotlin
commonMain
```

contains:

```kotlin
import android.some.api.SomeClass
```

The common compilation should reject this because the Android-specific API is not part of the common platform contract.

The compiler therefore acts as an architectural guard.

---

## 27. Compilation Does Not Mean Execution

A successful compilation means the compiler accepted the program and produced the required output.

It does not guarantee:

```text
Correct business behavior
Correct UI behavior
Correct lifecycle handling
Correct native API usage
Correct performance
```

Testing remains necessary.

---

## 28. Incremental Compilation

Large KMP projects can contain many Kotlin source files.

Recompiling everything after every small change would be expensive.

Build systems therefore use incremental compilation techniques to avoid unnecessary work.

Conceptually:

```text
Source change
    ↓
Determine affected inputs
    ↓
Recompile affected parts
    ↓
Reuse unaffected outputs
```

Exact behavior depends on the Kotlin version, build configuration, and target.

---

## 29. Why Incremental Compilation Matters

Suppose a developer changes:

```text
commonMain/UserFormatter.kt
```

The changed common code may affect multiple target compilations:

```text
Android
iOS
Desktop
Web
```

depending on the project.

A change in common code therefore has a potentially broader compilation impact than a change in a single platform source set.

---

## 30. Platform-Only Changes Have Smaller Scope

If a developer changes:

```text
androidMain/AndroidLogger.kt
```

the iOS compilation generally does not need to rebuild that implementation.

Conceptually:

```text
androidMain change
      ↓
Android compilation
```

while:

```text
commonMain change
      ↓
Multiple target compilations
```

This is another reason source-set boundaries matter for build performance.

---

## 31. Compiler Configuration Matters

Compilation can be influenced by:

```text
Kotlin version
Language/API settings
Compiler options
Target configuration
Opt-in requirements
Platform toolchain
```

A source file that compiles under one toolchain configuration may require changes under another.

Reproducible builds therefore depend on controlled versions and consistent configuration.

---

## 32. Kotlin Compiler vs Gradle

These are related but different.

### Gradle

Gradle is the build orchestrator:

```text
Task graph
Dependencies
Configuration
Build lifecycle
Plugin integration
Artifact packaging
```

### Kotlin compiler

The Kotlin compiler performs Kotlin compilation:

```text
Parsing
Resolution
Type checking
IR generation
Backend compilation
```

A useful mental model is:

```text
Gradle
  │
  ├── configures
  ├── schedules
  └── invokes
          │
          ▼
    Kotlin compiler
          │
          ▼
       artifacts
```

---

## 33. Compiler Plugins

Kotlin supports compiler plugins that can extend or modify compiler behavior.

They are different from ordinary runtime libraries.

A library provides code your application can call.

A compiler plugin participates during compilation.

Conceptually:

```text
Source
  +
Compiler plugin
  ↓
Compiler
  ↓
Generated/compiled output
```

Compiler-plugin compatibility can have requirements beyond ordinary library compatibility.

---

## 34. The Compilation Pipeline

A useful high-level diagram is:

```text
┌───────────────────────────────┐
│         Kotlin Source         │
│ commonMain / platform source  │
└───────────────┬───────────────┘
                │
                ▼
┌───────────────────────────────┐
│       Gradle Configuration    │
│ targets / source sets / deps  │
└───────────────┬───────────────┘
                │
                ▼
┌───────────────────────────────┐
│       Kotlin Compiler         │
│ parse / resolve / type-check  │
└───────────────┬───────────────┘
                │
                ▼
┌───────────────────────────────┐
│       Intermediate Form       │
│              IR               │
└───────────────┬───────────────┘
                │
        ┌───────┼────────┐
        │       │        │
        ▼       ▼        ▼
       JVM    Native    Web
        │       │        │
        ▼       ▼        ▼
     Android    iOS    JS/Wasm
```

This is a conceptual model rather than a complete implementation diagram.

---

## 35. What Happens When You Run a Build?

A simplified KMP build looks like:

```text
./gradlew build
        │
        ▼
Gradle loads project
        │
        ▼
KMP plugin configures targets
        │
        ▼
Source sets and dependencies resolved
        │
        ▼
Compilation tasks selected
        │
        ▼
Kotlin compiler invoked
        │
        ▼
Frontend analysis
        │
        ▼
IR processing
        │
        ▼
Target backend
        │
        ▼
Platform artifacts
        │
        ▼
Packaging / tests / verification
```

The real task graph is more detailed, but this sequence provides a useful mental model.

---

## 36. Why KMP Compilation Feels Different From Android

An Android developer may initially think:

```text
Kotlin
  ↓
JVM
  ↓
APK
```

KMP introduces:

```text
Kotlin
  ↓
Shared source model
  ↓
Multiple target compilations
  ↓
Different backends/toolchains
  ↓
Different artifacts
```

The conceptual shift is:

> **The source code can be shared even though the compilation target is not.**

---

## 37. Common Code Is Compiled in Context

Suppose:

```kotlin
// commonMain

class UserService(
    private val repository: UserRepository
)
```

The compiler needs to know:

```text
Which UserRepository?
Which dependencies?
Which source-set declarations?
Which expected declarations?
Which target?
Which compiler configuration?
```

Compilation is therefore contextual.

This is why source-set architecture and dependency management have a direct relationship with compiler behavior.

---

## 38. The Compiler as an Architectural Guard

Good KMP architecture allows compiler errors to protect boundaries.

For example:

```text
commonMain
    ❌ Android Context
    ❌ UIKit UIViewController
    ❌ platform-specific dependency

androidMain
    ✅ Android APIs

iosMain
    ✅ Apple APIs
```

Instead of relying only on code review, the source-set model and compiler can reject invalid platform coupling.

---

## 39. Compilation Does Not Mean "Shared"

A source file can participate in multiple compilations without every platform implementation being identical.

For example:

```text
commonMain
    business rules

androidMain
    Android integration

iosMain
    iOS integration
```

The goal is not:

```text
100% identical code
```

The goal is:

```text
Maximum appropriate reuse
+
Correct platform integration
```

---

## 40. A Practical Debugging Model

When a KMP compilation fails, ask:

### 1. Which target is failing?

```text
Android?
iOS?
Desktop?
WebAssembly?
```

### 2. Which source set is involved?

```text
commonMain?
androidMain?
iosMain?
```

### 3. Is the dependency available to that source set?

```text
common dependency?
platform dependency?
```

### 4. Is the declaration platform-specific?

```text
expect?
actual?
native API?
```

### 5. Is the correct target configured?

```text
Target declaration
architecture
toolchain
```

### 6. Is the compiler/toolchain version compatible?

This sequence often narrows the problem quickly.

---

## 41. Common Compilation Mistakes

### Mistake 1 — Platform API in `commonMain`

```kotlin
import android.content.Context
```

**Problem:** common compilation cannot rely on Android-only APIs.

### Mistake 2 — Dependency Added to the Wrong Source Set

```text
Library
   ↓
androidMain
```

but used from:

```text
commonMain
```

**Problem:** the common compilation does not have that dependency.

### Mistake 3 — Missing `actual`

```kotlin
expect fun readDeviceId(): String
```

with no matching platform implementation.

**Problem:** the target compilation cannot satisfy the expected declaration.

### Mistake 4 — Incorrect Target

The project expects an iOS compilation but the required target or architecture is not configured correctly.

**Problem:** the requested compilation cannot be produced.

### Mistake 5 — Assuming All Targets Share the Same APIs

```kotlin
commonMain
    ↓
platform-specific API
```

**Problem:** the common source becomes tied to one target.

---

## 42. Compilation vs Linking vs Packaging

These concepts are related but not identical.

### Compilation

Transforms source into compiled output.

```text
Kotlin → compiled output
```

### Linking

Combines compiled code and required native components into a usable native artifact where applicable.

```text
Compiled objects
+
Libraries
+
Native components
↓
Native artifact
```

### Packaging

Creates the final distributable artifact.

```text
Code
+
Resources
+
Metadata
+
Platform packaging
↓
Application/package
```

A build can therefore fail after Kotlin source compilation has already succeeded.

---

## 43. Why "It Compiles" Is Not the End

Consider:

```text
commonMain
       ↓
Compiler succeeds
       ↓
Android app builds
       ↓
iOS framework builds
```

That still does not prove:

```text
Correct runtime behavior
```

A mature KMP project also needs:

```text
Unit tests
Integration tests
Platform tests
UI tests
Static analysis
CI
Release validation
```

---

## 44. Compilation and API Design

A good common API makes compilation work for you.

For example:

```kotlin
interface SecureStorage {
    fun save(key: String, value: String)
    fun read(key: String): String?
}
```

The common code depends on a stable contract.

Platform code handles:

```text
Android implementation
iOS implementation
```

The compiler then verifies that the target-specific implementation fits the expected API.

---

## 45. Compilation and Architectural Boundaries

A well-structured KMP project often follows:

```text
                   commonMain
                       │
             ┌─────────┴─────────┐
             │                   │
        Business logic       Interfaces
             │                   │
             └─────────┬─────────┘
                       │
                Platform boundary
                  /           \
                 /             \
          Android             iOS
             │                  │
        Native APIs        Native APIs
```

The compiler helps maintain these boundaries.

---

## 46. The Most Important Mental Model

Do not think of KMP as:

```text
Write Kotlin once
        ↓
Convert it to every platform
```

A better model is:

```text
Write shared Kotlin
        ↓
Organize it into source sets
        ↓
Create target-specific compilations
        ↓
Analyze common + platform code
        ↓
Generate target-specific output
        ↓
Integrate with the platform build
```

This explains much of what you see in real KMP projects.

---

## 47. Compilation Summary

Remember the pipeline as:

```text
SOURCE
  ↓
Source sets
  ↓
Dependencies
  ↓
Target configuration
  ↓
Compiler frontend
  ↓
Resolution + type checking
  ↓
IR
  ↓
Target backend
  ↓
Platform artifact
```

And the architectural relationship:

```text
commonMain
    ↓
shared declarations + business logic

platform source sets
    ↓
platform implementations

target compilation
    ↓
validates the complete platform-specific program
```

---

## 48. Key Takeaways

> **1. KMP does not compile one universal application binary.**

Each target has its own compilation.

> **2. `commonMain` participates in target-specific compilations.**

Shared code is compiled in the context of each supported target.

> **3. Gradle orchestrates; the Kotlin compiler compiles.**

Understanding this separation makes build problems easier to diagnose.

> **4. Source sets define compilation boundaries.**

They determine which code and dependencies are available to a compilation.

> **5. `expect` / `actual` is part of the multiplatform compilation model.**

Platform implementations are not selected through ad-hoc runtime platform checks.

> **6. Kotlin IR provides a common compiler representation.**

Target backends can then produce platform-specific output.

> **7. Android and iOS follow different downstream toolchains.**

Shared Kotlin does not mean identical platform compilation.

> **8. Compilation is an architectural safety mechanism.**

A good source-set and dependency structure allows the compiler to reject invalid platform coupling.

---

## Final Perspective

The Kotlin compiler is more than a tool that turns `.kt` files into binaries.

In a KMP project, it sits at the center of a larger model:

```text
                    KMP PROJECT
                         │
              ┌──────────┴──────────┐
              │                     │
         Common Source         Platform Source
              │                     │
              └──────────┬──────────┘
                         │
                  Target Compilation
                         │
              ┌──────────┼──────────┐
              │          │          │
            JVM        Native       Web
              │          │          │
              ▼          ▼          ▼
          Android       iOS       JS/Wasm
```

The power of Kotlin Multiplatform comes from this separation.

You write common logic once, but you do not ask every platform to become the same platform.

> **The compiler validates the shared model, target-specific code supplies platform behavior, and each backend produces the artifact required by its platform.**

Once this mental model is clear, concepts such as source sets, `expect` / `actual`, dependencies, Gradle tasks, Kotlin/Native, and platform artifacts become much easier to reason about.

---

### Quick Reference

| Concept | Responsibility |
|---|---|
| **Gradle** | Orchestrates the build |
| **KMP plugin** | Configures targets, source sets, and compilations |
| **Source sets** | Define groups of source and dependency inputs |
| **Kotlin compiler** | Analyzes and compiles Kotlin |
| **Frontend** | Parsing, resolution, type and semantic analysis |
| **IR** | Intermediate compiler representation |
| **Backend** | Generates target-specific output |
| **`expect` / `actual`** | Connects common contracts with platform implementations |
| **Kotlin/JVM** | JVM-oriented compilation |
| **Kotlin/Native** | Native platform compilation |
| **JS / Wasm** | Web-oriented compilation |
| **Packaging tools** | Produce final platform application/artifacts |

> **Compiler knowledge turns KMP from a collection of Gradle configurations into a system you can reason about.**
