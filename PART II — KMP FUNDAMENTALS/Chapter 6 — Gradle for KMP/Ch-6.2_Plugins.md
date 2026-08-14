# Chapter 6 — Gradle for KMP

## Part 2 — Plugins

> **A Gradle plugin is not just a line you add to make the build work. In a KMP project, plugins define capabilities, introduce build concepts, and shape how Gradle understands the project.**

Once you understand why Gradle exists, the next question is:

> **How does Gradle know that this is a Kotlin Multiplatform project?**

The answer begins with **plugins**.

A typical KMP module may contain something like:

```kotlin
plugins {
    kotlin("multiplatform")
}
```

An Android application may have:

```kotlin
plugins {
    id("com.android.application")
}
```

A shared module may also use plugins for:

```text
Serialization
Android Library
Code generation
Publishing
Testing
```

These declarations are not random configuration.

They tell Gradle:

```text
"What kind of project is this?"
"What capabilities should this project have?"
"What build model should be available?"
"What tasks and extensions should be created?"
```

That makes plugins one of the first things you should understand when reading a KMP build file.

---

# 1. What Is a Gradle Plugin?

At a high level, a Gradle plugin is a reusable piece of build logic.

It can add:

```text
Tasks
Extensions
Configurations
Conventions
Dependencies
Compilation behavior
Packaging behavior
```

Think of it as:

```text
                    Gradle
                      │
                      ▼
                    Plugin
                      │
          ┌───────────┼───────────┐
          ▼           ▼           ▼
        Tasks      Extensions   Build Model
```

Without plugins, every project would need to manually define large amounts of build logic.

Plugins allow that logic to be packaged and reused.

---

# 2. Why Does KMP Need Plugins?

Gradle by itself does not automatically understand concepts such as:

```text
commonMain
androidMain
iosMain
Kotlin Multiplatform targets
Kotlin/Native compilations
```

The Kotlin Multiplatform plugin adds those concepts to the Gradle project model.

Conceptually:

```text
                  Gradle
                    │
                    ▼
        Kotlin Multiplatform Plugin
                    │
        ┌───────────┼───────────┐
        ▼           ▼           ▼
      Targets    Source Sets  Compilations
        │           │           │
        └───────────┼───────────┘
                    ▼
               KMP Build Model
```

This is why the plugin is fundamental to a KMP shared module.

---

# 3. Plugin vs Dependency

This distinction is extremely important.

A **dependency** is generally something your application or library code uses.

A **plugin** changes or extends the build itself.

Think:

```text
Dependency
    │
    ▼
Application / Library Code
```

while:

```text
Plugin
    │
    ▼
Gradle Build
```

For example:

```text
Ktor
```

may be a runtime/library dependency.

While:

```text
Kotlin Multiplatform plugin
```

is build tooling.

The two solve completely different problems.

---

# 4. Plugin vs Library

Suppose your shared code contains:

```kotlin
class ProductRepository
```

and it uses a networking library.

The networking library participates in the compiled application.

The KMP plugin does not become business logic inside:

```text
ProductRepository
```

Instead, the plugin configures how the project containing that code is built.

A simplified picture:

```text
                  Gradle Build
                       │
                  KMP Plugin
                       │
                       ▼
                 Build Model
                       │
                       ▼
              Shared Kotlin Code
                       │
                Networking Library
                       │
                       ▼
                    Runtime
```

---

# 5. The `plugins {}` Block

The first place to look in a Gradle build file is usually:

```kotlin
plugins {
    ...
}
```

For example:

```kotlin
plugins {
    kotlin("multiplatform")
}
```

or:

```kotlin
plugins {
    id("com.android.application")
}
```

The block tells Gradle which plugins should be applied to the project.

Think of it as:

```text
This module needs these build capabilities.
```

---

# 6. The Kotlin Multiplatform Plugin

The central plugin for a KMP shared module is the Kotlin Multiplatform plugin.

A simplified declaration may look like:

```kotlin
plugins {
    kotlin("multiplatform")
}
```

Depending on the project's plugin-management setup and Kotlin version, you may also encounter the fully qualified plugin ID form.

The important concept is not the syntax.

It is:

```text
Kotlin Multiplatform Plugin
          │
          ▼
KMP Build Model
```

---

# 7. What Does the KMP Plugin Give You?

The Kotlin Multiplatform plugin enables configuration around concepts such as:

```text
Targets
Source Sets
Compilations
Platform-specific dependencies
Common dependencies
Native targets
Testing
```

Conceptually:

```text
                 KMP Plugin
                     │
       ┌─────────────┼─────────────┐
       ▼             ▼             ▼
    Targets      Source Sets   Compilations
       │             │             │
       └─────────────┼─────────────┘
                     ▼
                 KMP Module
```

It effectively gives Gradle the vocabulary needed to describe a multiplatform Kotlin module.

---

# 8. The Plugin Creates a Project Model

Consider:

```kotlin
kotlin {
    androidTarget()

    iosArm64()
    iosSimulatorArm64()
}
```

Without the Kotlin Multiplatform plugin, this configuration would have no meaning in an ordinary Gradle project.

With the plugin:

```text
kotlin { ... }
```

becomes a KMP configuration area.

That is a key idea:

> **Plugins don't merely execute commands. They can introduce new concepts into the Gradle model.**

---

# 9. Plugin Extensions

A plugin can expose configuration through an extension.

For example:

```kotlin
kotlin {
    ...
}
```

The:

```text
kotlin
```

configuration is associated with the Kotlin tooling.

Conceptually:

```text
KMP Plugin
    │
    ▼
Kotlin Extension
    │
    ├── Targets
    ├── Source Sets
    └── Compilation Configuration
```

This is why build files can feel declarative.

You are configuring a model exposed by the plugin.

---

# 10. Plugins Add Tasks

Plugins can also contribute tasks.

For example, after applying:

```text
Kotlin Multiplatform
```

the project can expose tasks related to:

```text
Kotlin compilation
Testing
Native compilation
Verification
```

The exact task names depend on the plugin versions and targets configured.

This is why:

```bash
./gradlew tasks
```

is useful.

It shows what the current project actually knows how to do.

---

# 11. Don't Memorize Every Plugin Task

A common mistake is learning a command from one tutorial:

```bash
./gradlew someTask
```

and assuming it will always exist.

KMP projects evolve.

Plugin versions evolve.

Target configurations evolve.

Therefore:

```text
Current Project
      │
      ▼
Applied Plugins
      │
      ▼
Configured Targets
      │
      ▼
Available Tasks
```

The task graph is generated from the project's configuration.

---

# 12. Plugins and Module Types

Different modules often require different plugins.

For example:

```text
androidApp
```

may use:

```text
Android Application Plugin
```

while:

```text
shared
```

may use:

```text
Kotlin Multiplatform Plugin
```

The resulting project might look like:

```text
                 Root Project
                      │
          ┌───────────┴───────────┐
          ▼                       ▼
      androidApp                shared
          │                       │
          ▼                       ▼
 Android Application       Kotlin Multiplatform
       Plugin                   Plugin
```

The plugins help define what each module is capable of.

---

# 13. Android Application Plugin

An Android application module commonly applies an Android application plugin.

Conceptually:

```kotlin
plugins {
    id("com.android.application")
}
```

This tells Gradle that the module is an Android application.

It enables Android-specific build concepts such as:

```text
Android variants
Manifest processing
Resources
Packaging
APK / application outputs
```

The exact configuration depends on the Android Gradle Plugin and project setup.

---

# 14. Android Library Plugin

A shared KMP module may also involve Android library configuration depending on how the project is structured.

Conceptually:

```kotlin
id("com.android.library")
```

means:

```text
This module participates as an Android library.
```

In a KMP project, the Kotlin Multiplatform and Android tooling can work together to make a module usable from Android while it also supports other targets.

The exact plugin arrangement depends on the current KMP project template and Kotlin/Android tooling.

---

# 15. KMP Plugin + Android Tooling

A common KMP architecture contains:

```text
shared
   │
   ├── Kotlin Multiplatform
   │
   └── Android-specific configuration
```

while:

```text
androidApp
   │
   └── Android Application
```

Conceptually:

```text
                 Gradle
                   │
          ┌────────┴────────┐
          ▼                 ▼
       shared            androidApp
          │                 │
   KMP + Android        Android App
      tooling             tooling
```

The two modules have different responsibilities.

---

# 16. Plugins Are About Capability

When you see:

```kotlin
plugins {
    ...
}
```

ask:

> **What capability is this plugin adding?**

For example:

```text
KMP Plugin
→ Multiplatform targets and source sets

Android Application Plugin
→ Android application build

Serialization Plugin
→ Serialization-related compiler/build support

Publishing Plugin
→ Publication and artifact configuration
```

This question is much more useful than simply memorizing plugin IDs.

---

# 17. Serialization Plugin

A KMP project may use Kotlin Serialization.

You may see a plugin related to:

```text
Kotlin Serialization
```

This plugin is different from the serialization library itself.

Think:

```text
Serialization Plugin
        │
        ▼
Build / Compiler Support
```

while:

```text
Serialization Library
        │
        ▼
Application Runtime
```

Again:

```text
Plugin ≠ Dependency
```

---

# 18. Plugin and Runtime Library Can Work Together

A feature may require both:

```text
Plugin
+
Library
```

Conceptually:

```text
                 Serialization
                      │
             ┌────────┴────────┐
             ▼                 ▼
          Plugin             Library
             │                 │
             ▼                 ▼
       Build Support       Runtime APIs
```

This pattern appears in many Kotlin projects.

The plugin configures build/compiler behavior.

The library provides APIs used by your code.

---

# 19. Why Plugin Management Exists

Imagine a project with:

```text
10 modules
```

and many modules need:

```text
Kotlin
Android
Serialization
```

You don't want every module independently resolving arbitrary plugin versions.

Gradle provides plugin management mechanisms so plugin versions and repositories can be controlled centrally.

This is often configured in:

```text
settings.gradle.kts
```

or related settings files.

---

# 20. Plugin Resolution

At a high level:

```text
plugins {
    ...
}
```

declares the plugins a project needs.

Gradle then resolves them using the project's plugin management configuration.

Conceptually:

```text
build.gradle.kts
       │
       ▼
Plugin Request
       │
       ▼
Plugin Management
       │
       ▼
Plugin Resolution
       │
       ▼
Plugin Applied
```

This is different from resolving ordinary application dependencies.

---

# 21. Plugin Versions

A plugin normally has a version associated with it.

For example, conceptually:

```text
Kotlin Plugin
     │
     ▼
Version X
```

The exact version syntax depends on how the project manages plugins.

The important principle is:

> **Plugin versions are part of the build toolchain, not application dependency versions.**

They should be upgraded deliberately.

---

# 22. Why Plugin Upgrades Can Be Risky

Suppose you upgrade:

```text
Kotlin
```

The change can affect:

```text
KMP plugin behavior
Compiler behavior
Native compilation
Gradle compatibility
Android integration
Third-party plugins
```

Similarly, upgrading:

```text
Android Gradle Plugin
```

can affect:

```text
Gradle version
Android build configuration
Task behavior
Variant APIs
```

This is why plugin upgrades should be treated as toolchain changes.

---

# 23. Plugin Compatibility Is a Graph

Think:

```text
                 Kotlin
                   │
                   ▼
             KMP Plugin
                   │
          ┌────────┴────────┐
          ▼                 ▼
       Gradle              Android
          │                 │
          ▼                 ▼
      Build System       AGP / SDK
                              │
                              ▼
                           Android
```

And for iOS:

```text
Kotlin
   │
   ▼
Kotlin/Native
   │
   ▼
Xcode / Apple SDK
```

The exact compatibility matrix changes over time.

The important lesson is that a plugin version is never isolated from the rest of the toolchain.

---

# 24. Plugins and Convention

A plugin can also establish conventions.

For example:

```text
Apply plugin
     │
     ▼
Standard configuration
```

This allows a team to avoid repeating the same build logic across many modules.

Large projects often create their own convention plugins for this reason.

---

# 25. Why Convention Plugins Matter

Imagine:

```text
feature-a
feature-b
feature-c
feature-d
```

and every module repeats:

```text
Kotlin settings
Testing settings
Compiler options
Static analysis
Common dependencies
```

The build becomes repetitive.

A convention plugin can centralize those rules.

Conceptually:

```text
                 Convention Plugin
                         │
          ┌──────────────┼──────────────┐
          ▼              ▼              ▼
      feature-a      feature-b      feature-c
```

This is especially useful in large KMP codebases.

---

# 26. Build Logic as a Separate Concern

As projects grow, you may move reusable build logic into dedicated build-logic modules or convention plugins.

Then:

```text
Application Module
       │
       ▼
Convention Plugin
       │
       ▼
Shared Build Rules
```

The application module becomes smaller and easier to read.

This is an advanced topic, but the concept is worth knowing early.

---

# 27. Plugin Composition

A project can apply multiple plugins.

For example, conceptually:

```kotlin
plugins {
    kotlin("multiplatform")
    kotlin("plugin.serialization")
}
```

The resulting build model becomes:

```text
              Gradle Project
                    │
          ┌─────────┴─────────┐
          ▼                   ▼
        KMP                  Serialization
       Plugin                  Plugin
          │                   │
          └─────────┬─────────┘
                    ▼
              Configured Module
```

Each plugin contributes its own capabilities.

They can also interact.

---

# 28. Plugin Order

In most modern Gradle plugin configurations, you should not think of plugin declarations as a simple procedural script where line 1 runs completely before line 2.

The important thing is:

```text
Which plugins are applied?
What extensions do they provide?
How do their configurations interact?
```

Some plugins have relationships or requirements, so follow their documented setup rather than rearranging plugin declarations experimentally.

---

# 29. Plugin IDs vs Kotlin Helper Syntax

You may see:

```kotlin
kotlin("multiplatform")
```

or:

```kotlin
id("org.jetbrains.kotlin.multiplatform")
```

These represent plugin requests using different syntax.

The exact form depends on the project's plugin management and Kotlin Gradle plugin setup.

When reading a project, focus first on:

```text
Which plugin?
```

rather than:

```text
Which syntax did they use?
```

---

# 30. Plugin Aliases

With a version catalog, you may see plugin aliases such as:

```kotlin
plugins {
    alias(libs.plugins.kotlin.multiplatform)
}
```

Conceptually:

```text
libs.versions.toml
       │
       ▼
Plugin Alias
       │
       ▼
build.gradle.kts
```

This moves plugin coordinates and versions into a centralized catalog.

It can make multi-module projects easier to maintain.

---

# 31. Why Aliases Help

Without aliases, several modules might repeat:

```text
Plugin ID
Plugin Version
```

With aliases:

```text
Central Definition
        │
        ├── shared
        ├── feature-a
        └── feature-b
```

This reduces duplication.

It also makes version upgrades easier to review.

---

# 32. Plugins and Source Sets

The relationship can be visualized as:

```text
KMP Plugin
    │
    ▼
Source Set Model
    │
    ├── commonMain
    ├── commonTest
    ├── androidMain
    └── iosMain
```

The plugin creates the environment in which these source sets can be configured.

This is why:

```text
commonMain
```

is meaningful in a KMP module but not as a generic Gradle concept.

---

# 33. Plugins and Targets

Similarly:

```text
KMP Plugin
    │
    ▼
Target Model
    │
    ├── Android
    ├── iOS Device
    └── iOS Simulator
```

The plugin exposes the APIs used to declare and configure those targets.

---

# 34. Plugins and Compilations

Targets eventually lead to compilations.

Conceptually:

```text
Plugin
  │
  ▼
Target
  │
  ▼
Compilation
  │
  ▼
Compiler
  │
  ▼
Artifact
```

This is why applying the correct plugin is fundamental.

The plugin establishes the build model that makes these relationships possible.

---

# 35. A Plugin Does Not Compile Your Business Logic by Itself

This distinction is useful.

When you apply:

```text
KMP Plugin
```

you are not saying:

```text
Run my business logic.
```

You are saying:

```text
Configure this project as a Kotlin Multiplatform build.
```

Then the resulting build tasks compile the application code.

---

# 36. Plugin vs Task

Another important distinction:

```text
Plugin
```

defines or contributes build capabilities.

```text
Task
```

performs a unit of build work.

Think:

```text
Plugin
   │
   ▼
Provides / Configures
   │
   ▼
Tasks
   │
   ▼
Execute Work
```

For example:

```text
KMP Plugin
     │
     ▼
KMP-related tasks
     │
     ▼
Compilation / Testing / Verification
```

---

# 37. Plugin vs Extension vs Task

These three concepts are worth separating:

### Plugin

Adds build capabilities.

```text
Plugin
```

### Extension

Provides configuration for those capabilities.

```text
kotlin {
    ...
}
```

### Task

Performs work.

```text
compile...
test...
assemble...
```

A simplified relationship is:

```text
Plugin
  │
  ├── Extension
  │      │
  │      ▼
  │   Configuration
  │
  └── Tasks
         │
         ▼
       Work
```

This mental model will help enormously when reading Gradle code.

---

# 38. Plugin and Extension Example

Conceptually:

```kotlin
plugins {
    kotlin("multiplatform")
}

kotlin {
    androidTarget()
    iosArm64()
}
```

Think of it as:

```text
Apply KMP Plugin
       │
       ▼
Kotlin Multiplatform Extension
       │
       ▼
Configure Targets
       │
       ▼
Generate / Configure Build Tasks
```

You are configuring the plugin's model rather than manually creating every compilation task yourself.

---

# 39. Android Plugin Example

Similarly:

```kotlin
plugins {
    id("com.android.application")
}
```

followed by Android configuration:

```kotlin
android {
    ...
}
```

can be understood as:

```text
Apply Android Application Plugin
            │
            ▼
      Android Extension
            │
            ▼
     Configure Android
            │
            ▼
    Android Build Tasks
```

The pattern is similar even though the capabilities are different.

---

# 40. Plugin Configuration Is Declarative

This is one of Gradle's strengths.

Instead of manually saying:

```text
Create compiler task
Set source directory
Connect dependency
Create packaging task
```

you declare:

```kotlin
kotlin {
    iosArm64()
}
```

The plugin interprets the declaration and configures the necessary build model.

Conceptually:

```text
Declaration
    │
    ▼
Plugin Model
    │
    ▼
Tasks + Configurations
```

---

# 41. Why This Matters for KMP

The KMP project contains many moving pieces.

Declarative plugin configuration allows you to express:

```text
Targets
Source Sets
Dependencies
```

at a higher level.

You don't need to manually manage every low-level compiler operation.

That is the power of the plugin ecosystem.

---

# 42. What Happens If the Plugin Is Missing?

Suppose you write:

```kotlin
kotlin {
    iosArm64()
}
```

without the appropriate Kotlin Multiplatform plugin.

Gradle cannot interpret the configuration as intended.

You may see errors related to:

```text
Unknown configuration
Unresolved reference
Missing extension
Missing task
```

The root problem is:

```text
The project does not have the capability being configured.
```

---

# 43. Reading Plugin Errors

When Gradle reports something like:

```text
Could not find method ...
```

or:

```text
Unresolved reference ...
```

ask:

```text
Which plugin provides this API?
```

For example:

```text
kotlin { ... }
```

is associated with Kotlin tooling.

Similarly:

```text
android { ... }
```

requires appropriate Android tooling.

This is a powerful debugging technique.

---

# 44. Plugin Errors vs Dependency Errors

Compare:

### Plugin problem

```text
Build configuration
        │
        ▼
Plugin / Extension
        │
        ▼
Gradle configuration failure
```

### Dependency problem

```text
Source Set
    │
    ▼
Dependency
    │
    ▼
Resolution / Compilation failure
```

The location of the error often tells you which category to investigate.

---

# 45. Plugins and KMP Architecture

The plugins in a module tell you a lot about the module's role.

For example:

```text
KMP Plugin
```

suggests:

```text
Multiplatform Kotlin module
```

while:

```text
Android Application Plugin
```

suggests:

```text
Android application module
```

Therefore, the plugin block is often the quickest way to understand an unfamiliar Gradle module.

---

# 46. A Simple Module Map

Consider:

```text
Project
│
├── androidApp
│     └── Android Application Plugin
│
└── shared
      └── Kotlin Multiplatform Plugin
```

Immediately you can infer:

```text
androidApp
→ produces the Android application

shared
→ contains multiplatform code
```

The plugin block provides architectural clues before you even read the source code.

---

# 47. Plugins in a Larger KMP Project

A production repository might look more like:

```text
Project
│
├── androidApp
│
├── shared
│
├── core
│
├── feature-auth
│
├── feature-products
│
└── build-logic
```

Each module may use different plugins or convention plugins.

Conceptually:

```text
                  Build Logic
                      │
          ┌───────────┼───────────┐
          ▼           ▼           ▼
       Android       KMP        Feature
       Modules      Modules     Modules
```

This is where plugin architecture becomes increasingly important.

---

# 48. Convention Plugins in Large Projects

Suppose every KMP module needs:

```text
Kotlin Multiplatform
Testing
Compiler configuration
Common repositories
Code quality
```

Instead of repeating configuration:

```text
feature-a → same configuration
feature-b → same configuration
feature-c → same configuration
```

you can create a convention plugin:

```text
convention-kmp
```

Then:

```text
feature-a ─┐
feature-b ─┼──► convention-kmp
feature-c ─┘
```

The shared build rules become centralized.

---

# 49. Why Convention Plugins Are Better Than Copy-Paste

Copy-paste build logic creates drift.

For example:

```text
feature-a → Kotlin settings A
feature-b → Kotlin settings B
feature-c → Kotlin settings A
```

Eventually nobody knows why.

A convention plugin gives:

```text
One Build Convention
        │
        ▼
Many Modules
```

Changing the convention updates the behavior consistently.

---

# 50. Plugins Are Part of the Build Architecture

At this point, the relationship should be clear:

```text
Application Architecture
        │
        ▼
Module Architecture
        │
        ▼
Plugin Configuration
        │
        ▼
Build Model
        │
        ▼
Artifacts
```

Plugins are therefore not just syntax at the top of a Gradle file.

They are part of how the repository is organized.

---

# 51. A Realistic KMP Module

A simplified shared module might look like:

```kotlin
plugins {
    kotlin("multiplatform")
    kotlin("plugin.serialization")
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

Read this as:

```text
Apply KMP capabilities
        │
        ├── Multiplatform
        └── Serialization
                │
                ▼
        Configure targets
                │
                ▼
        Configure source sets
                │
                ▼
        Configure dependencies
```

The exact configuration of a production project will be richer.

---

# 52. The Order of Understanding

When you see a KMP plugin block, think:

```text
Plugin
  ↓
What capability?
  ↓
What extension?
  ↓
What targets?
  ↓
What source sets?
  ↓
What dependencies?
  ↓
What tasks?
```

This sequence is much more useful than memorizing Gradle syntax.

---

# 53. Don't Treat Plugin Configuration as Magic

When you see:

```kotlin
kotlin {
    ...
}
```

don't think:

> "This is some special KMP syntax."

Think:

```text
A plugin exposed a configuration model.
I am configuring that model.
The plugin will translate the configuration into build behavior.
```

That mental shift makes Gradle much easier to learn.

---

# 54. The Plugin Pipeline

A useful visualization is:

```text
                 Plugin Request
                       │
                       ▼
                 Plugin Resolution
                       │
                       ▼
                   Plugin Applied
                       │
                       ▼
                 Build Extensions
                       │
                       ▼
                   Configuration
                       │
                       ▼
                  Task Graph
                       │
                       ▼
                    Build
```

This is the lifecycle you should keep in mind.

---

# 55. Why Plugins Matter for KMP

Without the right plugins, Gradle cannot provide the project model required to express:

```text
Android target
iOS target
commonMain
androidMain
iosMain
Native compilation
```

The plugin layer is what connects:

```text
Kotlin Multiplatform concepts
```

to:

```text
Gradle's build system.
```

---

# 56. The Deeper Architectural Lesson

There is a broader lesson here.

When your application grows, you don't want every module to understand every platform.

Similarly, you don't want every build file to understand every build rule.

Plugins and convention plugins can create boundaries.

For example:

```text
KMP Convention
      │
      ▼
KMP Modules

Android Convention
      │
      ▼
Android Modules
```

The build architecture can therefore mirror the application architecture.

---

# 57. Plugin Ownership

In a large project, ask:

```text
Who owns this plugin?
```

It might be:

```text
Official Kotlin plugin
Official Android plugin
Third-party plugin
Internal company plugin
Convention plugin
```

This matters when debugging.

If a build breaks after an upgrade, knowing which plugin owns the affected configuration helps narrow the investigation.

---

# 58. Third-Party Plugins

KMP projects may use third-party plugins for:

```text
Code generation
Database integration
Static analysis
Dependency injection
Code quality
Publishing
```

Before applying one, verify:

```text
Does it support KMP?
Which targets does it support?
Which Kotlin versions does it support?
Which Gradle versions does it support?
Does it work with the current Android tooling?
```

A plugin can look convenient while creating a platform compatibility problem later.

---

# 59. Plugin Selection Is an Engineering Decision

Don't choose a plugin simply because:

```text
"Everyone uses it."
```

Ask:

```text
What problem does it solve?
What does it add to the build?
Which targets does it support?
How does it affect build time?
How actively is it maintained?
What does it lock us into?
```

Build dependencies are dependencies too.

---

# 60. Keep the Plugin Surface Small

A useful principle is:

> **Use the smallest set of plugins that gives the project the capabilities it actually needs.**

Every additional plugin can introduce:

```text
Configuration
Compatibility requirements
Tasks
Build time
Upgrade work
Potential conflicts
```

This doesn't mean avoiding plugins.

It means using them intentionally.

---

# 61. Plugin Debugging Checklist

When a plugin-related build error appears, ask:

```text
[ ] Is the plugin applied?
[ ] Is the plugin ID correct?
[ ] Is the plugin version compatible?
[ ] Is plugin resolution configured?
[ ] Is the expected extension available?
[ ] Is the configuration block in the correct module?
[ ] Does another plugin affect this configuration?
[ ] Is the target supported?
[ ] Is the Gradle version compatible?
[ ] Is the Kotlin version compatible?
```

This checklist can eliminate a lot of random experimentation.

---

# 62. The Most Important Distinction

Remember these three layers:

```text
Plugin
   │
   ▼
Build Configuration
   │
   ▼
Task Execution
```

For example:

```text
KMP Plugin
   │
   ▼
kotlin { ... }
   │
   ▼
Compilation Tasks
```

The plugin provides the capability.

The configuration describes what you want.

The tasks perform the work.

---

# 63. Plugin Mental Model for Android Developers

If you already know Android Gradle, think of KMP plugins as an extension of a familiar idea.

You already know that applying Android tooling makes:

```text
android {
    ...
}
```

meaningful.

KMP works with the same general philosophy:

```text
KMP Plugin
   │
   ▼
kotlin {
    ...
}
```

The difference is the model being configured.

Android tooling understands Android application/library concepts.

KMP tooling understands multiplatform targets and source sets.

---

# 64. The Complete Plugin Picture

Keep this model:

```text
                         Gradle
                           │
                           ▼
                     Plugin System
                           │
              ┌────────────┼────────────┐
              ▼            ▼            ▼
             KMP         Android    Serialization
            Plugin        Plugin       Plugin
              │            │            │
              ▼            ▼            ▼
          Multiplatform  Android      Build /
             Model        Model       Compiler
              │            │          Support
              └────────────┼────────────┘
                           ▼
                       Build Model
                           │
                           ▼
                       Task Graph
                           │
             ┌─────────────┴─────────────┐
             ▼                           ▼
          Android                        iOS
             │                           │
             ▼                           ▼
       Android Output              Native Output
```

---

# Chapter Takeaways

> [!TIP]
> **A Gradle plugin adds capabilities to the build. In KMP, the Kotlin Multiplatform plugin is what gives Gradle the vocabulary to understand targets, source sets, compilations, and multiplatform dependencies.**

Remember:

1. A Gradle plugin is reusable build logic that adds capabilities to a project.
2. Plugins can add tasks, extensions, configurations, and build conventions.
3. The Kotlin Multiplatform plugin provides the model needed to configure KMP projects.
4. A plugin is different from a runtime/library dependency.
5. The `plugins {}` block declares plugin requests for a module.
6. `kotlin { ... }` is meaningful in a KMP module because Kotlin tooling provides the relevant configuration model.
7. Targets describe where Kotlin code can be compiled.
8. Source sets describe groups of code with a particular compilation context.
9. Plugins can contribute tasks that become part of the Gradle task graph.
10. Plugin-generated task names can change with tooling and project configuration.
11. Android application and KMP shared modules can require different plugins because they have different responsibilities.
12. Plugin versions are part of the build toolchain.
13. Kotlin, Gradle, Android tooling, JDK, Xcode, and Apple SDK versions can have compatibility relationships.
14. Plugin aliases can centralize plugin declarations through version catalogs.
15. Plugins and dependencies solve different problems.
16. A plugin can expose extensions that provide declarative configuration.
17. A task performs build work; a plugin provides or configures the capabilities that make tasks possible.
18. Convention plugins help large projects avoid duplicated build configuration.
19. Third-party plugins should be evaluated for target and toolchain compatibility.
20. Every plugin adds some build complexity, so plugin selection should be intentional.
21. Plugin-related failures should be investigated through the plugin, extension, configuration, and task layers.
22. The plugin block is often the fastest way to understand the purpose of an unfamiliar Gradle module.
23. Build architecture can use plugins and conventions to reinforce application architecture.
24. Good KMP developers understand what a plugin contributes instead of treating Gradle configuration as magic.

---

# Final Mental Model

When you see:

```kotlin
plugins {
    kotlin("multiplatform")
}
```

don't read it as:

```text
"Some Gradle syntax required by KMP."
```

Read it as:

```text
                     Gradle
                       │
                       ▼
                  Apply Plugin
                       │
                       ▼
             Kotlin Multiplatform
                       │
             ┌─────────┼─────────┐
             ▼         ▼         ▼
          Targets   Source Sets  Compilations
             │         │         │
             └─────────┼─────────┘
                       ▼
                   Build Model
                       │
                       ▼
                   Task Graph
                       │
              ┌────────┴────────┐
              ▼                 ▼
           Android              iOS
              │                 │
              ▼                 ▼
        Android Output      Native Output
```

And remember:

> **A plugin is the bridge between Gradle's generic build system and the specialized concepts of a technology. In KMP, the Kotlin Multiplatform plugin teaches Gradle how to model multiple targets, source sets, compilations, and platform-aware dependencies. Once you understand that relationship, the `plugins {}` block stops looking like boilerplate and starts telling you what the module is capable of building.**
