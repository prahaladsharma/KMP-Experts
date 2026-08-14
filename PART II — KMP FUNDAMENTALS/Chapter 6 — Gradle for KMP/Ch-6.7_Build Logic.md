# Chapter 6 — Gradle for KMP

## Part 7 — Build Logic

> **When a KMP repository grows, Gradle configuration becomes a system of its own. Build Logic is the layer that turns repeated build decisions into reusable, maintainable, and testable engineering infrastructure.**

By this point, we have looked at:

- Why Gradle matters in KMP
- Plugins
- Targets
- Dependencies
- Version Catalogs
- Convention Plugins

There is one larger concept that connects them:

```text
Build Logic
```

Build Logic is the code that defines **how the repository itself is built**.

It can contain:

```text
Convention Plugins
Build Utilities
Shared Configuration
Dependency Rules
Task Configuration
Publishing Rules
Quality Rules
Build Validation
```

The goal is not to make Gradle more complicated.

The goal is to prevent a growing repository from becoming a collection of unrelated build scripts.

---

# 1. What Is Build Logic?

Build Logic is reusable code that configures and controls the build.

Think of it as:

```text
Application Code
       │
       └── Builds the product

Build Logic
       │
       └── Builds the product's build
```

That second statement may sound strange, but it is an important mental model.

Your application needs:

```text
Architecture
Features
Libraries
Tests
```

Your build needs:

```text
Plugins
Targets
Dependencies
Compiler Rules
Tasks
Publishing
Validation
```

Build Logic manages the second group.

---

# 2. Why Build Logic Matters in KMP

A small project can survive with:

```text
settings.gradle.kts
build.gradle.kts
module/build.gradle.kts
```

As the project grows:

```text
shared
core
data
domain
feature-auth
feature-orders
feature-profile
androidApp
```

each module starts needing build configuration.

Without centralized build logic:

```text
Module A
   └── Build Configuration

Module B
   └── Build Configuration

Module C
   └── Build Configuration

Module D
   └── Build Configuration
```

The same rules slowly get copied.

Build Logic changes that relationship:

```text
                    Build Logic
                         │
          ┌──────────────┼──────────────┐
          ▼              ▼              ▼
       Module A       Module B       Module C
```

---

# 3. Build Logic Is More Than Convention Plugins

Convention Plugins are an important part of Build Logic.

But:

```text
Build Logic
    │
    ├── Convention Plugins
    ├── Build Utilities
    ├── Shared Configuration
    ├── Task Configuration
    ├── Validation
    └── Publishing Logic
```

So:

> **Convention Plugins are a form of Build Logic, not the entire Build Logic layer.**

This distinction becomes important in larger repositories.

---

# 4. The Three-Layer Gradle Model

A useful model is:

```text
                 Gradle Build
                      │
        ┌─────────────┼─────────────┐
        ▼             ▼             ▼
  Version Catalog  Build Logic    Modules
        │             │             │
        │             │             │
      What?          How?          Product
        │             │             │
        └─────────────┼─────────────┘
                      ▼
                Final Build
```

### Version Catalog

Defines:

```text
Versions
Libraries
Plugins
Aliases
```

### Build Logic

Defines:

```text
Rules
Conventions
Tasks
Configuration
```

### Modules

Contain:

```text
Application and library code
```

This separation is extremely useful when a KMP repository becomes large.

---

# 5. Why Not Put Everything in `build.gradle.kts`?

A module build file should describe the module.

For example:

```kotlin
plugins {
    id("company.kmp.library")
}

kotlin {
    // module-specific configuration
}
```

It should not contain hundreds of lines of repeated project-wide rules.

When the root or module build file becomes responsible for:

```text
Every module
Every platform
Every compiler rule
Every CI task
Every publishing rule
```

the build becomes difficult to understand.

Build Logic gives those responsibilities a home.

---

# 6. A Typical Build Logic Structure

A project might look like:

```text
project/
│
├── gradle/
│   └── libs.versions.toml
│
├── build-logic/
│   └── convention/
│       ├── build.gradle.kts
│       └── src/
│           └── main/
│               └── kotlin/
│                   ├── kmp.library.gradle.kts
│                   ├── android.library.gradle.kts
│                   ├── quality.gradle.kts
│                   └── publishing.gradle.kts
│
├── shared/
├── core/
├── feature-auth/
└── androidApp/
```

The exact structure can vary.

The important concept is:

```text
Build Logic
     │
     └── Reusable build behavior
```

---

# 7. `build-logic` as an Included Build

A common modern Gradle architecture places build logic in an included build such as:

```text
build-logic/
```

and includes it from the root settings.

Conceptually:

```text
Root Build
    │
    ├── Application Modules
    │
    └── Included Build
            │
            └── build-logic
```

This gives the build logic its own project boundary.

That boundary is valuable because Build Logic can have:

```text
Its own source files
Its own dependencies
Its own plugins
Its own tests
```

---

# 8. Why an Included Build?

Build Logic is itself software.

Giving it a dedicated build allows it to be developed more like a normal Gradle project.

Conceptually:

```text
Application Build
        │
        ▼
Uses Build Logic

Build Logic Build
        │
        ▼
Builds reusable Gradle code
```

This separation makes large repositories easier to organize.

---

# 9. `buildSrc` vs Dedicated Build Logic

Historically, Gradle projects often used:

```text
buildSrc/
```

for custom build logic.

That approach can still be encountered in existing projects.

However, for larger modern builds, a dedicated included build such as:

```text
build-logic/
```

often provides a cleaner and more explicit boundary.

The important lesson is not:

```text
"One directory is always correct."
```

The important lesson is:

> **Keep reusable build logic separate from application modules and organize it as maintainable build code.**

---

# 10. Why Large `build.gradle.kts` Files Become a Problem

Imagine:

```text
root/build.gradle.kts
```

contains:

```text
Kotlin configuration
Android configuration
KMP configuration
Compose configuration
Testing
Lint
Publishing
Signing
CI tasks
Dependency rules
```

Then every change requires understanding the entire file.

The build becomes:

```text
One giant configuration surface
```

instead of:

```text
Small modules
+
Focused build logic
```

---

# 11. Build Logic Creates Boundaries

A healthy build can have:

```text
KMP Convention
       │
       ▼
KMP Modules

Android Convention
       │
       ▼
Android Libraries

Quality Convention
       │
       ▼
Applicable Modules

Publishing Convention
       │
       ▼
Published Libraries
```

Each boundary has a clear responsibility.

---

# 12. Build Logic Should Be Intentional

Build Logic should answer:

```text
What rule are we enforcing?
Why is it shared?
Which modules need it?
What happens if it changes?
```

Avoid creating build abstractions simply because:

```text
"Gradle allows it."
```

Build Logic is most valuable when it removes real repetition or enforces meaningful standards.

---

# 13. Build Logic as Infrastructure

Think of your repository as:

```text
Product
   │
   ├── Application Code
   ├── Tests
   └── Build Infrastructure
```

Build Logic belongs to:

```text
Build Infrastructure
```

It supports:

```text
Development
Testing
CI
Release
Publishing
Quality
```

This makes it a first-class part of the engineering system.

---

# 14. Build Logic and KMP Targets

KMP projects are particularly good candidates for shared build logic because target configuration can be repetitive.

For example:

```text
Every KMP library
    │
    ├── Android
    ├── iOS device
    └── iOS simulator
```

Instead of repeating this in every module:

```text
KMP Convention
      │
      ▼
Standard Targets
```

The target policy has one owner.

---

# 15. Build Logic and Source Sets

Build Logic can also standardize:

```text
commonMain
commonTest
androidMain
iosMain
```

where appropriate.

Conceptually:

```text
KMP Convention
      │
      ├── Targets
      ├── Source Sets
      └── Test Configuration
```

The module can then focus on actual implementation.

---

# 16. Build Logic and Dependencies

Build Logic can decide that a certain class of modules receives a standard dependency.

For example:

```text
KMP library modules
       │
       └── Standard test dependency
```

But dependency declarations should remain deliberate.

A convention should not silently inject every possible library into every module.

---

# 17. Build Logic and Version Catalogs

A common architecture is:

```text
gradle/libs.versions.toml
        │
        ▼
Dependency and Plugin Definitions
        │
        ▼
Build Logic
        │
        ▼
Module Configuration
```

For example:

```text
Catalog
→ Kotlin version

Build Logic
→ Apply Kotlin Multiplatform

Module
→ Apply company.kmp.library
```

This separation keeps ownership clear.

---

# 18. Build Logic and Plugin Aliases

A Version Catalog can define plugin aliases.

Build Logic can then apply or configure those plugins.

Conceptually:

```text
Version Catalog
      │
      ▼
Plugin Alias
      │
      ▼
Convention Plugin
      │
      ▼
Module
```

This allows plugin versions to remain centralized while build behavior remains centralized separately.

---

# 19. Build Logic and Gradle APIs

Build Logic uses Gradle APIs to configure projects.

For example:

```kotlin
class ExamplePlugin : Plugin<Project> {

    override fun apply(project: Project) {
        // Configure the project
    }
}
```

Or a precompiled script plugin can express the same kind of behavior through Kotlin DSL.

The exact implementation depends on the Gradle architecture chosen by the project.

---

# 20. Build Logic Is Code

This deserves emphasis.

Build Logic has:

```text
Inputs
Rules
Outputs
Dependencies
Failure Modes
Tests
```

It should therefore be treated like production infrastructure.

Avoid writing:

```text
mysterious Gradle magic
```

that nobody understands.

Prefer:

```text
Small
Explicit
Named
Testable
Documented
```

build components.

---

# 21. Build Logic and Type Safety

Kotlin-based Gradle configuration provides strong tooling support.

Build Logic written in Kotlin can benefit from:

```text
IDE completion
Type checking
Refactoring
Compile-time feedback
```

This is one reason Kotlin DSL is especially comfortable for Android and KMP teams.

But type safety does not automatically make build logic well designed.

Good structure still matters.

---

# 22. Build Logic and Reuse

The most important property of Build Logic is reuse.

Without reuse:

```text
Configuration
Configuration
Configuration
Configuration
```

With reuse:

```text
Build Rule
    │
    ├── Module A
    ├── Module B
    ├── Module C
    └── Module D
```

One rule can control many modules.

---

# 23. Build Logic and Consistency

Suppose every KMP library should use:

```text
Same Kotlin settings
Same target model
Same test conventions
Same quality checks
```

Build Logic can make that consistent.

Without it:

```text
Module A → Standard
Module B → Slightly different
Module C → Old configuration
Module D → Custom configuration
```

Configuration drift is reduced when shared behavior has one owner.

---

# 24. Build Logic and Drift

Build drift happens when modules slowly become different.

For example:

```text
2026:
All modules use the same configuration.

Later:
Module A → updated
Module B → old
Module C → custom
Module D → forgotten
```

A central convention can prevent this.

```text
Central Rule
      │
      ▼
Applicable Modules
```

---

# 25. Build Logic and Migration

Build Logic becomes especially valuable during migrations.

Suppose the project moves to:

```text
New Kotlin version
```

or:

```text
New KMP target configuration
```

Without centralized logic:

```text
Update every module
```

With centralized logic:

```text
Update build convention
       │
       ▼
Validate affected modules
```

This can dramatically reduce migration effort.

---

# 26. Build Logic and Toolchain Upgrades

KMP projects often involve coordinated upgrades:

```text
Gradle
Kotlin
KMP
Android tooling
Libraries
```

Build Logic can isolate many of the changes.

For example:

```text
Old KMP API
      │
      ▼
Convention Plugin
      │
      ▼
New KMP API
```

Modules may remain unchanged.

That is a major architectural advantage.

---

# 27. Build Logic and Compiler Configuration

Compiler configuration is a good candidate for centralization.

For example:

```text
Language level
Opt-ins
Compiler warnings
Common compiler options
```

The convention can provide:

```text
Approved Compiler Policy
```

to all relevant modules.

---

# 28. Build Logic and Testing

Build Logic can standardize:

```text
Test dependencies
Test tasks
Verification
Coverage
Test reporting
```

For example:

```text
quality convention
       │
       ├── Unit Tests
       ├── Static Analysis
       └── Coverage
```

The exact tools depend on the repository.

---

# 29. Build Logic and CI

CI should ideally use the same build rules developers use locally.

Conceptually:

```text
Developer
   │
   ▼
Build Logic
   │
   ▼
Local Build

CI
   │
   ▼
Build Logic
   │
   ▼
CI Build
```

This avoids maintaining one set of rules for local development and another for CI.

---

# 30. Build Logic and Release

Release workflows can also use shared build logic.

For example:

```text
Publishing
Signing
Artifact validation
Version checks
Release metadata
```

can be centralized where the same behavior applies to multiple libraries.

---

# 31. Build Logic and Publishing

Suppose the repository publishes:

```text
core
network
database
shared-ui
```

Each may need:

```text
Maven metadata
Repository configuration
Versioning
Signing
```

A publishing convention can provide common rules:

```text
Publishing Convention
       │
       ├── core
       ├── network
       ├── database
       └── shared-ui
```

Module-specific metadata can remain explicit.

---

# 32. Build Logic and Quality Gates

A project can establish:

```text
No build without tests
No release with failing quality checks
No publication without validation
```

through build logic.

Conceptually:

```text
Build
  │
  ├── Compile
  ├── Test
  ├── Quality
  └── Publish
```

The build becomes an enforcement mechanism.

---

# 33. Build Logic and Repository Governance

For larger teams, Build Logic can encode standards such as:

```text
Supported JDK
Supported Kotlin version
Approved targets
Required tests
Required quality checks
Publishing rules
```

This turns informal engineering standards into executable rules.

---

# 34. Build Logic and Developer Experience

Good Build Logic reduces cognitive load.

A developer opening:

```text
feature-auth/build.gradle.kts
```

might see:

```kotlin
plugins {
    id("company.kmp.library")
    id("company.quality")
}
```

That communicates:

```text
This is a KMP library.
It follows company quality rules.
```

The details live in the build infrastructure.

---

# 35. Build Logic as Executable Documentation

A written document can say:

> "All KMP modules support Android and iOS."

A convention can enforce:

```text
Android
+
iOS
```

The second is executable documentation.

The repository itself demonstrates its build rules.

---

# 36. Avoid Hidden Rules

Build Logic should be discoverable.

If:

```kotlin
id("company.kmp.library")
```

does twenty unrelated things, developers may struggle to understand the module.

Prefer conventions whose names communicate their purpose.

For example:

```text
company.kmp.library
company.quality
company.publishing
```

is clearer than:

```text
company.default
```

---

# 37. Build Logic Naming

Good names communicate intent:

```text
kmp.library
android.library
android.application
compose.multiplatform
quality
publishing
```

Poor names:

```text
common
default
base2
stuff
main
```

Build Logic names are part of the developer experience.

---

# 38. Build Logic Composition

Build rules can be composed.

For example:

```text
Feature Module
      │
      ├── KMP Library
      └── Quality
```

Another module:

```text
Published Library
      │
      ├── KMP Library
      ├── Quality
      └── Publishing
```

This avoids creating a unique mega-plugin for every module combination.

---

# 39. Build Logic and Separation of Concerns

A useful division is:

```text
Version Catalog
→ Versions and dependency aliases

KMP Convention
→ KMP configuration

Android Convention
→ Android configuration

Quality Convention
→ Quality configuration

Publishing Convention
→ Publishing configuration
```

Each piece has one job.

---

# 40. Build Logic and Module-Specific Configuration

Not everything belongs in Build Logic.

Suppose:

```text
feature-payment
```

needs a unique generated source directory.

That may remain in:

```text
feature-payment/build.gradle.kts
```

because it is module-specific.

Use this rule:

> **Shared behavior belongs in Build Logic; unique behavior belongs in the module.**

---

# 41. The Right Abstraction Boundary

Think:

```text
Repeated + Stable + Meaningful
             │
             ▼
         Build Logic

Unique + Local + Temporary
             │
             ▼
         Module Build
```

This prevents both:

```text
Too little abstraction
```

and:

```text
Too much abstraction
```

---

# 42. Build Logic and Gradle Configuration Avoidance

Large repositories can contain:

```text
50+
100+
```

modules.

If Build Logic eagerly configures everything, configuration time can grow significantly.

Prefer Gradle APIs and patterns that avoid unnecessary work.

Conceptually:

```text
Only configure what is needed
        │
        ▼
Better configuration performance
```

This becomes increasingly important at scale.

---

# 43. Don't Resolve Dependencies Eagerly

Avoid unnecessary dependency resolution during configuration.

Instead of:

```text
Resolve everything
      │
      ▼
Configure project
```

prefer:

```text
Declare configuration
      │
      ▼
Resolve when needed
```

This aligns with Gradle's lazy configuration model.

---

# 44. Don't Scan the Repository Unnecessarily

A Build Logic plugin should not repeatedly scan:

```text
Every source file
Every module
Every directory
```

just to determine configuration.

Use Gradle's model and APIs where possible.

Build performance is an architectural concern.

---

# 45. Build Logic and Task Registration

When creating custom tasks, prefer lazy task registration where appropriate.

Conceptually:

```text
Register Task
      │
      ▼
Gradle configures it
      │
      ▼
Task runs only when requested
```

This helps large builds remain efficient.

---

# 46. Build Logic and Custom Tasks

Custom tasks can be useful for:

```text
Validation
Code generation
Release checks
Dependency verification
Documentation
```

But don't create custom tasks for something Gradle already provides.

Before adding a task, ask:

```text
Does Gradle already solve this?
Does an existing plugin solve it?
Is this task reusable?
Does it belong in Build Logic?
```

---

# 47. Build Logic and Custom Gradle Plugins

A custom plugin is useful when:

```text
Several modules need the same configuration
```

or:

```text
A build behavior deserves a named abstraction
```

For example:

```text
company.kmp.library
```

is better than copying a 100-line KMP configuration into every module.

---

# 48. Build Logic and Convention Plugins

The relationship can be summarized as:

```text
Build Logic
    │
    └── Convention Plugins
           │
           ├── KMP
           ├── Android
           ├── Quality
           └── Publishing
```

Convention Plugins are one of the main delivery mechanisms for reusable build logic.

---

# 49. Build Logic and Precompiled Scripts

Precompiled Kotlin DSL plugins can make conventions readable.

Conceptually:

```text
build-logic/
    │
    └── src/main/kotlin/
          │
          ├── kmp.library.gradle.kts
          └── quality.gradle.kts
```

The exact structure depends on the chosen build-logic setup.

The key is that these scripts become reusable plugin implementations.

---

# 50. Build Logic and Plugin Dependencies

If a convention applies:

```text
Kotlin Multiplatform
```

the build-logic project may need the corresponding plugin available to its implementation.

Likewise, an Android convention may need Android Gradle Plugin APIs.

This is why Build Logic has its own dependency graph.

---

# 51. Build Logic Dependency Graph

Think:

```text
Build Logic
     │
     ├── Gradle API
     ├── Kotlin Gradle Plugin
     ├── Android Gradle Plugin
     └── Other Build Plugins
```

This graph is separate from:

```text
Application
     │
     ├── Coroutines
     ├── Networking
     └── Database
```

Keeping those graphs conceptually separate prevents confusion.

---

# 52. Build Logic and Version Catalog Access

A build-logic project may need access to dependency information from the repository's Version Catalog.

The exact mechanism depends on how the included build is structured.

The important design principle is:

```text
Versions have one owner.
```

Avoid maintaining:

```text
Application dependency version
```

and:

```text
Build-logic dependency version
```

independently when they are intended to be the same.

---

# 53. Avoid Duplicate Version Sources

Bad:

```text
libs.versions.toml
    Kotlin = X

build-logic
    Kotlin = Y

root build
    Kotlin = Z
```

Now the repository has multiple sources of truth.

Prefer:

```text
Central Version Definition
          │
          ├── Build Logic
          └── Modules
```

where practical.

---

# 54. Build Logic and Toolchain Ownership

A mature repository should make it clear:

```text
Who owns Kotlin version?
Who owns target configuration?
Who owns compiler options?
Who owns quality rules?
Who owns publishing?
```

For example:

```text
Version Catalog → versions
KMP Convention → targets
Quality Convention → quality
Publishing Convention → publishing
```

Clear ownership makes maintenance easier.

---

# 55. Build Logic and Migration From Copy-Paste

Suppose a project already has:

```text
30 modules
```

with repeated KMP configuration.

Don't rewrite everything blindly.

A practical migration can be:

```text
Find repeated configuration
        │
        ▼
Group similar modules
        │
        ▼
Extract one convention
        │
        ▼
Migrate a few modules
        │
        ▼
Validate
        │
        ▼
Migrate remaining modules
```

This reduces migration risk.

---

# 56. Build Logic Migration: Start With Stable Rules

Good first candidates:

```text
Kotlin configuration
KMP targets
Common compiler options
Test setup
Quality checks
```

Avoid starting with:

```text
Every special-case task
```

Stable rules provide the highest return.

---

# 57. Build Logic Migration: Preserve Behavior

When extracting a convention, the goal is initially:

```text
Same build behavior
+
Less duplication
```

Do not simultaneously:

```text
Upgrade Kotlin
Change targets
Replace dependencies
Rewrite testing
```

unless necessary.

Separating refactoring from functional changes makes failures easier to diagnose.

---

# 58. Build Logic Testing

Build Logic can be validated through:

```text
Test projects
Gradle TestKit
Functional build tests
CI builds
```

For simple conventions, integration through real module builds may be enough.

For complex plugins, dedicated functional tests can provide stronger confidence.

---

# 59. What Should a Build Logic Test Prove?

A useful test might verify:

```text
Plugin applies successfully
Required target exists
Expected task exists
Compiler configuration is applied
Dependency is available
Unexpected platform configuration is absent
```

The exact assertions depend on the convention.

The goal is to verify behavior, not implementation details.

---

# 60. Build Logic and Regression Protection

Imagine a KMP convention accidentally stops configuring:

```text
iOS simulator
```

A functional build test can catch the regression before dozens of modules fail later.

Conceptually:

```text
Build Logic Change
       │
       ▼
Functional Test
       │
       ▼
PASS / FAIL
       │
       ▼
CI
```

This makes Build Logic safer to evolve.

---

# 61. Build Logic and Documentation

A convention should have a clear purpose.

For example:

```text
company.kmp.library
```

should be understandable from its name.

Additional documentation can explain:

```text
Supported targets
Expected module type
Included dependencies
Special behavior
Customization points
```

Good naming reduces the amount of documentation required.

---

# 62. Build Logic and Customization

A convention should provide sensible defaults while allowing legitimate exceptions.

For example:

```text
Standard KMP targets
```

may be applied by default.

But a special module may need:

```text
Additional target
```

The design should make intentional customization possible without forcing developers to copy the entire convention.

---

# 63. Avoid Configuration Escape Hatches Everywhere

Too many customization flags create:

```text
if (specialCaseA)
if (specialCaseB)
if (specialCaseC)
```

inside the convention.

Eventually the convention becomes a giant configuration engine.

When exceptions become common, reconsider whether the modules actually belong to the same convention.

---

# 64. Build Logic and Module Categories

A useful classification can be:

```text
KMP Library
Android Library
Android Application
Compose Multiplatform Module
Published Library
Test Utility
```

Then create conventions around these meaningful categories.

This is more scalable than creating one convention per module.

---

# 65. Build Logic and Repository Architecture

The repository can therefore have:

```text
                    Repository
                        │
        ┌───────────────┴───────────────┐
        ▼                               ▼
   Build Infrastructure             Product Code
        │                               │
        ├── Version Catalog              ├── Shared
        ├── Convention Plugins           ├── Android
        ├── Build Tasks                  └── iOS
        └── Validation
```

The build infrastructure supports the product without becoming part of the product itself.

---

# 66. Build Logic and KMP's Multiplatform Nature

KMP increases the number of build dimensions:

```text
Android
iOS
Simulator
Native
JVM
```

That makes centralized build rules particularly valuable.

Without Build Logic:

```text
Platform differences
+
Module differences
+
Toolchain differences
=
Complex Gradle
```

Build Logic helps control that complexity.

---

# 67. Build Logic and Platform Boundaries

Good Build Logic should respect platform boundaries.

For example:

```text
KMP Convention
   │
   ├── Common configuration
   ├── Android target
   └── iOS targets
```

while:

```text
Android Application Convention
```

handles Android application-specific build concerns.

Don't force every platform concern into one shared plugin.

---

# 68. Build Logic and Native Configuration

iOS and Kotlin/Native may require native-specific configuration.

Some of that belongs in:

```text
KMP convention
```

if every KMP module needs it.

Some may belong in:

```text
specific module
```

if only one module uses the behavior.

Again:

```text
Shared rule → Build Logic
Unique rule → Module
```

---

# 69. Build Logic and Compose

A repository using Compose Multiplatform may have:

```text
KMP Convention
      │
      ▼
Compose Convention
      │
      ▼
UI Modules
```

This lets the KMP convention own platform targets while the Compose convention owns UI-specific build configuration.

The layers remain separate.

---

# 70. Build Logic and Quality

Quality is another cross-cutting concern.

Instead of every module configuring:

```text
Lint
Formatting
Static analysis
Coverage
```

independently:

```text
Quality Convention
        │
        ▼
Applicable Modules
```

This creates consistent engineering standards.

---

# 71. Build Logic and Security

Build infrastructure can also support security controls.

For example:

```text
Dependency verification
Repository restrictions
Artifact validation
Release checks
```

The exact implementation depends on project requirements.

Security-related build rules should be explicit and carefully reviewed because they can affect the entire repository.

---

# 72. Build Logic and Dependency Governance

A repository can use Build Logic to enforce policies such as:

```text
Only approved repositories
Required dependency versions
Forbidden dependency groups
Required quality checks
```

The exact enforcement mechanism should be selected according to the project's needs.

The important idea is:

> **Build Logic can turn repository policies into executable rules.**

---

# 73. Build Logic and Repositories

Repository configuration deserves special attention.

A build may use:

```text
Maven Central
Google
Internal Maven repository
```

Build Logic can help standardize repository policy.

But repositories should not be added casually.

Every repository is part of the dependency supply chain.

---

# 74. Build Logic and Reproducibility

A reproducible build needs consistent:

```text
Toolchain
Dependencies
Repositories
Build rules
```

Build Logic can centralize part of that environment.

Conceptually:

```text
Version Catalog
+
Build Logic
+
Gradle Wrapper
+
Controlled Repositories
        │
        ▼
Reproducible Build
```

No single mechanism guarantees reproducibility.

It is the combination that matters.

---

# 75. Build Logic and Build Scans

Build diagnostics can help understand:

```text
Configuration time
Task execution
Dependency resolution
Cache behavior
```

If a repository becomes slow, inspect the build rather than assuming application code is responsible.

Build Logic is part of the performance surface.

---

# 76. Build Logic and Configuration Cache

Gradle's configuration cache can significantly improve repeated builds when a project is compatible with it.

Poorly designed Build Logic can prevent or reduce those benefits.

Common areas requiring attention include:

```text
Eager access to project state
Undeclared inputs
Direct environment access
Improper task configuration
External side effects during configuration
```

The exact compatibility requirements depend on the Gradle version and APIs used.

The principle is:

> **Write Build Logic that respects Gradle's lazy and declarative model.**

---

# 77. Build Logic and Incremental Builds

Good Build Logic should avoid unnecessarily invalidating work.

For example:

```text
Small build configuration change
```

should not cause:

```text
Every module to rebuild from scratch
```

where avoidable.

This requires careful use of:

```text
Inputs
Outputs
Task configuration
Caching
```

The more sophisticated the build becomes, the more these concepts matter.

---

# 78. Build Logic and Gradle Cacheability

Custom tasks should declare inputs and outputs correctly when appropriate.

Conceptually:

```text
Inputs
  │
  ▼
Task
  │
  ▼
Outputs
```

Gradle can then reason about:

```text
Can this task be skipped?
Can the result be reused?
```

Build Logic that creates custom tasks should respect these principles.

---

# 79. Build Logic and Environment Variables

Be careful when Build Logic reads:

```text
Environment variables
System properties
Local machine paths
```

These can introduce hidden configuration inputs.

For example:

```text
Developer A
    JAVA_HOME = X

Developer B
    JAVA_HOME = Y
```

If build logic behaves differently, reproducibility suffers.

Environment-dependent behavior should be explicit where possible.

---

# 80. Build Logic and Secrets

Never hardcode:

```text
Passwords
API keys
Signing secrets
Tokens
```

into Build Logic.

Build configuration is source code.

Secrets should be supplied through appropriate secure mechanisms.

Build Logic should consume secrets only when required and without exposing them in logs.

---

# 81. Build Logic and Signing

Signing configuration can be centralized for release builds, but secrets should remain external.

Think:

```text
Build Logic
   │
   └── Signing Rules

Secure Environment
   │
   └── Credentials
```

The build knows how to use credentials.

The repository should not contain the credentials themselves.

---

# 82. Build Logic and CI Environment

A good build should make its assumptions explicit:

```text
Required JDK
Required SDK
Required environment variables
Required credentials
```

CI should provide them.

Build Logic should validate missing requirements with useful errors.

---

# 83. Build Logic Error Messages

Bad:

```text
Build failed.
```

Better:

```text
The publishing convention requires a repository URL.
Configure the release repository before running the publishing task.
```

Good Build Logic should fail clearly.

Developers spend less time debugging infrastructure when errors explain the actual requirement.

---

# 84. Build Logic and Diagnostics

Useful diagnostics include:

```text
Which convention is applied?
Which targets are configured?
Which versions are selected?
Which repository is used?
Which task failed?
```

Avoid excessive logging.

The goal is:

```text
Useful information
```

rather than:

```text
Hundreds of lines of Gradle output
```

---

# 85. Build Logic and Developer Onboarding

A well-structured build can be understood through:

```text
gradle/libs.versions.toml
build-logic/
module/build.gradle.kts
```

A new developer can learn:

```text
What dependencies exist
How modules are configured
Which conventions apply
```

without reading every module's entire Gradle file.

---

# 86. Build Logic and Repository Scale

At small scale:

```text
Build Logic
```

may feel like extra abstraction.

At large scale:

```text
Build Logic
```

becomes infrastructure.

The transition usually happens when:

```text
Repeated configuration
+
Multiple modules
+
Multiple platforms
+
Frequent upgrades
```

make centralized rules valuable.

---

# 87. A Practical Build Logic Architecture

A strong example:

```text
build-logic/
│
├── convention/
│   └── src/main/kotlin/
│       ├── kmp.library.gradle.kts
│       ├── android.library.gradle.kts
│       ├── android.application.gradle.kts
│       ├── compose.multiplatform.gradle.kts
│       ├── quality.gradle.kts
│       └── publishing.gradle.kts
│
└── tests/
```

Then:

```text
gradle/
└── libs.versions.toml
```

and:

```text
modules/
├── shared
├── core
├── features
└── androidApp
```

This is only one possible architecture, but it illustrates the separation of responsibilities.

---

# 88. Build Logic Dependency Flow

The complete flow can look like:

```text
                    Version Catalog
                           │
                           ▼
                    Plugin / Library
                      Definitions
                           │
                           ▼
                      Build Logic
                           │
           ┌───────────────┼───────────────┐
           ▼               ▼               ▼
        KMP Rule       Android Rule     Quality Rule
           │               │               │
           └───────────────┼───────────────┘
                           ▼
                         Module
                           │
                           ▼
                         Build
```

This is a scalable Gradle architecture.

---

# 89. Build Logic and Module Simplicity

The end goal is not:

```text
More Gradle files.
```

The end goal is:

```text
Simpler module configuration.
```

A module should ideally communicate:

```text
What kind of module am I?
What is unique about me?
What dependencies do I explicitly need?
```

The shared mechanics should live elsewhere.

---

# 90. Build Logic and Explicitness

There is a balance.

Too little Build Logic:

```text
Everything duplicated.
```

Too much Build Logic:

```text
Nothing is visible in the module.
```

The best architecture keeps:

```text
Common mechanics → centralized
Important module decisions → visible
Unique configuration → local
```

This is one of the most important design principles for build architecture.

---

# 91. Build Logic and Ownership Boundaries

A useful ownership model is:

| Concern | Recommended Owner |
|---|---|
| Dependency versions | Version Catalog |
| Dependency aliases | Version Catalog |
| KMP target policy | KMP Convention |
| Android module policy | Android Convention |
| Compiler policy | Convention / Build Logic |
| Quality policy | Quality Convention |
| Publishing policy | Publishing Convention |
| Product dependency choice | Module |
| Product-specific build behavior | Module |
| Gradle runtime version | Gradle Wrapper |

This keeps responsibilities understandable.

---

# 92. Build Logic Review Checklist

Before merging Build Logic changes:

```text
[ ] Does this solve repeated build behavior?
[ ] Is the responsibility clearly defined?
[ ] Is the convention name descriptive?
[ ] Are versions coming from the intended source of truth?
[ ] Are target assumptions explicit?
[ ] Are dependencies scoped correctly?
[ ] Is configuration lazy where appropriate?
[ ] Does it avoid unnecessary work?
[ ] Does it preserve module customization?
[ ] Does it have useful error messages?
[ ] Does CI validate the change?
```

---

# 93. Build Logic Debugging Checklist

When a module build behaves unexpectedly:

```text
1. Identify the module.
2. List its applied plugins.
3. Identify the convention plugins.
4. Inspect the convention implementation.
5. Check the Version Catalog.
6. Check target configuration.
7. Check source-set configuration.
8. Check dependency configuration.
9. Check generated tasks.
10. Reproduce with the smallest relevant Gradle task.
```

This approach makes Gradle debugging systematic.

---

# 94. Build Logic Upgrade Checklist

Before upgrading the build toolchain:

```text
[ ] Check Gradle compatibility.
[ ] Check Kotlin compatibility.
[ ] Check Android tooling compatibility.
[ ] Check KMP plugin APIs.
[ ] Check convention plugins.
[ ] Check custom tasks.
[ ] Check build-logic dependencies.
[ ] Build Android.
[ ] Build iOS.
[ ] Run shared tests.
[ ] Run CI.
```

Do not forget Build Logic during a toolchain upgrade.

---

# 95. Build Logic Failure Modes

Common problems include:

```text
Convention no longer applies
Target API changed
Plugin API changed
Compiler option deprecated
Task no longer exists
Configuration cache issue
Dependency resolution issue
Build logic dependency mismatch
```

These failures are normal consequences of treating build infrastructure as software.

The solution is not to avoid Build Logic.

The solution is to maintain it properly.

---

# 96. Build Logic and Long-Term Maintenance

A good Build Logic architecture should make future changes easier:

```text
New module
     │
     ▼
Apply convention
     │
     ▼
Inherit standard configuration
```

instead of:

```text
New module
     │
     ▼
Copy an old Gradle file
     │
     ▼
Edit 30 places
     │
     ▼
Hope configuration is correct
```

That difference becomes enormous over years.

---

# 97. Build Logic and the KMP Developer

For an Android developer moving into KMP, Build Logic is worth learning because it explains why modern repositories may not look like traditional Android projects.

You may open a module and see:

```kotlin
plugins {
    id("company.kmp.library")
}
```

and wonder:

> "Where did all the configuration go?"

The answer is:

```text
Build Logic.
```

The module is intentionally thin because the repository has centralized its build conventions.

---

# 98. Reading an Existing KMP Repository

When you enter a new KMP repository, inspect in this order:

```text
1. settings.gradle.kts
2. gradle/libs.versions.toml
3. build-logic/
4. shared module
5. platform modules
6. feature modules
```

This helps you understand:

```text
Project structure
Dependency vocabulary
Build conventions
Targets
Source sets
Application architecture
```

before changing anything.

---

# 99. Build Logic as a Map of the Repository

The build infrastructure often reveals the architecture.

If you see:

```text
kmp.library
android.library
compose.multiplatform
publishing
quality
```

you can infer:

```text
The repository contains shared libraries,
Android-specific libraries,
shared UI,
published modules,
and centralized quality rules.
```

The build is therefore also a map of the engineering organization.

---

# 100. The Final Mental Model

The complete Gradle architecture can now be visualized as:

```text
                         GRADLE
                           │
        ┌──────────────────┼──────────────────┐
        ▼                  ▼                  ▼
 Version Catalog       Build Logic          Wrapper
        │                  │                  │
        │          ┌───────┼────────┐         │
        │          ▼       ▼        ▼         │
        │        KMP     Android  Quality      │
        │      Convention Convention Convention│
        │          │       │        │         │
        └──────────┼───────┼────────┘         │
                   ▼       ▼                  │
                       Modules                │
                           │                  │
                           ▼                  │
                        Targets               │
                           │                  │
                           ▼                  │
                       Compilation            │
                           │                  │
                           ▼                  │
                         Output               │
```

Every piece has a responsibility.

```text
Gradle Wrapper
→ Which Gradle runtime?

Version Catalog
→ Which versions and dependencies?

Build Logic
→ Which build rules?

Convention Plugins
→ How should module categories be configured?

Module Build
→ What is unique about this module?

KMP Targets
→ Where should the code compile?

Source Sets
→ Which code belongs to which compilation?
```

---

# Chapter Takeaways

> [!TIP]
> **Build Logic is the infrastructure layer of a large Gradle repository. It turns repeated configuration into reusable rules, centralizes build behavior, and allows KMP projects to scale across modules and platforms without turning every `build.gradle.kts` file into a copy of the others.**

Remember:

1. Build Logic is reusable code that configures and controls the build.
2. Build Logic is separate from application runtime code.
3. Convention Plugins are one important form of Build Logic.
4. Version Catalogs and Build Logic solve different problems.
5. Version Catalogs describe dependency and plugin metadata.
6. Build Logic describes build behavior and rules.
7. A dedicated `build-logic` included build is a common way to organize reusable build code.
8. Older projects may use `buildSrc`; existing projects should be understood before being migrated.
9. Build Logic should be treated as real software infrastructure.
10. KMP projects benefit from Build Logic because target and source-set configuration can otherwise become repetitive.
11. KMP target policies are strong candidates for centralized conventions.
12. Compiler configuration, testing, quality, and publishing can also be centralized where they are genuinely shared.
13. Build Logic should remain focused and composable.
14. Avoid mega-plugins that configure unrelated concerns.
15. Shared behavior belongs in Build Logic; unique module behavior should remain in the module.
16. Version ownership should remain centralized rather than duplicated inside Build Logic.
17. Build Logic has its own dependency graph and should not be confused with application dependencies.
18. Build Logic should avoid expensive configuration-time work.
19. Lazy configuration and Gradle's configuration model become increasingly important as repositories grow.
20. Custom tasks should declare appropriate inputs and outputs when applicable.
21. Build Logic should be designed with configuration-cache and incremental-build behavior in mind.
22. Build infrastructure can enforce quality, security, dependency, and publishing policies.
23. Build Logic can act as executable documentation for repository standards.
24. Build Logic should provide sensible defaults without hiding every important module decision.
25. Too little Build Logic causes duplication; too much hides the build architecture.
26. Build Logic should have clear ownership boundaries.
27. Functional testing can protect important convention behavior.
28. Build Logic must be considered during Gradle, Kotlin, KMP, and Android tooling upgrades.
29. A well-designed build architecture makes adding new modules much easier.
30. The central principle is: **Build Logic should centralize stable, repeated, meaningful build behavior while keeping module-specific decisions visible and intentional.**

---

# Final Thought

A mature KMP repository does not treat Gradle as a collection of files that happen to make the project compile.

It treats Gradle as an engineering system.

```text
                    Build System
                         │
        ┌────────────────┼────────────────┐
        ▼                ▼                ▼
    Dependency        Build Rules       Modules
    Management           │                │
        │                ▼                ▼
        │           Conventions       Product Code
        │                │                │
        └────────────────┼────────────────┘
                         ▼
                       Build
```

The important shift is this:

> **Build configuration is architecture too.**

A poorly structured application can become difficult to maintain.

The same is true for a poorly structured build.

When KMP projects grow from:

```text
1 shared module
```

to:

```text
10+ shared and platform modules
```

the difference becomes obvious.

Without Build Logic:

```text
Copy
Paste
Modify
Fix
Repeat
```

With well-designed Build Logic:

```text
Define
        │
        ▼
Reuse
        │
        ▼
Validate
        │
        ▼
Scale
```

That is the real purpose of Build Logic.

It is not about hiding Gradle.

It is about giving Gradle an architecture that can grow with the application.
