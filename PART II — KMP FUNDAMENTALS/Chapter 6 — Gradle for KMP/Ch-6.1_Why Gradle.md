# Chapter 6 --- Gradle for KMP

## Part 1 --- Why Gradle

> **If Kotlin Multiplatform is the architecture, Gradle is the machinery
> that turns that architecture into a buildable project.**

When an Android developer starts with Kotlin Multiplatform, the first
unfamiliar thing is often not Kotlin. It is the build configuration.

A KMP project quickly introduces files such as:

``` text
build.gradle.kts
settings.gradle.kts
gradle.properties
gradle/
gradle/libs.versions.toml
```

And inside the shared module you may see:

``` kotlin
kotlin {
    androidTarget()
    iosArm64()
    iosSimulatorArm64()
}
```

At first this can look like configuration noise.

It isn't.

These declarations describe how the project is assembled, which
platforms it targets, which dependencies are available, how source sets
relate to targets, and how shared Kotlin becomes platform-specific
output.

That makes Gradle much more important in KMP than simply being:

> "The thing that builds my project."

------------------------------------------------------------------------

# 1. Why Do We Need Gradle?

Every software project eventually needs to answer questions such as:

``` text
What are my modules?
What platforms do I support?
Which code belongs to each platform?
Which libraries do I depend on?
How should the code be compiled?
How should tests run?
How should artifacts be produced?
```

A build system gives those decisions a repeatable form.

For KMP, the problem becomes more interesting because one project can
target multiple platforms:

``` text
                 KMP Project
                      │
          ┌───────────┴───────────┐
          ▼                       ▼
       Android                    iOS
          │                       │
          ▼                       ▼
   Android Toolchain        Kotlin/Native
```

Gradle coordinates much of this project configuration and task
execution.

------------------------------------------------------------------------

# 2. Gradle Is More Than a Build Command

A common mental model is:

``` text
./gradlew build
       ↓
    Gradle
       ↓
    APK
```

That is incomplete.

Gradle is a build automation system. It can model:

``` text
Projects
Modules
Tasks
Dependencies
Plugins
Configurations
Source Sets
Artifacts
Testing
Packaging
```

In KMP, these concepts are connected to multiple compilation targets.

A better mental model is:

``` text
                    Gradle
                      │
        ┌─────────────┼─────────────┐
        ▼             ▼             ▼
     Projects       Plugins      Dependencies
        │             │             │
        └─────────────┼─────────────┘
                      ▼
                   Targets
                      │
          ┌───────────┴───────────┐
          ▼                       ▼
      Android                    iOS
          │                       │
          ▼                       ▼
      Android                Native Output
      Compilation
```

------------------------------------------------------------------------

# 3. Why KMP Needs a Build System

Without a build system, a multiplatform project would have to
coordinate:

``` text
Kotlin compiler
Android toolchain
Kotlin/Native
Android SDK
Apple SDKs
Dependencies
Source sets
Target architectures
Tests
Packaging
```

And this would have to work consistently on:

``` text
Developer machines
CI
Release environments
```

Gradle provides a declarative way to describe much of that model.

Instead of manually performing:

``` text
compile this
copy that
link this
package that
```

we describe the project and its relationships.

Gradle then determines the required work.

------------------------------------------------------------------------

# 4. The KMP Build Has Multiple Dimensions

An Android-only project can often be visualized as:

``` text
Source
  ↓
Android
  ↓
APK / AAB
```

KMP introduces multiple target paths:

``` text
                    Shared Source
                         │
             ┌───────────┴───────────┐
             ▼                       ▼
          Android                    iOS
             │                       │
             ▼                       ▼
       Android Build            Native Build
```

The build system needs to understand both:

``` text
What is shared?
```

and:

``` text
What is platform-specific?
```

This is one reason Gradle configuration becomes important.

------------------------------------------------------------------------

# 5. Gradle Connects Architecture to Build

Think of the relationship as:

``` text
Architecture
     │
     ▼
Gradle Configuration
     │
     ▼
Build Model
     │
     ▼
Tasks
     │
     ▼
Artifacts
```

For example, `commonMain` is not merely a directory. It participates in
a target compilation model.

Similarly:

``` text
androidMain
iosMain
```

represent platform-specific compilation contexts.

Gradle, together with the Kotlin Multiplatform plugin, expresses these
relationships.

------------------------------------------------------------------------

# 6. The Kotlin Multiplatform Plugin

Gradle itself does not automatically understand every KMP concept.

The Kotlin Multiplatform plugin adds the multiplatform project model.

Conceptually:

``` text
Gradle
  │
  ▼
Kotlin Multiplatform Plugin
  │
  ├── Targets
  ├── Source Sets
  ├── Compilations
  └── Dependency Relationships
```

A useful distinction is:

``` text
Gradle
    = build automation system

Kotlin Multiplatform plugin
    = Kotlin tooling that adds multiplatform concepts to the Gradle build
```

------------------------------------------------------------------------

# 7. A Typical KMP Configuration

A simplified configuration can look like:

``` kotlin
plugins {
    kotlin("multiplatform")
}

kotlin {
    androidTarget()

    iosArm64()
    iosSimulatorArm64()
}
```

The exact syntax depends on the Kotlin and project versions.

The important information is:

``` text
This module supports:
    Android
    iOS device
    iOS simulator
```

That target declaration becomes part of the build model.

------------------------------------------------------------------------

# 8. What Is a Target?

A target represents a platform or environment for which Kotlin code is
compiled.

Examples include:

``` text
Android
iOS device
iOS simulator
```

Conceptually:

``` text
                KMP Module
                    │
        ┌───────────┼───────────┐
        ▼           ▼           ▼
     Android    iOS Device   iOS Simulator
```

A target is therefore more specific than saying:

``` text
"We support mobile."
```

It identifies a compilation destination.

------------------------------------------------------------------------

# 9. Why Targets Matter

Suppose your module contains:

``` text
commonMain
```

The build must know:

``` text
For which platforms should this code be compiled?
```

The target declarations answer that question.

``` text
commonMain
    │
    ├── Android
    ├── iOS Device
    └── iOS Simulator
```

The target configuration therefore defines part of the project's
compilation universe.

------------------------------------------------------------------------

# 10. What Is a Source Set?

A source set is a logical group of source files and resources that share
a compilation context.

Important KMP source sets include:

``` text
commonMain
commonTest
androidMain
iosMain
```

A useful mental model is:

``` text
                   shared
                     │
              ┌──────┴──────┐
              ▼             ▼
          Production       Tests
              │             │
        ┌─────┴─────┐       ▼
        ▼           ▼   commonTest
   commonMain   Platform
                Source Sets
```

Source sets help express which code belongs to the shared and
platform-specific parts of the project.

------------------------------------------------------------------------

# 11. Source Sets and Targets Work Together

Remember:

``` text
Source Sets
     +
Targets
     =
Compilations
```

Simplified:

``` text
commonMain
     │
     ├── Android
     └── iOS

androidMain
     │
     └── Android

iosMain
     │
     └── iOS
```

The actual Kotlin Multiplatform model is richer than this diagram, but
this mental model is extremely useful when reading a build file.

------------------------------------------------------------------------

# 12. Why Not Just Use One Source Folder?

An Android-only project may have:

``` text
src/main/
src/test/
```

KMP needs more information:

``` text
This code is common.
This code is Android-specific.
This code is iOS-specific.
This test is common.
This test is platform-specific.
```

Source sets make those boundaries explicit.

------------------------------------------------------------------------

# 13. Dependencies Are Target-Aware

Adding a dependency in KMP is not simply:

``` text
Add Library
```

You also need to ask:

``` text
Which source set uses it?
Which targets must compile it?
Does the library support those targets?
```

For example:

``` kotlin
commonMain.dependencies {
    implementation(...)
}
```

has a different meaning from:

``` kotlin
androidMain.dependencies {
    implementation(...)
}
```

Dependency placement is therefore an architectural decision.

------------------------------------------------------------------------

# 14. Why Dependency Placement Matters

Imagine an Android-only library.

If it is added to:

``` text
commonMain
```

you are effectively asking every target consuming that source set to
support it.

That can lead to:

``` text
Android → PASS
iOS     → FAIL
```

A dependency belongs in the narrowest source set that actually needs it.

Conceptually:

``` text
commonMain
 ├── Multiplatform networking
 ├── Serialization
 └── Domain libraries

androidMain
 └── Android-only library

iosMain
 └── Apple-specific library
```

------------------------------------------------------------------------

# 15. `build.gradle.kts`

The file:

``` text
build.gradle.kts
```

contains Gradle configuration written using Kotlin DSL.

The `.kts` means:

``` text
Kotlin DSL
```

rather than Groovy DSL.

But remember:

> **A Gradle Kotlin DSL file is build configuration, not application
> runtime code.**

It uses Kotlin syntax to describe the build.

------------------------------------------------------------------------

# 16. Application Code vs Build Code

Keep these categories separate.

### Application code

``` text
commonMain/
androidApp/
iosApp/
```

### Build code

``` text
build.gradle.kts
settings.gradle.kts
gradle/
gradle.properties
```

The first category becomes part of your application or library.

The second category tells the tooling how to assemble it.

------------------------------------------------------------------------

# 17. `settings.gradle.kts`

At the root you will typically find:

``` text
settings.gradle.kts
```

It is concerned with the overall Gradle build structure.

Conceptually:

``` text
settings.gradle.kts
        │
        ▼
     Root Build
        │
        ├── androidApp
        └── shared
```

It helps Gradle understand which projects/modules belong to the build
and can also participate in plugin and dependency management
configuration.

------------------------------------------------------------------------

# 18. Module `build.gradle.kts`

A module-level file such as:

``` text
shared/build.gradle.kts
```

can describe:

``` text
KMP plugin
Targets
Source sets
Dependencies
Compiler-related configuration
Native integration
```

While:

``` text
androidApp/build.gradle.kts
```

can describe:

``` text
Android application plugin
Android configuration
Application dependencies
Packaging
```

The module boundary matters.

------------------------------------------------------------------------

# 19. Plugins

Gradle plugins add capabilities to a project.

Conceptually:

``` text
Gradle
   │
   ▼
Plugin
   │
   ▼
Additional Build Capabilities
```

Examples include plugins for:

``` text
Kotlin Multiplatform
Android Application
Android Library
Kotlin Serialization
Testing
Publishing
```

The exact plugin IDs and versions depend on the project.

------------------------------------------------------------------------

# 20. Why Plugins Matter in KMP

The Kotlin Multiplatform plugin provides the concepts needed for:

``` text
Targets
Source Sets
Compilations
```

Conceptually:

``` text
Gradle
   │
   ▼
KMP Plugin
   │
   ├── Android Target
   ├── iOS Target
   ├── commonMain
   ├── androidMain
   └── iosMain
```

Without the relevant plugin, Gradle would not have the KMP-specific
model represented by this configuration.

------------------------------------------------------------------------

# 21. Toolchain Versions

A KMP build involves several pieces of tooling:

``` text
Gradle
Kotlin
Android Gradle Plugin
JDK
Android SDK
Xcode
Apple SDKs
```

These tools have compatibility relationships.

Therefore, changing one version blindly can break the build.

A safer approach is:

``` text
Current Project
      │
      ▼
Supported Versions
      │
      ▼
Controlled Upgrade
```

rather than:

``` text
Latest Everything
       │
       ▼
Change Everything
```

------------------------------------------------------------------------

# 22. Why Version Management Matters

Think of the build as several connected toolchains:

``` text
Kotlin
   │
   ▼
KMP Plugin
   │
   ▼
Gradle / Android Tooling
   │
   ▼
Android Output
```

and:

``` text
Kotlin
   │
   ▼
Kotlin/Native
   │
   ▼
Xcode / Apple SDK
   │
   ▼
iOS Output
```

A KMP build crosses more than one toolchain, so compatibility becomes a
real engineering concern.

------------------------------------------------------------------------

# 23. Gradle Wrapper

A project normally includes:

``` text
gradlew
gradlew.bat
gradle/
└── wrapper/
```

The wrapper lets the repository specify the Gradle distribution it
expects.

Run:

``` bash
./gradlew build
```

or on Windows:

``` powershell
.\gradlew.bat build
```

The project can therefore use its configured Gradle version instead of
relying on an arbitrary globally installed version.

------------------------------------------------------------------------

# 24. Why the Wrapper Matters

Without a wrapper:

``` text
Developer A → Gradle X
Developer B → Gradle Y
CI           → Gradle Z
```

With a wrapper:

``` text
Repository
     │
     ▼
Configured Gradle Version
     │
 ┌───┼───┐
 ▼   ▼   ▼
Dev Dev  CI
```

This improves build reproducibility.

The wrapper does not make every part of the environment identical, but
controlling Gradle itself removes one major source of variation.

------------------------------------------------------------------------

# 25. `gradle.properties`

Another common file is:

``` text
gradle.properties
```

It can contain Gradle and project properties.

Depending on the project, it may configure:

``` text
Gradle behavior
JVM-related settings
Android-related properties
Project flags
```

The exact contents vary.

Avoid copying properties from another project without understanding what
they change.

------------------------------------------------------------------------

# 26. Version Catalogs

Modern Gradle projects may use:

``` text
gradle/libs.versions.toml
```

A version catalog can centralize:

``` text
Dependency versions
Plugin versions
Dependency aliases
Plugin aliases
```

Conceptually:

``` text
libs.versions.toml
        │
        ├── Versions
        ├── Libraries
        └── Plugins
                │
                ▼
        build.gradle.kts
```

This makes dependency and plugin version ownership easier to manage
across modules.

------------------------------------------------------------------------

# 27. Why Version Catalogs Help KMP

A KMP repository may contain dependencies for:

``` text
Common
Android
iOS
Testing
Build plugins
```

Centralizing versions reduces repeated version strings.

But remember:

> A centralized version is not automatically a compatible version.

Compatibility still needs to be verified when upgrading the toolchain.

------------------------------------------------------------------------

# 28. The IDE Is Not the Build System

You can press:

``` text
Run
```

inside Android Studio.

But the build configuration still exists independently of the IDE.

Similarly, Xcode provides the native iOS development environment while
Gradle participates in the KMP shared-module build.

A useful model is:

``` text
IDE
 │
 ▼
Build Tooling
 │
 ▼
Compiler / Platform Toolchain
 │
 ▼
Artifact
```

The IDE gives you a convenient interface.

It is not the entire build system.

------------------------------------------------------------------------

# 29. Why CI Exposes This

On your machine:

``` text
Click Run
```

may hide much of the build process.

On CI, you normally need explicit commands and configuration:

``` text
./gradlew ...
```

This is why Gradle knowledge becomes important for:

``` text
CI/CD
Automation
Release Builds
Build Troubleshooting
```

------------------------------------------------------------------------

# 30. Gradle Tasks

Gradle organizes work into tasks.

Common categories include:

``` text
compile...
test...
check...
assemble...
```

The exact task names depend on the plugins, modules, targets, and
tooling versions in your project.

Think:

``` text
High-Level Task
       │
       ▼
Dependent Tasks
       │
       ▼
Compilation / Testing / Packaging
```

------------------------------------------------------------------------

# 31. The Task Graph

Suppose a simplified build has:

``` text
build
 ├── check
 │    └── test
 └── assemble
```

The graph can be visualized as:

``` text
                 build
                /                    ▼       ▼
           check     assemble
             │
             ▼
            test
```

Gradle resolves these relationships and executes the necessary work.

KMP projects can have much larger graphs because multiple targets and
compilations participate.

------------------------------------------------------------------------

# 32. Why the Task Graph Matters

When a build is slow or fails, ask:

``` text
Which task failed?
Which task is slow?
Which module owns the task?
Which target is involved?
Which source set contributed?
```

Instead of:

> "Gradle is broken."

you can begin investigating the actual build graph.

------------------------------------------------------------------------

# 33. A Simplified Build Lifecycle

A useful mental model is:

``` text
Gradle Invocation
       │
       ▼
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
```

You don't need to memorize every internal Gradle detail yet.

The important distinction is:

``` text
Configuration
```

creates the build model, while:

``` text
Execution
```

performs the required work.

------------------------------------------------------------------------

# 34. Configuration Is Not Compilation

When Gradle evaluates:

``` kotlin
kotlin {
    iosArm64()
}
```

it is configuring the build model.

It is not simply "compiling iOS" at that line.

Think:

``` text
Configuration
      │
      ▼
Build Model
      │
      ▼
Compilation Tasks
      │
      ▼
Execution
```

This distinction becomes useful when debugging Gradle behavior.

------------------------------------------------------------------------

# 35. Why KMP Build Files Can Look Complicated

A KMP module may configure:

``` text
Plugins
Targets
Source Sets
Dependencies
Compiler Options
Native Integration
Tests
Publishing
```

A large build file can therefore contain a lot of information.

Don't memorize every block.

Learn to identify the responsibility of each block.

------------------------------------------------------------------------

# 36. How to Read a KMP Build File

When you open:

``` text
shared/build.gradle.kts
```

read it in this order:

``` text
1. Plugins
2. Kotlin targets
3. Source sets
4. Dependencies
5. Platform-specific configuration
6. Additional build configuration
```

This gives you a mental map before you get lost in individual syntax.

------------------------------------------------------------------------

# 37. Plugins First

Start with:

``` kotlin
plugins {
    ...
}
```

Ask:

``` text
Which capabilities are being added?
```

For example:

``` text
Kotlin Multiplatform
Serialization
Android Library
```

The plugin list tells you what kind of module you are looking at.

------------------------------------------------------------------------

# 38. Targets Second

Find:

``` kotlin
kotlin {
    ...
}
```

Ask:

``` text
Which platforms does this module support?
```

You may see:

``` text
Android
iOS Device
iOS Simulator
```

Now you know the compilation destinations.

------------------------------------------------------------------------

# 39. Source Sets Third

Find:

``` kotlin
sourceSets {
    ...
}
```

Ask:

``` text
What code is shared?
What code is platform-specific?
What tests are shared?
```

This connects the build file directly to the folder structure from the
previous chapter.

------------------------------------------------------------------------

# 40. Dependencies Fourth

Inspect:

``` kotlin
dependencies {
    ...
}
```

Ask:

``` text
Which dependencies belong to common code?
Which are Android-only?
Which are iOS-specific?
```

Dependency placement is one of the most important parts of a KMP build.

------------------------------------------------------------------------

# 41. Platform Configuration Last

Finally inspect configuration related to:

``` text
Android
iOS
Native frameworks
Compiler options
Packaging
Publishing
```

Now that you understand the targets and dependencies, these sections
make much more sense.

------------------------------------------------------------------------

# 42. A Build File Is a Map

Think of:

``` text
shared/build.gradle.kts
```

as a map of the module.

It tells you:

``` text
What is this module?
Which platforms does it support?
What code is shared?
What code is platform-specific?
What does it depend on?
How should it be built?
```

Once you read it this way, Gradle becomes far less intimidating.

------------------------------------------------------------------------

# 43. Gradle and Architecture

Consider:

``` text
commonMain
   │
   ▼
Repository
   │
   ▼
Network Library
```

The build configuration must express the network library as a dependency
that is compatible with the relevant common targets.

Similarly:

``` text
Android-only API
```

should remain on the Android side.

The build configuration therefore reinforces architectural boundaries.

------------------------------------------------------------------------

# 44. The "Share Everything" Trap

One of the biggest KMP mistakes is:

``` text
More commonMain
=
Better KMP
```

It doesn't.

Forcing platform-specific code into `commonMain` can create:

``` text
Too many abstractions
Platform branching
Harder testing
Harder debugging
Unclear ownership
```

The goal is not maximum sharing.

The goal is useful sharing.

------------------------------------------------------------------------

# 45. Gradle as a Guardrail

Suppose:

``` text
commonMain
     │
     ▼
Android-only dependency
     │
     ▼
iOS compilation
```

The iOS build can expose the problem.

That failure is useful because it tells you:

> **This code is not actually common.**

The build system can therefore act as a guardrail around your
architecture.

------------------------------------------------------------------------

# 46. Build Failures Can Be Architecture Feedback

Imagine:

``` text
commonMain
     │
     ▼
Platform-specific library
     │
     ▼
Target compilation failure
```

It is tempting to say:

> "Gradle is causing trouble."

A better question is:

> **"Did I place this dependency in the correct source set?"**

KMP build failures often tell you where your platform boundary is wrong.

------------------------------------------------------------------------

# 47. Gradle and Reproducibility

A healthy project should ideally be buildable by:

``` text
Developer A
Developer B
CI
Release Engineer
```

using the repository's configuration.

That means version-controlled build configuration matters:

``` text
Git Repository
      │
      ├── Source
      ├── Gradle Configuration
      ├── Wrapper
      └── Version Definitions
              │
              ▼
         Reproducible Build
```

------------------------------------------------------------------------

# 48. Gradle and CI/CD

In a production KMP project, Gradle may participate in:

``` text
Unit Tests
Static Analysis
Android Build
Shared Module Build
Artifact Generation
Verification
Publishing
```

The iOS application may additionally require:

``` text
Xcode Build
Signing
Provisioning
Apple release workflows
```

So the overall pipeline can look like:

``` text
                    CI/CD
                      │
          ┌───────────┴───────────┐
          ▼                       ▼
       Gradle                    Xcode
          │                       │
          ▼                       ▼
   Shared / Android           iOS Application
       Outputs                    Output
```

Understanding Gradle early will make later CI/CD work much easier.

------------------------------------------------------------------------

# 49. Why Android Developers Should Learn Gradle More Deeply

If you come from Android, you may already know enough Gradle to:

``` text
Add a dependency
Change a version
Run a build
Create variants
```

KMP asks for deeper reasoning about:

``` text
Modules
Plugins
Targets
Source Sets
Compilations
Dependency scopes
Native outputs
Task graphs
Toolchain compatibility
```

You don't need to become a Gradle plugin author.

But you should become comfortable reading and reasoning about Gradle.

------------------------------------------------------------------------

# 50. The Shift in Thinking

Instead of:

> "Which Gradle command should I run?"

ask:

> **"What does this Gradle configuration tell the build system to do?"**

Instead of:

> "Why is this dependency failing?"

ask:

> **"Which source set is consuming this dependency, and which targets
> need to support it?"**

Instead of:

> "Why is iOS failing?"

ask:

> **"Which part of the multiplatform compilation graph is failing?"**

These questions lead to much better debugging.

------------------------------------------------------------------------

# 51. A Practical Example

Imagine:

``` text
shared/
└── src/
    ├── commonMain/
    ├── androidMain/
    └── iosMain/
```

And:

``` text
commonMain
```

contains:

``` kotlin
class ProductRepository
```

The build configuration conceptually expresses:

``` text
commonMain
   │
   ▼
Multiplatform Network Dependency
   │
   ├── Android implementation
   └── iOS implementation
```

Then:

``` text
Android App → shared
iOS App     → shared
```

Both applications consume the shared module.

------------------------------------------------------------------------

# 52. Another Example: Android-Only Dependency

Suppose someone adds:

``` text
Android-only Library
```

to:

``` text
commonMain
```

The graph becomes:

``` text
commonMain
     │
     ├── Android-only Library
     │
     ├── Android
     │
     └── iOS
```

The problem is immediately visible.

The library belongs in:

``` text
androidMain
```

or:

``` text
androidApp
```

depending on what the library is responsible for.

------------------------------------------------------------------------

# 53. Gradle Configuration Reflects the Architecture

A healthy project can often be understood from:

``` text
settings.gradle.kts
        │
        ▼
Module Structure
        │
        ▼
shared/build.gradle.kts
        │
        ├── Targets
        ├── Source Sets
        └── Dependencies
```

Application code tells you:

``` text
What the product does.
```

Build configuration tells you:

``` text
How the product is assembled.
```

Both are part of the engineering system.

------------------------------------------------------------------------

# 54. What Gradle Does Not Decide

Gradle does not decide:

``` text
Which business rules should be shared.
Whether KMP is right for your product.
Which UI framework you should use.
How your domain should behave.
Which architecture is best for your team.
```

Those are engineering decisions.

Gradle provides the machinery to implement those decisions.

Think:

``` text
Architectural Decision
        │
        ▼
Gradle Configuration
        │
        ▼
Build
```

not:

``` text
Gradle
   │
   ▼
Architecture
```

------------------------------------------------------------------------

# 55. Don't Blame Gradle for Architecture Problems

If a project contains:

``` text
commonMain
   └── 500 classes
```

while almost everything is platform-specific behind complicated
abstractions, the problem may not be Gradle.

It may be the architecture.

Gradle is executing the model you gave it.

Good Gradle knowledge helps you identify that distinction.

------------------------------------------------------------------------

# 56. Gradle as an Engineering Contract

A useful mental model is:

> **Gradle configuration is a contract between your source code and your
> build environment.**

It describes:

``` text
Modules
+
Plugins
+
Targets
+
Dependencies
+
Tasks
+
Toolchains
```

The build system uses that contract to produce the required outputs.

------------------------------------------------------------------------

# 57. Why Learn This Before Advanced KMP?

Before moving into:

``` text
expect / actual
Dependency Injection
KMP Networking
KMP Database
Shared State
Native Interop
```

you should understand:

``` text
How the project is built.
```

Otherwise every Gradle error can feel unrelated.

Once you understand the build model, errors become easier to classify:

``` text
Module problem
Target problem
Source-set problem
Dependency problem
Toolchain problem
Task problem
```

------------------------------------------------------------------------

# 58. The KMP Build Mental Model

Keep this diagram:

``` text
                         Gradle
                           │
                           ▼
                Kotlin Multiplatform
                           │
              ┌────────────┼────────────┐
              ▼            ▼            ▼
          Source Sets   Dependencies   Targets
              │            │            │
              └────────────┼────────────┘
                           ▼
                      Compilations
                           │
             ┌─────────────┴─────────────┐
             ▼                           ▼
          Android                        iOS
             │                           │
             ▼                           ▼
      Android Toolchain             Kotlin/Native
             │                           │
             ▼                           ▼
       Android Output              Native Output
```

This is the foundation for everything that follows in the Gradle
section.

------------------------------------------------------------------------

# Chapter Takeaways

> \[!TIP\] **Gradle is not just the command you run to build a KMP
> application. It is one of the places where the project's module,
> dependency, target, and compilation boundaries are defined.**

Remember:

1.  Gradle is a build automation system, not merely a build command.
2.  KMP adds additional build dimensions because one project can target
    multiple platforms.
3.  The Kotlin Multiplatform plugin adds KMP-specific concepts to the
    Gradle build model.
4.  Targets describe compilation destinations such as Android and iOS.
5.  Source sets organize code according to its compilation context.
6.  `commonMain` represents shared production code.
7.  `androidMain` represents Android-specific shared-module code.
8.  `iosMain` represents iOS-specific shared-module code.
9.  Source sets and targets work together to form compilations.
10. Dependency placement is an architectural decision in KMP.
11. A dependency that works on Android is not automatically suitable for
    `commonMain`.
12. `build.gradle.kts` is build configuration written using Kotlin DSL.
13. Application Kotlin and build-script Kotlin have different
    responsibilities.
14. `settings.gradle.kts` helps define the overall Gradle build
    structure.
15. Module-level `build.gradle.kts` files configure individual modules.
16. Plugins add capabilities to the Gradle build.
17. The Gradle wrapper helps keep the Gradle version consistent across
    environments.
18. Version catalogs can centralize dependency and plugin versions.
19. Gradle tasks represent units of build work.
20. The task graph determines which work is required for a requested
    task.
21. Build configuration and application architecture are closely
    connected in KMP.
22. Build failures can reveal incorrect platform boundaries.
23. The IDE is not the same thing as the underlying build system.
24. CI exposes whether your build is actually reproducible.
25. Gradle does not decide your architecture; it implements the build
    model you define.
26. Good KMP developers learn to read Gradle configuration rather than
    blindly copying commands.

------------------------------------------------------------------------

# Final Mental Model

When you open a KMP `build.gradle.kts`, don't see:

``` text
A giant Gradle file
```

See:

``` text
                    BUILD MODEL
                        │
                        ▼
                      Gradle
                        │
            ┌───────────┼───────────┐
            ▼           ▼           ▼
          Plugins     Targets    Dependencies
            │           │           │
            └───────────┼───────────┘
                        ▼
                   Source Sets
                        │
             ┌──────────┴──────────┐
             ▼                     ▼
         commonMain           Platform Sources
             │                 ┌────┴────┐
             │                 ▼         ▼
             │             Android      iOS
             │                 │         │
             └─────────────────┴─────────┘
                              │
                              ▼
                         Compilations
                              │
                   ┌──────────┴──────────┐
                   ▼                     ▼
               Android                 Native
                Output                Output
```

> **Gradle is the machinery that makes the KMP project model executable.
> Once you understand how Gradle connects plugins, targets, source sets,
> dependencies, and tasks, the KMP build stops looking like
> configuration syntax and starts looking like a map of how your
> multiplatform application is actually assembled.**
