# Chapter 9 — Kotlin Compiler Deep Dive

## Part 2 — Metadata

> **In Kotlin Multiplatform, compiled code is not only about executable output. Metadata carries information that helps other Kotlin compilations understand declarations, types, APIs, relationships, and platform-specific capabilities.**

Metadata is one of the less visible parts of the KMP compilation model, but it explains many concepts that otherwise seem mysterious:

- How common Kotlin code can be consumed by different targets
- How declarations remain understandable across compilations
- How dependencies expose Kotlin APIs
- Why metadata and platform binaries are different things
- How source-set dependencies influence what the compiler can see
- Why a library can be compiled without being a final application artifact

The goal of this part is to build a practical mental model rather than memorize compiler internals.

---

## 1. What Is Kotlin Metadata?

At a high level, metadata is information about compiled Kotlin declarations.

It can describe things such as:

```text
Classes
Functions
Properties
Types
Visibility
Generic information
Annotations
Declaration relationships
Kotlin-specific language information
```

A useful simplified model is:

```text
Kotlin source
     ↓
Compiler analysis
     ↓
Compiled representation
     +
Kotlin metadata
```

The metadata helps Kotlin tooling and later compilations understand what was declared without requiring the original source files.

---

## 2. Why Metadata Matters in KMP

KMP is built around sharing Kotlin declarations across multiple target compilations.

Consider:

```kotlin
class User(
    val id: String,
    val name: String
)
```

Another Kotlin compilation needs to understand that `User`:

```text
is a class
has an id property
has a name property
uses String
has specific visibility
```

Metadata provides Kotlin-specific information that is not always represented directly by the target platform's native format.

This makes metadata an important part of Kotlin's cross-compilation model.

---

## 3. Metadata Is Not the Same as Bytecode

For JVM compilation, it is tempting to think:

```text
Kotlin → JVM bytecode
```

But Kotlin also records Kotlin-specific information.

Conceptually:

```text
                 Kotlin source
                      │
                      ▼
                  Compiler
                      │
             ┌────────┴────────┐
             │                 │
             ▼                 ▼
       JVM bytecode       Kotlin metadata
```

The bytecode allows the JVM environment to execute the program.

The metadata helps Kotlin tools and compilers understand Kotlin declarations.

These are related, but they are not the same thing.

---

## 4. Metadata Preserves Kotlin's View of the Program

Consider:

```kotlin
data class User(
    val id: String,
    val name: String
)
```

The generated JVM representation contains platform-level details.

Kotlin metadata can preserve Kotlin-level concepts such as:

```text
Class declaration
Primary constructor
Properties
Property types
Data-class semantics
Kotlin signatures
```

This distinction becomes especially important when Kotlin code is consumed by another Kotlin compilation.

---

## 5. A Simple Mental Model

Think about metadata as a Kotlin-readable description of compiled declarations.

```text
┌───────────────────────┐
│     Kotlin source     │
└───────────┬───────────┘
            │
            ▼
┌───────────────────────┐
│       Compiler        │
└───────────┬───────────┘
            │
       ┌────┴─────┐
       │          │
       ▼          ▼
 Platform      Kotlin
 output       metadata
       │          │
       ▼          ▼
Runtime      Kotlin compiler
```

The exact storage and representation differ by target, but the conceptual separation is useful.

---

## 6. Metadata and Multiplatform Libraries

Imagine a KMP library:

```text
shared-library
│
├── common declarations
├── Android implementation
└── iOS implementation
```

An application consuming the library needs to know what APIs are available.

For example:

```kotlin
import com.example.UserRepository

val repository = UserRepository()
```

The consuming compilation needs information about:

```text
UserRepository
constructors
functions
properties
types
visibility
dependencies
```

Compiled library information allows the compiler to resolve those declarations.

---

## 7. Common Metadata

KMP has a concept of common metadata representing declarations from common source code.

A simplified view:

```text
commonMain
     │
     ▼
Common compilation
     │
     ▼
Common metadata
     │
     ├───────────────┐
     ▼               ▼
Android target     iOS target
compilation        compilation
```

The target compilation can use the common declarations while combining them with target-specific source and dependencies.

The exact internal pipeline varies by Kotlin version, but this model is useful for understanding why common code can be reused without turning it into one universal binary.

---

## 8. Metadata Is About Declarations, Not Just Execution

This distinction is important.

Suppose common code declares:

```kotlin
expect fun platformName(): String
```

The common compilation needs to preserve information about:

```text
platformName
return type = String
expect declaration
visibility
signature
```

The platform compilation then provides the corresponding implementation.

Conceptually:

```text
Common metadata
      │
      ▼
expect platformName()
      │
      ▼
Target compilation
      │
      ▼
actual platformName()
```

Metadata therefore participates in the compiler's understanding of the program.

---

## 9. Metadata and `expect` / `actual`

Consider:

```kotlin
// commonMain

expect class Platform {
    val name: String
}
```

And:

```kotlin
// androidMain

actual class Platform {
    actual val name: String = "Android"
}
```

The common side communicates the expected declaration.

The target compilation must resolve the corresponding actual declaration.

A simplified model is:

```text
common declaration
       │
       │ expected API
       ▼
common metadata
       │
       ▼
target compiler
       │
       ▼
actual declaration
```

The metadata is part of the information the compiler uses to understand the common API.

---

## 10. Metadata and Dependency Resolution

Suppose:

```kotlin
commonMain.dependencies {
    implementation("com.example:shared-library:1.0.0")
}
```

When the compiler processes code that uses the library, it needs information about the library's declarations.

For example:

```kotlin
import com.example.Logger

val logger = Logger()
```

The compiler must resolve:

```text
Logger
constructor
methods
properties
types
visibility
```

Metadata and compiled library information make that possible.

---

## 11. Metadata Does Not Replace the Dependency

Metadata is not a magic substitute for a library.

You still need the appropriate dependency available to the compilation.

Think of it as:

```text
Dependency
   │
   ├── compiled implementation
   └── Kotlin declaration information
            ↓
       Compiler input
```

The compiler needs the relevant dependency in its compilation environment.

---

## 12. Common vs Platform Information

A useful distinction is:

```text
Common metadata
    ↓
What common Kotlin declares

Platform-specific compiled information
    ↓
What the target provides
```

For example:

```text
commonMain
    UserRepository
    User
    expect Platform

androidMain
    actual Platform
    Android implementation

iosMain
    actual Platform
    iOS implementation
```

The target compiler combines the relevant information to build the target.

---

## 13. Metadata Is Not a Runtime Database

A common misunderstanding is to imagine metadata as something the application continuously queries at runtime.

That is not the right model.

Metadata is primarily compiler/tooling-oriented information.

Think:

```text
Build time
    ↓
Compiler understands declarations
```

rather than:

```text
Application runtime
    ↓
Application queries metadata
```

Some Kotlin metadata can be inspected programmatically, but that is a separate use case.

---

## 14. Metadata and Reflection Are Different

Metadata should not be confused with runtime reflection.

### Metadata

Primarily supports:

```text
Compilation
Tooling
Kotlin declaration understanding
API analysis
```

### Reflection

Supports runtime inspection such as:

```text
Discovering classes
Inspecting properties
Calling functions dynamically
```

They may interact in the broader Kotlin ecosystem, but they solve different problems.

---

## 15. Why JVM Developers Notice Metadata

On the JVM, Kotlin metadata is commonly associated with compiled `.class` files.

A Kotlin class may therefore contain:

```text
JVM bytecode
+
Kotlin-specific metadata
```

This allows Kotlin-aware tools and compilers to recover Kotlin-level information from JVM artifacts.

For example, Kotlin tooling can distinguish concepts that are represented differently at the JVM level.

---

## 16. Kotlin Metadata and Java Interoperability

Java and Kotlin do not model every language concept in exactly the same way.

For example:

```kotlin
val name: String?
```

contains Kotlin nullability information.

The JVM bytecode does not represent Kotlin nullability in exactly the same way Kotlin source does.

Kotlin metadata helps preserve Kotlin-specific information that Kotlin tooling can understand.

This is one reason Kotlin can provide a richer source-level experience than the underlying JVM representation alone would suggest.

---

## 17. Metadata and Nullability

Consider:

```kotlin
fun findUser(id: String): User?
```

Kotlin understands:

```text
id is non-null
return value may be null
```

Kotlin's type system and compiled representation work together to preserve and enforce these semantics where applicable.

Metadata contributes Kotlin-specific declaration information that is important for Kotlin consumers.

---

## 18. Metadata and Generic Types

Consider:

```kotlin
class Repository<T> {
    fun save(value: T)
}
```

The compiler needs information about:

```text
Repository<T>
T
save(value: T)
```

Generic information is part of the declaration model.

This becomes especially important when another Kotlin compilation consumes the compiled library.

---

## 19. Metadata and API Surface

Metadata can be thought of as part of the information needed to understand a library's Kotlin API.

For a library:

```text
shared-core
```

the compiler needs to know which declarations are exposed.

A simplified representation:

```text
Library
  │
  ├── Implementation
  │
  └── Kotlin API information
         │
         ▼
      Consumer
```

This is one reason changing a public declaration can affect downstream compilation.

---

## 20. Metadata and Binary Compatibility

Library authors need to think about compatibility between published versions.

Consider:

```kotlin
class UserRepository {
    fun getUser(id: String): User
}
```

A later release changes the public API:

```kotlin
class UserRepository {
    fun getUser(id: Long): User
}
```

Consumers compiling against the new version see a different API.

Metadata and other compiled information are part of how the consumer compiler understands that API.

This is why public API evolution should be deliberate.

---

## 21. Metadata and ABI

ABI stands for **Application Binary Interface**.

A simplified distinction is:

```text
Source API
    ↓
What developers write against

ABI
    ↓
What compiled consumers link/use

Metadata
    ↓
Kotlin-specific information used to understand declarations
```

These concepts overlap in practical tooling but are not identical.

Metadata should not be described as the complete ABI.

---

## 22. Metadata in a KMP Library

A published KMP library may contain platform-specific artifacts and information needed by consumers.

Conceptually:

```text
KMP library
│
├── Common information
│
├── JVM/Android artifact
│
├── Native artifact(s)
│
└── Platform-specific metadata/information
```

The exact publication layout depends on the Kotlin and Gradle versions and publishing configuration.

The important point is:

> A multiplatform library is more than a single binary file.

---

## 23. Metadata and Source Distribution

A consumer usually does not need the original source code simply to compile against a published binary.

For example:

```text
Library source
      ↓
Library compilation
      ↓
Published artifact
      ↓
Consumer compilation
```

The consumer compiler can use compiled declaration information.

Source code may still be published separately for:

```text
Debugging
IDE navigation
Documentation
Developer experience
```

But it is not the same thing as compiler metadata.

---

## 24. Metadata and IDE Support

Kotlin-aware IDE tooling needs information about declarations to provide features such as:

```text
Code completion
Navigation
Type information
Documentation lookup
Override suggestions
Refactoring
Usage search
```

Metadata and compiler models contribute to this broader tooling experience.

This is another reason metadata is not merely an implementation detail.

---

## 25. Metadata and Source Sets

Source sets determine which declarations participate in a compilation.

For example:

```text
commonMain
    │
    ▼
common declarations
    │
    ├── Android compilation
    │
    └── iOS compilation
```

Metadata represents information about declarations produced by these compilations.

So the relationship can be viewed as:

```text
Source sets
    ↓
Compilation inputs
    ↓
Compiler analysis
    ↓
Metadata + target output
```

---

## 26. Metadata and the Source-Set Hierarchy

Consider:

```text
commonMain
    │
    ├── appleMain
    │      └── iosMain
    │
    └── androidMain
```

An iOS compilation can use declarations from:

```text
commonMain
appleMain
iosMain
```

The compiler needs information about the declarations visible through this hierarchy.

This is one reason source-set hierarchy and dependency resolution cannot be treated as unrelated topics.

---

## 27. Metadata and Compilation Boundaries

A useful mental model is:

```text
Source
  ↓
Compilation boundary
  ↓
Compiler understands declarations
  ↓
Metadata / compiled information
  ↓
Another compilation can consume the API
```

Each compilation has its own context.

This is especially important in large KMP projects where many modules depend on one another.

---

## 28. Multi-Module KMP Projects

Imagine:

```text
:core
:network
:data
:feature
:app
```

A simplified dependency graph:

```text
app
 │
 ▼
feature
 │
 ▼
data
 │
 ├── network
 └── core
```

Each module can have its own compilations and published outputs.

Metadata and compiled declarations allow downstream modules to understand upstream APIs.

---

## 29. Metadata Is Important for Incremental Builds

Build systems try to avoid recompiling everything unnecessarily.

If a dependency's public declarations have not meaningfully changed, downstream work can often be reduced.

Conceptually:

```text
Module A
   ↓
Compiled information
   ↓
Module B
```

If Module A changes internally without changing relevant API information, the build system may be able to limit the amount of downstream recompilation.

The exact behavior depends on compiler and build-system implementation.

---

## 30. Metadata and API Changes

Compare:

### Internal change

```kotlin
private fun formatUser(): String
```

Implementation changes may have limited impact outside the module.

### Public change

```kotlin
fun formatUser(): String
```

Changing the public signature can affect consumers.

This difference matters because downstream compilation depends on the exposed API.

---

## 31. Metadata and Serialization

Some Kotlin libraries use compiler plugins and generated metadata or code to support serialization and other language-level features.

For example:

```kotlin
@Serializable
data class User(
    val id: String
)
```

A serialization solution may need compiler-assisted information about the class.

The important lesson is:

> Compiler metadata, compiler plugins, generated code, and runtime libraries can work together, but they are separate concepts.

Do not treat them as interchangeable.

---

## 32. Metadata and Compiler Plugins

Compiler plugins may inspect or transform Kotlin declarations during compilation.

A simplified model:

```text
Source
  ↓
Compiler frontend
  ↓
Plugin integration
  ↓
IR / compiler processing
  ↓
Target output
```

Metadata can provide Kotlin declaration information that is useful to compiler and tooling ecosystems.

However, not every compiler plugin simply "reads metadata"; implementation details vary.

---

## 33. Metadata Versioning

Kotlin metadata is tied to Kotlin compiler evolution.

This means library compatibility can depend on:

```text
Kotlin version
Compiler version
Metadata representation
Target artifact
Plugin version
```

A library compiled with one toolchain may not always be consumable by every other toolchain configuration.

This is why Kotlin version alignment matters in KMP projects.

---

## 34. Why Version Alignment Matters

Consider:

```text
Application
    ↓
Kotlin compiler version A
    ↓
KMP library compiled with version B
```

If the versions or published artifacts are incompatible, compilation can fail before the application runs.

Possible symptoms include errors related to:

```text
Incompatible metadata
Unsupported binary format
Compiler version mismatch
Plugin incompatibility
Unresolved declarations
```

The exact error depends on the situation.

---

## 35. Metadata Errors Are Usually Build-Time Problems

If you see an error resembling:

```text
Module was compiled with an incompatible version of Kotlin
```

the issue is usually related to the build toolchain or compiled dependency compatibility.

A practical debugging sequence is:

```text
Check Kotlin version
        ↓
Check KMP plugin version
        ↓
Check dependency versions
        ↓
Check compiler plugins
        ↓
Clean/rebuild if appropriate
```

Do not immediately assume that the application runtime is broken.

---

## 36. Metadata Is Not "Extra Source Code"

Another misconception is:

> "Metadata is basically the original Kotlin source stored inside the binary."

That is not accurate.

Metadata is structured compiler information.

Conceptually:

```text
Source code
   ↓
Compiler analysis
   ↓
Structured declaration information
```

It is not intended to be a complete copy of the original source.

---

## 37. Metadata Is Not a Universal Format Across All Platforms

Another important nuance:

```text
JVM
Native
JS
Wasm
```

do not all have identical artifact structures.

The Kotlin compiler and build tools provide platform-specific mechanisms for carrying and consuming Kotlin declaration information.

Therefore, avoid thinking of metadata as one identical file that is simply copied to every platform.

---

## 38. Common Metadata vs Platform Artifact

Consider:

```text
Shared business logic
       │
       ▼
Common compilation
       │
       ├── common declaration information
       │
       └── target-specific compilation
                    │
                    ├── Android artifact
                    └── iOS artifact
```

The common declaration model is not itself the final Android APK or iOS framework.

It is part of the information needed to construct and consume platform-specific outputs.

---

## 39. Metadata and the Compiler's View

Think like the compiler.

When it sees:

```kotlin
val user = repository.getUser("123")
```

it needs to know:

```text
repository type
getUser function
parameter type
return type
visibility
generic information
available declarations
```

Metadata and compiled dependency information help provide this information when the declaration comes from another compiled module.

---

## 40. A Practical Example

Suppose a shared module contains:

```kotlin
package com.example.user

class UserRepository {
    fun getUser(id: String): User {
        // implementation
    }
}
```

A feature module uses:

```kotlin
import com.example.user.UserRepository

class UserService {
    private val repository = UserRepository()

    fun load(id: String): User {
        return repository.getUser(id)
    }
}
```

The feature compilation needs to understand:

```text
UserRepository exists
UserRepository has a constructor
getUser exists
getUser accepts String
getUser returns User
```

It does not need the original source text merely to type-check this usage.

---

## 41. Metadata in the KMP Mental Model

Connect the previous parts of this chapter:

```text
Part 1 — Compilation
        ↓
Compiler processes source
        ↓
Part 2 — Metadata
        ↓
Compiler-readable declaration information
        ↓
Part 3 — Target/backend processing
        ↓
Platform-specific output
```

This creates a broader model:

```text
                    Kotlin source
                         │
                         ▼
                  Source-set model
                         │
                         ▼
                    Compilation
                         │
              ┌──────────┴──────────┐
              │                     │
              ▼                     ▼
       Kotlin metadata        Target output
              │                     │
              ▼                     ▼
     Kotlin consumers          Platform runtime
```

---

## 42. Debugging Metadata Problems

When a KMP build reports a metadata or binary compatibility error, use a systematic approach.

### Step 1 — Identify the failing module

```text
Which module is being compiled?
```

### Step 2 — Identify the dependency

```text
Which library/module is being consumed?
```

### Step 3 — Compare Kotlin versions

```text
Application Kotlin
vs
Dependency Kotlin
```

### Step 4 — Check compiler plugins

Especially if the project uses:

```text
Compose
Serialization
Other compiler-integrated tooling
```

### Step 5 — Check target compatibility

A dependency may support one target but not another.

### Step 6 — Inspect the build graph

Determine which dependency is introducing the incompatible artifact.

---

## 43. Common Metadata Mistakes

### Mistake 1 — Treating Metadata as Runtime Code

**Problem:** metadata is primarily compiler/tooling information.

**Better approach:** think of metadata as part of the compilation ecosystem.

---

### Mistake 2 — Assuming Bytecode Is the Whole Kotlin API

**Problem:** Kotlin-specific information is not fully captured by the platform representation alone.

**Better approach:** distinguish target output from Kotlin declaration information.

---

### Mistake 3 — Ignoring Kotlin Version Compatibility

**Problem:** compiled libraries and compiler tooling need compatible versions.

**Better approach:** keep the Kotlin toolchain and compiler plugins aligned.

---

### Mistake 4 — Assuming One Metadata Artifact Fits Every Target

**Problem:** different targets have different compilation and artifact models.

**Better approach:** reason in terms of target-specific compilations and published variants.

---

### Mistake 5 — Treating Every Build Error as a Runtime Problem

**Problem:** metadata incompatibility normally appears during compilation.

**Better approach:** inspect compiler, dependency, and plugin versions first.

---

## 44. Metadata and Architecture

Metadata is not just compiler trivia.

It reinforces an important KMP architectural principle:

```text
Common API
    ↓
Explicit contract
    ↓
Platform implementations
    ↓
Target compilation
```

The compiler needs enough information to understand each layer and enforce the relationships between them.

That makes the compilation model more predictable.

---

## 45. The Most Important Mental Model

Do not think:

```text
KMP
 ↓
Common source
 ↓
One binary
```

Think:

```text
KMP project
     ↓
Source-set graph
     ↓
Target-specific compilations
     ↓
Kotlin declaration information
     +
Target-specific output
     ↓
Platform artifacts
```

Metadata belongs to the declaration-information side of this model.

---

## 46. Metadata Summary

Remember metadata as:

```text
Kotlin source
      ↓
Compiler
      ↓
Kotlin declaration information
      +
Target-specific compiled output
```

It helps Kotlin tooling and subsequent compilations understand declarations such as:

```text
Classes
Functions
Properties
Types
Generics
Visibility
Annotations
Relationships
```

In KMP, this becomes especially important because common declarations are consumed by multiple target compilations.

---

## 47. Key Takeaways

> **1. Metadata is compiler-oriented information about Kotlin declarations.**

It helps Kotlin understand compiled APIs.

> **2. Metadata is different from platform output.**

JVM bytecode, native artifacts, JavaScript, and WebAssembly output are not the same thing as Kotlin metadata.

> **3. Metadata supports cross-module compilation.**

A consumer can compile against a published Kotlin library without requiring its original source code.

> **4. Metadata participates in the multiplatform compilation model.**

Common declarations must remain understandable to target compilations.

> **5. `expect` / `actual` depends on compiler understanding of declarations.**

Metadata is part of the broader information model used during compilation.

> **6. Version compatibility matters.**

Kotlin, KMP, compiler plugins, and compiled dependencies must be compatible.

> **7. Metadata is not runtime business logic.**

It should primarily be understood as build-time/compiler/tooling information.

> **8. A KMP library is more than one binary.**

Published multiplatform libraries can contain multiple target-specific artifacts and associated declaration information.

---

## Final Perspective

Metadata is one of the invisible mechanisms that makes Kotlin's compiled ecosystem usable.

When you write:

```kotlin
val user = repository.getUser("123")
```

the compiler needs to understand much more than the text of that line.

It needs to know:

```text
What is repository?
What is getUser?
What parameter does it accept?
What does it return?
Is it visible?
Which module provides it?
Which target is being compiled?
```

That information comes from the compilation environment, source-set model, dependencies, and compiled Kotlin declaration information.

For KMP, this becomes even more important because the same common API can participate in multiple target compilations.

The practical mental model is:

```text
                 COMMON KOTLIN
                       │
                       ▼
                 Compilation
                       │
             ┌─────────┴─────────┐
             │                   │
      Kotlin declaration      Target output
        information               │
             │              ┌────┴────┐
             │              │         │
             ▼              ▼         ▼
       Kotlin compiler   Android     iOS
        + tooling        artifact   artifact
```

> **Metadata is part of the bridge between Kotlin source-level meaning and compiled multiplatform artifacts.**

Once you understand that distinction, compiler errors, dependency resolution, binary compatibility, source-set boundaries, and KMP library publishing become much easier to reason about.

---

### Quick Reference

| Concept | Role |
|---|---|
| **Source code** | Human-readable Kotlin program |
| **Compilation** | Transforms and validates Kotlin for a target |
| **Metadata** | Kotlin-specific declaration information |
| **JVM bytecode** | JVM-oriented executable representation |
| **Kotlin/Native output** | Native target representation/artifact |
| **Kotlin/JS / Wasm output** | Web-oriented target output |
| **Source sets** | Define compilation inputs and sharing boundaries |
| **`expect` / `actual`** | Connect common declarations with platform implementations |
| **Dependency** | Supplies declarations and implementation to a compilation |
| **Compiler plugin** | Extends compiler behavior |
| **ABI** | Binary-level interface consumed by compiled code |
| **IDE tooling** | Uses compiler/declaration information for developer assistance |

> **When you understand metadata, compiled KMP libraries stop looking like black boxes—you can reason about what the compiler knows, what the target needs, and where compatibility problems originate.**
