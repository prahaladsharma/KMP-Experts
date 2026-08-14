# Chapter 6 — Gradle for KMP

## Part 4 — Dependencies

> **In KMP, adding a dependency is not simply adding a library to a project. You are deciding which code can depend on what, which targets must support it, and where that dependency belongs in the architecture.**

A typical Android developer may be used to writing:

```kotlin
dependencies {
    implementation("some.library:artifact:version")
}
```

and moving on.

In Kotlin Multiplatform, the same idea becomes more nuanced.

You now have:

```text
commonMain
androidMain
iosMain
commonTest
```

and potentially several targets:

```text
Android
iOS Device
iOS Simulator
```

The dependency question therefore changes from:

> "Which library do I need?"

to:

> **"Which source set needs it, which targets must support it, and is it actually multiplatform?"**

That is the foundation of dependency management in KMP.

---

# 1. What Is a Dependency?

A dependency is software that your project relies on instead of implementing everything itself.

Examples include libraries for:

```text
Networking
Serialization
Database
Logging
Date / Time
Coroutines
Testing
Dependency Injection
```

Conceptually:

```text
Your Code
   │
   ▼
Dependency
   │
   ▼
Additional Capability
```

Dependencies allow teams to build on established functionality rather than repeatedly solving the same problem.

But in KMP, dependency selection must consider platform compatibility.

---

# 2. The Dependency Problem Changes in KMP

In a single-platform Android project:

```text
Application
     │
     ▼
Android Dependencies
```

In KMP:

```text
                     KMP Module
                         │
             ┌───────────┴───────────┐
             ▼                       ▼
        Shared Dependencies     Platform Dependencies
             │                       │
             ▼                 ┌─────┴─────┐
         commonMain           Android      iOS
```

The build needs to know where each dependency belongs.

That is why KMP dependency configuration is source-set aware.

---

# 3. The Most Important Rule

Keep this rule in mind:

> **A dependency should be declared in the narrowest source set that genuinely needs it.**

For example:

```text
Only common code needs it
        ↓
commonMain

Only Android code needs it
        ↓
androidMain

Only iOS code needs it
        ↓
iosMain

Only shared tests need it
        ↓
commonTest
```

This simple rule prevents many dependency-related problems.

---

# 4. `commonMain` Dependencies

If a dependency is used by genuinely shared production code, it belongs in:

```text
commonMain
```

Conceptually:

```kotlin
commonMain.dependencies {
    implementation(...)
}
```

The important question is not:

> "Can I put this library in commonMain?"

The better question is:

> **"Does this library provide an implementation compatible with every target that consumes commonMain?"**

That distinction is critical.

---

# 5. `androidMain` Dependencies

If a library is Android-specific, it belongs in:

```text
androidMain
```

Conceptually:

```kotlin
androidMain.dependencies {
    implementation(...)
}
```

This means:

```text
Android-specific code
        │
        ▼
Android-specific dependency
```

The dependency does not become part of the common compilation boundary.

---

# 6. `iosMain` Dependencies

Similarly, dependencies required only by iOS-specific code can be associated with:

```text
iosMain
```

Conceptually:

```kotlin
iosMain.dependencies {
    implementation(...)
}
```

This keeps Apple-specific implementation details away from:

```text
commonMain
```

---

# 7. Common Tests Have Their Own Dependencies

Production dependencies and test dependencies should remain separate.

Shared tests belong to:

```text
commonTest
```

Conceptually:

```kotlin
commonTest.dependencies {
    implementation(...)
}
```

Think:

```text
commonMain
    │
    └── Production Dependencies

commonTest
    │
    └── Test Dependencies
```

This keeps the runtime dependency graph cleaner.

---

# 8. Source Set Determines Visibility

Consider:

```text
commonMain
```

and:

```text
androidMain
```

A dependency added to:

```text
androidMain
```

should not suddenly become available to:

```text
commonMain
```

The direction matters.

Think:

```text
commonMain
   │
   ▼
Shared API
   │
   ▼
Platform-specific implementation
```

not:

```text
Platform dependency
      │
      ▼
Shared code
```

This protects the architecture.

---

# 9. Dependency Placement Is an Architectural Decision

Consider a database library.

If the library supports all your KMP targets and your repository layer uses it from shared code, it may belong in:

```text
commonMain
```

But if the database library is Android-only:

```text
androidMain
```

is a more appropriate boundary.

The same feature can therefore have different dependency placements depending on its implementation.

---

# 10. Multiplatform Dependency vs Platform Dependency

This distinction is fundamental.

### Multiplatform dependency

A library provides compatible implementations for multiple KMP targets.

```text
Library
 ├── Android
 └── iOS
```

It can potentially be used from:

```text
commonMain
```

### Platform dependency

A library targets only one platform.

```text
Android-only Library
```

It belongs in:

```text
androidMain
```

The dependency's platform support determines where it can safely live.

---

# 11. `commonMain` Does Not Mean "Anything Shared"

A common misconception is:

> "If the code is shared, all its dependencies should be in commonMain."

Not necessarily.

Suppose:

```text
Shared Repository
```

uses:

```text
Platform-specific Database
```

The repository may need an abstraction in:

```text
commonMain
```

while implementations use:

```text
androidMain
iosMain
```

The architecture becomes:

```text
commonMain
   │
   ▼
Repository Interface
   │
   ├── Android Implementation
   │       │
   │       ▼
   │   Android Dependency
   │
   └── iOS Implementation
           │
           ▼
       iOS Dependency
```

This is often healthier than forcing the platform library into common code.

---

# 12. Dependency Graph

A dependency graph can be visualized as:

```text
                    commonMain
                        │
          ┌─────────────┼─────────────┐
          ▼             ▼             ▼
       Network      Serialization    Domain
       Library        Library        Library
          │             │             │
          └─────────────┼─────────────┘
                        ▼
                  Platform Targets
                  ┌─────┴─────┐
                  ▼           ▼
               Android       iOS
```

The common dependencies must be compatible with the targets consuming them.

---

# 13. Why Android Developers Get Tripped Up

In Android, you may have years of experience choosing libraries based on:

```text
Popularity
Documentation
GitHub stars
Android support
```

KMP adds another filter:

```text
KMP target support
```

A library can be excellent for Android and still be the wrong choice for:

```text
commonMain
```

because it does not support the required iOS target.

---

# 14. "Works on Android" Is Not Enough

Suppose:

```text
Library X
```

works perfectly on Android.

You add it to:

```text
commonMain
```

Then:

```text
Android → PASS
iOS → FAIL
```

The library may not have a compatible iOS implementation.

Therefore:

> **Android compatibility is not the same as multiplatform compatibility.**

This is one of the most important dependency lessons in KMP.

---

# 15. Dependency Compatibility Matrix

Before adopting a library, think in terms of a matrix:

| Dependency | Android | iOS Device | iOS Simulator |
|---|:---:|:---:|:---:|
| Library A | ✓ | ✓ | ✓ |
| Library B | ✓ | ✓ | ✓ |
| Library C | ✓ | — | — |

If Library C is declared in:

```text
commonMain
```

the iOS compilation becomes a problem.

The matrix makes the issue obvious.

---

# 16. Target Support Can Change

A library's platform support is not necessarily permanent.

A dependency may:

```text
Add iOS support
Drop a target
Change native implementation
Change supported architectures
Change Kotlin compatibility
```

Therefore, dependency evaluation should consider the version you are actually using.

Don't assume:

```text
Library X supports iOS
```

without checking the relevant version and documentation.

---

# 17. Dependency Versions Matter

Two versions of the same library can have different compatibility.

Conceptually:

```text
Library 1.x
    └── Android

Library 2.x
    ├── Android
    └── iOS
```

Therefore:

```text
Library Name
```

is not enough.

You need to know:

```text
Library
+
Version
+
Target Support
```

---

# 18. Dependency Declaration

A simplified KMP dependency configuration may look like:

```kotlin
kotlin {
    sourceSets {
        commonMain.dependencies {
            implementation("...")
        }

        androidMain.dependencies {
            implementation("...")
        }

        iosMain.dependencies {
            implementation("...")
        }
    }
}
```

The exact syntax depends on the current Kotlin Multiplatform Gradle APIs and project configuration.

The concept is stable:

```text
Source Set
     │
     ▼
Dependency Scope
```

---

# 19. Dependency Scope

A dependency declaration should communicate:

```text
Who needs this dependency?
```

For example:

```text
commonMain
```

means:

```text
Shared production code needs it.
```

while:

```text
commonTest
```

means:

```text
Shared tests need it.
```

And:

```text
androidMain
```

means:

```text
Android-specific shared-module code needs it.
```

This is much more informative than putting everything into one giant list.

---

# 20. Why Narrow Dependency Scope Matters

Imagine:

```text
commonMain
```

has:

```text
20 dependencies
```

even though only:

```text
5
```

are genuinely multiplatform.

Now every new target must deal with all 20.

The common compilation boundary becomes fragile.

A narrower dependency graph reduces unnecessary coupling.

---

# 21. The Dependency Boundary

Think:

```text
                     commonMain
                         │
                ┌────────┴────────┐
                ▼                 ▼
        Common Dependencies   Common APIs
                │                 │
                └────────┬────────┘
                         ▼
                 Platform Boundary
                    ┌────┴────┐
                    ▼         ▼
                 Android      iOS
                    │         │
               Platform     Platform
              Dependencies Dependencies
```

The dependency boundary should follow the code boundary.

---

# 22. Transitive Dependencies

A library may itself depend on other libraries.

For example:

```text
Your Code
   │
   ▼
Library A
   │
   ▼
Library B
```

Library B is a transitive dependency.

This matters in KMP because Library A may be multiplatform while one of its transitive dependencies has limited target support.

Conceptually:

```text
commonMain
   │
   ▼
Library A
   │
   ▼
Library B
   │
   ▼
Target Compatibility
```

You need to consider the complete dependency graph, not only direct dependencies.

---

# 23. Hidden Platform Dependencies

A library can appear multiplatform at the top level but still contain platform-specific pieces.

This is why dependency support should be evaluated based on:

```text
Actual targets
Actual version
Actual configuration
```

not simply:

```text
"Multiplatform" appears in the README.
```

---

# 24. Dependency Resolution

Gradle resolves dependencies before and during the build process.

A simplified model is:

```text
Dependency Declaration
        │
        ▼
Repository Resolution
        │
        ▼
Version Selection
        │
        ▼
Dependency Graph
        │
        ▼
Compilation
```

For KMP, target compatibility is part of the equation.

---

# 25. Repositories

Gradle needs repositories from which dependencies and plugins can be resolved.

A project may use repositories such as:

```text
Maven Central
Google
Internal company repository
```

The exact repository configuration depends on the project.

A dependency must be available from a repository that Gradle can access.

---

# 26. Don't Add Random Repositories

A common reaction to:

```text
Could not resolve dependency
```

is:

> "Add another repository."

That can be dangerous.

Additional repositories can introduce:

```text
Resolution ambiguity
Security concerns
Build reproducibility issues
Unexpected artifact selection
```

First determine:

```text
Is the dependency actually published?
Is the version correct?
Is the repository configured correctly?
```

Then add only the repository that is genuinely required.

---

# 27. Dependency Resolution and Security

Dependencies are part of your software supply chain.

Before adopting a library, consider:

```text
Who publishes it?
Is the artifact from the expected repository?
Is the version known and verified?
Is the project maintained?
Does it support the required targets?
```

A dependency should not be selected purely because it solves a technical problem.

---

# 28. Direct vs Transitive Dependencies

A direct dependency is declared by your project:

```text
Your Module
   │
   ▼
Library A
```

A transitive dependency is brought in by another library:

```text
Your Module
   │
   ▼
Library A
   │
   ▼
Library B
```

Understanding this graph is useful when:

```text
Version conflicts occur
A target fails to compile
A native artifact is missing
Build size increases
```

---

# 29. Dependency Conflicts

Suppose:

```text
Library A → Utility 1.5
Library B → Utility 2.0
```

Now Gradle must resolve a compatible graph.

Conceptually:

```text
        Library A
            │
            ▼
        Utility 1.5
            │
            ├────────┐
            │        │
        Resolution  │
            │        │
            ▼        ▼
        Selected Version
```

The exact resolution behavior depends on Gradle's dependency model and configuration.

The important lesson is:

> **Adding one dependency can change the dependency graph beyond that one library.**

---

# 30. KMP Adds Another Dimension to Conflicts

A dependency may resolve correctly for Android but fail for iOS.

For example:

```text
Android dependency graph
        │
        ▼
      PASS

iOS dependency graph
        │
        ▼
      FAIL
```

The project can therefore have target-specific dependency resolution problems.

This is another reason to think in terms of target-aware dependency graphs.

---

# 31. Platform-Specific Dependency Example

Imagine a shared abstraction:

```kotlin
interface SecureStorage {
    fun save(key: String, value: String)
    fun read(key: String): String?
}
```

The common code depends only on:

```text
SecureStorage
```

Android can use:

```text
Android-specific secure storage implementation
```

iOS can use:

```text
Apple-specific secure storage implementation
```

The dependency graph becomes:

```text
                 commonMain
                     │
                     ▼
              SecureStorage API
                     │
              ┌──────┴──────┐
              ▼             ▼
           Android          iOS
              │             │
              ▼             ▼
       Android Library   Apple API
```

This keeps platform dependencies out of the common boundary.

---

# 32. Don't Hide Bad Dependencies Behind Abstractions

An abstraction is not a magic solution.

Suppose:

```text
commonMain
   │
   ▼
Repository
   │
   ▼
Android-only library
```

If the repository directly imports the Android library, the code is still platform-specific.

The correct architecture is usually:

```text
commonMain
   │
   ▼
Abstraction
   │
   ├── Android implementation
   └── iOS implementation
```

The dependency should follow the implementation boundary.

---

# 33. Dependency Injection and KMP

Dependency injection makes this boundary particularly visible.

Conceptually:

```text
commonMain
    │
    ▼
Repository
    │
    ▼
Interface
    │
 ┌──┴──┐
 ▼     ▼
Android iOS
Impl    Impl
```

Platform-specific implementations can receive platform-specific dependencies.

The common layer remains unaware of those concrete libraries.

---

# 34. Dependencies and `expect` / `actual`

Another approach is to define:

```kotlin
expect class PlatformStorage
```

and implement:

```kotlin
actual class PlatformStorage
```

for each target.

Then:

```text
commonMain
   │
   ▼
expect
   │
   ├── Android actual
   │      │
   │      ▼
   │   Android dependency
   │
   └── iOS actual
          │
          ▼
       iOS dependency
```

The dependency follows the target-specific implementation.

---

# 35. Common Dependencies Should Be Truly Common

A useful test is:

> **Could the code compile for every target using this dependency without introducing platform-specific assumptions?**

If yes:

```text
commonMain
```

may be appropriate.

If no:

```text
Platform-specific source set
```

is usually the better boundary.

---

# 36. Dependency Selection Before Architecture

Don't start by asking:

> "Which library is most popular?"

Start with:

```text
What capability do we need?
        │
        ▼
Does the capability need to be shared?
        │
        ▼
Which targets need it?
        │
        ▼
Which libraries support those targets?
        │
        ▼
Which library best fits the architecture?
```

This avoids choosing a library first and then forcing the architecture around it.

---

# 37. The Cost of a Dependency

A dependency has more than a version number.

It also brings:

```text
API surface
Transitive dependencies
Build cost
Target support
Native integration
Upgrade cost
Security considerations
Maintenance burden
```

In KMP, target support should be considered part of that cost.

---

# 38. Dependency Evaluation Matrix

A practical evaluation can look like:

| Question | Answer |
|---|---|
| What problem does it solve? | Networking |
| Common or platform-specific? | Common |
| Android support? | Yes |
| iOS support? | Yes |
| Required architectures? | Supported |
| Native integration? | Needed |
| Testing support? | Available |
| Maintenance status? | Acceptable |
| License? | Compatible with project requirements |
| Build impact? | Acceptable |

This is much stronger than:

```text
"Library X is popular."
```

---

# 39. Dependency Placement Rules

A simple rule set:

```text
Shared production code
        ↓
commonMain

Android-specific shared code
        ↓
androidMain

iOS-specific shared code
        ↓
iosMain

Shared tests
        ↓
commonTest

Platform-specific tests
        ↓
Platform test source set
```

This should be your default mental model.

---

# 40. Dependency Scope and API Leakage

Be careful when exposing dependencies through public APIs.

Suppose:

```text
commonMain
```

has a dependency whose type appears in a public API:

```kotlin
fun fetch(): SomeLibraryType
```

Now your architecture is coupled to that library.

Even if the dependency is technically valid, it may be better to expose your own domain type:

```kotlin
fun fetch(): Product
```

and keep the library-specific type inside the implementation.

This is especially important in shared libraries.

---

# 41. Keep Third-Party Types at the Boundary

A healthy architecture often looks like:

```text
External Library
      │
      ▼
Adapter / Mapper
      │
      ▼
Domain Model
      │
      ▼
Application
```

Instead of:

```text
External Library
      │
      ▼
Every Layer
```

This reduces coupling and makes dependency replacement easier.

---

# 42. Dependency and Mapping

Suppose a networking library returns:

```text
NetworkResponse
```

Your domain may need:

```text
User
```

Use a mapper:

```text
NetworkResponse
      │
      ▼
Mapper
      │
      ▼
User
```

The external dependency stays closer to the data boundary.

This makes shared architecture easier to evolve.

---

# 43. Dependency and Testing

Dependencies affect testability.

If a shared repository directly constructs a concrete networking client:

```text
Repository
   │
   ▼
Concrete Client
```

testing becomes harder.

A better structure is:

```text
Repository
   │
   ▼
Network Interface
   │
   ├── Real Implementation
   └── Test Implementation
```

Dependency management and architecture therefore influence each other.

---

# 44. Test Dependencies Should Stay in Tests

Avoid putting testing libraries into:

```text
commonMain
```

when they are only needed by:

```text
commonTest
```

Conceptually:

```text
commonMain
   └── Production Dependencies

commonTest
   └── Test Dependencies
```

This keeps production code independent from test tooling.

---

# 45. Dependency Injection Libraries

Dependency injection libraries can be useful in KMP, but evaluate them carefully.

Ask:

```text
Does it support all required targets?
Does it require platform-specific setup?
Does it add compile-time or runtime complexity?
Does it work with the Kotlin version we use?
```

Don't select a DI library solely because it is familiar from Android.

---

# 46. Familiar Android Libraries Need Re-Evaluation

An Android developer may be comfortable with a library that has:

```text
Excellent Android support
```

But KMP changes the question to:

```text
Excellent Android support
+
Required iOS support
+
Compatible KMP architecture
```

The right library for Android is not automatically the right library for shared KMP code.

---

# 47. Dependency Selection and Migration

When migrating an Android application toward KMP, inventory dependencies:

```text
Dependency
   │
   ├── Multiplatform
   ├── Android-only
   └── Replace / Abstract
```

For example:

```text
Existing Android Dependency
          │
          ▼
     Can it run in common?
          │
     ┌────┴────┐
     ▼         ▼
    Yes        No
     │         │
     ▼         ▼
commonMain   Platform
             Boundary
```

This is one of the most practical KMP migration exercises.

---

# 48. Dependency Migration Matrix

For an existing Android application:

| Existing Dependency | Shared? | Android-only? | KMP Action |
|---|:---:|:---:|---|
| Networking | ✓ | — | Evaluate KMP library |
| JSON serialization | ✓ | — | Use compatible shared library |
| Android UI | — | ✓ | Keep in Android app |
| Android database | — | ✓ | Replace or abstract |
| Logging | ? | ? | Evaluate target support |
| Analytics | — | ✓ / platform-specific | Keep behind abstraction |

The goal is not to replace everything.

The goal is to place each dependency at the correct boundary.

---

# 49. Don't Force Every Dependency Into `commonMain`

A migration can fail when teams interpret:

```text
KMP migration
```

as:

```text
Move all dependencies to commonMain.
```

That is not the goal.

Instead:

```text
Shared business capability
       │
       ▼
Evaluate shared implementation
       │
       ├── Common dependency
       │
       └── Platform abstraction
```

The architecture should determine the dependency boundary.

---

# 50. Dependency Trees

Gradle provides tools for inspecting dependency graphs.

A common starting point is:

```bash
./gradlew dependencies
```

The exact useful task and configuration can vary by module and plugin.

The purpose is to answer:

```text
What dependencies are actually present?
Which versions were selected?
What is transitive?
```

For a specific problem, inspect the relevant module and configuration rather than treating the entire repository as one graph.

---

# 51. Dependency Insight

Gradle also provides dependency insight mechanisms for understanding why a particular dependency version or path exists.

Conceptually:

```text
Why is this library here?
        │
        ▼
Dependency Graph
        │
        ▼
Path A
Path B
Path C
        │
        ▼
Selected Version
```

This becomes valuable when KMP builds fail because of:

```text
Version conflicts
Native variants
Unexpected transitive dependencies
```

---

# 52. Read the Error From the Target

When dependency resolution fails, first identify:

```text
Which target?
```

Then:

```text
Which source set?
```

Then:

```text
Which dependency?
```

For example:

```text
iOS compilation
      │
      ▼
commonMain dependency
      │
      ▼
Library X
      │
      ▼
No compatible artifact
```

This tells you much more than simply:

```text
"Gradle failed."
```

---

# 53. Dependency Resolution Is Not Always a Library Problem

A dependency failure can also come from:

```text
Wrong version
Repository configuration
Plugin/toolchain mismatch
Unsupported target
Architecture mismatch
Transitive dependency
Incorrect source-set placement
```

Therefore, avoid immediately replacing the library.

First identify the actual failure layer.

---

# 54. Target Compatibility and Native Artifacts

For native targets, dependency support can involve native artifacts compatible with the required target.

A simplified view:

```text
Library
   │
   ├── Android artifact
   ├── iOS device artifact
   └── iOS simulator artifact
```

The exact publishing and resolution model depends on the library.

The important point is:

> **A library needs compatible artifacts or implementations for the target that consumes it.**

---

# 55. Dependency Version Catalogs

KMP projects may centralize dependency versions in:

```text
gradle/libs.versions.toml
```

Conceptually:

```text
libs.versions.toml
        │
        ├── Kotlin
        ├── Networking
        ├── Serialization
        └── Testing
```

Then module build files can reference aliases.

This improves consistency across modules.

---

# 56. Centralization Does Not Mean Common Scope

A version catalog answers:

```text
What version are we using?
```

It does not answer:

```text
Where should this dependency be used?
```

You can centrally define a dependency and still place it only in:

```text
androidMain
```

or:

```text
commonTest
```

Keep these concepts separate.

---

# 57. Dependency Management Has Two Dimensions

Think:

```text
             Dependency Management
                       │
             ┌─────────┴─────────┐
             ▼                   ▼
        Version / Source      Scope / Target
             │                   │
             ▼                   ▼
        Which library?       Where is it used?
```

Both matter.

A correct version in the wrong source set is still a bad configuration.

---

# 58. Dependency Constraints

Large projects may need to control versions across a dependency graph.

Conceptually:

```text
Library A ──┐
            ├──► Utility
Library B ──┘
```

A constraint or centralized version policy can help maintain consistency.

This is especially useful when multiple modules depend on overlapping libraries.

The exact Gradle mechanism should be chosen according to the project's needs.

---

# 59. Avoid Version Drift

Without centralized management:

```text
Module A → Library 2.1
Module B → Library 2.2
Module C → Library 2.0
```

This creates confusion.

A version catalog or dependency management strategy can establish:

```text
One approved version
        │
        ├── Module A
        ├── Module B
        └── Module C
```

This makes upgrades easier to reason about.

---

# 60. Dependency Upgrades in KMP

When upgrading a dependency, don't test only Android.

Ask:

```text
Does Android still compile?
Does iOS still compile?
Does the simulator still compile?
Do common tests pass?
Do platform tests pass?
Did native integration change?
```

A dependency upgrade can succeed on one target and fail on another.

---

# 61. Dependency Updates Should Be Incremental

A risky upgrade looks like:

```text
Kotlin ↑
Gradle ↑
AGP ↑
Library A ↑
Library B ↑
Library C ↑
```

all at once.

Then:

```text
Build fails
```

and nobody knows why.

A safer process is:

```text
One meaningful change
        │
        ▼
Build
        │
        ▼
Test
        │
        ▼
Next change
```

This is especially useful for KMP because multiple toolchains are involved.

---

# 62. Dependency Security

Dependency management is also security management.

For every external dependency, consider:

```text
Source
Version
Publisher
Integrity
Maintenance
Known vulnerabilities
License
Target support
```

Do not copy arbitrary dependency coordinates from untrusted sources.

Use trusted repositories and verify the official documentation for the library and version you intend to use.

---

# 63. Dependency Licenses

A dependency can also carry licensing requirements.

Before adopting a library in a commercial product, check:

```text
License
Notice requirements
Distribution requirements
Third-party obligations
```

The exact legal requirements depend on the license and how the software is distributed.

Engineering teams should treat license review as part of dependency selection.

---

# 64. Dependency Maintenance

A library is not free after adding:

```kotlin
implementation(...)
```

You now own the relationship with that dependency.

That includes:

```text
Updates
Compatibility
Security
Build failures
Migration work
Documentation changes
```

In KMP, also:

```text
Target support
Native changes
Compiler compatibility
```

The real cost is the ongoing maintenance.

---

# 65. Dependency Ownership

For a large team, it can help to know:

```text
Who owns this dependency?
Why was it selected?
Which modules use it?
Which targets depend on it?
```

A simple internal dependency catalog or documentation can prevent:

```text
Duplicate libraries
Conflicting versions
Unnecessary replacements
Unowned integrations
```

---

# 66. Avoid Duplicate Libraries

Suppose the project has:

```text
Networking Library A
Networking Library B
Networking Library C
```

all solving the same problem.

This increases:

```text
Build size
Maintenance
Knowledge burden
Potential conflicts
```

A shared KMP architecture benefits from deliberate dependency consolidation.

---

# 67. Dependency and Abstraction Boundaries

A useful architecture is:

```text
                 Application
                      │
                      ▼
                 Domain Layer
                      │
                      ▼
               Stable Interfaces
                      │
          ┌───────────┴───────────┐
          ▼                       ▼
      Shared Impl             Platform Impl
          │                       │
          ▼                       ▼
    Common Library          Platform Library
```

Dependencies should follow these boundaries.

They should not dictate the architecture from the bottom upward.

---

# 68. Dependency-Driven Architecture Is Dangerous

Avoid:

```text
We chose Library X
      │
      ▼
Our architecture must look like X
```

Prefer:

```text
Product Requirement
      │
      ▼
Architecture
      │
      ▼
Dependency Selection
```

The library should support your architecture rather than becoming your architecture.

---

# 69. KMP Dependency Decision Flow

A practical decision flow:

```text
Need a capability
       │
       ▼
Does shared code need it?
       │
   ┌───┴────┐
   ▼        ▼
  Yes       No
   │         │
   ▼         ▼
Evaluate   Platform
KMP support Dependency
   │
   ▼
Supports all required targets?
   │
 ┌─┴──┐
 ▼    ▼
Yes   No
 │     │
 ▼     ▼
Common  Abstract /
Dependency Platform Impl
```

This flow is simple enough to use during real development.

---

# 70. A Networking Example

Suppose:

```text
ProductRepository
```

needs networking.

The wrong approach is:

```text
commonMain
   │
   ▼
Android-only networking client
```

A better approach is:

```text
commonMain
   │
   ▼
Multiplatform Network API
   │
   ├── Android
   └── iOS
```

Or:

```text
commonMain
   │
   ▼
Network Interface
   │
   ├── Android implementation
   └── iOS implementation
```

The correct choice depends on the library and architecture.

---

# 71. A Database Example

Suppose the application needs local persistence.

Start with the capability:

```text
Local Database
```

Then ask:

```text
Should the database API be shared?
Which targets are required?
Does the chosen library support them?
Do platform-specific features matter?
```

The result might be:

```text
commonMain
   │
   ▼
Shared Database API
   │
   ├── Android
   └── iOS
```

or:

```text
commonMain
   │
   ▼
Database abstraction
   │
   ├── Android database implementation
   └── iOS database implementation
```

The target support determines which option is practical.

---

# 72. A Logging Example

Logging is often a good example of a capability that can have different implementations.

Common code might depend on:

```text
Logger interface
```

while platform code provides:

```text
Android logger
iOS logger
```

Conceptually:

```text
commonMain
   │
   ▼
Logger
   │
 ┌─┴─┐
 ▼   ▼
Android iOS
```

The important part is not the library.

It is the boundary.

---

# 73. Dependencies and the Clean Architecture Boundary

In a clean architecture style, you generally want:

```text
Domain
   │
   ▼
Application / Use Cases
   │
   ▼
Interfaces
   │
   ▼
Infrastructure
```

Third-party libraries usually belong closer to:

```text
Infrastructure
```

rather than becoming domain concepts.

KMP makes this especially useful because infrastructure can differ by target.

---

# 74. Don't Let a Library Define Your Domain

Avoid domain models like:

```kotlin
data class User(
    val response: ThirdPartyResponse
)
```

Prefer:

```kotlin
data class User(
    val id: String,
    val name: String
)
```

and map:

```text
ThirdPartyResponse
       │
       ▼
Mapper
       │
       ▼
User
```

This keeps your domain independent from external dependency types.

---

# 75. Dependencies and API Stability

If a shared public API exposes third-party types, changing the dependency can become a breaking API change.

For example:

```kotlin
fun getUser(): ThirdPartyUser
```

couples your API to:

```text
ThirdPartyUser
```

Prefer:

```kotlin
fun getUser(): User
```

when the third-party type is an implementation detail.

This is particularly important if the shared module may eventually be published or consumed by multiple applications.

---

# 76. Dependency Boundaries in a Published KMP Library

If your KMP module becomes a library:

```text
Consumer
   │
   ▼
Your KMP API
   │
   ▼
Internal Dependencies
```

You should carefully control which dependencies become part of the consumer-facing API.

A library's dependency graph can become part of its compatibility surface.

---

# 77. Dependencies and Build Performance

Every dependency can affect build performance.

Potential costs include:

```text
Dependency resolution
Compilation
Annotation / code generation
Native compilation
Linking
Incremental build behavior
```

KMP adds native compilation considerations.

A dependency that is cheap on Android may have a different build cost for native targets.

---

# 78. Dependencies and Native Build Time

Native compilation can be sensitive to:

```text
Number of source files
Dependency graph size
Compiler work
Linking
Generated code
```

Therefore:

> **A dependency decision can affect both architecture and build performance.**

This becomes increasingly important as the shared module grows.

---

# 79. Keep the Shared Dependency Graph Healthy

A healthy shared module often looks like:

```text
commonMain
   │
   ├── Small set of core dependencies
   │
   ├── Stable abstractions
   │
   └── Platform boundaries
          │
      ┌───┴───┐
      ▼       ▼
   Android    iOS
```

Not:

```text
commonMain
   │
   ├── Android Library
   ├── iOS Library
   ├── Random Utility
   ├── Another Framework
   ├── Platform API
   └── Everything Else
```

The first graph is easier to evolve.

---

# 80. Dependency Review Checklist

Before adding a dependency to `commonMain`, ask:

```text
[ ] Is the capability genuinely shared?
[ ] Does the library support every required target?
[ ] Does the exact version support those targets?
[ ] Are required architectures supported?
[ ] Are transitive dependencies compatible?
[ ] Does the library expose platform-specific APIs?
[ ] Does it fit our architecture?
[ ] Does it introduce significant build cost?
[ ] Is it actively maintained?
[ ] Is the license acceptable?
[ ] Is the source trustworthy?
[ ] Can we replace it later if necessary?
```

This is a strong baseline for production KMP projects.

---

# 81. Dependency Debugging Checklist

When a dependency fails:

```text
1. Identify the failing target.
2. Identify the source set.
3. Identify the direct dependency.
4. Inspect transitive dependencies.
5. Verify the exact version.
6. Verify target support.
7. Verify repository configuration.
8. Check Kotlin / Gradle compatibility.
9. Check native architecture support where relevant.
10. Re-run the smallest useful Gradle task.
```

Avoid random upgrades until you understand the failure.

---

# 82. The Dependency Mental Model

Keep this picture:

```text
                         Dependency
                              │
                ┌─────────────┴─────────────┐
                ▼                           ▼
           Source Set                  Target Support
                │                           │
                ▼                           ▼
           Where used?                Where can it run?
                │                           │
                └─────────────┬─────────────┘
                              ▼
                       Compilation
                              │
                              ▼
                           Output
```

A dependency is valid only when both sides make sense:

```text
Correct Scope
+
Compatible Target
```

---

# Chapter Takeaways

> [!TIP]
> **KMP dependency management is about more than adding libraries. It is about controlling the relationship between source sets, targets, architecture, and external software.**

Remember:

1. A dependency is external software your project relies on for additional capability.
2. KMP makes dependency management source-set and target aware.
3. A dependency should generally be declared in the narrowest source set that genuinely needs it.
4. `commonMain` should contain dependencies that are compatible with every target consuming the shared code.
5. `androidMain` is appropriate for Android-specific dependencies used by shared-module Android code.
6. `iosMain` is appropriate for iOS-specific dependencies used by shared-module iOS code.
7. `commonTest` should contain dependencies needed by shared tests rather than production code.
8. A multiplatform library is not automatically compatible with every target your project may add.
9. Android compatibility does not imply iOS compatibility.
10. Dependency compatibility must be evaluated for the exact version being used.
11. Transitive dependencies can introduce target or version problems even when the direct dependency looks compatible.
12. Dependency placement is an architectural decision.
13. Platform-specific dependencies should generally stay behind platform-specific source-set boundaries.
14. `expect` / `actual` and interfaces can help isolate platform-specific dependencies.
15. Dependency versions and dependency scope are separate concerns.
16. Version catalogs can centralize versions without determining where dependencies are used.
17. Dependency repositories should be chosen deliberately rather than added randomly to fix resolution errors.
18. Dependencies are part of the software supply chain and should be evaluated for trust, maintenance, security, and licensing.
19. Third-party library types should not unnecessarily leak into stable domain APIs.
20. Dependency choices affect build performance as well as runtime behavior.
21. Adding a new KMP target can expose previously hidden dependency incompatibilities.
22. A dependency upgrade should be tested across all relevant targets, not only Android.
23. Dependency graphs should be inspected when resolution or compilation failures are unclear.
24. A good KMP architecture lets product requirements drive dependency selection, rather than allowing a library to dictate the architecture.
25. The goal is not to put everything into `commonMain`; the goal is to share what is genuinely common and isolate what is platform-specific.

---

# Final Mental Model

When you see:

```kotlin
sourceSets {
    commonMain.dependencies {
        implementation("...")
    }

    androidMain.dependencies {
        implementation("...")
    }

    iosMain.dependencies {
        implementation("...")
    }
}
```

don't read it as three different Gradle syntax blocks.

Read it as an architectural map:

```text
                         Dependencies
                              │
             ┌────────────────┼────────────────┐
             ▼                ▼                ▼
         commonMain       androidMain        iosMain
             │                │                │
             ▼                ▼                ▼
       Shared Library     Android Library    iOS Library
             │                │                │
             ▼                ▼                ▼
          Android            Android             iOS
          + iOS
```

And remember the central rule:

> **The right KMP dependency is not simply the library that solves the problem. It is the library—or abstraction—that solves the problem at the correct architectural boundary and supports the targets that actually need it. Once dependency scope and target compatibility become part of your decision-making, Gradle stops being a place where libraries are listed and becomes a precise map of what each part of your multiplatform application is allowed to depend on.**
