# Chapter 5 — Creating Your First KMP Project

## Part 2 — Generated Files

> **A KMP project becomes much easier to understand once you stop treating generated files as boilerplate and start reading them as a map of the build system.**

After creating the project with the wizard, the next step is not to immediately write application code.

Open the project.

Look at the files.

At first, the structure can feel familiar to an Android developer:

```text
build.gradle.kts
settings.gradle.kts
gradle/
src/
```

But a KMP project introduces another dimension:

```text
Android
+
iOS
+
Shared Kotlin
```

The generated files are what connect those worlds.

This part walks through the important files and explains what each one is responsible for.

---

# 1. Start With the Project Tree

A newly generated KMP project can look conceptually similar to:

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
├── gradle.properties
└── gradlew
```

The exact structure depends on the current Kotlin Multiplatform and Android tooling.

You may see additional files such as:

```text
gradlew.bat
local.properties
gradle/libs.versions.toml
```

or other generated configuration.

Don't worry about every file yet.

Start with the ones that explain the architecture.

---

# 2. The Five Files to Understand First

For a first KMP project, begin with:

```text
1. settings.gradle.kts
2. build.gradle.kts
3. shared/build.gradle.kts
4. androidApp/build.gradle.kts
5. gradle/libs.versions.toml
```

Then look at:

```text
gradle.properties
```

and:

```text
gradle/
```

These files form much of the project's build foundation.

---

# 3. `settings.gradle.kts`

If Gradle is the coordinator of the build, `settings.gradle.kts` is one of the files that tells Gradle what the project contains.

Conceptually:

```text
settings.gradle.kts
        │
        ├── Project identity
        ├── Modules
        └── Plugin / dependency configuration
```

A simplified example may look like:

```kotlin
rootProject.name = "KmpFirstProject"

include(":androidApp")
include(":shared")
```

The exact generated configuration can differ.

The important idea is:

> **Gradle needs to know which projects participate in the build.**

---

# 4. Project vs Module

Android developers often use the words:

```text
Project
Module
```

interchangeably.

They are not the same.

Think of:

```text
Root Project
     │
     ├── androidApp
     └── shared
```

The root project coordinates the overall build.

The modules contain specific application or library responsibilities.

A KMP project commonly has:

```text
Root
│
├── Android Application
├── Shared Multiplatform Module
└── iOS Application Integration
```

---

# 5. Why `settings.gradle.kts` Matters

Suppose you create a new module:

```text
shared-feature
```

but Gradle does not know that it belongs to the build.

Then the source files may exist on disk, but the build system won't treat them as part of the project in the expected way.

Conceptually:

```text
Folder Exists
     │
     ▼
Gradle Knows About It?
     │
 ┌───┴───┐
 ▼       ▼
Yes      No
 │        │
 ▼        ▼
Build    Ignored
```

This is why `settings.gradle.kts` matters.

---

# 6. `pluginManagement`

Modern Gradle projects often use `pluginManagement` in `settings.gradle.kts`.

Conceptually:

```kotlin
pluginManagement {
    repositories {
        google()
        mavenCentral()
        gradlePluginPortal()
    }
}
```

This tells Gradle where it can resolve build plugins.

The exact repositories depend on the plugins used by the project.

---

# 7. `dependencyResolutionManagement`

You may also see configuration related to dependency resolution.

Conceptually:

```kotlin
dependencyResolutionManagement {
    repositories {
        google()
        mavenCentral()
    }
}
```

This defines where dependencies can be resolved from.

For example:

```text
KMP Library
     │
     ▼
Gradle Dependency Resolution
     │
     ├── Google
     ├── Maven Central
     └── Other Configured Repository
```

Use trusted repositories and understand the provenance of dependencies before adding them to production projects.

---

# 8. `build.gradle.kts` at the Root

The root `build.gradle.kts` is different from:

```text
shared/build.gradle.kts
```

and:

```text
androidApp/build.gradle.kts
```

The root file generally contains configuration that applies to the overall project or makes plugins available to modules.

Conceptually:

```text
Root build.gradle.kts
          │
    ┌─────┴─────┐
    ▼           ▼
androidApp    shared
```

The module-level files then configure the individual modules.

---

# 9. Root Build File vs Module Build File

Think about the relationship like this:

```text
                 Root Build
                     │
          ┌──────────┴──────────┐
          ▼                     ▼
   Android Module          Shared Module
     Build File              Build File
```

The root file coordinates.

The module files describe module-specific behavior.

This distinction becomes increasingly important as a KMP project grows.

---

# 10. `shared/build.gradle.kts`

This is one of the most important files in the project.

It describes the shared KMP module.

Conceptually:

```text
shared/build.gradle.kts
          │
          ├── Kotlin Multiplatform Plugin
          ├── Targets
          ├── Source Sets
          ├── Dependencies
          └── Framework / Native Configuration
```

This is where the KMP build model becomes explicit.

---

# 11. The Kotlin Multiplatform Plugin

The shared module needs the Kotlin Multiplatform Gradle plugin.

Conceptually:

```kotlin
plugins {
    kotlin("multiplatform")
}
```

The exact generated plugin declaration may use version catalogs or plugin aliases.

For example:

```kotlin
plugins {
    alias(libs.plugins.kotlin.multiplatform)
}
```

The syntax may differ.

The purpose remains the same:

```text
Kotlin Multiplatform Plugin
          │
          ▼
Enables KMP Build Configuration
```

---

# 12. Declaring Targets

Inside the shared module, target declarations tell Kotlin where the code should be compiled.

A simplified example:

```kotlin
kotlin {
    androidTarget()

    iosArm64()
    iosSimulatorArm64()
}
```

Conceptually:

```text
                     shared
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
       Android       iOS Device   iOS Simulator
       Target          Target         Target
```

The exact target list depends on the platforms your project supports.

---

# 13. Why iOS Has Multiple Targets

Android developers sometimes wonder:

> "Why does iOS need more than one target?"

Because:

```text
Physical Device
```

and:

```text
Simulator
```

can use different architectures and binaries.

Conceptually:

```text
                 iOS
                  │
        ┌─────────┴─────────┐
        ▼                   ▼
   Physical Device       Simulator
        │                   │
        ▼                   ▼
     Native              Native
     Target              Target
```

The project needs appropriate outputs for the environment being built.

---

# 14. Source Sets

The shared Gradle configuration also describes source sets.

Conceptually:

```text
kotlin {
    sourceSets {
        commonMain.dependencies {
            // shared dependencies
        }

        commonTest.dependencies {
            // shared test dependencies
        }
    }
}
```

The exact syntax can change as Kotlin tooling evolves.

The architectural idea is stable:

```text
Source Set
     │
     ▼
Code + Dependencies
```

---

# 15. `commonMain`

The `commonMain` source set is where platform-independent production code belongs.

Typical examples:

```text
Domain Models
Use Cases
Business Rules
Repository Contracts
Networking
Serialization
Shared State
```

Conceptually:

```text
commonMain
    │
    ├── domain
    ├── data
    ├── networking
    └── presentation
```

The code can then participate in multiple target compilations.

---

# 16. `androidMain`

Android-specific dependencies and implementation belong here.

For example:

```text
androidMain
    │
    ├── Android APIs
    ├── Android Storage
    └── Android-specific Libraries
```

Conceptually:

```text
commonMain
     ▲
     │
androidMain
```

Android-specific code can use shared code.

---

# 17. `iosMain`

Similarly:

```text
iosMain
    │
    ├── Apple APIs
    ├── iOS Services
    └── iOS-specific Libraries
```

Conceptually:

```text
commonMain
     ▲
     │
iosMain
```

This source set participates in the native iOS compilation.

---

# 18. `commonTest`

Shared tests are configured through:

```text
commonTest
```

For example:

```text
commonTest
    │
    ├── Domain Tests
    ├── Use Case Tests
    └── Repository Tests
```

This lets you verify shared behavior once where platform independence is appropriate.

---

# 19. Dependency Declarations

One of the most important things to understand is that dependencies can be target-aware.

For example:

```text
commonMain
     │
     ▼
Multiplatform Library
```

The library must support the targets where the shared source is compiled.

Conceptually:

```text
Shared Dependency
       │
       ├── Android Artifact
       │
       └── iOS Artifact
```

This is why blindly adding any Kotlin or Android dependency to `commonMain` doesn't work.

---

# 20. Android-Only Dependency

Suppose you have:

```text
Android-only library
```

It should not normally be placed in:

```text
commonMain
```

because iOS cannot compile against an Android-only API.

Instead:

```text
androidMain
     │
     ▼
Android-only Library
```

while:

```text
commonMain
```

depends only on APIs that are available to all required targets.

---

# 21. The Dependency Graph

The result can be visualized as:

```text
                         Dependencies
                              │
                ┌─────────────┴─────────────┐
                ▼                           ▼
             commonMain                androidMain
                │                           │
                ▼                           ▼
       Multiplatform APIs            Android APIs
                │
                ▼
            iOS Target
```

The important rule is:

> **A shared source set must only depend on libraries compatible with the targets that consume it.**

---

# 22. `androidApp/build.gradle.kts`

The Android application module has its own Gradle configuration.

It may contain:

```text
Android application plugin
Android SDK configuration
Application ID
Build types
Dependencies
Compose configuration
Signing configuration
```

Conceptually:

```text
androidApp/build.gradle.kts
          │
          ├── Android Plugin
          ├── SDK
          ├── Application ID
          ├── Build Types
          └── shared Dependency
```

---

# 23. The Android App Depends on Shared

The relationship is usually:

```text
androidApp
     │
     ▼
shared
```

Conceptually:

```kotlin
dependencies {
    implementation(project(":shared"))
}
```

The exact configuration depends on the generated template.

The architectural meaning is:

```text
Android Application
       │
       ▼
Shared KMP Module
```

---

# 24. Android Resources

The Android application can continue to own Android-specific resources:

```text
strings.xml
drawables
fonts
Android themes
manifest
```

KMP does not require these to move into the shared module.

For native UI architecture:

```text
Android Resources → Android
```

is perfectly reasonable.

---

# 25. `iosApp`

The iOS application is slightly different from the Android module because Apple's application project is normally managed through Xcode.

You may see:

```text
iosApp/
```

containing an Xcode project or workspace structure.

Conceptually:

```text
iosApp
   │
   ├── Swift / SwiftUI
   ├── Xcode Project
   └── Apple Resources
```

The shared Kotlin output is then integrated into that application.

---

# 26. The iOS Integration Boundary

Think:

```text
KMP Shared Module
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

This boundary is fundamental.

The iOS application remains a native Apple application.

---

# 27. `gradle/libs.versions.toml`

Many modern Gradle projects use a version catalog:

```text
gradle/
└── libs.versions.toml
```

This file can centralize dependency and plugin versions.

For example:

```toml
[versions]
kotlin = "..."
agp = "..."

[libraries]
some-library = { ... }

[plugins]
kotlinMultiplatform = { ... }
```

The exact entries depend on the project.

---

# 28. Why Version Catalogs Matter

Without a version catalog, versions can become scattered:

```text
Module A → Kotlin version
Module B → Kotlin version
Module C → Library version
```

With a catalog:

```text
                libs.versions.toml
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
      Module A      Module B      Module C
```

This makes dependency management easier to centralize.

---

# 29. Version Catalog Is Not Dependency Resolution

These concepts are related but different.

A version catalog says:

```text
"What version/name should we use?"
```

Repository configuration says:

```text
"Where can Gradle find it?"
```

Dependency declaration says:

```text
"Which module needs it?"
```

So:

```text
Version Catalog
       +
Repository
       +
Dependency Declaration
       =
Dependency Resolution
```

---

# 30. `gradle.properties`

You may also find:

```text
gradle.properties
```

This file can contain project and Gradle properties.

For example:

```text
org.gradle.jvmargs=...
```

It may also contain Android or Kotlin-related configuration depending on the project.

Treat this file as build configuration, not application business logic.

---

# 31. `gradle/` Directory

The `gradle` directory commonly contains Gradle wrapper-related files and version catalog configuration.

For example:

```text
gradle/
├── libs.versions.toml
└── wrapper/
    ├── gradle-wrapper.jar
    └── gradle-wrapper.properties
```

The exact contents can vary.

The important point is:

```text
gradle/
     │
     ├── Build tooling support
     └── Dependency/version configuration
```

---

# 32. Gradle Wrapper

A generated project commonly includes:

```text
gradlew
gradlew.bat
```

These scripts allow the project to invoke the Gradle version defined by the wrapper configuration.

Conceptually:

```text
Developer
    │
    ▼
./gradlew build
    │
    ▼
Configured Gradle Version
    │
    ▼
Project Build
```

This helps teams avoid relying on an arbitrary globally installed Gradle version.

---

# 33. Why the Wrapper Matters

Imagine:

```text
Developer A → Gradle 8.x
Developer B → Gradle 9.x
CI            → another version
```

That can create inconsistent builds.

With the wrapper:

```text
Project
  │
  ▼
Wrapper Configuration
  │
  ▼
Expected Gradle Version
```

The project controls the Gradle version it expects.

---

# 34. `local.properties`

Android projects may also contain:

```text
local.properties
```

This is generally machine-specific configuration.

For example, it can point tooling toward the local Android SDK.

Because it can contain local environment information, it should not normally be treated as portable project configuration.

It is commonly excluded from source control.

---

# 35. Generated Files vs Source Files

At this point, separate the project into two categories.

### Source

```text
commonMain
androidMain
iosMain
commonTest
Android UI
iOS UI
```

### Build Configuration

```text
settings.gradle.kts
build.gradle.kts
gradle.properties
libs.versions.toml
wrapper files
```

The distinction is:

```text
Source
  ↓
What the application does

Build Configuration
  ↓
How the application is built
```

---

# 36. Don't Put Business Logic in Gradle

It may sound obvious, but large projects sometimes accumulate logic in build scripts.

Avoid turning Gradle into an application configuration language.

Business logic belongs in application code.

Gradle should primarily describe:

```text
Build
Targets
Dependencies
Packaging
Verification
```

---

# 37. The Build File as a Map

Instead of reading:

```text
shared/build.gradle.kts
```

as random configuration, read it as a map.

Ask:

```text
Which plugin is applied?
Which targets exist?
Which source sets exist?
Which dependencies are shared?
Which dependencies are platform-specific?
How is the iOS output configured?
```

This makes the file much easier to understand.

---

# 38. Read `shared/build.gradle.kts` Top to Bottom

A useful reading order is:

```text
1. Plugins
      ↓
2. Kotlin Targets
      ↓
3. Source Sets
      ↓
4. Dependencies
      ↓
5. Native / Framework Configuration
```

Conceptually:

```text
shared/build.gradle.kts
          │
          ▼
        Plugins
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
       Outputs
```

This mirrors the build pipeline from the previous chapter.

---

# 39. Connecting the Files

Now connect everything.

```text
                 settings.gradle.kts
                         │
                         ▼
                    Root Project
                         │
              ┌──────────┴──────────┐
              ▼                     ▼
        androidApp                shared
              │                     │
              │             ┌───────┼───────┐
              │             ▼       ▼       ▼
              │         commonMain android  ios
              │                     Main    Main
              │
              ▼
          Android App
```

Then:

```text
shared
   │
   ▼
Kotlin Multiplatform
   │
   ├── Android
   └── iOS
```

This is the project graph.

---

# 40. From File to Build

Every important file contributes to the build.

```text
settings.gradle.kts
        │
        ▼
Project Structure
        │
        ▼
build.gradle.kts
        │
        ▼
Plugins / Targets
        │
        ▼
Source Sets
        │
        ▼
Dependencies
        │
        ▼
Compiler
        │
        ▼
Artifacts
```

This is why understanding generated files is important.

They are not random boilerplate.

---

# 41. A Simple Dependency Example

Imagine:

```text
commonMain
    │
    ▼
NetworkClient
    │
    ▼
Multiplatform HTTP Library
```

and:

```text
androidMain
    │
    ▼
Android-specific implementation
```

while:

```text
iosMain
    │
    ▼
iOS-specific implementation
```

The Gradle configuration describes these relationships.

---

# 42. Source Set Dependency Diagram

Think about source sets like this:

```text
                         commonMain
                             │
               ┌─────────────┴─────────────┐
               ▼                           ▼
          androidMain                   iosMain
               │                           │
               ▼                           ▼
          Android APIs                 Apple APIs
```

The common source set forms the shared foundation.

The platform source sets extend it with target-specific behavior.

---

# 43. `commonMain` Is Not a Dumping Ground

Once developers discover that code can be shared, there is a temptation to put everything there.

Avoid:

```text
commonMain/
├── AndroidCode.kt
├── IOSCode.kt
├── BusinessLogic.kt
├── UIHelper.kt
└── Everything.kt
```

Instead:

```text
commonMain/
├── domain/
├── data/
├── networking/
└── presentation/
```

and:

```text
androidMain/
└── platform/
```

```text
iosMain/
└── platform/
```

Clear structure makes runtime and build behavior easier to reason about.

---

# 44. Platform Dependencies Belong at the Edge

For example:

```text
Android-only database API
```

should not become a dependency of:

```text
commonMain
```

unless the technology explicitly provides a supported multiplatform abstraction.

Instead:

```text
commonMain
      │
      ▼
Database Interface
      ▲
      │
 ┌────┴────┐
 ▼         ▼
Android    iOS
```

This preserves portability.

---

# 45. Generated Files and Source Control

Not every generated file should necessarily be committed.

A practical repository often contains:

```text
Source Code
Gradle Configuration
Wrapper
Version Catalog
Project Files
```

while excludes:

```text
Build Outputs
Local Machine Configuration
IDE Cache
Temporary Files
```

Use the project's generated `.gitignore` as a starting point and review it before publishing the repository.

---

# 46. What You Should Commit

For a typical project, important build configuration such as:

```text
build.gradle.kts
settings.gradle.kts
gradle.properties
gradle/libs.versions.toml
gradle wrapper configuration
```

is part of the project definition.

The source code and build configuration together allow another developer or CI system to reproduce the build.

---

# 47. What You Should Not Blindly Commit

Avoid committing generated or machine-specific output such as:

```text
build/
.idea/
local.properties
```

unless there is a specific reason and the project conventions require it.

The exact `.gitignore` should reflect the tools and project setup.

---

# 48. Generated Files and Reproducibility

A good project should be reproducible.

Conceptually:

```text
Git Repository
      │
      ├── Source
      ├── Gradle Configuration
      ├── Wrapper
      └── Version Definitions
             │
             ▼
          Clone
             │
             ▼
          Build
```

The goal is that a clean checkout can reconstruct the project without relying on one developer's local machine.

---

# 49. The First Inspection Exercise

After creating the project, open these files in order:

```text
1. settings.gradle.kts
2. build.gradle.kts
3. shared/build.gradle.kts
4. androidApp/build.gradle.kts
5. gradle/libs.versions.toml
6. gradle.properties
```

For each file, write down:

```text
What does this file configure?
Who consumes this configuration?
Which target does it affect?
```

This simple exercise will teach you more than memorizing Gradle syntax.

---

# 50. The File Responsibility Map

Use this as a quick reference:

| File / Directory | Primary Responsibility |
|---|---|
| `settings.gradle.kts` | Project/module structure and Gradle-level configuration |
| Root `build.gradle.kts` | Root build/plugin configuration |
| `shared/build.gradle.kts` | KMP targets, source sets, shared dependencies |
| `androidApp/build.gradle.kts` | Android application configuration |
| `gradle/libs.versions.toml` | Centralized dependency/plugin versions |
| `gradle.properties` | Gradle/project properties |
| `gradle/wrapper/` | Gradle wrapper configuration |
| `commonMain/` | Shared production Kotlin |
| `commonTest/` | Shared tests |
| `androidMain/` | Android-specific Kotlin |
| `iosMain/` | iOS-specific Kotlin |
| `iosApp/` | Native iOS application integration |

The exact generated structure may change with tooling versions, but these responsibilities remain useful.

---

# 51. The Complete Generated Project Map

```text
KmpFirstProject/
│
├── androidApp/
│   │
│   ├── src/
│   │
│   └── build.gradle.kts
│
├── shared/
│   │
│   ├── src/
│   │   ├── commonMain/
│   │   ├── commonTest/
│   │   ├── androidMain/
│   │   └── iosMain/
│   │
│   └── build.gradle.kts
│
├── iosApp/
│   │
│   └── Xcode Project / Swift Sources
│
├── gradle/
│   ├── libs.versions.toml
│   └── wrapper/
│
├── build.gradle.kts
├── settings.gradle.kts
├── gradle.properties
├── gradlew
└── gradlew.bat
```

Now the project should start to look less like a collection of files and more like a system.

---

# 52. The Relationship Between Configuration and Code

The project can be viewed as two connected layers.

### Build Layer

```text
Gradle
 │
 ├── Plugins
 ├── Targets
 ├── Source Sets
 └── Dependencies
```

### Code Layer

```text
Kotlin
 │
 ├── commonMain
 ├── androidMain
 ├── iosMain
 └── commonTest
```

Together:

```text
             Build Configuration
                     │
                     ▼
                Source Sets
                     │
                     ▼
                   Code
                     │
                     ▼
                  Targets
                     │
                     ▼
                 Artifacts
```

---

# 53. Why This Matters for Debugging

Suppose a class works in:

```text
Android
```

but fails in:

```text
iOS
```

Don't immediately blame the Kotlin code.

Check:

```text
Which source set contains it?
Which dependency does it use?
Does that dependency support iOS?
Which target is compiling it?
Which platform implementation is selected?
```

The generated project structure gives you the clues.

---

# 54. A Practical Debugging Example

Imagine:

```text
commonMain
    │
    ▼
SomeLibrary
```

Android builds successfully.

iOS fails.

Your first question should be:

```text
Does SomeLibrary support iOS?
```

If the answer is no, the issue is not necessarily your business logic.

It is a dependency/source-set boundary problem.

The correct architecture might be:

```text
commonMain
      │
      ▼
Abstraction
      ▲
      │
 ┌────┴────┐
 ▼         ▼
Android    iOS
```

---

# 55. Generated Files as Documentation

One of the best ways to learn KMP is to treat the generated Gradle files as documentation.

Instead of copying configuration from tutorials without understanding it, ask:

```text
Why did the wizard create this?
What problem does this line solve?
Which target consumes it?
What happens if I remove it?
```

This turns project configuration into something you can reason about.

---

# 56. Don't Memorize Every Gradle Line

You don't need to memorize:

```kotlin
every plugin alias
every repository declaration
every compiler option
every generated task
```

You need to understand:

```text
Plugin
   ↓
Target
   ↓
Source Set
   ↓
Dependency
   ↓
Output
```

Once that mental model is strong, individual Gradle syntax becomes much easier to learn.

---

# 57. The Most Important File

If you are an Android developer starting KMP, spend extra time with:

```text
shared/build.gradle.kts
```

because it connects:

```text
Kotlin Multiplatform
        │
        ├── Targets
        ├── Source Sets
        ├── Dependencies
        └── Native Outputs
```

It is the closest thing to a map of the shared module's build behavior.

---

# 58. The Most Important Folder

For application logic:

```text
shared/src/commonMain/
```

is the folder to understand deeply.

This is where the question:

> **"What can actually be shared?"**

becomes code.

---

# 59. The Most Important Boundary

The most important boundary is:

```text
commonMain
     │
     ├────────── Android-specific
     │
     └────────── iOS-specific
```

Do not think of platform source sets as places where you put "leftover code."

Think of them as deliberate extension points.

---

# 60. From Wizard to Architecture

The progression should now be clear:

```text
Wizard
  │
  ▼
Generated Files
  │
  ▼
Understand Modules
  │
  ▼
Understand Source Sets
  │
  ▼
Understand Targets
  │
  ▼
Understand Dependencies
  │
  ▼
Start Writing Shared Code
```

This is a much better learning path than immediately copying a large sample project.

---

# Chapter Takeaways

> [!TIP]
> **Generated KMP files are not meaningless boilerplate. They describe how the project is organized, compiled, and connected to Android and iOS.**

Remember:

1. `settings.gradle.kts` helps define the Gradle project structure.
2. The root `build.gradle.kts` provides project-level build configuration.
3. `shared/build.gradle.kts` defines the KMP module.
4. KMP targets determine where shared code is compiled.
5. Source sets determine which source participates in each compilation.
6. `commonMain` contains platform-independent production code.
7. `androidMain` contains Android-specific implementation.
8. `iosMain` contains iOS-specific implementation.
9. `commonTest` contains shared tests.
10. `androidApp` is the Android application module.
11. `iosApp` represents the native iOS application integration.
12. `libs.versions.toml` can centralize dependency and plugin versions.
13. `gradle.properties` contains build/project properties.
14. The Gradle wrapper helps keep the build tooling version consistent.
15. `local.properties` is typically machine-specific and should not be treated as portable configuration.
16. Shared dependencies must support the targets that consume them.
17. Android-only dependencies should remain on the Android side of the boundary.
18. The generated project is a starting structure, not your final production architecture.
19. Read Gradle files as a map of plugins, targets, source sets, dependencies, and outputs.
20. Understanding the generated project makes later KMP debugging much easier.

---

# Final Mental Model

When you open a generated KMP project, don't see:

```text
A lot of Gradle files
+
A lot of folders
+
Some generated code
```

See:

```text
                         KMP PROJECT
                              │
                   ┌──────────┴──────────┐
                   ▼                     ▼
               Build Layer           Code Layer
                   │                     │
                   ▼                     ▼
                Gradle              Source Sets
                   │                     │
          ┌────────┼────────┐      ┌─────┼─────┐
          ▼        ▼        ▼      ▼     ▼     ▼
       Plugins   Targets  Deps   common Android iOS
                                   Main   Main  Main
                                     │      │     │
                                     └──────┼─────┘
                                            ▼
                                      Target Outputs
                                            │
                                  ┌─────────┴─────────┐
                                  ▼                   ▼
                              Android               iOS
                              Application         Application
```

The key idea is simple:

> **The generated files are the blueprint of the KMP build. Learn to read that blueprint, and the project stops feeling like magic.**
