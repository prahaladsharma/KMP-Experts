# Chapter 3 — Android vs Flutter vs React Native vs Compose vs KMP

## Part 7 — Choosing the Right Technology

> **The right technology is not the one with the most features. It is the one that creates the best engineering outcome for your product.**

After comparing Native Android, Flutter, React Native, Compose Multiplatform, and Kotlin Multiplatform, it would be easy to finish this chapter with a winner.

That would miss the point.

There is no universal winner.

A technology that is an excellent choice for a consumer application may be a poor choice for a warehouse application. A framework that works well for a greenfield product may be difficult to introduce into a ten-year-old native codebase.

Even the official Kotlin guidance emphasizes choosing based on project requirements, platform targets, team expertise, and the amount of code you actually want to share rather than looking for a single "best" cross-platform solution.

The real engineering question is:

> **What does this product need, and which technology gives us the best trade-off?**

---

# Start With the Product, Not the Framework

One of the most common mistakes in technology selection is starting with a technology.

For example:

> "Our team wants to use Flutter."

or:

> "KMP is becoming popular, so we should migrate."

or:

> "React Native will let us share everything."

These statements begin with a solution.

A better process starts with the product.

```text
                    PRODUCT
                       │
                       ▼
              Business Requirements
                       │
                       ▼
              Technical Requirements
                       │
                       ▼
             Architectural Constraints
                       │
                       ▼
                Technology Choice
```

The technology comes last.

---

# The First Question: How Many Platforms?

Start with the simplest question:

> **How many platforms actually matter to this product?**

### Scenario 1 — Android Only

```text
              Product
                 │
                 ▼
              Android
                 │
                 ▼
           Native Android
```

If the product will only exist on Android, introducing a cross-platform framework may add complexity without solving a real problem.

Native Android is likely the simplest architecture.

---

### Scenario 2 — Android + iOS

```text
              Product
             /       \
            ▼         ▼
        Android       iOS
```

Now code duplication becomes an architectural concern.

The decision becomes more interesting.

Possible approaches include:

```text
Native Android + Native iOS
          │
          ├── Maximum platform independence
          └── Maximum duplicated implementation

Flutter
          │
          └── High application/UI sharing

React Native
          │
          └── Shared React-based application model

KMP
          │
          └── Selective sharing

Compose Multiplatform
          │
          └── Shared Kotlin UI + logic
```

---

# The Second Question: What Needs to Be Shared?

This is arguably the most important question in the entire chapter.

Don't ask:

> **"How much code can we share?"**

Ask:

> **"Which code represents the same product behavior on every platform?"**

Consider an e-commerce application.

### Likely Shared

```text
Product Models
Pricing Rules
Discount Rules
Cart Logic
Authentication
Validation
API Models
Networking
Caching
Synchronization
```

### Likely Platform-Specific

```text
Android Back Navigation
iOS Navigation Gestures
Android Notifications
Apple Push Integration
Biometric APIs
Platform UI
Platform-specific Hardware
```

This gives us a much clearer architectural boundary.

```text
              PRODUCT LOGIC
                   │
                   ▼
             Shared Layer
                   │
          ┌────────┴────────┐
          ▼                 ▼
       Android             iOS
       Native              Native
       Layer               Layer
```

KMP is particularly strong when this is the architecture you actually want.

---

# The Third Question: How Important Is Native UI?

The UI can completely change the technology decision.

Ask:

> **Does Android need to behave like Android and iOS need to behave like iOS?**

If the answer is strongly **yes**, native UI becomes more valuable.

```text
Android
   │
   ▼
Jetpack Compose

iOS
   │
   ▼
SwiftUI
```

Shared business logic can still sit underneath:

```text
        Jetpack Compose          SwiftUI
               │                   │
               └─────────┬─────────┘
                         ▼
                   Shared KMP
                         │
             ┌───────────┼───────────┐
             ▼           ▼           ▼
           Domain       Data       Network
```

This architecture gives the product two things:

**Native user experience**

+

**Shared business behavior**

That combination is one of the strongest reasons to consider KMP.

---

# When Shared UI Makes More Sense

There are also products where visual consistency is more important than platform-specific UI.

For example:

```text
Brand-heavy Consumer App
          │
          ▼
   Consistent UI Design
          │
     ┌────┴────┐
     ▼         ▼
 Android       iOS
```

In such a product, shared UI may provide significant value.

Possible choices include:

- Flutter
- React Native
- Compose Multiplatform

Compose Multiplatform is especially interesting for Kotlin teams because it can share UI alongside shared KMP logic. Current official documentation lists Compose Multiplatform as Stable for Android, iOS, and desktop, while its WebAssembly-based web target is Beta.

---

# The Fourth Question: How Much Platform-Specific Functionality Exists?

This question is often underestimated.

Consider two applications.

### Application A

```text
Login
Feed
Profile
Search
Settings
```

Platform dependency:

```text
LOW
```

A shared approach may work very well.

---

### Application B

```text
Bluetooth
Barcode Scanner
RFID
USB
Background Processing
Device Management
Printing
Camera
Specialized Hardware
```

Platform dependency:

```text
HIGH
```

Native development becomes more attractive.

This doesn't automatically eliminate KMP.

In fact, KMP can still be useful for the business logic while platform integrations remain native.

Official Kotlin guidance specifically identifies hardware-intensive applications and highly platform-specific UX as scenarios where native development can be valuable, while also noting that KMP can share business logic underneath native UI.

---

# The Fifth Question: What Does the Team Already Know?

Technology selection is also a people decision.

Imagine:

```text
Team A

30 Android Developers
Kotlin
Jetpack Compose
Gradle
```

For this team:

```text
KMP
Compose Multiplatform
Native Android
```

may have a relatively low learning barrier.

Now consider:

```text
Team B

40 React Developers
TypeScript
React
Node.js
```

React Native may be a more natural choice.

And:

```text
Team C

25 Dart / Flutter Developers
```

Flutter may be the obvious organizational choice.

The technology must fit the people who will maintain it.

---

# Existing Skills Are an Asset

A technology decision should not throw away years of accumulated engineering knowledge without a very good reason.

Consider an Android organization that already has:

```text
Kotlin
   │
Jetpack Compose
   │
Coroutines
   │
Flow
   │
Gradle
   │
Android Architecture
```

Moving to another ecosystem means introducing:

```text
New Language
New Tooling
New Build System
New Architecture
New Libraries
New Debugging Model
New Hiring Requirements
```

Sometimes that investment is justified.

Sometimes it isn't.

KMP has a particularly interesting position here because it allows existing Kotlin expertise to remain valuable while extending the codebase to additional platforms. Official Kotlin guidance also highlights Kotlin skill reuse as one factor in choosing KMP.

---

# The Sixth Question: Greenfield or Existing Application?

This changes everything.

## Greenfield Application

You have:

```text
No Existing Code
No Existing Architecture
No Migration Cost
```

The team can choose almost anything.

```text
Requirements
    │
    ▼
Architecture
    │
    ▼
Technology
```

---

## Existing Application

Now you have:

```text
Years of Code
+
Existing Architecture
+
Existing Tests
+
Existing CI/CD
+
Existing Developers
+
Existing Release Process
```

The decision becomes:

```text
          Existing Application
                  │
          ┌───────┴───────┐
          ▼               ▼
       Rewrite         Incremental
                        Adoption
```

For mature applications, incremental adoption is often far more realistic.

KMP explicitly supports incremental adoption, including sharing a small piece of logic, sharing business logic while keeping native UI, or gradually making an existing Android application multiplatform.

---

# The Seventh Question: What Is the Expected Lifetime?

A prototype and a ten-year enterprise application should not necessarily use the same decision criteria.

For a prototype:

```text
Time to Market
       │
       ▼
Developer Speed
       │
       ▼
Technology Choice
```

For a long-lived enterprise application:

```text
Maintainability
      +
Team Scalability
      +
Architecture
      +
Testing
      +
Platform Evolution
      +
Migration Strategy
      │
      ▼
Technology Choice
```

The longer the expected lifetime, the more important architecture becomes.

---

# The Eighth Question: How Much Duplication Can We Accept?

This is where the economics become interesting.

Imagine a feature that requires:

```text
500 lines of business logic
```

If two platforms independently implement it:

```text
Android → 500
iOS     → 500

Total → 1,000 lines
```

But the actual cost isn't simply 1,000 lines.

You also maintain:

```text
2 Implementations
2 Test Suites
2 Bug Fixes
2 Reviews
2 Release Paths
2 Opportunities for Divergence
```

Over time, the cost compounds.

---

# Duplication Is Not Always Bad

This is important.

Developers sometimes treat duplicated code as automatically bad.

It isn't.

Duplication can be a deliberate architectural choice.

For example:

```text
Android UI
```

and:

```text
iOS UI
```

may intentionally be different.

Trying to force them into one implementation could create more complexity than maintaining two implementations.

The real problem is **accidental duplication**.

```text
Intentional Difference
        │
        ▼
      Good

Unnecessary Duplication
        │
        ▼
      Cost
```

KMP is most valuable when it removes the second category without destroying the first.

---

# The Ninth Question: What Is the Cost of Abstraction?

Every abstraction has a price.

Sharing code can reduce duplication.

But it can also introduce:

- Additional build complexity
- More architectural rules
- Platform boundaries
- Dependency constraints
- Debugging complexity
- Team learning requirements

Therefore:

```text
Value of Sharing
       >
Cost of Sharing
```

should be true before introducing a shared abstraction.

If the opposite is true:

```text
Cost of Sharing
       >
Value of Sharing
```

keep the implementation native.

---

# A Practical Technology Matrix

The following matrix is not a universal ranking.

It is a decision aid.

| Requirement | Native Android | Flutter | React Native | Compose MP | KMP |
|---|:---:|:---:|:---:|:---:|:---:|
| Android-only | ⭐⭐⭐⭐⭐ | ⭐⭐ | ⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ |
| Native Android control | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| Shared UI | ❌ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | Optional |
| Shared business logic | ❌ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| Native UI freedom | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| Kotlin ecosystem | ⭐⭐⭐⭐⭐ | ⭐ | ⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| React ecosystem | ⭐ | ⭐ | ⭐⭐⭐⭐⭐ | ⭐ | ⭐ |
| Dart ecosystem | ⭐ | ⭐⭐⭐⭐⭐ | ⭐ | ⭐ | ⭐ |
| Incremental Android adoption | ⭐⭐⭐⭐⭐ | ⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| Hardware-heavy Android app | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| Shared architecture flexibility | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |

Again, these are architectural signals—not benchmark scores.

---

# The Decision Tree

A practical decision can be made using a simple sequence.

```text
                    START
                      │
                      ▼
             How many platforms?
                      │
             ┌────────┴────────┐
             ▼                 ▼
          One              Multiple
             │                 │
             ▼                 ▼
          Native       What should be shared?
                               │
                    ┌──────────┴──────────┐
                    ▼                     ▼
                  UI + Logic            Logic
                    │                     │
                    ▼                     ▼
             Shared UI needed?       Native UI?
                    │                     │
              ┌─────┴─────┐         ┌────┴────┐
              ▼           ▼         ▼         ▼
             Yes          No       Yes        No
              │            │        │          │
              ▼            ▼        ▼          ▼
       Flutter / RN /   Native   KMP       Evaluate
       Compose MP       + KMP              alternatives
```

This is only a starting point.

Real projects require deeper analysis.

---

# A Better KMP Decision Rule

If you're specifically evaluating KMP, ask these five questions:

### 1. Do Android and iOS share meaningful business behavior?

```text
No → KMP may provide limited value.

Yes → Continue.
```

### 2. Do we want native UI?

```text
Yes → KMP + Native UI

No → KMP + Compose Multiplatform
```

### 3. Do we need deep platform integration?

```text
Yes → Keep those parts platform-specific.

No → Share more if valuable.
```

### 4. Do we already have Kotlin expertise?

```text
Yes → Adoption becomes easier.

No → Evaluate learning and hiring cost.
```

### 5. Is incremental migration important?

```text
Yes → KMP becomes particularly attractive.

No → Compare all viable approaches.
```

---

# Three Real-World Scenarios

Let's apply the framework.

---

## Scenario 1 — Consumer Social Application

Requirements:

```text
Android + iOS
Shared UI
Rapid development
Consistent branding
Moderate platform integration
```

Potential choices:

```text
Flutter
React Native
Compose Multiplatform
```

KMP with native UI may be less attractive if the primary goal is maximum UI sharing.

Compose Multiplatform may become attractive if the organization is already strongly invested in Kotlin.

---

## Scenario 2 — Enterprise Banking Application

Requirements:

```text
Android + iOS
Strong security
Native authentication
Native UX
Large business domain
Complex business rules
Long product lifetime
```

A strong candidate architecture could be:

```text
Android UI ───────┐
                  │
                  ▼
            Shared KMP Core
                  │
       ┌──────────┼──────────┐
       ▼          ▼          ▼
    Domain      Data      Networking
                  │
                  ▼
          Platform Security
```

Here, KMP's selective sharing model becomes particularly compelling.

---

## Scenario 3 — Industrial Warehouse Application

Requirements:

```text
Android
Barcode Scanner
RFID
Bluetooth
Printers
Offline-first
Dedicated Hardware
```

The first question should be:

> **Why introduce cross-platform technology if there is only one platform?**

Native Android may be the best answer.

If an iOS application becomes necessary later, KMP could then be evaluated for the shared business and data layers.

---

# Don't Choose KMP Because It Is Popular

This is worth emphasizing.

Technology popularity is not an architecture strategy.

Don't choose KMP because:

- Someone posted about it.
- A conference talked about it.
- A company adopted it.
- It appears in job descriptions.
- It is the "future."
- It allows you to say the application is cross-platform.

Choose it when it solves a real problem.

---

# Don't Reject KMP Because It Isn't 100% Shared

This is the opposite mistake.

Some teams evaluate KMP and ask:

> "Can we share everything?"

If the answer is no, they conclude that KMP failed.

That's the wrong measurement.

Consider:

```text
Application
│
├── 20% Platform-specific
└── 80% Shared
```

If that 20% represents:

```text
Native UI
Hardware
Security
Platform APIs
```

then the architecture may be exactly what the product needs.

**80% meaningful sharing can be better than 100% artificial sharing.**

---

# The 80/20 Trap

Do not turn code-sharing percentages into a goal.

You might hear statements such as:

> "Our architecture shares 90% of the code."

That sounds impressive.

But ask:

> **"What exactly is that 90%?"**

If the shared code is mostly trivial utilities while the important business rules remain duplicated, the percentage doesn't mean much.

Conversely:

```text
Shared:
Authentication
Payments
Pricing
Order State
Synchronization
Business Rules
```

may be extremely valuable even if the total shared percentage is lower.

Measure **business value**, not lines of code.

---

# The Cost Model

A serious technology decision should consider four categories.

```text
                 Total Cost
                     │
       ┌─────────────┼─────────────┐
       ▼             ▼             ▼
 Development     Maintenance    Migration
       │             │             │
       └─────────────┼─────────────┘
                     ▼
                  People
```

More specifically:

### Development Cost

- Initial implementation
- Training
- Tooling
- Architecture

### Maintenance Cost

- Bug fixes
- Dependencies
- Platform updates
- Feature parity

### Migration Cost

- Existing code
- Tests
- CI/CD
- Release process

### People Cost

- Hiring
- Training
- Team structure
- Knowledge transfer

A framework that looks cheap during development may become expensive during maintenance.

---

# The Five-Year Question

Before approving a major architecture decision, ask:

> **"What will this look like five years from now?"**

Imagine:

```text
Year 1
  │
  ▼
10 Developers

Year 2
  │
  ▼
30 Developers

Year 3
  │
  ▼
Multiple Teams

Year 4
  │
  ▼
Android + iOS + Desktop

Year 5
  │
  ▼
Large Enterprise Platform
```

Can the architecture evolve?

Can teams work independently?

Can new developers understand the boundaries?

Can platform-specific functionality be introduced without breaking everything?

Can the application survive a major OS change?

Those questions are more important than the framework's current popularity.

---

# The Most Important Decision: Architecture Boundary

After comparing all five approaches, we can reduce the entire chapter to one architectural decision:

```text
                   APPLICATION
                        │
                        ▼
               ┌─────────────────┐
               │ Shared Boundary │
               └────────┬────────┘
                        │
              ┌─────────┴─────────┐
              ▼                   ▼
        Shared Responsibilities
              │
              ▼
              KMP
```

Everything outside that boundary remains platform-specific.

The boundary itself is the architecture.

The technology simply implements it.

---

# A Technology Selection Checklist

Before selecting a technology, answer these questions honestly:

### Product

- [ ] Which platforms are required?
- [ ] Which platforms are planned later?
- [ ] How important is platform-specific UX?
- [ ] How long will the product live?

### Architecture

- [ ] What business logic is common?
- [ ] What data layer is common?
- [ ] What UI should remain native?
- [ ] Which platform APIs are unavoidable?

### Team

- [ ] What languages does the team already know?
- [ ] What ecosystem does the organization already use?
- [ ] Can we hire the required expertise?
- [ ] What is the learning cost?

### Existing Application

- [ ] Is this greenfield?
- [ ] How large is the existing codebase?
- [ ] Can migration happen incrementally?
- [ ] What existing CI/CD infrastructure must remain?

### Long Term

- [ ] Can the architecture scale?
- [ ] Can we add another platform?
- [ ] Can platform-specific code remain isolated?
- [ ] Can we test shared business logic independently?
- [ ] Can the architecture evolve without a rewrite?

If these questions cannot be answered, the technology decision isn't ready.

---

# 🧭 The Final Decision Map

A simplified map looks like this:

```text
                         START
                           │
                           ▼
                  Single Platform?
                     /          \
                   YES           NO
                    │             │
                    ▼             ▼
              Native First    Multiple Targets
                                  │
                                  ▼
                         What should be shared?
                           /             \
                         UI              Logic
                         │                 │
                         ▼                 ▼
                  Shared UI Needed?   Native UI Needed?
                     /     \             /       \
                   YES      NO         YES        NO
                    │        │          │          │
                    ▼        ▼          ▼          ▼
              Flutter /   Native   KMP +      Compare
              RN / CMP     + KMP   Native UI   Alternatives
```

Where:

```text
CMP = Compose Multiplatform
RN  = React Native
KMP = Kotlin Multiplatform
```

The tree is intentionally simplified.

Real architecture requires context.

---

# So, Which One Should You Choose?

There is no single answer.

But we can make the recommendation more concrete.

### Choose Native Android when:

```text
Android is the primary or only target
+
Deep Android integration matters
+
Hardware/platform capabilities dominate
```

### Consider Flutter when:

```text
Android + iOS
+
Highly shared UI
+
Consistent visual experience
+
Dart / Flutter expertise
```

### Consider React Native when:

```text
Android + iOS
+
Shared React application model
+
Strong JavaScript / TypeScript organization
```

### Consider Compose Multiplatform when:

```text
Android + iOS / Desktop
+
Kotlin expertise
+
Shared Compose UI
+
Shared application logic
```

### Consider KMP when:

```text
Android + iOS
+
Meaningful shared business logic
+
Native UI is important
+
Platform-specific integration matters
+
Incremental adoption is valuable
```

These are not rules.

They are starting points.

---

# ⭐ The Architectural Principle

The strongest conclusion from this chapter is not:

> **"KMP is better than Flutter."**

It is not:

> **"Native Android is better than cross-platform."**

And it is not:

> **"Compose Multiplatform is the future."**

The better conclusion is:

> **Choose the smallest abstraction that solves the real problem.**

If you only need Android:

```text
Native Android
```

If you need shared UI:

```text
Choose a shared UI technology
```

If you need shared business logic but native UI:

```text
Kotlin Multiplatform
```

If you need both:

```text
KMP + Compose Multiplatform
```

Architecture should follow the requirement.

Not the other way around.

---

# 🧠 The Final Mental Model

Keep this model in mind whenever you evaluate a cross-platform technology:

```text
                  BUSINESS
                     │
                     ▼
               PRODUCT NEEDS
                     │
                     ▼
              PLATFORM NEEDS
                     │
                     ▼
             SHARING BOUNDARY
                     │
          ┌──────────┴──────────┐
          ▼                     ▼
       Shared                Native
       Code                  Code
          │                     │
          └──────────┬──────────┘
                     ▼
                TECHNOLOGY
```

The technology should be the **result** of the architectural decision.

It should not be the starting point.

---

# Final Chapter 3 Summary

We started with five very different approaches:

| Technology | Core Philosophy |
|------------|-----------------|
| **Native Android** | Maximize Android platform control |
| **Flutter** | Share the application and UI |
| **React Native** | Share a React-based application model |
| **Compose Multiplatform** | Share Kotlin Compose UI |
| **Kotlin Multiplatform** | Share the parts that genuinely belong together |

The deeper difference is not syntax.

It is **where each technology draws the boundary between shared and platform-specific code**.

```text
Native Android
      │
      ▼
Platform Boundary
      │
      ▼
Android

Flutter
      │
      ▼
Large Shared Application Boundary
      │
      ▼
Android + iOS

React Native
      │
      ▼
Shared React Application Boundary
      │
      ▼
Android + iOS

Compose Multiplatform
      │
      ▼
Shared Kotlin UI + Logic Boundary
      │
      ▼
Android + iOS + Desktop

Kotlin Multiplatform
      │
      ▼
Selective Sharing Boundary
      │
      ├── Shared Logic
      ├── Shared Data
      ├── Shared Networking
      └── Optional Shared UI
```

And that brings us to the central idea of this entire chapter:

> [!IMPORTANT]
> **Cross-platform development is not fundamentally about writing one codebase. It is about deciding where one codebase creates more value than multiple implementations.**

That is the decision an architect needs to make.

---

# 📌 Chapter 3 — Final Takeaways

- There is no universally best mobile technology.
- Technology selection should begin with product requirements.
- Native Android remains the strongest choice for Android-only and deeply Android-specific products.
- Flutter is compelling when highly shared UI and application code are the priority.
- React Native is particularly attractive for organizations with strong React and TypeScript expertise.
- Compose Multiplatform is compelling for Kotlin teams that want to share Compose-based UI.
- Kotlin Multiplatform provides the most flexible sharing boundary because UI sharing is optional.
- Native UI and shared business logic can coexist.
- Platform-specific code is not a failure of cross-platform architecture.
- Code-sharing percentage should never be the primary success metric.
- Business value matters more than lines of shared code.
- Existing applications should be evaluated differently from greenfield projects.
- Long-term maintenance matters more than initial development speed.
- The best architecture is the one whose trade-offs match the product.

---

## 🚀 The Question We Carry Forward

The next chapters will move from **technology selection** into **Kotlin Multiplatform itself**.

We now know why KMP exists.

We understand how it differs from Native Android, Flutter, React Native, and Compose Multiplatform.

We understand that KMP is not about replacing native development.

The next question is therefore much more practical:

> **How does Kotlin Multiplatform actually work under the hood?**

We'll start with the foundation:

```text
Kotlin Source Code
        │
        ▼
Common Source Sets
        │
        ▼
Platform Targets
        │
        ├──────────────┐
        ▼              ▼
     Android           iOS
        │              │
        ▼              ▼
     Native           Native
     Binary           Binary
```

Understanding this model is the key to everything that follows.

---

## 📚 Official References

- [Kotlin Multiplatform Documentation](https://kotlinlang.org/docs/multiplatform/)
- [Kotlin Multiplatform Supported Platforms](https://kotlinlang.org/docs/multiplatform/supported-platforms.html)
- [Kotlin Multiplatform Project Structure](https://kotlinlang.org/docs/multiplatform/multiplatform-discover-project.html)
- [Kotlin Multiplatform — Get Started](https://kotlinlang.org/docs/mpp-get-started.html)
- [Kotlin Multiplatform and Flutter](https://kotlinlang.org/docs/multiplatform/kotlin-multiplatform-flutter.html)
- [Kotlin Multiplatform and React Native](https://kotlinlang.org/docs/multiplatform/kotlin-multiplatform-react-native.html)
- [Native vs Cross-Platform App Development](https://kotlinlang.org/docs/native-and-cross-platform.html)

> **Author's Note**
>
> Technology evolves. The architectural principles in this chapter are intended to remain useful even as frameworks, libraries, and platform APIs change.
````
