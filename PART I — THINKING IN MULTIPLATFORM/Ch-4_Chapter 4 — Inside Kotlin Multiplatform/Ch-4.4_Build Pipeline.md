# Chapter 4 — Inside Kotlin Multiplatform

## Part 4 — Build Pipeline

> **A KMP project may look like one application in the IDE, but under the hood it is a coordinated build graph producing platform-specific outputs.**

Once the compiler model is clear, the next question is practical:

> **What actually happens when we press Build?**

For a traditional Android application, most developers are comfortable with:

```text
Source Code
    ↓
Gradle
    ↓
Kotlin Compiler
    ↓
Android Build Tools
    ↓
APK / AAB
```

KMP expands this into a multi-target pipeline:

```text
                         KMP Project
                              │
                              ▼
                           Gradle
                              │
                 ┌────────────┴────────────┐
                 ▼                         ▼
          Android Target              iOS Target
                 │                         │
                 ▼                         ▼
          Source Set Graph          Source Set Graph
                 │                         │
                 ▼                         ▼
           Kotlin/JVM              Kotlin/Native
                 │                         │
                 ▼                         ▼
        Android Artifacts          Native Artifacts
                 │                         │
                 ▼                         ▼
             Android                   Xcode
                                        │
                                        ▼
                                     iOS App
```

Understanding this pipeline is critical because many KMP problems that appear to be "Kotlin problems" are actually **build configuration, target, dependency, or integration problems**.

---

# 1. The Build Is a Graph, Not a Single Command

When you run:

```bash
./gradlew build
```

it is tempting to imagine:

```text
build
  ↓
compile
  ↓
package
```

A real KMP project is closer to:

```text
                         build
                           │
             ┌─────────────┴─────────────┐
             ▼                           ▼
       Android tasks                iOS tasks
             │                           │
       ┌─────┴─────┐               ┌─────┴─────┐
       ▼           ▼               ▼           ▼
    Compile      Test           Compile      Test
       │           │               │           │
       └─────┬─────┘               └─────┬─────┘
             ▼                           ▼
          Package                    Framework
```

The actual task graph is significantly larger.

The important idea is:

> **A KMP build is a graph of target-specific tasks connected by shared inputs and dependencies.**

---

# 2. Gradle Is the Orchestrator

KMP relies heavily on Gradle to coordinate the build.

Gradle doesn't simply "compile Kotlin."

It coordinates things such as:

- targets
- source sets
- dependencies
- compilation tasks
- tests
- packaging
- generated sources
- native framework outputs
- Android integration

A simplified model:

```text
                    Gradle
                      │
          ┌───────────┼───────────┐
          ▼           ▼           ▼
       Android       iOS          Tests
        Target      Targets
          │           │
          ▼           ▼
       Compiler    Compiler
          │           │
          ▼           ▼
       Artifact    Artifact
```

Think of Gradle as the **build coordinator**.

The Kotlin compiler is one of the major workers inside that system.

---

# 3. The Build Lifecycle

A simplified Gradle lifecycle looks like:

```text
Initialization
      │
      ▼
Configuration
      │
      ▼
Task Graph
      │
      ▼
Execution
      │
      ▼
Artifacts
```

Let's look at these stages in the context of KMP.

---

# 4. Stage 1 — Initialization

Gradle first determines which build is being executed.

It identifies the project structure and included builds/modules.

For example:

```text
KMP Project
│
├── androidApp
├── shared
└── iosApp
```

The Gradle build needs to understand:

```text
What projects exist?
What modules are included?
What build configuration applies?
```

Conceptually:

```text
Gradle Invocation
       │
       ▼
Project Discovery
       │
       ▼
Module Structure
```

---

# 5. Stage 2 — Configuration

Gradle then evaluates the build configuration.

This is where plugins, targets, source sets, dependencies, and other configuration are established.

For example:

```kotlin
kotlin {
    androidTarget()

    iosArm64()
    iosSimulatorArm64()
}
```

Conceptually:

```text
KMP Configuration
       │
       ├── Android Target
       │
       ├── iOS Device Target
       │
       └── iOS Simulator Target
```

The build now knows what it needs to produce.

---

# 6. Stage 3 — Task Graph Creation

Gradle determines which tasks are required for the requested operation.

For example:

```bash
./gradlew build
```

may involve tasks related to:

```text
Compilation
Testing
Packaging
Verification
```

A simplified graph:

```text
                         build
                           │
             ┌─────────────┴─────────────┐
             ▼                           ▼
       Android Build                Native Build
             │                           │
       ┌─────┴─────┐               ┌─────┴─────┐
       ▼           ▼               ▼           ▼
    Compile      Test           Compile      Test
```

Gradle then executes the tasks in the required order.

---

# 7. Stage 4 — Compilation

Now the Kotlin compiler becomes heavily involved.

For Android:

```text
commonMain
    +
androidMain
    │
    ▼
Kotlin/JVM
```

For iOS:

```text
commonMain
    +
iosMain
    │
    ▼
Kotlin/Native
```

Conceptually:

```text
                    Source Sets
                         │
             ┌───────────┴───────────┐
             ▼                       ▼
          Android                    iOS
             │                       │
             ▼                       ▼
        Kotlin/JVM              Kotlin/Native
```

---

# 8. Stage 5 — Testing

After compilation, tests can run depending on the requested Gradle task.

Shared tests may live in:

```text
commonTest
```

Platform-specific tests may exist for:

```text
Android
iOS
```

Conceptually:

```text
                  Tests
                    │
          ┌─────────┴─────────┐
          ▼                   ▼
      commonTest         Platform Tests
                              │
                     ┌────────┴────────┐
                     ▼                 ▼
                  Android             iOS
```

This allows business logic to be tested once where appropriate while platform-specific behavior receives platform-specific testing.

---

# 9. Stage 6 — Packaging

Compilation doesn't automatically mean you have an application.

The output still needs to be packaged.

For Android:

```text
Kotlin
  ↓
JVM-oriented output
  ↓
Android Build
  ↓
DEX
  ↓
APK / AAB
```

For iOS:

```text
Kotlin
  ↓
Kotlin/Native
  ↓
Native Output
  ↓
Framework / Integration
  ↓
Xcode
  ↓
iOS Application
```

Packaging is therefore another layer of the build pipeline.

---

# 10. The Complete High-Level Pipeline

We can now visualize the entire process:

```text
                       Developer
                           │
                           ▼
                    Gradle Command
                           │
                           ▼
                       Gradle
                           │
                    ┌──────┴──────┐
                    ▼             ▼
                Android           iOS
                 Target          Target
                    │             │
                    ▼             ▼
              Source Sets    Source Sets
                    │             │
                    ▼             ▼
               Kotlin/JVM   Kotlin/Native
                    │             │
                    ▼             ▼
               Android       Native Output
               Toolchain          │
                    │             ▼
                    ▼           Xcode
                APK / AAB          │
                                   ▼
                                iOS App
```

This is the mental model you should carry into the rest of KMP development.

---

# 11. The Shared Module Is Not the Application

A common KMP project often has a structure similar to:

```text
project/
│
├── androidApp/
│
├── shared/
│
└── iosApp/
```

These modules have different responsibilities.

```text
                 Application
                      │
        ┌─────────────┼─────────────┐
        ▼             ▼             ▼
    androidApp      shared        iosApp
        │             │             │
        ▼             ▼             ▼
     Android       Shared          iOS
        UI          Logic           UI
```

The `shared` module is usually where multiplatform logic lives.

It is not necessarily the complete application.

---

# 12. The Android Application Build

A common architecture might look like:

```text
androidApp
     │
     ├── Android UI
     ├── Android resources
     └── Android-specific integration
             │
             ▼
          shared
```

The Android application consumes the shared module.

Conceptually:

```text
Android UI
    │
    ▼
Shared KMP Module
    │
    ▼
Shared Business Logic
```

The final Android application is assembled using the Android build system.

---

# 13. The iOS Application Build

The iOS application follows a similar conceptual structure:

```text
iosApp
   │
   ├── Swift / SwiftUI
   ├── iOS resources
   └── Apple-specific integration
            │
            ▼
         shared
```

Conceptually:

```text
Swift / SwiftUI
      │
      ▼
KMP Shared Framework
      │
      ▼
Shared Kotlin Logic
```

This gives you a powerful architecture:

```text
Android UI ──────┐
                 │
                 ▼
            Shared Logic
                 ▲
                 │
iOS UI ──────────┘
```

---

# 14. What Happens When You Build Android?

Let's follow the Android path more carefully.

Imagine:

```bash
./gradlew assembleDebug
```

The high-level sequence is:

```text
Gradle
  │
  ▼
Android Debug Build
  │
  ▼
Resolve Dependencies
  │
  ▼
Compile Kotlin
  │
  ▼
Process Android Code
  │
  ▼
DEX
  │
  ▼
Package
  │
  ▼
APK
```

KMP adds the shared module into this process:

```text
commonMain
    +
androidMain
    │
    ▼
Android compilation
    │
    ▼
shared artifact
    │
    ▼
androidApp
    │
    ▼
APK
```

---

# 15. What Happens When You Build iOS?

The iOS process is different.

The Kotlin portion can be built using Gradle and Kotlin/Native.

Conceptually:

```text
Gradle
  │
  ▼
iOS Target
  │
  ▼
Resolve Dependencies
  │
  ▼
Compile commonMain + iosMain
  │
  ▼
Kotlin/Native
  │
  ▼
Native Framework / Output
  │
  ▼
Xcode
  │
  ▼
iOS Application
```

This is one of the biggest differences an Android developer experiences when moving into KMP.

---

# 16. Why There Are Two Build Worlds

A KMP developer is effectively working across two ecosystems:

```text
                 KMP Project
                     │
           ┌─────────┴─────────┐
           ▼                   ▼
       Android World        Apple World
           │                   │
        Gradle              Xcode
           │                   │
      Kotlin/JVM          Swift / Native
           │                   │
           ▼                   ▼
       Android               iOS
```

KMP creates the bridge.

It doesn't erase the boundaries.

---

# 17. Gradle and Xcode Are Partners

It is tempting to think:

```text
Gradle vs Xcode
```

But a better model is:

```text
Gradle
  │
  ▼
Kotlin/Native
  │
  ▼
Shared Native Output
  │
  ▼
Xcode
  │
  ▼
iOS Application
```

Each tool has a role.

### Gradle

Coordinates:

- Kotlin compilation
- KMP targets
- dependencies
- shared module build
- framework generation

### Xcode

Coordinates:

- iOS application build
- Swift compilation
- Apple SDK integration
- signing
- provisioning
- application packaging

---

# 18. The Dependency Resolution Stage

Before compilation can happen, dependencies must be resolved.

Suppose:

```kotlin
commonMain.dependencies {
    implementation("some.multiplatform:library")
}
```

Gradle must determine which artifacts are appropriate for the target.

Conceptually:

```text
Shared Dependency
       │
       ▼
Dependency Resolution
       │
 ┌─────┴─────┐
 ▼           ▼
Android      iOS
Artifact    Artifact
```

This is one reason multiplatform libraries need to publish target-compatible artifacts.

---

# 19. Why Dependency Resolution Can Fail

Consider:

```text
commonMain
    │
    ▼
Library A
```

If Library A supports:

```text
Android ✅
iOS ❌
```

then the shared source cannot simply use it as if it were universally available.

You may see build failures during dependency resolution or compilation.

The architecture should instead look like:

```text
commonMain
    │
    ▼
Multiplatform API
    │
    ├── Android implementation
    │
    └── iOS implementation
```

or use a genuinely multiplatform dependency.

---

# 20. Dependency Resolution Is Target-Aware

This is an important concept.

A KMP dependency isn't simply:

```text
download one JAR
```

and use it everywhere.

Different targets may consume different artifacts.

Conceptually:

```text
                    Library
                       │
              ┌────────┴────────┐
              ▼                 ▼
           Android              iOS
              │                 │
              ▼                 ▼
        JVM artifact       Native artifact
```

The developer sees one dependency declaration.

The build system may resolve target-specific components behind the scenes.

---

# 21. Compilation Depends on Dependency Resolution

The relationship is:

```text
Dependencies
      │
      ▼
Resolution
      │
      ▼
Source Compilation
      │
      ▼
Target Output
```

If dependency resolution fails:

```text
❌ Compilation cannot proceed correctly
```

If compilation fails:

```text
❌ Packaging cannot proceed
```

Therefore:

> **A KMP build is a chain of dependent stages.**

---

# 22. Source Set Metadata

KMP libraries also publish metadata that helps Gradle understand their multiplatform structure.

Conceptually:

```text
KMP Library
     │
     ├── Common metadata
     ├── Android artifact
     ├── iOS artifact
     └── Other target artifacts
```

The consuming build uses this information to select the appropriate component.

This is one of the mechanisms that makes multiplatform dependency resolution possible.

---

# 23. Build Variants Still Matter

Android developers already know:

```text
debug
release
```

KMP projects can still live inside an Android application that has:

```text
Debug
Release
Staging
Production
```

and other variants.

For example:

```text
Android
  │
  ├── Debug
  └── Release
```

The shared module may therefore participate in different application builds.

This becomes important when configuration differs between environments.

---

# 24. Build Configuration Is Not Business Logic

A common mistake is allowing build configuration to leak into application architecture.

For example:

```text
Production API
Staging API
Development API
```

should not result in business logic becoming full of Gradle-specific conditionals.

Instead, think:

```text
Build Configuration
       │
       ▼
Environment Configuration
       │
       ▼
Application
```

The build pipeline provides configuration.

The application consumes it.

---

# 25. Generated Code

Modern builds may also generate code.

For example:

```text
Schema
  │
  ▼
Code Generator
  │
  ▼
Generated Kotlin
  │
  ▼
Compiler
```

A KMP project can therefore contain:

```text
Hand-written source
+
Generated source
```

both participating in the compilation.

When debugging build failures, remember that the source you see in the IDE may not be the complete set of sources passed to the compiler.

---

# 26. Incremental Compilation

Large projects can become expensive to build if everything is recompiled after every change.

Modern Kotlin and Gradle builds use incremental techniques to avoid unnecessary work.

Conceptually:

```text
Previous Build
      │
      ▼
Detect Changes
      │
      ▼
Compile Affected Parts
      │
      ▼
Reuse Unchanged Outputs
```

Instead of:

```text
Change one file
      │
      ▼
Compile entire project
```

the build system attempts to minimize the amount of work.

---

# 27. Why KMP Build Performance Matters

A KMP project may have:

```text
Android target
iOS device target
iOS simulator target
Tests
Shared libraries
Generated code
```

As the project grows, build performance can become a real engineering concern.

You may see:

```text
Long configuration time
Long dependency resolution
Slow compilation
Repeated native compilation
Large CI builds
```

This is why build engineering should not be treated as an afterthought.

---

# 28. Gradle Configuration vs Execution

One subtle but important Gradle concept is the difference between:

```text
Configuration
```

and:

```text
Execution
```

Conceptually:

```text
Gradle
  │
  ├── Configuration Phase
  │       │
  │       ▼
  │    Task Graph
  │
  └── Execution Phase
          │
          ▼
       Tasks Run
```

A poorly designed build can spend significant time during configuration before actual compilation even starts.

For large KMP projects, understanding this distinction becomes valuable.

---

# 29. Task Graph Example

A simplified task dependency might look like:

```text
compileCommon
      │
      ▼
compileAndroid
      │
      ▼
packageAndroid
      │
      ▼
assemble
```

For iOS:

```text
compileCommon
      │
      ▼
compileIos
      │
      ▼
linkFramework
      │
      ▼
Xcode Build
```

The real Gradle task graph is more complex.

The important idea is that tasks have dependencies.

Gradle executes them in the correct order.

---

# 30. Compilation vs Linking

These two concepts are easy to confuse.

### Compilation

Turns source code into a lower-level representation or compiled output.

```text
Kotlin Source
     ↓
Compiler
     ↓
Compiled Code
```

### Linking

Combines compiled pieces and dependencies into a final native artifact.

```text
Compiled Code
     +
Libraries
     +
Runtime Components
     ↓
Linked Output
```

For native targets, linking is especially important.

---

# 31. Kotlin/Native Linking

A simplified native pipeline:

```text
Kotlin Source
      │
      ▼
Kotlin/Native Compiler
      │
      ▼
Compiled Native Code
      │
      ▼
Linking
      │
      ▼
Native Framework / Binary
```

The final result needs to contain or reference the required native components.

This is one reason native build failures can look very different from JVM compilation errors.

---

# 32. Why Native Build Errors Feel Different

An Android developer may be used to errors such as:

```text
Unresolved reference
Type mismatch
Duplicate class
```

Native builds can introduce different categories:

```text
Linker errors
Architecture mismatch
Framework integration issues
Apple SDK issues
Symbol resolution problems
```

For example:

```text
Device target
      │
      ▼
ARM64
```

cannot simply consume an incompatible binary built for another architecture.

Understanding the build pipeline makes these errors easier to reason about.

---

# 33. Device vs Simulator

One practical example is iOS.

You might build for:

```text
Physical iPhone
```

or:

```text
Simulator
```

These can require different target artifacts.

Conceptually:

```text
                 iOS Shared Code
                       │
             ┌─────────┴─────────┐
             ▼                   ▼
        Device Target       Simulator Target
             │                   │
             ▼                   ▼
         iosArm64        iosSimulatorArm64
```

This is not merely a deployment detail.

It is part of the compilation and artifact model.

---

# 34. Framework Generation

For iOS integration, KMP can produce a framework that the Apple application can consume.

Conceptually:

```text
commonMain
    +
iosMain
    │
    ▼
Kotlin/Native
    │
    ▼
iOS Framework
    │
    ▼
Xcode
```

The framework acts as a bridge between the Kotlin shared module and the iOS application.

---

# 35. The Framework Boundary

Think of the framework as:

```text
┌───────────────────────────────┐
│        Kotlin Shared          │
│                               │
│  Domain                       │
│  Data                         │
│  Networking                   │
│  Business Logic               │
└───────────────┬───────────────┘
                │
                ▼
        iOS Framework Boundary
                │
                ▼
┌───────────────────────────────┐
│            iOS                │
│                               │
│  Swift / SwiftUI              │
│  Apple SDKs                   │
└───────────────────────────────┘
```

This boundary is extremely important for architecture.

---

# 36. What Happens in CI?

The same pipeline runs in CI.

A simplified pipeline might look like:

```text
                    CI
                     │
                     ▼
                Checkout
                     │
                     ▼
              Restore Caches
                     │
                     ▼
              Resolve Dependencies
                     │
                     ▼
              Compile Shared Code
                     │
             ┌───────┴───────┐
             ▼               ▼
          Android            iOS
             │               │
             ▼               ▼
           Tests            Tests
             │               │
             ▼               ▼
          Package           Framework
             │               │
             └───────┬───────┘
                     ▼
                  Artifacts
```

For production teams, CI becomes an important part of the KMP build architecture.

---

# 37. CI and Local Builds Should Be Predictable

A healthy build should behave consistently:

```text
Developer Machine
       │
       ▼
     Build
       │
       ▼
      CI
       │
       ▼
   Same Source
       │
       ▼
 Predictable Output
```

If something builds locally but fails consistently in CI, investigate:

- JDK version
- Kotlin version
- Gradle version
- Xcode version
- dependency resolution
- native SDK configuration
- architecture
- environment variables
- signing configuration

---

# 38. Caching

KMP builds can benefit significantly from caching.

Conceptually:

```text
Previous Build
      │
      ▼
Reusable Outputs
      │
      ▼
Next Build
      │
      ▼
Less Work
```

Gradle supports build caching mechanisms that can reduce repeated work.

In large organizations, caching can have a major effect on developer productivity and CI cost.

---

# 39. Clean Build vs Incremental Build

These are very different operations.

### Incremental Build

```text
Change
  ↓
Rebuild affected work
```

### Clean Build

```text
Delete previous outputs
        ↓
Rebuild everything needed
```

When debugging:

```bash
./gradlew clean
```

can sometimes help identify stale build artifacts.

But using clean builds for every change is usually expensive.

A healthy KMP workflow should rely on incremental builds most of the time.

---

# 40. Why "Clean and Rebuild" Is Not a Strategy

A common developer habit is:

> "It failed. Clean and rebuild."

Sometimes it works.

But it doesn't explain the problem.

A better approach is:

```text
Build Failure
     │
     ▼
Identify Stage
     │
 ┌───┼───────────────┐
 ▼   ▼               ▼
Resolve Compile     Link
Deps
```

Then investigate the appropriate layer.

---

# 41. Debugging the Build Pipeline

When a build fails, ask:

### 1. Did Gradle configure successfully?

```text
Configuration failure
```

### 2. Did dependencies resolve?

```text
Dependency failure
```

### 3. Did Kotlin compile?

```text
Compilation failure
```

### 4. Did native linking succeed?

```text
Linker failure
```

### 5. Did Xcode build?

```text
Apple build failure
```

### 6. Did packaging/signing succeed?

```text
Packaging failure
```

This classification dramatically reduces debugging time.

---

# 42. A Build Failure Map

```text
                    Build Failure
                         │
          ┌──────────────┼──────────────┐
          ▼              ▼              ▼
     Configuration   Compilation     Packaging
          │              │              │
          ▼              ▼              ▼
        Gradle         Kotlin         Android/
                                      Xcode
                         │
                    ┌────┴────┐
                    ▼         ▼
                   JVM      Native
```

Instead of asking:

> "Why doesn't KMP build?"

ask:

> **"Which stage of the pipeline failed?"**

---

# 43. Build Pipeline and Architecture

The build pipeline also reflects your architecture.

For example:

```text
commonMain
    │
    ▼
Shared Business Logic
    │
    ├── Android implementation
    │
    └── iOS implementation
```

becomes:

```text
Shared Source
    │
    ├── Android Compilation
    │
    └── iOS Compilation
```

So your source-set architecture directly influences the build graph.

Poor boundaries often create:

```text
Complex dependencies
+
More platform-specific code
+
More complicated builds
```

Good boundaries tend to create:

```text
Clear dependencies
+
Predictable targets
+
Simpler builds
```

---

# 44. The Build Pipeline Is an Architectural Feedback Loop

This is an important insight.

Your architecture affects the build.

The build exposes architectural problems.

For example:

```text
Too much Android code in commonMain
              │
              ▼
Dependency / compilation problem
              │
              ▼
Architectural boundary is wrong
```

The compiler and build system therefore become useful feedback mechanisms.

---

# 45. A Practical Example

Imagine this:

```kotlin
commonMain {
    implementation("android.some:library")
}
```

The developer's intention may be:

> "I only need this one feature."

But the build sees:

```text
commonMain
    │
    ▼
Android-only dependency
    │
    ▼
iOS compilation
    │
    ▼
❌ Unsupported dependency
```

The architecture should instead look like:

```text
commonMain
    │
    ▼
Abstraction
    │
    ├── androidMain
    │      └── Android Library
    │
    └── iosMain
           └── Apple Framework
```

---

# 46. Build Pipeline and Dependency Injection

Dependency injection can help preserve those boundaries.

For example:

```text
                 Common Service
                      │
                      ▼
               SecureStorage
                      │
            ┌─────────┴─────────┐
            ▼                   ▼
       AndroidStorage       IOSStorage
            │                   │
            ▼                   ▼
       Android API          Apple API
```

The build pipeline sees:

```text
commonMain
+
androidMain
+
iosMain
```

instead of forcing a platform-specific dependency into common code.

---

# 47. Build Pipeline and Testing Strategy

The same boundaries influence tests.

```text
                 Shared Logic
                      │
                      ▼
                 commonTest
                      │
                      ▼
                 Shared Tests
```

Platform behavior:

```text
Android
   │
   ▼
Android Tests
```

and:

```text
iOS
   │
   ▼
iOS Tests
```

The build system can then execute the appropriate tests for each target.

---

# 48. The Build Pipeline Is More Than "Compile"

A production KMP pipeline may contain:

```text
Checkout
   ↓
Dependency Resolution
   ↓
Code Generation
   ↓
Compilation
   ↓
Unit Tests
   ↓
Static Analysis
   ↓
Native Linking
   ↓
Packaging
   ↓
Signing
   ↓
Artifact Publishing
```

The exact pipeline depends on the organization.

But the important lesson is:

> **Compilation is only one stage of the build pipeline.**

---

# 49. Library Publishing

The build pipeline becomes even more interesting when the shared module is published as a library.

Instead of:

```text
Shared Module
    ↓
Application
```

you may have:

```text
Shared Module
    ↓
Build Artifacts
    ↓
Repository
    ↓
Other Applications
```

Conceptually:

```text
                 KMP Library
                     │
              ┌──────┴──────┐
              ▼             ▼
          Android         iOS
          Artifact       Artifact
              │             │
              └──────┬──────┘
                     ▼
                Consumers
```

This is how organizations can create reusable multiplatform libraries.

---

# 50. The Complete KMP Build Mental Model

At this point, the complete picture looks like:

```text
                           SOURCE
                             │
                             ▼
                      KMP Source Sets
                             │
                  ┌──────────┴──────────┐
                  ▼                     ▼
             Android Target        iOS Target
                  │                     │
                  ▼                     ▼
            Dependency             Dependency
             Resolution             Resolution
                  │                     │
                  ▼                     ▼
             Kotlin/JVM            Kotlin/Native
                  │                     │
                  ▼                     ▼
              Compile                Compile
                  │                     │
                  ▼                     ▼
                Test                  Test
                  │                     │
                  ▼                     ▼
              Package               Link /
                                    Framework
                  │                     │
                  ▼                     ▼
              APK / AAB              Xcode
                                        │
                                        ▼
                                     iOS App
```

This is the build pipeline you should visualize whenever you work on a KMP project.

---

# 51. The Five Layers of the KMP Build

A useful way to simplify everything is to think in five layers.

## Layer 1 — Source

```text
commonMain
androidMain
iosMain
```

## Layer 2 — Configuration

```text
Gradle
KMP Plugin
Targets
Dependencies
```

## Layer 3 — Compilation

```text
Kotlin/JVM
Kotlin/Native
```

## Layer 4 — Packaging

```text
APK
AAB
Framework
Native Output
```

## Layer 5 — Platform Build

```text
Android Toolchain
Xcode
```

Together:

```text
Source
  ↓
Configuration
  ↓
Compilation
  ↓
Packaging
  ↓
Platform Build
```

---

# 52. What Changes When You Add Another Platform?

Suppose today you have:

```text
Android
iOS
```

Tomorrow you add:

```text
Desktop
```

The pipeline expands:

```text
                    commonMain
                        │
          ┌─────────────┼─────────────┐
          ▼             ▼             ▼
       Android          iOS         Desktop
          │             │             │
          ▼             ▼             ▼
      JVM/Android    Native        JVM/Desktop
          │             │             │
          ▼             ▼             ▼
       Android        iOS         Desktop App
```

This is one of the strengths of KMP.

But it also means:

> **Every additional target adds build, testing, dependency, and maintenance responsibilities.**

Multiplatform doesn't mean unlimited platforms come for free.

---

# 53. Build Complexity Grows With Targets

A simple application:

```text
Android + iOS
```

has a manageable build graph.

Add:

```text
Desktop
Web
Wasm
```

and you introduce more:

```text
Targets
Dependencies
Tests
Artifacts
CI Jobs
Toolchains
```

Conceptually:

```text
More Targets
     │
     ├── More Compilation
     ├── More Testing
     ├── More Dependencies
     ├── More Artifacts
     └── More CI Complexity
```

This is why target selection should always be intentional.

---

# 54. The Build Pipeline Should Reflect the Product

If your application only needs:

```text
Android + iOS
```

there may be little reason to add:

```text
Desktop
Web
Wasm
```

simply because KMP supports them.

Every target has a cost.

A good architecture asks:

```text
Do we need this target?
        │
        ▼
Does the product justify it?
        │
        ▼
Can the team support it?
```

The build pipeline should serve the product—not the other way around.

---

# 55. A Useful Debugging Checklist

When a KMP build fails, walk through this order:

```text
┌───────────────────────────────┐
│ 1. Gradle configuration       │
└───────────────┬───────────────┘
                ▼
┌───────────────────────────────┐
│ 2. Dependency resolution      │
└───────────────┬───────────────┘
                ▼
┌───────────────────────────────┐
│ 3. Source-set compatibility   │
└───────────────┬───────────────┘
                ▼
┌───────────────────────────────┐
│ 4. Kotlin compilation         │
└───────────────┬───────────────┘
                ▼
┌───────────────────────────────┐
│ 5. Native linking             │
└───────────────┬───────────────┘
                ▼
┌───────────────────────────────┐
│ 6. Xcode / Android build      │
└───────────────┬───────────────┘
                ▼
┌───────────────────────────────┐
│ 7. Packaging / signing        │
└───────────────────────────────┘
```

This simple checklist can save hours of random troubleshooting.

---

# 56. The Most Important Build Pipeline Principle

> [!IMPORTANT]
> **Do not treat the KMP build as one compiler command. Treat it as a coordinated pipeline of target-specific tasks.**

The simplified relationship is:

```text
                     Gradle
                       │
                ┌──────┴──────┐
                ▼             ▼
             Android          iOS
                │             │
                ▼             ▼
           Kotlin/JVM     Kotlin/Native
                │             │
                ▼             ▼
            Android        Framework /
             Output        Native Output
                │             │
                ▼             ▼
              APK            Xcode
                              │
                              ▼
                           iOS App
```

---

# 57. Chapter Takeaways

> [!TIP]
> **A KMP build is a coordinated graph that turns shared Kotlin source into target-specific artifacts.**

Remember:

1. Gradle orchestrates the KMP build.
2. The build is a graph, not a single compilation step.
3. Source sets determine which code participates in each target.
4. Dependencies are resolved according to target compatibility.
5. Android commonly follows a Kotlin/JVM-based path.
6. iOS uses Kotlin/Native.
7. Compilation and packaging are separate stages.
8. Native targets involve linking as well as compilation.
9. iOS integration involves Xcode.
10. Android integration involves the Android build toolchain.
11. Shared modules are not necessarily complete applications.
12. Android and iOS applications can consume the same shared module.
13. Incremental compilation reduces unnecessary work.
14. Build caching can improve large project and CI performance.
15. Clean builds are useful for diagnosis but should not replace understanding the failure.
16. A build failure should first be classified by pipeline stage.
17. Additional targets increase build and maintenance complexity.
18. Build configuration should remain separate from business logic.
19. Build boundaries often expose architectural problems.
20. The build pipeline should support the product architecture rather than dictate it.

---

# Final Mental Model

When you run:

```bash
./gradlew build
```

don't imagine:

```text
"Gradle compiles my KMP project."
```

Think:

```text
                         Gradle
                           │
                           ▼
                    Build Configuration
                           │
                           ▼
                     Target Graph
                           │
             ┌─────────────┴─────────────┐
             ▼                           ▼
          Android                        iOS
             │                           │
             ▼                           ▼
       Source Sets                  Source Sets
             │                           │
             ▼                           ▼
        Kotlin/JVM                 Kotlin/Native
             │                           │
             ▼                           ▼
          Compile                     Compile
             │                           │
             ▼                           ▼
           Test                    Link / Framework
             │                           │
             ▼                           ▼
        APK / AAB                     Xcode
                                         │
                                         ▼
                                      iOS App
```

The key relationship is:

```text
Source
   ↓
Source Sets
   ↓
Target Configuration
   ↓
Dependency Resolution
   ↓
Compilation
   ↓
Testing
   ↓
Linking / Packaging
   ↓
Platform Application
```

Once you understand this pipeline, build errors stop looking like random KMP failures.

You can start asking the right question:

> **Which stage of the pipeline is responsible for this failure?**

And that is the difference between simply using KMP and actually understanding how a KMP project is built.
