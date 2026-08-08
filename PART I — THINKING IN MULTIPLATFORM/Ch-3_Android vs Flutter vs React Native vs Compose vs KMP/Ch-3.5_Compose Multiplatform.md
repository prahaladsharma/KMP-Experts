# Chapter 3 — Android vs Flutter vs React Native vs Compose vs KMP

## Part 5 — Compose Multiplatform

> **Compose Multiplatform asks a very different question: what if the declarative UI model Android developers already know could move beyond Android?**

For an Android developer, Compose Multiplatform can feel immediately familiar.

The syntax looks familiar.

The declarative programming model feels familiar.

The concepts of `@Composable`, `Modifier`, state, recomposition, layouts, and Material components are familiar.

But the target is no longer only Android.

Compose Multiplatform extends the Compose UI model to additional platforms, including iOS and desktop. Its web support is also evolving separately.

That makes Compose Multiplatform particularly interesting in our comparison.

Flutter starts with its own cross-platform UI ecosystem.

React Native starts from React and JavaScript/TypeScript.

Compose Multiplatform starts from **Kotlin and the Compose programming model**.

---

# What Is Compose Multiplatform?

Compose Multiplatform is a declarative UI framework developed by JetBrains that extends the Compose approach beyond Android.

A simplified architecture looks like this:

```text
                    Kotlin
                       │
                       ▼
             Compose Multiplatform
                       │
              ┌────────┼────────┐
              ▼        ▼        ▼
           Android    iOS     Desktop
              │        │        │
              ▼        ▼        ▼
          Platform  Platform  Platform
```

On Android, Compose Multiplatform builds on Jetpack Compose.

On other supported platforms, Compose Multiplatform provides the corresponding implementation of the Compose APIs.

This creates an important distinction:

> **Compose Multiplatform is a UI technology built on top of Kotlin Multiplatform.**

It is not synonymous with Kotlin Multiplatform itself.

---

# KMP and Compose Multiplatform Are Not the Same Thing

This distinction is fundamental.

Think of the relationship like this:

```text
                 Kotlin Multiplatform
                         │
          ┌──────────────┴──────────────┐
          │                             │
          ▼                             ▼
   Shared Business Logic         Compose Multiplatform
                                      │
                                      ▼
                                Shared UI
```

Kotlin Multiplatform can exist without shared UI.

You can build:

```text
Android UI ───────┐
                  │
             Shared KMP Logic
                  │
iOS UI ───────────┘
```

Or you can choose to share the UI as well:

```text
             Compose Multiplatform UI
                       │
              ┌────────┼────────┐
              ▼        ▼        ▼
           Android    iOS     Desktop
```

The choice is architectural.

The framework doesn't force you to share everything.

---

# The Compose Familiarity Advantage

For Android developers already using Jetpack Compose, one of the biggest attractions is the learning curve.

Consider a simple Compose function:

```kotlin
@Composable
fun Greeting(name: String) {
    Text(
        text = "Hello, $name"
    )
}
```

The mental model remains declarative:

```text
State
  │
  ▼
Composable
  │
  ▼
UI
```

Instead of learning an entirely new UI paradigm, an Android developer can extend an existing skill set into a multiplatform environment.

That can be particularly valuable for teams that already have significant Compose expertise.

---

# From Android Compose to Multiplatform Compose

Jetpack Compose was designed for Android.

Compose Multiplatform takes the broader Compose programming model and makes it available across additional targets.

The relationship can be visualized as:

```text
             Jetpack Compose
                    │
                    │ Compose foundation
                    ▼
          Compose Multiplatform
                    │
       ┌────────────┼────────────┐
       ▼            ▼            ▼
    Android         iOS        Desktop
```

Official documentation describes Compose Multiplatform as extending Google's Jetpack Compose toolkit to additional platforms.

This is why the technology can feel so natural to experienced Android Compose developers.

---

# A Shared UI Example

A simple multiplatform UI can live in common code.

Conceptually:

```text
shared/
└── src/
    └── commonMain/
        └── kotlin/
            └── App.kt
```

The shared UI can then be used by multiple platform applications.

```text
                 commonMain
                     │
                     ▼
              Compose UI
                     │
         ┌───────────┼───────────┐
         ▼           ▼           ▼
      Android       iOS       Desktop
```

Current KMP documentation demonstrates shared Compose UI running across Android, iOS, desktop, and web targets, with the appropriate target-specific entry points.

---

# The Architecture Behind It

A typical project may look like:

```text
KMP Project
│
├── shared
│   ├── commonMain
│   │   ├── UI
│   │   ├── Domain
│   │   ├── Data
│   │   └── Business Logic
│   │
│   ├── androidMain
│   └── iosMain
│
├── androidApp
│
├── iosApp
│
└── desktopApp
```

The exact structure varies by project.

The important architectural concept is that **common code and platform-specific code coexist**.

The official KMP documentation describes source sets such as `commonMain`, `androidMain`, and `iosMain` as logical groups of code targeting different platforms.

---

# Compose Multiplatform Does Not Remove Native Entry Points

This is another important point.

A multiplatform UI does not mean the operating systems disappear.

Each platform still has an entry point.

For example:

```text
Android
   │
   ▼
Activity
   │
   ▼
Common Compose UI
```

```text
iOS
   │
   ▼
@main / App Entry
   │
   ▼
Common Compose UI
```

```text
Desktop
   │
   ▼
main()
   │
   ▼
Common Compose UI
```

The platform-specific application starts the application.

The shared UI then becomes part of that platform application.

This preserves the platform boundary instead of pretending that it doesn't exist.

---

# The Big Advantage: Kotlin Everywhere

For an Android team, the technology stack can become remarkably consistent.

```text
                    Kotlin
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
       Android        iOS        Desktop
          │            │            │
       Compose       Compose      Compose
```

Instead of:

```text
Android → Kotlin
iOS     → Swift
```

the team can potentially share Kotlin-based UI and application code.

This doesn't eliminate the need for Swift or Objective-C knowledge.

Platform-specific integration can still require native iOS code.

But it changes how much of the product needs to be implemented separately.

---

# Compose Multiplatform and Native APIs

Shared UI doesn't mean that every API is automatically multiplatform.

A feature may require:

```text
Camera
Bluetooth
Location
Biometrics
Push Notifications
Apple APIs
Android APIs
```

Some APIs have multiplatform equivalents.

Others remain platform-specific.

The official documentation explicitly notes that platform-specific APIs may need to be implemented in platform-specific source sets.

The architecture therefore remains:

```text
              Shared Compose UI
                     │
             Shared Application Logic
                     │
             ┌───────┴────────┐
             ▼                ▼
       Android APIs        iOS APIs
```

This is an important escape hatch.

---

# Platform-Specific UI Is Still Possible

Sharing UI doesn't mean every screen has to be identical.

A project can still introduce platform-specific behavior.

Conceptually:

```text
                   Shared Screen
                         │
              ┌──────────┴──────────┐
              ▼                     ▼
        Android-specific       iOS-specific
             UI                    UI
```

This gives teams the ability to share the majority of a feature while customizing the parts where platform differences genuinely matter.

That flexibility is one of the reasons Compose Multiplatform fits naturally into a broader KMP architecture.

---

# Shared UI vs Native UI

At this point, we can visualize three possible architectures.

### Option 1 — Native UI + Shared KMP Logic

```text
 Android UI                 iOS UI
     │                         │
     └──────────┬──────────────┘
                ▼
        Shared KMP Logic
```

### Option 2 — Shared Compose UI + Shared Logic

```text
          Compose Multiplatform
                   │
                   ▼
            Shared UI + Logic
                   │
          ┌────────┴────────┐
          ▼                 ▼
       Android              iOS
```

### Option 3 — Hybrid

```text
             Shared KMP Core
                    │
          ┌─────────┴─────────┐
          ▼                   ▼
   Shared Compose UI     Native UI
          │                   │
       Android               iOS
```

This is where Compose Multiplatform becomes particularly interesting.

It doesn't force one architecture.

It gives the team another architectural option.

---

# Compose Multiplatform and Native Experience

A common question is:

> **"If the UI is shared, will the application still feel native?"**

The answer requires nuance.

A shared UI can behave consistently across platforms, but platform conventions are not identical.

For example:

```text
Android
├── Back navigation
├── Material conventions
├── Android system behavior
└── Android accessibility

iOS
├── Navigation conventions
├── Gesture behavior
├── Apple conventions
└── iOS accessibility
```

Compose Multiplatform provides platform-specific integrations and continues to evolve its platform behavior.

But a team should still evaluate whether a completely shared UI is appropriate for the product.

The framework provides the capability.

The product determines whether that capability should be used.

---

# Compose Multiplatform and iOS

For Android developers, iOS is often the most important part of the Compose Multiplatform story.

Compose Multiplatform's iOS target reached Stable status with version 1.8.0 in May 2025, marking a significant milestone for production use of shared Compose UI on iOS.

Since then, the framework has continued to evolve.

For example, later releases have added improvements to iOS behavior, tooling, previews, navigation, and text input.

This matters because cross-platform UI is not a one-time feature.

It is an evolving platform.

---

# Compose Multiplatform and Desktop

Desktop is another important target.

The same Compose programming model can be used for applications running on JVM desktop targets.

Conceptually:

```text
                 Common Compose UI
                         │
              ┌──────────┼──────────┐
              ▼          ▼          ▼
           Android      iOS      Desktop
```

This expands the architectural possibilities beyond mobile.

A product that begins as:

```text
Android + iOS
```

can potentially evolve toward:

```text
Android + iOS + Desktop
```

without creating an entirely separate UI technology stack.

---

# Compose Multiplatform and Web

Web support exists as part of the broader Compose Multiplatform ecosystem, but its maturity differs from Android, iOS, and desktop.

Current official documentation describes Compose Multiplatform as Stable for Android, iOS, and desktop, while web support based on Kotlin/Wasm remains in Beta.

This distinction is important when making production architecture decisions.

Don't treat every target as having the same maturity level simply because the framework supports it.

---

# Compose Multiplatform Resources

Sharing UI also creates a resource-sharing problem.

Applications need:

```text
Images
Strings
Fonts
Icons
Localization
Other Assets
```

Compose Multiplatform provides a multiplatform resources mechanism for accessing resources from common code.

This helps reduce another form of platform duplication.

Instead of maintaining completely separate resource access patterns, common code can consume multiplatform resources through the supported resource APIs.

---

# Developer Experience

For Android developers familiar with Jetpack Compose, the developer experience can be one of the strongest reasons to consider Compose Multiplatform.

The workflow feels conceptually familiar:

```text
Write Kotlin
    │
    ▼
Create Composables
    │
    ▼
Preview / Run
    │
    ▼
Iterate
```

Compose Multiplatform has also introduced tooling improvements such as unified previews and stable Compose Hot Reload in recent releases.

This reduces the conceptual distance between Android Compose development and multiplatform UI development.

---

# Migration From an Existing Android Application

This is where Compose Multiplatform becomes particularly interesting for Android teams.

Imagine an existing application:

```text
Large Android Application
        │
        ├── Kotlin
        ├── Jetpack Compose
        ├── ViewModel
        ├── Repository
        ├── Room
        └── Android APIs
```

A team doesn't necessarily need to rewrite everything.

A possible evolution could be:

```text
Existing Android App
        │
        ▼
Extract Shared Logic
        │
        ▼
Introduce KMP
        │
        ▼
Share Selected UI
        │
        ▼
Introduce iOS
        │
        ▼
Gradually Increase Sharing
```

Official KMP guidance explicitly supports both approaches:

- share logic while keeping native UI
- share both logic and UI using Compose Multiplatform

It also supports incremental migration from existing Android applications.

This is a significant architectural difference from a rewrite-oriented mindset.

---

# Compose Multiplatform vs Flutter

The two frameworks can appear similar because both can provide shared UI.

But their ecosystems are different.

### Flutter

```text
Dart
 │
 ▼
Flutter Framework
 │
 ▼
Flutter Engine
 │
 ▼
Platform
```

### Compose Multiplatform

```text
Kotlin
 │
 ▼
Compose Multiplatform
 │
 ▼
Kotlin Multiplatform
 │
 ▼
Platform
```

For an organization already invested in Kotlin and Android, Compose Multiplatform can offer a much smaller conceptual jump.

For a team already invested in Dart and Flutter, the opposite may be true.

Technology decisions are always contextual.

---

# Compose Multiplatform vs React Native

The difference is equally interesting.

### React Native

```text
JavaScript / TypeScript
          │
          ▼
     React Native
          │
          ▼
   Native Platform
```

### Compose Multiplatform

```text
Kotlin
  │
  ▼
Compose Multiplatform
  │
  ▼
Kotlin Multiplatform
  │
  ▼
Platform
```

The programming language, ecosystem, tooling, and native integration model are different.

A company with a large React organization may naturally prefer React Native.

A company with a large Kotlin and Android organization may naturally find Compose Multiplatform more attractive.

---

# Compose Multiplatform vs Native Android

This comparison is especially important for Android developers.

Native Android:

```text
Kotlin
  │
  ▼
Jetpack Compose
  │
  ▼
Android
```

Compose Multiplatform:

```text
Kotlin
  │
  ▼
Compose Multiplatform
  │
  ▼
Android + iOS + Desktop
```

The Android development experience can remain familiar while the target surface expands.

But the moment iOS becomes a first-class target, the engineering team must also account for:

- iOS build tooling
- Xcode
- Apple platform integration
- Platform-specific APIs
- App Store requirements
- iOS-specific testing

Multiplatform does not remove those responsibilities.

It reduces the amount of duplicated application code.

---

# The Most Important Distinction: Compose vs KMP

This distinction deserves its own section because it is one of the most common sources of confusion.

```text
             Kotlin Multiplatform
                    │
          ┌─────────┴─────────┐
          │                   │
       Shared Logic       Shared UI
          │                   │
          │             Compose Multiplatform
          │                   │
          └─────────┬─────────┘
                    ▼
               Applications
```

You can use KMP without Compose Multiplatform.

You can use Compose Multiplatform as the UI layer on top of KMP.

They solve different parts of the architecture.

> [!IMPORTANT]
> **KMP is the multiplatform foundation. Compose Multiplatform is an optional shared UI layer built on that foundation.**

Keeping this distinction clear will become essential in the later chapters of this book.

---

# Compose Multiplatform Scorecard

Using the evaluation criteria from Part 1:

| Dimension | Compose Multiplatform |
|-----------|-----------------------|
| Kotlin Ecosystem | ⭐⭐⭐⭐⭐ |
| Android Developer Familiarity | ⭐⭐⭐⭐⭐ |
| Shared UI | ⭐⭐⭐⭐⭐ |
| Shared Business Logic | ⭐⭐⭐⭐⭐* |
| Native Platform Access | ⭐⭐⭐⭐ |
| Android Integration | ⭐⭐⭐⭐⭐ |
| iOS UI Sharing | ⭐⭐⭐⭐ |
| Desktop UI Sharing | ⭐⭐⭐⭐ |
| Migration from Android | ⭐⭐⭐⭐ |
| Cross-Platform Development | ⭐⭐⭐⭐⭐ |
| Platform-Specific Escape Hatches | ⭐⭐⭐⭐⭐ |

`*` Shared business logic comes from Kotlin Multiplatform itself rather than from Compose Multiplatform alone.

These ratings are conceptual comparisons, not benchmarks.

---

# The Architectural Trade-Off

Compose Multiplatform introduces an attractive middle ground for Kotlin teams.

```text
                  Kotlin
                     │
                     ▼
             Shared Architecture
                     │
           ┌─────────┴─────────┐
           ▼                   ▼
    Shared Compose UI      Native UI
           │                   │
           └─────────┬─────────┘
                     ▼
              Native Platforms
```

The team can decide how far it wants to go.

### Share only logic

```text
KMP
│
└── Business Logic
```

### Share logic + UI

```text
KMP
│
├── Business Logic
└── Compose Multiplatform UI
```

### Mix both

```text
KMP
│
├── Shared Logic
├── Shared Compose UI
└── Native Platform UI
```

That flexibility is one of the strongest arguments for considering the KMP ecosystem as an architectural platform rather than a single framework.

---

# When Compose Multiplatform Is Attractive

Compose Multiplatform becomes particularly interesting when:

- The organization is already invested in Kotlin.
- Android teams already use Jetpack Compose.
- Android and iOS are both important targets.
- The team wants to share significant amounts of UI.
- Consistent UI across platforms is desirable.
- Shared business logic is also valuable.
- Desktop support is part of the product roadmap.
- Gradual adoption matters.
- The team wants the ability to fall back to native code.

---

# When Native UI May Still Be Better

Shared UI isn't automatically the correct decision.

Native UI may remain preferable when:

- Platform-specific UX is a major product differentiator.
- Android and iOS designs intentionally differ.
- The application depends heavily on native UI components.
- The product requires deep platform-specific behavior.
- Existing native teams and codebases are already highly mature.
- Sharing UI would create more abstraction than value.

In those cases, KMP can still provide significant value without sharing the UI.

```text
Android UI ───────┐
                  │
            Shared KMP
                  │
iOS UI ───────────┘
```

This is an important point.

> **Using KMP does not mean using Compose Multiplatform.**

---

# 🧠 The Bigger Lesson

Compose Multiplatform changes the conversation from:

> **"Can we share business logic?"**

to:

> **"How much of the application should we share?"**

That could include:

```text
Business Logic
      ↓
Data Layer
      ↓
Networking
      ↓
State Management
      ↓
UI
```

But sharing is not automatically beneficial at every layer.

The architecture should determine the boundary.

---

# Key Takeaways

> [!TIP]
> **Compose Multiplatform is especially compelling for Kotlin teams that want to extend the Compose experience beyond Android.**

The major points are:

- Compose Multiplatform is a UI framework built on Kotlin Multiplatform.
- It extends the declarative Compose programming model beyond Android.
- It supports shared UI across supported platforms including Android, iOS, and desktop.
- Android developers can leverage many familiar Compose concepts.
- Platform-specific APIs remain accessible through platform-specific code.
- Compose Multiplatform does not require all UI to be shared.
- KMP can be used independently for shared business logic.
- Compose Multiplatform can be introduced incrementally alongside existing native applications.
- Android, iOS, and desktop are currently Stable targets, while web support has a different maturity level.
- The strongest architectural advantage is flexibility: **share logic, share UI, or combine both approaches.**

---

## Looking Ahead

We have now examined four approaches:

```text
Native Android
      │
      ▼
Maximum Android Control


Flutter
      │
      ▼
Shared UI + Cross-Platform Rendering


React Native
      │
      ▼
React + Shared Application Model + Native Integration


Compose Multiplatform
      │
      ▼
Kotlin + Shared Compose UI + KMP
```

One approach remains.

**Kotlin Multiplatform.**

Unlike the technologies we've examined so far, KMP does not begin by asking us to replace the native UI layer.

It starts with a simpler proposition:

> **What if we could share the parts of the application that are truly common—and leave everything else exactly where it belongs?**

That question takes us to the final comparison in this chapter.
````
