# Chapter 5 — Creating Your First KMP Project

## Part 4 — Build

> **A KMP project is not complete when the code compiles once. It is complete when you understand what the build is doing, why it is doing it, and how Android and iOS outputs are produced.**

Once the project structure is clear, the next step is to understand the **build**.

For an Android developer, `./gradlew assembleDebug` may feel familiar.

In KMP, however, the build has another dimension:

```text
One Project
    │
    ├── Android Compilation
    │
    ├── Shared Kotlin Compilation
    │
    └── iOS Native Compilation / Integration
```

The build system has to understand:

```text
Source Sets
+
Targets
+
Dependencies
+
Platform Toolchains
+
Artifacts
```

That is what makes the KMP build different from a normal Android-only build.

---

# 1. What Does "Build" Mean in KMP?

At the simplest level:

```text
Source Code
    │
    ▼
Compiler
    │
    ▼
Platform Output
```

But a KMP project has more than one output.

Conceptually:

```text
                       KMP Source
                           │
                 ┌─────────┴─────────┐
                 ▼                   ▼
              Android                iOS
                 │                   │
                 ▼                   ▼
          Android Toolchain     Kotlin/Native
                 │                   │
                 ▼                   ▼
          Android Artifact      Native Output
```

The shared Kotlin source participates in different target-specific compilations.

---

# 2. The Build Is a Graph

Do not think about the build as:

```text
Run Gradle
    ↓
Magic happens
```

Think about it as a graph:

```text
                    Gradle
                      │
                      ▼
                Project Model
                      │
          ┌───────────┼───────────┐
          ▼           ▼           ▼
      Android      Shared        iOS
       Target      Source       Target
          │           │           │
          └───────────┼───────────┘
                      ▼
                 Compilation
                      │
                      ▼
                  Artifacts
```

Every source set, dependency, target, plugin, and task participates in this graph.

---

# 3. The First Rule: Build Before Changing

After creating the project:

```text
Create Project
      │
      ▼
Build
      │
      ▼
Success
      │
      ▼
Only Then Modify
```

This gives you a **known-good baseline**.

If you add:

```text
Networking
Database
DI
Navigation
Serialization
```

before the first successful build, debugging becomes much harder.

You won't know whether the failure came from:

```text
Environment
+
Generated Project
+
Dependency
+
Configuration
+
Your Code
```

Start clean.

---

# 4. The Gradle Wrapper

A generated project normally includes:

```text
gradlew
gradlew.bat
gradle/
└── wrapper/
```

The wrapper lets the project use its configured Gradle version.

On macOS/Linux:

```bash
./gradlew build
```

On Windows:

```powershell
.\gradlew.bat build
```

You do not need a separately installed global Gradle version for normal wrapper-based builds.

---

# 5. Why the Wrapper Matters

Imagine three environments:

```text
Developer A
Gradle X

Developer B
Gradle Y

CI
Gradle Z
```

This can produce inconsistent behavior.

The wrapper gives the project a defined Gradle distribution:

```text
             Git Repository
                    │
                    ▼
           Gradle Wrapper
                    │
                    ▼
          Expected Gradle Version
                    │
          ┌─────────┴─────────┐
          ▼                   ▼
      Developer              CI
          │                   │
          └─────────┬─────────┘
                    ▼
             Consistent Build
```

This is one reason the wrapper should normally be committed to source control.

---

# 6. The First Useful Command

From the project root:

```bash
./gradlew tasks
```

On Windows PowerShell:

```powershell
.\gradlew.bat tasks
```

This asks Gradle to show the tasks available to the project.

You may see many tasks.

Don't try to memorize them.

Instead, understand that the task list is a window into the build graph.

---

# 7. Inspect the Build Before Running Everything

A useful first sequence is:

```text
1. tasks
2. projects
3. dependencies
4. build
```

For example:

```bash
./gradlew tasks
./gradlew projects
./gradlew dependencies
./gradlew build
```

On Windows:

```powershell
.\gradlew.bat tasks
.\gradlew.bat projects
.\gradlew.bat dependencies
.\gradlew.bat build
```

Some tasks may be module-specific or require additional configuration.

The goal is understanding, not blindly running every command.

---

# 8. What `projects` Tells You

Run:

```bash
./gradlew projects
```

Conceptually, the output should help you see something like:

```text
Root Project
│
├── androidApp
└── shared
```

The exact output depends on the project template.

This confirms what Gradle considers part of the build.

---

# 9. What `dependencies` Tells You

A dependency report helps you understand:

```text
Which libraries are connected?
Which module consumes them?
Which configurations contain them?
```

Conceptually:

```text
shared
 │
 ├── Kotlin Multiplatform
 ├── Serialization
 └── Network Library
```

while:

```text
androidApp
 │
 ├── AndroidX
 ├── Compose
 └── shared
```

The exact dependency graph depends on the project.

---

# 10. What `build` Actually Means

When you run:

```bash
./gradlew build
```

you are not running one simple operation.

Gradle determines which tasks are required and executes the necessary task graph.

Conceptually:

```text
                    build
                      │
             ┌────────┴────────┐
             ▼                 ▼
        Compilation         Tests
             │                 │
             ▼                 ▼
          Packaging        Verification
             │                 │
             └────────┬────────┘
                      ▼
                  Build Result
```

The actual graph can be much larger.

---

# 11. Gradle Is Task-Oriented

Gradle doesn't simply think:

```text
"Build the app."
```

It thinks in tasks and dependencies between tasks.

For example:

```text
Task A
  │
  ▼
Task B
  │
  ▼
Task C
```

If Task C requires Task B, Gradle knows that B must be completed first.

This is why running one high-level task can trigger many smaller operations.

---

# 12. KMP Adds Target-Specific Tasks

In a normal Android project, you may be familiar with tasks such as:

```text
assembleDebug
compileDebugKotlin
test
lint
```

KMP adds target-aware tasks for:

```text
Android
iOS
Other configured targets
```

The exact names vary by Kotlin and Gradle versions.

Do not build scripts around memorized task names from another project.

Instead, discover the tasks generated for the current project.

---

# 13. Source Set to Compilation

The build must connect source sets to targets.

For example:

```text
commonMain
     │
     ├──────────────► Android Compilation
     │
     └──────────────► iOS Compilation
```

While:

```text
androidMain
     │
     ▼
Android Compilation
```

and:

```text
iosMain
     │
     ▼
iOS Compilation
```

This is one of the most important KMP build concepts.

---

# 14. Common Code Is Compiled for Targets

Suppose:

```text
commonMain/
└── Greeting.kt
```

contains:

```kotlin
class Greeting {
    fun message(): String {
        return "Hello from KMP"
    }
}
```

The source is common.

But the build does not create one universal executable called:

```text
KMP.greeting
```

Instead, the shared code participates in the compilation of each supported target.

Conceptually:

```text
                  Greeting.kt
                       │
              ┌────────┴────────┐
              ▼                 ▼
         Android Target      iOS Target
              │                 │
              ▼                 ▼
        Android Output      Native Output
```

---

# 15. The Android Build Path

For Android, the path can be simplified to:

```text
Kotlin Source
     │
     ▼
KMP / Kotlin Compilation
     │
     ▼
Android Compilation
     │
     ▼
Android Build Tools
     │
     ▼
APK / AAB
```

The exact task graph contains many more steps.

But this is the mental model to keep.

---

# 16. The iOS Build Path

For iOS:

```text
Kotlin Shared Source
        │
        ▼
Kotlin/Native
        │
        ▼
Native Output
        │
        ▼
Xcode Integration
        │
        ▼
iOS Application
```

The iOS application itself is still built using Apple's toolchain.

KMP participates by providing Kotlin/Native output from the shared module.

---

# 17. Android and iOS Are Not Built Identically

This is important.

KMP does not turn Android and iOS into the same build system.

Instead:

```text
                  KMP
                   │
          ┌────────┴────────┐
          ▼                 ▼
       Android             iOS
          │                 │
     Android Build       Apple Build
       Toolchain           Toolchain
```

The shared Kotlin code connects the two worlds.

The native platforms still retain their own build systems and packaging requirements.

---

# 18. What Happens to `commonMain`?

Think of:

```text
commonMain
```

as a source definition.

The compiler/toolchain uses it as part of the compilation for supported targets.

Conceptually:

```text
commonMain
    │
    ├── Android
    │
    └── iOS
```

This is why the APIs used by `commonMain` must be compatible with the targets that consume them.

---

# 19. What Happens to `androidMain`?

`androidMain` is Android-specific.

Therefore:

```text
androidMain
     │
     ▼
Android Target
```

An iOS compilation does not simply take Android-specific code along with it.

That separation is one of the foundations of KMP.

---

# 20. What Happens to `iosMain`?

Similarly:

```text
iosMain
    │
    ▼
iOS Target
```

Its code participates in the appropriate Kotlin/Native compilation.

This lets the shared module contain both:

```text
Common Behavior
+
Platform Implementations
```

without mixing the two.

---

# 21. The Build Dependency Graph

A simplified build graph looks like:

```text
                    Root Project
                         │
                         ▼
                       Gradle
                         │
              ┌──────────┴──────────┐
              ▼                     ▼
         androidApp               shared
              │                     │
              │             ┌───────┼───────┐
              │             ▼       ▼       ▼
              │         commonMain android  ios
              │                     Main    Main
              │
              └──────────────┐
                             ▼
                        Android Output

shared
   │
   ▼
Kotlin/Native
   │
   ▼
iOS Output
```

This is simplified, but it gives you the right mental model.

---

# 22. Build Variants

Android developers are already familiar with build variants.

For example:

```text
debug
release
```

A production project may have:

```text
debug
release
staging
benchmark
```

KMP doesn't eliminate these concepts.

It adds multiplatform targets to the build model.

You can therefore have:

```text
Android
 ├── Debug
 └── Release

iOS
 ├── Debug
 └── Release
```

The exact configuration depends on your project and platform setup.

---

# 23. Debug vs Release

The difference between debug and release is not simply:

```text
Debug = works
Release = production
```

Build configuration can influence:

```text
Optimization
Signing
Debugging
Assertions
Logging
Packaging
Compiler behavior
```

A KMP application needs release validation for both platforms.

---

# 24. Building Shared Code

You can also focus on the shared module rather than the complete application.

Conceptually:

```text
shared
   │
   ├── commonMain
   ├── androidMain
   └── iosMain
```

The available Gradle tasks depend on the configured targets and Kotlin version.

Use:

```bash
./gradlew tasks
```

to discover the exact tasks generated by your project.

This is safer than copying commands from an older tutorial.

---

# 25. Why Task Names Change

KMP tooling evolves.

Gradle plugins evolve.

Kotlin evolves.

Target support evolves.

Therefore, a command that works in one project may not be the correct command in another.

For example:

```text
Task name from Tutorial A
```

may not exist in:

```text
Your Current Project
```

The reliable workflow is:

```text
Discover
   ↓
Inspect
   ↓
Run
   ↓
Understand
```

not:

```text
Copy
   ↓
Paste
   ↓
Hope
```

---

# 26. The Build Scan Mental Model

When a build runs, imagine the following:

```text
                 Gradle Invocation
                         │
                         ▼
                 Configuration Phase
                         │
                         ▼
                   Task Graph
                         │
                         ▼
                Execution Phase
                         │
             ┌───────────┼───────────┐
             ▼           ▼           ▼
          Compile      Test       Package
             │           │           │
             └───────────┼───────────┘
                         ▼
                       Result
```

Gradle's actual internals are more sophisticated, but this model is useful when diagnosing builds.

---

# 27. Configuration vs Execution

One useful distinction:

### Configuration

Gradle determines:

```text
What projects exist?
What plugins are applied?
What targets exist?
What tasks exist?
What dependencies exist?
```

### Execution

Gradle executes:

```text
Compilation
Tests
Packaging
Verification
```

Conceptually:

```text
Configuration
      │
      ▼
Task Graph
      │
      ▼
Execution
```

This distinction becomes very useful when Gradle builds become slow.

---

# 28. Why a KMP Build Can Feel Slow

A multiplatform build can involve:

```text
Gradle
+
Kotlin compiler
+
Android toolchain
+
Kotlin/Native
+
Xcode / Apple toolchain
+
Multiple dependencies
```

So a build may do considerably more work than an Android-only build.

But don't assume every build needs to compile everything.

Gradle's incremental and task-avoidance mechanisms can skip work when inputs have not changed.

---

# 29. Incremental Builds

A simplified model:

```text
First Build
   │
   ▼
Compile Everything Required
```

Then:

```text
Small Source Change
   │
   ▼
Determine Affected Tasks
   │
   ▼
Compile Only Necessary Work
```

This is why incremental builds can be much faster than clean builds.

---

# 30. Avoid `clean` as a Habit

A common Android developer habit is:

```bash
./gradlew clean
./gradlew build
```

after every problem.

Don't do this automatically.

A clean build removes generated build outputs and forces work to happen again.

Use clean builds when there is a real reason to invalidate generated state.

Otherwise:

```text
Incremental Build
```

is usually more informative and faster.

---

# 31. When a Clean Build Is Useful

A clean build can help when:

```text
Stale generated output
Unexpected cached state
Changed build configuration
Tooling migration
Corrupted intermediate output
```

But first ask:

> **Do I actually need to delete the build state?**

If the answer is no, don't make every build a clean build.

---

# 32. Build Cache

Gradle and Kotlin tooling can reuse work under appropriate conditions.

Conceptually:

```text
Previous Build
      │
      ▼
Cached / Reusable Work
      │
      ▼
Next Build
      │
      ▼
Less Work
```

This can make repeated builds much faster.

The exact caching behavior depends on the task, inputs, Gradle configuration, and environment.

---

# 33. Dependencies Affect Build Time

Suppose your project has:

```text
5 dependencies
```

Then:

```text
50 dependencies
```

Then:

```text
150 dependencies
```

The build graph becomes more complex.

More dependencies can mean:

```text
More resolution
More compilation
More metadata
More native artifacts
More processing
More opportunities for incompatibility
```

This is one reason to avoid adding libraries without a clear need.

---

# 34. Native Dependencies Need Extra Attention

A library used in `commonMain` must support the relevant targets.

For Android:

```text
Android artifact
```

For iOS:

```text
Native-compatible artifact
```

If a dependency does not support a target, the shared source cannot simply use it for that target.

This is one of the first places where KMP builds differ sharply from Android-only projects.

---

# 35. Build Failure: Read the First Meaningful Error

When a Gradle build fails, the output may contain:

```text
warnings
stack traces
task names
dependency messages
multiple errors
```

Don't immediately scroll to the last line and assume it is the root cause.

Look for:

```text
First meaningful failure
```

For example:

```text
Task A
Task B
Task C FAILED
```

Then inspect the error associated with Task C.

---

# 36. Build Failure Triage

A useful sequence is:

```text
Build Failed
    │
    ▼
Which Task Failed?
    │
    ▼
Which Target?
    │
    ▼
Which Source Set?
    │
    ▼
Which Dependency?
    │
    ▼
Which Configuration?
```

This maps naturally to the KMP architecture.

---

# 37. Example: Android Works, iOS Fails

Suppose:

```text
Android → PASS
iOS     → FAIL
```

Do not conclude:

```text
"KMP is broken."
```

Ask:

```text
Is the dependency available on iOS?
Is the code using an Android API?
Is the iOS source set configured?
Is the Apple toolchain available?
Is the target architecture supported?
Is the Xcode integration correct?
```

This turns the failure into a structured investigation.

---

# 38. Example: iOS Works, Android Fails

The same reasoning applies:

```text
iOS     → PASS
Android → FAIL
```

Check:

```text
Android SDK
Android plugin
Android target
Android-specific dependency
Gradle configuration
Manifest/application configuration
```

The build result itself tells you which platform boundary needs investigation.

---

# 39. Build Logs Are Architectural Feedback

A build failure can reveal an architectural problem.

For example:

```text
commonMain
    │
    ▼
Android-only library
    │
    ▼
iOS Build Failure
```

The compiler is effectively telling you:

> "This shared source depends on something that isn't shared."

That is not merely a syntax problem.

It is a dependency-boundary problem.

---

# 40. Build and Architecture

The build system enforces some architectural boundaries.

For example:

```text
commonMain
     │
     ▼
Multiplatform Dependency
```

is valid.

But:

```text
commonMain
     │
     ▼
Android-only Dependency
```

creates a portability problem.

Therefore:

```text
Architecture
      ↕
Build Configuration
```

are tightly connected in KMP.

---

# 41. The Build Pipeline

The complete mental model can now be visualized:

```text
                         Source Code
                             │
                             ▼
                     Gradle Configuration
                             │
                             ▼
                       Target Selection
                             │
               ┌─────────────┴─────────────┐
               ▼                           ▼
           Android                       iOS
               │                           │
               ▼                           ▼
        Kotlin / Android             Kotlin/Native
          Compilation                Compilation
               │                           │
               ▼                           ▼
        Android Artifact             Native Output
               │                           │
               ▼                           ▼
           APK / AAB                 Xcode / App
```

The shared code sits in the middle.

---

# 42. The Role of Xcode

For iOS development, Xcode remains important.

The KMP build does not replace:

```text
Xcode
Apple SDKs
Apple signing
Apple application packaging
```

Instead:

```text
KMP
 │
 ▼
Shared Kotlin Native Output
 │
 ▼
Xcode
 │
 ▼
iOS Application
```

This is a crucial expectation to set early.

---

# 43. Building on macOS vs Other Systems

For an Android-only developer, this may be surprising:

```text
Android
→ broad development environment options

iOS
→ Apple platform requirements
```

If your project includes an iOS application, the complete iOS build and signing workflow requires the appropriate Apple environment.

This is not a KMP limitation created by the project.

It is part of Apple's platform development model.

---

# 44. Local Build vs CI Build

Your local build is only one environment.

A production team also needs:

```text
CI
```

The build should be reproducible there.

Conceptually:

```text
Developer Machine
        │
        ▼
     Build
        │
        ▼
       Git
        │
        ▼
       CI
        │
        ▼
     Build Again
```

If the project only builds on one developer's machine, it is not yet a healthy build system.

---

# 45. CI and KMP

A multiplatform CI pipeline may eventually contain separate platform workflows:

```text
                    CI
                     │
          ┌──────────┴──────────┐
          ▼                     ▼
      Android Build          iOS Build
          │                     │
          ▼                     ▼
       Tests                  Tests
          │                     │
          ▼                     ▼
       Artifact              Artifact
```

The exact implementation depends on the CI platform and release strategy.

---

# 46. Build Reproducibility

A good KMP build should minimize reliance on:

```text
One Developer's Machine
One IDE Installation
One Global Gradle Version
One Local Configuration
```

Instead, use:

```text
Version-controlled configuration
+
Gradle Wrapper
+
Documented environment
+
Reproducible CI
```

This becomes increasingly important as the project grows.

---

# 47. Build Output

After a successful build, generated outputs may appear under directories such as:

```text
build/
```

or platform-specific output locations.

Do not confuse:

```text
Source Code
```

with:

```text
Build Artifacts
```

The source is what you maintain.

The artifacts are what the build produces.

---

# 48. Don't Commit Build Outputs

Normally, generated directories such as:

```text
build/
```

should not be committed to Git.

The repository should contain the inputs required to reproduce the output.

Think:

```text
Git
 │
 ├── Source
 ├── Build Configuration
 └── Wrapper
        │
        ▼
      Build
        │
        ▼
    Generated Output
```

---

# 49. Build Once, Understand Twice

When the first build succeeds, don't immediately move on.

Ask:

```text
Which modules were built?
Which targets were compiled?
Which source sets participated?
Which dependencies were resolved?
Which artifacts were generated?
```

You don't need to answer every question immediately.

The important thing is to develop the habit of looking behind the command.

---

# 50. A Useful First-Build Exercise

After creating the project:

### Step 1

Run:

```bash
./gradlew projects
```

### Step 2

Inspect:

```text
settings.gradle.kts
```

### Step 3

Inspect:

```text
shared/build.gradle.kts
```

### Step 4

Run:

```bash
./gradlew tasks
```

### Step 5

Build the Android target using the appropriate generated task or IDE configuration.

### Step 6

Open the iOS project in Xcode and validate the iOS target when working on a compatible Apple development environment.

### Step 7

Make one small change in `commonMain`.

### Step 8

Build again.

The objective is to observe what changed in the build.

---

# 51. Make One Small Change

For example:

```kotlin
class Greeting {

    fun message(): String {
        return "Hello from KMP!"
    }
}
```

Change it to:

```kotlin
class Greeting {

    fun message(): String {
        return "Hello from Kotlin Multiplatform!"
    }
}
```

Then build again.

You have now created an experiment:

```text
Source Change
     │
     ▼
Affected Compilation
     │
     ▼
Updated Output
```

This is how you start understanding incremental builds.

---

# 52. Don't Start With `clean`

For this experiment, do not immediately run:

```bash
./gradlew clean
```

You want to see what the build system can reuse.

A normal incremental build is more educational here.

---

# 53. Build Performance Is an Engineering Concern

As a KMP project grows, build performance becomes important.

Potential contributors include:

```text
Number of modules
Number of dependencies
Kotlin compilation
Kotlin/Native compilation
Generated sources
Annotation processing / code generation
Gradle configuration
CI environment
```

A project that takes:

```text
10 seconds
```

to build is very different from one that takes:

```text
15 minutes
```

to build.

Build architecture matters.

---

# 54. Avoid Premature Optimization

Don't optimize a project that contains:

```text
One screen
Two classes
One shared module
```

First understand the normal build.

Then measure.

Then optimize.

Useful questions later include:

```text
What task is slow?
Why is it slow?
Is the task doing necessary work?
Can the dependency graph be reduced?
Can modules be separated?
Can caching help?
```

---

# 55. Build and Dependency Boundaries

A well-structured KMP project makes the build graph easier to reason about.

For example:

```text
feature-products
      │
      ▼
domain
      │
      ▼
core-network
```

is easier to understand than:

```text
Everything
   ↕
Everything
```

The build system becomes a reflection of the architecture.

---

# 56. A Build Failure Is Not Always a Code Failure

When you see:

```text
BUILD FAILED
```

remember that the failure can originate from:

```text
Code
Configuration
Dependency
Toolchain
Environment
Target
Packaging
Signing
```

Therefore:

```text
BUILD FAILED
```

is a symptom.

The failed task and error message tell you what to investigate.

---

# 57. The Build Contract

A useful way to think about the build is as a contract:

```text
Source Code
      +
Build Configuration
      +
Dependencies
      +
Toolchains
      │
      ▼
Expected Artifact
```

If one part of the contract is inconsistent, the build can fail.

This is especially important in multiplatform projects because there are multiple target environments.

---

# 58. The First Successful Build

A successful first build proves several things at once:

```text
✓ Gradle works
✓ Kotlin tooling works
✓ Project structure is valid
✓ Targets are configured
✓ Dependencies resolve
✓ Compiler can process the source
✓ Platform toolchain is available
✓ Output can be produced
```

That's why the first build is more important than it looks.

---

# 59. The Build Mental Model

At this point, you should be able to visualize:

```text
                      KMP PROJECT
                           │
                           ▼
                         Gradle
                           │
                     Project Model
                           │
              ┌────────────┴────────────┐
              ▼                         ▼
        Shared Source               App Modules
              │                         │
      ┌───────┼───────┐          ┌──────┴──────┐
      ▼       ▼       ▼          ▼             ▼
 commonMain android  ios      Android         iOS
              Main    Main     App            App
      │       │       │          │             │
      └───────┴───────┘          │             │
              │                  │             │
              ▼                  ▼             ▼
        Target Compilation    Android       Xcode/
              │               Build         Apple Build
              ▼                  │             │
         Native Outputs          └──────┬──────┘
                                        ▼
                                  Applications
```

The build is the mechanism that turns this graph into real platform artifacts.

---

# 60. The Most Important Lesson

A KMP build is not:

```text
Android Build + iOS Build
```

It is better understood as:

```text
Shared Source
     │
     ├── Target-specific compilation
     │
     ├── Platform-specific dependencies
     │
     └── Native platform packaging
```

KMP provides the shared compilation model.

Android and iOS still retain their platform-specific build responsibilities.

---

# Chapter Takeaways

> [!TIP]
> **Don't treat Gradle as a command-line button you press when the project needs to build. In KMP, Gradle is part of the architecture.**

Remember:

1. The KMP build is a task graph, not a single operation.
2. The Gradle wrapper provides a project-controlled Gradle version.
3. `./gradlew tasks` is useful for discovering the tasks available in your current project.
4. `./gradlew projects` helps you understand the Gradle project structure.
5. Build the generated project before making significant changes.
6. `commonMain` participates in the compilation of supported targets.
7. `androidMain` is compiled for Android.
8. `iosMain` is compiled for iOS targets.
9. Android and iOS still use their native platform toolchains.
10. KMP does not replace Xcode or Apple's native application build process.
11. A dependency available on Android is not automatically available on iOS.
12. Build failures can indicate architecture or dependency-boundary problems.
13. Don't use `clean` automatically after every failure.
14. Incremental builds are important for developer productivity.
15. Build performance becomes an engineering concern as projects grow.
16. The generated task names can change with tooling versions; discover them rather than blindly copying commands.
17. Build outputs should normally be generated, not committed.
18. CI should be able to reproduce the project build.
19. A successful first build establishes a known-good baseline.
20. Understanding the build graph makes KMP debugging much more systematic.

---

# Final Mental Model

When you run:

```bash
./gradlew build
```

don't imagine:

```text
Command
   │
   ▼
Magic
   │
   ▼
APK
```

Imagine:

```text
                         ./gradlew build
                                  │
                                  ▼
                           Gradle Project
                                  │
                                  ▼
                             Task Graph
                                  │
              ┌───────────────────┼───────────────────┐
              ▼                   ▼                   ▼
          Source Sets         Dependencies          Targets
              │                   │                   │
              └───────────────────┼───────────────────┘
                                  ▼
                           Kotlin Compilation
                                  │
                    ┌─────────────┴─────────────┐
                    ▼                           ▼
                Android                     Kotlin/Native
                    │                           │
                    ▼                           ▼
              Android Output                iOS Output
                    │                           │
                    ▼                           ▼
                 APK/AAB                  Xcode / iOS App
```

The key idea is simple:

> **The build is the bridge between your KMP source structure and the native applications that eventually run on Android and iOS. Once you understand that bridge, Gradle output stops looking like noise and starts telling you exactly what the project is doing.**
