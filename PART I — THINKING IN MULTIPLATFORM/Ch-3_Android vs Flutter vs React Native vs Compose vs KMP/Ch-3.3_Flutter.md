# Chapter 3 — Android vs Flutter vs React Native vs Compose vs KMP

## Part 3 — Flutter

> **Flutter changed the cross-platform conversation by taking control of the rendering layer.**

When evaluating Flutter, it is important not to think of it as "Android, but cross-platform."

Flutter makes a fundamentally different architectural decision.

Instead of asking Android or iOS to provide the application's user interface, Flutter provides its own UI framework and rendering pipeline.

That decision is the foundation of Flutter's strengths—and also explains many of its trade-offs.

---

# What Is Flutter?

Flutter is Google's UI toolkit for building applications across multiple platforms using a single codebase.

The primary programming language is **Dart**.

At a simplified level:

```text
                  Flutter Application
                          │
                          ▼
                    Dart Code
                          │
                          ▼
                  Flutter Framework
                          │
                          ▼
                   Flutter Engine
                          │
                          ▼
                 Platform Integration
                  ┌───────┴───────┐
                  ▼               ▼
               Android           iOS
```

The important difference from native Android is immediately visible.

A Flutter application doesn't primarily build its UI from Android's standard UI toolkit.

Flutter owns much of the rendering experience.

---

# The Flutter Mental Model

A Flutter application is built from widgets.

Everything is represented as a widget:

- Text
- Buttons
- Layouts
- Images
- Lists
- Navigation
- Screens
- Gestures

A simplified application might look like:

```text
MaterialApp
    │
    ▼
Scaffold
    │
    ├── AppBar
    │
    └── Column
          │
          ├── Text
          ├── TextField
          └── Button
```

This widget-based model provides a consistent way of describing UI.

The developer thinks in terms of a widget tree rather than traditional platform-specific UI components.

---

# Why Flutter Took a Different Approach

Flutter's core philosophy can be summarized as:

> **Control the UI rendering experience instead of depending entirely on the platform UI toolkit.**

This gives Flutter a major advantage.

If Flutter controls rendering, the same UI implementation can behave consistently across platforms.

Consider:

```text
             Flutter UI Code

                   │

          ┌────────┴────────┐
          ▼                 ▼
       Android             iOS

       Same UI             Same UI
```

The developer doesn't have to maintain completely separate UI implementations.

This is one of Flutter's biggest attractions.

---

# Figure 3.2 — Flutter Rendering Model

```text
                  Flutter Widgets
                         │
                         ▼
                 Flutter Framework
                         │
                         ▼
                  Flutter Engine
                         │
                         ▼
                 Rendering Pipeline
                         │
              ┌──────────┴──────────┐
              ▼                     ▼
          Android Surface        iOS Surface
```

> **Key Idea**
>
> Flutter controls much more of the rendering pipeline than a traditional native application.

---

# The "Write Once" Appeal

Suppose a company wants to build a product for Android and iOS.

With native development:

```text
Android Code
     +
iOS Code
```

With Flutter:

```text
          Flutter Codebase
                 │
          ┌──────┴──────┐
          ▼             ▼
      Android           iOS
```

A large portion of the application can be implemented in Dart.

This can significantly reduce duplicated UI and application code.

For organizations trying to move quickly, this is a compelling proposition.

---

# Flutter and UI Consistency

One of Flutter's strongest characteristics is visual consistency.

If the product team wants a particular design system to look almost identical across platforms, Flutter can make that easier.

For example:

```text
Design System

     │
     ▼

Flutter Widgets

     │
 ┌───┴───┐
 ▼       ▼
Android  iOS
```

The same widget hierarchy can be rendered on both platforms.

This is especially useful for products where visual consistency is more important than following every platform-specific design convention.

---

# But Consistency Is a Choice

There is an important distinction here.

A consistent interface is not automatically a native interface.

Android users and iOS users have different expectations.

For example:

```text
Android

Material-oriented
Navigation patterns
Back gesture
System conventions
```

versus:

```text
iOS

Apple-oriented
Navigation patterns
Gesture conventions
System conventions
```

Flutter can reproduce platform-specific behavior, but the team must intentionally design and implement it.

This is not necessarily a problem.

It is simply an architectural trade-off.

---

# Flutter Widgets vs Native Views

A simplified comparison looks like this.

### Native Android

```text
Jetpack Compose / Android Views
            │
            ▼
       Android APIs
            │
            ▼
       Android OS
```

### Flutter

```text
Flutter Widgets
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

Flutter introduces another layer between application code and the operating system.

That additional layer enables cross-platform consistency.

It also means developers must understand the framework's rendering model.

---

# Flutter's Development Experience

One of Flutter's major advantages is developer productivity.

Flutter became popular partly because of its development workflow.

Features such as hot reload make it possible to make changes and see results quickly during development.

The typical workflow becomes:

```text
Change Code
    │
    ▼
Hot Reload
    │
    ▼
Observe UI
    │
    ▼
Adjust
    │
    └───────────┐
                │
                ▼
             Repeat
```

This tight feedback loop can make UI development extremely productive.

---

# Flutter and Application Architecture

Flutter does not force a single application architecture.

Teams can implement:

- MVVM
- Clean Architecture
- BLoC
- Riverpod-based architectures
- Redux-style architectures
- Custom state management

The framework primarily provides the UI and application development environment.

Architecture remains the responsibility of the engineering team.

A simplified architecture might look like:

```text
┌───────────────────────────────┐
│          Flutter UI           │
└───────────────┬───────────────┘
                │
                ▼
┌───────────────────────────────┐
│       State Management        │
└───────────────┬───────────────┘
                │
                ▼
┌───────────────────────────────┐
│       Business Logic          │
└───────────────┬───────────────┘
                │
                ▼
┌───────────────────────────────┐
│       Data / Repository       │
└───────────────┬───────────────┘
                │
                ▼
          Platform / API
```

The architectural quality of a Flutter application still depends heavily on how the engineering team structures the system.

---

# Platform Integration

No matter how good a cross-platform framework is, applications eventually need native capabilities.

For example:

```text
Camera
Bluetooth
Biometrics
Location
Notifications
Files
Sensors
Background Tasks
```

Flutter provides mechanisms for interacting with platform-specific functionality.

A simplified model looks like:

```text
                 Flutter App
                     │
                     ▼
             Flutter Platform API
                     │
             ┌───────┴───────┐
             ▼               ▼
       Android Code       iOS Code
             │               │
             ▼               ▼
       Android APIs       iOS APIs
```

This allows developers to extend Flutter when a requirement isn't fully covered by the shared framework.

However, platform integration introduces additional architecture and maintenance considerations.

---

# Flutter Plugins

Plugins provide another important part of the Flutter ecosystem.

A plugin can expose platform functionality through a common API.

For example:

```text
Flutter Application

        │
        ▼

   Common Plugin API

      /       \
     /         \
Android       iOS
Implementation Implementation
```

This model allows application code to remain mostly platform-independent.

The underlying implementations can still be platform-specific.

This is a useful pattern, but it introduces another dependency layer that engineering teams need to maintain.

---

# The Plugin Dependency Question

A production application may depend on many plugins.

For example:

```text
Application
    │
    ├── Authentication Plugin
    ├── Analytics Plugin
    ├── Camera Plugin
    ├── Maps Plugin
    ├── Push Plugin
    └── Storage Plugin
```

Each plugin becomes part of the application's technology stack.

Teams should therefore consider:

- Maintenance
- Platform support
- Compatibility
- Release cadence
- Documentation
- Community health

A cross-platform framework doesn't eliminate platform complexity.

It can move some of that complexity into plugins and integrations.

---

# Performance: A More Nuanced View

It is tempting to summarize Flutter performance as:

> "Flutter is fast."

That statement is too broad.

Performance depends on the application.

Factors include:

- Rendering complexity
- Number of widgets
- Animation workload
- Image processing
- Network operations
- Memory usage
- Native integrations
- Application architecture

Flutter's rendering architecture can provide excellent performance for many applications.

But performance should always be evaluated against the actual workload.

> [!IMPORTANT]
> Avoid choosing a mobile technology based on generic performance claims. Measure startup, rendering, memory, and interaction performance for the application you are actually building.

---

# Flutter and Native APIs

Flutter applications can still interact with Android and iOS.

The architecture therefore becomes:

```text
                  Flutter
                     │
        ┌────────────┴────────────┐
        ▼                         ▼
 Android Integration       iOS Integration
        │                         │
        ▼                         ▼
 Android APIs               iOS APIs
```

This is important because cross-platform development does not eliminate platform-specific requirements.

It simply changes where those requirements are handled.

---

# Flutter's Strengths

Flutter is particularly attractive when a team values:

### 🎨 Shared UI

The same UI code can target multiple platforms.

### ⚡ Fast Development

The development feedback loop can be very fast.

### 🧩 Consistent Design

Applications can maintain a consistent visual identity across platforms.

### 👥 Single Technology Stack

A team can build much of the product using Dart and Flutter.

### 🚀 Greenfield Development

Flutter can be particularly attractive when building a new application without an existing native codebase.

---

# Where Flutter Requires Careful Evaluation

Flutter may require additional consideration when an application depends heavily on platform-specific behavior.

Examples include:

- Specialized hardware
- Deep Android integration
- Deep iOS integration
- Platform-specific accessibility behavior
- Highly native interaction patterns
- Existing large native codebases

This doesn't mean Flutter cannot support these scenarios.

It means the engineering team should understand the additional integration layer before making the decision.

---

# Flutter in an Existing Native Application

Greenfield development and migration are very different problems.

Consider an existing Android application:

```text
8 Years
   │
   ├── 100+ Modules
   ├── Large Native Codebase
   ├── Existing Tests
   ├── Existing CI/CD
   └── Existing Android Expertise
```

Introducing Flutter doesn't automatically eliminate this investment.

The organization must decide:

```text
Rewrite?
   │
   ├── Yes → High migration cost
   │
   └── No → Hybrid architecture
```

Migration strategy therefore becomes an important part of technology selection.

---

# Flutter vs Native Android

At this point, we can make a more meaningful comparison.

| Dimension | Native Android | Flutter |
|-----------|----------------|---------|
| Android Integration | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| Native Android UI | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| Shared UI | ⭐ | ⭐⭐⭐⭐⭐ |
| Cross-Platform Development | ⭐ | ⭐⭐⭐⭐⭐ |
| Development Speed | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| Platform Control | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| Existing Android Migration | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| Visual Consistency | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| Android-Specific Hardware | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| Single UI Codebase | ❌ | ✅ |

These ratings are relative and depend heavily on the product.

They are not benchmarks.

---

# The Architectural Trade-Off

Flutter makes one major trade:

```text
          More Shared UI
                 │
                 ▼
       Less UI Duplication
                 │
                 ▼
       More Framework Control
                 │
                 ▼
       Additional Abstraction
```

Native Android makes a different trade:

```text
       Maximum Android Control
                 │
                 ▼
        Maximum Native Fit
                 │
                 ▼
      Platform-Specific Code
                 │
                 ▼
       More Cross-Platform
          Duplication
```

Neither trade is universally correct.

The product determines which one makes sense.

---

# When Flutter Can Be a Strong Choice

Flutter can be particularly attractive when:

- Android and iOS are both first-class targets.
- The UI should be highly shared.
- Consistent visual design is important.
- The team prefers a single UI technology.
- The application is greenfield.
- Rapid UI development is a priority.
- Platform-specific requirements are manageable.

---

# When Native Android May Be Better

Native Android can remain the stronger option when:

- Android is the only target.
- The application relies heavily on Android APIs.
- Hardware integration is extensive.
- The application requires deeply Android-specific behavior.
- The organization already has a large Android codebase.
- Native Android expertise is a major organizational strength.

---

# 🧠 The Important Lesson

Flutter demonstrates an important idea:

> **Sharing the UI can dramatically reduce duplicated development work.**

But it also introduces another question:

> **Do we actually want the UI to be shared?**

For some products, the answer is clearly yes.

For others, native UI is a product requirement.

This distinction becomes critical when we compare Flutter with Kotlin Multiplatform.

Flutter asks:

```text
"Can we share the application experience?"
```

Kotlin Multiplatform asks:

```text
"Which parts of the application should be shared?"
```

Those questions sound similar.

Architecturally, they are very different.

---

# Key Takeaways

> [!TIP]
> **Flutter is strongest when shared UI and cross-platform development are central requirements.**

The major points are:

- Flutter uses Dart as its primary development language.
- It provides its own widget and rendering model.
- A large portion of the application can be shared across platforms.
- Shared UI is one of Flutter's biggest strengths.
- Flutter can still integrate with native platform APIs.
- Platform-specific functionality may require plugins or native implementations.
- Performance must be evaluated using real application workloads.
- Flutter can be attractive for greenfield products with strong cross-platform requirements.
- Migration from large native applications requires careful planning.
- Flutter optimizes for significant code and UI sharing rather than preserving completely separate native UI implementations.

---

## Looking Ahead

Flutter demonstrates one end of the cross-platform spectrum:

> **Share a large portion of the application, including the UI.**

The next approach takes a different route.

**React Native** keeps JavaScript or TypeScript at the center of application development while maintaining a strong relationship with native platform components.

That gives us another interesting architectural trade-off to examine:

> **What happens when shared application logic meets native UI components?**
````
