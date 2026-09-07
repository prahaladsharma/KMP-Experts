# Chapter 9 — Kotlin Compiler Deep Dive

## Part 3 — Kotlin IR

> **Kotlin IR (Intermediate Representation) is the compiler's structured representation of Kotlin code used between source-level analysis and target-specific code generation.**

If metadata explains **what Kotlin declarations mean**, Kotlin IR helps explain **how the compiler transforms those declarations into executable target code**.

For Kotlin Multiplatform, this distinction is particularly important because the same Kotlin source can ultimately target very different platforms.

```text
Kotlin Source
     │
     ▼
Frontend / Analysis
     │
     ▼
   Kotlin IR
     │
     ├───────────────┐
     ▼               ▼
JVM Backend      Native Backend
     │               │
     ▼               ▼
JVM Output       Native Output
```

The exact compiler architecture evolves over time, but this high-level model is an excellent foundation for understanding KMP compilation.

---

## 1. What Is Intermediate Representation?

An **Intermediate Representation**, or IR, is a structured representation of a program used internally by a compiler.

Instead of immediately translating:

```kotlin
fun add(a: Int, b: Int): Int {
    return a + b
}
```

directly into platform machine code or JVM instructions, the compiler can represent the program in a form that is easier to analyze and transform.

Conceptually:

```text
Source Code
    ↓
IR
    ↓
Target-specific representation
    ↓
Executable output
```

The IR acts as an important bridge between Kotlin-level program meaning and platform-specific code generation.

---

## 2. Why Does Kotlin Need IR?

Kotlin targets multiple platforms:

```text
JVM
Android
iOS / Native
macOS / Native
Linux / Native
Windows / Native
JavaScript
WebAssembly
```

Each platform has different execution models and output formats.

If every compiler feature had to be implemented independently for every backend, compiler development would become extremely difficult.

A shared IR provides a common representation where many transformations can be performed once.

```text
                    Kotlin Source
                         │
                         ▼
                    Kotlin IR
                         │
        ┌────────────────┼────────────────┐
        ▼                ▼                ▼
       JVM             Native             JS/Wasm
     backend          backend            backend
        │                │                │
        ▼                ▼                ▼
     JVM output       Native output     Web output
```

This is one of the major architectural benefits of IR.

---

## 3. The Compiler Pipeline

A simplified Kotlin compiler pipeline can be viewed as:

```text
Source Code
     │
     ▼
Parsing
     │
     ▼
Frontend / Semantic Analysis
     │
     ▼
Kotlin IR
     │
     ▼
IR Transformations
     │
     ▼
Backend
     │
     ▼
Target Output
```

This is intentionally simplified.

Modern Kotlin compiler internals contain additional stages, compiler phases, plugins, lowerings, and target-specific processing.

The important idea is the direction:

> **Source-level Kotlin meaning is progressively transformed into a representation that can be lowered and generated for a target platform.**

---

## 4. Kotlin IR Is Not Source Code

Consider:

```kotlin
val result = user.name.uppercase()
```

The IR is not simply another text version of this Kotlin code.

It is a structured compiler representation containing nodes and relationships describing things such as:

```text
Variable declaration
Property access
Function call
Receiver
Arguments
Types
Control flow
Expressions
```

Conceptually:

```text
Variable
  │
  └── Call
       │
       ├── Receiver → user.name
       └── Function → uppercase()
```

The compiler can transform this structure without manipulating raw source text.

---

## 5. IR as a Tree-Like Structure

A simplified example:

```kotlin
fun greet(name: String): String {
    return "Hello $name"
}
```

can be thought of conceptually as:

```text
Function: greet
│
├── Parameter: name
│
├── Return Type: String
│
└── Body
    │
    └── Return
        │
        └── String Template
            ├── "Hello "
            └── name
```

The actual Kotlin IR is considerably richer and has specific compiler data structures.

The diagram is only a mental model.

---

## 6. IR Contains More Than Syntax

The source code contains syntax.

The compiler needs more than syntax.

For example:

```kotlin
user.getName()
```

requires the compiler to understand:

```text
user type
getName declaration
receiver type
parameter types
return type
visibility
dispatch
```

By the time code reaches important IR processing stages, the compiler has already performed substantial semantic analysis.

This makes IR useful for transformations that operate on the meaning and structure of the program rather than on raw source text.

---

## 7. Frontend vs IR

A useful conceptual distinction is:

### Frontend

Responsible for understanding Kotlin source:

```text
Syntax
Types
Names
Resolution
Semantic rules
Diagnostics
```

### IR

Represents the analyzed program in a compiler-oriented form suitable for further transformations and backend processing.

```text
Kotlin source
      ↓
Frontend
      ↓
Analyzed program
      ↓
IR
```

### Backend

Transforms and generates target-specific output:

```text
IR
 ↓
Lowering
 ↓
Target-specific code generation
 ↓
Platform output
```

---

## 8. What Is IR Lowering?

One of the most important concepts around Kotlin IR is **lowering**.

Lowering means transforming higher-level constructs into representations closer to what the target backend needs.

Think:

```text
High-level Kotlin concept
          ↓
       Lowering
          ↓
More explicit representation
          ↓
       Lowering
          ↓
Target-oriented representation
```

A compiler can therefore gradually reduce complex language concepts into simpler operations.

---

## 9. A Simple Lowering Example

Suppose Kotlin contains a high-level construct:

```kotlin
data class User(
    val id: String,
    val name: String
)
```

A data class has compiler-generated behavior such as:

```text
equals()
hashCode()
toString()
componentN()
copy()
```

The compiler can represent and eventually generate these behaviors without requiring the developer to manually write them.

Conceptually:

```text
data class
    ↓
IR transformation / lowering
    ↓
Generated functions and implementation details
    ↓
Target backend
```

The exact internal phases are compiler-version dependent.

---

## 10. Another Example: `when`

Consider:

```kotlin
val result = when (state) {
    State.Loading -> "Loading"
    State.Success -> "Success"
    State.Error -> "Error"
}
```

At the source level, this is concise.

The backend eventually needs a lower-level representation that can be translated to the target platform.

Conceptually:

```text
when expression
      ↓
IR representation
      ↓
lowering
      ↓
conditional / dispatch structure
      ↓
target code
```

The developer writes the high-level Kotlin construct.

The compiler performs the transformations.

---

## 11. IR and Kotlin Multiplatform

This is where IR becomes especially interesting for KMP.

A common Kotlin source file can be processed into Kotlin IR before being handled by the appropriate target backend.

For example:

```text
commonMain
    │
    ▼
Kotlin compiler
    │
    ▼
Kotlin IR
    │
    ├───────────────┐
    ▼               ▼
Android/JVM       Kotlin/Native
backend           backend
    │               │
    ▼               ▼
Android output    iOS output
```

The shared source does not become one universal executable.

Instead, it participates in target-specific compilation pipelines.

---

## 12. One Source, Multiple Backends

Consider:

```kotlin
class Greeting {
    fun message(): String = "Hello"
}
```

The Kotlin source can be shared.

But:

```text
Android
    ↓
JVM-oriented backend

iOS
    ↓
Kotlin/Native backend
```

The backends ultimately generate different forms of target output.

IR gives the compiler a common representation from which these different paths can proceed.

---

## 13. Why This Matters for KMP Architecture

A common misconception is:

> "KMP compiles common Kotlin once and copies the result to every platform."

That is not the right mental model.

A better model is:

```text
                 common Kotlin
                      │
                      ▼
                 compilation
                      │
                      ▼
                   Kotlin IR
                      │
          ┌───────────┴───────────┐
          ▼                       ▼
      JVM backend             Native backend
          │                       │
          ▼                       ▼
     Android output           iOS output
```

The common source is shared.

The final target compilation remains platform-aware.

---

## 14. IR and Platform-Specific Code

Suppose common code calls:

```kotlin
expect fun platformName(): String
```

Android provides:

```kotlin
actual fun platformName(): String = "Android"
```

iOS provides:

```kotlin
actual fun platformName(): String = "iOS"
```

The compiler must resolve the appropriate declarations for each target.

The resulting compilation is therefore target-specific.

```text
commonMain
    expect platformName()
           │
           ▼
       resolution
       /       \
      /         \
 Android       iOS
 actual        actual
```

IR is part of the broader compiler pipeline that eventually transforms the resolved program into target output.

---

## 15. IR and `expect` / `actual`

`expect` / `actual` is a compiler feature.

The compiler must verify relationships between declarations.

For example:

```kotlin
// commonMain
expect class Platform {
    val name: String
}
```

and:

```kotlin
// androidMain
actual class Platform {
    actual val name: String = "Android"
}
```

The compiler checks that the actual declaration satisfies the expected contract.

After resolution and analysis, the relevant declarations participate in the target compilation.

The important mental model is:

```text
expect/actual resolution
        ↓
compiler representation
        ↓
IR processing
        ↓
target backend
```

---

## 16. IR and Compiler Plugins

IR is especially important when working with Kotlin compiler plugins.

A compiler plugin can participate in compiler processing and, depending on the plugin architecture, inspect or transform compiler representations.

Conceptually:

```text
Kotlin source
     ↓
Compiler
     ↓
IR
     ↓
Plugin / transformation
     ↓
Modified IR
     ↓
Backend
```

This allows language or framework tooling to generate or transform behavior at compile time.

---

## 17. IR and Generated Code

Some Kotlin features are implemented partly through compiler-generated behavior.

For example:

```kotlin
data class User(
    val id: String
)
```

The developer does not explicitly write every generated method.

The compiler can represent generated declarations during compilation.

Conceptually:

```text
Developer code
      +
Compiler-generated declarations
      ↓
IR
      ↓
Backend
```

This is one reason IR is powerful: the compiler can operate on a unified internal representation.

---

## 18. IR and Compose

Jetpack Compose and Compose Multiplatform use compiler-integrated tooling.

The compiler can transform code to support framework-specific behavior.

At a conceptual level:

```text
Composable Kotlin
      ↓
Compiler processing
      ↓
IR transformations
      ↓
Target backend
      ↓
Platform output
```

The exact implementation details depend on the Compose compiler and Kotlin versions.

The broader lesson is:

> **IR provides a powerful place for compiler transformations to operate on Kotlin programs.**

---

## 19. IR and Serialization

Kotlin serialization also demonstrates the importance of compiler-assisted transformations.

For example:

```kotlin
@Serializable
data class User(
    val id: String,
    val name: String
)
```

The serialization ecosystem can generate or integrate serialization-related code during compilation.

A simplified model is:

```text
Kotlin declaration
       ↓
Compiler processing
       ↓
Generated / transformed representation
       ↓
IR / backend
       ↓
Target artifact
```

Again, the exact mechanism should not be reduced to "serialization simply edits IR"; implementation details vary by toolchain.

---

## 20. IR Is Not the Final Machine Code

This distinction is critical.

Think:

```text
Kotlin source
     ↓
IR
     ↓
Lowered IR
     ↓
Backend
     ↓
Target-specific representation
     ↓
Executable/library artifact
```

IR is an intermediate stage.

It is not:

```text
JVM bytecode
```

and it is not:

```text
ARM machine code
```

It is a compiler representation used before final target-specific generation.

---

## 21. JVM and Kotlin/Native

The target backend matters.

For JVM:

```text
Kotlin
  ↓
IR
  ↓
JVM backend
  ↓
JVM bytecode
```

For Kotlin/Native:

```text
Kotlin
  ↓
IR
  ↓
Native backend
  ↓
Native-oriented output
```

This is one of the key reasons the same Kotlin language can support multiple platforms.

---

## 22. IR and Kotlin/JS

The same conceptual model applies to Kotlin/JS:

```text
Kotlin
  ↓
IR
  ↓
JS backend
  ↓
JavaScript output
```

The target backend has different requirements from JVM or Native.

The shared compiler representation reduces the amount of language-level compiler logic that needs to be independently implemented for every backend.

---

## 23. IR and WebAssembly

Kotlin/Wasm follows the same broad compiler principle:

```text
Kotlin
  ↓
Compiler
  ↓
IR
  ↓
Wasm-oriented backend
  ↓
WebAssembly output
```

The target-specific backend is responsible for producing output appropriate for the platform.

---

## 24. Why IR Helps Compiler Consistency

Without a shared intermediate representation, imagine implementing every Kotlin language feature separately:

```text
Feature
 ├── JVM implementation
 ├── Native implementation
 ├── JS implementation
 └── Wasm implementation
```

With a shared IR approach, many transformations can operate on a common representation:

```text
Feature
    ↓
Common compiler processing
    ↓
IR
    ↓
Target-specific backend
```

This does not eliminate backend-specific work.

It reduces unnecessary duplication.

---

## 25. Backend-Specific Lowerings

Not every transformation is completely platform-independent.

A target may need specific handling.

For example:

```text
Common IR
    │
    ├── JVM-specific lowering
    │
    ├── Native-specific lowering
    │
    └── JS/Wasm-specific lowering
```

This allows the compiler to preserve a shared language model while still respecting target capabilities and constraints.

---

## 26. IR and Platform Constraints

Different platforms have different characteristics.

For example:

```text
JVM
- Managed runtime
- JVM bytecode
- Garbage collection

Native
- Native machine code
- Platform-specific ABI
- Native runtime model

JS
- JavaScript runtime
- JS ecosystem

Wasm
- WebAssembly execution model
```

The compiler cannot simply generate identical output for all of them.

IR provides a common starting point, while target-specific processing handles differences.

---

## 27. IR and Optimization

IR can also be useful for optimization.

A compiler can analyze and transform program structures before final code generation.

Conceptually:

```text
IR
 ↓
Analysis
 ↓
Transformation
 ↓
Optimized IR
 ↓
Backend
```

Examples of compiler optimization concepts include:

```text
Removing unnecessary work
Simplifying expressions
Inlining where applicable
Eliminating unreachable code
Improving control flow
```

The exact optimizations depend on the Kotlin compiler, backend, configuration, and target.

---

## 28. IR and Function Inlining

Consider:

```kotlin
inline fun square(x: Int): Int {
    return x * x
}
```

The `inline` modifier communicates a compiler-level transformation opportunity.

The compiler can process this through its compilation pipeline and produce target-specific output reflecting the inline semantics.

The important point is not the exact IR node sequence.

The important point is:

```text
Kotlin feature
    ↓
Compiler understanding
    ↓
IR transformation
    ↓
Backend output
```

---

## 29. IR and Control Flow

Consider:

```kotlin
fun calculate(value: Int): Int {
    if (value > 10) {
        return value * 2
    }

    return value + 1
}
```

A compiler representation must capture the control-flow structure:

```text
Function
   │
   ▼
Condition
 ┌─┴─────────────┐
 ▼               ▼
true             false
 │                 │
return x * 2      return x + 1
```

The backend can then translate that structure into the target platform's control-flow instructions.

---

## 30. IR and Type Information

Compiler transformations must preserve type correctness.

Consider:

```kotlin
val count: Int = 10
```

The compiler understands:

```text
count
type = Int
value = 10
```

If a transformation changes the representation, it must preserve the required type semantics.

This is another reason a structured IR is preferable to simple text substitution.

---

## 31. Why Text-Based Transformation Would Be Dangerous

Imagine trying to implement compiler transformations by editing Kotlin source strings:

```text
Find "data class"
Replace with generated functions
Rewrite calls
Change expressions
```

This would be fragile because source text does not directly encode all semantic relationships.

IR provides structured nodes and compiler information.

Instead of:

```text
String manipulation
```

the compiler can perform:

```text
Structured transformation
```

This is safer and more precise.

---

## 32. IR and Compiler Diagnostics

Compiler diagnostics can occur at different stages.

Some errors are detected during source analysis.

Others may appear during later compiler processing.

For example:

```text
Source analysis
     ↓
Type checking
     ↓
IR transformation
     ↓
Backend processing
```

A failure during an IR-related stage may therefore look very different from a normal Kotlin syntax error.

When debugging compiler or plugin issues, identifying the failing stage is useful.

---

## 33. IR and Incremental Compilation

Large projects benefit from avoiding unnecessary work.

A simplified build:

```text
Module A
   ↓
Module B
   ↓
Module C
```

If Module A changes, the build system and compiler determine what needs to be rebuilt.

IR is one part of the compiler pipeline involved in producing compiled output.

The exact incremental compilation strategy is more complex than simply "recompile changed IR."

---

## 34. IR and Multi-Module KMP Projects

Consider:

```text
:core
:data
:feature
:app
```

Each module may have multiple target compilations.

Conceptually:

```text
                 :core
               /       \
              /         \
        Android          iOS
           │               │
           ▼               ▼
         IR              IR
           │               │
           ▼               ▼
       backend          backend
```

As the project grows, understanding these compilation boundaries helps explain:

```text
Build time
Dependency resolution
Compiler errors
Target-specific behavior
Binary compatibility
```

---

## 35. IR and the KMP Build Graph

The Gradle build graph determines which compilations are created and which dependencies they receive.

Then the Kotlin compiler processes each compilation.

Conceptually:

```text
Gradle configuration
        ↓
KMP source-set graph
        ↓
Compilation inputs
        ↓
Kotlin compiler
        ↓
IR
        ↓
Target backend
        ↓
Artifact
```

This shows an important separation:

> **Gradle organizes the build; the Kotlin compiler processes Kotlin code.**

---

## 36. IR Does Not Replace Gradle

A common misunderstanding is:

> "IR is part of Gradle."

It is better to separate the responsibilities.

### Gradle

Handles build orchestration:

```text
Projects
Tasks
Dependencies
Variants
Toolchain configuration
Build lifecycle
```

### Kotlin compiler

Handles Kotlin compilation:

```text
Parsing
Analysis
IR
Lowering
Code generation
```

The two systems work together, but they solve different problems.

---

## 37. IR and the Runtime

IR exists during compilation.

It should therefore be distinguished from runtime structures.

```text
Build time
──────────────
Kotlin source
Compiler
IR
Lowerings
Backend
Artifact

Runtime
──────────────
Application
Platform runtime
Kotlin runtime components
```

The application does not normally execute Kotlin IR directly.

---

## 38. IR and Native Compilation

Kotlin/Native makes the compiler pipeline especially interesting for KMP developers.

For an iOS target, a simplified path is:

```text
commonMain / iosMain
        ↓
Kotlin compiler
        ↓
Kotlin IR
        ↓
Native-specific processing
        ↓
Native backend
        ↓
Apple platform artifact
```

The final result is native platform code rather than JVM bytecode.

---

## 39. IR and Objective-C / Swift Interoperability

Kotlin/Native supports interoperability with Apple technologies.

For example, a Kotlin declaration may need to participate in an Objective-C or Swift-facing API.

The compiler must account for:

```text
Kotlin declaration
       ↓
Native representation
       ↓
Interop / ABI requirements
       ↓
Apple-compatible output
```

IR is part of the compiler processing pipeline that precedes final target output.

Interop itself involves additional compiler and toolchain mechanisms.

---

## 40. IR and `suspend` Functions

Consider:

```kotlin
suspend fun loadUser(): User
```

A suspend function has semantics that require compiler transformation.

The compiler needs to represent coroutine-related behavior in a form that the target backend can generate correctly.

Conceptually:

```text
suspend function
       ↓
compiler transformation
       ↓
IR representation
       ↓
lowering
       ↓
target-specific implementation
```

This is a powerful example of why the compiler cannot simply translate source syntax directly to final machine instructions.

---

## 41. IR and `inline` / `reified`

Consider:

```kotlin
inline fun <reified T> create(): T
```

`reified` type parameters depend on compiler behavior because normal runtime generic information is not sufficient to implement the language semantics directly.

The compiler processes these constructs and performs the required transformations.

Again:

```text
High-level Kotlin feature
        ↓
Compiler analysis
        ↓
IR / lowering
        ↓
Backend output
```

---

## 42. IR and Language Features

Many Kotlin features require compiler understanding.

Examples include:

```text
Classes
Functions
Properties
Data classes
Delegation
Coroutines
Inline functions
Generics
Smart casts
When expressions
Object declarations
Sealed hierarchies
```

The compiler transforms these high-level concepts into representations suitable for target execution.

IR is central to that transformation architecture.

---

## 43. How to Think About IR as a Developer

You usually do not need to inspect IR every day.

Instead, use IR as a mental model when dealing with:

- Compiler plugins
- Compose compiler behavior
- Serialization
- Advanced Kotlin compilation
- KMP target differences
- Compiler errors
- Native compilation
- Generated code
- Performance-related compiler transformations

It helps answer:

> **"What happens to my Kotlin code between source code and the final platform artifact?"**

---

## 44. A Practical Debugging Model

When a compiler-related problem appears, think in stages:

```text
1. Source
   ↓
2. Resolution / analysis
   ↓
3. IR creation
   ↓
4. IR transformation / lowering
   ↓
5. Backend processing
   ↓
6. Target output
```

Then ask:

```text
Where does the failure occur?
```

This is much more effective than treating the compiler as one black box.

---

## 45. Common IR Mistakes

### Mistake 1 — Thinking IR Is Bytecode

**Incorrect:**

```text
IR = JVM bytecode
```

**Better:**

```text
IR → backend → target-specific output
```

---

### Mistake 2 — Thinking IR Is Source Code

IR is structured compiler data, not a second copy of the source.

---

### Mistake 3 — Assuming All Targets Have Identical Backends

KMP supports different target environments with different backend requirements.

---

### Mistake 4 — Assuming Every Compiler Feature Is Platform-Independent

Many transformations can be shared, but target-specific lowering and code generation remain necessary.

---

### Mistake 5 — Assuming Gradle Generates IR

Gradle orchestrates the build.

The Kotlin compiler performs the language compilation and IR processing.

---

## 46. Metadata vs IR

The previous part introduced metadata.

It is useful to compare the two.

| Concept | Primary purpose |
|---|---|
| **Metadata** | Kotlin-specific information about compiled declarations |
| **IR** | Structured compiler representation used for transformations and code generation |
| **Backend** | Converts compiler representation into target output |
| **Artifact** | Result consumed by the platform/build ecosystem |

A simplified relationship:

```text
Source
  ↓
Compiler analysis
  ├───────────────┐
  ▼               ▼
Metadata          IR
                  │
                  ▼
               Lowering
                  │
                  ▼
                Backend
                  │
                  ▼
              Artifact
```

They are related, but they solve different problems.

---

## 47. The Complete Compiler Mental Model

Putting the previous parts together:

```text
                    Kotlin Source
                         │
                         ▼
                Source-set / Module
                         │
                         ▼
              Frontend / Analysis
                         │
              ┌──────────┴──────────┐
              ▼                     ▼
          Metadata                  IR
              │                     │
              │              IR transformations
              │                     │
              │                  Lowering
              │                     │
              │                     ▼
              │                  Backend
              │                     │
              │                     ▼
              │               Target artifact
              │
              ▼
      Kotlin declaration
       understanding
```

This is a simplified conceptual model, not a complete compiler implementation diagram.

---

## 48. KMP From Source to Platform

For a KMP project:

```text
                  commonMain
                      │
                      ▼
                 Kotlin compiler
                      │
                      ▼
                      IR
                      │
          ┌───────────┴───────────┐
          ▼                       ▼
      Android/JVM               iOS
        backend                Native
          │                    backend
          ▼                       ▼
      Android output         Native output
```

Platform-specific source participates in the relevant target compilation.

This is why KMP provides code sharing without pretending that Android and iOS are identical platforms.

---

## 49. Why IR Matters for Multiplatform Architecture

KMP architecture is often discussed in terms of:

```text
commonMain
androidMain
iosMain
```

But underneath the source-set model is a compiler pipeline.

```text
Architecture
     ↓
Source sets
     ↓
Compilation inputs
     ↓
Compiler analysis
     ↓
IR
     ↓
Target backend
     ↓
Platform artifact
```

Understanding this pipeline helps architects make better decisions about:

```text
What belongs in common code
What requires platform APIs
How compiler plugins affect the build
Why target-specific behavior exists
Why some libraries support only certain targets
```

---

## 50. The Most Important Mental Model

Do not think:

```text
KMP
 ↓
common Kotlin
 ↓
copy code to Android and iOS
```

Think:

```text
                    Shared Kotlin
                         │
                         ▼
                  Compiler analysis
                         │
                         ▼
                      Kotlin IR
                         │
              ┌──────────┴──────────┐
              ▼                     ▼
          JVM backend           Native backend
              │                     │
              ▼                     ▼
        Android artifact       iOS artifact
```

The same Kotlin language model is shared.

The final execution model remains platform-specific.

---

## 51. Key Takeaways

> **1. IR means Intermediate Representation.**

It is a structured compiler representation between source-level Kotlin and target-specific output.

> **2. IR is not bytecode.**

The backend still has to generate target-specific output.

> **3. IR enables shared compiler transformations.**

Many Kotlin language features can be processed through common compiler infrastructure before target-specific generation.

> **4. KMP relies on target-specific compilation.**

Common Kotlin does not become one universal binary.

> **5. Lowering transforms high-level constructs.**

Compiler transformations gradually make Kotlin constructs suitable for backend processing.

> **6. Compiler plugins can work with compiler representations.**

This enables powerful compile-time transformations and generated behavior.

> **7. IR and metadata are different.**

Metadata primarily helps describe Kotlin declarations, while IR represents the program for compiler processing and code generation.

> **8. Gradle and the Kotlin compiler have different responsibilities.**

Gradle orchestrates the build; the Kotlin compiler performs Kotlin compilation.

---

## Final Perspective

When an Android developer first encounters KMP, the visible architecture looks simple:

```text
commonMain
androidMain
iosMain
```

But underneath that structure is a sophisticated compiler pipeline.

Your common Kotlin code is analyzed and represented by the compiler. The compiler then performs transformations and lowerings before passing the appropriate representation to the target backend.

Conceptually:

```text
             Kotlin Source
                  │
                  ▼
             Compilation
                  │
                  ▼
               Kotlin IR
                  │
          ┌───────┴───────┐
          ▼               ▼
       JVM Backend     Native Backend
          │               │
          ▼               ▼
       Android             iOS
       Artifact          Artifact
```

This is the key architectural insight:

> **Kotlin Multiplatform shares the language and compiler model, not the runtime implementation of every platform.**

Once you understand IR, several KMP concepts become easier to connect:

```text
Source Sets
     ↓
Compilation
     ↓
Metadata
     ↓
IR
     ↓
Lowering
     ↓
Target Backend
     ↓
Platform Artifact
```

That is the path from the Kotlin code you write to the platform-specific application code that ultimately runs.

---

### Quick Reference

| Term | Meaning |
|---|---|
| **Frontend** | Understands and analyzes Kotlin source |
| **IR** | Intermediate compiler representation |
| **Lowering** | Transforms higher-level constructs into lower-level representations |
| **Backend** | Generates target-specific output |
| **JVM backend** | Produces JVM-oriented output |
| **Native backend** | Produces native-oriented output |
| **JS backend** | Produces JavaScript-oriented output |
| **Wasm backend** | Produces WebAssembly-oriented output |
| **Compiler plugin** | Extends or transforms compiler processing |
| **Metadata** | Kotlin-specific declaration information |
| **Artifact** | Compiled output consumed by the build/platform ecosystem |

> **IR is the bridge between Kotlin's high-level language model and the very different execution environments supported by Kotlin Multiplatform.**
