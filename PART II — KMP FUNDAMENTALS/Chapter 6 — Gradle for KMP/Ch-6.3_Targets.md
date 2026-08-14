# Chapter 6 — Gradle for KMP

## Part 3 — Targets

> **A KMP target answers one fundamental question: where should this Kotlin code be compiled and run?**

Once the Kotlin Multiplatform plugin is applied, one of the most important concepts in the Gradle configuration is the **target**.

You may see declarations such as:

```kotlin
kotlin {
    androidTarget()

    iosArm64()
    iosSimulatorArm64()
}
```

These declarations are not simply a list of platforms.

They tell the Kotlin Multiplatform build model which compilation destinations the module supports.

That distinction matters.

A KMP module is not automatically "Android and iOS" just because it contains shared Kotlin code.

The project must define its targets.

---

# 1. What Is a Target?

A target represents a platform or environment for which Kotlin code can be compiled.

A simplified model is:

```text
                    KMP Module
                        │
          ┌─────────────┼─────────────┐
          ▼             ▼             ▼
       Android      iOS Device    iOS Simulator
```

Each target represents a different compilation environment.

For example:

```text
Android
```

means the Kotlin code is compiled for Android.

While:

```text
iosArm64
```

represents an iOS device target using the ARM64 architecture.

And:

```text
iosSimulatorArm64
```

represents an iOS simulator target running on an ARM64 Mac.

The names matter because a target is more precise than simply saying:

```text
iOS
```

---

# 2. Why Does KMP Need Targets?

Consider this shared code:

```kotlin
class UserRepository {
    fun getUser(): User {
        // shared implementation
    }
}
```

The compiler needs to know:

```text
Where should this code be compiled?
```

A KMP project might answer:

```text
Android
iOS device
iOS simulator
```

So the build model becomes:

```text
                  commonMain
                      │
          ┌───────────┼───────────┐
          ▼           ▼           ▼
       Android     iOS Device   iOS Simulator
```

The same shared source can participate in multiple platform compilations.

---

# 3. Target Is Not the Same as Platform

This is a subtle but important distinction.

Developers often say:

```text
Android target
iOS target
```

That is useful shorthand, but technically a target identifies a specific compilation environment.

For example:

```text
iOS
```

can involve:

```text
Physical iPhone / iPad
iOS Simulator
```

and those environments can have different architectures.

So:

```text
Platform
   │
   └── Target(s)
```

is often a better mental model.

---

# 4. iOS Has More Than One Target

A typical modern KMP project may configure:

```kotlin
kotlin {
    iosArm64()
    iosSimulatorArm64()
}
```

This represents:

```text
iOS
├── Physical device
│    └── ARM64
│
└── Simulator
     └── ARM64
```

Why separate them?

Because the code is being compiled for different environments.

The simulator and a physical device are not simply interchangeable compilation destinations.

---

# 5. Architecture Matters

Target declarations can encode CPU architecture.

For example:

```text
iosArm64
```

indicates:

```text
iOS
ARM64
```

while:

```text
iosSimulatorArm64
```

indicates:

```text
iOS Simulator
ARM64
```

Architecture matters because native binaries must match the environment in which they execute.

Conceptually:

```text
Source Code
    │
    ▼
Target
    │
    ▼
Architecture
    │
    ▼
Native Binary
```

---

# 6. Why the Simulator Needs Its Own Target

Suppose you develop on an Apple Silicon Mac.

Your iPhone target is:

```text
iosArm64
```

Your simulator target can be:

```text
iosSimulatorArm64
```

They both use ARM64, but they represent different environments.

Think:

```text
              ARM64
                │
        ┌───────┴────────┐
        ▼                ▼
   iOS Device       iOS Simulator
```

Same architecture does not mean same target.

---

# 7. Android as a KMP Target

Android can be configured as a KMP target using current Kotlin Multiplatform tooling.

A simplified example is:

```kotlin
kotlin {
    androidTarget()
}
```

This tells the KMP model that the shared module has an Android compilation target.

Conceptually:

```text
KMP Module
    │
    ▼
Android Target
    │
    ▼
Android Toolchain
    │
    ▼
Android Output
```

The exact Android integration depends on the Kotlin, Android Gradle Plugin, and project configuration versions.

---

# 8. Targets Define Compilation Destinations

A useful way to think about targets is:

```text
Source
  │
  ▼
Target Selection
  │
  ▼
Compilation
  │
  ▼
Platform Output
```

For example:

```text
commonMain
    │
    ├── Android → Android output
    │
    ├── iOS Device → Native output
    │
    └── iOS Simulator → Native output
```

This is the bridge between shared source code and platform-specific binaries.

---

# 9. Target Declarations Are Build Configuration

When you write:

```kotlin
kotlin {
    androidTarget()
    iosArm64()
    iosSimulatorArm64()
}
```

you are not writing application code.

You are describing the build model.

Think:

```text
Gradle Configuration
        │
        ▼
Target Model
        │
        ▼
Compilation Tasks
        │
        ▼
Platform Artifacts
```

The target declarations tell the tooling what should exist.

---

# 10. Targets and Source Sets

Targets and source sets work together.

A simplified model is:

```text
                 KMP Module
                     │
            ┌────────┴────────┐
            ▼                 ▼
       Source Sets          Targets
            │                 │
            └────────┬────────┘
                     ▼
                Compilations
```

For example:

```text
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

The source set tells you **which code**.

The target tells you **where it is compiled**.

---

# 11. The Most Important Distinction

Remember:

> **Source set = code organization.**

> **Target = compilation destination.**

For example:

```text
commonMain
```

means:

```text
Shared source
```

while:

```text
iosArm64
```

means:

```text
Compile for an iOS ARM64 device target.
```

This distinction will make the rest of KMP much easier to understand.

---

# 12. One Source, Multiple Targets

One of KMP's central ideas is:

```text
                    commonMain
                        │
          ┌─────────────┼─────────────┐
          ▼             ▼             ▼
       Android      iOS Device    iOS Simulator
```

The source is written once.

The build system creates the appropriate compilation paths for the configured targets.

This is where the phrase:

> "Write once, share where it makes sense"

becomes concrete.

---

# 13. Shared Code Does Not Mean Shared Binary

This is an important distinction.

Suppose:

```text
commonMain
```

contains:

```kotlin
class CartCalculator
```

That source may be compiled for:

```text
Android
iOS Device
iOS Simulator
```

But the resulting outputs are not one universal binary.

Conceptually:

```text
                  CartCalculator
                        │
          ┌─────────────┼─────────────┐
          ▼             ▼             ▼
       Android       iOS Device    iOS Simulator
          │             │             │
          ▼             ▼             ▼
     Android        Native Output   Native Output
      Output
```

The source is shared.

The compilation outputs are target-specific.

---

# 14. Why This Matters Architecturally

This explains why KMP is different from simply shipping one cross-platform runtime.

The project can share:

```text
Domain logic
Business rules
Networking abstractions
Data models
Validation
Persistence abstractions
```

while each target still produces native-oriented output appropriate to that platform.

The target model is what makes this possible.

---

# 15. Target Hierarchies

Modern KMP projects can also use hierarchical source-set relationships.

A simplified example:

```text
commonMain
    │
    ▼
appleMain
    │
    ├── iosArm64
    └── iosSimulatorArm64
```

This is useful when some code is shared across Apple targets but is not appropriate for every platform.

The idea is:

```text
Common
  │
  ▼
Apple
  │
  ├── iOS Device
  ├── iOS Simulator
  ├── macOS
  └── Other Apple Targets
```

The exact source-set hierarchy depends on the targets configured in the project.

---

# 16. Why Hierarchical Source Sets Matter

Imagine:

```text
Code A
```

works everywhere:

```text
Android
iOS
macOS
```

That belongs naturally in:

```text
commonMain
```

Now imagine:

```text
Code B
```

uses an Apple-specific API.

It should not be in:

```text
commonMain
```

But it might be shared across:

```text
iOS
macOS
```

An intermediate Apple-oriented source set can represent that boundary.

Conceptually:

```text
commonMain
    │
    ▼
appleMain
    │
    ├── iOS
    └── macOS
```

This avoids unnecessary duplication without pretending the code is universally portable.

---

# 17. Target-Specific Source Sets

At the platform edge, you can still have target-specific code.

For example:

```text
commonMain
     │
     ▼
appleMain
     │
     ├── iosMain
     │      ├── iosArm64
     │      └── iosSimulatorArm64
     │
     └── macosMain
```

The exact source-set hierarchy is generated and configured based on the targets in the project.

The important idea is:

```text
Share at the highest valid level.
Specialize at the lowest necessary level.
```

---

# 18. Target Selection Is an Architectural Decision

Adding a target is not always free.

When you add:

```text
iosArm64()
```

you are saying:

```text
This module must support compilation for this environment.
```

That affects:

```text
Compilation
Dependencies
Testing
Build time
CI
Native compatibility
```

So target selection should reflect actual product requirements.

---

# 19. Don't Add Every Target Automatically

It can be tempting to configure:

```text
Android
iOS
macOS
watchOS
tvOS
Linux
Windows
JavaScript
Wasm
```

just because Kotlin Multiplatform can support multiple environments.

But every additional target can increase:

```text
Build complexity
Dependency constraints
Testing requirements
CI workload
Maintenance
```

A better principle is:

> **Configure the targets your product actually needs.**

---

# 20. Target Support Is a Dependency Question Too

Suppose a library supports:

```text
Android
iOS
```

but not:

```text
macOS
```

If you add macOS as a target, that dependency may become a problem.

The graph becomes:

```text
New Target
    │
    ▼
Dependency Compatibility
    │
    ├── Supported
    └── Unsupported
```

This is why target decisions should not be made independently of dependency decisions.

---

# 21. Target Configuration and Libraries

Consider:

```text
commonMain
   │
   ├── Library A
   ├── Library B
   └── Library C
```

Now add:

```text
New Target
```

The question becomes:

```text
Do A, B, and C support the new target?
```

If one doesn't:

```text
Target
  │
  ▼
Compilation
  │
  ▼
Dependency Resolution
  │
  ▼
Failure
```

This is one of the most common reasons a previously working KMP build can fail after adding a target.

---

# 22. Targets and `expect` / `actual`

Target configuration becomes especially important when using:

```text
expect
actual
```

The simplified relationship is:

```text
commonMain
   │
   ▼
expect declaration
   │
   ├── Android → actual
   │
   └── iOS → actual
```

The target determines which platform-specific implementation participates in the compilation.

This is one of the reasons target understanding should come before deep `expect` / `actual` work.

---

# 23. A Simple Example

Common code:

```kotlin
expect fun platformName(): String
```

Android-specific implementation:

```kotlin
actual fun platformName(): String = "Android"
```

iOS-specific implementation:

```kotlin
actual fun platformName(): String = "iOS"
```

Conceptually:

```text
                  commonMain
                      │
                expect declaration
                      │
             ┌────────┴────────┐
             ▼                 ▼
          Android              iOS
             │                 │
           actual            actual
```

The target-specific compilations determine which `actual` implementation is used.

---

# 24. Target-Specific Dependencies

Sometimes a dependency belongs only to one target.

For example:

```text
Android-specific library
```

should generally be associated with Android source code rather than common code.

Conceptually:

```text
commonMain
     │
     └── Shared Dependency

androidMain
     │
     └── Android Dependency

iosMain
     │
     └── iOS Dependency
```

Target-aware dependency placement protects platform boundaries.

---

# 25. Target-Specific APIs

A platform API may exist only on one target.

For example:

```text
Android Context
```

is not an iOS concept.

Similarly:

```text
UIKit
```

is not an Android concept.

The target model helps you keep these APIs in the appropriate source-set boundary.

```text
Android API
    │
    ▼
androidMain

Apple API
    │
    ▼
iosMain / Apple-specific source set
```

---

# 26. Targets and Native Compilation

For native targets, Kotlin/Native participates in the compilation process.

A simplified pipeline is:

```text
Kotlin Source
      │
      ▼
KMP Target
      │
      ▼
Kotlin/Native
      │
      ▼
Native Compilation
      │
      ▼
Native Artifact
```

The exact artifact and integration depend on the target and project configuration.

---

# 27. Targets and Android Compilation

For Android, the build involves Android-specific tooling.

A simplified pipeline is:

```text
Kotlin Source
      │
      ▼
Android Target
      │
      ▼
Android Toolchain
      │
      ▼
Android Artifact
```

The complete Android build also involves resources, manifests, packaging, and Android application or library configuration.

---

# 28. Targets Are Part of the Build Graph

Think of the KMP build as a graph:

```text
                     KMP Module
                         │
             ┌───────────┼───────────┐
             ▼           ▼           ▼
          Android     iOS Device   iOS Simulator
             │           │           │
             ▼           ▼           ▼
        Compilation  Compilation  Compilation
             │           │           │
             ▼           ▼           ▼
           Output      Output       Output
```

Adding a target adds another branch to this graph.

That branch can introduce:

```text
New compilation
New dependencies
New tasks
New tests
New CI work
```

---

# 29. Target and Build Time

More targets generally mean more work.

For example:

```text
3 targets
   ↓
Multiple compilations
   ↓
More tasks
   ↓
More build work
```

This does not mean every target multiplies build time by a fixed amount.

Gradle caching, incremental compilation, task avoidance, and the project structure all matter.

The practical lesson is:

> **Every target has a build cost.**

---

# 30. Target and CI

A local developer may only test:

```text
Android
```

while CI should verify:

```text
Android
iOS device-related compilation
iOS simulator-related compilation
```

depending on the project's release requirements.

A target that exists in the build configuration but is never tested can become a hidden source of failures.

---

# 31. Target and Testing

Targets affect testing too.

You may have:

```text
commonTest
```

for shared tests.

Then platform-specific tests can exist where needed.

Conceptually:

```text
                 Tests
                   │
          ┌────────┴────────┐
          ▼                 ▼
     Common Tests       Platform Tests
          │                 │
          ▼           ┌─────┴─────┐
      commonTest    Android       iOS
```

Testing strategy should follow the target model.

---

# 32. A Target Is a Promise

When you declare:

```kotlin
iosArm64()
```

you are making a promise:

> **This module is intended to be compilable for that target.**

That means:

```text
Code
Dependencies
Compiler configuration
Native integrations
```

all need to be compatible with that target.

This is why adding targets should be intentional.

---

# 33. Target Names Tell a Story

When you see:

```text
iosArm64
```

you can infer:

```text
Platform → iOS
Architecture → ARM64
Environment → Physical device
```

When you see:

```text
iosSimulatorArm64
```

you can infer:

```text
Platform → iOS
Environment → Simulator
Architecture → ARM64
```

Learning to read target names is a simple but valuable KMP skill.

---

# 34. Common Target Families

Depending on the Kotlin Multiplatform version and project needs, you may encounter target families such as:

```text
Android
iOS
macOS
watchOS
tvOS
Linux
Windows
JavaScript
WebAssembly
```

Not every target has the same maturity, APIs, libraries, or project requirements.

Always evaluate the actual target support for the tooling and dependencies used by your project.

---

# 35. Don't Confuse Target Availability With Library Availability

A platform may be technically supported by Kotlin Multiplatform while a particular library does not support it.

For example:

```text
Kotlin target → supported
Library       → unsupported
```

The application may still fail to build.

Therefore:

```text
KMP Target Support
        ≠
Every Library Supports That Target
```

This distinction becomes increasingly important in real projects.

---

# 36. Target Support Matrix

A useful way to reason about dependencies is:

| Dependency | Android | iOS | macOS |
|---|:---:|:---:|:---:|
| Library A | ✓ | ✓ | ✓ |
| Library B | ✓ | ✓ | — |
| Library C | ✓ | — | — |

Now suppose your project adds:

```text
macOS
```

Library B and Library C become potential problems.

The target decision therefore changes the dependency compatibility matrix.

---

# 37. Target Configuration and Architecture

A strong KMP architecture usually follows this principle:

```text
Common
  │
  ├── Platform-independent logic
  │
  └── Stable abstractions
          │
          ▼
     Platform targets
          │
      ┌───┴───┐
      ▼       ▼
   Android    iOS
```

The target boundary becomes the natural place for platform-specific behavior.

---

# 38. Target Configuration Is Not UI Configuration

A target does not mean:

```text
Use Android UI
Use iOS UI
```

It means:

```text
Compile Kotlin for this environment.
```

UI sharing is a separate architectural decision.

You can have:

```text
KMP shared logic
+
Native UI
```

or:

```text
KMP shared logic
+
Compose Multiplatform UI
```

The target model supports the compilation environments; it does not dictate your UI strategy.

---

# 39. Targets and Compose Multiplatform

If a project uses Compose Multiplatform, the target configuration may support platforms where Compose is intended to run.

Conceptually:

```text
KMP
 │
 ├── Shared Logic
 │
 └── Compose Multiplatform
        │
        ├── Android
        └── iOS
```

The important point is:

```text
KMP target
```

and:

```text
Compose UI target
```

are related but conceptually different concerns.

---

# 40. Target Selection and Product Requirements

Suppose your product supports:

```text
Android phones
iPhones
```

Then your initial target set might focus on:

```text
Android
iOS device
iOS simulator
```

If later the product expands to:

```text
macOS
```

you can evaluate adding the relevant target.

The target set should evolve with the product rather than being determined by curiosity.

---

# 41. Targets and Incremental Adoption

One of KMP's strengths is that you do not necessarily need to migrate every platform at once.

A project can begin with:

```text
Android
```

and later introduce:

```text
iOS
```

as the shared module grows.

Conceptually:

```text
Phase 1
Android
   │
   ▼
Shared Module

Phase 2
Android + iOS
   │
   ▼
Expanded Target Model
```

The build configuration evolves with the migration.

---

# 42. Adding iOS to an Existing Android Project

A simplified migration can eventually result in:

```text
KMP Shared Module
       │
   ┌───┴────┐
   ▼        ▼
Android    iOS
```

The important work is not merely adding:

```kotlin
iosArm64()
```

You also need to evaluate:

```text
Dependencies
Source-set boundaries
Native APIs
Testing
Build integration
CI
```

The target declaration is the beginning of the work, not the entire migration.

---

# 43. Target-Specific Failures

A common situation is:

```text
Android → PASS
iOS → FAIL
```

When that happens, inspect:

```text
1. iOS target configuration
2. iOS-compatible dependencies
3. iOS source-set code
4. Native APIs
5. Compiler/toolchain compatibility
6. Framework or native integration
```

Do not assume the shared business logic itself is the problem.

The target-specific build path is a separate branch of the compilation graph.

---

# 44. Target-Specific Compilation Errors

Suppose:

```text
commonMain
```

contains a type that is available only on Android.

Android may compile:

```text
Android → PASS
```

while iOS reports:

```text
Unresolved reference
```

This is valuable information.

It means:

```text
The supposedly common code depends on an Android-specific API.
```

The target exposed the architectural mistake.

---

# 45. Target-Specific Dependency Errors

Another example:

```text
commonMain
    │
    ▼
Dependency X
    │
    ▼
Android → PASS
iOS → FAIL
```

Possible causes include:

```text
Dependency X lacks iOS support
Incorrect source-set placement
Incorrect version
Native compatibility issue
```

Again, the target is revealing the actual boundary.

---

# 46. Target Configuration and Build Files

When reading:

```kotlin
kotlin {
    ...
}
```

look for target declarations first.

For example:

```kotlin
kotlin {
    androidTarget()

    iosArm64()
    iosSimulatorArm64()
}
```

You can immediately draw:

```text
shared
  │
  ├── Android
  ├── iOS Device
  └── iOS Simulator
```

Then investigate how source sets and dependencies connect to those targets.

---

# 47. A Target-Centric Reading Strategy

When entering an unfamiliar KMP project, ask:

```text
What targets exist?
```

Then:

```text
What source sets exist?
```

Then:

```text
Which dependencies are shared?
```

Then:

```text
Which dependencies are target-specific?
```

Finally:

```text
How are the outputs consumed?
```

This gives you a practical understanding of the build.

---

# 48. Target-Centric Debugging

A useful debugging flow is:

```text
Build Failure
     │
     ▼
Which Target?
     │
     ├── Android
     │
     └── iOS
          │
          ▼
Which Source Set?
          │
          ▼
Which Dependency / API?
          │
          ▼
Which Toolchain?
```

This is much more effective than changing random Gradle versions.

---

# 49. Targets and Framework Generation

For iOS integration, a KMP shared module may produce native artifacts that can be consumed by the iOS application.

The exact integration depends on the project's current KMP configuration.

Conceptually:

```text
KMP Shared Module
        │
        ▼
iOS Target
        │
        ▼
Native Artifact / Framework Integration
        │
        ▼
iOS Application
```

The target is therefore part of the bridge between shared Kotlin and the Apple application.

---

# 50. Why Target Names May Change Over Time

Kotlin Multiplatform tooling evolves.

Target configuration APIs can be improved, renamed, deprecated, or replaced by newer approaches.

For example, Android target configuration has evolved over Kotlin releases.

Therefore, when writing build configuration:

```text
Follow the current Kotlin documentation
```

rather than blindly copying a configuration from an old project.

The architectural concept remains:

```text
Declare compilation targets
```

even when syntax changes.

---

# 51. Target APIs Are Tooling APIs

This is an important mindset.

Code such as:

```kotlin
androidTarget()
iosArm64()
```

is part of the build configuration API.

It is not application API.

Therefore, changes in:

```text
Kotlin
KMP plugin
Gradle
```

can affect these declarations without changing your business logic.

---

# 52. Target Configuration and Toolchain Evolution

The target model is stable conceptually:

```text
Where should this code compile?
```

But implementation details evolve:

```text
Target declarations
Plugin APIs
Compiler options
Native integration
Android integration
```

As a KMP developer, learn the model first and the syntax second.

That makes version upgrades much easier to understand.

---

# 53. A Target Is a Build Boundary

Think of each target as a boundary:

```text
                 Shared Logic
                      │
          ┌───────────┼───────────┐
          ▼           ▼           ▼
       Android       iOS        Other
          │           │           │
          ▼           ▼           ▼
      Platform     Platform     Platform
      APIs         APIs         APIs
```

The target tells you where platform assumptions become valid.

This is why targets are closely related to architecture.

---

# 54. Don't Over-Abstract Platform Differences

Suppose Android and iOS have genuinely different APIs.

You don't always need to hide every difference behind an enormous abstraction layer.

Sometimes the right structure is simply:

```text
commonMain
    │
    ▼
Shared abstraction
    │
    ├── androidMain
    └── iosMain
```

The target boundary gives you a clean place to implement the platform-specific behavior.

---

# 55. Targets and the Principle of Least Sharing

A useful principle is:

> **Share code when the behavior is genuinely common; specialize when the platform meaningfully differs.**

The target model supports this directly.

```text
Common behavior
      │
      ▼
commonMain

Platform behavior
      │
 ┌────┴────┐
 ▼         ▼
Android    iOS
```

This is healthier than trying to force identical implementations where the platforms are genuinely different.

---

# 56. Target Matrix for a Simple Mobile Product

For a product targeting Android and iOS, the build model might be:

| Area | Android | iOS Device | iOS Simulator |
|---|:---:|:---:|:---:|
| Shared Kotlin | ✓ | ✓ | ✓ |
| Platform APIs | ✓ | ✓ | ✓ |
| Shared Tests | ✓ | ✓ | ✓ |
| Android APIs | ✓ | — | — |
| Apple APIs | — | ✓ | ✓ |

This makes the role of targets concrete.

---

# 57. Target Matrix for a Growing Product

Suppose the product later adds macOS:

| Area | Android | iOS | macOS |
|---|:---:|:---:|:---:|
| Domain Logic | ✓ | ✓ | ✓ |
| Networking | ✓ | ✓ | ✓ |
| Apple-specific Logic | — | ✓ | ✓ |
| Android APIs | ✓ | — | — |
| macOS-only APIs | — | — | ✓ |

Now a hierarchy can become useful:

```text
commonMain
    │
    ▼
appleMain
    │
    ├── iOS
    └── macOS
```

The target model grows with the product.

---

# 58. What Happens When You Remove a Target?

Removing a target can also change the build.

For example:

```text
Before:
Android + iOS + macOS

After:
Android + iOS
```

This may remove:

```text
Compilation tasks
Dependency requirements
Native outputs
CI work
```

But it can also make some source sets or dependencies unnecessary.

Target changes should therefore be treated as project-level build changes.

---

# 59. Targets and Build Variants Are Different

Android developers may know:

```text
debug
release
free
paid
```

These are variants/configurations inside the Android build model.

A KMP target is different.

Think:

```text
Target
→ Where is the code compiled?

Variant
→ Which configuration of that platform build is being produced?
```

They can interact, but they are not the same concept.

---

# 60. Target vs Architecture

Another distinction:

```text
Target
```

may include architecture information, but target and architecture are not universally identical concepts.

For example:

```text
iosArm64
```

communicates both:

```text
iOS device
ARM64
```

while a platform can have multiple architectures or target environments.

The important thing is to understand what the specific target declaration represents in your project and tooling version.

---

# 61. Targets and Native Interop

When shared Kotlin needs to interact with platform APIs, the target becomes critical.

For iOS:

```text
Kotlin/Native
     │
     ▼
iOS Target
     │
     ▼
Apple SDK Interop
```

For Android:

```text
Kotlin
     │
     ▼
Android Target
     │
     ▼
Android APIs
```

The target determines the platform environment available during compilation.

---

# 62. Target-Specific Testing Should Be Intentional

If you add a target:

```text
macOS
```

but never compile or test it in CI, you may discover problems only after a developer needs it.

A useful rule is:

```text
Target declared
     ↓
Target compiled
     ↓
Target tested where practical
     ↓
Target maintained
```

A target should not become a forgotten branch of the build graph.

---

# 63. Target and CI Cost

Suppose:

```text
Android
iOS
macOS
```

are all targets.

CI may now need:

```text
Android checks
iOS checks
macOS checks
```

The exact workflow depends on the project.

The key lesson:

> **Every target creates an ongoing maintenance responsibility.**

---

# 64. Target Configuration and Developer Experience

A well-configured target model makes it obvious:

```text
What can this module build?
```

A poorly configured one may contain:

```text
Unused targets
Unsupported dependencies
Broken native configurations
Unclear source-set relationships
```

Good target configuration improves not only the build but also developer understanding.

---

# 65. Target Configuration Is a Form of Documentation

This is an underrated benefit.

When someone opens:

```kotlin
kotlin {
    androidTarget()
    iosArm64()
    iosSimulatorArm64()
}
```

they immediately know:

```text
This module targets Android and iOS.
```

The build file documents the platform scope of the module.

---

# 66. Target Configuration and Project Scope

A shared module may support:

```text
Android + iOS
```

while another module may be:

```text
Android-only
```

That distinction can be expressed through the build configuration.

For example:

```text
shared
 └── KMP targets

androidFeature
 └── Android tooling
```

This keeps platform responsibilities visible.

---

# 67. Target Selection Checklist

Before adding a new target, ask:

```text
[ ] Does the product need this platform?
[ ] Does the Kotlin toolchain support it?
[ ] Do our dependencies support it?
[ ] Does our architecture support it?
[ ] Can we test it?
[ ] Can CI build it?
[ ] Can we maintain it?
[ ] Does it provide enough value to justify the complexity?
```

This prevents target sprawl.

---

# 68. Target Debugging Checklist

When one target fails:

```text
[ ] Identify the failing target.
[ ] Check target declaration.
[ ] Check source-set hierarchy.
[ ] Check target-specific dependencies.
[ ] Check common dependencies for target support.
[ ] Check platform APIs.
[ ] Check compiler/toolchain versions.
[ ] Check native integration.
[ ] Reproduce with the smallest relevant Gradle task.
```

Start with the target rather than treating the entire KMP project as one build.

---

# 69. The Deeper Mental Model

Think of KMP as:

```text
                     Shared Source
                          │
                          ▼
                   Target Selection
                          │
             ┌────────────┼────────────┐
             ▼            ▼            ▼
          Android       iOS Device   iOS Simulator
             │            │            │
             ▼            ▼            ▼
        Compilation   Compilation   Compilation
             │            │            │
             ▼            ▼            ▼
          Platform     Native       Native
           Output      Output       Output
```

The source is shared where appropriate.

The compilation is target-specific.

The output is platform-specific.

That is the core idea.

---

# Chapter Takeaways

> [!TIP]
> **A target defines where a KMP module is compiled. Source sets describe which code participates in those compilations. Understanding the relationship between the two is one of the foundations of Kotlin Multiplatform.**

Remember:

1. A KMP target represents a compilation destination.
2. A target is more precise than simply naming a platform.
3. Android and iOS can each involve specific target environments.
4. `iosArm64` represents an iOS ARM64 device target.
5. `iosSimulatorArm64` represents an ARM64 iOS simulator target.
6. Android can be configured as a KMP target using current Kotlin Multiplatform tooling.
7. Source sets and targets work together to form compilations.
8. `commonMain` describes shared source; it is not itself a target.
9. A target describes where that source can be compiled.
10. Shared source does not mean one universal binary is produced.
11. Different targets can require different toolchains and native integrations.
12. Hierarchical source sets can allow sharing across related platforms without forcing everything into `commonMain`.
13. Target selection affects dependencies, compilation, testing, CI, and maintenance.
14. Adding every possible target is usually unnecessary.
15. A new target should be evaluated together with dependency compatibility.
16. `expect` / `actual` implementations are connected to target-specific compilations.
17. Platform APIs belong at appropriate target-specific boundaries.
18. Target-specific build failures can reveal incorrect architecture or dependency placement.
19. Targets add branches to the Gradle build graph.
20. More targets generally mean more build and maintenance work.
21. Target declarations can serve as documentation of a module's platform scope.
22. A target should be compiled and tested according to the project's release requirements.
23. Target and variant are different concepts.
24. Target and CPU architecture are related but should not be treated as universally interchangeable concepts.
25. KMP target APIs can evolve with Kotlin tooling, so current project documentation should be followed.
26. Good target design follows the principle: **share where behavior is genuinely common and specialize where platforms meaningfully differ.**

---

# Final Mental Model

When you see:

```kotlin
kotlin {
    androidTarget()

    iosArm64()
    iosSimulatorArm64()
}
```

don't read it as:

```text
Three random Gradle declarations.
```

Read it as:

```text
                         KMP Module
                             │
                    Target Configuration
                             │
             ┌───────────────┼───────────────┐
             ▼               ▼               ▼
          Android         iOS Device      iOS Simulator
             │               │               │
             ▼               ▼               ▼
        Compilation      Compilation      Compilation
             │               │               │
             ▼               ▼               ▼
       Android Output    Native Output    Native Output
```

Then connect it to the source-set model:

```text
                         commonMain
                             │
             ┌───────────────┼───────────────┐
             ▼               ▼               ▼
          Android         iOS Device      iOS Simulator
             │               │               │
       androidMain        iosMain          iosMain
             │               │               │
             └───────────────┼───────────────┘
                             ▼
                       Platform Outputs
```

And remember:

> **A target is the build system's answer to "where does this code need to compile?" Once you understand targets, source sets, and their relationship, KMP stops looking like one large shared codebase and starts looking like what it really is: one architectural model with multiple deliberate compilation paths.**
