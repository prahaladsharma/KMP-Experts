# Chapter 9 — Kotlin Compiler Deep Dive

## Part 6 — WebAssembly (Wasm)

> **WebAssembly gives Kotlin another compilation target: a compact, portable binary format designed to run efficiently in web browsers and other Wasm runtimes.**

Kotlin/Wasm extends Kotlin's multiplatform model into WebAssembly.

Instead of compiling Kotlin source into JVM bytecode or native machine-oriented output, the compiler produces WebAssembly-oriented artifacts.

A simplified model is:

```text
Kotlin Source
     │
     ▼
Kotlin Compiler
     │
     ▼
Kotlin IR
     │
     ▼
Wasm-specific transformations
     │
     ▼
WebAssembly
     │
     ├── Browser
     │
     └── Other Wasm runtimes
```

The important architectural idea is the same as with other Kotlin targets:

> **The source language can remain Kotlin, while the compiler backend determines the target representation and runtime model.**

---

## 1. What Is WebAssembly?

WebAssembly, commonly called **Wasm**, is a portable binary instruction format designed for efficient execution.

It is commonly associated with web browsers, but Wasm is not limited to browsers.

Conceptually:

```text
Source language
      ↓
Compiler
      ↓
WebAssembly module
      ↓
Wasm runtime
      ↓
CPU
```

Languages other than Kotlin can also target WebAssembly.

---

## 2. Why Kotlin Targets Wasm

Kotlin already supports multiple compilation targets.

A simplified view is:

```text
                    Kotlin
                       │
                       ▼
                 Kotlin Compiler
                       │
              ┌────────┼────────┐
              ▼        ▼        ▼
             JVM     Native    Wasm
              │        │        │
           Android    iOS     Browser
                              / Wasm
```

Wasm provides Kotlin with a path to web and portable Wasm environments while retaining Kotlin language and tooling concepts.

---

## 3. Kotlin/Wasm

Kotlin/Wasm is the Kotlin compiler target for WebAssembly.

It enables Kotlin code to be compiled to WebAssembly rather than JVM bytecode or native platform output.

Conceptually:

```text
Kotlin
  ↓
Kotlin/Wasm
  ↓
Wasm module
  ↓
Wasm runtime
```

The exact generated artifacts and integration model depend on the Kotlin version, Gradle configuration, and target environment.

---

## 4. Kotlin/Wasm and Kotlin Multiplatform

KMP can share code across several targets.

For example:

```text
                    commonMain
                        │
                        ▼
                  Kotlin Compiler
                        │
          ┌─────────────┼─────────────┐
          ▼             ▼             ▼
         JVM          Native          Wasm
          │             │              │
       Android          iOS         Browser
```

This means Wasm can participate as another target in a multiplatform architecture.

The important distinction is:

```text
Shared source
     ≠
Shared generated artifact
```

Each target has its own compilation path.

---

## 5. WebAssembly Is Not JavaScript

A common misconception is:

```text
Kotlin/Wasm = Kotlin compiled to JavaScript
```

That is incorrect.

Conceptually:

```text
Kotlin/JS
    ↓
JavaScript

Kotlin/Wasm
    ↓
WebAssembly
```

These are different compilation targets.

They have different output formats, runtime characteristics, interoperability models, and tooling considerations.

---

## 6. Wasm Binary Format

A WebAssembly module is represented in a compact binary format.

For development and inspection, WebAssembly can also be represented in a human-readable text format commonly called **WAT**.

Conceptually:

```text
Wasm binary
     ↕
WAT representation
```

Developers generally work with Kotlin source rather than writing Wasm instructions directly.

---

## 7. The Kotlin/Wasm Compilation Pipeline

A simplified pipeline is:

```text
                 Kotlin Source
                       │
                       ▼
                Compiler Analysis
                       │
                       ▼
                    Kotlin IR
                       │
                       ▼
               Wasm Transformations
                       │
                       ▼
                Wasm Code Generation
                       │
                       ▼
                 Wasm Module
                       │
                       ▼
                 Wasm Runtime
```

The exact internal compiler stages evolve over time, but the target-aware model remains useful.

---

## 8. Kotlin IR and Wasm

Kotlin IR provides a common intermediate representation for Kotlin compiler processing.

Conceptually:

```text
Kotlin source
     ↓
Analysis
     ↓
Kotlin IR
     ↓
Wasm backend
     ↓
WebAssembly
```

This allows Kotlin language constructs to be transformed before final Wasm generation.

---

## 9. Why an Intermediate Representation Matters

Consider a Kotlin function:

```kotlin
fun add(a: Int, b: Int): Int {
    return a + b
}
```

The compiler does not need to immediately convert the source into Wasm instructions.

Instead, it can represent the program using compiler-level structures.

Conceptually:

```text
Kotlin syntax
     ↓
Semantic representation
     ↓
Kotlin IR
     ↓
Wasm representation
```

This separation makes compiler transformations easier to reason about.

---

## 10. Target-Specific Lowering

Kotlin features do not necessarily have one-to-one WebAssembly instructions.

For example:

```text
Kotlin classes
Kotlin objects
Properties
Lambdas
Coroutines
Generics
Exceptions
Collections
```

must be represented using the capabilities of the Wasm target and its runtime.

The compiler therefore performs target-specific transformations.

Conceptually:

```text
Kotlin feature
     ↓
IR transformation
     ↓
Wasm-compatible representation
```

---

## 11. WebAssembly's Execution Model

WebAssembly uses a structured execution model designed for safe and portable execution.

A simplified view is:

```text
Wasm Module
    │
    ├── Code
    ├── Data
    ├── Imports
    └── Exports
```

The runtime loads the module and makes its exported functionality available to the host environment.

---

## 12. Modules

A Wasm application is commonly packaged as one or more modules.

Conceptually:

```text
Application
     ↓
Wasm module
     ↓
Runtime
```

The host can instantiate the module and provide imported capabilities.

For browser applications, JavaScript and browser APIs commonly form part of the host environment.

---

## 13. Imports and Exports

Wasm supports imports and exports.

Conceptually:

```text
Host
  │
  ├── imports → Wasm
  │
  └── calls ← exported Wasm functions
```

This creates an interoperability boundary.

Kotlin/Wasm applications can therefore communicate with their host environment through supported integration mechanisms.

---

## 14. Browser as a Wasm Host

A browser can act as a Wasm runtime.

A simplified model is:

```text
Browser
   │
   ├── JavaScript environment
   │
   └── WebAssembly runtime
            │
            ▼
        Kotlin/Wasm
```

This is important because WebAssembly itself does not provide every browser capability.

Browser APIs such as:

```text
DOM
Fetch
Web Storage
Canvas
Events
Web APIs
```

belong to the browser environment.

---

## 15. Wasm Does Not Replace Web APIs

A Kotlin/Wasm application still needs a way to interact with browser capabilities.

Think of the architecture as:

```text
Kotlin/Wasm
     │
     ▼
WebAssembly
     │
     ▼
Browser runtime
     │
     ▼
Web APIs
```

WebAssembly provides the execution target.

The browser provides the environment.

---

## 16. Kotlin/Wasm and JavaScript Interoperability

For browser applications, Kotlin/Wasm may need to interact with JavaScript.

Conceptually:

```text
Kotlin
  ↕
Wasm
  ↕
JavaScript
  ↕
Browser APIs
```

This is different from Kotlin/JS, where JavaScript is the primary compilation target.

The interoperability boundary should therefore be treated as a platform boundary.

---

## 17. Why the Boundary Matters

Suppose application code needs:

```text
window
document
localStorage
fetch
DOM events
```

These are browser concepts.

They should not automatically be placed in `commonMain`.

A better architectural model is:

```text
commonMain
    │
    └── shared application logic

wasmJsMain
    │
    └── browser/Wasm-specific integration
```

This keeps platform-specific concerns at the correct layer.

---

## 18. Wasm Source Sets in KMP

A multiplatform project can organize source sets around shared and target-specific code.

A conceptual structure might look like:

```text
shared/
└── src/
    ├── commonMain/
    ├── commonTest/
    └── wasmJsMain/
```

Depending on the project and targets, intermediate source sets can also be used.

For example:

```text
commonMain
    │
    └── wasmJsMain
            │
            └── wasmJs-specific code
```

The exact hierarchy is determined by the Gradle target configuration.

---

## 19. Example Source Set

A simplified Gradle configuration can look like:

```kotlin
kotlin {
    wasmJs {
        browser()
    }

    sourceSets {
        commonMain.dependencies {
            // Shared dependencies
        }

        wasmJsMain.dependencies {
            // Wasm/browser-specific dependencies
        }
    }
}
```

The exact syntax can change as Kotlin and Gradle APIs evolve, so project configuration should follow the Kotlin version's current documentation.

---

## 20. Wasm and Compose Multiplatform

Compose Multiplatform can target Wasm for supported web scenarios.

Conceptually:

```text
Compose UI
    ↓
Kotlin/Wasm
    ↓
WebAssembly
    ↓
Browser
```

This enables Kotlin-based declarative UI to participate in web applications through WebAssembly.

The UI architecture can remain Kotlin-centric while the target runtime changes.

---

## 21. Shared UI Architecture

A multiplatform application can conceptually look like:

```text
                    commonMain
                        │
                ┌───────┴───────┐
                │               │
             Domain            UI
                │               │
                └───────┬───────┘
                        │
             ┌──────────┼──────────┐
             ▼          ▼          ▼
           Android      iOS       Wasm
```

The amount of code that can be shared depends on the libraries and platform APIs used.

---

## 22. Wasm Runtime

WebAssembly itself is an execution format.

Kotlin/Wasm also requires runtime support for Kotlin language and library behavior that cannot be expressed directly through basic Wasm instructions.

Conceptually:

```text
Wasm module
     +
Kotlin runtime support
     +
Host environment
     ↓
Running Kotlin/Wasm application
```

This is important when considering application size, startup behavior, and runtime capabilities.

---

## 23. Memory Model

WebAssembly provides linear memory concepts for applications that need memory.

A simplified model is:

```text
Wasm Instance
     │
     └── Linear Memory
              │
              ├── application data
              └── runtime-managed data
```

Kotlin's runtime and generated code use the available Wasm memory model to represent application state.

Developers generally do not manage Wasm memory manually when writing ordinary Kotlin code.

---

## 24. Garbage Collection

Modern WebAssembly environments can provide garbage-collection capabilities through Wasm GC.

This is significant for languages such as Kotlin because Kotlin programs naturally work with managed objects.

Conceptually:

```text
Kotlin objects
      ↓
Wasm GC / runtime support
      ↓
managed object lifetime
```

The exact capabilities available depend on the selected Wasm environment and Kotlin/Wasm implementation.

---

## 25. Wasm GC and Language Runtimes

Without appropriate runtime support, a high-level language would need to build substantial object-management machinery on top of lower-level Wasm primitives.

Wasm GC provides standardized primitives that can better support managed-language runtimes.

The architecture can therefore be thought of as:

```text
Kotlin object model
       ↓
Kotlin/Wasm runtime
       ↓
Wasm GC capabilities
       ↓
Wasm runtime
```

---

## 26. Exceptions

Kotlin supports exceptions:

```kotlin
try {
    loadData()
} catch (e: Exception) {
    handleError(e)
}
```

Exception handling needs to be represented in the Wasm execution model.

The compiler and runtime cooperate to map Kotlin exception semantics onto the target.

Conceptually:

```text
Kotlin exception
      ↓
compiler transformation
      ↓
Wasm exception/runtime support
```

---

## 27. Coroutines on Wasm

Kotlin coroutines are another example of a high-level language abstraction that must be represented for the target.

For example:

```kotlin
suspend fun loadUser(): User {
    return repository.load()
}
```

The compiler transforms coroutine-related constructs into a form that can execute on the selected target.

Conceptually:

```text
suspend function
      ↓
Kotlin compiler
      ↓
IR transformations
      ↓
Wasm-compatible representation
      ↓
Wasm runtime
```

---

## 28. `suspend` Does Not Mean "Wasm Instruction"

There is no simple Wasm instruction called:

```text
SUSPEND
```

Instead, coroutine semantics are implemented through compiler transformations and runtime mechanisms.

This is the same broader compiler lesson seen in Kotlin/JVM:

> **High-level language constructs are translated into target-compatible lower-level representations.**

---

## 29. Kotlin Collections on Wasm

Kotlin collections such as:

```kotlin
List
Set
Map
MutableList
```

are part of the Kotlin programming model.

The Wasm target needs runtime and generated-code support for these abstractions.

The source remains:

```kotlin
val users = mutableListOf<User>()
```

while the compiled representation is target-specific.

---

## 30. Strings

Kotlin strings also require runtime representation.

The source-level abstraction:

```kotlin
val name = "Kotlin"
```

is not itself a Wasm primitive.

The compiler and runtime provide the required string representation and operations.

This illustrates the layered model:

```text
Kotlin String
     ↓
Kotlin/Wasm runtime representation
     ↓
Wasm-supported memory/object model
```

---

## 31. Numeric Types

Some Kotlin primitive types map naturally to Wasm numeric concepts.

For example:

```text
Int
Long
Float
Double
```

have corresponding low-level numeric representations that can be efficiently mapped to Wasm capabilities.

However, Kotlin's complete type semantics still belong to the Kotlin language and compiler.

---

## 32. Classes and Objects

Kotlin supports:

```kotlin
class User(
    val name: String
)
```

and:

```kotlin
object Configuration
```

WebAssembly does not have a direct Kotlin class/object syntax.

The compiler and runtime provide the required representation.

Conceptually:

```text
Kotlin class/object
       ↓
compiler lowering
       ↓
Wasm-compatible object representation
```

---

## 33. Reflection

Reflection deserves special attention.

Reflection is deeply connected to:

```text
Metadata
Runtime information
Generated structures
Dynamic lookup
```

Not every reflection capability available on another Kotlin target should be assumed to behave identically on Wasm.

When designing a multiplatform library, prefer explicit APIs and compile-time mechanisms when full reflection is not required.

---

## 34. Serialization on Wasm

Serialization libraries can be especially useful in Wasm applications because they provide structured data conversion without requiring a JVM-style reflection model.

Conceptually:

```text
Kotlin data class
      ↓
Serialization plugin/runtime
      ↓
Wasm-compatible generated code
      ↓
JSON / other format
```

This is a common multiplatform architectural pattern.

---

## 35. Networking

A browser-based Wasm application does not automatically gain arbitrary native networking capabilities.

Instead, it operates within the host environment's security and networking model.

For example:

```text
Kotlin/Wasm
     ↓
HTTP client abstraction
     ↓
Browser-compatible networking
     ↓
Web environment
```

Libraries and APIs must support the Wasm target and its host constraints.

---

## 36. Browser Security Model

Wasm applications running in browsers are subject to browser security rules.

These can include:

```text
Same-origin policy
CORS
Content Security Policy
Sandboxing
Permission restrictions
```

These are not Kotlin compiler concepts.

They belong to the browser environment.

Therefore:

```text
Compiler problem
    ≠
Browser security problem
```

Distinguishing the layers makes debugging easier.

---

## 37. Wasm and the DOM

The DOM is a browser API.

It is not part of WebAssembly itself.

The architectural path is:

```text
Kotlin/Wasm
     ↓
Interop boundary
     ↓
Browser JavaScript / Web APIs
     ↓
DOM
```

This is why browser-specific code should remain isolated from platform-independent business logic.

---

## 38. Startup and Download Size

Web applications must download the required application resources before useful execution can begin.

A simplified model is:

```text
Browser
   ↓
Download application assets
   ↓
Load Wasm
   ↓
Initialize runtime
   ↓
Start application
```

Therefore, generated artifact size and initialization work matter.

Compiler optimizations, dead-code elimination, resource optimization, and application architecture can influence the final experience.

---

## 39. Wasm and Performance

WebAssembly is designed for efficient execution.

However, application performance depends on much more than the binary format.

Consider:

```text
Kotlin source
      ↓
Compiler
      ↓
Generated Wasm
      ↓
Runtime initialization
      ↓
Browser
      ↓
DOM / Web APIs
```

Performance bottlenecks can occur at any of these boundaries.

---

## 40. CPU Work vs Browser Work

A useful distinction is:

```text
CPU-heavy computation
        ↓
Wasm can be a strong fit

DOM-heavy interaction
        ↓
Browser APIs become important
```

A fast Wasm function does not automatically make every browser operation fast.

DOM manipulation, layout, rendering, networking, and JavaScript interoperability have their own costs.

---

## 41. Wasm and JavaScript Interop Cost

When application code crosses boundaries:

```text
Kotlin/Wasm
     ↕
JavaScript
```

there can be additional conversion and invocation overhead.

Therefore, architecture should avoid unnecessary boundary crossings.

Prefer:

```text
Many operations
      ↓
one well-defined boundary
```

over:

```text
operation
  ↓
JS
  ↓
Wasm
  ↓
JS
  ↓
Wasm
  ↓
...
```

when the workload makes such crossings expensive.

---

## 42. Dependency Support

Not every Kotlin library automatically supports Wasm.

A library may support:

```text
JVM
Android
iOS
```

but not:

```text
Wasm
```

KMP architects should therefore inspect target support before adding dependencies.

A useful dependency matrix is:

| Dependency | common | Android | iOS | Wasm |
|---|---:|---:|---:|---:|
| Library A | ✓ | ✓ | ✓ | ✓ |
| Library B | ✓ | ✓ | ✓ | — |
| Library C | ✓ | ✓ | — | ✓ |

The exact support should always be verified against the library's current documentation.

---

## 43. Source Set Dependency Resolution

Suppose:

```text
commonMain
   ↓
Library A
```

and:

```text
wasmJsMain
   ↓
Library B
```

The build system resolves dependencies according to the source set and target graph.

Conceptually:

```text
commonMain
    │
    ├── shared dependency
    │
    ▼
wasmJsMain
    │
    └── Wasm-specific dependency
```

This allows shared code and target-specific code to coexist cleanly.

---

## 44. Platform-Specific APIs

If a library is only available for JVM:

```text
JVM-only dependency
      ↓
commonMain
      ✕
```

It should not be placed directly into common code that must compile for Wasm.

Instead, isolate platform-specific behavior.

For example:

```text
commonMain
   ↓
expect API

wasmJsMain
   ↓
Wasm implementation
```

or use a multiplatform abstraction provided by a library.

---

## 45. `expect` / `actual` with Wasm

A common architecture can be:

```kotlin
// commonMain
expect fun platformName(): String
```

and:

```kotlin
// wasmJsMain
actual fun platformName(): String = "Wasm"
```

The important idea is:

```text
common API
     ↓
target-specific implementation
```

Wasm can therefore participate in the same multiplatform abstraction pattern as other targets.

---

## 46. Wasm and Source Set Hierarchy

A project with multiple targets may have:

```text
commonMain
    │
    ├── androidMain
    │
    ├── iosMain
    │
    └── wasmJsMain
```

Or a more specialized hierarchy:

```text
commonMain
     │
     └── webMain
           │
           └── wasmJsMain
```

The actual hierarchy depends on the targets and source-set relationships configured by the project.

---

## 47. Browser-Specific Architecture

A clean Wasm application can separate:

```text
commonMain
├── Domain
├── Use cases
├── Shared models
├── Shared state
└── Shared business rules

wasmJsMain
├── Browser integration
├── Web-specific APIs
└── Wasm-specific wiring
```

This keeps the shared layer independent from browser implementation details.

---

## 48. Wasm Application Boundary

A useful mental model is:

```text
┌─────────────────────────────────────┐
│           Shared Kotlin             │
│                                     │
│ Domain • State • Use Cases • Models │
└──────────────────┬──────────────────┘
                   │
                   ▼
          ┌────────────────┐
          │ Kotlin/Wasm    │
          └───────┬────────┘
                  │
          ┌───────┴────────┐
          ▼                ▼
      Web APIs         Wasm Runtime
          │                │
          └───────┬────────┘
                  ▼
               Browser
```

This diagram helps prevent platform concerns from leaking into common code.

---

## 49. Wasm vs Kotlin/JS

Both can target the web, but their compilation targets differ.

| Area | Kotlin/JS | Kotlin/Wasm |
|---|---|---|
| Primary output | JavaScript | WebAssembly |
| Runtime environment | JavaScript | Wasm runtime + host |
| Browser APIs | JavaScript/web APIs | Host/web APIs through supported interop |
| Compilation model | Kotlin → JS | Kotlin → Wasm |
| JavaScript ecosystem integration | Direct | Through interoperability boundary |
| Binary format | JS source/modules | Wasm module |

The right choice depends on application requirements, library support, ecosystem needs, and target constraints.

---

## 50. Wasm vs JVM

| Area | Kotlin/JVM | Kotlin/Wasm |
|---|---|---|
| Output | JVM bytecode | WebAssembly |
| Runtime | JVM | Wasm runtime |
| Main environments | Android, backend, desktop JVM | Web and Wasm environments |
| Java ecosystem | Extensive | Not JVM-based |
| Browser execution | Not native | Designed for web/Wasm environments |
| GC | JVM GC | Wasm/Kotlin runtime model |
| Interop | Java/JVM | Host/JavaScript/Web APIs |

The two targets can share Kotlin source while producing completely different artifacts.

---

## 51. Wasm vs Kotlin/Native

| Area | Kotlin/Native | Kotlin/Wasm |
|---|---|---|
| Output | Native-oriented artifact | Wasm module |
| Typical targets | iOS, macOS, Linux and others | Web/Wasm environments |
| Host | Native OS/runtime | Wasm host |
| Browser focus | No | Yes |
| Memory/runtime model | Kotlin/Native | Kotlin/Wasm + Wasm runtime |
| Interop | Native platform APIs | Wasm/host APIs |

Both are non-JVM targets, but their execution environments are very different.

---

## 52. Debugging Kotlin/Wasm

When debugging a Wasm application, think in layers:

```text
Kotlin source
      ↓
Kotlin compiler
      ↓
Kotlin IR
      ↓
Wasm output
      ↓
Wasm runtime
      ↓
JavaScript/browser host
      ↓
Browser APIs
```

A problem may originate in any layer.

---

## 53. Common Wasm Problems

Typical issues can include:

```text
Unsupported dependency
Missing Wasm target support
Browser API incompatibility
JavaScript interop issues
Large application artifacts
Startup performance
Source-set dependency errors
Unsupported reflection behavior
Browser security restrictions
Runtime initialization failures
```

The correct fix depends on which layer owns the problem.

---

## 54. Diagnose by Boundary

Use this approach:

```text
Does common code compile?
       │
       ▼
Does Wasm compilation succeed?
       │
       ▼
Does the module load?
       │
       ▼
Does runtime initialization succeed?
       │
       ▼
Does browser interop work?
       │
       ▼
Does the Web API behave correctly?
```

This is much more effective than treating every issue as a generic "Kotlin problem."

---

## 55. Artifact Inspection

When necessary, inspect the generated Wasm artifacts.

A useful workflow is:

```text
Kotlin source
      ↓
Build
      ↓
Generated Wasm artifacts
      ↓
Inspect module/resources
      ↓
Browser runtime
```

The goal is not to become a Wasm bytecode expert for every application.

The goal is to understand enough of the generated artifact to identify architectural or build problems.

---

## 56. Toolchain Awareness

Wasm development involves multiple tools:

```text
Kotlin compiler
Gradle
Wasm tooling
Browser
JavaScript tooling
Web server
Debugger
```

A failure can come from mismatched versions or unsupported combinations.

Therefore, keep the following aligned:

```text
Kotlin version
Gradle configuration
Plugins
Dependencies
Browser/runtime capabilities
```

---

## 57. Testing

KMP source-set testing can include common tests:

```text
commonTest
     ↓
shared test logic
     ↓
target-specific test execution
```

Wasm-specific tests depend on the configured target and testing environment.

A good architecture keeps as much business logic as possible in `commonMain`, allowing a large portion of the behavior to be validated through shared tests.

---

## 58. WebAssembly Is a Target, Not an Architecture

This distinction is important.

Avoid thinking:

```text
"We use Wasm, therefore our architecture is different."
```

Instead:

```text
Application architecture
        +
Wasm target
```

A well-designed KMP application can keep:

```text
Domain
Use cases
State
Models
Business rules
```

shared while isolating:

```text
Browser APIs
Web-specific rendering
Wasm-specific integration
```

in the appropriate source sets.

---

## 59. What Should Go Into `commonMain`?

Good candidates include:

```text
Business rules
Domain models
Use cases
State management
Validation
Networking abstractions
Serialization models
Shared repositories
Pure Kotlin utilities
```

Avoid putting direct browser dependencies there unless the entire project architecture intentionally supports them across all required targets.

---

## 60. What Should Go Into `wasmJsMain`?

Typical candidates include:

```text
Browser integration
Wasm-specific implementations
Web platform APIs
Target-specific initialization
Browser-specific persistence
Target-specific UI integration
```

The principle is:

> **Keep the shared layer platform-independent and move platform knowledge toward the target boundary.**

---

## 61. Architecture Example

A practical structure might look like:

```text
shared/
└── src/
    ├── commonMain/
    │   ├── domain/
    │   ├── data/
    │   ├── presentation/
    │   └── platform/
    │
    ├── commonTest/
    │
    ├── androidMain/
    │
    ├── iosMain/
    │
    └── wasmJsMain/
        ├── platform/
        └── web/
```

The exact structure should follow the project's scale and responsibilities.

---

## 62. A Complete Mental Model

For Kotlin/Wasm, use this mental model:

```text
                         Kotlin Source
                              │
                              ▼
                       Compiler Analysis
                              │
                              ▼
                          Kotlin IR
                              │
                              ▼
                      Wasm-specific lowering
                              │
                              ▼
                       Wasm code generation
                              │
                              ▼
                         Wasm Module
                              │
                     ┌────────┴────────┐
                     ▼                 ▼
                  Browser         Other Wasm Host
                     │
              ┌──────┴──────┐
              ▼             ▼
        JavaScript        Web APIs
              │             │
              └──────┬──────┘
                     ▼
                  Application
```

This model explains why Kotlin/Wasm is more than simply "Kotlin for the browser."

---

## 63. Key Takeaways

> **1. Kotlin/Wasm is a real compilation target.**

Kotlin source can be compiled into WebAssembly rather than JavaScript or JVM bytecode.

> **2. Wasm is not JavaScript.**

Kotlin/Wasm and Kotlin/JS represent different target architectures.

> **3. Kotlin IR remains important.**

The compiler can transform Kotlin semantics before generating target-specific Wasm output.

> **4. Browser APIs are host capabilities.**

DOM, storage, networking, and other web APIs are provided by the browser environment rather than by WebAssembly itself.

> **5. Source-set boundaries matter.**

Shared code belongs in common source sets; Wasm/browser-specific behavior belongs in the appropriate target source set.

> **6. Dependencies must support Wasm.**

A dependency that works on JVM and iOS does not automatically work on Wasm.

> **7. Runtime support matters.**

Kotlin language features require runtime and compiler support on the Wasm target.

> **8. Performance is end-to-end.**

Generated Wasm performance is only one part of the application. Startup, runtime initialization, browser APIs, DOM operations, and boundary crossings also matter.

> **9. Wasm can participate in KMP architecture.**

The same common Kotlin architecture can target Android, iOS, and Wasm when dependencies and platform boundaries are designed correctly.

> **10. Wasm is a target, not a replacement for architecture.**

Good multiplatform architecture still depends on clear boundaries between shared business logic and platform-specific capabilities.

---

## Final Perspective

WebAssembly adds another important dimension to Kotlin Multiplatform.

The fundamental pipeline is:

```text
Kotlin
   ↓
Kotlin Compiler
   ↓
Kotlin IR
   ↓
Wasm Backend
   ↓
WebAssembly
   ↓
Wasm Runtime / Browser
```

And in a KMP application:

```text
                         commonMain
                             │
                             ▼
                       Kotlin Compiler
                             │
                             ▼
                          Kotlin IR
                             │
                  ┌──────────┼──────────┐
                  ▼          ▼          ▼
                 JVM        Native      Wasm
                  │           │          │
               Android        iOS      Browser
```

The architectural lesson is straightforward:

> **Compile shared Kotlin once conceptually, but let each target express that shared model through its own backend and runtime.**

For Wasm, that means understanding three boundaries clearly:

```text
Kotlin
  ↓
WebAssembly
  ↓
Host environment
```

Once these boundaries are clear, Kotlin/Wasm becomes much easier to reason about—from compiler output and runtime behavior to source-set design, dependency selection, browser integration, and performance.

---

### Quick Reference

| Term | Meaning |
|---|---|
| **Wasm** | WebAssembly, a portable binary instruction format |
| **Kotlin/Wasm** | Kotlin compiler target that generates WebAssembly |
| **Kotlin IR** | Intermediate representation used during Kotlin compilation |
| **Wasm module** | Compiled WebAssembly unit loaded by a Wasm runtime |
| **Wasm runtime** | Environment that loads and executes WebAssembly |
| **Host** | Environment providing capabilities to a Wasm module |
| **Import** | Capability/function supplied to a Wasm module |
| **Export** | Function/value exposed by a Wasm module |
| **WAT** | Human-readable text representation of WebAssembly |
| **Wasm GC** | WebAssembly garbage-collection capabilities for managed objects |
| **Kotlin/JS** | Kotlin target that compiles to JavaScript |
| **Kotlin/Native** | Kotlin target that produces native-oriented artifacts |
| **wasmJsMain** | KMP source set commonly used for Wasm JavaScript/browser targeting |
| **commonMain** | Shared multiplatform source set |
| **Interop** | Communication between Kotlin/Wasm and host technologies |
| **Host API** | Capability provided by the environment running Wasm |

> **The key idea: Kotlin/Wasm does not remove the need to understand platforms. It changes the platform boundary—from JVM or native APIs to a WebAssembly runtime and its host environment.**
