# Chapter 8 — `expect` / `actual`

## Part 2 — Compiler Resolution

> **`expect` / `actual` works because the Kotlin compiler connects a common declaration with the appropriate platform implementation during compilation.**

The previous part introduced the architectural purpose of `expect` / `actual`.

Now we move one level deeper:

**How does the compiler actually resolve the relationship?**

Understanding this process is important because `expect` / `actual` is not a runtime mechanism. The compiler uses the multiplatform source-set and target configuration to determine which `actual` declaration satisfies an `expect` declaration.

---

## 1. The Basic Resolution Flow

At a high level, the process looks like this:

```text
commonMain
    │
    │ expect declaration
    ▼
Kotlin compiler
    │
    │ target-aware resolution
    ▼
Platform source set
    │
    │ actual declaration
    ▼
Platform compilation
    │
    ▼
Platform artifact
```

For example:

```kotlin
// commonMain
expect fun platformName(): String
```

The Android compilation resolves it against:

```kotlin
// androidMain
actual fun platformName(): String {
    return "Android"
}
```

While the iOS compilation resolves it against:

```kotlin
// iosMain
actual fun platformName(): String {
    return "iOS"
}
```

The important point is that **there is no runtime search for an implementation**.

---

# 2. `expect` Is a Compiler-Level Contract

Consider:

```kotlin
expect fun platformName(): String
```

The declaration tells the compiler:

```text
A declaration with this contract
must have a corresponding platform implementation.
```

The compiler records the declaration as an expected platform-specific API.

When a target is compiled, the compiler must determine:

```text
Which actual declaration satisfies this expect?
```

If it cannot find one, compilation fails.

---

# 3. The Source Set Is Part of the Resolution Story

A KMP project normally contains source sets such as:

```text
commonMain
androidMain
iosMain
```

The source-set hierarchy determines which code is visible to which target.

A simplified structure might be:

```text
commonMain
    │
    ├───────────────┐
    ▼               ▼
androidMain      iosMain
```

An `expect` declaration in `commonMain` is therefore visible to the platform compilations that depend on `commonMain`.

The corresponding `actual` implementation must be available to the relevant target.

---

# 4. A Simple Example

### `commonMain`

```kotlin
expect fun platformName(): String
```

### `androidMain`

```kotlin
actual fun platformName(): String {
    return "Android"
}
```

### `iosMain`

```kotlin
actual fun platformName(): String {
    return "iOS"
}
```

The conceptual compiler view is:

```text
Android compilation

expect platformName()
        │
        ▼
actual platformName() → "Android"
```

and:

```text
iOS compilation

expect platformName()
        │
        ▼
actual platformName() → "iOS"
```

The same common declaration participates in different target compilations.

---

# 5. There Is No Runtime `if`

A common misunderstanding is to imagine something like:

```kotlin
if (platform == ANDROID) {
    useAndroidImplementation()
} else {
    useIosImplementation()
}
```

That is not how `expect` / `actual` works.

Instead:

```text
                common code
                    │
               expect API
                    │
          ┌─────────┴─────────┐
          ▼                   ▼
     Android build        iOS build
          │                   │
       actual               actual
          │                   │
          ▼                   ▼
     Android code          iOS code
```

Each platform compilation resolves its own implementation.

---

# 6. Compile-Time vs Runtime

This distinction is critical.

### Runtime dispatch

Usually means:

```text
Application starts
       ↓
Check platform
       ↓
Select implementation
```

### `expect` / `actual`

Works conceptually like:

```text
Compilation starts
       ↓
Select target
       ↓
Resolve expected declaration
       ↓
Find matching actual
       ↓
Compile target
       ↓
Produce platform artifact
```

The application does not need to perform a runtime implementation lookup.

---

# 7. What Happens If `actual` Is Missing?

Suppose:

```kotlin
expect fun platformName(): String
```

exists in:

```text
commonMain
```

but the Android target has no corresponding:

```kotlin
actual fun platformName(): String
```

The compiler cannot satisfy the expectation for that target.

The result is a compilation error.

Conceptually:

```text
commonMain
    │
    └── expect platformName()
              │
              ▼
        Android target
              │
              X
        actual not found
              │
              ▼
        Compilation fails
```

This is one of the biggest benefits of the mechanism.

A missing platform implementation is detected during development/build rather than discovered later at runtime.

---

# 8. Matching Is More Than a Function Name

The compiler does not simply search for:

```text
platformName
```

and accept any function with that name.

The declarations must correspond appropriately.

For example:

```kotlin
expect fun platformName(): String
```

requires an appropriate:

```kotlin
actual fun platformName(): String
```

Changing the return type would break the relationship:

```kotlin
actual fun platformName(): Int
```

The compiler must ensure that the expected declaration and actual declaration are compatible.

---

# 9. Signature Compatibility Matters

Consider:

```kotlin
expect fun saveValue(
    key: String,
    value: String
)
```

The implementation must satisfy the same API contract:

```kotlin
actual fun saveValue(
    key: String,
    value: String
) {
    // implementation
}
```

An incompatible declaration such as:

```kotlin
actual fun saveValue(
    value: String
) {
}
```

does not satisfy the expectation.

The compiler therefore acts as a contract checker.

---

# 10. Classes Can Also Use `expect` / `actual`

The mechanism is not limited to functions.

For example:

```kotlin
// commonMain
expect class Platform {
    val name: String
}
```

Android can provide:

```kotlin
// androidMain
actual class Platform {
    actual val name: String = "Android"
}
```

iOS can provide:

```kotlin
// iosMain
actual class Platform {
    actual val name: String = "iOS"
}
```

The compiler resolves the expected class against the appropriate actual class for each target.

---

# 11. Properties Can Be Expected

For example:

```kotlin
expect val platformName: String
```

An actual implementation can provide:

```kotlin
actual val platformName: String
    get() = "Android"
```

Again, the important concept is:

```text
expect declaration
        ↓
compiler resolution
        ↓
actual declaration
```

---

# 12. Objects and Other Declarations

`expect` / `actual` can be used for several Kotlin declarations, including suitable:

```text
Functions
Classes
Objects
Properties
Type-related declarations
```

The exact language rules depend on the declaration type and Kotlin version.

The architectural principle remains the same:

```text
Common declaration
        ↓
Platform implementation
```

---

# 13. Resolution Follows the Multiplatform Model

KMP does not treat every source file as an isolated compilation unit.

Instead, it understands relationships between:

```text
Source sets
Targets
Compilations
Dependencies
```

For example:

```text
commonMain
    │
    ├─────────────┐
    ▼             ▼
androidMain    iosMain
    │             │
Android        iOS
compilation   compilation
```

The compiler uses this model to determine which declarations are available to each compilation.

---

# 14. Intermediate Source Sets Change the Picture

Consider:

```text
commonMain
     │
     ▼
 appleMain
   │    │
   ▼    ▼
 iosMain macosMain
```

Suppose `commonMain` contains:

```kotlin
expect fun platformName(): String
```

The `actual` implementation could potentially be provided at:

```text
appleMain
```

if that implementation is valid for all targets using that source set.

The resolution path becomes:

```text
commonMain
    │
    │ expect
    ▼
appleMain
    │
    │ actual
    ▼
iOS / macOS compilations
```

This is why source-set hierarchy is not merely an organizational feature.

It influences dependency and declaration visibility.

---

# 15. The Compiler Must See the Correct `actual`

Consider:

```text
commonMain
    │
    └── expect
          │
          ▼
      appleMain
          │
          └── actual
```

If the actual declaration is placed somewhere outside the relevant compilation's source-set dependency graph, it cannot satisfy the expectation for that target.

This is an important debugging clue:

> If an `actual` declaration appears correct but the compiler still says it is missing, check the source-set hierarchy and target configuration.

---

# 16. Think in Terms of Compilation Graphs

A useful mental model is:

```text
             commonMain
                 │
        ┌────────┴────────┐
        ▼                 ▼
   androidMain         appleMain
                           │
                    ┌──────┴──────┐
                    ▼             ▼
                 iosMain       macosMain
```

Each target compilation sees the source sets that belong to its dependency path.

Therefore:

```text
Android compilation
    → commonMain
    → androidMain

iOS compilation
    → commonMain
    → appleMain
    → iosMain

macOS compilation
    → commonMain
    → appleMain
    → macosMain
```

This is a simplified model, but it is extremely useful when reasoning about resolution.

---

# 17. Why Source Set Hierarchy Matters

Suppose you accidentally put an iOS-only implementation into:

```text
commonMain
```

That is not allowed because common code cannot depend on iOS-only APIs.

Instead:

```text
commonMain
    expect
       │
       ▼
iosMain
    actual
```

The source-set boundary tells the compiler where platform-specific code is allowed.

---

# 18. Resolution Is Target-Aware

A KMP project may contain multiple targets:

```text
Android
iOS
JVM
Desktop
Wasm
```

The same common source set may participate in several compilations.

The compiler therefore needs target-specific resolution.

Conceptually:

```text
              commonMain
                  │
       ┌──────────┼──────────┐
       ▼          ▼          ▼
    Android      iOS       Desktop
       │          │          │
    actual      actual     actual
```

Each compilation receives the appropriate platform implementation.

---

# 19. Why One `expect` Can Have Multiple `actual`s

The common declaration is shared:

```kotlin
expect fun platformName(): String
```

But the implementation naturally differs:

```text
Android → "Android"
iOS     → "iOS"
Desktop → "Desktop"
```

Therefore:

```text
1 expect
   ↓
multiple target-specific actual implementations
```

This is one of the defining characteristics of multiplatform compilation.

---

# 20. The Compiler Prevents Accidental Ambiguity

A platform compilation should not end up with multiple incompatible implementations that could both satisfy the same expectation.

The source-set hierarchy and compiler rules help determine the valid declaration.

If the project structure creates an invalid or ambiguous setup, the build can fail rather than silently choosing an arbitrary implementation.

This is another reason to keep the source-set hierarchy clean.

---

# 21. `actual` Is Not an Interface Implementation

Consider:

```kotlin
interface Storage {
    fun save(value: String)
}
```

and:

```kotlin
class AndroidStorage : Storage
```

Here the runtime application can potentially choose:

```text
AndroidStorage
```

through dependency injection or another composition mechanism.

With:

```kotlin
expect fun storage(): Storage
```

the platform implementation is connected through the multiplatform compilation model.

The two mechanisms solve related but different problems.

---

# 22. Compiler Resolution vs Dependency Injection

A useful comparison:

| Concern | `expect` / `actual` | Dependency Injection |
|---|---|---|
| Main purpose | Platform-specific declaration | Dependency composition |
| Selection | Target-aware compilation | Application configuration |
| Typical timing | Compile time | Often runtime/startup |
| Source-set awareness | Yes | Not inherently |
| Platform boundary | Explicit | Depends on architecture |
| Testing flexibility | Requires platform/test structure | Often highly flexible |
| Runtime implementation switching | Not the primary purpose | Possible |

Neither mechanism replaces the other.

---

# 23. A Practical Example: Platform Clock

Suppose business logic needs a current time abstraction.

### `commonMain`

```kotlin
expect fun currentTimeMillis(): Long
```

### `androidMain`

```kotlin
actual fun currentTimeMillis(): Long {
    return System.currentTimeMillis()
}
```

### `iosMain`

```kotlin
actual fun currentTimeMillis(): Long {
    // Platform-specific implementation
    return ...
}
```

The common use case can simply call:

```kotlin
fun createTimestamp(): Long {
    return currentTimeMillis()
}
```

The compiler connects the common call to the appropriate target implementation.

---

# 24. What the Generated Application Sees

At the conceptual level, after target-specific compilation, Android code behaves as though it had a concrete implementation:

```text
Android application
        │
        ▼
platformName()
        │
        ▼
Android actual implementation
```

The iOS application has its own resolved implementation:

```text
iOS application
        │
        ▼
platformName()
        │
        ▼
iOS actual implementation
```

There is no need for the application to maintain a universal implementation registry.

---

# 25. Compilation Errors Are Useful Feedback

When you see errors related to:

```text
expected declaration
actual declaration
missing actual
actual has no corresponding expected declaration
```

do not immediately assume the Kotlin syntax is wrong.

Check:

1. Is the `expect` declaration in the correct source set?
2. Is the `actual` declaration in the correct source set?
3. Is the target configured?
4. Does the source set belong to that target?
5. Does the declaration signature match?
6. Is an intermediate source set involved?
7. Is the expected declaration visible to the target compilation?
8. Is the actual declaration visible to the target compilation?

These questions solve many resolution problems.

---

# 26. A Debugging Diagram

When an `actual` is reported as missing, draw the source-set graph:

```text
                 commonMain
                     │
              expect Foo
                     │
            ┌────────┴────────┐
            ▼                 ▼
       androidMain         appleMain
            │                 │
         actual Foo      ┌────┴────┐
                         ▼         ▼
                      iosMain   macosMain
```

Then ask:

```text
Does the target compilation
actually include the source set
where the actual declaration lives?
```

If the answer is no, the compiler cannot resolve it.

---

# 27. Keep the Contract Small

A smaller expectation makes compiler resolution easier to reason about.

Prefer:

```kotlin
expect fun secureStorage(): SecureStorage
```

over exposing a large platform abstraction such as:

```kotlin
expect class PlatformManager {
    fun getContext(): Any
    fun getActivity(): Any
    fun getViewController(): Any
    fun openUrl(url: String)
    fun saveSecureValue(...)
    fun getDeviceInfo(...)
    ...
}
```

Large platform abstractions often become difficult to maintain and can leak platform concepts into shared code.

---

# 28. Resolution Does Not Make Platforms Identical

The compiler can connect:

```text
expect
```

to:

```text
actual
```

but it does not magically make Android and iOS APIs equivalent.

The implementations can remain completely different internally.

For example:

```text
expect:
    secureStore.save(key, value)

Android actual:
    Android security APIs

iOS actual:
    Apple security APIs
```

The common contract is shared.

The implementation remains platform-native.

---

# 29. `expect` / `actual` and Clean Architecture

The compiler-resolution model fits naturally into layered architecture.

For example:

```text
┌───────────────────────────────┐
│        Presentation           │
├───────────────────────────────┤
│          Use Cases            │
├───────────────────────────────┤
│          Domain               │
├───────────────────────────────┤
│ Shared platform capability    │
│          expect               │
└───────────────┬───────────────┘
                │
        ┌───────┴────────┐
        ▼                ▼
   Android actual    iOS actual
```

The common layers remain portable.

Platform implementation stays at the platform boundary.

---

# 30. The Most Important Mental Model

Do not think:

```text
expect → find actual at runtime
```

Think:

```text
expect
  ↓
target compilation
  ↓
source-set resolution
  ↓
matching actual
  ↓
platform compilation
```

That is the mental model that makes KMP much easier to understand.

---

# 31. What Happens When a New Platform Is Added?

Suppose the application originally supports:

```text
Android
iOS
```

and later adds:

```text
Desktop
```

If `commonMain` contains:

```kotlin
expect fun platformName(): String
```

the new target needs a suitable implementation.

Conceptually:

```text
Before:

expect
 ├── Android actual
 └── iOS actual

After:

expect
 ├── Android actual
 ├── iOS actual
 └── Desktop actual
```

This makes platform-specific responsibilities visible during compilation.

---

# 32. Why This Is Better Than Hidden Runtime Logic

Consider a runtime approach:

```kotlin
when (platform) {
    ANDROID -> ...
    IOS -> ...
    DESKTOP -> ...
}
```

When a new platform is added, you may forget to update one or more branches.

With `expect` / `actual`, the new target's compilation can expose missing implementations.

This shifts part of the problem from:

```text
Runtime correctness
```

toward:

```text
Compile-time correctness
```

---

# 33. A Compiler-Oriented View

A simplified conceptual process looks like this:

```text
Step 1
Read common declarations

        ↓

Step 2
Identify target compilation

        ↓

Step 3
Build the source-set visibility graph

        ↓

Step 4
Find expected declarations

        ↓

Step 5
Find corresponding actual declarations

        ↓

Step 6
Validate compatibility

        ↓

Step 7
Compile the target

        ↓

Step 8
Produce target-specific output
```

The real compiler implementation is more sophisticated, but this model is useful for architecture and debugging.

---

# 34. Common Resolution Mistakes

### ❌ Wrong source set

Putting an Android implementation into `iosMain` will not satisfy the Android target.

### ❌ Wrong signature

The actual declaration must correctly correspond to the expectation.

### ❌ Missing target configuration

An `iosMain` implementation is useful only when the corresponding iOS target/compilation is configured.

### ❌ Incorrect source-set hierarchy

An `actual` in an unrelated source set may not be visible to the target compilation.

### ❌ Assuming runtime selection

`expect` / `actual` does not provide a runtime plugin registry.

### ❌ Overusing `expect`

Not every interface needs an `expect` declaration.

### ❌ Leaking platform types

Avoid making common APIs depend on Android or iOS implementation classes.

---

# 35. Resolution and Testing

Testing introduces another important consideration.

Common tests can validate shared behavior:

```text
commonTest
    ↓
common logic
```

Platform-specific tests can validate actual implementations:

```text
androidTest / Android-specific tests
ios-specific tests
```

A useful structure is:

```text
commonTest
    │
    ├── validates common behavior
    │
    ▼
expect contract

platform tests
    │
    ├── Android actual
    └── iOS actual
```

This keeps the testing responsibilities aligned with the source-set architecture.

---

# 36. `expect` / `actual` Is a Contract, Not a Shortcut

It can be tempting to use:

```kotlin
expect
```

whenever a platform difference appears.

A better approach is to first identify:

```text
What is actually different?
```

Then define the smallest possible expected API.

For example:

```kotlin
expect fun openUrl(url: String)
```

is usually easier to reason about than:

```kotlin
expect class PlatformNavigationManager
```

with dozens of unrelated operations.

---

# 37. Design for the Compiler

Good KMP architecture makes the compiler's job straightforward.

Prefer:

```text
Small contract
      ↓
Clear source-set ownership
      ↓
One appropriate actual implementation
      ↓
Target compilation
```

Avoid:

```text
Large abstraction
      ↓
Platform conditionals
      ↓
Multiple overlapping source sets
      ↓
Unclear ownership
```

The compiler can enforce a clean architecture only when the architecture itself is clear.

---

# 38. A Complete Example

### Common

```kotlin
expect class PlatformInfo {
    val name: String
}
```

### Android

```kotlin
actual class PlatformInfo {
    actual val name: String = "Android"
}
```

### iOS

```kotlin
actual class PlatformInfo {
    actual val name: String = "iOS"
}
```

Common business logic:

```kotlin
class GreetingUseCase(
    private val platformInfo: PlatformInfo
) {
    fun execute(): String {
        return "Running on ${platformInfo.name}"
    }
}
```

The important architectural relationship is:

```text
commonMain
    │
    ├── PlatformInfo expectation
    │
    └── GreetingUseCase
             │
             ▼
       PlatformInfo
             │
       ┌─────┴─────┐
       ▼           ▼
    Android       iOS
     actual       actual
```

The business logic stays common.

The platform implementation stays platform-specific.

---

# 39. The Bigger Picture

`expect` / `actual` is one piece of a larger KMP compilation architecture:

```text
Kotlin source
     │
     ▼
Source sets
     │
     ▼
Target configuration
     │
     ▼
Compiler resolution
     │
     ├── common declarations
     ├── expected declarations
     └── actual declarations
     │
     ▼
Platform compilation
     │
     ▼
Platform artifact
```

This is why understanding `expect` / `actual` also requires understanding:

- Source sets
- Target configuration
- Compilation
- Dependency graphs
- Intermediate source sets
- Platform-specific APIs

---

# 40. Chapter Takeaways

> [!IMPORTANT]
> **The Kotlin compiler resolves `expect` declarations against appropriate `actual` declarations for each multiplatform target during compilation.**

Remember:

1. `expect` defines a platform-dependent declaration in shared code.
2. `actual` provides the implementation for a specific target.
3. Resolution is target-aware.
4. Resolution is tied to the KMP source-set model.
5. `expect` / `actual` is not a runtime platform switch.
6. The compiler performs the important matching work during compilation.
7. Each platform compilation can resolve the same expectation differently.
8. One `expect` can have multiple platform-specific `actual` implementations.
9. An Android compilation resolves against Android-visible source sets.
10. An iOS compilation resolves against iOS-visible source sets.
11. An intermediate source set can provide an `actual` when appropriate.
12. Source-set hierarchy determines declaration visibility.
13. An `actual` outside the target's source-set graph cannot satisfy the target.
14. Missing `actual` implementations can cause compilation failures.
15. Signature compatibility is required.
16. Function return types must correspond appropriately.
17. Function parameters must correspond appropriately.
18. Expected classes require corresponding actual implementations.
19. Expected properties require corresponding actual declarations.
20. The compiler validates the expected/actual relationship.
21. `expect` does not dynamically search for implementations.
22. There is no runtime implementation registry created by `expect`.
23. Platform code remains platform-specific after resolution.
24. Compiler resolution does not make platform APIs identical.
25. The goal is controlled platform separation.
26. Source-set hierarchy is part of the resolution model.
27. Clean source-set design makes resolution easier to understand.
28. Incorrect source-set placement is a common cause of resolution errors.
29. Incorrect signatures are another common cause.
30. Missing target configuration can also cause problems.
31. `expect` / `actual` and dependency injection solve different problems.
32. Interfaces and `expect` / `actual` are not interchangeable concepts.
33. Interfaces remain valuable for application abstractions.
34. Dependency injection remains valuable for object composition.
35. Use `expect` / `actual` for genuine platform boundaries.
36. Keep the expected API as small as possible.
37. Prefer capability-based APIs over large platform managers.
38. Avoid leaking Android or iOS types into common APIs.
39. Keep platform implementations close to their platform APIs.
40. Use intermediate source sets when implementations can genuinely be shared.
41. Do not force different platforms to share incompatible implementations.
42. A new target should expose missing platform implementations early.
43. Compile-time detection is safer than relying entirely on runtime branches.
44. Common tests can validate shared behavior.
45. Platform tests can validate platform implementations.
46. Compiler errors can be useful architectural feedback.
47. When debugging, inspect the source-set dependency graph.
48. Verify that the target compilation can see the `actual`.
49. Verify that the `expect` and `actual` declarations correspond.
50. Understand the compilation graph before changing source-set structure.
51. Design APIs that make the compiler's resolution path obvious.
52. **The compiler is not just building the application; it is enforcing the boundary between shared intent and platform implementation.**

---

## Final Thought

The most useful way to understand `expect` / `actual` is to stop thinking about it as a special runtime feature.

Think about it as a **compiler-enforced bridge**:

```text
                 commonMain
                     │
               ┌─────▼─────┐
               │  expect   │
               │ Contract  │
               └─────┬─────┘
                     │
              Target compilation
                     │
          ┌──────────┴──────────┐
          ▼                     ▼
       Android                  iOS
          │                     │
       actual                  actual
          │                     │
          ▼                     ▼
    Android APIs            iOS APIs
```

The common code defines the requirement.

The source-set hierarchy determines visibility.

The compiler resolves the appropriate implementation.

The platform compiler produces the final target-specific artifact.

That is the real power of `expect` / `actual`:

> **The platform difference is resolved before your application runs, while the common architecture remains clean and shared.**
