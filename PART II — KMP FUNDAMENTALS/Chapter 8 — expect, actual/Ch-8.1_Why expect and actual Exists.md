# Chapter 8 — `expect` / `actual`

## Part 1 — Why `expect` / `actual` Exists

> **`expect` / `actual` exists because shared code sometimes needs to ask the platform to do something that only the platform itself knows how to do.**

Kotlin Multiplatform is built around a powerful idea:

```text
Share as much code as possible
without pretending that every platform is identical.
```

Android, iOS, JVM desktop, macOS, WebAssembly, and other platforms have different operating-system APIs, runtime environments, security models, lifecycle concepts, and platform capabilities.

KMP therefore needs a mechanism that allows common code to say:

> "I need this capability."

while allowing each platform to say:

> "Here is how I provide it."

That is the fundamental idea behind:

```kotlin
expect
```

and:

```kotlin
actual
```

---

## 1. The Problem `expect` / `actual` Solves

Imagine a shared KMP application that needs to know the current platform.

The business logic should ideally remain in:

```text
commonMain
```

But the actual implementation may be different:

```text
Android → Android APIs
iOS     → iOS APIs
Desktop → JVM/Desktop APIs
```

The common layer should not contain code such as:

```kotlin
if (android) {
    // Android implementation
} else if (ios) {
    // iOS implementation
}
```

That approach quickly becomes difficult to maintain.

Instead, common code defines what it needs:

```kotlin
expect fun platformName(): String
```

Then platform source sets provide the implementation:

```kotlin
actual fun platformName(): String {
    // Android implementation
}
```

and:

```kotlin
actual fun platformName(): String {
    // iOS implementation
}
```

The shared code knows the **contract**.

The platform knows the **implementation**.

---

## 2. Common Code Cannot Know Everything

One of the biggest misconceptions about KMP is:

> "If code is in `commonMain`, it should be able to do everything."

It cannot.

`commonMain` is intentionally restricted to APIs that are available to all targets that consume that source set.

For example, common code can work with:

```kotlin
String
List
Map
Coroutine abstractions
Shared business logic
Serialization models
Interfaces
Domain models
```

But it cannot directly assume that an Android-only API exists.

For example:

```kotlin
android.content.Context
```

does not belong in ordinary `commonMain` code.

Similarly, an iOS-specific framework API cannot simply be imported into common code.

This is where `expect` / `actual` becomes useful.

---

# 3. Think in Terms of Capability

Instead of asking:

> "How can common code access Android or iOS?"

ask:

> "What capability does common code require?"

For example:

```text
Common code needs:
    Secure storage
```

The platforms may provide:

```text
Android → Android secure storage mechanism
iOS     → Keychain
```

The common layer does not need to know those implementation details.

It only needs a contract:

```text
SecureStorage
```

This separation is one of the most important architectural ideas behind `expect` / `actual`.

---

# 4. The Basic Mental Model

The simplest mental model is:

```text
                    commonMain
                        │
                  ┌─────▼─────┐
                  │   expect  │
                  │  Contract │
                  └─────┬─────┘
                        │
              ┌─────────┴─────────┐
              │                   │
              ▼                   ▼
        androidMain            iosMain
              │                   │
        ┌─────▼─────┐       ┌─────▼─────┐
        │  actual   │       │  actual   │
        │ Android   │       │   iOS     │
        └───────────┘       └───────────┘
```

In one sentence:

> **`expect` describes what is required; `actual` provides how it works on a specific platform.**

---

# 5. `expect` Means "This Is Required"

Consider:

```kotlin
expect fun platformName(): String
```

The word:

```kotlin
expect
```

tells Kotlin:

> A platform-specific implementation is expected to exist for this declaration.

The common source set declares the requirement.

It does not provide the platform implementation itself.

---

# 6. `actual` Means "Here Is the Implementation"

Android might provide:

```kotlin
actual fun platformName(): String {
    return "Android"
}
```

iOS might provide:

```kotlin
actual fun platformName(): String {
    return "iOS"
}
```

Both implementations satisfy the same common declaration.

The relationship is:

```text
expect fun platformName()
        │
        ├── actual → Android
        │
        └── actual → iOS
```

---

# 7. Why Not Just Use Interfaces?

A natural question is:

> "Why do we need `expect` / `actual` when Kotlin already has interfaces?"

You can absolutely use interfaces.

For example:

```kotlin
interface PlatformInfo {
    fun name(): String
}
```

Then platform implementations can implement that interface:

```kotlin
class AndroidPlatformInfo : PlatformInfo {
    override fun name(): String = "Android"
}
```

This is often a very good design.

But interfaces require you to provide and wire an implementation through mechanisms such as:

```text
Dependency Injection
Factories
Constructors
Service locators
Platform composition roots
```

`expect` / `actual` is a language-level mechanism for expressing platform-specific declarations directly in the multiplatform source-set model.

---

# 8. `expect` / `actual` Is Not Dependency Injection

These concepts should not be confused.

### `expect` / `actual`

Defines:

```text
Platform-specific implementation mapping
```

### Dependency Injection

Defines:

```text
How dependencies are provided to objects
```

For example:

```text
expect / actual
        ↓
Creates platform-specific implementation
        ↓
Dependency Injection
        ↓
Provides it to application components
```

They can work together.

---

# 9. A Simple Example

Imagine a shared application wants to display the current platform.

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

Now common code can simply call:

```kotlin
fun greeting(): String {
    return "Running on ${platformName()}"
}
```

The common code does not contain:

```text
Android code
iOS code
Platform checks
```

It only knows about the capability.

---

# 10. The Important Separation

The architecture becomes:

```text
┌───────────────────────────────┐
│          commonMain           │
│                               │
│   Business logic              │
│   Domain logic                │
│   Shared state                │
│   expect declaration          │
└───────────────┬───────────────┘
                │
                │ contract
                ▼
       ┌─────────────────┐
       │ Platform layer  │
       └────────┬────────┘
                │
        ┌───────┴────────┐
        ▼                ▼
   Android actual    iOS actual
```

This maintains a clean boundary between:

```text
What the application needs
```

and:

```text
How a platform provides it
```

---

# 11. Why This Matters in KMP

Without a mechanism like `expect` / `actual`, developers might put platform-specific logic directly into common code.

That can lead to:

```kotlin
if (isAndroid) {
    ...
}

if (isIOS) {
    ...
}

if (isDesktop) {
    ...
}
```

As platforms increase:

```text
Android
iOS
Desktop
macOS
Wasm
```

the number of platform conditions can grow rapidly.

This creates:

```text
Complexity
Reduced readability
Harder testing
Platform leakage
Tight coupling
```

`expect` / `actual` provides a structured alternative.

---

# 12. Platform-Specific Code Should Stay Where It Belongs

A useful rule is:

> **Keep platform-specific implementation in the lowest source set that can correctly share it.**

For example:

```text
commonMain
    ↓
appleMain
    ↓
iosMain
```

If an implementation works for both:

```text
iOS
macOS
```

it may belong in:

```text
appleMain
```

If it only works for iOS:

```text
iosMain
```

This connects `expect` / `actual` directly with the source-set hierarchy discussed in Chapter 7.

---

# 13. `expect` Can Live in `commonMain`

A common pattern is:

```text
commonMain
    └── expect declaration
```

For example:

```kotlin
expect class Platform {
    val name: String
}
```

The platform source sets then provide the corresponding `actual` declaration.

---

# 14. `actual` Can Live at Different Levels

The implementation does not always need to be directly inside:

```text
androidMain
iosMain
```

It can sometimes live in an intermediate source set.

For example:

```text
commonMain
    │
    │ expect
    ▼
appleMain
    │
    │ actual
    ├─────────────┐
    ▼             ▼
 iosMain       macosMain
```

If both Apple targets can use the same implementation, this can eliminate duplication.

---

# 15. Why Not Put Platform APIs in `commonMain`?

Because doing so would destroy one of the primary benefits of multiplatform architecture.

Suppose:

```text
commonMain
```

imports:

```text
android.content.Context
```

Now common code is no longer truly common.

You have effectively created:

```text
Android-dependent shared code
```

The source-set boundary has been violated.

---

# 16. The Platform Leakage Problem

Platform leakage occurs when platform-specific details escape into shared layers.

For example:

```text
commonMain
   ↓
Android Context
   ↓
Business logic
```

Now your business logic depends on Android.

A healthier architecture is:

```text
commonMain
   ↓
Platform capability
   ↓
androidMain
```

The dependency direction remains clean.

---

# 17. `expect` Helps Make Platform Boundaries Explicit

The declaration:

```kotlin
expect fun readSecureValue(key: String): String?
```

makes the boundary visible.

Anyone reading the common code immediately knows:

```text
This operation depends on platform behavior.
```

That is better than hiding platform assumptions inside complex shared code.

---

# 18. Contract vs Implementation

Think of the two keywords as:

```text
expect
   ↓
WHAT

actual
   ↓
HOW
```

For example:

```text
WHAT:
"Give me a secure storage implementation."

HOW on Android:
"Use Android's supported secure storage mechanism."

HOW on iOS:
"Use the appropriate Apple security facility."
```

The common layer should care about:

```text
What it needs.
```

The platform layer should care about:

```text
How to provide it.
```

---

# 19. Why Compile-Time Mapping Is Valuable

Compile-time mapping gives you:

```text
Type checking
Missing implementation detection
Platform-specific compilation
Clear source-set ownership
```

Instead of carrying all possible platform implementations into a runtime decision tree.

---

# 20. `expect` / `actual` Keeps Common Code Honest

When common code declares:

```kotlin
expect fun getDeviceId(): String
```

it acknowledges:

```text
This capability is platform-dependent.
```

The common layer does not pretend that every platform has the same implementation.

KMP allows the boundary to be explicit.

---

# 21. Prefer Capabilities Over Platform Objects

Instead of:

```text
Give me Android Context.
```

prefer:

```text
I need access to application storage.
```

Instead of:

```text
Give me UIViewController.
```

prefer:

```text
I need to perform this platform-specific operation.
```

This leads to better architecture.

---

# 22. When `expect` / `actual` Is a Good Choice

Use it when:

- A capability fundamentally depends on the platform.
- The common API should be known at compile time.
- The implementation is naturally tied to a target.
- Platform API access needs to remain outside `commonMain`.
- A compiler-enforced platform contract is valuable.
- An intermediate source set can provide a shared implementation for multiple targets.

---

# 23. When It May Not Be the Best Choice

Consider alternatives when:

- The abstraction is primarily business logic.
- Multiple implementations are required for testing.
- Runtime implementation selection is important.
- Dependency injection is already the natural composition mechanism.
- A normal Kotlin interface communicates the architecture more clearly.

The presence of a platform boundary does not automatically mean `expect` / `actual` is the best solution.

---

# 24. Common Mistakes

### ❌ Treating `expect` as a runtime platform switch

It is better understood as a compile-time multiplatform declaration mechanism.

### ❌ Putting platform APIs directly into `commonMain`

This breaks portability.

### ❌ Using `expect` for every abstraction

Many business abstractions are better represented with interfaces.

### ❌ Returning platform objects from common APIs

This can leak implementation details upward.

### ❌ Creating huge platform APIs

A platform abstraction should expose the smallest useful capability.

### ❌ Ignoring the source-set hierarchy

The correct `actual` location depends on which targets can safely share the implementation.

### ❌ Duplicating identical actual implementations

If multiple targets can genuinely share the same implementation, consider an appropriate intermediate source set.

### ❌ Hiding platform differences behind conditionals

Repeated platform checks in common code can indicate that platform behavior belongs in `actual` implementations.

---

# 25. A Better Design Pattern

Instead of:

```text
commonMain
    └── PlatformUtils
          ├── if Android
          ├── if iOS
          └── if Desktop
```

prefer:

```text
commonMain
    └── expect PlatformUtils
              │
       ┌──────┼──────┐
       ▼      ▼      ▼
    Android   iOS   Desktop
      actual  actual  actual
```

This makes platform ownership explicit.

---

# 26. The Architectural Flow

A mature KMP architecture may look like:

```text
┌─────────────────────────────┐
│         commonMain          │
│                             │
│ Domain                      │
│ Use Cases                   │
│ Repositories                │
│ State                       │
│                             │
│ expect declarations         │
└──────────────┬──────────────┘
               │
               ▼
      Platform capability
               │
      ┌────────┴─────────┐
      ▼                  ▼
┌──────────────┐   ┌──────────────┐
│ androidMain  │   │   iosMain    │
│              │   │              │
│ actual       │   │ actual       │
│ Android APIs │   │ iOS APIs     │
└──────────────┘   └──────────────┘
```

This is the core pattern.

---

# 27. The Bigger KMP Philosophy

`expect` / `actual` demonstrates an important KMP philosophy:

> **Multiplatform does not mean platform-independent. It means platform-aware without unnecessary duplication.**

The goal is not to erase platforms.

The goal is to create clean boundaries between:

```text
Shared logic
```

and:

```text
Platform capabilities
```

---

# 28. Five Questions to Ask Before Using `expect` / `actual`

Before introducing an `expect` declaration, ask:

### 1. Is this genuinely platform-dependent?

If not, keep it common.

### 2. Can an interface solve the problem more naturally?

If yes, consider an interface.

### 3. What is the smallest capability common code needs?

Expose that rather than a platform object.

### 4. Which source set should own the implementation?

Use the hierarchy to determine this.

### 5. Will the abstraction still make sense if another platform is added?

A good platform abstraction should scale.

---

# 29. Chapter Takeaways

> [!IMPORTANT]
> **`expect` / `actual` is a mechanism for expressing platform-specific requirements without leaking platform implementation details into shared code.**

Remember:

1. `expect` defines a declaration required by common code.
2. `actual` provides the platform-specific implementation.
3. `expect` describes **what** is needed.
4. `actual` describes **how** it is implemented.
5. The mechanism is designed around Kotlin Multiplatform's source-set model.
6. Common code cannot directly depend on arbitrary platform APIs.
7. Platform-specific APIs belong in appropriate platform source sets.
8. `expect` can define a platform capability in common code.
9. `actual` satisfies that capability for a target.
10. The mapping is primarily a compile-time concept.
11. It should not be thought of as runtime platform dispatch.
12. The compiler can help detect missing platform implementations.
13. This becomes especially useful when adding new targets.
14. `expect` / `actual` can work with intermediate source sets.
15. A shared `actual` can sometimes live at a platform-family level.
16. Platform-specific `actual` implementations can live at leaf source sets.
17. Source-set hierarchy determines where implementation can be shared.
18. `expect` / `actual` and source-set hierarchy should be designed together.
19. `expect` / `actual` is not the same as dependency injection.
20. `expect` / `actual` is not the same as an interface.
21. Interfaces remain useful for business abstractions.
22. Dependency injection remains useful for object composition.
23. Use `expect` / `actual` when a platform boundary is naturally expressed through the KMP source-set model.
24. Avoid using `expect` / `actual` for every abstraction.
25. Keep platform APIs out of `commonMain`.
26. Prefer capability-based abstractions.
27. Avoid exposing platform objects unnecessarily to common code.
28. Keep the abstraction as small as possible.
29. Avoid large platform utility classes.
30. Repeated platform checks in common code can indicate poor separation.
31. `expect` / `actual` can reduce platform conditionals.
32. `expect` / `actual` can make platform boundaries explicit.
33. Shared business logic can remain independent of platform APIs.
34. Repositories can depend on platform capabilities rather than platform classes.
35. Use cases can remain common while platform services remain platform-specific.
36. MVI state-management logic can remain common while platform services are isolated.
37. Clean Architecture can use `expect` / `actual` as a platform boundary.
38. Shared tests can validate common behavior.
39. Platform tests can validate actual implementations.
40. `expect` / `actual` does not eliminate the need for platform-specific testing.
41. A good abstraction should remain understandable when new targets are added.
42. New targets should make missing platform responsibilities visible.
43. Platform-family implementations can reduce duplication.
44. Do not force platform sharing when implementations are genuinely different.
45. Do not confuse similar platforms with identical capabilities.
46. The source-set hierarchy should determine the correct sharing level.
47. The common layer should request capabilities, not implementation details.
48. Platform implementations should remain close to the APIs they use.
49. `expect` / `actual` helps preserve a clean boundary between shared and platform-specific code.
50. The goal is not to eliminate platform differences.
51. The goal is to isolate platform differences.
52. **KMP succeeds when common code owns the business intent and platform code owns the platform reality.**

---

## Final Thought

The most important idea behind `expect` / `actual` is not the syntax.

It is the architectural boundary.

```text
                    COMMON CODE
                         │
                  "I need this."
                         │
                         ▼
                    ┌────────┐
                    │ expect │
                    └────┬───┘
                         │
             ┌───────────┴───────────┐
             │                       │
             ▼                       ▼
        Android                    iOS
             │                       │
          actual                  actual
             │                       │
             ▼                       ▼
       Android APIs             iOS APIs
```

The common layer does not need to know every platform's implementation.

It only needs a clear contract.

The platform provides the implementation.

That simple separation is what allows KMP to share substantial application logic while still respecting the reality that **Android is not iOS, iOS is not desktop, and no platform should be forced to pretend otherwise.**
