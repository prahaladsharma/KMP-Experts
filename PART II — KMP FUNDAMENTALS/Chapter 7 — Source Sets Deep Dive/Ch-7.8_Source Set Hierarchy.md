# Chapter 7 — Source Sets Deep Dive

## Part 8 — Source Set Hierarchy

> **A source set hierarchy is the structure that determines how Kotlin Multiplatform shares code and dependencies across targets. The better the hierarchy reflects your real platform relationships, the less duplication your project needs.**

A KMP project rarely stops at:

```text
commonMain
├── androidMain
└── iosMain
```

As the number of targets grows, a flat structure becomes difficult to maintain.

A more realistic project might contain:

```text
                         commonMain
                             │
            ┌────────────────┼────────────────┐
            │                │                │
         jvmMain          nativeMain       wasmJsMain
            │                │
       ┌────┴────┐      ┌────┼─────┐
       │         │      │    │     │
   androidMain desktopMain iosMain macosMain
```

The hierarchy allows code to be shared at the **right level**, instead of forcing everything into `commonMain` or duplicating code in every platform source set.

---

## 1. What Is a Source Set Hierarchy?

A source set hierarchy defines relationships between source sets.

For example:

```text
commonMain
     │
     └── appleMain
             │
             ├── iosMain
             └── macosMain
```

In this model:

```text
iosMain
```

can use code from:

```text
appleMain
commonMain
```

while:

```text
macosMain
```

can also use:

```text
appleMain
commonMain
```

This creates a natural inheritance-like structure for shared source code.

---

## 2. Why Hierarchy Matters

Without a useful hierarchy, developers often end up with duplication:

```text
iosMain
   └── NetworkHelper.kt

macosMain
   └── NetworkHelper.kt
```

If both implementations are identical, the architecture is telling us something:

```text
These targets probably share a common abstraction or implementation.
```

A hierarchy can introduce:

```text
appleMain
   └── NetworkHelper.kt
```

and both targets can consume it.

---

## 3. The Core Mental Model

Think of the source-set hierarchy as a tree:

```text
                         commonMain
                             │
             ┌───────────────┼───────────────┐
             │               │               │
          appleMain        jvmMain       wasmJsMain
             │               │
         ┌───┴───┐       ┌───┴────┐
         │       │       │        │
      iosMain macosMain androidMain desktopMain
```

The important rule is:

> **A source set can use code and dependencies from the source sets above it in its hierarchy.**

This gives every source set access to the appropriate shared layers.

---

## 4. `commonMain` Is the Root

At the top of most KMP hierarchies is:

```text
commonMain
```

It contains code that is genuinely common to the targets that consume it.

Typical examples:

```text
Business rules
Domain models
Use cases
Repository interfaces
Shared networking abstractions
Serialization models
Validation
Shared utilities
```

A simplified hierarchy is:

```text
commonMain
   │
   ├── Android
   ├── iOS
   ├── Desktop
   └── Web
```

Everything below `commonMain` can potentially reuse its code.

---

## 5. Platform Source Sets

Platform source sets sit below common or intermediate source sets.

Examples include:

```text
androidMain
iosMain
macosMain
desktopMain
wasmJsMain
```

A platform source set is the place for code that is specific to that target or platform family.

For example:

```text
androidMain
    └── Android Context usage

iosMain
    └── UIKit integration
```

---

## 6. Intermediate Source Sets

The real power of the hierarchy appears when multiple targets share something that is not universal.

For example:

```text
commonMain
     │
     └── appleMain
           ├── iosMain
           └── macosMain
```

`appleMain` is an intermediate source set.

It represents:

```text
Apple-family shared code
```

rather than:

```text
All-platform shared code
```

---

## 7. Why Intermediate Source Sets Are Important

Consider:

```text
iOS
macOS
```

Both may share:

```text
Foundation-based functionality
Apple-specific APIs
Common Apple utilities
```

But Android cannot use those APIs.

Putting that code in:

```text
commonMain
```

would be incorrect.

Duplicating it in:

```text
iosMain
macosMain
```

would be unnecessary.

The intermediate source set solves the problem:

```text
commonMain
    │
    └── appleMain
          ├── iosMain
          └── macosMain
```

---

## 8. Source Set Hierarchy vs Platform

An important distinction is:

```text
Source set
```

is not necessarily the same thing as:

```text
Platform
```

A source set is a logical grouping of code.

For example:

```text
appleMain
```

is not an executable target by itself.

It is a place to share code between relevant Apple targets.

---

## 9. Hierarchy Controls Visibility

Suppose:

```text
commonMain
     │
     └── appleMain
             │
             ├── iosMain
             └── macosMain
```

A file in:

```text
appleMain
```

can be used by:

```text
iosMain
macosMain
```

But code in:

```text
iosMain
```

cannot automatically be used by:

```text
macosMain
```

because the dependency direction is downward.

Conceptually:

```text
commonMain
     ↓
appleMain
     ↓
iosMain
```

Visibility flows from parent to child.

---

## 10. The Parent-Child Rule

A useful mental model is:

```text
Parent
  ↓
Child
```

The child can use parent code.

The parent cannot use child-specific code.

For example:

```text
appleMain
   ↓
iosMain
```

`iosMain` can use:

```text
appleMain
```

But:

```text
appleMain
```

cannot import:

```text
iosMain
```

This prevents platform-specific implementation details from leaking upward.

---

## 11. Dependency Inheritance

The same hierarchy concept applies to dependencies.

For example:

```text
appleMain
   └── Apple-compatible dependency
```

Then:

```text
iosMain
macosMain
```

can consume that dependency through the hierarchy, provided the dependency is compatible with those targets.

This makes intermediate source sets useful for both:

```text
Code sharing
Dependency sharing
```

---

## 12. Hierarchy and `commonTest`

Testing follows the source-set model too.

A simplified structure can look like:

```text
commonTest
     │
     ├── Android tests
     ├── iOS tests
     └── Desktop tests
```

Shared tests can live in:

```text
commonTest
```

while platform-specific tests remain in the relevant platform test source sets.

---

## 13. A More Realistic Hierarchy

A larger KMP project might conceptually look like:

```text
commonMain
│
├── jvmMain
│   ├── androidMain
│   └── desktopMain
│
├── appleMain
│   ├── iosMain
│   ├── macosMain
│   └── tvosMain
│
└── wasmJsMain
```

The exact hierarchy depends on the targets configured in the project.

The important idea is not the names.

The important idea is:

```text
Group targets according to what they can genuinely share.
```

---

## 14. Hierarchy Is Determined by Compatibility

You should not create an intermediate source set simply because two targets have similar names.

The real question is:

```text
Can these targets safely share this code?
```

For example:

```text
iOS + macOS
```

may share Apple APIs.

But:

```text
Android + iOS
```

cannot directly share platform APIs merely because both are mobile.

---

## 15. Hierarchy and API Availability

Platform API availability is a major consideration.

For example:

```text
Apple Foundation
```

may be available to multiple Apple targets.

But:

```text
UIKit
```

is not universally available across every Apple target.

Therefore, even inside:

```text
appleMain
```

you must still understand the APIs supported by every target consuming that source set.

---

## 16. Do Not Overload Intermediate Source Sets

An intermediate source set should contain only code that is genuinely shared by its children.

Bad design:

```text
appleMain
    ├── Foundation code
    ├── iOS-only UIKit code
    └── macOS-only AppKit code
```

Better:

```text
appleMain
    └── Shared Apple-compatible code

iosMain
    └── UIKit-specific code

macosMain
    └── AppKit-specific code
```

The hierarchy should reflect capability boundaries.

---

## 17. Source Set Hierarchy and `expect/actual`

The hierarchy works particularly well with `expect/actual`.

For example:

```kotlin
// commonMain
expect class PlatformInfo
```

An intermediate source set can provide a shared `actual` when the same implementation is valid for multiple targets.

Conceptually:

```text
commonMain
    │
    └── appleMain
            └── actual PlatformInfo
                    │
             ┌──────┴──────┐
             ▼             ▼
          iosMain       macosMain
```

This avoids duplicating the same implementation.

---

## 18. `expect/actual` at Different Levels

The implementation does not always have to be directly inside the final platform source set.

For example:

```text
commonMain
    expect Platform
          │
          ▼
      appleMain
    actual Platform
          │
     ┌────┴────┐
     ▼         ▼
   iOS       macOS
```

This is useful when Apple targets can use the same implementation.

---

## 19. When the Actual Implementation Must Be Platform-Specific

If the implementations differ:

```text
commonMain
    expect Platform
          │
     ┌────┴────┐
     ▼         ▼
   iOS       macOS
  actual     actual
```

Each final source set can provide its own implementation.

The hierarchy does not force sharing where sharing is inappropriate.

---

## 20. Hierarchy and Interfaces

Another useful pattern is:

```text
commonMain
    interface Storage
          ▲
          │
    appleMain
    shared Apple implementation
          ▲
       ┌──┴──┐
       │     │
      iOS   macOS
```

The common layer defines the contract.

The intermediate layer can provide a shared implementation.

The platform layer can override it when required.

---

## 21. Hierarchy and Dependency Injection

Dependency injection can follow the same structure:

```text
commonMain
    │
    └── Service interface
             ▲
             │
        appleMain
             │
       Shared provider
          ▲       ▲
          │       │
        iOS     macOS
```

This keeps platform-specific dependencies at the appropriate level.

---

## 22. Hierarchy and Networking

Suppose all targets use the same networking abstraction:

```text
commonMain
    └── API client
```

If only Apple targets require a shared networking implementation:

```text
commonMain
    │
    └── appleMain
            └── Apple networking implementation
```

Then:

```text
iosMain
macosMain
```

can reuse it.

---

## 23. Hierarchy and Storage

Storage often requires more careful separation.

For example:

```text
commonMain
    └── Storage interface
```

Then:

```text
androidMain
    └── Android storage

appleMain
    └── Apple storage
```

and:

```text
iosMain
macosMain
```

can share the Apple implementation if appropriate.

---

## 24. Hierarchy and Logging

Logging is another good example.

If a logging implementation works for all targets:

```text
commonMain
    └── Logging
```

If it works only across JVM targets:

```text
jvmMain
    └── Logging
```

If it works only across Apple targets:

```text
appleMain
    └── Logging
```

The hierarchy lets you express this accurately.

---

## 25. Hierarchy and UI

UI code often has a different sharing structure from business logic.

For example:

```text
commonMain
    └── Business logic

common UI
    └── Compose Multiplatform
```

while platform-specific UI integration may remain in:

```text
androidMain
iosMain
desktopMain
```

The correct hierarchy depends on the UI technology and supported targets.

---

## 26. Source Set Hierarchy Is Not the Same as UI Architecture

Do not confuse:

```text
Source-set hierarchy
```

with:

```text
MVVM
MVI
Clean Architecture
Layered Architecture
```

The source-set hierarchy answers:

> **Where can this code be shared?**

Application architecture answers:

> **How should responsibilities be organized?**

They complement each other.

---

## 27. Automatic Source Set Hierarchy

Modern Kotlin Multiplatform projects can use Kotlin's default source-set hierarchy.

When targets are configured, Kotlin can establish common source sets for compatible target groups.

Conceptually:

```text
commonMain
    │
    ├── nativeMain
    │
    └── jvmMain
```

The exact hierarchy depends on the configured targets and Kotlin tooling.

This reduces the amount of manual source-set wiring required.

---

## 28. Why the Default Hierarchy Is Useful

The default hierarchy provides:

```text
Target grouping
Code sharing
Dependency sharing
Less configuration
Better conventions
```

It is usually preferable to start with the default hierarchy and customize only when the project's requirements justify it.

---

## 29. Explicit Hierarchy Configuration

Sometimes a project needs a custom intermediate source set.

For example:

```text
commonMain
    │
    └── customSharedMain
            ├── targetA
            └── targetB
```

Kotlin Multiplatform provides source-set configuration mechanisms for expressing these relationships.

The exact syntax can evolve with Kotlin and the KMP Gradle plugin, so project configuration should follow the version-specific Kotlin documentation.

---

## 30. Avoid Manual `dependsOn` Everywhere

Older KMP projects often contained explicit relationships such as:

```kotlin
iosMain.dependsOn(commonMain)
```

or multiple manual relationships.

Modern KMP projects generally benefit from the default hierarchy and typed source-set configuration.

Manual relationships should be introduced only when they represent a real architectural need.

---

## 31. Why Manual Hierarchies Can Become Fragile

A manually maintained hierarchy can become difficult when targets change.

For example:

```text
Add tvOS
Add watchOS
Add macOS
Add Wasm
```

Every new target may require additional configuration.

A default hierarchy can reduce this maintenance burden.

---

## 32. Hierarchy and Target Growth

Imagine a project starts with:

```text
Android
iOS
```

Then expands to:

```text
Android
iOS
macOS
Desktop
WebAssembly
```

A flat architecture might become:

```text
commonMain
androidMain
iosMain
macosMain
desktopMain
wasmJsMain
```

A better logical structure may be:

```text
commonMain
├── jvmMain
│   ├── androidMain
│   └── desktopMain
├── appleMain
│   ├── iosMain
│   └── macosMain
└── wasmJsMain
```

The hierarchy grows with the platform model.

---

## 33. Hierarchy as a Platform Family Model

A useful way to think about intermediate source sets is:

```text
Common capability
        ↓
Platform family
        ↓
Specific target
```

For example:

```text
commonMain
     ↓
appleMain
     ↓
iosMain
```

or:

```text
commonMain
     ↓
jvmMain
     ↓
desktopMain
```

This is more expressive than a simple list of platforms.

---

## 34. Platform Families

A platform family represents targets with meaningful shared capabilities.

Examples can include:

```text
Apple targets
JVM targets
Native targets
```

The exact source-set structure depends on the Kotlin Multiplatform targets configured in the project.

Do not create a family merely for organizational appearance.

---

## 35. Capability-Based Hierarchy

A stronger approach is to ask:

```text
What capability do these targets share?
```

rather than:

```text
What name do these platforms have?
```

For example:

```text
Targets:
iOS
macOS

Shared capability:
Apple Foundation APIs
```

This justifies:

```text
appleMain
```

---

## 36. Avoid False Sharing

Suppose two platforms have similar APIs but different behavior.

Forcing them into one intermediate source set can produce:

```text
if (platform == X)
```

everywhere.

That is a warning sign.

A hierarchy should reduce platform conditionals, not create more of them.

---

## 37. The Conditional Code Smell

If an intermediate source set contains code like:

```kotlin
if (isIOS) {
    ...
} else {
    ...
}
```

repeatedly, ask:

```text
Should this code really be shared?
```

Sometimes the correct design is:

```text
shared abstraction
+
platform implementations
```

instead of a giant intermediate source set.

---

## 38. Hierarchy and Abstraction Boundaries

A good hierarchy creates natural boundaries:

```text
commonMain
    ↓
Shared contract
    ↓
Intermediate shared implementation
    ↓
Platform-specific implementation
```

This makes the architecture easier to reason about.

---

## 39. Dependency Placement with Hierarchy

Consider:

```text
commonMain
    │
    └── appleMain
            │
            ├── iosMain
            └── macosMain
```

If a dependency supports Apple targets but not Android:

```text
appleMain.dependencies {
    implementation(...)
}
```

may be the appropriate location.

This avoids:

```text
commonMain
```

being coupled to an Apple-only library.

---

## 40. Hierarchy and Dependency Resolution

The previous part discussed dependency resolution.

The hierarchy is the structure that helps determine where a dependency belongs.

Think:

```text
Dependency supports all targets
        ↓
commonMain

Dependency supports Apple targets
        ↓
appleMain

Dependency supports iOS only
        ↓
iosMain
```

This creates a direct relationship between:

```text
Target compatibility
```

and:

```text
Source-set placement
```

---

## 41. Hierarchy and Testing

A similar approach can be used for tests.

For example:

```text
commonTest
    │
    └── appleTest
            ├── iosTest
            └── macosTest
```

The exact source-set structure depends on the targets and testing setup.

The principle remains:

```text
Share tests where behavior is shared.
Keep platform-specific tests where behavior differs.
```

---

## 42. Shared Tests Are Valuable

A well-designed hierarchy lets you write a test once:

```text
commonTest
    ↓
RepositoryTest
```

and run it across multiple targets.

This provides stronger confidence that shared business logic behaves consistently.

---

## 43. Platform Tests Still Matter

Shared tests do not replace platform tests.

For example:

```text
commonTest
    └── Repository behavior

androidTest
    └── Android integration

iosTest
    └── iOS integration
```

Both levels are valuable.

---

## 44. Hierarchy and Build Performance

A clean source-set hierarchy can also improve build organization.

It helps Gradle and Kotlin understand:

```text
Which code belongs to which target
Which dependencies are relevant
Which compilations need which sources
```

However, hierarchy design should primarily be driven by correctness and maintainability, not premature build optimization.

---

## 45. Hierarchy and Compilation

Each target ultimately compiles the source it can see through its source-set hierarchy.

Conceptually:

```text
iosMain
   │
   ├── ios-specific code
   ├── appleMain code
   └── commonMain code
```

An iOS compilation therefore sees the relevant shared layers.

---

## 46. Source Set Hierarchy and IDE Navigation

A well-designed hierarchy also improves developer experience.

When opening a shared class, the developer can quickly understand:

```text
common
Apple shared
iOS-specific
```

The source structure becomes documentation.

---

## 47. Naming Intermediate Source Sets

Use names that communicate intent.

Good examples:

```text
appleMain
jvmMain
nativeMain
```

when they accurately represent the shared capability.

Avoid vague names such as:

```text
shared2
platformCommon
miscMain
tempMain
```

The source tree should explain itself.

---

## 48. Naming Based on Capability

If a custom source set exists because two targets share a particular capability, the name should communicate that relationship.

For example:

```text
appleMain
```

is more meaningful than:

```text
mobileSharedMain
```

if the code is specifically Apple-platform code.

---

## 49. A Practical Decision Tree

When deciding where code belongs, ask:

```text
Is it shared by every supported target?
        │
       Yes
        ↓
   commonMain
        │
       No
        ↓
Is it shared by a platform family?
        │
       Yes
        ↓
Intermediate source set
        │
       No
        ↓
Platform-specific source set
```

This simple decision tree solves many hierarchy questions.

---

## 50. Dependency Decision Tree

For dependencies:

```text
Does the dependency support all consuming targets?
        │
       Yes
        ↓
     commonMain

       No
        ↓

Does it support a subset with a shared source set?
        │
       Yes
        ↓
Intermediate source set

       No
        ↓
Platform source set
```

This connects dependency management directly to source-set architecture.

---

## 51. Architecture Example

A scalable KMP project could look like:

```text
shared/
│
├── commonMain/
│   ├── domain/
│   ├── data/
│   └── common/
│
├── commonTest/
│
├── jvmMain/
│
├── androidMain/
│   └── platform/
│
├── desktopMain/
│   └── platform/
│
├── appleMain/
│   └── platform/
│
├── iosMain/
│   └── platform/
│
├── macosMain/
│   └── platform/
│
└── wasmJsMain/
    └── platform/
```

The exact folders are a project choice; the important part is the source-set relationship.

---

## 52. Architecture Example: Shared Business Logic

A typical business layer may remain entirely common:

```text
commonMain
    ├── models
    ├── repositories
    ├── usecases
    ├── validation
    └── state
```

Platform source sets then provide:

```text
Storage
Networking engines
Platform services
Permissions
Lifecycle integration
```

This keeps business logic portable.

---

## 53. Architecture Example: Platform Services

```text
commonMain
    │
    └── PlatformService
             ▲
             │
        ┌────┴─────┐
        │          │
   appleMain    androidMain
        │
   ┌────┴────┐
   ▼         ▼
  iOS      macOS
```

This structure avoids forcing platform details into shared code.

---

## 54. Hierarchy and Clean Architecture

KMP source sets can map naturally onto Clean Architecture boundaries.

For example:

```text
commonMain
    ├── Domain
    ├── Use Cases
    └── Repository contracts

Platform source sets
    ├── Platform data sources
    └── Platform integrations
```

The source-set hierarchy reinforces the architectural dependency direction.

---

## 55. Hierarchy and MVI

The same principle can work with MVI:

```text
commonMain
    ├── State
    ├── Intent
    ├── Reducer
    └── Business logic
```

Platform layers can provide:

```text
Platform event sources
Platform lifecycle
Platform services
```

The state machine remains shared where possible.

---

## 56. Hierarchy Is an Architectural Contract

Once established, the hierarchy communicates:

```text
What is common?
What belongs to a platform family?
What is target-specific?
```

Breaking the hierarchy casually can create architectural coupling.

Treat source-set boundaries as meaningful contracts.

---

## 57. Reviewing a Source Set Hierarchy

During code review, ask:

```text
Does this code belong at this level?
Could it move upward?
Should it move downward?
Does it depend on a more specific target?
Is an intermediate source set justified?
Is code being duplicated?
```

These questions keep the hierarchy healthy.

---

## 58. Smell: Too Much Code in `commonMain`

Warning signs include:

```text
Platform checks
expect/actual everywhere
Platform-specific imports
Large dependency lists
Target-specific conditionals
```

This may indicate that `commonMain` is doing too much.

---

## 59. Smell: Too Much Duplication

Another warning sign:

```text
Same class
Same algorithm
Same test
Same dependency
```

appears in:

```text
iosMain
macosMain
```

or other sibling source sets.

Consider whether an intermediate source set can provide the shared implementation.

---

## 60. Smell: Giant Intermediate Source Set

The opposite problem is:

```text
appleMain
```

becoming a large dumping ground.

If it contains many target checks:

```text
if iOS
if macOS
if tvOS
```

it may need to be split.

The goal is not maximum sharing.

The goal is **meaningful sharing**.

---

## 61. Source Set Hierarchy and Maintainability

A good hierarchy reduces:

```text
Duplication
Conditional logic
Platform leakage
Dependency confusion
```

and improves:

```text
Discoverability
Testing
Reuse
Architecture
```

---

## 62. Source Set Hierarchy and Scalability

A two-target project may survive with a simple structure:

```text
commonMain
androidMain
iosMain
```

A multi-platform product benefits much more from a deliberate hierarchy.

For example:

```text
commonMain
├── jvmMain
│   ├── androidMain
│   └── desktopMain
├── appleMain
│   ├── iosMain
│   └── macosMain
└── wasmJsMain
```

The structure scales with the product.

---

## 63. A Useful Rule of Thumb

> **Move code upward only when its assumptions remain valid for every child source set.**

This is one of the safest rules for hierarchy design.

For example:

```text
Apple Foundation code
```

can move from:

```text
iosMain
```

to:

```text
appleMain
```

only if it is valid for every target consuming `appleMain`.

---

## 64. Another Useful Rule

> **Move code downward when a parent source set would otherwise need to know about a specific platform.**

For example:

```text
UIKit-specific code
```

should stay in:

```text
iosMain
```

rather than:

```text
appleMain
```

if other Apple targets cannot use it.

---

## 65. The Three-Level Model

A practical KMP hierarchy often follows:

```text
Level 1
commonMain
    ↓
Level 2
Platform family / intermediate source set
    ↓
Level 3
Platform-specific source set
```

Example:

```text
commonMain
    ↓
appleMain
    ↓
iosMain
```

Not every project needs all three levels.

Use only the levels that represent real sharing.

---

## 66. Do Not Build the Hierarchy for the Diagram

A beautiful hierarchy is not automatically a good hierarchy.

The hierarchy exists to support:

```text
Code reuse
Correct dependencies
Clear boundaries
Platform compatibility
```

Do not create intermediate source sets simply because the project diagram looks more sophisticated.

---

## 67. Keep the Hierarchy Boring

The best source-set hierarchy is often predictable:

```text
commonMain
    ↓
platform family
    ↓
platform target
```

Developers should understand it without needing a separate document.

Good architecture is often boring in the best possible way.

---

## 68. Source Set Hierarchy Checklist

Before creating an intermediate source set, ask:

```text
Do at least two targets genuinely share code?
Do they share the same required APIs?
Can they share the same dependencies?
Would duplication otherwise occur?
Does the source set have a clear name?
Will future targets benefit from it?
Does it reduce platform conditionals?
```

If the answer is mostly yes, an intermediate source set may be justified.

---

## 69. Hierarchy Checklist for Code Placement

For every new file:

```text
Can all supported targets use it?
        ↓
commonMain

Only a platform family?
        ↓
Intermediate source set

Only one target?
        ↓
Platform source set
```

This simple process can prevent many architecture problems.

---

## 70. Hierarchy Checklist for Dependencies

For every new dependency:

```text
Target support verified?
        ↓
Source set selected
        ↓
Transitive dependencies checked
        ↓
Version compatibility checked
        ↓
Build validated
        ↓
Runtime validated
```

This combines the lessons from dependency resolution with source-set design.

---

# Common Mistakes

### ❌ Putting every dependency in `commonMain`

Not every dependency is multiplatform.

### ❌ Duplicating shared code

If multiple targets can safely use the same implementation, consider an intermediate source set.

### ❌ Sharing code too aggressively

If the code depends on a platform-specific API, keep it lower in the hierarchy.

### ❌ Treating `iosMain` and `macosMain` as identical

They may share some APIs but still have different platform capabilities.

### ❌ Creating unnecessary custom source sets

Start with the default hierarchy and introduce custom intermediate sets only when they solve a real problem.

### ❌ Using platform checks everywhere

Repeated checks often indicate that code belongs in platform-specific implementations.

### ❌ Making parent source sets depend on child source sets

The dependency direction should remain from parent to child.

### ❌ Assuming target names determine compatibility

Compatibility comes from APIs, dependencies, compiler targets, and platform capabilities.

### ❌ Testing only one target

A hierarchy is only useful if every relevant target can compile and run its part of the graph.

---

# A Complete Mental Model

The complete source-set model can be visualized as:

```text
                         ┌──────────────────┐
                         │    commonMain    │
                         │  Shared logic    │
                         └────────┬─────────┘
                                  │
               ┌──────────────────┼──────────────────┐
               │                  │                  │
               ▼                  ▼                  ▼
          ┌─────────┐        ┌─────────┐        ┌───────────┐
          │ jvmMain │        │appleMain│        │ wasmJsMain│
          └────┬────┘        └────┬────┘        └───────────┘
               │                  │
          ┌────┴────┐        ┌────┼────┐
          ▼         ▼        ▼         ▼
     androidMain desktopMain iosMain macosMain
```

Each layer answers a different question:

```text
commonMain
"What can everyone share?"

Intermediate source set
"What can this family of targets share?"

Platform source set
"What is unique to this target?"
```

That is the essence of source-set hierarchy design.

---

# Chapter Takeaways

> [!IMPORTANT]
> **A source-set hierarchy is not merely a Gradle structure. It is a map of your application's platform boundaries.**

Remember:

1. A source-set hierarchy defines relationships between KMP source sets.
2. `commonMain` is usually the root of shared production code.
3. Child source sets can use code from their parent source sets.
4. Parent source sets should not depend on child-specific code.
5. Dependencies can follow the same hierarchy.
6. Intermediate source sets allow selective sharing.
7. Intermediate source sets are useful when multiple targets share capabilities but not all targets do.
8. `appleMain` can represent Apple-family shared code when appropriate.
9. `jvmMain` can represent JVM-family shared code when appropriate.
10. The exact hierarchy depends on the configured KMP targets.
11. Modern KMP tooling can establish a default source-set hierarchy.
12. Start with the default hierarchy when it fits the project.
13. Add custom source sets only when they represent real shared behavior.
14. Source-set hierarchy is different from application architecture.
15. Source sets answer where code can be shared.
16. Architecture answers how responsibilities are organized.
17. `commonMain` should contain genuinely multiplatform code.
18. Platform-specific APIs should remain at an appropriate lower level.
19. A platform family source set should contain only code valid for every child target.
20. Do not put iOS-only APIs into an Apple-wide source set.
21. Do not put Android-only APIs into common code.
22. Avoid unnecessary duplication between sibling source sets.
23. Avoid creating huge intermediate source sets.
24. Repeated platform checks can indicate an incorrect hierarchy.
25. The hierarchy should reduce conditional platform logic.
26. The hierarchy should reflect real capability boundaries.
27. `expect/actual` can work at different hierarchy levels.
28. A shared `actual` can sometimes be implemented in an intermediate source set.
29. Platform-specific `actual` implementations remain possible at leaf source sets.
30. Interfaces can be shared in `commonMain`.
31. Platform implementations can live in intermediate or platform source sets.
32. Dependency injection can follow source-set boundaries.
33. Dependency placement should follow target compatibility.
34. A dependency supporting all relevant targets can belong in `commonMain`.
35. A dependency supporting only a platform family can belong in an intermediate source set.
36. A dependency supporting one target should remain in that target's source set.
37. Dependency resolution and source-set hierarchy are closely related.
38. Shared tests can live in `commonTest`.
39. Platform-specific tests should remain platform-specific.
40. Shared tests do not replace platform integration tests.
41. A good hierarchy improves code discoverability.
42. A good hierarchy makes platform boundaries visible.
43. A good hierarchy can reduce maintenance.
44. A good hierarchy can reduce dependency duplication.
45. A good hierarchy can reduce platform leakage.
46. A good hierarchy can make target expansion easier.
47. Do not design the hierarchy merely for visual complexity.
48. Use meaningful source-set names.
49. Name intermediate source sets according to the capability or platform family they represent.
50. Avoid vague source-set names.
51. Move code upward only when its assumptions remain valid for every child.
52. Move code downward when parent-level code would otherwise depend on a specific platform.
53. Two similar platforms do not automatically justify a shared source set.
54. Shared APIs and dependencies are stronger reasons for creating an intermediate source set.
55. A source-set hierarchy should remain understandable to new contributors.
56. The default hierarchy is often sufficient for many projects.
57. Custom hierarchy configuration should solve a concrete problem.
58. Manual source-set relationships can become fragile as targets grow.
59. A growing project benefits from explicit platform-family thinking.
60. A source-set hierarchy should scale with the number of supported targets.
61. Business logic should generally remain as high in the hierarchy as its portability allows.
62. Platform integrations should generally remain as low as their platform requirements demand.
63. Avoid putting platform checks into shared implementations when separate platform implementations are clearer.
64. Use intermediate source sets to express meaningful reuse.
65. Do not force sharing when platform behavior is genuinely different.
66. Test hierarchy assumptions across every supported target.
67. Validate dependencies at the level where they are declared.
68. Review source-set placement during code review.
69. Treat source-set boundaries as architectural boundaries.
70. The best hierarchy balances reuse and isolation.
71. **The goal is not maximum sharing; the goal is correct sharing.**

---

## Final Thought

A mature KMP project does not try to make every line of code common.

It tries to make **the right lines of code common**.

The source-set hierarchy makes that distinction visible:

```text
                         commonMain
                             │
                    Shared capabilities
                             │
                ┌────────────┼────────────┐
                ▼            ▼            ▼
            jvmMain       appleMain    wasmJsMain
                │            │
          ┌─────┴─────┐   ┌──┴─────┐
          ▼           ▼   ▼        ▼
      Android      Desktop iOS    macOS
```

The strongest KMP architecture usually follows a simple principle:

> **Share code at the highest level where its assumptions remain valid, and move platform-specific behavior downward when those assumptions stop being universal.**

That is what turns a collection of targets into a maintainable multiplatform architecture.
