# Chapter 3 — Android vs Flutter vs React Native vs Compose vs KMP

## Part 6 — Kotlin Multiplatform

> **Kotlin Multiplatform does not ask you to abandon native development. It asks you to stop duplicating the parts of the application that do not need to be duplicated.**

After looking at Native Android, Flutter, React Native, and Compose Multiplatform, we finally arrive at Kotlin Multiplatform.

This is where the comparison becomes more interesting.

Kotlin Multiplatform is not simply another cross-platform UI framework.

Its fundamental idea is different:

```text
                Kotlin Multiplatform
                         │
             ┌───────────┴───────────┐
             ▼                       ▼
       Shared Code              Native Code
             │                       │
             ▼                       ▼
      Common Business          Platform-specific
          Logic                    APIs
```

The team decides **what should be shared**.

That could be:

- One small utility
- A validation module
- Domain models
- Business rules
- Networking
- Database access
- Caching
- State management
- Most of the application
- UI, when Compose Multiplatform is used

This flexibility is the defining characteristic of KMP.

---

# What Is Kotlin Multiplatform?

Kotlin Multiplatform is a technology from JetBrains that allows Kotlin code to be shared across multiple platforms.

The core technology became Stable in November 2023. Current official documentation lists Android, iOS, and JVM desktop as Stable targets for Kotlin Multiplatform, with additional targets having their own maturity levels.

At its simplest:

```text
                    Kotlin
                       │
                       ▼
                Shared Module
                       │
             ┌─────────┴─────────┐
             ▼                   ▼
          Android                iOS
             │                   │
             ▼                   ▼
        Native App           Native App
```

The important part is what happens **inside** the shared module.

The shared module doesn't have to contain the entire application.

It contains the code that genuinely benefits from being shared.

---

# KMP Is Not "One App Running Everywhere"

This is probably the first misconception we need to remove.

KMP does not mean:

```text
Write One App
      │
      ▼
Run Everywhere
```

Instead, the mental model is:

```text
             Product
                │
       ┌────────┴────────┐
       ▼                 ▼
    Android              iOS
       │                 │
       └────────┬────────┘
                ▼
          Shared Logic
```

Android remains an Android application.

iOS remains an iOS application.

The common parts are implemented once in Kotlin and reused.

That distinction is extremely important.

---

# The Core Idea: Selective Code Sharing

Imagine an application with the following layers:

```text
┌──────────────────────────────┐
│             UI               │
├──────────────────────────────┤
│       Presentation           │
├──────────────────────────────┤
│       Business Logic         │
├──────────────────────────────┤
│       Data / Repository      │
├──────────────────────────────┤
│       Networking             │
├──────────────────────────────┤
│       Platform APIs          │
└──────────────────────────────┘
```

KMP allows the team to decide which of these layers should be shared.

For example:

```text
UI                  → Native
Presentation        → Shared
Business Logic      → Shared
Data                → Shared
Networking          → Shared
Platform APIs       → Native
```

Or:

```text
UI                  → Shared
Presentation        → Shared
Business Logic      → Shared
Data                → Shared
Networking          → Shared
Platform APIs       → Native
```

There is no requirement that both architectures look the same.

---

# The KMP Spectrum

It is useful to think about KMP as a spectrum rather than a binary decision.

```text
Less Sharing                                      More Sharing
     │                                                   │
     ▼                                                   ▼

  One Module     Business Logic     Full Logic      Logic + UI
     │                │                │                │
     ▼                ▼                ▼                ▼
   Small           Domain           Most App       Compose MP
   Utility          Layer            Code             UI
```

This is one of KMP's strongest architectural characteristics.

Official Kotlin documentation explicitly describes KMP as allowing teams to share everything from a small piece of logic to most of an application, while keeping platform-specific code where necessary.

---

# The Most Important Question

When adopting KMP, don't begin with:

> "How much code can we share?"

Begin with:

> **"Which code has the same business meaning on every platform?"**

That changes the entire conversation.

For example:

```text
Discount Calculation
        │
        ▼
Same Business Rule
        │
        ▼
       KMP
```

But:

```text
Android Back Gesture
        │
        ▼
Android-specific behavior
        │
        ▼
Android
```

The first belongs naturally in shared code.

The second does not.

---

# Common Code and Platform Code

A typical KMP project contains common and platform-specific source sets.

A simplified structure looks like:

```text
shared/
│
├── src/
│   │
│   ├── commonMain/
│   │
│   ├── androidMain/
│   │
│   └── iosMain/
│
└── build.gradle.kts
```

`commonMain` contains code that can be shared across the relevant platforms.

Platform source sets contain code that requires platform-specific APIs or behavior.

This source-set model is a fundamental part of Kotlin Multiplatform.

---

# Figure 3.4 — The KMP Source-Set Model

```text
                       shared
                         │
                  ┌──────┴──────┐
                  ▼             ▼
             commonMain
                  │
        ┌─────────┴─────────┐
        ▼                   ▼
   androidMain           iosMain
        │                   │
        ▼                   ▼
   Android APIs          iOS APIs
```

The common source set is the shared foundation.

Platform source sets provide the parts that cannot—or should not—be common.

---

# A Simple Business Logic Example

Consider a shopping application.

The discount rule is:

```text
If cart total > ₹5,000
apply 10% discount.
```

This rule has nothing inherently Android-specific or iOS-specific.

So it can live in shared code:

```kotlin
class DiscountCalculator {

    fun calculate(total: Double): Double {
        return if (total > 5000) {
            total * 0.10
        } else {
            0.0
        }
    }
}
```

Then:

```text
Android
   │
   ├── Android UI
   │
   └── Shared DiscountCalculator
```

and:

```text
iOS
   │
   ├── iOS UI
   │
   └── Shared DiscountCalculator
```

There is one business rule.

There are two native applications.

That is the KMP model in its simplest form.

---

# Why This Matters

Without sharing:

```text
Android
   │
   └── DiscountCalculator

iOS
   │
   └── DiscountCalculator
```

Two implementations.

Two test suites.

Two opportunities for divergence.

With KMP:

```text
                 DiscountCalculator
                         │
                ┌────────┴────────┐
                ▼                 ▼
             Android             iOS
```

One implementation.

One business rule.

The platform UI remains independent.

---

# KMP and Native UI

This is where KMP differs significantly from frameworks that primarily promote a shared UI approach.

You can use:

```text
Android → Jetpack Compose
iOS     → SwiftUI
```

while sharing the underlying business logic.

For example:

```text
              Android UI
                  │
                  ▼
         ┌────────────────┐
         │                │
         │   Shared KMP   │
         │     Logic      │
         │                │
         └────────────────┘
                  ▲
                  │
              iOS UI
```

This means a team can maintain a genuinely platform-specific user experience while still eliminating duplicated business logic.

---

# KMP Does Not Require Compose Multiplatform

This deserves repeating because it is one of the most important concepts in this book.

You can use:

```text
Kotlin Multiplatform
+
Jetpack Compose
+
SwiftUI
```

without using Compose Multiplatform.

For example:

```text
Android
┌───────────────────┐
│ Jetpack Compose   │
└─────────┬─────────┘
          │
          ▼
     Shared KMP
          ▲
          │
┌─────────┴─────────┐
│      SwiftUI      │
└───────────────────┘
iOS
```

Compose Multiplatform is an optional UI framework within the KMP ecosystem.

KMP itself is broader than shared UI.

---

# KMP + Compose Multiplatform

If the team decides that sharing UI provides value, Compose Multiplatform can be added.

Then the architecture becomes:

```text
                KMP
                 │
       ┌─────────┴─────────┐
       ▼                   ▼
 Shared Logic        Compose Multiplatform
                           │
                           ▼
                    Shared UI
```

The resulting application may look like:

```text
                 Shared Module
                       │
          ┌────────────┴────────────┐
          ▼                         ▼
   Compose Multiplatform      Shared Business Logic
          │                         │
          └────────────┬────────────┘
                       ▼
              Android + iOS
```

This is an optional extension—not a requirement.

---

# Platform-Specific APIs

Real applications cannot avoid platform APIs.

Consider:

```text
Android BiometricPrompt
iOS LocalAuthentication
```

These APIs solve similar problems.

But they are not the same API.

KMP provides mechanisms for keeping a common interface while supplying platform-specific implementations.

One mechanism is `expect` / `actual`.

The common code defines the expected API:

```kotlin
expect fun authenticateUser(): Boolean
```

Android provides its implementation:

```kotlin
actual fun authenticateUser(): Boolean {
    // Android implementation
}
```

iOS provides another:

```kotlin
actual fun authenticateUser(): Boolean {
    // iOS implementation
}
```

The common code can work with the shared abstraction while each platform uses its own implementation.

This is the purpose of the `expect` / `actual` mechanism described in the official Kotlin documentation.

---

# Figure 3.5 — Common API, Native Implementation

```text
                 commonMain

          expect authenticateUser()
                     │
             ┌───────┴───────┐
             ▼               ▼
        androidMain       iosMain
             │               │
             ▼               ▼
       Android API        iOS API
```

The shared layer defines the contract.

The platforms provide the implementation.

---

# KMP Is Not About Hiding Platforms

This is another important architectural distinction.

A poor multiplatform architecture might try to pretend:

```text
Android = iOS
```

They are not.

A better architecture acknowledges:

```text
Android ≠ iOS
```

while identifying:

```text
Business Rules(Android)
       =
Business Rules(iOS)
```

That distinction is the heart of good KMP architecture.

---

# KMP and Direct Platform Access

One of KMP's important characteristics is that shared Kotlin code can interoperate with platform-specific functionality through the appropriate platform mechanisms.

This means the architecture does not need to rely entirely on a giant abstraction layer that hides every platform API.

Conceptually:

```text
              Shared Kotlin
                    │
        ┌───────────┴───────────┐
        ▼                       ▼
 Android Platform          iOS Platform
        │                       │
        ▼                       ▼
 Android SDK               Apple APIs
```

This allows platform-specific code to remain genuinely platform-specific.

---

# KMP and Native Performance

KMP's model is different from a framework that requires an application-wide runtime bridge for all shared logic.

Kotlin Multiplatform compiles Kotlin code for the target platforms. Current Kotlin documentation describes KMP as enabling shared code while retaining native capabilities and performance characteristics.

That doesn't mean:

> "Every KMP application is automatically faster."

Performance still depends on the application.

The important architectural point is that KMP does not require the team to sacrifice native platform execution simply to share business logic.

---

# KMP and Testing

Shared business logic creates another significant advantage.

Imagine:

```text
Authentication
Validation
Pricing
Cart
Checkout
Sync
Caching
```

If these rules are shared, they can be tested at the shared layer.

```text
             Shared Tests
                  │
          ┌───────┴───────┐
          ▼               ▼
       Android            iOS
```

Platform-specific behavior still requires platform-specific tests.

But identical business rules don't need completely independent implementations just because the UI platforms are different.

---

# Example: Authentication

Consider an authentication flow.

The business requirements might be:

```text
1. Validate email
2. Validate password
3. Call authentication API
4. Store session
5. Refresh token
6. Handle expiration
```

These rules are largely platform-independent.

The UI is not.

Android might implement:

```text
Jetpack Compose
       │
       ▼
Android ViewModel
       │
       ▼
Shared Authentication
```

iOS might implement:

```text
SwiftUI
   │
   ▼
iOS Presentation
   │
   ▼
Shared Authentication
```

The architecture becomes:

```text
                Authentication
                      │
             ┌────────┴────────┐
             ▼                 ▼
         Android UI         iOS UI
```

The business behavior stays aligned.

---

# KMP and Offline-First Applications

KMP becomes particularly interesting when applications contain significant data and synchronization logic.

Consider:

```text
API
 │
 ▼
Repository
 │
 ├── Cache
 │
 ├── Database
 │
 └── Sync
```

If the synchronization rules are the same across platforms, implementing them twice creates unnecessary risk.

KMP can allow:

```text
               Shared Data Layer
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
       Network       Cache        Database
          │
          ▼
       Android + iOS
```

The platform-specific storage implementation can remain separate when necessary.

The synchronization policy itself can be shared.

---

# KMP and Existing Android Applications

This is where KMP can become especially valuable.

Suppose the organization already has:

```text
Large Android Application
        │
        ├── Kotlin
        ├── Jetpack Compose
        ├── MVVM / MVI
        ├── Repository
        ├── Room
        └── Android APIs
```

The organization doesn't necessarily need to rewrite the application.

Instead:

```text
Existing Android App
        │
        ▼
Identify Shared Logic
        │
        ▼
Extract KMP Module
        │
        ▼
Move Selected Logic
        │
        ▼
Introduce iOS
        │
        ▼
Expand Sharing Gradually
```

Official KMP guidance explicitly supports incremental adoption, including migrating an existing Android application and starting with a small piece of shared logic.

---

# Incremental Adoption

A realistic migration may look like:

### Phase 1

```text
Android
  │
  └── KMP
       └── Network Models
```

### Phase 2

```text
KMP
├── Models
├── Networking
└── Validation
```

### Phase 3

```text
KMP
├── Domain
├── Data
├── Networking
├── Database
└── Business Rules
```

### Phase 4

```text
KMP
├── Domain
├── Data
├── State
├── Business Rules
└── Optional Shared UI
```

The organization can stop at any point.

That is a major architectural advantage.

---

# KMP and Team Structure

KMP also changes how teams can think about ownership.

Traditional organization:

```text
Android Team
      │
      ▼
Android Implementation

iOS Team
      │
      ▼
iOS Implementation
```

A KMP organization can introduce:

```text
               Shared Domain
                    │
             ┌──────┴──────┐
             ▼             ▼
        Android Team    iOS Team
             │             │
          Native UI      Native UI
```

The teams can collaborate around shared business behavior while retaining platform expertise.

This can be particularly useful for organizations where Android and iOS need to remain independently productive.

---

# KMP Does Not Eliminate iOS Development

This is worth stating clearly.

A common misconception is:

> "If we use KMP, we don't need iOS developers."

That is not a sound assumption.

A production iOS application still requires knowledge of:

- iOS application lifecycle
- Xcode
- Apple APIs
- Swift / SwiftUI where applicable
- Signing
- App Store distribution
- Platform conventions
- Accessibility
- Platform-specific behavior

KMP reduces duplication.

It does not eliminate the platform.

---

# KMP vs Flutter

The architectural difference can be summarized simply.

### Flutter

```text
Dart
 │
 ▼
Flutter
 │
 ├── Shared UI
 └── Shared Application
```

### KMP

```text
Kotlin
 │
 ▼
Shared Code
 │
 ├── Business Logic
 ├── Data
 ├── Networking
 └── Optional UI
```

Flutter generally encourages a more unified application stack.

KMP gives the team more control over where the boundary is drawn.

Neither model is universally better.

---

# KMP vs React Native

React Native:

```text
JavaScript / TypeScript
          │
          ▼
     React Native
          │
          ▼
Native Platform Integration
```

KMP:

```text
Kotlin
  │
  ▼
Shared Kotlin
  │
  ├── Common Code
  │
  └── Platform Code
```

For organizations already deeply invested in Kotlin and Android, KMP can reuse existing language and architectural knowledge.

For organizations with a strong React and TypeScript ecosystem, React Native may be the more natural fit.

---

# KMP vs Compose Multiplatform

This comparison is critical.

```text
             Kotlin Multiplatform
                     │
          ┌──────────┴──────────┐
          ▼                     ▼
    Shared Logic          Compose Multiplatform
                                │
                                ▼
                           Shared UI
```

KMP is the broader technology.

Compose Multiplatform is one way to share UI within that ecosystem.

Therefore:

```text
KMP without Compose MP
        ✅

KMP + Compose MP
        ✅

Compose MP without KMP
        ❌
```

Conceptually, Compose Multiplatform sits on top of KMP. Official Kotlin documentation makes the same distinction.

---

# KMP vs Native Android

This comparison reveals the real purpose of KMP.

Native Android:

```text
Kotlin
  │
  ▼
Android
```

KMP:

```text
Kotlin
  │
  ▼
Shared Code
  │
 ┌┴──────────────┐
 ▼               ▼
Android          iOS
```

If Android is the only target, KMP may add unnecessary complexity.

If Android and iOS share substantial business behavior, the calculation changes.

The question becomes:

> **How much duplicated business logic are we willing to maintain?**

---

# The KMP Architectural Trade-Off

KMP makes a deliberate trade:

```text
                 Selective Sharing
                        │
                        ▼
              Less Business Duplication
                        │
                        ▼
             Native Platform Freedom
                        │
                        ▼
              More Architectural Design
```

That last point matters.

KMP gives you flexibility.

But flexibility requires discipline.

The team must decide:

- What belongs in `commonMain`?
- What belongs in `androidMain`?
- What belongs in `iosMain`?
- What should be an interface?
- What should use platform implementations?
- What should remain completely native?

KMP doesn't answer these questions automatically.

Architecture still belongs to the engineering team.

---

# The Biggest KMP Advantage

The strongest argument for KMP is not:

> "Write Kotlin instead of Swift."

It is:

> **"Keep one source of truth for business behavior that is genuinely common across platforms."**

Consider:

```text
Business Requirement
        │
        ▼
Single Implementation
        │
   ┌────┴────┐
   ▼         ▼
Android     iOS
```

That can reduce:

- Duplicate implementation
- Behavioral drift
- Duplicate business tests
- Synchronization work
- Feature parity problems

The value grows as the shared business domain becomes more complex.

---

# The Biggest KMP Challenge

KMP's flexibility can also become its weakness if the architecture isn't clear.

A poorly designed project can become:

```text
commonMain
   │
   ├── Android logic
   ├── iOS logic
   ├── UI logic
   ├── Platform hacks
   └── Everything else
```

That defeats the purpose.

A good KMP architecture should make the boundary obvious:

```text
                Common
                  │
        ┌─────────┴─────────┐
        ▼                   ▼
  Platform-Independent   Platform-Specific
        │                   │
        ▼                   ▼
 Business Rules          Native APIs
 Domain Models            Native UI
 Shared Data              OS Integration
```

The objective is not maximum common code.

The objective is **meaningful common code**.

---

# KMP Scorecard

Using the evaluation criteria from Part 1:

| Dimension | Kotlin Multiplatform |
|-----------|----------------------|
| Kotlin Ecosystem | ⭐⭐⭐⭐⭐ |
| Android Integration | ⭐⭐⭐⭐⭐ |
| iOS Integration | ⭐⭐⭐⭐⭐ |
| Native UI Support | ⭐⭐⭐⭐⭐ |
| Selective Code Sharing | ⭐⭐⭐⭐⭐ |
| Shared Business Logic | ⭐⭐⭐⭐⭐ |
| Shared UI | ⭐⭐⭐⭐* |
| Platform API Access | ⭐⭐⭐⭐⭐ |
| Incremental Adoption | ⭐⭐⭐⭐⭐ |
| Existing Android Migration | ⭐⭐⭐⭐⭐ |
| Architectural Flexibility | ⭐⭐⭐⭐⭐ |
| Cross-Platform Development | ⭐⭐⭐⭐⭐ |

`*` Shared UI depends on Compose Multiplatform rather than KMP core alone.

These are architectural ratings, not benchmark results.

---

# The Five Approaches So Far

We can now see the philosophical differences.

```text
┌─────────────────────┬───────────────────────────────────────┐
│ Approach            │ Primary Philosophy                    │
├─────────────────────┼───────────────────────────────────────┤
│ Native Android      │ Maximum Android control               │
│ Flutter             │ Share the application and UI          │
│ React Native        │ Share React-based application model   │
│ Compose MP          │ Share Kotlin Compose UI               │
│ KMP                 │ Share what is genuinely common        │
└─────────────────────┴───────────────────────────────────────┘
```

This is the comparison that matters.

Not:

> "Which framework is the winner?"

But:

> **"Which architectural philosophy matches our product?"**

---

# The KMP Sweet Spot

For many organizations, the most interesting architecture may look like this:

```text
                 Android                         iOS
                    │                             │
                    ▼                             ▼
              Native UI                     Native UI
                    │                             │
                    └──────────────┬──────────────┘
                                   ▼
                           Shared KMP Layer
                                   │
                     ┌─────────────┼─────────────┐
                     ▼             ▼             ▼
                  Domain        Data          Network
                     │             │             │
                     └─────────────┴─────────────┘
                                   │
                                   ▼
                            Shared Business
                                Rules
```

The UI remains native.

The business behavior becomes shared.

This architecture often provides a useful balance:

```text
Native Experience
        +
Shared Business Logic
        =
Reduced Duplication
```

---

# But KMP Can Go Further

If the product benefits from shared UI, we can extend the architecture:

```text
                 Shared KMP Module
                        │
          ┌─────────────┴─────────────┐
          ▼                           ▼
   Compose Multiplatform       Shared Business Logic
          │                           │
          └─────────────┬─────────────┘
                        ▼
                   Android + iOS
```

Or we can keep only selected screens shared.

Or only selected components.

Or none of the UI.

That is the point.

---

# The Decision Is Not Binary

KMP gives us a continuum:

```text
                    KMP
                     │
      ┌──────────────┼──────────────┐
      ▼              ▼              ▼
  Small Module    Shared Logic    Shared UI
      │              │              │
      ▼              ▼              ▼
   Low Sharing     Medium         High Sharing
```

This makes adoption reversible.

If sharing more code creates problems:

```text
Share Less
```

If sharing provides more value:

```text
Share More
```

The architecture can evolve with the product.

---

# 🧠 The Real KMP Philosophy

Kotlin Multiplatform can be summarized with one principle:

> **Share the code that represents the product, not the code that merely happens to compile on multiple platforms.**

That distinction is subtle.

Consider:

```text
Price Calculation
```

This represents the product.

Share it.

Now consider:

```text
Android Back Gesture
```

This represents the platform.

Keep it native.

The architecture becomes clearer when we think in terms of **responsibility** rather than **technology**.

---

# Key Takeaways

> [!TIP]
> **KMP is not about replacing native development. It is about reducing unnecessary duplication while preserving native capabilities.**

The most important points are:

- Kotlin Multiplatform enables Kotlin code sharing across multiple platforms.
- KMP became Stable in November 2023.
- Android and iOS are Stable targets for the core KMP technology.
- KMP allows selective code sharing rather than requiring an all-or-nothing architecture.
- Business logic, domain models, networking, data access, caching, and other common logic are natural candidates for sharing.
- Platform-specific APIs remain accessible through platform-specific source sets and mechanisms such as `expect` / `actual`.
- Native Android and iOS UI can remain completely separate.
- Compose Multiplatform can optionally be added when sharing UI provides value.
- KMP supports incremental adoption, including migration from existing Android applications.
- KMP does not eliminate the need for native platform knowledge.
- The goal is not maximum code sharing.
- The goal is **maximum architectural value from the code that is shared**.

---

# The Final Mental Model

After everything we've examined in this chapter, keep this picture in mind:

```text
                       PRODUCT
                          │
                          ▼
                 ┌─────────────────┐
                 │ What is common? │
                 └────────┬────────┘
                          │
              ┌───────────┴───────────┐
              ▼                       ▼
        Shared Responsibility    Platform Responsibility
              │                       │
              ▼                       ▼
             KMP              Android / iOS Native
              │                       │
              └───────────┬───────────┘
                          ▼
                    Final Product
```

And if shared UI is valuable:

```text
                       PRODUCT
                          │
                          ▼
                    Kotlin / KMP
                          │
            ┌─────────────┴─────────────┐
            ▼                           ▼
       Shared Logic             Compose Multiplatform
            │                           │
            └─────────────┬─────────────┘
                          ▼
                    Android + iOS
```

This is the architectural difference that matters most.

---

## Chapter 3 — The Comparison in One View

| Technology | Main Idea | UI Sharing | Logic Sharing | Native Freedom |
|------------|-----------|:----------:|:-------------:|:--------------:|
| **Native Android** | Build specifically for Android | ❌ | ❌ | ⭐⭐⭐⭐⭐ |
| **Flutter** | Share application + rendering | ✅ | ✅ | ⭐⭐⭐ |
| **React Native** | Share React application model | ✅ | ✅ | ⭐⭐⭐⭐ |
| **Compose Multiplatform** | Share Compose UI with Kotlin | ✅ | ✅ | ⭐⭐⭐⭐ |
| **Kotlin Multiplatform** | Share selected code | Optional | ✅ | ⭐⭐⭐⭐⭐ |

The table is useful.

But the deeper conclusion is more important:

> **The most powerful cross-platform architecture may not be the one that shares the most code. It may be the one that gives you the freedom to decide what should be shared.**

That is where Kotlin Multiplatform stands apart.

---

## Closing Thought

The mobile industry spent years trying to answer one question:

> **"How can we write one application for multiple platforms?"**

Kotlin Multiplatform reframes that question:

> **"Which parts of the application should actually be the same?"**

That is a much more useful engineering question.

And once you start thinking this way, KMP stops looking like another cross-platform framework.

It starts looking like an **architectural tool for controlling duplication**.

---

### Official References

- [Kotlin Multiplatform Documentation](https://kotlinlang.org/docs/multiplatform/)
- [Kotlin Multiplatform Supported Platforms](https://kotlinlang.org/docs/multiplatform/supported-platforms.html)
- [Share Code Across Platforms](https://kotlinlang.org/docs/multiplatform/multiplatform-share-on-platforms.html)
- [Kotlin Multiplatform Project Structure](https://kotlinlang.org/docs/multiplatform/multiplatform-discover-project.html)
- [Expected and Actual Declarations](https://kotlinlang.org/docs/multiplatform/multiplatform-expect-actual.html)
````
