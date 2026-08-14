# Chapter 6 — Gradle for KMP

## Part 5 — Version Catalog

> **A growing KMP project does not only have more source code. It also has more versions to manage: Kotlin, Gradle plugins, Android tooling, libraries, testing frameworks, and platform-specific dependencies. Version Catalogs provide a central place to describe those versions and dependencies in a structured way.**

As a KMP project grows, Gradle files can slowly become repetitive.

You may find the same dependency versions repeated across:

```text
shared
androidApp
feature modules
data modules
test modules
```

For example:

```kotlin
implementation("some.library:core:1.2.3")
```

and somewhere else:

```kotlin
implementation("some.library:core:1.2.3")
```

and again:

```kotlin
implementation("some.library:core:1.2.3")
```

At first this seems harmless.

But when the project contains dozens of modules, repeated versions become a maintenance problem.

This is where **Gradle Version Catalogs** become useful.

---

# 1. What Is a Version Catalog?

A Version Catalog is a centralized declaration of:

```text
Dependency versions
Dependency coordinates
Plugin aliases
Library aliases
Bundles
```

The conventional file is:

```text
gradle/libs.versions.toml
```

A simplified structure looks like:

```text
project/
│
├── gradle/
│   └── libs.versions.toml
│
├── shared/
│   └── build.gradle.kts
│
└── androidApp/
    └── build.gradle.kts
```

The catalog becomes a shared vocabulary for the build.

---

# 2. Why Does KMP Need Version Catalogs?

KMP projects commonly involve multiple layers of tooling:

```text
Gradle
Kotlin
KMP
Android Gradle Plugin
Android SDK
iOS / Xcode
Libraries
Testing
```

And a repository can contain many modules.

Without centralized dependency management:

```text
Module A
   └── Kotlin version

Module B
   └── Kotlin version

Module C
   └── Kotlin version
```

With a version catalog:

```text
              libs.versions.toml
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
       Module A     Module B     Module C
```

The build becomes easier to reason about.

---

# 3. The `libs.versions.toml` File

A Version Catalog is commonly stored at:

```text
gradle/libs.versions.toml
```

A simplified example:

```toml
[versions]
kotlin = "2.x.x"
coroutines = "x.x.x"

[libraries]
kotlin-coroutines = { module = "org.jetbrains.kotlinx:kotlinx-coroutines-core", version.ref = "coroutines" }
```

The exact versions should come from the versions intentionally selected for your project.

The important structure is:

```text
[versions]
[libraries]
[plugins]
```

---

# 4. Versions Section

The `[versions]` section defines reusable version values.

For example:

```toml
[versions]
kotlin = "2.x.x"
coroutines = "x.x.x"
serialization = "x.x.x"
```

Conceptually:

```text
versions
   │
   ├── kotlin
   ├── coroutines
   └── serialization
```

The goal is to avoid repeating the same version in multiple dependency declarations.

---

# 5. Libraries Section

The `[libraries]` section defines dependency aliases.

For example:

```toml
[libraries]
coroutines-core = {
    module = "org.jetbrains.kotlinx:kotlinx-coroutines-core",
    version.ref = "coroutines"
}
```

Then a module can reference the alias rather than repeating the full Maven coordinate.

Conceptually:

```text
libs.versions.toml
        │
        ▼
coroutines-core
        │
        ▼
Gradle Module
```

---

# 6. Why Aliases Help

Without an alias:

```kotlin
implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core:...")
```

With an alias:

```kotlin
implementation(libs.coroutines.core)
```

The build file becomes easier to read.

Instead of asking:

> "What exact Maven coordinate is this?"

you can read:

```text
coroutines.core
```

and understand the dependency's purpose.

---

# 7. Version Catalog Is Not a Dependency Repository

A Version Catalog does **not** contain the actual library binaries.

It contains metadata describing dependencies.

Think:

```text
Version Catalog
       │
       ▼
Coordinates + Versions + Aliases
       │
       ▼
Gradle Resolution
       │
       ▼
Repository
       │
       ▼
Artifact
```

The catalog tells Gradle what dependency you want.

The repository provides the actual artifact.

---

# 8. Version Catalog vs Repository

These concepts should not be confused.

### Version Catalog

Answers:

```text
Which dependency?
Which version?
Which alias?
```

### Repository

Answers:

```text
Where can Gradle obtain the dependency?
```

Conceptually:

```text
Catalog
  │
  ▼
Dependency Coordinates
  │
  ▼
Repository
  │
  ▼
Artifact
```

---

# 9. Plugin Aliases

Version Catalogs can also define plugins.

For example:

```toml
[plugins]
kotlin-multiplatform = {
    id = "org.jetbrains.kotlin.multiplatform",
    version.ref = "kotlin"
}
```

Then a module can use an alias such as:

```kotlin
plugins {
    alias(libs.plugins.kotlin.multiplatform)
}
```

The exact alias name depends on the catalog.

The important concept is:

```text
Plugin
   │
   ▼
Central Alias
   │
   ▼
Module
```

---

# 10. Why Plugin Catalogs Matter in KMP

KMP projects often have several modules.

For example:

```text
shared
feature-auth
feature-products
feature-orders
```

Many of these modules may use the same Kotlin or KMP plugin version.

Instead of repeating plugin coordinates:

```text
Module A → Kotlin version
Module B → Kotlin version
Module C → Kotlin version
```

you can centralize them:

```text
libs.versions.toml
       │
       ▼
Kotlin Plugin
       │
       ├── shared
       ├── feature-auth
       └── feature-products
```

This reduces version drift.

---

# 11. Version Catalog and KMP Plugin

A typical concept might look like:

```toml
[versions]
kotlin = "2.x.x"

[plugins]
kotlin-multiplatform = {
    id = "org.jetbrains.kotlin.multiplatform",
    version.ref = "kotlin"
}
```

Then:

```kotlin
plugins {
    alias(libs.plugins.kotlin.multiplatform)
}
```

The catalog becomes the central declaration point.

Always use versions compatible with the rest of your project's Gradle and Android tooling.

---

# 12. Android Plugin Aliases

A project may also centralize Android plugin versions.

Conceptually:

```toml
[versions]
agp = "x.x.x"

[plugins]
android-application = {
    id = "com.android.application",
    version.ref = "agp"
}
```

Then:

```kotlin
plugins {
    alias(libs.plugins.android.application)
}
```

The exact version should be selected according to the project's supported toolchain.

---

# 13. The Central Toolchain Picture

A KMP project can therefore have:

```text
                 libs.versions.toml
                         │
          ┌──────────────┼──────────────┐
          ▼              ▼              ▼
       Kotlin           AGP         Libraries
          │              │              │
          ▼              ▼              ▼
       KMP Build      Android       Shared Code
```

This gives the repository a central place to describe important build dependencies.

---

# 14. Version Catalog Does Not Solve Compatibility

A Version Catalog can tell every module to use:

```text
Kotlin X
```

but it cannot guarantee that:

```text
Kotlin X
+
Gradle Y
+
AGP Z
+
Library A
```

are compatible.

Compatibility remains an engineering responsibility.

The catalog provides consistency.

It does not provide automatic correctness.

---

# 15. Consistency vs Compatibility

These are different concepts.

### Consistency

```text
All modules use Kotlin X.
```

### Compatibility

```text
Kotlin X works with the selected Gradle,
Android tooling, libraries, and targets.
```

A Version Catalog helps strongly with the first.

You still need to validate the second.

---

# 16. Centralizing Versions

Suppose:

```toml
[versions]
coroutines = "x.x.x"
```

Then multiple libraries can reference:

```toml
version.ref = "coroutines"
```

Conceptually:

```text
                 coroutines version
                         │
             ┌───────────┴───────────┐
             ▼                       ▼
       coroutines-core         coroutines-test
```

If the project upgrades the version, the central version can be updated rather than editing every dependency declaration individually.

---

# 17. One Version, Multiple Artifacts

Many ecosystems publish multiple artifacts for the same library family.

For example:

```text
library-core
library-test
library-jvm
```

A version catalog can express:

```text
One Version
    │
    ├── Artifact A
    ├── Artifact B
    └── Artifact C
```

This helps keep related artifacts aligned.

---

# 18. Dependency Aliases Improve Readability

Compare:

```kotlin
implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core:...")
```

with:

```kotlin
implementation(libs.kotlinx.coroutines.core)
```

The second form communicates intent more clearly when the catalog is well organized.

The module says:

```text
I depend on Kotlin Coroutines.
```

The catalog handles:

```text
Which artifact?
Which version?
```

---

# 19. Version Catalog Naming Matters

Good aliases should be predictable.

For example:

```text
libs.kotlinx.coroutines.core
libs.kotlinx.serialization.json
libs.androidx.lifecycle.runtime
```

Poor naming can make the catalog difficult to use:

```text
libs.lib1
libs.networkNew
libs.temp
```

The catalog is part of the build's developer experience.

Treat naming as an API.

---

# 20. Naming Should Reflect the Ecosystem

A useful pattern is:

```text
vendor
    │
    └── library
          │
          └── artifact
```

For example:

```text
libs.androidx.lifecycle.runtime
```

or:

```text
libs.kotlinx.coroutines.core
```

The exact naming convention is a team choice.

The goal is consistency.

---

# 21. Bundles

Version Catalogs can also group related libraries using bundles.

Conceptually:

```toml
[bundles]
networking = [
    "network-core",
    "network-serialization"
]
```

Then a module can reference the bundle.

The result is:

```text
Networking Bundle
       │
       ├── Core
       └── Serialization
```

Bundles can reduce repetitive dependency declarations.

Use them when the libraries genuinely belong together.

---

# 22. Don't Overuse Bundles

A bundle should represent a meaningful group.

Avoid creating bundles such as:

```text
everything
```

containing dozens of unrelated libraries.

Otherwise:

```text
implementation(libs.bundles.everything)
```

hides what the module actually depends on.

A good catalog improves visibility rather than reducing it.

---

# 23. KMP Source Sets Still Control Scope

This is extremely important.

Suppose the catalog defines:

```text
libs.android.someLibrary
```

That does not mean the dependency automatically becomes available everywhere.

You still decide:

```kotlin
androidMain.dependencies {
    implementation(libs.android.someLibrary)
}
```

The catalog defines the dependency.

The source set defines its scope.

Think:

```text
Version Catalog
      │
      ▼
What dependency?
      │
      ▼
Source Set
      │
      ▼
Where used?
```

---

# 24. Version Catalog Does Not Make Android Libraries Multiplatform

This is one of the most important warnings.

Suppose:

```toml
android-library = {
    module = "example:android-library",
    version = "..."
}
```

You can reference:

```kotlin
libs.android.library
```

from the build file.

But that does not make the library compatible with:

```text
iOS
```

The catalog is only an aliasing mechanism.

Target compatibility remains unchanged.

---

# 25. CommonMain Still Needs Compatible Dependencies

Imagine:

```toml
[libraries]
some-library = {
    module = "example:some-library",
    version.ref = "some"
}
```

Then:

```kotlin
commonMain.dependencies {
    implementation(libs.some.library)
}
```

This is valid only if the dependency supports the targets consuming `commonMain`.

The alias does not bypass target compatibility.

---

# 26. Version Catalog and Platform Dependencies

A clean structure can be:

```text
libs.versions.toml
        │
        ├── Common Dependencies
        │
        ├── Android Dependencies
        │
        └── Test Dependencies
```

Then source sets decide usage:

```text
commonMain  → common libraries
androidMain → Android libraries
iosMain     → iOS libraries
commonTest  → test libraries
```

This creates a clear separation between:

```text
Definition
```

and:

```text
Usage
```

---

# 27. Version Catalog and Architecture

A Version Catalog should not decide your architecture.

For example, don't say:

> "This library is already in the catalog, so every module should use it."

Instead:

```text
Module responsibility
        │
        ▼
Architecture
        │
        ▼
Required capability
        │
        ▼
Catalog dependency
```

The catalog is an available vocabulary, not a mandatory dependency list.

---

# 28. Version Catalog as Build Vocabulary

A good catalog creates a common vocabulary:

```text
libs.kotlinx.coroutines.core
libs.kotlinx.serialization.json
libs.androidx.lifecycle.runtime
```

Developers across the repository can use the same names.

This reduces ambiguity.

Instead of:

```text
Which coroutine version are we using?
```

the repository has one defined answer.

---

# 29. Version Catalog and Multi-Module KMP

Imagine:

```text
Project
│
├── androidApp
├── shared
├── core
├── feature-auth
├── feature-products
└── feature-orders
```

The catalog can provide common definitions:

```text
                    libs.versions.toml
                            │
          ┌─────────────────┼─────────────────┐
          ▼                 ▼                 ▼
       Plugins          Libraries          Versions
          │                 │                 │
          └─────────────────┼─────────────────┘
                            ▼
                       All Modules
```

This is especially useful as the repository grows.

---

# 30. Version Drift Without a Catalog

Without centralization:

```text
shared
  └── library 1.4

feature-a
  └── library 1.4

feature-b
  └── library 1.3

feature-c
  └── library 1.5
```

Now the repository contains uncertainty.

With a catalog:

```text
library = 1.4
     │
     ├── shared
     ├── feature-a
     ├── feature-b
     └── feature-c
```

All modules can use the same declared version where appropriate.

---

# 31. Centralization Reduces Upgrade Effort

Without a catalog:

```text
Search repository
      │
      ▼
Find dependency
      │
      ▼
Edit multiple files
      │
      ▼
Hope nothing was missed
```

With a catalog:

```text
Update catalog
      │
      ▼
Build
      │
      ▼
Test
```

This does not remove upgrade risk.

It reduces the number of places where the version is manually maintained.

---

# 32. Version Catalog and Dependency Updates

Suppose a dependency moves from:

```text
1.x
```

to:

```text
2.x
```

The catalog provides one obvious location for the version change.

But after changing it, test:

```text
Android
iOS
Common tests
Platform tests
```

The catalog simplifies the change.

It does not replace testing.

---

# 33. Version Catalog and Plugin Upgrades

The same principle applies to plugins.

For example:

```text
Kotlin
Android Gradle Plugin
Serialization Plugin
```

can have centrally managed versions.

A plugin upgrade can still affect:

```text
Gradle compatibility
Compiler behavior
KMP target configuration
Native builds
Android builds
```

Centralization makes the change easier to control, not automatically safer.

---

# 34. Toolchain Versions Deserve Extra Attention

A KMP project may have several critical version relationships:

```text
Gradle
   │
   ├── Kotlin
   │      │
   │      └── KMP
   │
   └── Android Gradle Plugin
          │
          └── Android build tooling
```

A Version Catalog can centralize some of these declarations.

But the project still needs a validated compatibility matrix.

---

# 35. Version Catalog and Gradle Wrapper

One common misunderstanding is that the Version Catalog manages the Gradle wrapper version.

It does not.

The Gradle wrapper is configured separately, typically through:

```text
gradle/wrapper/gradle-wrapper.properties
```

Think:

```text
Gradle Wrapper
      │
      ▼
Gradle Runtime

Version Catalog
      │
      ▼
Dependencies + Plugins
```

They solve related but different problems.

---

# 36. Version Catalog and Kotlin Version

The Kotlin plugin version can be centralized in a Version Catalog.

But the Kotlin version participates in the overall toolchain.

Therefore, changing:

```text
Kotlin version
```

should trigger compatibility validation for:

```text
Gradle
KMP
Android tooling
Libraries
Native targets
```

---

# 37. Version Catalog and Android SDK

Android SDK values are not the same thing as dependency versions.

A project may define:

```text
compileSdk
minSdk
targetSdk
```

through Android configuration or shared build conventions.

These can be centralized using different Gradle mechanisms.

The important distinction is:

```text
Version Catalog
→ Dependency/plugin metadata

Android configuration
→ Android build settings
```

Do not force every build setting into the catalog just because centralization is possible.

---

# 38. Catalog vs Convention Plugin

These two are often used together.

### Version Catalog

Good for:

```text
Versions
Libraries
Plugins
Aliases
Bundles
```

### Convention Plugin

Good for:

```text
Build rules
Compiler options
Common module configuration
Testing configuration
Repositories or conventions
```

Conceptually:

```text
                 Build Architecture
                        │
              ┌─────────┴─────────┐
              ▼                   ▼
       Version Catalog       Convention Plugins
              │                   │
       What versions?       How should modules
       Which libraries?      be configured?
```

They complement each other.

---

# 39. A Strong KMP Build Setup

A mature KMP repository may look like:

```text
Project
│
├── gradle/
│   └── libs.versions.toml
│
├── build-logic/
│   └── convention plugins
│
├── shared/
├── androidApp/
├── feature-auth/
└── feature-products/
```

The responsibilities become:

```text
Version Catalog
→ Dependency and plugin vocabulary

Convention Plugins
→ Shared build conventions

Module build files
→ Module-specific configuration
```

This separation keeps Gradle manageable.

---

# 40. Version Catalog Is Not a Magic Dependency Container

Avoid turning the catalog into:

```text
Every library we have ever used
```

A large catalog can become difficult to maintain.

Remove obsolete dependencies and keep names meaningful.

The catalog should represent supported project dependencies, not historical leftovers.

---

# 41. Catalog Organization

A practical catalog can group versions logically:

```toml
[versions]
kotlin = "..."
agp = "..."
coroutines = "..."
serialization = "..."
```

Then libraries:

```toml
[libraries]
...
```

and plugins:

```toml
[plugins]
...
```

Keep related entries together.

For very large projects, consistent naming becomes increasingly important.

---

# 42. Naming Conflicts

Aliases eventually become a namespace.

For example:

```text
libs.network.core
libs.network.logging
libs.network.test
```

Good naming makes it easy to discover dependencies.

Poor naming creates confusion:

```text
libs.network1
libs.network2
libs.networkNew
```

Treat aliases like code identifiers.

---

# 43. Version References

One useful feature is referencing a version from `[versions]`.

Conceptually:

```toml
[versions]
serialization = "x.x.x"

[libraries]
serialization-json = {
    module = "org.jetbrains.kotlinx:kotlinx-serialization-json",
    version.ref = "serialization"
}
```

Now:

```text
One version
     │
     └── Multiple related artifacts
```

This helps keep a library family aligned.

---

# 44. Catalog Without Version References

It is also possible to declare dependency versions directly in library definitions.

Conceptually:

```toml
[libraries]
example = {
    module = "group:artifact",
    version = "x.x.x"
}
```

This can be appropriate for dependencies that do not share a version with anything else.

The choice depends on how you want to organize the catalog.

---

# 45. Version Catalog and Library Families

For a library ecosystem with many artifacts:

```text
Library
├── core
├── runtime
├── testing
└── integration
```

centralizing the family version can help:

```text
library-version
      │
      ├── core
      ├── runtime
      ├── testing
      └── integration
```

This is especially useful when the publisher expects those artifacts to remain aligned.

---

# 46. Don't Assume Every Artifact Shares a Version

Some ecosystems use independent versioning.

For such libraries:

```text
Artifact A → 1.2
Artifact B → 4.7
```

Forcing them into one shared version can be misleading.

Use version references where the dependency relationship actually makes sense.

---

# 47. Catalog Bundles and KMP

Bundles can be useful for shared test dependencies.

Conceptually:

```toml
[bundles]
common-testing = [
    "kotlin-test",
    "coroutines-test"
]
```

Then:

```text
commonTest
    │
    ▼
common-testing bundle
```

The exact libraries should be selected based on the project's test strategy.

---

# 48. Don't Hide Platform Differences With Bundles

Suppose a bundle contains:

```text
Android-only library
iOS-compatible library
```

Using the bundle in:

```text
commonMain
```

would be a design problem.

Bundles simplify declarations.

They do not remove target compatibility requirements.

---

# 49. Catalog and Source Sets Work Together

The relationship is:

```text
                  Version Catalog
                        │
               Dependency Alias
                        │
              ┌─────────┼─────────┐
              ▼         ▼         ▼
          commonMain androidMain iosMain
              │         │         │
              ▼         ▼         ▼
           Common    Android      iOS
          Dependency Dependency Dependency
```

This is the model to remember.

---

# 50. Catalog and Target Compatibility

Before placing:

```kotlin
implementation(libs.some.library)
```

in:

```text
commonMain
```

ask:

```text
Does this dependency support every configured target?
```

If not:

```text
Move it to the appropriate platform source set
```

or:

```text
Introduce an abstraction
```

The catalog does not change this rule.

---

# 51. Version Catalog as a Contract

A well-maintained catalog can communicate:

```text
These are the libraries and versions our project intentionally uses.
```

That makes it a kind of build contract.

Developers can inspect:

```text
gradle/libs.versions.toml
```

to understand a large part of the project's technology stack.

---

# 52. Catalog Review During Code Review

When a dependency is added, a reviewer can check:

```text
Did the version go into the catalog?
Is the alias named correctly?
Is the dependency placed in the correct source set?
Does it support all required targets?
Is the version appropriate?
```

This makes dependency changes easier to review.

---

# 53. Dependency Addition Workflow

A clean workflow is:

```text
Need a library
      │
      ▼
Evaluate target support
      │
      ▼
Select version
      │
      ▼
Add version to catalog
      │
      ▼
Add library alias
      │
      ▼
Use alias in source set
      │
      ▼
Build all relevant targets
      │
      ▼
Run tests
```

This creates a repeatable process.

---

# 54. Plugin Addition Workflow

Similarly:

```text
Need a plugin
      │
      ▼
Verify compatibility
      │
      ▼
Add plugin version
      │
      ▼
Add plugin alias
      │
      ▼
Apply alias in module
      │
      ▼
Build
      │
      ▼
Test
```

This keeps plugin management consistent.

---

# 55. Version Catalog and CI

CI benefits from centralized versions because every environment reads the same catalog.

Conceptually:

```text
Developer Machine ─┐
                    │
CI Machine ─────────┼──► libs.versions.toml
                    │
Release Build ──────┘
```

This reduces the chance that different modules silently use different dependency versions.

It does not eliminate environment differences, but it reduces one important source of inconsistency.

---

# 56. Version Catalog and Reproducibility

A reproducible build depends on many things:

```text
Gradle version
Plugin versions
Dependency versions
Repositories
JDK
SDKs
Build environment
```

The Version Catalog helps pin part of this information.

It is one piece of the reproducibility puzzle, not the complete solution.

---

# 57. Avoid Dynamic Versions

Be cautious with dependency versions such as:

```text
latest.release
```

or other dynamic version expressions.

They can make builds change without a source-code change.

Prefer explicit versions that can be reviewed and reproduced.

The catalog is a natural place to keep those versions visible.

---

# 58. Version Catalog and Dependency Locking

For larger projects, Gradle dependency locking can provide an additional layer of reproducibility.

Conceptually:

```text
Version Catalog
      │
      ▼
Intended Versions

Dependency Lock
      │
      ▼
Resolved Versions
```

These solve different problems.

A catalog says:

```text
What versions do we intend to use?
```

Locking can help record:

```text
What versions were actually resolved?
```

Use the mechanism appropriate to your project's build governance.

---

# 59. Catalog and Transitive Dependencies

The catalog typically defines direct dependencies.

It does not automatically list every transitive dependency.

For example:

```text
Catalog
   │
   ▼
Library A
   │
   ▼
Library B
   │
   ▼
Library C
```

A dependency tree can still contain many artifacts not explicitly listed in the catalog.

This is why dependency inspection remains important.

---

# 60. Catalog and Dependency Constraints

If a project needs more precise control over versions across a graph, Gradle provides additional mechanisms such as dependency constraints.

The conceptual separation is:

```text
Version Catalog
→ Convenient declaration and central naming

Dependency Constraints
→ Rules about acceptable versions
```

A mature project may use both.

---

# 61. Catalog and Build Logic Should Have Clear Ownership

A useful separation is:

```text
libs.versions.toml
    │
    └── "What do we depend on?"

Convention plugin
    │
    └── "How should this type of module be configured?"

Module build file
    │
    └── "What is unique about this module?"
```

This prevents build files from becoming giant configuration scripts.

---

# 62. A KMP Example

A simplified catalog:

```toml
[versions]
kotlin = "2.x.x"
coroutines = "x.x.x"
serialization = "x.x.x"

[libraries]
coroutines-core = {
    module = "org.jetbrains.kotlinx:kotlinx-coroutines-core",
    version.ref = "coroutines"
}

serialization-json = {
    module = "org.jetbrains.kotlinx:kotlinx-serialization-json",
    version.ref = "serialization"
}

[plugins]
kotlin-multiplatform = {
    id = "org.jetbrains.kotlin.multiplatform",
    version.ref = "kotlin"
}
```

Then a KMP module can conceptually use:

```kotlin
plugins {
    alias(libs.plugins.kotlin.multiplatform)
}

kotlin {
    sourceSets {
        commonMain.dependencies {
            implementation(libs.coroutines.core)
            implementation(libs.serialization.json)
        }
    }
}
```

The exact plugin and dependency versions should always be selected for the project's current toolchain.

---

# 63. What This Build File Now Communicates

Instead of:

```text
Long Maven coordinates
Many repeated versions
Plugin IDs everywhere
```

you can read:

```text
This is a KMP module.

It uses:
- Kotlin Multiplatform
- Coroutines
- Serialization
```

The catalog contains the implementation details.

The module build file communicates intent.

---

# 64. Catalog and Readability

This is one of the strongest reasons to use Version Catalogs.

Compare:

```kotlin
implementation(
    "org.jetbrains.kotlinx:kotlinx-serialization-json:..."
)
```

with:

```kotlin
implementation(libs.kotlinx.serialization.json)
```

The second form is easier to scan.

A build file should make the module's intent obvious.

---

# 65. Catalog and Discoverability

A developer joining the project can open:

```text
gradle/libs.versions.toml
```

and quickly see:

```text
Kotlin
Android tooling
Networking
Serialization
Testing
Other major libraries
```

This makes the catalog useful as lightweight build documentation.

---

# 66. Catalog and Dependency Governance

For larger teams, the catalog can support governance.

For example:

```text
Approved libraries
Approved versions
Approved plugins
```

can be represented centrally.

This can reduce accidental introduction of:

```text
Duplicate libraries
Unmaintained libraries
Incompatible versions
```

The catalog does not enforce every rule automatically, but it creates a central place for the team to manage the dependency vocabulary.

---

# 67. Don't Turn the Catalog Into a Junk Drawer

A healthy catalog should be:

```text
Current
Consistent
Named clearly
Reviewed
```

Avoid:

```text
Unused entries
Temporary aliases
Duplicate coordinates
Obsolete versions
Unclear names
```

A catalog that contains everything eventually becomes hard to trust.

---

# 68. Version Catalog and Modularization

As modules increase:

```text
1 module
   ↓
5 modules
   ↓
20 modules
   ↓
50 modules
```

centralized dependency definitions become increasingly valuable.

The catalog allows module build files to stay relatively small:

```text
Module
  │
  ├── Plugins
  ├── Targets
  ├── Source Sets
  └── Dependencies
```

while versions remain centralized.

---

# 69. Catalog and KMP Scaling

A scalable KMP repository may use:

```text
                   Build System
                       │
        ┌──────────────┴──────────────┐
        ▼                             ▼
 Version Catalog                 Convention Plugins
        │                             │
        ▼                             ▼
 Dependency Vocabulary           Build Conventions
        │                             │
        └──────────────┬──────────────┘
                       ▼
                    Modules
                       │
             ┌─────────┴─────────┐
             ▼                   ▼
           Android              iOS
```

This separation becomes valuable when a project has many shared and platform-specific modules.

---

# 70. Version Catalog Is a Tool, Not a Goal

Don't introduce a Version Catalog merely because:

> "Modern projects use it."

Introduce it because it solves real problems:

```text
Repeated versions
Repeated coordinates
Plugin version management
Multi-module consistency
Dependency discoverability
Upgrade maintenance
```

If the project is tiny, the benefit may be smaller.

As the project grows, the benefit becomes much more obvious.

---

# 71. Common Mistake: Centralizing Everything

Centralization can go too far.

For example:

```text
Every Android setting
Every compiler option
Every source-set rule
Every dependency
Every task
```

should not necessarily live in:

```text
libs.versions.toml
```

A Version Catalog is designed primarily for dependency and plugin metadata.

Use convention plugins and other Gradle mechanisms for behavior.

---

# 72. Common Mistake: Treating Catalog Aliases as Architecture

Suppose:

```text
libs.database
```

exists.

That does not mean:

```text
Every module should use database.
```

The alias is simply available.

Architecture still determines:

```text
Which module needs it?
Which source set needs it?
Which target supports it?
```

---

# 73. Common Mistake: Ignoring Target Support

A developer adds:

```kotlin
implementation(libs.some.library)
```

to:

```text
commonMain
```

because the alias exists.

But:

```text
some.library → Android only
```

The catalog did not prevent the mistake.

Always validate:

```text
Dependency
+
Source Set
+
Target Set
```

---

# 74. Common Mistake: Updating Only the Catalog

Changing:

```toml
kotlin = "new-version"
```

is not the end of a Kotlin upgrade.

You should validate:

```text
Gradle wrapper
Kotlin compiler
KMP plugin
Android tooling
Native compilation
Dependencies
CI
```

The catalog makes the version change centralized.

The compatibility work remains.

---

# 75. Common Mistake: Poor Alias Names

Bad:

```text
libs.lib1
libs.lib2
libs.temp
```

Good:

```text
libs.kotlinx.coroutines.core
libs.kotlinx.serialization.json
```

The catalog is read by humans.

Names should communicate intent.

---

# 76. Common Mistake: Duplicate Definitions

Avoid:

```text
library-a
libraryA
a-library
```

all pointing to the same artifact.

Choose one clear alias.

Duplicate aliases increase cognitive load and can make migrations confusing.

---

# 77. Common Mistake: Keeping Deprecated Dependencies

If a library is removed from the project:

```text
Remove it from the catalog.
```

Don't keep it indefinitely "just in case."

A stale catalog becomes less trustworthy.

---

# 78. Common Mistake: Mixing Responsibilities

Avoid putting:

```text
Build behavior
```

into a file intended primarily for:

```text
Dependency metadata
```

A cleaner architecture is:

```text
Catalog
→ Dependencies and plugin metadata

Convention plugins
→ Build behavior

Module build
→ Module-specific behavior
```

---

# 79. Version Catalog Review Checklist

When adding a dependency:

```text
[ ] Is the dependency genuinely required?
[ ] Does the exact version support required KMP targets?
[ ] Is the version centralized?
[ ] Is the alias named clearly?
[ ] Is the source set correct?
[ ] Are transitive dependencies acceptable?
[ ] Is the dependency maintained?
[ ] Is the license acceptable?
[ ] Is the repository trusted?
[ ] Does the dependency belong in commonMain or a platform source set?
```

---

# 80. Version Upgrade Checklist

Before upgrading a version:

```text
[ ] Identify affected modules.
[ ] Check Kotlin / Gradle compatibility.
[ ] Check Android tooling compatibility.
[ ] Check KMP target compatibility.
[ ] Check dependency release notes.
[ ] Update the catalog.
[ ] Build Android.
[ ] Build iOS.
[ ] Run shared tests.
[ ] Run relevant platform tests.
[ ] Verify CI.
```

This is especially important for major version changes.

---

# 81. Version Catalog and Release Discipline

A good repository can make dependency changes visible in code review.

For example:

```text
gradle/libs.versions.toml
```

changes from:

```text
coroutines = old
```

to:

```text
coroutines = new
```

The reviewer can immediately identify:

```text
A dependency upgrade occurred.
```

Then the corresponding source changes and test results can be evaluated.

---

# 82. The Build File Becomes More Intentional

With a well-designed catalog, a KMP build file can communicate:

```text
Plugins
Targets
Source Sets
Dependencies
```

without being overloaded by:

```text
Coordinates
Versions
Repeated plugin IDs
```

That makes the build easier to understand.

---

# 83. The Version Catalog Mental Model

Keep this picture:

```text
                    Version Catalog
                           │
          ┌────────────────┼────────────────┐
          ▼                ▼                ▼
       Versions         Libraries         Plugins
          │                │                │
          └────────────────┼────────────────┘
                           ▼
                     Module Build Files
                           │
                  ┌────────┼────────┐
                  ▼        ▼        ▼
              commonMain androidMain iosMain
                  │        │        │
                  ▼        ▼        ▼
               Shared    Android      iOS
             Dependencies Dependencies Dependencies
```

The catalog centralizes **what** the project uses.

The source sets decide **where** it is used.

---

# 84. The Bigger KMP Build Model

By now, the Gradle picture looks like:

```text
                         Gradle
                           │
             ┌─────────────┼─────────────┐
             ▼             ▼             ▼
          Plugins       Targets      Dependencies
             │             │             │
             ▼             ▼             ▼
          Build Model   Compilations   Source Sets
             │             │             │
             └─────────────┼─────────────┘
                           ▼
                      Build Outputs
```

Version Catalogs sit inside the dependency and plugin management layer.

They do not replace:

```text
Targets
Source Sets
Compilations
Tasks
```

They make the dependency and plugin definitions easier to manage.

---

# Chapter Takeaways

> [!TIP]
> **A Version Catalog centralizes dependency and plugin metadata, while KMP source sets and targets determine where those dependencies are actually used. The catalog improves consistency and readability; it does not remove the need to understand platform compatibility.**

Remember:

1. A Version Catalog provides centralized dependency and plugin metadata.
2. The conventional catalog file is `gradle/libs.versions.toml`.
3. `[versions]` can define reusable version values.
4. `[libraries]` can define dependency aliases.
5. `[plugins]` can define plugin aliases.
6. `[bundles]` can group related dependencies.
7. A Version Catalog does not contain the actual library binaries.
8. A repository provides artifacts; a catalog describes which artifacts the build wants.
9. Version catalogs improve consistency across multi-module projects.
10. Version catalogs improve build-file readability.
11. A catalog alias does not determine dependency scope.
12. Source sets still determine where a dependency is used.
13. `commonMain` dependencies must support the targets consuming common code.
14. An Android-only dependency remains Android-only even when represented by a catalog alias.
15. Centralized versions improve upgrade maintenance but do not guarantee compatibility.
16. Plugin versions can be centralized just like library versions.
17. The Gradle wrapper version is managed separately from the Version Catalog.
18. Android SDK configuration and dependency metadata are related but separate concerns.
19. Convention plugins and Version Catalogs solve different problems and can work together.
20. Version catalogs should use clear, predictable naming.
21. Bundles should represent meaningful groups rather than hiding large dependency sets.
22. A catalog should contain current, intentional dependencies rather than becoming a historical archive.
23. Dependency versions should generally be explicit and reproducible.
24. Version catalogs do not eliminate transitive dependency analysis.
25. Dependency locking and Version Catalogs can complement each other.
26. A dependency should still be evaluated for target support, maintenance, security, licensing, and architecture.
27. A catalog should support the architecture rather than dictate it.
28. Updating a catalog entry can affect every module using that alias, so upgrades should be tested across relevant targets.
29. The strongest KMP build setups separate dependency metadata, build conventions, and module-specific configuration.
30. The central principle is: **the Version Catalog defines the project's dependency vocabulary; the KMP source-set and target model determines where that vocabulary is allowed to be used.**

---

# Final Mental Model

When you see:

```kotlin
implementation(libs.kotlinx.coroutines.core)
```

don't think:

```text
"Gradle shorthand for a dependency."
```

Think:

```text
                 libs.versions.toml
                         │
                         ▼
                Dependency Definition
                         │
                  ┌──────┴──────┐
                  ▼             ▼
               Alias          Version
                  │             │
                  └──────┬──────┘
                         ▼
                  Module Build File
                         │
                         ▼
                    Source Set
                         │
              ┌──────────┴──────────┐
              ▼                     ▼
          commonMain            platformMain
              │                     │
              ▼                     ▼
       Shared Targets         Platform Target
              │
              ▼
          Compilation
```

And remember:

> **A Version Catalog is the central vocabulary of your Gradle dependencies and plugins. It tells the build what libraries and versions the project has chosen, while the KMP source-set and target model determines where those choices are valid. Used properly, it turns dependency management from scattered strings across dozens of build files into a clear, reviewable, and maintainable part of the project's architecture.**
