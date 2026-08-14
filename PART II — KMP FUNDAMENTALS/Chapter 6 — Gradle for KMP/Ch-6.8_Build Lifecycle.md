# Chapter 6 — Gradle for KMP

## Part 8 — Build Lifecycle

> **A Gradle build is not a single command that immediately compiles your code. It moves through a lifecycle: discovering the build, configuring it, constructing the work graph, and finally executing the required tasks. Understanding that lifecycle is essential when you start writing serious KMP Build Logic.**

A KMP project can look simple from the outside:

```text
./gradlew build
```

But a lot happens before the first source file is compiled.

Gradle must determine:

```text
Which build is this?
Which projects participate?
Which plugins are available?
Which modules exist?
Which targets are configured?
Which tasks exist?
Which tasks are required?
Which dependencies are needed?
Which work can be skipped?
```

Only after those decisions can Gradle execute the required work.

A useful high-level model is:

```text
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
      │
      ▼
Build Result
```

This lifecycle is especially important for KMP because one logical module can participate in multiple platform compilations.

---

# 1. The Three Core Phases

Gradle describes three major build phases:

```text
1. Initialization
2. Configuration
3. Execution
```

They happen in this order.

```text
┌─────────────────┐
│  Initialization  │
└────────┬────────┘
         ▼
┌─────────────────┐
│  Configuration   │
└────────┬────────┘
         ▼
┌─────────────────┐
│    Execution     │
└─────────────────┘
```

Each phase has a different responsibility.

---

# 2. Phase 1 — Initialization

Initialization answers:

> **What builds are participating in this Gradle invocation?**

Gradle determines:

```text
Root build
Included builds
Projects
Project hierarchy
```

The settings file plays an important role here.

For a typical project:

```text
settings.gradle.kts
```

is evaluated during this stage.

---

# 3. The Settings File

The settings file describes the structure of the build.

For example:

```kotlin
rootProject.name = "KmpBook"

include(":shared")
include(":androidApp")
include(":core")
```

Conceptually:

```text
settings.gradle.kts
        │
        ▼
Project Structure
        │
        ├── shared
        ├── core
        └── androidApp
```

The settings file is therefore different from a module's:

```text
build.gradle.kts
```

The settings file helps define **what participates in the build**.

---

# 4. Initialization in a KMP Repository

A KMP repository may contain:

```text
Root Build
   │
   ├── shared
   ├── core
   ├── feature-auth
   ├── feature-orders
   └── androidApp
```

During initialization, Gradle discovers the participating structure.

If the repository also uses included build logic:

```text
build-logic/
```

that build can participate as well.

Conceptually:

```text
Initialization
      │
      ├── Root Build
      ├── Included Build Logic
      └── Project Structure
```

---

# 5. Included Builds

Included builds are separate Gradle builds that participate in a larger build.

A common use case is:

```text
build-logic/
```

for convention plugins.

Conceptually:

```text
Main Build
    │
    ├── shared
    ├── core
    └── androidApp
          │
          └── uses
                │
                ▼
            build-logic
```

This is different from simply having another source directory.

An included build has its own Gradle build structure.

---

# 6. Why Initialization Matters

If initialization is wrong:

```text
Project not included
Included build not available
Settings plugin not resolved
Wrong project structure
```

then later phases cannot fix the problem.

The build must first know:

```text
What exists?
```

before it can decide:

```text
How to configure it?
```

---

# 7. Phase 2 — Configuration

After initialization, Gradle moves into configuration.

Configuration answers:

> **How should the participating projects and tasks be configured?**

During this phase Gradle evaluates the build logic for the projects involved in the build.

Conceptually:

```text
Projects
   │
   ▼
Build Scripts + Plugins
   │
   ▼
Extensions
   │
   ▼
Tasks
   │
   ▼
Task Graph
```

This is where much of the Gradle work developers see in:

```text
build.gradle.kts
```

takes place.

---

# 8. What Happens During Configuration?

Gradle may:

```text
Apply plugins
Configure extensions
Configure KMP targets
Create source sets
Register tasks
Configure dependencies
Configure compiler options
Configure Android settings
Configure testing
Build the task graph
```

For KMP, this can be substantial.

A single shared module can result in tasks associated with:

```text
Android
iOS device
iOS simulator
Common source sets
Tests
Packaging
```

depending on the project's configuration.

---

# 9. Plugins Participate in Configuration

When a module declares:

```kotlin
plugins {
    id("company.kmp.library")
}
```

the plugin contributes build configuration.

Conceptually:

```text
Module
  │
  ▼
Plugin
  │
  ├── KMP configuration
  ├── Targets
  ├── Compiler options
  └── Tasks
```

The plugin is not merely a label.

It changes the build model.

---

# 10. Convention Plugins and Lifecycle

This is where the previous section on Convention Plugins becomes important.

A convention plugin is executed as part of project configuration.

For example:

```kotlin
plugins {
    id("company.kmp.library")
}
```

can cause the project to receive:

```text
KMP plugin
Target configuration
Compiler configuration
Testing rules
```

The exact order depends on plugin relationships and Gradle's configuration model.

The important point is:

> **Convention Plugins are part of the configuration architecture of the build.**

---

# 11. Configuration Is Not Execution

This distinction is critical.

Consider:

```kotlin
tasks.register("hello") {
    println("Configured")
    doLast {
        println("Executed")
    }
}
```

The first statement is part of configuration.

The `doLast` action runs when the task executes.

Conceptually:

```text
Configuration
    │
    └── Define the task

Execution
    │
    └── Run the task
```

This distinction explains many Gradle surprises.

---

# 12. A Common Beginner Mistake

A developer may write:

```kotlin
println("Running task")
```

inside a task configuration block and expect it to print only when the task runs.

But:

```kotlin
tasks.register("example") {
    println("Hello")
}
```

runs during configuration.

If you want work to happen during task execution:

```kotlin
tasks.register("example") {
    doLast {
        println("Hello")
    }
}
```

The lifecycle determines when the code runs.

---

# 13. Configuration-Time Code

Code such as:

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

describes how the project should be configured.

This happens before Gradle executes the selected tasks.

Therefore:

```text
Configuration code
```

should generally be:

```text
Fast
Predictable
Declarative
Lazy where appropriate
```

---

# 14. Why Configuration Performance Matters

Suppose a repository has:

```text
100 modules
```

and configuration takes:

```text
5 seconds
```

before any useful task execution starts.

Every developer feels that cost repeatedly.

If build logic performs unnecessary work:

```text
Configuration
   │
   ├── File scanning
   ├── Network calls
   ├── Heavy computation
   └── Eager dependency resolution
```

the build becomes slower even when the requested task is small.

---

# 15. Configuration Should Describe Work

A good mental model is:

```text
Configuration
→ Describe the work

Execution
→ Perform the work
```

For example:

```text
Configuration:
"Compile this source set with these options."

Execution:
"Actually compile the source set."
```

This separation is central to Gradle.

---

# 16. Phase 3 — Execution

After configuration, Gradle executes the selected task graph.

Execution answers:

> **What work actually needs to happen now?**

For example:

```bash
./gradlew test
```

may cause Gradle to execute:

```text
Compile
   │
   ▼
Test
```

while:

```bash
./gradlew assemble
```

may produce a different graph.

The command determines the requested work.

---

# 17. The Task Graph

Gradle does not simply execute tasks in the order they appear in files.

It builds a dependency graph.

For example:

```text
compile
   │
   ▼
test
   │
   ▼
check
   │
   ▼
build
```

If you request:

```text
build
```

Gradle determines which prerequisite tasks are required.

---

# 18. Directed Acyclic Graph

Gradle's task graph can be understood as a:

```text
Directed Acyclic Graph
```

or:

```text
DAG
```

For example:

```text
             compile
            /       \
           ▼         ▼
      process       test
           \         /
            ▼       ▼
              check
                │
                ▼
               build
```

The graph represents task relationships.

---

# 19. Task Dependencies

A task can depend on another task.

For example:

```kotlin
tasks.register("package") {
    dependsOn("compile")
}
```

Conceptually:

```text
compile
   │
   ▼
package
```

Gradle uses these relationships to determine execution order.

---

# 20. Task Graph Is Not a Fixed Sequence

A common misconception is:

```text
Gradle always runs:
A → B → C → D
```

Instead:

```text
Requested Task
      │
      ▼
Required Dependencies
      │
      ▼
Task Graph
      │
      ▼
Executable Tasks
```

The graph changes depending on what you request.

---

# 21. Example: `test`

Suppose a KMP module has:

```text
compileCommonMain
compileCommonTest
test
```

Requesting:

```bash
./gradlew test
```

does not mean:

```text
Run every task in the repository.
```

Gradle selects the tasks required to satisfy the requested task.

This is one reason task graphs matter.

---

# 22. Example: `assemble`

A build command such as:

```bash
./gradlew assemble
```

may involve different tasks from:

```bash
./gradlew test
```

The exact graph depends on the plugins, targets, and module structure.

The important principle is:

> **The requested task determines the relevant portion of the build graph.**

---

# 23. KMP Creates Multiple Task Dimensions

KMP adds another layer of complexity.

A shared module may target:

```text
Android
iOS ARM64
iOS Simulator ARM64
```

Each target can contribute platform-specific tasks.

Conceptually:

```text
                   KMP Module
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
     Android        iOS Device    iOS Simulator
        │              │              │
      Tasks          Tasks           Tasks
```

Therefore, understanding the task graph becomes especially useful when debugging KMP builds.

---

# 24. Common Source Sets and Tasks

KMP has:

```text
commonMain
commonTest
```

and platform source sets such as:

```text
androidMain
iosMain
```

The build system connects these source sets to appropriate compilations and tests.

Conceptually:

```text
commonMain
    │
    ├── Android compilation
    └── iOS compilation
```

The actual task graph is generated by the Kotlin Multiplatform tooling and the configured targets.

---

# 25. One Source Tree, Multiple Compilations

This is one of KMP's most important build concepts.

You can have:

```text
commonMain/
```

but that code may be compiled for:

```text
Android
iOS
```

Therefore:

```text
Source code
      │
      ▼
KMP configuration
      │
      ├── Android compilation
      └── iOS compilation
```

Gradle coordinates the build work.

---

# 26. Platform-Specific Source Sets

Platform-specific source sets such as:

```text
androidMain
iosMain
```

feed platform-specific compilation.

Conceptually:

```text
commonMain
     │
     ├──────────────┐
     ▼              ▼
androidMain      iosMain
     │              │
     ▼              ▼
Android          iOS
```

The build lifecycle connects these pieces.

---

# 27. Task Configuration Avoidance

Large builds benefit from avoiding unnecessary task configuration.

Instead of eagerly configuring every possible task:

```text
Configure everything
```

Gradle encourages lazy APIs where appropriate.

For example:

```kotlin
tasks.register("generateReport") {
    ...
}
```

is preferred over patterns that eagerly create and configure tasks when the task may never be needed.

---

# 28. `register` vs Eager Task Creation

Conceptually:

```kotlin
tasks.create("example")
```

creates the task eagerly.

Where appropriate:

```kotlin
tasks.register("example")
```

allows Gradle to configure the task lazily.

The distinction becomes increasingly important in large repositories.

---

# 29. Why Lazy Configuration Helps

Imagine:

```text
100 modules
×
20 tasks per module
```

That is:

```text
2000 tasks
```

If every task is eagerly configured:

```text
Configuration cost increases.
```

Lazy task registration allows Gradle to avoid unnecessary work.

This is one of the techniques that helps large builds scale.

---

# 30. `tasks.named`

When a task already exists, prefer lazy access where appropriate:

```kotlin
tasks.named("test") {
    // configure
}
```

rather than eagerly realizing it unnecessarily.

The exact choice depends on whether the task is guaranteed to exist and which plugin creates it.

---

# 31. `configureEach`

The same principle applies to collections of Gradle model objects.

For example:

```kotlin
tasks.withType<SomeTask>().configureEach {
    // configuration
}
```

allows Gradle to configure relevant instances without unnecessarily realizing every task.

This style is particularly useful in reusable Build Logic.

---

# 32. Eager Configuration Can Hurt KMP Builds

A KMP project may already generate many tasks.

Adding custom Build Logic that eagerly configures:

```text
Every task
Every target
Every configuration
Every dependency
```

can multiply configuration work.

Therefore:

```text
KMP
+
Large repository
+
Poor Build Logic
=
Slow Gradle configuration
```

---

# 33. Configuration Cache

Gradle also provides a Configuration Cache that can reuse the result of the configuration phase for compatible builds.

Conceptually:

```text
First build
   │
   ├── Initialization
   ├── Configuration
   └── Cache configuration result

Later build
   │
   └── Reuse compatible configuration
```

This can significantly reduce repeated configuration work.

---

# 34. Configuration Cache Is Not the Build Cache

These are different concepts.

### Configuration Cache

Optimizes:

```text
Configuration phase
```

### Build Cache

Can reuse:

```text
Task outputs
```

Think:

```text
Configuration Cache
→ "Do I need to configure the build again?"

Build Cache
→ "Do I need to perform this task again?"
```

Both can contribute to faster builds.

---

# 35. Why Build Logic Must Respect Configuration Cache

Build Logic that depends on:

```text
Undeclared state
Mutable global state
Unexpected environment access
Improper lifecycle hooks
```

may create configuration-cache problems.

Modern Gradle builds should be designed with the configuration model in mind.

---

# 36. Configuration Inputs

Build configuration may depend on:

```text
Gradle properties
Environment values
System properties
Project files
Plugin versions
Source configuration
```

If these inputs affect the build, they should be handled in ways compatible with Gradle's model.

Hidden inputs make builds harder to reproduce and cache.

---

# 37. Execution Is Where Real Work Happens

During execution, Gradle can:

```text
Compile Kotlin
Compile Java
Compile native code
Run tests
Package artifacts
Generate resources
Run verification
Publish artifacts
```

The work depends on the task graph.

This is why execution should not be confused with configuration.

---

# 38. `doFirst` and `doLast`

Gradle task actions execute during the execution phase.

For example:

```kotlin
tasks.register("example") {
    doFirst {
        println("Before task actions")
    }

    doLast {
        println("After task actions")
    }
}
```

These actions belong to execution.

By contrast:

```kotlin
println("Configured")
```

inside the configuration block runs during configuration.

---

# 39. Why This Difference Matters

Consider:

```kotlin
tasks.register("generate") {
    val input = File("input.txt")

    doLast {
        println(input.readText())
    }
}
```

The file is read during task execution.

If instead you do:

```kotlin
val content = File("input.txt").readText()
```

during configuration, the file is read even if:

```text
generate
```

is never executed.

That can create unnecessary work.

---

# 40. Configuration-Time I/O

Avoid expensive operations such as:

```text
Network calls
Large file reads
Repository scans
Process execution
CPU-heavy calculations
```

during configuration unless there is a strong reason.

Why?

Because configuration may happen even when the requested task does not need that work.

---

# 41. A Simple Example

Bad pattern:

```kotlin
val version = File("version.txt").readText()

tasks.register("publish") {
    ...
}
```

The file is read during configuration.

Better design:

```kotlin
tasks.register("publish") {
    doLast {
        val version = File("version.txt").readText()
        ...
    }
}
```

The exact implementation should use Gradle's provider and input APIs where appropriate, but the principle is important:

```text
Defer work until it is needed.
```

---

# 42. Build Lifecycle and Dependencies

Dependency declaration and dependency resolution are related but not identical.

A module may declare:

```kotlin
dependencies {
    implementation(libs.kotlinx.coroutines.core)
}
```

during configuration.

Gradle can defer much of the actual resolution work until it is required.

This distinction matters for build performance.

---

# 43. Eager Dependency Resolution

Build Logic can accidentally force dependency resolution early.

For example, directly querying resolved dependency state during configuration can cause work to happen sooner than necessary.

Avoid this unless the build genuinely requires it.

Prefer Gradle's lazy APIs and provider-based configuration where possible.

---

# 44. Build Lifecycle and Plugins

Gradle has different plugin scopes.

Conceptually:

```text
Init Plugin
     │
     ▼
Initialization

Settings Plugin
     │
     ▼
Settings / Build Structure

Project Plugin
     │
     ▼
Project Configuration
```

This is useful when deciding where a piece of Build Logic belongs.

---

# 45. Init Plugins

Init plugins operate at the Gradle build level.

They can affect builds through initialization.

They are useful for scenarios such as:

```text
Enterprise-wide build policy
Global repository configuration
Environment-specific setup
Build diagnostics
```

They should be used carefully because they can affect builds beyond a single repository.

---

# 46. Settings Plugins

Settings plugins operate around the settings/build-structure level.

They are useful for concerns such as:

```text
Project inclusion
Plugin management
Repository management
Build structure
```

This makes them different from ordinary project conventions.

---

# 47. Project Plugins

Project plugins are the most common type for module-level Build Logic.

Examples:

```text
company.kmp.library
company.android.library
company.quality
```

They configure individual projects.

Conceptually:

```text
Project Plugin
      │
      ▼
Project Configuration
      │
      ▼
Tasks + Extensions + Targets
```

---

# 48. Choosing the Correct Lifecycle Level

A useful rule:

```text
Whole Gradle environment
        → Init level

Build structure
        → Settings level

Module configuration
        → Project level
```

Do not use a project convention to solve a settings problem.

Do not use an init script when a repository-level convention is sufficient.

The narrower scope is often easier to understand and control.

---

# 49. Build Lifecycle and `settings.gradle.kts`

A KMP project may use settings for:

```text
Root project name
Included modules
Included builds
Plugin management
Dependency resolution management
```

Conceptually:

```text
settings.gradle.kts
        │
        ▼
Build Structure
        │
        ▼
Project Configuration
```

This is why the settings file is the starting point for understanding a complex Gradle repository.

---

# 50. Build Lifecycle and `build.gradle.kts`

The project build file is primarily concerned with:

```text
Plugins
Extensions
Dependencies
Tasks
Module configuration
```

It operates at the project level.

For a KMP module:

```text
shared/build.gradle.kts
```

may configure:

```text
KMP
Targets
Source Sets
Dependencies
Compiler options
```

---

# 51. Build Lifecycle and Convention Plugins

Convention Plugins sit naturally in the project configuration layer.

For example:

```text
feature-auth/build.gradle.kts
          │
          ▼
company.kmp.library
          │
          ▼
Build Logic
          │
          ▼
KMP Configuration
```

This keeps module scripts small while preserving explicit plugin application.

---

# 52. Build Lifecycle and Task Graph Construction

Once projects are configured, Gradle constructs the task graph.

Conceptually:

```text
Requested Task
      │
      ▼
Task Dependencies
      │
      ▼
Task Graph
      │
      ▼
Execution
```

The graph represents the work necessary to satisfy the request.

---

# 53. Why Task Graphs Matter for Debugging

Suppose:

```bash
./gradlew build
```

takes a long time.

You should ask:

```text
Which tasks are running?
Which tasks are dependencies?
Which target is being built?
Which module caused the work?
```

Instead of simply assuming:

```text
"KMP is slow."
```

The task graph gives you a concrete place to investigate.

---

# 54. KMP Task Graph Example

A simplified graph might look like:

```text
                    build
                      │
             ┌────────┴────────┐
             ▼                 ▼
          check             assemble
             │                 │
       ┌─────┴─────┐      ┌────┴────┐
       ▼           ▼      ▼         ▼
    commonTest   androidTest   Android   iOS
```

The real graph is more detailed and depends on:

```text
Targets
Plugins
Modules
Requested task
Build configuration
```

The diagram is a mental model, not a literal task list.

---

# 55. `dependsOn` vs Input/Output Relationships

Gradle can model relationships through explicit task dependencies:

```kotlin
tasks.named("package") {
    dependsOn("compile")
}
```

But Gradle's model also relies heavily on task inputs and outputs.

The broader principle is:

> **Describe relationships so Gradle can determine the correct execution graph.**

Avoid manually forcing ordering when a real data dependency is the better model.

---

# 56. Ordering Is Not the Same as Dependency

Sometimes developers use:

```kotlin
mustRunAfter
```

when they actually need:

```text
dependsOn
```

These concepts are different.

### Dependency

```text
A depends on B
```

means B is required before A.

### Ordering

```text
A must run after B
```

controls order but does not necessarily make B required.

Understanding this distinction prevents incorrect build graphs.

---

# 57. Parallel Execution

Independent tasks can sometimes execute in parallel.

For example:

```text
Module A compile
       │
       └──────┐
              ▼
           independent
              ▲
       ┌──────┘
Module B compile
```

Gradle can exploit task independence when the build is configured appropriately.

This is another reason to declare accurate task relationships.

---

# 58. Incorrect Dependencies Reduce Parallelism

Suppose you declare:

```text
Module A depends on Module B
```

even though there is no real dependency.

Now Gradle may have to execute:

```text
B → A
```

instead of:

```text
A
B
```

in parallel.

Unnecessary dependencies can therefore hurt build performance.

---

# 59. Accurate Task Graphs Matter

A good build graph should represent reality.

```text
Real dependency
      │
      ▼
Declare dependency

No dependency
      │
      ▼
Don't create artificial ordering
```

This allows Gradle to optimize execution more effectively.

---

# 60. Build Lifecycle and Incremental Work

Gradle attempts to avoid work that does not need to be repeated.

Depending on the task and configuration, Gradle can use:

```text
Up-to-date checks
Task outputs
Build Cache
Incremental processing
Configuration Cache
```

These mechanisms operate at different levels.

---

# 61. Up-to-Date Tasks

A task may be considered up-to-date when its relevant inputs and outputs have not changed.

Conceptually:

```text
Inputs unchanged
      │
      ▼
Outputs still valid
      │
      ▼
Skip task execution
```

This can make repeated builds much faster.

---

# 62. Build Cache

The Build Cache can reuse task outputs across builds when the task and cache configuration allow it.

Think:

```text
Task Inputs
     │
     ▼
Previously Cached Output?
     │
    Yes
     │
     ▼
Reuse Output
```

This is different from Configuration Cache.

---

# 63. Configuration Cache vs Build Cache

Keep the distinction clear:

```text
Configuration Cache
────────────────────
Caches configuration result.

Build Cache
───────────
Caches task outputs.
```

A large KMP build can benefit from both.

---

# 64. Build Lifecycle and CI

CI commonly runs commands such as:

```bash
./gradlew check
./gradlew build
```

The lifecycle is the same:

```text
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

What changes is the environment and requested task.

---

# 65. Local Build vs CI Build

The same project may be built:

```text
Developer machine
```

and:

```text
CI machine
```

Both use the same lifecycle.

But CI may execute:

```text
Full verification
All tests
Quality checks
Packaging
Publishing
```

while a developer may execute:

```text
One module test
One platform build
```

Understanding the lifecycle helps explain the difference.

---

# 66. Build Lifecycle and Release Pipelines

A release pipeline may look like:

```text
Checkout
   │
   ▼
Gradle Initialization
   │
   ▼
Configuration
   │
   ▼
Verification
   │
   ▼
Build
   │
   ▼
Publish
```

The Gradle lifecycle sits inside the larger CI/CD lifecycle.

They should not be confused.

---

# 67. Build Lifecycle and Android

In a KMP project, Android-specific Gradle plugins participate in the same Gradle lifecycle.

Conceptually:

```text
Gradle
  │
  ▼
KMP + Android Plugins
  │
  ▼
Android Target
  │
  ▼
Android Tasks
```

The Android build is one part of the overall multiplatform build.

---

# 68. Build Lifecycle and iOS

The same KMP project may configure iOS targets.

Conceptually:

```text
Gradle
  │
  ▼
KMP Plugin
  │
  ▼
iOS Targets
  │
  ▼
Kotlin/Native Tasks
  │
  ▼
Native Artifacts
```

The exact integration depends on the KMP project structure and how the iOS application is connected to the shared module.

---

# 69. Gradle Does Not Mean "Android Only"

This is an important KMP mindset.

Gradle is the build orchestration system.

KMP uses it to coordinate multiplatform compilation and related tasks.

So:

```text
Gradle
   │
   ├── Android
   ├── JVM
   ├── iOS
   └── Other Kotlin targets
```

can be part of the same repository.

---

# 70. Build Lifecycle and IDEs

Android Studio and IntelliJ-based environments may invoke Gradle tasks behind the scenes.

For example:

```text
Sync
Build
Test
Run
```

can involve Gradle configuration and task execution.

Therefore, an IDE problem may sometimes actually be:

```text
Gradle configuration problem
```

rather than:

```text
IDE problem
```

---

# 71. Gradle Sync

During a sync-like operation, the IDE needs information about:

```text
Projects
Plugins
Dependencies
Source Sets
Tasks
Build Configuration
```

The Gradle model helps the IDE understand the project.

If configuration fails:

```text
IDE model
```

may also become incomplete.

---

# 72. Build Lifecycle and Debugging

When debugging a Gradle problem, identify the phase.

Ask:

```text
Did initialization fail?
Did configuration fail?
Did task graph construction fail?
Did execution fail?
```

This immediately narrows the search.

---

# 73. Initialization Failure

Typical symptoms:

```text
Settings error
Project not found
Included build failure
Plugin management failure
```

Investigate:

```text
settings.gradle.kts
Included builds
Plugin repositories
Project inclusion
```

---

# 74. Configuration Failure

Typical symptoms:

```text
Plugin configuration error
Unknown extension
Invalid target configuration
Compiler configuration error
Dependency declaration error
```

Investigate:

```text
build.gradle.kts
Convention Plugins
Plugin versions
Build Logic
KMP configuration
```

---

# 75. Task Graph Failure

Typical symptoms:

```text
Task not found
Dependency cycle
Unexpected task selection
Invalid task relationship
```

Investigate:

```text
Task registration
Task dependencies
Plugin configuration
Requested task
```

---

# 76. Execution Failure

Typical symptoms:

```text
Compilation error
Test failure
Packaging failure
Linker failure
Native compilation failure
```

Investigate:

```text
Source code
Compiler
Dependencies
Platform tooling
Task inputs
Environment
```

The lifecycle phase tells you where to start.

---

# 77. Build Lifecycle and KMP Compiler Errors

Suppose:

```text
./gradlew build
```

fails with:

```text
Compilation error
```

The lifecycle has already passed:

```text
Initialization
Configuration
Task graph construction
```

The failure is occurring during execution.

That means changing:

```text
settings.gradle.kts
```

may be irrelevant unless the configuration caused the wrong task or inputs to be selected.

---

# 78. Build Lifecycle and Plugin Errors

If you see:

```text
Could not apply plugin
```

the problem is usually in configuration.

Possible causes:

```text
Plugin version mismatch
Unsupported Gradle version
Invalid plugin configuration
Incorrect convention
Missing plugin dependency
```

This is different from a source compilation failure.

---

# 79. Build Lifecycle and Build Logic

A useful debugging map is:

```text
Build Logic
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

If Build Logic creates the wrong task configuration, the eventual failure may appear much later.

Therefore, debugging should trace the problem backward:

```text
Failed task
   │
   ▼
Task configuration
   │
   ▼
Convention Plugin
   │
   ▼
Build Logic
```

---

# 80. Lifecycle Hooks

Gradle exposes lifecycle-related APIs that can observe or react to build events.

Examples include concepts such as:

```text
settings evaluated
projects loaded
before project
after project
projects evaluated
```

These APIs can be useful for specialized build tooling.

But lifecycle hooks should not become the default solution for ordinary module configuration.

---

# 81. Why Lifecycle Hooks Can Become Dangerous

Global lifecycle hooks can create hidden behavior.

For example:

```text
Every project
      │
      ▼
Hidden configuration
```

A developer may open:

```text
feature-auth/build.gradle.kts
```

and not realize that another global hook changes its configuration.

Prefer:

```text
Explicit plugin application
```

when a convention can solve the problem cleanly.

---

# 82. Build Logic Should Prefer Plugins Over Global Hooks

Instead of:

```text
Configure every project globally
```

prefer:

```kotlin
plugins {
    id("company.kmp.library")
}
```

This makes the configuration boundary visible.

Global hooks still have legitimate use cases, but they should be applied intentionally.

---

# 83. Build Lifecycle and `afterEvaluate`

`afterEvaluate` is a lifecycle callback that can modify configuration after a project has been evaluated.

It may be useful in some advanced scenarios, but it can also make build logic harder to reason about.

Prefer direct configuration and plugin APIs when possible.

The more a build depends on lifecycle timing tricks, the harder it can become to maintain.

---

# 84. Prefer Model-Based Configuration

A modern Gradle mindset is:

```text
Declare model
      │
      ▼
Gradle understands model
      │
      ▼
Gradle builds task graph
      │
      ▼
Gradle executes work
```

Avoid relying unnecessarily on:

```text
"Run this code after that file happened to be evaluated."
```

Explicit model relationships are easier to understand.

---

# 85. Build Lifecycle and Providers

Gradle's Provider APIs help represent values lazily.

Conceptually:

```text
Provider<T>
    │
    ▼
Value available when needed
```

This can help Build Logic avoid eagerly reading:

```text
Environment
Properties
Files
Task values
```

before they are needed.

---

# 86. Why Lazy Values Matter

Suppose a property is needed only by:

```text
publish
```

There is little reason to resolve it while configuring:

```text
test
```

or:

```text
compile
```

Lazy values help keep the build efficient and model-driven.

---

# 87. Build Lifecycle and Environment

Build behavior may depend on:

```text
Local machine
CI
Release mode
Developer properties
```

Such inputs should be modeled carefully.

For example:

```text
Release credentials
```

should not be loaded during configuration for a simple local unit test unless required.

---

# 88. Build Lifecycle and Secrets

Never make the lifecycle dependent on secrets unnecessarily.

Bad design:

```text
Every Gradle invocation
       │
       ▼
Requires release credentials
```

Better:

```text
Publishing task
       │
       ▼
Requires publishing credentials
```

This keeps unrelated development workflows independent.

---

# 89. Build Lifecycle and Network Access

Avoid unnecessary network access during configuration.

For example:

```text
./gradlew test
```

should not need to call an unrelated external service simply because a custom convention plugin performs a network request.

Network-dependent behavior belongs as close as possible to the task that genuinely requires it.

---

# 90. Build Lifecycle and Reproducibility

A predictable build should behave consistently for the same:

```text
Inputs
Configuration
Toolchain
Dependencies
Environment assumptions
```

A build that performs hidden configuration-time operations can become difficult to reproduce.

Lifecycle discipline improves reliability.

---

# 91. A Complete KMP Build Lifecycle

A simplified KMP build can be visualized as:

```text
                 ./gradlew build
                         │
                         ▼
                 Initialization
                         │
             ┌───────────┴───────────┐
             ▼                       ▼
        settings.gradle.kts      Included Builds
             │                       │
             └───────────┬───────────┘
                         ▼
                    Configuration
                         │
             ┌───────────┼───────────┐
             ▼           ▼           ▼
          Plugins      Targets    Dependencies
             │           │           │
             └───────────┼───────────┘
                         ▼
                    Task Graph
                         │
             ┌───────────┼───────────┐
             ▼           ▼           ▼
          Android        iOS       Tests
             │           │           │
             └───────────┼───────────┘
                         ▼
                     Execution
                         │
                         ▼
                       Output
```

This is the mental model to keep while working with KMP Gradle builds.

---

# 92. Example: What Happens When You Run `build`

Imagine:

```bash
./gradlew build
```

A simplified sequence is:

```text
1. Gradle starts.
2. Settings are evaluated.
3. Participating projects are identified.
4. Included build logic is made available.
5. Project plugins are applied.
6. KMP targets are configured.
7. Source sets and dependencies are configured.
8. Tasks are registered/configured.
9. Gradle determines the task graph.
10. Required tasks execute.
11. Outputs are produced.
```

The exact internal sequence is more detailed, but this model is enough to reason about most everyday build behavior.

---

# 93. Example: What Happens When You Run a Test

Suppose:

```bash
./gradlew :shared:test
```

Conceptually:

```text
Initialization
      │
      ▼
Configuration
      │
      ▼
Identify :shared:test
      │
      ▼
Determine prerequisites
      │
      ▼
Build task graph
      │
      ▼
Execute required compilation/test tasks
```

Gradle does not need to execute every unrelated module task merely because those modules exist.

---

# 94. Example: What Happens When You Run an Android Build

A command targeting Android may produce:

```text
Initialization
      │
      ▼
Configuration
      │
      ▼
KMP + Android configuration
      │
      ▼
Android-related task graph
      │
      ▼
Compilation
      │
      ▼
Packaging
```

The exact tasks depend on the Android plugin, KMP setup, build types, variants, and requested task.

---

# 95. Example: What Happens When You Build iOS Shared Code

A KMP iOS build can involve:

```text
Initialization
      │
      ▼
Configuration
      │
      ▼
KMP iOS target configuration
      │
      ▼
Native compilation/linking tasks
      │
      ▼
Execution
      │
      ▼
Native artifact
```

The Gradle lifecycle remains the same even though the output target differs.

---

# 96. Lifecycle and Build Performance

A useful performance model is:

```text
Build Time
   =
Initialization
+
Configuration
+
Task Execution
```

Optimization therefore requires asking:

```text
Is initialization slow?
Is configuration slow?
Are tasks slow?
Are tasks being executed unnecessarily?
Are cache opportunities being missed?
```

Do not optimize only the compilation phase.

---

# 97. Configuration Bottlenecks

If:

```text
./gradlew help
```

is already slow, that is a strong signal that the problem may be in:

```text
Initialization
Configuration
Build Logic
Plugin loading
```

because `help` does not represent a normal full application build.

This can be a useful diagnostic technique.

---

# 98. Execution Bottlenecks

If configuration is fast but:

```text
./gradlew build
```

is slow, investigate:

```text
Compilation
Tests
Native linking
Packaging
Task inputs
Cache behavior
```

The lifecycle helps separate the problem.

---

# 99. Build Lifecycle and `help`

A simple command such as:

```bash
./gradlew help
```

can be useful when investigating configuration behavior.

It allows you to examine how much work happens before meaningful task execution.

It can also help reveal configuration failures that are unrelated to compilation.

---

# 100. Build Lifecycle and `tasks`

Similarly:

```bash
./gradlew tasks
```

can help you inspect the tasks available to the build.

For a KMP project, this can reveal how many platform-specific and module-specific tasks exist.

That is useful when learning an unfamiliar repository.

---

# 101. Build Lifecycle and Task Inspection

When debugging, useful commands can include:

```bash
./gradlew tasks
```

and targeted task invocations such as:

```bash
./gradlew :shared:test
```

or:

```bash
./gradlew :shared:build
```

The exact task names vary by project.

The important idea is to narrow the requested graph while investigating.

---

# 102. Avoid Debugging the Entire Repository at Once

If:

```text
100-module repository
```

has a failure, start with:

```text
One module
One target
One task
```

Conceptually:

```text
Entire Build
     │
     ▼
Affected Module
     │
     ▼
Affected Task
```

This reduces noise.

---

# 103. Build Lifecycle and Logging

Gradle provides different logging levels for diagnostics.

For example, more verbose logging can help identify:

```text
Configuration behavior
Task execution
Dependency resolution
```

Use increased logging when needed, but avoid treating verbose logs as a substitute for understanding the lifecycle.

---

# 104. Build Lifecycle and Build Scans

Build diagnostics can provide additional information about:

```text
Task execution
Performance
Dependency resolution
Configuration
```

For teams maintaining large builds, such diagnostics can help identify systemic problems.

The key is to investigate the phase and task rather than simply looking at total build time.

---

# 105. Build Lifecycle and Error Ownership

When a build fails, identify which layer owns the problem.

```text
Settings
→ Build structure

Plugin
→ Configuration

Convention
→ Build Logic

Task
→ Execution

Compiler
→ Source/toolchain

Platform
→ Android/iOS environment
```

This avoids changing unrelated parts of the repository.

---

# 106. Build Lifecycle and Architecture

The lifecycle is not just a Gradle implementation detail.

It influences how you design Build Logic.

A good Build Logic architecture respects:

```text
Initialization
→ Define build structure

Configuration
→ Define build behavior

Execution
→ Perform work
```

This separation makes the build easier to reason about.

---

# 107. The Most Important Rule

> **Do not perform work during configuration that can safely happen during task execution.**

This rule prevents many build-performance problems.

For example:

```text
Don't:
Configuration → expensive file scan

Prefer:
Execution → expensive file scan
```

when the scan is only needed by a specific task.

---

# 108. The Second Important Rule

> **Do not make task relationships more restrictive than the real dependency graph.**

If two tasks are independent:

```text
Keep them independent.
```

This allows Gradle more freedom to optimize execution.

---

# 109. The Third Important Rule

> **Make Build Logic explicit.**

Prefer:

```kotlin
plugins {
    id("company.kmp.library")
}
```

over invisible global configuration when a project-level convention is sufficient.

Explicit build architecture is easier to maintain.

---

# 110. The Fourth Important Rule

> **Use lazy configuration where appropriate.**

Prefer:

```kotlin
tasks.register(...)
tasks.named(...)
configureEach(...)
Provider<T>
```

where those APIs match the situation.

This reduces unnecessary configuration and helps large builds scale.

---

# 111. The Fifth Important Rule

> **Understand the requested task before debugging the whole build.**

If the command is:

```bash
./gradlew :shared:test
```

start with:

```text
:shared:test
```

and its dependencies.

Do not assume the entire repository is involved.

---

# 112. Build Lifecycle Review Checklist

When reviewing KMP Build Logic, ask:

```text
[ ] Is this initialization logic?
[ ] Is this settings-level logic?
[ ] Is this project configuration?
[ ] Is this task execution?
[ ] Is work happening too early?
[ ] Can the value be lazy?
[ ] Can the task be registered lazily?
[ ] Is the task dependency real?
[ ] Are unnecessary dependencies being introduced?
[ ] Does the logic work with the project's caching strategy?
[ ] Does it work for all intended KMP targets?
[ ] Does it remain understandable to module developers?
```

---

# 113. Lifecycle Debugging Checklist

When a Gradle build fails:

```text
1. Identify the command.
2. Identify the failing phase.
3. Identify the project/module.
4. Identify the plugin or convention involved.
5. Identify the task.
6. Inspect task dependencies.
7. Check platform target configuration.
8. Check Build Logic.
9. Check toolchain compatibility.
10. Re-run the smallest relevant task.
```

This turns Gradle debugging into a structured process.

---

# 114. Final Mental Model

Keep this diagram in mind:

```text
                         GRADLE
                           │
                           ▼
                  ┌─────────────────┐
                  │  Initialization  │
                  └────────┬────────┘
                           │
                           ▼
                  settings.gradle.kts
                           │
                           ▼
                  Build Structure
                           │
                           ▼
                  ┌─────────────────┐
                  │  Configuration  │
                  └────────┬────────┘
                           │
              ┌────────────┼────────────┐
              ▼            ▼            ▼
           Plugins       Targets    Dependencies
              │            │            │
              └────────────┼────────────┘
                           ▼
                     Build Logic
                           │
                           ▼
                    Task Graph / DAG
                           │
                           ▼
                  ┌─────────────────┐
                  │    Execution    │
                  └────────┬────────┘
                           │
             ┌─────────────┼─────────────┐
             ▼             ▼             ▼
          Android          iOS          Tests
             │             │             │
             └─────────────┼─────────────┘
                           ▼
                         Output
```

Once this model becomes familiar, Gradle errors become much easier to classify.

You stop asking:

> "Why is Gradle doing this?"

and start asking:

> "At which phase was this decision made?"

That is a much more powerful question.

---

# Chapter Takeaways

> [!TIP]
> **Gradle's Build Lifecycle is the foundation behind every KMP build. Initialization determines what participates, configuration determines how the build is modeled, the task graph determines what work is required, and execution performs that work.**

Remember:

1. Gradle has three primary phases: initialization, configuration, and execution.
2. Initialization determines the participating builds and projects.
3. `settings.gradle.kts` plays a central role in initialization.
4. Included builds such as `build-logic` participate in the larger Gradle build.
5. Configuration evaluates project build logic and establishes the build model.
6. Plugins and Convention Plugins participate in project configuration.
7. Configuration is different from task execution.
8. Code inside a task configuration block can execute during configuration.
9. `doFirst` and `doLast` actions execute when the task runs.
10. Gradle constructs a task graph based on requested tasks and their relationships.
11. The task graph is modeled as a directed acyclic graph.
12. KMP can produce many platform- and source-set-specific tasks.
13. The same common source can participate in multiple platform compilations.
14. Task configuration avoidance is important for large builds.
15. `tasks.register` supports lazy task registration.
16. `tasks.named` and `configureEach` can help avoid unnecessary realization.
17. Configuration should generally be fast, predictable, and declarative.
18. Expensive file, network, or CPU work should not be performed during configuration without a strong reason.
19. Dependency declaration and dependency resolution are separate concepts.
20. Build Logic should avoid forcing dependency resolution earlier than necessary.
21. Configuration Cache optimizes the configuration phase.
22. Build Cache focuses on reusable task outputs.
23. Configuration Cache and Build Cache solve different problems.
24. Accurate task dependencies allow Gradle to optimize execution.
25. Artificial task ordering can reduce parallelism.
26. `dependsOn` and ordering relationships such as `mustRunAfter` are not interchangeable.
27. Init, settings, and project plugins operate at different levels of the Gradle lifecycle.
28. Project-level Convention Plugins are often appropriate for module build conventions.
29. Global lifecycle hooks should be used carefully because they can create hidden behavior.
30. Build Logic should prefer explicit, model-based configuration over lifecycle timing tricks.
31. Provider-based and lazy APIs help defer values and work until they are needed.
32. KMP adds platform dimensions that make understanding the task graph particularly valuable.
33. Android and iOS builds can participate in the same Gradle lifecycle through KMP tooling.
34. `./gradlew tasks` can help explore an unfamiliar build.
35. Targeted Gradle tasks are useful when debugging large repositories.
36. If initialization fails, inspect settings and build structure.
37. If configuration fails, inspect plugins, conventions, and build logic.
38. If task graph construction fails, inspect task registration and dependencies.
39. If execution fails, inspect source code, toolchains, platform configuration, and task inputs.
40. Build performance should be analyzed across initialization, configuration, and execution.
41. The most important Build Logic principle is to describe work during configuration and perform work during execution.
42. The second important principle is to model real dependencies accurately.
43. The third important principle is to keep build behavior explicit.
44. The fourth important principle is to use lazy configuration where appropriate.
45. The fifth important principle is to debug the smallest relevant task graph first.
46. The central mental model is: **Initialization defines the build, Configuration defines the work, the Task Graph defines what is required, and Execution performs that work.**

---

# Final Thought

A Gradle build may start with one command:

```bash
./gradlew build
```

but that command represents a complete build lifecycle:

```text
What exists?
      │
      ▼
Initialization
      │
      ▼
How should it behave?
      │
      ▼
Configuration
      │
      ▼
What work is actually required?
      │
      ▼
Task Graph
      │
      ▼
Do the work.
      │
      ▼
Execution
```

For KMP, this mental model becomes even more important because one shared codebase can produce work for multiple platforms.

```text
                   commonMain
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
       Android       iOS Device   iOS Simulator
          │            │            │
          └────────────┼────────────┘
                       ▼
                  Gradle Tasks
                       │
                       ▼
                    Execution
```

The goal of Build Logic is not to fight this lifecycle.

It is to work with it.

Good KMP Build Logic:

```text
Configures intentionally
        │
        ▼
Creates accurate task relationships
        │
        ▼
Defers unnecessary work
        │
        ▼
Uses caching effectively
        │
        ▼
Scales across platforms and modules
```

Once you understand the lifecycle, Gradle stops looking like a collection of mysterious scripts.

It becomes what it really is:

> **A model-driven build system that turns project structure and build rules into a graph of work, then executes only the work required to produce the requested result.**
