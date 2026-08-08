
# Chapter 3 — Android vs Flutter vs React Native vs Compose vs KMP

## Part 1 — Evaluation Criteria

> **Before choosing a technology, define what "better" actually means.**

Comparing mobile technologies is surprisingly difficult.

A developer may say:

> "Flutter is better because development is faster."

Another may say:

> "Native Android is better because performance is excellent."

Someone else may prefer React Native because their organization already has a large JavaScript team.

And another team may choose Kotlin Multiplatform because they want to share business logic while keeping native UI.

All of these statements can be correct.

The problem is that they are answering **different questions**.

There is no universally best mobile technology.

There is only a technology that is **better suited to a particular product, team, business requirement, and long-term strategy**.

Before comparing Android, Flutter, React Native, Compose Multiplatform, and Kotlin Multiplatform, we therefore need a common set of evaluation criteria.

---

## Why a Simple Feature Comparison Isn't Enough

A typical comparison looks like this:

| Technology | Language | Cross-Platform | UI |
|------------|----------|----------------|----|
| Android | Kotlin | ❌ | Native |
| Flutter | Dart | ✅ | Shared |
| React Native | JavaScript / TypeScript | ✅ | Native-based |
| Compose Multiplatform | Kotlin | ✅ | Shared |
| KMP | Kotlin | ✅ | Native / Shared |

This table is useful.

But it doesn't help an engineering team make a serious architectural decision.

A real production decision involves much more than programming language or UI strategy.

For example:

- How much code needs to be shared?
- How important is native UX?
- How much platform-specific functionality is required?
- How large is the existing engineering team?
- What skills does the organization already have?
- How difficult will migration be?
- What happens five years from now?
- How does the technology affect testing?
- How does it affect CI/CD?
- What happens when a new operating-system capability is introduced?

Those questions matter far more than a simple feature checklist.

---

# The Five Technologies We'll Compare

Throughout this chapter, we'll evaluate five approaches.

### 1. Native Android

The application is built specifically for Android using Kotlin and the Android ecosystem.

```text
Kotlin
   │
   ▼
Android SDK
   │
   ▼
Android Application
```

---

### 2. Flutter

Flutter provides a shared development environment and its own rendering system for building applications across platforms.

```text
Dart
 │
 ▼
Flutter Framework
 │
 ▼
Flutter Engine
 │
 ├── Android
 └── iOS
```

---

### 3. React Native

React Native allows developers to build mobile applications using JavaScript or TypeScript while interacting with platform capabilities through React Native's architecture.

```text
JavaScript / TypeScript
          │
          ▼
     React Native
          │
    ┌─────┴─────┐
    ▼           ▼
 Android       iOS
```

---

### 4. Compose Multiplatform

Compose Multiplatform extends the declarative UI model of Jetpack Compose to multiple platforms.

```text
             Kotlin
                │
                ▼
       Compose Multiplatform
                │
       ┌────────┼────────┐
       ▼        ▼        ▼
    Android    iOS    Desktop
```

---

### 5. Kotlin Multiplatform

Kotlin Multiplatform focuses on sharing Kotlin code where sharing provides value while allowing platform-specific implementations where necessary.

```text
              Kotlin
                 │
                 ▼
        Shared Business Logic
                 │
       ┌─────────┴─────────┐
       ▼                   ▼
 Android Application   iOS Application
```

The important distinction is that **Kotlin Multiplatform doesn't require the entire application to become shared**.

That difference will become increasingly important as we move through this chapter.

---

# The Evaluation Framework

We'll evaluate each approach across the following dimensions:

| # | Criteria | Why It Matters |
|---|----------|----------------|
| 01 | 🎯 Product Fit | Does it solve the actual product problem? |
| 02 | 🧩 Code Sharing | What can realistically be reused? |
| 03 | 🎨 UI Strategy | Shared UI, native UI, or both? |
| 04 | 📱 Native Experience | How naturally does the app behave on each platform? |
| 05 | ⚡ Performance | How much runtime overhead does the architecture introduce? |
| 06 | 🔌 Platform Access | How easily can platform APIs be integrated? |
| 07 | 👨‍💻 Developer Experience | How productive is the engineering team? |
| 08 | 🧪 Testing | How easily can shared and platform-specific behavior be tested? |
| 09 | 🏗️ Architecture | How well does it support large applications? |
| 10 | 📦 Build & Release | How complex are builds and deployments? |
| 11 | 👥 Team Skills | What expertise does the organization need? |
| 12 | 🔄 Migration | How difficult is adoption for an existing product? |
| 13 | 📈 Scalability | Can the architecture survive product growth? |
| 14 | 🔮 Long-Term Fit | Will the decision remain reasonable as the product evolves? |

These criteria give us a much stronger foundation than simply asking which framework is "best."

---

# 01 — Product Fit

The first question should always be:

> **What are we actually building?**

Consider two completely different applications.

### Application A — Simple Consumer App

```text
Login
Profile
Feed
Settings
```

The application has limited platform-specific requirements.

Almost everything revolves around screens and network requests.

A cross-platform approach may provide significant value.

---

### Application B — Industrial Warehouse App

```text
Barcode Scanner
Bluetooth
RFID
Hardware Buttons
Background Processing
Offline Mode
Device Management
Printing
```

Now the situation is completely different.

The application depends heavily on platform and hardware capabilities.

Native Android may be the better choice.

The technology decision should therefore begin with the **product**, not the framework.

---

# 02 — Code Sharing

The next question is:

> **How much code do we actually want to share?**

There are several possible strategies.

### Strategy A — Share Almost Everything

```text
              Shared Code

        ┌──────────────────┐
        │       UI         │
        │     Logic        │
        │      Data        │
        │    Networking    │
        └──────────────────┘
```

This is the philosophy behind many cross-platform solutions.

---

### Strategy B — Share Selected Layers

```text
Android UI                  iOS UI
    │                          │
    └──────────┬───────────────┘
               ▼
       Shared Business Logic
               │
       Shared Data Layer
```

This is where Kotlin Multiplatform becomes particularly interesting.

The question isn't:

> "How much code can we share?"

It becomes:

> **"Which code should be shared?"**

That is a much more architectural question.

---

# 03 — UI Strategy

The user interface is one of the biggest differences between these technologies.

There are three broad approaches.

### Native UI

```text
Android → Jetpack Compose

iOS     → SwiftUI
```

Each platform owns its interface.

---

### Shared UI

```text
             Shared UI

          Compose / Flutter

             /       \
            /         \
       Android        iOS
```

The same UI architecture is used across platforms.

---

### Hybrid

```text
              Shared Logic

             /          \
            /            \
     Native Android    Native iOS
```

Business logic is shared while the UI remains native.

Each approach has legitimate use cases.

There is no universal winner.

---

# 04 — Native Experience

A mobile application isn't successful simply because it runs on two platforms.

It must **feel right** on those platforms.

Android users expect Android behavior.

iOS users expect iOS behavior.

Native experience includes:

- Navigation
- Gestures
- Animations
- Accessibility
- Platform conventions
- System integrations
- Keyboard behavior
- Permissions
- Notifications

A technology that makes code sharing easy but makes native behavior difficult may not be the right choice for every product.

---

# 05 — Performance

Performance should not be reduced to:

> "Is it native?"

Real performance is multidimensional.

We need to consider:

- Startup time
- Rendering performance
- Memory usage
- CPU usage
- Network efficiency
- Binary size
- Interoperability overhead
- Background processing
- Platform API interaction

A framework can perform extremely well while still introducing architectural trade-offs elsewhere.

Therefore, performance should be evaluated using the application's actual workload rather than theoretical assumptions.

> [!IMPORTANT]
> Performance claims should be validated with benchmarks from your own application. A framework comparison without workload-specific measurements can easily become misleading.

---

# 06 — Platform Access

Every mobile application eventually needs platform APIs.

Even a simple application may require:

```text
Camera
Bluetooth
Location
Notifications
Biometrics
Files
Contacts
Sensors
Background Tasks
```

The critical question becomes:

> **How difficult is it to access these capabilities when the shared abstraction doesn't cover them?**

This is particularly important for enterprise applications.

A technology that works beautifully for 90% of the application may still require careful architecture for the remaining 10%.

That final 10% can sometimes represent the most complicated part of the system.

---

# 07 — Developer Experience

Developer productivity is more than writing code quickly.

A good development environment should make it easy to:

- Create features
- Debug problems
- Run tests
- Inspect state
- Profile performance
- Understand build failures
- Review code
- Onboard new engineers

We should therefore evaluate:

```text
Developer Experience
        │
        ├── IDE
        ├── Build System
        ├── Debugging
        ├── Hot Reload
        ├── Documentation
        ├── Tooling
        └── Community
```

A technology that saves ten hours of coding but creates five hours of debugging isn't necessarily more productive.

---

# 08 — Testing

Testing becomes particularly important when code is shared.

Suppose the application contains:

```text
Authentication
Validation
Pricing
Cart
Checkout
Payments
```

If these components are shared, we should ideally test them once at the shared layer.

```text
             Shared Tests

                 │
      ┌──────────┼──────────┐
      ▼          ▼          ▼
 Android       iOS       Desktop
```

But platform-specific behavior still requires platform-specific testing.

The goal isn't to eliminate platform tests.

The goal is to avoid testing the same business rule independently on every platform.

---

# 09 — Architecture

A technology should support the architecture the product needs—not dictate it unnecessarily.

For a small application, almost any reasonable architecture may work.

For a large enterprise application, we may need:

- Modularization
- Dependency Injection
- Clean Architecture
- Feature isolation
- Shared domain models
- Repository abstraction
- Offline-first design
- Automated testing
- CI/CD
- Observability

The important question is:

> **Can this technology continue supporting our architecture as the application grows?**

A technology that works beautifully for a prototype may behave very differently when the project reaches hundreds of developers and dozens of modules.

---

# 10 — Build and Release

Developers often underestimate build complexity.

A mobile application isn't finished when the code compiles.

We also need:

```text
Source Code
     │
     ▼
Compilation
     │
     ▼
Testing
     │
     ▼
Signing
     │
     ▼
Packaging
     │
     ▼
CI/CD
     │
     ▼
App Store / Play Store
```

With multiple platforms, the pipeline becomes more complex.

A technology decision should therefore consider:

- Build time
- CI requirements
- Signing
- Dependency management
- Release automation
- Platform-specific tooling

Build engineering becomes increasingly important as organizations scale.

---

# 11 — Team Skills

Technology choices must also consider the people building the product.

Imagine a company with:

```text
20 Android Engineers
3 iOS Engineers
0 Flutter Engineers
0 React Engineers
```

Choosing a technology that requires an entirely different skill set may create significant organizational cost.

Existing expertise matters.

So does hiring.

So does developer retention.

A technology should fit not only the application but also the organization responsible for maintaining it.

---

# 12 — Migration Complexity

Greenfield projects are easy to evaluate.

Existing applications are much harder.

Imagine an Android application with:

```text
8 years of development

500+ screens

80+ modules

100+ developers

Thousands of tests

Multiple backend systems
```

Migrating everything to a new technology isn't realistic.

A good architecture should allow gradual adoption.

For example:

```text
Existing Android App
        │
        ▼
Introduce Shared Module
        │
        ▼
Move One Feature
        │
        ▼
Test
        │
        ▼
Move Another Feature
        │
        ▼
Gradual Adoption
```

This is one of the areas where Kotlin Multiplatform can provide an important architectural advantage.

It doesn't require the entire application to move at once.

---

# 13 — Scalability

A technology decision should survive the life of the product.

Ask:

> **What happens when the application becomes ten times larger?**

Consider:

```text
10 Developers
      ↓
50 Developers
      ↓
200 Developers
      ↓
Multiple Teams
      ↓
Multiple Countries
      ↓
Multiple Platforms
```

As teams grow, architecture becomes increasingly important.

We need clear ownership.

We need modular boundaries.

We need predictable builds.

We need reliable testing.

We need consistent business behavior.

The right technology should help us scale these dimensions rather than become a bottleneck.

---

# 14 — Long-Term Fit

Finally, we need to think beyond the next release.

A technology decision can remain in production for five, ten, or even fifteen years.

Ask:

- Is the ecosystem actively maintained?
- Can we hire engineers?
- Can we migrate when requirements change?
- Does the technology support our target platforms?
- Can it integrate with future platform capabilities?
- Can the architecture evolve without a complete rewrite?

The best technology isn't necessarily the one with the most features today.

It may be the one that gives the organization the **most flexibility tomorrow**.

---

# A Better Comparison Model

Rather than declaring a winner, we can visualize the decision as a set of trade-offs.

```text
                    Product Requirements
                             │
                             ▼
                    ┌─────────────────┐
                    │ Evaluation      │
                    │ Criteria        │
                    └────────┬────────┘
                             │
       ┌─────────────┬───────┼────────┬─────────────┐
       ▼             ▼       ▼        ▼             ▼
   Code Sharing    Native   UI     Performance   Platform
                  Experience         Access
       │             │       │        │             │
       └─────────────┴───────┼────────┴─────────────┘
                             ▼
                    Technology Choice
```

The result isn't:

> **"Technology X is the best."**

The result should be:

> **"Technology X is the best fit for our requirements."**

That distinction separates technology selection from technology advocacy.

---

# The Most Important Criteria

Although we'll evaluate many dimensions throughout this chapter, five questions should remain at the center of the discussion.

### 1. What needs to be shared?

```text
UI?
Business Logic?
Data Layer?
Networking?
Everything?
```

### 2. How native does the experience need to be?

```text
Highly Native
      │
      ├── Native UI
      │
      └── Platform APIs
```

### 3. How much platform-specific functionality exists?

```text
Low Platform Dependency
        ↓
        KMP / Cross-platform options

High Platform Dependency
        ↓
        Native may be preferable
```

### 4. What does the existing team already know?

Technology adoption is an organizational decision as much as a technical one.

### 5. What will the application look like five years from now?

This question is often ignored.

It shouldn't be.

---

# The Decision Is About Trade-Offs

Every technology gives us something.

And every technology asks us to give something in return.

```text
                ┌─────────────────┐
                │  More Sharing   │
                └────────┬────────┘
                         │
                         ▼
              Potentially Less
             Platform Specificity

                         │
                         │
                         ▼

                ┌─────────────────┐
                │  More Native    │
                └────────┬────────┘
                         │
                         ▼
              Potentially More
                  Duplication
```

The goal isn't to reach one extreme.

The goal is to find the point where the trade-offs make sense for the product.

---

# 🧠 The Architectural Question

The most important question in this chapter isn't:

> **"Which framework should I choose?"**

It is:

> **"Which responsibilities should be shared, and which responsibilities should remain platform-specific?"**

Once we answer that question, technology selection becomes much easier.

A technology is simply the mechanism we use to implement that architectural decision.

---

# 📌 Evaluation Checklist

Before choosing a mobile architecture, ask:

- [ ] What platforms do we need to support?
- [ ] What business logic is common?
- [ ] How important is native UI?
- [ ] How many platform-specific APIs are required?
- [ ] What is our team's existing skill set?
- [ ] How large is the application?
- [ ] How long will the product live?
- [ ] How difficult will migration be?
- [ ] How will testing work?
- [ ] How will CI/CD work?
- [ ] How will the architecture scale?
- [ ] What happens when the product adds another platform?

If these questions aren't answered, choosing a framework based on popularity or personal preference is premature.

---

> [!TIP]
> **Don't choose a technology because it can share the most code. Choose it because it creates the best engineering outcome for your product.**

---

## What's Next?

We now have a common evaluation framework.

The next step is to apply it.

In the following parts of this chapter, we'll evaluate each approach using the same questions—starting with **Native Android** and then moving through **Flutter, React Native, Compose Multiplatform, and Kotlin Multiplatform**.

Only after examining them individually will we put them side by side and determine where each approach makes the most sense.
````

