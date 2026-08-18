# Chapter 7 — Source Sets Deep Dive

## Part 7 — Dependency Resolution

> **In Kotlin Multiplatform, dependency resolution is not simply about adding a library. It is about determining which dependency is available to which source set, which platform it supports, and how Gradle constructs the correct dependency graph for each target.**

Dependency management is one of the areas where KMP starts to feel different from conventional Android development.

In a traditional Android project, developers often think in terms of:

```kotlin
implementation("some-library")
```

and expect the dependency to be available throughout the module.

KMP introduces another dimension:

```text
Which source set?
Which target?
Which variant?
Which platform?
Which dependency version?
Which transitive dependencies?
```

A simplified KMP dependency graph looks like this:

```text
                         commonMain
                             │
                  ┌──────────┴──────────┐
                  │                     │
             common dependency     platform API
                  │
          ┌───────┼────────┬──────────────┐
          ▼       ▼        ▼              ▼
     androidMain iosMain desktopMain  wasmJsMain
          │       │        │              │
          ▼       ▼        ▼              ▼
      Android    iOS     Desktop          Web
```

Understanding this graph is critical because dependency placement directly affects portability.

---

## 1. What Is Dependency Resolution?

Dependency resolution is the process by which Gradle determines:

```text
What libraries are required?
Which versions should be used?
Where should they come from?
Which variants are compatible?
Which transitive dependencies are required?
Which dependency belongs to which target?
```

For a normal Android application, the graph may be relatively straightforward:

```text
app
 │
 ├── Retrofit
 ├── Room
 └── Hilt
```

A KMP project may instead have:

```text
shared
 │
 ├── commonMain
 │    ├── Serialization
 │    └── Networking
 │
 ├── androidMain
 │    └── Android-specific dependency
 │
 ├── iosMain
 │    └── Apple-specific dependency
 │
 └── wasmJsMain
      └── Web-specific dependency
```

Gradle must resolve these dependencies independently according to the configured target and source set.

---

## 2. The Most Important Rule

A dependency should be declared at the **highest source set that can actually use it**.

For example:

```text
Used by Android + iOS + Desktop + Web
                ↓
           commonMain
```

If a dependency only works on Android:

```text
Android only
    ↓
androidMain
```

If it only works on iOS:

```text
iOS only
    ↓
iosMain
```

If it only works with WebAssembly:

```text
Web only
    ↓
wasmJsMain
```

This simple rule prevents unnecessary platform coupling.

---

## 3. Dependency Placement

Consider:

```text
commonMain
```

If you add a dependency here:

```kotlin
commonMain.dependencies {
    implementation(libs.some.library)
}
```

the dependency becomes part of the shared source-set dependency graph.

That means the library must be compatible with the targets consuming that source set.

Conceptually:

```text
commonMain
     │
     ├── Android
     ├── iOS
     ├── Desktop
     └── Web
```

Therefore, `commonMain` is a powerful but demanding place to declare dependencies.

---

## 4. Android-Specific Dependency

Suppose a library depends on Android APIs.

It should not be placed in:

```text
commonMain
```

Instead:

```kotlin
androidMain.dependencies {
    implementation(libs.android.someLibrary)
}
```

The graph becomes:

```text
commonMain
     │
     ├── Android
     │     └── AndroidLibrary
     │
     ├── iOS
     ├── Desktop
     └── Web
```

Only the Android target receives the dependency.

---

## 5. iOS-Specific Dependency

The same principle applies to iOS.

If a dependency requires Apple-specific APIs:

```text
iosMain
    ↓
iOS dependency
```

The shared source set should depend on an abstraction rather than directly importing the platform implementation.

---

## 6. WebAssembly-Specific Dependency

For WebAssembly-specific functionality:

```text
wasmJsMain
    ↓
Wasm/Web dependency
```

This keeps Web APIs outside the shared source set.

For example:

```text
commonMain
    ↓
Storage interface
    ↑
    │
wasmJsMain
    ↓
Browser storage implementation
```

---

## 7. Source Set Hierarchy and Dependencies

KMP source sets can form relationships.

For example:

```text
commonMain
     │
     ├── androidMain
     │
     ├── iosMain
     │
     ├── desktopMain
     │
     └── wasmJsMain
```

A child source set can use dependencies available from its parent.

Therefore:

```text
commonMain dependency
        ↓
child source set
```

can inherit access according to the configured source-set hierarchy.

This is one of the reasons dependency placement matters so much.

---

## 8. Dependency Visibility Flows Downward

Think of source-set dependency visibility as a tree:

```text
             commonMain
                 │
       ┌─────────┼─────────┐
       ▼         ▼         ▼
    Android     iOS      Desktop
```

A dependency declared in:

```text
commonMain
```

can be available to the appropriate descendants.

But a dependency declared in:

```text
androidMain
```

does not suddenly become available to:

```text
iosMain
```

This directional model is important.

---

## 9. A Dependency Does Not Automatically Become Universal

Suppose:

```kotlin
androidMain.dependencies {
    implementation(libs.some.library)
}
```

That does **not** mean:

```text
iOS
Desktop
Web
```

can use it.

It belongs to the Android branch.

The dependency graph remains platform-aware.

---

## 10. Common Dependency Example

Imagine the application uses Kotlin serialization.

Conceptually:

```text
commonMain
    └── Serialization
```

The serialization library is used by:

```text
Android
iOS
Desktop
Web
```

when supported by the selected version/configuration.

The dependency therefore belongs at the shared level.

---

## 11. Platform Dependency Example

Now imagine an Android-specific integration:

```text
Android Camera API
```

It should remain:

```text
androidMain
```

The architecture becomes:

```text
commonMain
    │
    └── CameraProvider
            ↑
            │
       androidMain
            │
       Android Camera
```

The shared layer knows what it needs.

The platform layer knows how to provide it.

---

## 12. Dependency Configuration Names

KMP exposes dependency configurations associated with source sets.

Common examples include:

```text
commonMainImplementation
commonTestImplementation
androidMainImplementation
androidUnitTestImplementation
iosMainImplementation
```

The exact available configurations depend on the project and plugins.

Developers should think in terms of:

```text
Source set → dependency configuration
```

rather than treating the project as one large Android module.

---

## 13. `commonMainImplementation`

A common dependency can conceptually be declared through:

```kotlin
commonMain.dependencies {
    implementation(libs.some.library)
}
```

This is usually easier to understand than manually manipulating generated configuration names.

The modern KMP DSL encourages source-set-oriented configuration.

---

## 14. `commonTestImplementation`

Testing dependencies belong in:

```text
commonTest
```

when they support shared tests.

For example:

```kotlin
commonTest.dependencies {
    implementation(kotlin("test"))
}
```

The exact testing dependencies depend on the project.

The principle is:

```text
Production dependency → Main source set
Testing dependency → Test source set
```

---

## 15. Android Test Dependencies

Android-specific tests may require:

```text
Android testing APIs
Android test framework
Instrumentation dependencies
```

These belong in the appropriate Android test source set/configuration.

Do not place Android test infrastructure into:

```text
commonTest
```

unless the dependency genuinely supports common testing.

---

## 16. iOS Test Dependencies

Similarly, iOS-specific testing requirements should stay within the appropriate iOS test source set or framework configuration.

The goal is to avoid contaminating shared test code with platform-specific assumptions.

---

## 17. Dependency Graph Example

Consider:

```text
commonMain
 ├── kotlinx.serialization
 ├── networking
 └── coroutines
```

Then:

```text
androidMain
 └── Android-specific adapter

iosMain
 └── Apple-specific adapter

desktopMain
 └── Desktop-specific adapter

wasmJsMain
 └── Browser-specific adapter
```

The final graph is:

```text
                         shared
                           │
        ┌──────────────────┼──────────────────┐
        ▼                  ▼                  ▼
   Serialization       Networking         Coroutines
        │                  │                  │
        └──────────────────┼──────────────────┘
                           │
                      commonMain
                           │
        ┌────────────┬─────┼──────┬────────────┐
        ▼            ▼     ▼      ▼            ▼
     Android        iOS  Desktop  Web        Tests
```

---

## 18. Transitive Dependencies

A dependency can bring additional dependencies with it.

For example:

```text
Library A
   │
   ├── Library B
   │
   └── Library C
```

Gradle resolves the transitive dependency graph.

In KMP, this becomes more interesting because each dependency must be compatible with the target consuming it.

---

## 19. Why Transitive Dependencies Matter in KMP

A library may appear to support:

```text
commonMain
```

but introduce a transitive dependency that does not support one of your targets.

The result can be:

```text
Android ✓
iOS ✓
Desktop ✓
Web ✗
```

Therefore, target compatibility must be considered for the complete dependency graph, not just the direct dependency.

---

## 20. Dependency Metadata

KMP relies heavily on published metadata to understand which platforms and variants a library supports.

Conceptually:

```text
Library
   │
   ├── Common metadata
   ├── JVM variant
   ├── Android variant
   ├── Native variants
   └── Wasm/other variants
```

Gradle selects the appropriate variant for the consuming target.

---

## 21. Variant-Aware Dependency Resolution

Modern Gradle dependency resolution is variant-aware.

Instead of simply asking:

```text
Give me Library X.
```

Gradle effectively determines:

```text
I need Library X
for this platform
with these attributes
and these capabilities.
```

This is particularly important for KMP.

---

## 22. One Library, Multiple Variants

A KMP library may publish:

```text
common metadata
Android artifact
iOS native artifacts
JVM artifact
Wasm artifact
```

A consumer does not manually select every artifact.

Gradle resolves the appropriate variant.

Conceptually:

```text
Your KMP project
       │
       ▼
Dependency request
       │
       ▼
Gradle resolution
       │
 ┌─────┼───────────────┐
 ▼     ▼               ▼
Android iOS          Wasm
artifact native      artifact
```

---

## 23. Dependency Resolution Is Target-Specific

When building Android:

```text
Android dependency graph
```

is resolved.

When building iOS:

```text
iOS dependency graph
```

is resolved.

When building WebAssembly:

```text
Wasm dependency graph
```

is resolved.

They are related, but they are not necessarily identical.

---

## 24. Why "It Builds on Android" Is Not Enough

A common KMP mistake is:

```text
Add dependency
↓
Android build succeeds
↓
Assume KMP support is complete
```

This is unsafe.

The correct validation is:

```text
Android ✓
iOS ✓
Desktop ✓
Web ✓
Tests ✓
```

for the targets the project actually supports.

---

## 25. Dependency Compatibility Matrix

A useful engineering tool is a simple matrix:

| Dependency | Android | iOS | Desktop | Wasm |
|---|:---:|:---:|:---:|:---:|
| Serialization | ✓ | ✓ | ✓ | ✓* |
| Networking | ✓ | ✓ | ✓ | ✓* |
| Android API | ✓ | — | — | — |
| Browser API | — | — | — | ✓ |
| Native Apple API | — | ✓ | — | — |

`*` Always verify compatibility for the exact library version and project configuration.

This prevents assumptions.

---

## 26. Dependency Resolution Failure

A typical failure can look conceptually like:

```text
Could not resolve dependency
```

But the underlying reason may be:

```text
No compatible variant
Unsupported target
Missing metadata
Incorrect repository
Version conflict
Unsupported platform
Transitive dependency issue
```

The first step is to identify the target that is failing.

---

## 27. "No Matching Variant"

One common Gradle problem is a variant mismatch.

Conceptually:

```text
Consumer requires:
Wasm-compatible variant

Library provides:
JVM
Android
iOS
```

Gradle cannot select a compatible artifact.

The solution is not always to change Gradle configuration.

Sometimes the dependency simply does not support that target.

---

## 28. Unsupported Dependency

If a library is Android-only:

```text
Android ✓
iOS ✗
Desktop ✗
Wasm ✗
```

you cannot make it multiplatform merely by putting it in:

```text
commonMain
```

Instead:

```text
common abstraction
        ↑
androidMain implementation
```

may be required.

---

## 29. `expect/actual` and Dependencies

A common pattern is:

```kotlin
// commonMain

expect class PlatformService
```

and:

```kotlin
// androidMain

actual class PlatformService
```

The actual implementation can use an Android-only dependency.

Similarly:

```text
iosMain
desktopMain
wasmJsMain
```

can provide their own implementations.

The dependency stays with the platform implementation that needs it.

---

## 30. Dependency Injection Alternative

Instead of using `expect/actual`, dependency injection can provide platform services:

```kotlin
interface PlatformService {
    fun execute()
}
```

Then:

```text
Android → AndroidPlatformService
iOS → IOSPlatformService
Web → WasmPlatformService
```

The dependency is declared where the implementation lives.

This often keeps the common architecture explicit.

---

## 31. Choosing Between `expect/actual` and DI

There is no universal rule.

Use `expect/actual` when:

```text
The concept is fundamentally platform-specific
```

Dependency injection can be attractive when:

```text
The application already has a clear dependency graph
The abstraction is testable
Multiple implementations are useful
```

The important point is that platform dependencies remain at platform boundaries.

---

## 32. Dependency Resolution and Source Set Design

Dependency problems often reveal architecture problems.

For example:

```text
commonMain
   ↓
Android-only library
```

is usually a sign that a platform boundary is missing.

A better design is:

```text
commonMain
   ↓
Abstraction
   ↑
androidMain
   ↓
Android-only library
```

Dependency resolution can therefore act as an architectural feedback mechanism.

---

## 33. Version Conflicts

KMP projects can have many dependencies.

For example:

```text
Library A → Serialization 1.x
Library B → Serialization 2.x
```

Gradle must select compatible versions according to dependency resolution rules.

Version conflicts can become more complicated when different targets have different dependency graphs.

---

## 34. Version Alignment

Use consistent versions for related libraries where appropriate.

For example:

```text
Kotlin
KMP libraries
Serialization
Coroutines
Compose
```

should be managed deliberately.

Centralized version management reduces accidental drift.

---

## 35. Version Catalogs

A version catalog can centralize dependency coordinates.

Example:

```toml
[versions]
coroutines = "..."
serialization = "..."

[libraries]
kotlinx-coroutines = { ... }
kotlinx-serialization = { ... }
```

Then source sets can use aliases:

```kotlin
commonMain.dependencies {
    implementation(libs.kotlinx.coroutines)
}
```

This improves consistency.

---

## 36. Version Catalogs Do Not Solve Compatibility

A version catalog answers:

```text
Which version?
```

It does not answer:

```text
Does this dependency support Wasm?
Does it support iOS?
Is it appropriate for commonMain?
```

Those remain architectural and technical decisions.

---

## 37. Dependency Constraints

Gradle can use dependency constraints to influence resolution.

This can be useful when:

```text
Multiple libraries request different versions
```

But constraints should be introduced intentionally.

Do not use dependency constraints to hide fundamental incompatibilities.

---

## 38. Repositories

Gradle needs repositories from which dependencies can be resolved.

Common repositories include:

```text
Maven Central
Google Maven
Private company repositories
```

Repository configuration should be centralized where possible.

A dependency that exists in one repository but not another can produce confusing resolution failures.

---

## 39. Repository Management

A project should clearly define:

```text
Which repositories are trusted?
Where are internal artifacts stored?
Which repository provides a dependency?
```

Avoid adding random repositories simply to make a build pass.

Every repository increases the dependency supply-chain surface.

---

## 40. Dependency Verification

For production projects, dependency integrity matters.

Gradle supports mechanisms for dependency verification.

This can help detect unexpected changes to resolved artifacts.

A mature project should consider:

```text
Dependency versions
Artifact integrity
Repository trust
Supply-chain security
```

---

## 41. Dependency Locking

Dependency locking can make builds more reproducible.

Instead of allowing dependency versions to change unexpectedly:

```text
Resolved versions
       ↓
Locked
       ↓
Repeatable builds
```

This becomes especially useful in large KMP projects.

---

## 42. Reproducible Builds

A good KMP build should aim for:

```text
Same source
+
Same dependency graph
+
Same toolchain
=
Predictable build
```

This becomes increasingly important as the number of targets increases.

---

## 43. Gradle Cache

Gradle caches resolved dependencies and build information.

This improves build performance.

However, when debugging dependency issues, developers sometimes need to understand whether the problem comes from:

```text
Dependency metadata
Cache
Repository
Version conflict
Configuration
```

Use cache-clearing commands carefully rather than as the first solution to every build failure.

---

## 44. Dependency Insight

Gradle provides dependency inspection capabilities.

A dependency report can answer:

```text
Which dependencies are present?
Which version was selected?
Which dependency introduced it?
```

This is extremely useful when debugging KMP resolution.

Conceptually:

```text
Your dependency
      │
      └── Library A
             │
             └── Library B
                    │
                    └── Version conflict
```

Dependency insight helps expose this graph.

---

## 45. Dependency Trees

A dependency tree can help answer:

```text
Why is this library included?
Why did Gradle select this version?
Which library brought it in?
```

For large KMP projects, dependency inspection should become a normal engineering skill.

---

## 46. Target-Specific Dependency Trees

Because KMP has multiple targets, investigate the failing target.

For example:

```text
Android build
```

may resolve successfully while:

```text
Wasm build
```

fails.

The relevant dependency graph is therefore the Web/Wasm graph.

Do not assume the Android graph explains the problem.

---

## 47. Dependency Resolution and Build Variants

Android developers are familiar with:

```text
debug
release
```

KMP adds target-specific dimensions.

A dependency may need to be resolved differently depending on:

```text
Target
Source set
Compilation
Test configuration
Platform
```

Understanding these dimensions prevents many build surprises.

---

## 48. Common Dependency vs Intermediate Source Set

Large KMP projects may introduce intermediate source sets.

For example:

```text
commonMain
    │
    ├── appleMain
    │      ├── iosMain
    │      └── macosMain
    │
    └── jvmMain
           ├── androidMain
           └── desktopMain
```

A dependency shared by only a subset of targets can be placed in the appropriate intermediate source set.

This is more precise than putting everything in `commonMain`.

---

## 49. Intermediate Source Sets

Imagine a dependency works across:

```text
iOS
macOS
```

but not Android or Web.

Instead of:

```text
iosMain
    └── Library

macosMain
    └── Library
```

you can potentially have:

```text
appleMain
    └── Library
```

with:

```text
iosMain
macosMain
```

inheriting the dependency through the source-set hierarchy.

---

## 50. Dependency Reuse Through Intermediate Source Sets

This is one of the most powerful patterns for large KMP projects.

Instead of:

```text
iosMain → Library
macosMain → Library
```

you can potentially have:

```text
appleMain → Library
```

with:

```text
iosMain
macosMain
```

inheriting the dependency through the source-set hierarchy.

---

## 51. Dependency Placement as an Optimization

A useful hierarchy is:

```text
Highest possible reuse
        ↑
     commonMain
        ↑
 Intermediate source set
        ↑
   Platform source set
        ↑
   Target-specific code
        ↓
Lowest possible reuse
```

Place dependencies at the highest valid level.

---

## 52. Avoid Over-Generalization

The opposite mistake is placing dependencies too high.

For example:

```text
Android-only dependency
        ↓
commonMain
```

This creates unnecessary constraints.

A good KMP architecture balances:

```text
Reuse
+
Compatibility
+
Platform capability
```

---

## 53. Dependency Resolution Across Native Targets

Native targets can have more complex dependency resolution because native libraries may need platform-specific artifacts.

For example:

```text
iOS
    ├── iosArm64
    ├── iosSimulatorArm64
    └── other supported Apple architectures
```

The dependency must support the architectures required by the project.

---

## 54. Architecture-Specific Compatibility

A dependency may support:

```text
iosArm64
```

but have limitations elsewhere.

Therefore, native compatibility should be checked across the actual architectures your application builds.

The same principle applies to other native targets.

---

## 55. Framework Dependencies

When producing iOS frameworks, dependency handling can become important.

The final framework may need to account for:

```text
KMP code
Native dependencies
Framework packaging
Linking
Apple platform requirements
```

A dependency that works during compilation still needs to be validated in the final packaging process.

---

## 56. Linker Errors

Native dependency problems may appear as linker errors rather than simple Gradle resolution failures.

For example:

```text
Compilation succeeds
        ↓
Linking fails
        ↓
Native dependency problem
```

This is why:

```text
Resolved successfully
```

does not always mean:

```text
Fully compatible at runtime.
```

---

## 57. Runtime Compatibility

A dependency can pass:

```text
Gradle resolution
```

and still fail because of:

```text
Runtime behavior
Native API differences
Browser restrictions
Missing platform capability
Incorrect initialization
```

Dependency evaluation must therefore include:

```text
Build
+
Runtime
+
Target behavior
```

---

## 58. Dependency Initialization

Some platform dependencies require initialization.

For example:

```text
Browser SDK
Android SDK
Apple framework
```

The initialization code should be placed in the appropriate platform composition root.

Do not make `commonMain` responsible for platform startup details.

---

## 59. Dependency Lifecycle

Platform-specific dependencies may also have lifecycle requirements.

For example:

```text
Create
Initialize
Observe
Dispose
```

The lifecycle should be controlled by the appropriate platform layer.

---

## 60. Dependency Injection and Source Sets

A strong architecture can look like:

```text
commonMain
    │
    ├── Repository
    ├── UseCase
    └── Service interface
             ▲
             │
    ┌────────┼────────┐
    │        │        │
 Android    iOS      Web
    │        │        │
   DI       DI       DI
```

Each platform registers its own implementation.

---

## 61. Dependency Resolution vs Dependency Injection

These concepts are related but different.

### Dependency resolution

Gradle answers:

```text
Which library/artifact should be available?
```

### Dependency injection

Your application answers:

```text
Which implementation should this object receive?
```

For example:

```text
Gradle
  ↓
Android storage library
  ↓
AndroidStorage implementation
  ↓
DI
  ↓
Storage interface
```

---

## 62. Dependency Scope

Dependencies should be scoped according to their usage.

Examples:

```text
Production
Test
Platform
Shared
Build logic
```

Avoid making every dependency globally available.

Explicit scope improves maintainability.

---

## 63. Build Logic Dependencies

Gradle convention plugins may have their own dependencies.

These are different from application dependencies.

For example:

```text
build-logic
    ↓
Gradle plugin API
    ↓
Build configuration
```

Do not confuse:

```text
Build-time dependency
```

with:

```text
Application runtime dependency
```

---

## 64. Plugin Dependencies

KMP itself relies on Gradle plugins.

For example:

```text
Kotlin Multiplatform plugin
Android plugin
Compose plugin
Serialization plugin
```

These influence how Gradle creates targets and configurations.

Plugin versions should therefore be managed carefully.

---

## 65. Dependency Resolution and Plugins

A plugin can create:

```text
Targets
Source sets
Configurations
Tasks
Variants
```

Therefore, changing a plugin version can affect dependency resolution behavior.

This is another reason to keep:

```text
Kotlin
Gradle
Android Gradle Plugin
KMP-related plugins
```

compatible.

---

## 66. KMP Library Consumption

When consuming another KMP library, check:

```text
Supported targets
Published variants
Kotlin version
Dependency requirements
Platform limitations
```

Do not assume that:

```text
"It is a KMP library"
```

means:

```text
"It supports every KMP target."
```

---

## 67. Library Target Matrix

Before adopting a library, create a simple matrix:

```text
                    Android   iOS   Desktop   Wasm
Library A              ✓       ✓      ✓        ✓
Library B              ✓       ✓      —        —
Library C              ✓       —      —        —
```

This quickly reveals whether the dependency belongs in:

```text
commonMain
```

or a narrower source set.

---

## 68. Dependency Resolution and Architecture

Dependency selection should follow architecture.

Not:

```text
Library first
Architecture later
```

Instead:

```text
Requirement
    ↓
Abstraction
    ↓
Supported implementation
    ↓
Source set
    ↓
Dependency
```

This sequence prevents libraries from dictating the architecture unnecessarily.

---

## 69. Dependency Selection Checklist

Before adding a dependency, ask:

```text
Does it support my target?
Does it support my Kotlin version?
Does it support my Gradle setup?
Does it support native/Wasm where required?
Does it introduce platform-specific APIs?
Does it introduce large transitive dependencies?
Is it actively maintained?
Is the license appropriate?
Is it secure and trustworthy?
```

---

## 70. Open Source Dependency Evaluation

KMP projects often rely on open-source libraries.

Evaluate:

```text
License
Maintenance
Release history
Issue activity
Target support
Documentation
Security history
Community adoption
```

Do not select a dependency solely because it solves one small problem.

---

## 71. Avoid Dependency Sprawl

A project becomes harder to maintain when every feature introduces another library.

For example:

```text
Storage library
Network library
Date library
JSON library
Logging library
DI library
Image library
UI library
Navigation library
```

Every dependency introduces:

```text
Version management
Compatibility
Security
Build cost
Migration cost
```

Use dependencies deliberately.

---

## 72. Dependency Size Matters

For Web applications especially, dependency size can affect:

```text
Download size
Startup
Memory
Runtime performance
Caching
```

A dependency that is reasonable on a mobile device may have different implications for a Web application.

---

## 73. WebAssembly Dependency Constraints

When targeting WebAssembly, check:

```text
Wasm support
JavaScript interoperability
Browser support
Binary size
Initialization behavior
```

A dependency that works perfectly on JVM may not be usable on Wasm.

---

## 74. Common Mistake: Treating `commonMain` as "Default"

A common misconception is:

> "Everything should go into `commonMain`."

That is not the purpose of `commonMain`.

Its purpose is:

> **Code that is genuinely reusable across the relevant targets.**

Platform-specific code should remain platform-specific.

---

## 75. Common Mistake: Duplicating Dependencies

Another mistake is:

```text
androidMain → Library A
iosMain → Library A
desktopMain → Library A
```

when the library actually supports all those targets.

If the library is genuinely common, move it to:

```text
commonMain
```

This reduces duplication.

---

## 76. Common Mistake: Ignoring Transitive Dependencies

A developer verifies:

```text
Library A supports iOS
```

but does not check:

```text
Library A
   ↓
Library B
   ↓
Unsupported dependency
```

Always consider the complete dependency graph.

---

## 77. Common Mistake: Solving Resolution Errors by Downgrading Randomly

When Gradle reports an error:

```text
Could not resolve...
```

randomly changing versions can create more problems.

Instead identify:

```text
Target
Source set
Requested dependency
Selected variant
Missing capability
```

Then fix the actual incompatibility.

---

## 78. Common Mistake: Using Android Dependencies in Shared Code

For Android developers moving into KMP, this is especially common.

Code such as:

```text
Android Context
Android SharedPreferences
Android URI
Android Build APIs
```

should not automatically enter:

```text
commonMain
```

Create an abstraction.

---

## 79. Common Mistake: Assuming JavaScript Support Equals Wasm Support

A dependency may support:

```text
Kotlin/JS
```

without providing the same support for:

```text
Kotlin/Wasm
```

Always verify the exact target.

---

## 80. Common Mistake: Testing Only Android

A KMP project can look healthy when:

```text
Android build ✓
```

while:

```text
iOS ✗
Wasm ✗
```

fails.

For every shared dependency, validate the targets that matter to your product.

---

## 81. Dependency Resolution as a Design Signal

A useful architectural principle is:

> **When Gradle tells you a dependency cannot be resolved for a target, listen to what the dependency graph is telling you.**

Sometimes the correct solution is not:

```text
Change Gradle
```

but:

```text
Change the source-set boundary.
```

---

## 82. Practical Example

Suppose your shared application needs secure storage.

Bad approach:

```text
commonMain
    ↓
Android-specific secure storage library
```

Better approach:

```text
commonMain
    ↓
SecureStorage interface
    ↑
    ├── androidMain
    ├── iosMain
    └── wasmJsMain
```

Each target can select an appropriate implementation.

---

## 83. Practical Example: Analytics

Analytics is another useful example.

The shared layer can define:

```text
Analytics
```

The platform implementations can integrate with:

```text
Android analytics SDK
iOS analytics SDK
Web analytics SDK
```

The business layer remains independent.

---

## 84. Practical Example: Logging

Logging is often a good candidate for common code.

If the selected logging library supports your targets:

```text
commonMain
    ↓
Logging
```

The library can handle target-specific implementation internally.

If it does not support a target, introduce a platform abstraction or select another library.

---

## 85. Practical Example: Database

A database dependency requires more careful evaluation.

Ask:

```text
Does it support Android?
Does it support iOS?
Does it support Desktop?
Does it support Wasm?
```

If the database is not available everywhere:

```text
common repository abstraction
        ↑
platform-specific storage implementation
```

may be the better architecture.

---

## 86. Practical Example: Network Client

A multiplatform HTTP client can often live in:

```text
commonMain
```

while target-specific engines are selected underneath.

Conceptually:

```text
common API
    │
    ▼
HTTP client
    │
 ┌──┼─────────────┐
 ▼  ▼             ▼
Android iOS      Wasm
engine  engine   engine
```

This is an excellent example of dependency resolution working with KMP architecture.

---

## 87. Dependency Graph Mental Model

Always visualize:

```text
Application
     │
     ▼
Source Set
     │
     ▼
Dependency
     │
     ▼
Variant
     │
     ▼
Transitive Dependencies
     │
     ▼
Target
```

If any layer is incompatible, the build can fail.

---

## 88. Final Dependency Resolution Model

A mature KMP project should look conceptually like:

```text
                         Project
                            │
                     Source-set graph
                            │
          ┌─────────────────┼──────────────────┐
          ▼                 ▼                  ▼
     commonMain       intermediate sets    platform sets
          │                 │                  │
          └─────────────────┼──────────────────┘
                            ▼
                    Dependency requests
                            │
                            ▼
                     Gradle resolution
                            │
                            ▼
                    Compatible variants
                            │
                            ▼
                  Target-specific artifacts
                            │
          ┌─────────────┬────┼────┬─────────────┐
          ▼             ▼    ▼    ▼             ▼
       Android         iOS  JVM  Wasm          Native
```

This is the foundation for understanding KMP dependency behavior.

---

# Chapter Takeaways

> [!IMPORTANT]
> **Dependency resolution in KMP is target-aware. The right dependency is not merely the library you want—it is the compatible variant that belongs at the correct source-set level.**

Remember:

1. Dependency resolution determines which libraries and variants Gradle provides.
2. KMP dependency resolution is target-aware.
3. Source-set placement controls dependency visibility.
4. Put a dependency in the highest source set that can legitimately use it.
5. `commonMain` should contain genuinely multiplatform dependencies.
6. Android-only dependencies belong in `androidMain`.
7. iOS-specific dependencies belong in the appropriate iOS source set.
8. WebAssembly-specific dependencies belong in `wasmJsMain`.
9. Test dependencies should remain in test source sets.
10. A child source set can inherit dependencies from its parent according to the source-set hierarchy.
11. A dependency in `androidMain` does not automatically become available to iOS.
12. `commonMain` dependencies can constrain every target that consumes them.
13. Transitive dependencies matter as much as direct dependencies.
14. A KMP library can publish multiple platform variants.
15. Gradle selects compatible variants using dependency metadata and attributes.
16. Android success does not prove KMP compatibility.
17. Validate every target that your product supports.
18. "No matching variant" usually indicates a compatibility problem.
19. A platform-only library cannot be made common simply by moving its declaration.
20. `expect/actual` can isolate platform-specific implementations.
21. Dependency injection can also isolate platform-specific implementations.
22. Version catalogs centralize dependency coordinates.
23. Version catalogs do not guarantee target compatibility.
24. Repository configuration affects dependency resolution.
25. Avoid adding untrusted repositories just to resolve a library.
26. Dependency verification can improve supply-chain security.
27. Dependency locking can improve build reproducibility.
28. Dependency reports help diagnose resolution problems.
29. Dependency insight helps identify why a version was selected.
30. Debug the dependency graph for the failing target.
31. Native dependency resolution may involve architecture-specific artifacts.
32. Successful compilation does not guarantee successful native linking.
33. Successful resolution does not guarantee runtime compatibility.
34. Platform dependencies may require platform-specific initialization.
35. Keep platform initialization in the platform composition root.
36. Intermediate source sets can share dependencies across a subset of targets.
37. Intermediate source sets can reduce duplication.
38. Dependency placement should follow the source-set hierarchy.
39. Do not put platform-specific dependencies into `commonMain` merely for convenience.
40. Do not duplicate a genuinely common dependency across multiple platform source sets.
41. Check transitive dependencies before adopting a library.
42. Check target support before adding a dependency.
43. Check Kotlin and Gradle compatibility.
44. Check native architecture compatibility where applicable.
45. Check Wasm compatibility explicitly.
46. Kotlin/JS support does not automatically imply Kotlin/Wasm support.
47. JVM support does not imply native or Wasm support.
48. Build dependencies and application dependencies are different.
49. Plugin dependencies influence project configuration and available targets.
50. Dependency resolution and dependency injection solve different problems.
51. Dependency resolution answers **what is available**.
52. Dependency injection answers **what implementation the application receives**.
53. A common abstraction can hide platform-specific libraries.
54. Shared business logic should not depend directly on platform libraries.
55. Source-set boundaries are architectural boundaries.
56. Dependency failures can reveal misplaced abstractions.
57. Avoid dependency sprawl.
58. Evaluate open-source dependencies for license, maintenance, security, and target support.
59. Web dependencies should be evaluated for binary size and browser behavior.
60. Use measurement rather than assumptions when evaluating dependency cost.
61. Avoid random version changes when debugging Gradle failures.
62. Identify the failing target before changing configuration.
63. Inspect the resolved dependency graph.
64. Verify the selected variant.
65. Verify repositories.
66. Verify transitive dependencies.
67. Verify runtime behavior after resolution succeeds.
68. Treat dependency resolution as part of architecture.
69. The highest valid source set usually provides the greatest reuse.
70. The narrowest valid source set usually provides the strongest isolation.
71. **Good KMP dependency management balances reuse, compatibility, platform capability, and maintainability.**

---

## Final Thought

Dependency management in Kotlin Multiplatform is not simply a Gradle configuration problem.

It is an architectural problem.

The strongest KMP projects make dependency boundaries visible:

```text
                    commonMain
                        │
              Shared dependencies
                        │
          ┌─────────────┼─────────────┐
          ▼             ▼             ▼
     androidMain     iosMain      wasmJsMain
          │             │             │
     Android libs   Apple libs    Web/Wasm libs
```

The objective is not to force every dependency into `commonMain`.

The objective is to place every dependency **where it provides maximum meaningful reuse without breaking platform independence**.

> **A well-designed KMP dependency graph is a reflection of a well-designed architecture.**
