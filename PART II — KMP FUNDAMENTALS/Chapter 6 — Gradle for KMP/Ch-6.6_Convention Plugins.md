# Chapter 6 — Gradle for KMP

## Part 6 — Convention Plugins

> **A Version Catalog centralizes what your build uses. Convention Plugins centralize how your build is configured. Together, they turn Gradle from a collection of repeated scripts into an intentional build system.**

As a KMP project grows, another problem appears even after introducing a Version Catalog.

You may have successfully centralized:

```text
Kotlin version
Library versions
Plugin versions
Dependency aliases
```

but your module build files can still contain repeated configuration.

For example:

```text
shared/build.gradle.kts
feature-a/build.gradle.kts
feature-b/build.gradle.kts
feature-c/build.gradle.kts
```

may all repeat:

```text
Kotlin compiler configuration
Java compatibility
Kotlin compiler options
Testing configuration
KMP target configuration
Android library configuration
Code quality configuration
```

This creates a different kind of duplication.

The versions are centralized.

The **build logic is still duplicated**.

Convention Plugins solve this problem.

---

# 1. What Is a Convention Plugin?

A Convention Plugin is a Gradle plugin created by your project to apply a shared build convention to multiple modules.

Instead of repeating:

```kotlin
plugins {
    ...
}

kotlin {
    ...
}

android {
    ...
}
```

in many modules, you can define the behavior once.

Conceptually:

```text
Convention Plugin
       │
       ├── Module A
       ├── Module B
       ├── Module C
       └── Module D
```

Every module gets the same agreed-upon build behavior.

---

# 2. Why Do KMP Projects Need Convention Plugins?

A simple KMP project might have:

```text
shared
androidApp
```

You may not need much build abstraction.

But a larger project might contain:

```text
shared
core-domain
core-data
feature-auth
feature-profile
feature-orders
feature-products
```

Now multiple modules may require similar configuration.

Without conventions:

```text
Module A
 └── repeated Gradle configuration

Module B
 └── repeated Gradle configuration

Module C
 └── repeated Gradle configuration
```

With conventions:

```text
                  Convention Plugin
                         │
          ┌──────────────┼──────────────┐
          ▼              ▼              ▼
       Module A       Module B       Module C
```

The configuration becomes centralized.

---

# 3. Version Catalog vs Convention Plugin

These two concepts are often confused.

### Version Catalog

Answers:

> **What do we use?**

For example:

```text
Kotlin version
Coroutines version
Serialization version
Plugin versions
```

### Convention Plugin

Answers:

> **How should this type of module be configured?**

For example:

```text
Apply Kotlin Multiplatform
Configure targets
Configure compiler options
Configure testing
Configure common repositories
```

Think:

```text
Version Catalog
     │
     └── What?

Convention Plugin
     │
     └── How?
```

They complement each other.

---

# 4. The Problem With Copy-Paste Gradle

Imagine three KMP modules:

```text
shared
feature-auth
feature-orders
```

Each contains:

```kotlin
kotlin {
    androidTarget()
    iosArm64()
    iosSimulatorArm64()
}
```

and:

```kotlin
compilerOptions {
    ...
}
```

Then six months later, the project changes its target configuration.

Without conventions:

```text
Change shared
Change feature-auth
Change feature-orders
Change feature-profile
Change feature-cart
...
```

With a convention plugin:

```text
Change convention once
        │
        ▼
All participating modules
```

This is the real value.

---

# 5. Convention Is a Build Standard

A convention represents a project-level decision.

For example:

```text
All shared KMP libraries:
    - use KMP
    - support Android
    - support iOS
    - use the same compiler settings
    - use the same test configuration
```

Instead of documenting those rules only in a wiki, the build system can enforce them.

That makes the convention executable.

---

# 6. Build Logic as Code

A powerful mental model is:

```text
Application Code
       │
       └── Business behavior

Build Logic
       │
       └── Build behavior
```

Gradle configuration is software too.

It has:

```text
Rules
Dependencies
APIs
Versions
Tests
Maintenance
```

Treating build logic as real code makes large projects easier to maintain.

---

# 7. Where Convention Plugins Live

A common project structure is:

```text
project/
│
├── app/
├── shared/
├── feature-auth/
│
├── build-logic/
│   └── convention/
│
├── gradle/
│   └── libs.versions.toml
│
└── settings.gradle.kts
```

Another common approach is to use an included build such as:

```text
build-logic/
```

The exact structure can vary.

The architectural idea is:

```text
Application Modules
        │
        ▼
Project Build Logic
```

---

# 8. `build-logic` as a Separate Build

A common setup uses:

```text
build-logic/
```

as an included build.

Conceptually:

```text
Root Build
    │
    ├── Application Modules
    │
    └── build-logic
            │
            └── Convention Plugins
```

This keeps reusable build logic separate from application source code.

---

# 9. Why Not Put Everything in the Root Build File?

A common early-stage approach is:

```kotlin
subprojects {
    ...
}
```

or:

```kotlin
allprojects {
    ...
}
```

This can appear convenient.

But as the repository grows, global configuration can become difficult to understand.

You may no longer know:

```text
Which module gets this?
Why does this configuration apply here?
What happens if another module is added?
```

Convention Plugins make the relationship explicit.

---

# 10. Explicit Application

Instead of implicit global behavior:

```text
Every module gets configuration
```

you can have:

```kotlin
plugins {
    id("my.kmp.library")
}
```

Now the module explicitly says:

> "I follow the project's KMP library convention."

That is easier to reason about.

---

# 11. Convention Plugin Naming

Names should describe the convention.

Examples:

```text
my.kmp.library
my.android.library
my.android.application
my.kotlin.multiplatform
my.compose.multiplatform
```

The exact naming scheme is a team decision.

A good name should communicate:

```text
What type of module is this?
```

---

# 12. Convention Plugins Are Not Application Plugins

A convention plugin generally configures the build.

It is not your business application code.

Think:

```text
Convention Plugin
        │
        ├── Plugins
        ├── Targets
        ├── Compiler
        ├── Testing
        └── Build Rules
```

while:

```text
Application Module
        │
        ├── Business Logic
        ├── UI
        └── Product Features
```

Keep those responsibilities separate.

---

# 13. A Simple KMP Convention

Conceptually, a convention plugin may configure:

```text
Kotlin Multiplatform
        │
        ├── Android Target
        ├── iOS Targets
        ├── Common Compiler Options
        └── Common Test Configuration
```

Then every shared library can simply apply:

```kotlin
plugins {
    id("my.kmp.library")
}
```

The module no longer needs to repeat the complete setup.

---

# 14. What Belongs in a Convention Plugin?

Good candidates include:

```text
Plugin application
Target configuration
Compiler configuration
Java compatibility
Kotlin options
Common test setup
Lint configuration
Code quality rules
Common dependency setup
Packaging conventions
```

The exact contents depend on the project's needs.

---

# 15. What Should Not Automatically Go Into a Convention Plugin?

Avoid putting highly module-specific behavior into a shared convention.

For example:

```text
Payment feature API
Product-specific resources
One module's special dependency
One screen's build task
```

A convention should represent behavior shared by a meaningful category of modules.

---

# 16. Convention Plugins Should Be Narrow

A useful rule:

> **A convention should configure one meaningful kind of module.**

For example:

```text
kmp.library
```

could mean:

```text
This module is a shared multiplatform library.
```

while:

```text
android.application
```

means:

```text
This module is an Android application.
```

This is easier to maintain than:

```text
everything
```

---

# 17. Avoid the "Mega Convention"

A dangerous pattern is:

```text
my.company.all
```

which applies:

```text
Android
KMP
Compose
Testing
Publishing
Lint
Signing
Benchmarking
```

to every module.

Now every module receives configuration it may not need.

This recreates the same problem in a different form.

Prefer several focused conventions.

---

# 18. Layered Conventions

You can build conventions in layers.

Conceptually:

```text
Base Kotlin Convention
        │
        ├── KMP Library Convention
        │
        └── Android Library Convention
```

For example:

```text
kotlin.common
      │
      ▼
kmp.library
      │
      ▼
feature module
```

This allows shared rules without creating one enormous plugin.

---

# 19. Convention Plugin and Version Catalog

The two can work together:

```text
                 libs.versions.toml
                         │
                         ▼
                  Plugin / Library
                     Versions
                         │
                         ▼
                 Convention Plugin
                         │
                         ▼
                  Module Configuration
```

For example:

```text
Catalog
→ Kotlin plugin alias/version

Convention
→ Apply Kotlin Multiplatform

Module
→ Apply company KMP convention
```

This creates a clean build architecture.

---

# 20. Centralizing KMP Target Configuration

Suppose all shared modules support:

```text
Android
iOS Device
iOS Simulator
```

The convention can establish that standard:

```text
KMP Convention
    │
    ├── Android
    ├── iOS ARM64
    └── iOS Simulator ARM64
```

Now a module does not need to repeat target declarations.

If the supported target model changes, the convention can be updated centrally.

---

# 21. But Don't Assume Every Module Needs Every Target

This is important.

A project may contain:

```text
shared-core
ios-only-support
android-only-feature
```

Not every module should necessarily receive the same targets.

Therefore, use conventions based on meaningful module categories.

For example:

```text
kmp.library
```

for true multiplatform modules.

Do not apply it to every module automatically.

---

# 22. Convention Plugin and Source Sets

A KMP convention can establish a standard source-set model:

```text
commonMain
commonTest
androidMain
iosMain
```

Then individual modules can focus on their actual code.

Conceptually:

```text
Convention
    │
    ├── Targets
    ├── Source Sets
    └── Common Configuration
            │
            ▼
        KMP Module
```

---

# 23. Convention Plugin and Dependencies

A convention can also provide dependencies that every module of a certain category needs.

For example:

```text
Every KMP library
   │
   ├── Kotlin test
   └── Common test tooling
```

However, dependency conventions should be used carefully.

Not every dependency belongs in every module.

---

# 24. Dependency Convention vs Dependency Catalog

These are different.

### Catalog

Defines:

```text
libs.kotlinx.coroutines.core
```

### Convention

May decide:

```text
All KMP libraries receive this standard test dependency.
```

Conceptually:

```text
Catalog
  │
  ▼
Dependency Alias
  │
  ▼
Convention
  │
  ▼
Selected Modules
```

---

# 25. Avoid Hidden Dependencies

If a convention automatically adds:

```text
Library X
```

to every module, developers may not realize why the library is available.

This can make dependency graphs harder to understand.

Use automatic dependency injection only when the dependency is genuinely part of the convention.

Otherwise, keep dependency declarations explicit.

---

# 26. Convention Plugins and Compiler Options

Compiler configuration is a strong candidate for centralization.

For example:

```text
Language level
API level
Compiler options
Warnings
Opt-ins
```

A convention can establish project-wide defaults.

Conceptually:

```text
Compiler Policy
      │
      ▼
Convention Plugin
      │
      ├── Module A
      ├── Module B
      └── Module C
```

This prevents modules from slowly drifting apart.

---

# 27. Compiler Configuration Drift

Without conventions:

```text
Module A → compiler setting X
Module B → compiler setting Y
Module C → compiler setting X
Module D → compiler setting Z
```

Now the repository has inconsistent build behavior.

With a convention:

```text
Central Compiler Policy
          │
          ▼
All applicable modules
```

The build becomes predictable.

---

# 28. Java Compatibility

KMP projects may also need consistent Java compatibility settings for JVM-based parts of the build.

A convention can centralize project policy where appropriate.

Conceptually:

```text
Java Toolchain Policy
        │
        ▼
Convention
        │
        ├── Shared Module
        ├── Android Module
        └── Other JVM Modules
```

The exact configuration should match the project's supported toolchain.

---

# 29. Toolchains and Conventions

Toolchain configuration is another area where centralization can help.

For example:

```text
Required JDK
```

can be part of the build policy.

But remember:

```text
Gradle Wrapper
JDK
Kotlin
Android tooling
Native tooling
```

are related but separate components.

A convention plugin should not attempt to hide those distinctions.

---

# 30. Testing Conventions

Testing is another strong candidate.

A KMP library may consistently require:

```text
commonTest
```

with:

```text
Kotlin test
```

and potentially other shared testing tools.

A convention can establish common test configuration.

Conceptually:

```text
KMP Test Convention
        │
        ├── commonTest
        ├── Android test setup
        └── Native test setup
```

The exact testing configuration depends on the project's requirements.

---

# 31. Platform Test Conventions

Platform tests may require different configuration.

For example:

```text
Android tests
iOS tests
```

should not automatically be treated as identical.

A convention can establish common rules while leaving platform-specific details to appropriate modules.

---

# 32. Compose Multiplatform Conventions

If a project uses Compose Multiplatform consistently, a dedicated convention can configure the shared Compose build model.

Conceptually:

```text
Compose Convention
        │
        ├── Compose plugin
        ├── Compiler integration
        ├── Common dependencies
        └── Platform configuration
```

This can prevent every UI module from repeating the same setup.

---

# 33. KMP Library Convention

A typical KMP library convention might conceptually provide:

```text
Kotlin Multiplatform plugin
        │
        ├── Android target
        ├── iOS targets
        ├── Common compiler settings
        ├── Common test setup
        └── Standard build rules
```

Then:

```text
feature-auth
```

might simply declare:

```kotlin
plugins {
    id("my.kmp.library")
}
```

The module build becomes intentionally small.

---

# 34. Android Library Convention

An Android-only library can use a different convention:

```text
Android Library Convention
        │
        ├── Android Library Plugin
        ├── Java/Kotlin configuration
        ├── Testing
        └── Code quality
```

This keeps Android-only modules from inheriting KMP configuration.

---

# 35. Application Convention

An Android application can have:

```text
Android Application Convention
        │
        ├── Application Plugin
        ├── Common Android settings
        ├── Build types
        └── Testing
```

Again, the convention represents a module category.

---

# 36. Convention Plugins and Module Types

A mature repository may therefore look like:

```text
                         Build Logic
                             │
          ┌──────────────────┼──────────────────┐
          ▼                  ▼                  ▼
     KMP Library        Android Library     Android App
     Convention          Convention         Convention
          │                  │                  │
          ▼                  ▼                  ▼
      Shared Modules      Libraries         Applications
```

This creates clear build boundaries.

---

# 37. Convention Plugin Implementation

A convention plugin is itself Gradle code.

Conceptually:

```kotlin
class KmpLibraryConventionPlugin : Plugin<Project> {

    override fun apply(project: Project) {
        // Configure the project
    }
}
```

The exact implementation depends on the Gradle and Kotlin APIs used by your project.

The key idea is:

```text
Plugin
   │
   ▼
Project configuration
```

---

# 38. Precompiled Script Plugins

Another common approach is a Kotlin-based precompiled script plugin.

Conceptually:

```text
build-logic/
    convention/
        src/main/kotlin/
            my.kmp.library.gradle.kts
```

The file can contain:

```kotlin
plugins {
    ...
}
```

and build configuration.

This often feels natural to Kotlin developers because the convention itself is written in Kotlin DSL.

---

# 39. Why Precompiled Script Plugins Are Popular

They provide:

```text
Type-safe Kotlin DSL
Readable configuration
Reusable build logic
Plugin-like application
```

A module can then use:

```kotlin
plugins {
    id("my.kmp.library")
}
```

instead of copying a large configuration block.

---

# 40. Convention Plugins Are Compiled Build Logic

This is an important mental shift.

Instead of:

```text
Gradle scripts copied everywhere
```

you have:

```text
Reusable build code
        │
        ▼
Compiled / interpreted by Gradle
        │
        ▼
Applied to modules
```

The build itself becomes modular.

---

# 41. Convention Plugin Dependencies

Convention plugins may need access to:

```text
Gradle APIs
Kotlin Gradle Plugin
Android Gradle Plugin
Other build tooling
```

These dependencies belong to the build logic environment.

Do not confuse them with:

```text
Application runtime dependencies
```

The two graphs are different.

---

# 42. Build Logic vs Application Dependencies

Think:

```text
Build Logic
   │
   ├── Gradle API
   ├── Kotlin Gradle Plugin
   └── Android Gradle Plugin

Application
   │
   ├── Coroutines
   ├── Networking
   └── Serialization
```

Build logic dependencies help configure the build.

Application dependencies are consumed by the software being built.

---

# 43. Convention Plugins and Gradle Lifecycle

Gradle configuration happens through a lifecycle.

A convention plugin participates in configuring a project.

Conceptually:

```text
Gradle Starts
     │
     ▼
Project Evaluated / Configured
     │
     ▼
Convention Applied
     │
     ▼
Plugins + Targets + Options
     │
     ▼
Tasks
     │
     ▼
Execution
```

Understanding this becomes important when writing advanced convention logic.

---

# 44. Avoid Configuration-Time Work

Convention plugins should configure the build efficiently.

Avoid doing expensive work during configuration such as:

```text
Network calls
Large file scans
External process execution
```

A convention should primarily describe the build.

This helps keep Gradle configuration fast and predictable.

---

# 45. Convention Plugins and Lazy Configuration

Gradle encourages lazy configuration.

Conceptually:

```text
Declare what should happen
        │
        ▼
Gradle determines when it is needed
```

This helps avoid unnecessary work.

When writing convention plugins, prefer Gradle APIs designed for lazy configuration rather than eagerly resolving everything.

---

# 46. Don't Configure Tasks Blindly

A convention plugin may need to configure tasks.

But avoid assumptions such as:

```text
Every module always has Task X.
```

Plugins can be applied conditionally.

A better approach is:

```text
If plugin exists
    │
    ▼
Configure relevant extension/task
```

This keeps conventions reusable.

---

# 47. Convention Plugin and Plugin Ordering

Some configuration only makes sense after another plugin has been applied.

For example:

```text
KMP Convention
      │
      ▼
Kotlin Multiplatform Plugin
      │
      ▼
KMP Extension Available
```

The convention must configure the plugin at the correct point in the Gradle lifecycle.

This is why convention plugins should be written against stable Gradle plugin APIs.

---

# 48. Convention Plugins and Extension Configuration

A convention may configure an extension exposed by another plugin.

Conceptually:

```text
Apply Plugin
     │
     ▼
Extension Created
     │
     ▼
Convention Configures Extension
```

For example:

```text
KMP plugin
    │
    ▼
Kotlin Multiplatform extension
    │
    ▼
Convention configures targets
```

The exact API depends on the Kotlin and Gradle versions.

---

# 49. Convention Plugin and Android Configuration

Similarly:

```text
Android plugin
      │
      ▼
Android extension
      │
      ▼
Convention
      │
      ▼
Compile / namespace / test configuration
```

A convention can centralize common Android library settings.

Avoid putting application-specific values into a generic Android library convention.

---

# 50. Convention Plugins and Namespaces

For Android modules, namespace configuration may be centralized where the project has a predictable naming convention.

But if namespaces vary by module, the convention should derive them safely or allow explicit configuration.

A convention should reduce duplication without hiding important module identity.

---

# 51. Convention Plugins and Build Types

Build types such as:

```text
debug
release
```

may have shared configuration.

A convention can establish common rules.

But application-specific signing, deployment, or product-flavor configuration should generally remain closer to the application module.

---

# 52. Convention Plugins and Product Flavors

Product flavors are often application-specific.

If every module receives flavor configuration automatically:

```text
Library A
Library B
Library C
```

may inherit settings they do not need.

Therefore:

```text
Common rule → convention
Product-specific rule → module/application
```

is a useful boundary.

---

# 53. Convention Plugins and Code Quality

Code quality configuration is often highly reusable.

For example:

```text
Static analysis
Formatting
Test coverage
Lint
```

can be applied consistently.

Conceptually:

```text
Quality Convention
        │
        ├── Shared Module
        ├── Android Library
        └── Feature Module
```

This turns quality expectations into build rules.

---

# 54. Convention Plugins and CI

A convention plugin can help ensure that CI and local builds use the same project rules.

Conceptually:

```text
Developer
   │
   ▼
Convention
   │
   ▼
Local Build

CI
   │
   ▼
Convention
   │
   ▼
CI Build
```

This reduces "works on my machine" configuration differences.

---

# 55. Convention Plugins and Reproducibility

Centralized build behavior improves reproducibility because modules are less likely to drift.

Without conventions:

```text
Module A → compiler setting X
Module B → compiler setting Y
```

With conventions:

```text
Central rule
     │
     ├── A
     └── B
```

The build is easier to reason about.

---

# 56. Convention Plugins and Build Performance

Convention plugins can improve maintenance, but poorly written conventions can also hurt build performance.

Avoid:

```text
Heavy configuration logic
Unnecessary task creation
Eager dependency resolution
Repeated file scanning
```

Good build logic should be:

```text
Small
Predictable
Lazy where appropriate
Focused
```

---

# 57. Convention Plugins Should Have Clear Ownership

As a project grows, build logic should have ownership.

For example:

```text
build-logic/
    ├── kmp
    ├── android
    ├── compose
    └── quality
```

Teams can then understand where to make a change.

A giant file containing all conventions becomes difficult to maintain.

---

# 58. Organizing Build Logic

A possible structure:

```text
build-logic/
│
└── convention/
    │
    ├── src/main/kotlin/
    │   ├── kmp.library.gradle.kts
    │   ├── android.library.gradle.kts
    │   ├── android.application.gradle.kts
    │   └── quality.gradle.kts
    │
    └── build.gradle.kts
```

The exact layout is flexible.

The goal is discoverability.

---

# 59. One Convention, One Responsibility

Prefer:

```text
kmp.library
```

over:

```text
everything
```

Prefer:

```text
quality
```

over:

```text
all-build-settings
```

Focused conventions are easier to compose.

---

# 60. Composition

A module may eventually use more than one convention.

Conceptually:

```text
feature-orders
    │
    ├── kmp.library
    └── quality
```

This means:

```text
KMP build rules
+
Quality rules
```

without creating a new mega-plugin for every combination.

---

# 61. Convention Composition and Boundaries

Composition works well when conventions are independent.

For example:

```text
KMP
   +
Quality
   +
Publishing
```

Each convention owns a separate concern.

Avoid conventions that secretly depend on dozens of unrelated conventions.

---

# 62. Convention Plugin API Stability

Build logic depends on Gradle APIs and other plugin APIs.

When upgrading:

```text
Gradle
Kotlin
AGP
```

your convention plugins may need changes.

This is normal.

The convention layer is part of your build infrastructure.

---

# 63. Build Logic Has Its Own Upgrade Cost

A project may upgrade:

```text
Kotlin
```

and discover:

```text
Application code → fine
Build logic → requires migration
```

This is one reason convention plugins should remain small and use stable APIs.

---

# 64. Convention Plugins and KMP Evolution

KMP tooling evolves quickly.

Target APIs, compiler options, and Gradle integration can change over time.

A convention plugin can isolate those changes.

Instead of updating:

```text
30 modules
```

you may update:

```text
1 convention
```

This is one of the strongest arguments for build conventions in large KMP repositories.

---

# 65. Example: Target Configuration Centralization

Without convention:

```text
shared/build.gradle.kts
feature-a/build.gradle.kts
feature-b/build.gradle.kts
```

all contain:

```kotlin
kotlin {
    androidTarget()
    iosArm64()
    iosSimulatorArm64()
}
```

With convention:

```text
kmp.library
    │
    ├── Android
    ├── iOS Device
    └── iOS Simulator
```

Modules simply apply:

```kotlin
plugins {
    id("my.kmp.library")
}
```

Now the target policy has one owner.

---

# 66. Example: Compiler Configuration

Without convention:

```text
Module A → compiler settings
Module B → compiler settings
Module C → compiler settings
```

With convention:

```text
Compiler Convention
       │
       ▼
All KMP Libraries
```

A change to the compiler policy becomes centralized.

---

# 67. Example: Testing Configuration

Without convention:

```text
Every module configures tests independently.
```

With convention:

```text
kmp.library
    │
    └── Standard test configuration
```

The result is a more consistent testing environment.

---

# 68. Example: Compose Convention

Suppose multiple modules use Compose Multiplatform.

Without convention:

```text
Compose plugin
Compose compiler setup
Compose dependencies
Platform configuration
```

is repeated.

With:

```text
compose.multiplatform
```

the shared UI build rules can be centralized.

This reduces copy-paste and makes Compose upgrades easier to manage.

---

# 69. Convention Plugins and Dependency Injection

If a project has a standard DI approach, a convention may configure the necessary plugin.

But be careful.

A convention should represent:

```text
All modules of this category require DI
```

not:

```text
Some module happens to use DI
```

Otherwise the convention becomes too broad.

---

# 70. Convention Plugins and Serialization

Similarly, if every KMP module uses Kotlin serialization:

```text
KMP Convention
      │
      └── Serialization plugin
```

may be reasonable.

But if only some modules serialize data:

```text
Module-specific plugin application
```

may be clearer.

The decision should follow actual module responsibilities.

---

# 71. Convention Plugins and Publishing

Libraries that are published may need:

```text
Maven publication
Metadata
Signing
Versioning
Repository configuration
```

A dedicated publishing convention can centralize common behavior.

Keep publishing concerns separate from:

```text
KMP compilation
```

when possible.

---

# 72. Convention Plugins and Internal Libraries

A company may have:

```text
internal KMP libraries
```

that all follow the same publishing rules.

A convention plugin can enforce:

```text
Group
Publishing repository
Metadata
Documentation
```

This makes internal library publishing consistent.

---

# 73. Don't Hide Important Publishing Information

Publishing conventions should still allow modules to specify:

```text
Artifact name
Description
Version
Special metadata
```

A convention should provide sensible defaults without making customization impossible.

---

# 74. Convention Plugins and Architecture

The build architecture can mirror the application architecture:

```text
Application Architecture
    │
    ├── Domain
    ├── Data
    └── Features

Build Architecture
    │
    ├── KMP Convention
    ├── Android Convention
    ├── Compose Convention
    └── Quality Convention
```

Both should be modular.

---

# 75. Convention Plugins as Guardrails

A convention can prevent accidental configuration drift.

For example:

```text
All KMP libraries
    │
    └── Must use approved compiler configuration
```

Instead of relying only on documentation:

```text
Build system enforces the convention.
```

This is a powerful property.

---

# 76. Conventions and Team Scaling

A small team may remember:

```text
How KMP modules are configured.
```

A large team cannot rely on memory.

Convention plugins encode the team's decisions:

```text
Human Knowledge
      │
      ▼
Build Convention
      │
      ▼
Repeatable Configuration
```

New developers benefit immediately.

---

# 77. Convention Plugins and Onboarding

A new developer can inspect:

```text
plugins {
    id("company.kmp.library")
}
```

and know:

```text
This module follows the standard KMP library configuration.
```

They don't need to understand every Gradle detail before contributing.

The convention becomes executable documentation.

---

# 78. Convention Plugins and Code Review

Instead of reviewing repeated Gradle configuration in every module, reviewers can review:

```text
Convention change
```

once.

Then module changes can remain focused.

For example:

```text
Feature PR
   │
   ├── Kotlin source
   ├── Tests
   └── Small build change
```

rather than:

```text
Large repeated Gradle configuration
```

---

# 79. Convention Plugin Testing

Build logic is code.

That means it should be tested where practical.

Possible validation includes:

```text
Plugin applies successfully
Expected targets exist
Expected compiler settings are applied
Expected dependencies are present
Unexpected configuration is not applied
```

The exact testing approach depends on the complexity of the build logic.

---

# 80. Don't Build Abstraction Before You Need It

Convention Plugins are useful, but they should not be introduced just to make a two-module project look sophisticated.

A reasonable progression is:

```text
Small project
   │
   ▼
Simple Gradle configuration

Repeated configuration
   │
   ▼
Extract convention

Growing repository
   │
   ▼
Organize build logic
```

Abstraction should follow repetition and complexity.

---

# 81. Common Mistake: Convention Too Early

If two modules share only:

```text
three lines
```

a convention plugin may add more complexity than it removes.

Wait until the shared behavior is:

```text
Stable
Repeated
Meaningful
```

Then extract it.

---

# 82. Common Mistake: Convention Too Late

The opposite problem is allowing:

```text
20 modules
```

to contain:

```text
copy-pasted build logic
```

for years.

By then, every module may have subtle differences.

Refactoring becomes harder.

The right time is when repeated configuration becomes a maintenance concern.

---

# 83. Common Mistake: Hidden Configuration

A module should remain understandable.

If applying:

```kotlin
id("company.kmp.library")
```

changes:

```text
Targets
Dependencies
Compiler
Testing
Publishing
Signing
Packaging
```

developers may not know what happened.

Keep conventions focused and document their purpose through naming.

---

# 84. Common Mistake: Mega Plugins

Avoid:

```text
company.everything
```

which does all possible build configuration.

Instead:

```text
company.kmp.library
company.android.library
company.compose.multiplatform
company.quality
company.publishing
```

This creates composable build logic.

---

# 85. Common Mistake: Convention-Specific Business Logic

Never put:

```text
Product-specific business rules
```

into build conventions.

For example:

```text
if product == "X"
```

should not become a giant Gradle convention unless it represents a legitimate build concern.

Build logic should remain about the build.

---

# 86. Common Mistake: Excessive Cross-Module Knowledge

A convention should not need to know:

```text
Every module's internal source layout
Every feature's business rules
Every module's special case
```

If it does, the convention is probably too broad.

---

# 87. Common Mistake: Ignoring Build Performance

A convention that executes expensive operations for every module can slow the entire repository.

Remember:

```text
50 modules
×
Expensive configuration
=
Slow build
```

Keep conventions lightweight.

---

# 88. Common Mistake: Treating Build Logic as Untestable

Build logic can fail just like application code.

For example:

```text
Plugin API changed
Target declaration changed
Extension API changed
Task name changed
```

Build validation should therefore be part of CI.

---

# 89. Common Mistake: Hardcoding Versions in Conventions

Avoid:

```kotlin
someDependencyVersion = "1.2.3"
```

inside a convention if the project already uses a Version Catalog.

Prefer:

```text
Version Catalog
        │
        ▼
Convention Plugin
```

This keeps version ownership centralized.

---

# 90. Common Mistake: Mixing Dependency and Build Policy

A convention may use:

```text
libs.some.library
```

from the Version Catalog.

But the catalog should remain responsible for:

```text
What version?
What coordinate?
```

while the convention handles:

```text
When should this dependency be applied?
```

This separation keeps responsibilities clear.

---

# 91. Convention Plugin Review Checklist

Before creating a convention, ask:

```text
[ ] Is this configuration genuinely repeated?
[ ] Does it represent a meaningful module category?
[ ] Is the behavior stable?
[ ] Can the convention have one clear responsibility?
[ ] Can it be composed with other conventions?
[ ] Does it avoid unnecessary dependencies?
[ ] Does it avoid expensive configuration?
[ ] Does it use the Version Catalog for versions?
[ ] Is the name clear?
[ ] Can developers understand what it does?
```

---

# 92. Convention Plugin Debugging

When a module behaves unexpectedly:

```text
Module
  │
  ▼
Applied Plugins
  │
  ▼
Convention
  │
  ▼
Configured Extensions
  │
  ▼
Generated Tasks
```

Ask:

```text
Which convention was applied?
What did it configure?
Which plugin did it depend on?
Which target/source set did it create?
```

This is much more effective than changing Gradle settings randomly.

---

# 93. A Practical KMP Build Architecture

A mature repository can look like:

```text
project/
│
├── gradle/
│   └── libs.versions.toml
│
├── build-logic/
│   └── convention/
│       └── src/main/kotlin/
│           ├── kmp.library.gradle.kts
│           ├── android.library.gradle.kts
│           ├── android.application.gradle.kts
│           ├── compose.multiplatform.gradle.kts
│           └── quality.gradle.kts
│
├── shared/
├── core/
├── feature-auth/
├── feature-orders/
└── androidApp/
```

The responsibilities are clear:

```text
Version Catalog
→ Versions and dependency vocabulary

Convention Plugins
→ Build behavior

Modules
→ Product behavior
```

---

# 94. The Relationship Between All Three

The Gradle architecture can now be visualized as:

```text
                    Gradle Build
                         │
          ┌──────────────┼──────────────┐
          ▼              ▼              ▼
 Version Catalog   Convention Plugins   Modules
          │              │              │
          │              │              │
       What?           How?           Why?
          │              │              │
          └──────────────┼──────────────┘
                         ▼
                  Consistent Build
```

This is one of the most useful mental models for large KMP repositories.

---

# 95. Convention Plugins and Targets

Targets are a particularly good example of convention value.

Without conventions:

```text
Module A → Android + iOS
Module B → Android + iOS
Module C → Android + iOS
Module D → Android + iOS
```

With a KMP convention:

```text
KMP Library Convention
          │
          ├── Android
          ├── iOS Device
          └── iOS Simulator
```

The platform policy has a single owner.

---

# 96. Convention Plugins and Dependencies

A convention can also establish standard dependencies where justified.

For example:

```text
KMP library convention
       │
       ├── Standard test dependency
       └── Standard compiler configuration
```

But application-specific dependencies should remain explicit.

The convention should not become a hidden dependency injector.

---

# 97. Convention Plugins and Gradle Tasks

A convention may register or configure tasks related to:

```text
Testing
Code generation
Validation
Publishing
Quality checks
```

But tasks should be created only where they provide meaningful value.

Avoid creating dozens of tasks for every module simply because the plugin can.

---

# 98. Convention Plugins and Build Validation

A useful quality convention might establish:

```text
Unit tests
Static analysis
Formatting
Coverage checks
```

Then CI can rely on consistent tasks across modules.

Conceptually:

```text
Quality Convention
       │
       ▼
Standard Verification
       │
       ├── Module A
       ├── Module B
       └── Module C
```

This makes repository-wide automation easier.

---

# 99. Convention Plugins and Release Engineering

For publishable KMP libraries, conventions can centralize:

```text
Publishing metadata
Repository configuration
Signing policy
Documentation generation
```

Again, separate publishing from general KMP compilation where practical.

This keeps the build logic modular.

---

# 100. The Deeper Mental Model

Think of Convention Plugins as the layer between:

```text
Gradle capabilities
```

and:

```text
Project modules
```

The relationship becomes:

```text
                 Gradle / Plugins
                        │
                        ▼
                Project Conventions
                        │
             ┌──────────┼──────────┐
             ▼          ▼          ▼
           KMP       Android     Quality
         Convention Convention Convention
             │          │          │
             └──────────┼──────────┘
                        ▼
                      Modules
```

This is where project-specific build knowledge lives.

---

# Chapter Takeaways

> [!TIP]
> **Convention Plugins turn repeated Gradle configuration into reusable, explicit build standards. In a growing KMP project, they provide a central place to define how different categories of modules should be built without forcing every module to copy the same configuration.**

Remember:

1. A Convention Plugin is reusable Gradle build logic applied to multiple modules.
2. Convention Plugins solve build-configuration duplication.
3. A Version Catalog answers "what do we use?" while a Convention Plugin answers "how should it be configured?"
4. Convention Plugins are especially valuable in multi-module KMP repositories.
5. Common candidates include target configuration, compiler options, testing, Android configuration, quality rules, and publishing.
6. A convention should represent a meaningful category of modules.
7. Focused conventions are generally easier to maintain than one mega-plugin.
8. Explicitly applying a convention makes module build behavior easier to understand.
9. Convention Plugins can centralize KMP target configuration.
10. Not every module needs the same KMP targets.
11. Source-set and target configuration can be standardized through conventions.
12. Dependencies can be used from conventions, but hidden dependency injection should be avoided.
13. Version Catalogs and Convention Plugins complement each other.
14. Versions should generally remain owned by the Version Catalog rather than being hardcoded inside conventions.
15. Convention Plugins are build code and should be treated as maintainable software.
16. Precompiled script plugins are one common way to implement convention logic using Kotlin DSL.
17. Convention Plugins may depend on Gradle, Kotlin, Android, and other build-tool APIs.
18. Build-logic dependencies are separate from application runtime dependencies.
19. Convention Plugins should prefer focused, lazy, and predictable configuration.
20. Expensive configuration-time work should be avoided.
21. Convention Plugins can improve consistency across local development and CI.
22. Build logic itself has an upgrade cost when Gradle, Kotlin, or Android tooling changes.
23. Convention Plugins should not contain application business logic.
24. Product-specific configuration should remain in the relevant module when it is not a shared build concern.
25. Convention Plugins can act as executable documentation for project build standards.
26. Good conventions reduce copy-paste while keeping important module differences visible.
27. A convention should not hide so much behavior that developers cannot understand what applying it does.
28. Build logic should be tested and validated where its complexity justifies it.
29. Convention Plugins should be introduced when repeated build behavior becomes stable and meaningful, not merely for abstraction's sake.
30. The central principle is: **Version Catalogs centralize dependency vocabulary, Convention Plugins centralize build behavior, and modules remain responsible for product behavior.**

---

# Final Mental Model

When you see:

```kotlin
plugins {
    id("company.kmp.library")
}
```

don't think:

```text
"Just another Gradle plugin."
```

Think:

```text
                     Project Build Policy
                             │
                             ▼
                    company.kmp.library
                             │
             ┌───────────────┼───────────────┐
             ▼               ▼               ▼
          Plugins          Targets         Compiler
             │               │               │
             └───────────────┼───────────────┘
                             ▼
                          Source Sets
                             │
                             ▼
                          Testing
                             │
                             ▼
                         KMP Module
```

And remember:

> **A Convention Plugin is where a project turns repeated Gradle configuration into an intentional engineering rule. Version Catalogs tell the build what dependencies and plugins the project uses; Convention Plugins tell modules how those tools should be configured. When these responsibilities are separated cleanly, a large KMP repository becomes easier to scale, upgrade, review, and understand—without hiding the platform differences that make multiplatform development valuable in the first place.**
