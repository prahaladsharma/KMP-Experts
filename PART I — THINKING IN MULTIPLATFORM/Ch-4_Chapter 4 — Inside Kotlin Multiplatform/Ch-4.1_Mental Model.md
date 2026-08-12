# Chapter 4 — Inside Kotlin Multiplatform

## Part 1 — Mental Model

> **Before writing your first `commonMain` class, you need to understand what Kotlin Multiplatform is actually trying to do.**

Kotlin Multiplatform becomes much easier to understand once we stop thinking about it as a framework that "runs one application everywhere."

That mental model creates confusion very quickly.

A better way to think about KMP is this:

> **Kotlin Multiplatform lets us build a shared part of an application while allowing each platform to remain a real native application.**

That single idea explains most of the architecture that follows.

---

# 1. The Simplest KMP Mental Model

Imagine we have two applications:

```text
             PRODUCT
                │
       ┌────────┴────────┐
       ▼                 ▼
    Android               iOS
       │                 │
       ▼                 ▼
   Native App         Native App
       │                 │
       └────────┬────────┘
                ▼
          Shared KMP Code
```

There are still two applications.

Android is still Android.

iOS is still iOS.

What changes is that some code no longer needs to be implemented twice.

For example:

```text
                 Shared KMP
                    │
       ┌────────────┼────────────┐
       ▼            ▼            ▼
    Domain       Network       Data
     Logic        Layer        Layer
       │            │            │
       └────────────┼────────────┘
                    ▼
             Android + iOS
```

The shared module becomes a common engineering layer between the platform applications.

---

# 2. KMP Is About Sharing Code, Not Hiding Platforms

This is perhaps the most important concept in the entire KMP journey.

A beginner may imagine:

```text
              KMP
               │
               ▼
        One Universal App
               │
       ┌───────┴───────┐
       ▼               ▼
    Android            iOS
```

That is not the best mental model.

Instead, think:

```text
       Android Application          iOS Application
                │                         │
                │                         │
                └──────────┬──────────────┘
                           ▼
                    Shared KMP Layer
```

The platforms remain visible.

Their differences remain visible.

Their native APIs remain available.

KMP simply gives us a mechanism for moving genuinely common code into a shared implementation.

---

# 3. The Three-Layer Mental Model

For most applications, it helps to divide the architecture into three broad areas.

```text
┌────────────────────────────────────────────┐
│              Presentation                  │
│                                            │
│        Android UI      |      iOS UI       │
└───────────────────┬────────────────────────┘
                    │
                    ▼
┌────────────────────────────────────────────┐
│               Shared Core                  │
│                                            │
│  Domain | Business Rules | Data | Network │
└───────────────────┬────────────────────────┘
                    │
                    ▼
┌────────────────────────────────────────────┐
│            Platform Capabilities           │
│                                            │
│ Android APIs           iOS APIs            │
└────────────────────────────────────────────┘
```

This is not a mandatory KMP architecture.

It is a useful mental model.

It tells us that not every layer has the same reason to be shared.

---

# 4. What Usually Belongs in Shared Code?

Let's take a typical business application.

It may contain:

```text
Authentication
Product Catalog
Orders
Payments
Pricing
Validation
Caching
Networking
Database
Synchronization
Analytics Rules
```

A large portion of this logic may have exactly the same meaning on Android and iOS.

For example:

```text
If cart total > 5000
apply 10% discount.
```

There is nothing Android-specific about that rule.

There is nothing iOS-specific about it either.

So duplicating it may not provide any product value.

This is a natural candidate for shared code.

---

# 5. What Usually Remains Platform-Specific?

Now consider:

```text
Android Back Button
iOS Navigation Gesture
Android Notification APIs
Apple Push Notification APIs
Android Bluetooth APIs
Apple Bluetooth APIs
Android Activity
iOS UIViewController
```

These concepts belong to different operating systems.

Trying to force them into identical implementations can make the architecture harder to understand.

A better model is:

```text
              Shared Business Logic
                       │
          ┌────────────┴────────────┐
          ▼                         ▼
   Android-specific            iOS-specific
       code                       code
```

The objective is not to eliminate platform code.

The objective is to **put platform code where it belongs**.

---

# 6. Common Does Not Mean Identical

This distinction is subtle but important.

Suppose an application has authentication.

The business requirement may be:

```text
Authenticate user
      │
      ▼
Create session
      │
      ▼
Store credentials securely
```

The first and second steps may be completely common.

The third step may require platform-specific security APIs.

So the architecture could become:

```text
                 Authentication
                       │
             ┌─────────┴─────────┐
             ▼                   ▼
       Shared session       Platform security
             │              ┌──────┴──────┐
             │              ▼             ▼
             │          Android          iOS
             │
             └──────────────┬──────────────┘
                            ▼
                       Final Result
```

The feature is shared.

The implementation is not necessarily 100% shared.

That is perfectly normal.

---

# 7. The Most Useful KMP Question

Whenever you are deciding whether something belongs in `commonMain`, ask:

> **Does this code represent a platform-independent product rule, or does it represent a platform capability?**

For example:

| Code                                | Likely Location     |
| ----------------------------------- | ------------------- |
| Calculate discount                  | `commonMain`        |
| Validate email                      | `commonMain`        |
| Calculate order total               | `commonMain`        |
| Map API response to domain model    | `commonMain`        |
| Authentication state                | `commonMain`        |
| Android notification implementation | `androidMain`       |
| iOS notification implementation     | `iosMain`           |
| Android Activity                    | Android application |
| SwiftUI screen                      | iOS application     |
| Android Bluetooth API               | `androidMain`       |
| iOS Bluetooth API                   | `iosMain`           |

The exact boundary depends on the architecture.

But the principle remains:

> **Business meaning tends to be common. Platform capability tends to be platform-specific.**

---

# 8. KMP Is a Compilation Model Too

So far, we've talked about architecture.

But there is another important question:

> **How can the same Kotlin source code become usable on different platforms?**

This is where KMP becomes more interesting technically.

Kotlin is not simply interpreted by a universal KMP runtime.

Kotlin code is compiled for the target platform.

Conceptually:

```text
                 Kotlin Source
                       │
                       ▼
              Kotlin Multiplatform
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
       Android         iOS         JVM
          │            │            │
          ▼            ▼            ▼
       Android      Native        JVM
       Runtime      Binary       Runtime
```

The exact compilation pipeline depends on the target.

The important mental model is:

> **Shared Kotlin source is compiled into platform-appropriate output.**

This is fundamentally different from imagining one universal binary being copied to every platform.

---

# 9. One Source Does Not Mean One Binary

This is another common misunderstanding.

Suppose we write:

```kotlin
class PriceCalculator {
    fun calculate(price: Double): Double {
        return price * 0.9
    }
}
```

The source may be shared.

But Android and iOS don't simply execute the same binary.

Conceptually:

```text
             PriceCalculator.kt
                     │
              ┌──────┴──────┐
              ▼             ▼
          Android          iOS
              │             │
              ▼             ▼
        Platform output  Native output
```

This is one of the reasons KMP can preserve a native execution model while still sharing source code.

---

# 10. The Source Set Mental Model

KMP organizes code using **source sets**.

A simplified project might look like:

```text
shared/
└── src/
    ├── commonMain/
    ├── androidMain/
    └── iosMain/
```

Think of them as three levels of responsibility.

### `commonMain`

```text
Code that can be shared
```

### `androidMain`

```text
Code that requires Android
```

### `iosMain`

```text
Code that requires iOS
```

Visually:

```text
                 shared
                    │
          ┌─────────┴─────────┐
          ▼                   ▼
     commonMain          Platform Source Sets
                              │
                     ┌────────┴────────┐
                     ▼                 ▼
                androidMain         iosMain
```

This structure is one of the foundations of KMP architecture.

---

# 11. `commonMain` Is Not a Trash Folder

This is an architectural rule worth remembering.

When teams first adopt KMP, there is sometimes a temptation to move everything into `commonMain`.

That can lead to something like:

```text
commonMain/
├── AndroidWorkaround.kt
├── IOSWorkaround.kt
├── PlatformHack.kt
├── AnotherPlatformHack.kt
└── EverythingElse.kt
```

The result is technically multiplatform.

Architecturally, it is a mess.

`commonMain` should contain code whose behavior genuinely makes sense across the supported platforms.

A good shared module should communicate product concepts.

For example:

```text
commonMain/
├── domain/
├── data/
├── networking/
├── validation/
└── synchronization/
```

The exact package structure is a design decision.

The principle is more important than the folders.

---

# 12. Think in Terms of Responsibilities

Instead of asking:

> "Can this class compile on iOS?"

Ask:

> "Should this responsibility be shared?"

For example:

```text
UserRepository
```

may contain both:

```text
Shared responsibility
+
Platform-specific implementation
```

A better design might separate them.

```text
                 UserRepository
                       │
                ┌──────┴──────┐
                ▼             ▼
           Shared logic   Platform detail
```

This is where normal software architecture becomes extremely important.

KMP doesn't replace architectural principles.

It makes them more visible.

---

# 13. The Shared Core Mental Model

A mature KMP application can be thought of as a **shared core surrounded by native applications**.

```text
                  ┌─────────────────┐
                  │ Android App     │
                  │                 │
                  │ Native UI       │
                  │ Android APIs    │
                  └────────┬────────┘
                           │
                           ▼
                 ┌───────────────────┐
                 │                   │
                 │    Shared KMP     │
                 │       Core        │
                 │                   │
                 │ Domain            │
                 │ Business Rules    │
                 │ Data              │
                 │ Networking        │
                 │ State             │
                 └────────┬──────────┘
                          │
                          ▼
                  ┌─────────────────┐
                  │ iOS App         │
                  │                 │
                  │ Native UI       │
                  │ Apple APIs      │
                  └─────────────────┘
```

The diagram isn't suggesting that Android directly calls through to iOS.

It simply represents the shared module being consumed by both platform applications.

---

# 14. Why This Model Is Powerful

Imagine a business rule changes.

Before KMP:

```text
Requirement
    │
    ├───────────────┐
    ▼               ▼
Android Logic     iOS Logic
    │               │
    ▼               ▼
Android Tests     iOS Tests
```

After moving the rule into shared code:

```text
Requirement
      │
      ▼
Shared KMP Logic
      │
 ┌────┴────┐
 ▼         ▼
Android    iOS
```

The business rule now has one implementation.

The native applications consume it.

This can reduce one of the most expensive forms of cross-platform duplication:

> **Different implementations of the same business decision.**

---

# 15. KMP Is Not a Replacement for Architecture

KMP gives us mechanisms.

It does not automatically give us architecture.

You can build:

```text
Bad KMP
```

just as easily as:

```text
Bad Android
Bad iOS
Bad Flutter
Bad React Native
```

You still need to decide:

* Where business rules live
* Where state is managed
* Where dependencies enter
* Where platform APIs are accessed
* How data flows
* How errors are represented
* How testing is organized
* How modules depend on one another

KMP simply gives you another dimension:

```text
Shared
vs
Platform-specific
```

---

# 16. KMP and Clean Architecture

If you already understand Clean Architecture, KMP should not require you to throw it away.

Instead, think about platform boundaries.

A possible architecture:

```text
┌───────────────────────────────────────┐
│              Android UI               │
└───────────────────┬───────────────────┘
                    │
                    ▼
┌───────────────────────────────────────┐
│          Shared Presentation          │
└───────────────────┬───────────────────┘
                    │
                    ▼
┌───────────────────────────────────────┐
│              Domain                   │
│                                       │
│ Use Cases | Models | Business Rules   │
└───────────────────┬───────────────────┘
                    │
                    ▼
┌───────────────────────────────────────┐
│                Data                   │
│                                       │
│ Repository | Network | Cache          │
└───────────────────┬───────────────────┘
                    │
                    ▼
            Platform-specific
             implementations
```

The same shared core can then be consumed by iOS.

The exact architecture can vary, but the separation of responsibilities remains useful.

---

# 17. The Platform Boundary Is a Feature

Developers sometimes see platform-specific code as something they need to eliminate.

In KMP, the platform boundary is actually useful.

It tells us:

```text
             Common
                │
       ┌────────┴────────┐
       ▼                 ▼
 Platform-independent   Platform-specific
       │                 │
       ▼                 ▼
 Business rules       OS capabilities
```

The boundary gives us freedom.

If Android needs something special:

```text
Android
```

If iOS needs something special:

```text
iOS
```

The shared code doesn't need to pretend those differences don't exist.

---

# 18. A Useful Analogy

Think about a restaurant.

There is a common menu:

```text
Recipes
Ingredients
Pricing
Food Preparation Rules
```

But each restaurant location may have:

```text
Local Equipment
Local Regulations
Local Suppliers
Local Operations
```

You don't want every restaurant reinventing the recipe.

But you also don't want to force every location to have exactly the same kitchen.

KMP works similarly.

```text
              Shared Product Rules
                     │
        ┌────────────┴────────────┐
        ▼                         ▼
     Android                   iOS
     Kitchen                   Kitchen
```

The recipe is shared.

The kitchen remains platform-specific.

---

# 19. What KMP Is Really Optimizing

KMP is not primarily optimizing for:

```text
Minimum Number of Files
```

or:

```text
Maximum Percentage of Shared Code
```

It is optimizing for something more valuable:

> **A single implementation where the business behavior is genuinely the same, without giving up native control where the platforms differ.**

That leads to a useful equation:

```text
Shared Business Logic
        +
Native Platform Freedom
        =
KMP's Core Value
```

If that equation matches your product, KMP becomes interesting.

---

# 20. The Sharing Spectrum

There isn't a single correct amount of sharing.

Think of it as a spectrum:

```text
Less Sharing                                      More Sharing
     │                                                   │
     ▼                                                   ▼

Native UI     Shared Logic     Shared Data     Shared UI
     │              │               │              │
     ▼              ▼               ▼              ▼
 Android +       Android +       Android +      Compose MP
 iOS UI          iOS UI          iOS UI           UI
```

A team can choose where to stop.

For some products:

```text
Native UI + Shared Logic
```

is ideal.

For others:

```text
Shared UI + Shared Logic
```

may be better.

Neither is automatically superior.

---

# 21. The Most Important Mental Shift

Traditional cross-platform thinking often starts with:

> **"How can we make Android and iOS use the same code?"**

KMP encourages a different question:

> **"Which code should Android and iOS share?"**

That small change in wording has a major architectural impact.

The first question starts with implementation.

The second starts with responsibility.

---

# 22. KMP in One Diagram

If you remember only one diagram from this chapter, remember this one:

```text
                         APPLICATION
                              │
               ┌──────────────┴──────────────┐
               ▼                             ▼
        Android Application             iOS Application
               │                             │
               │                             │
               └──────────────┬──────────────┘
                              ▼
                     ┌─────────────────┐
                     │   Shared KMP    │
                     │      Core       │
                     ├─────────────────┤
                     │ Domain          │
                     │ Business Rules  │
                     │ Data            │
                     │ Networking      │
                     │ Shared State    │
                     └────────┬────────┘
                              │
                     ┌────────┴────────┐
                     ▼                 ▼
                Android APIs        iOS APIs
```

The top represents applications.

The middle represents shared product knowledge.

The bottom represents platform capabilities.

That is the mental model we will carry through the rest of the book.

---

# 23. A More Advanced Mental Model

As applications become larger, the architecture can be visualized as three zones:

```text
┌──────────────────────────────────────────────────────┐
│                 PLATFORM EXPERIENCE                  │
│                                                      │
│   Android UI                         iOS UI          │
│   Android Lifecycle                  iOS Lifecycle   │
│   Android APIs                       iOS APIs        │
└───────────────────────┬──────────────────────────────┘
                        │
                        ▼
┌──────────────────────────────────────────────────────┐
│                    SHARED CORE                        │
│                                                      │
│   Domain                                             │
│   Business Rules                                     │
│   State                                              │
│   Networking                                         │
│   Persistence                                        │
│   Synchronization                                    │
└───────────────────────┬──────────────────────────────┘
                        │
                        ▼
┌──────────────────────────────────────────────────────┐
│                   PLATFORM BRIDGE                    │
│                                                      │
│   Android-specific implementations                   │
│   iOS-specific implementations                       │
└──────────────────────────────────────────────────────┘
```

The names of these zones are conceptual.

The actual implementation depends on the project.

But this model helps us reason about responsibility.

---

# 24. What KMP Does Not Promise

It is equally important to understand what KMP does **not** promise.

KMP does not mean:

* Every API is available everywhere.
* Every library works identically on every target.
* Android knowledge automatically makes someone an iOS expert.
* Platform-specific code disappears.
* Every application should share its UI.
* Every application should use Compose Multiplatform.
* Migration is free.
* Architecture no longer matters.

KMP is a tool.

The quality of the resulting architecture still depends on engineering decisions.

---

# 25. The KMP Mindset

A good KMP developer starts thinking in terms of boundaries.

Instead of:

```text
Android developer
vs
iOS developer
```

the conversation becomes:

```text
Shared responsibility
vs
Platform responsibility
```

Instead of:

```text
How much code can I move?
```

the question becomes:

```text
What code has the same meaning on both platforms?
```

Instead of:

```text
Can I make Android and iOS identical?
```

the question becomes:

```text
Where should Android and iOS intentionally remain different?
```

This is the mindset shift that separates simply using KMP from designing a good multiplatform architecture.

---

# 26. A Simple Rule to Carry Forward

When you're unsure whether something belongs in shared code, use this rule:

> [!IMPORTANT]
> **If the requirement comes from the business, it is a strong candidate for shared code. If the requirement comes from the operating system, it is a strong candidate for platform-specific code.**

There will be exceptions.

There will be gray areas.

But this rule is an excellent starting point.

---

# 27. Chapter Takeaways

> [!TIP]
> **KMP is best understood as a selective code-sharing architecture, not as a "write once, run everywhere" framework.**

Remember these ideas:

1. **Android and iOS remain native applications.**
2. **KMP provides a shared Kotlin layer between them.**
3. **Not everything should be shared.**
4. **Business rules are strong candidates for shared code.**
5. **Platform capabilities usually remain platform-specific.**
6. **`commonMain` is for genuinely common behavior.**
7. **`androidMain` and `iosMain` provide platform-specific implementations.**
8. **Shared source code is compiled for the target platform.**
9. **KMP does not require shared UI.**
10. **Compose Multiplatform can be added when shared UI makes sense.**
11. **The goal is not maximum code sharing.**
12. **The goal is meaningful code sharing without losing native capabilities.**

---

# Closing Thought

The easiest way to misunderstand Kotlin Multiplatform is to think of it as a way to avoid writing Android and iOS applications.

The better way to understand it is this:

```text
                 Android
                    │
                    │
             ┌──────▼──────┐
             │             │
             │ Shared      │
             │ Product     │
             │ Knowledge   │
             │             │
             └──────▲──────┘
                    │
                    │
                    iOS
```

Android still exists.

iOS still exists.

Their differences still matter.

But the business knowledge that doesn't need to be different no longer has to be implemented twice.

That is the foundation of Kotlin Multiplatform.

And before we write a single line of advanced KMP code, we need to understand **how Kotlin actually reaches those different platforms**.

That is where the next part begins.
