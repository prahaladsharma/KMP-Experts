# Chapter 10 — Project Structure

## Part 1 — Parts

A KMP project can share Kotlin code across platforms, but sharing code is only one part of the architectural problem.

The next question is:

> **How should that shared code be divided?**

A project that starts with:

```text
shared/
    everything
```

may work for a small application.

As the application grows, that structure becomes difficult to understand, test, own, and change.

A better approach is to think of a KMP project as a collection of **meaningful parts**.

```text
KMP Project
    │
    ├── Applications
    ├── Features
    ├── Core
    ├── Platform
    ├── Integrations
    └── Build / Tooling
```

Each part should have a reason to exist.

> [!IMPORTANT]
> **Project structure is not about putting files into folders. It is about defining boundaries around responsibilities, dependencies, ownership, and change.**

---

# 1. What Is a "Part"?

In this chapter, a **part** means a meaningful architectural area of the project.

A part can be:

- an application
- a feature
- a domain capability
- a core library
- a platform implementation
- an external integration
- a testing module
- build logic
- developer tooling

For example:

```text
MyKmpProject
│
├── applications
├── features
├── core
├── platform
├── integrations
└── build-logic
```

These are not arbitrary folders.

They represent different architectural responsibilities.

---

# 2. Why Project Structure Matters

Imagine a project with:

```text
src/
├── User.kt
├── Order.kt
├── Payment.kt
├── Api.kt
├── Database.kt
├── LoginScreen.kt
├── AndroidScanner.kt
├── IosScanner.kt
├── Utils.kt
└── Helper.kt
```

Everything is technically in one place.

But several questions become difficult:

```text
Who owns Order?
Can Payment use User directly?
Where should API code live?
Can domain code access Android?
Where does iOS implementation belong?
Which code is safe to reuse?
```

Project structure should make these questions easier to answer.

---

# 3. Structure Communicates Architecture

Consider:

```text
features/orders/
```

A developer immediately expects:

```text
Order-related functionality
```

Consider:

```text
core/security/
```

The expectation becomes:

```text
Shared security capability
```

Consider:

```text
platform/scanner/
```

The expectation becomes:

```text
Platform-specific scanner integration
```

Good structure communicates intent before the developer reads the code.

---

# 4. A KMP Project Has Multiple Dimensions

KMP project structure is influenced by several dimensions:

```text
                 KMP PROJECT
                      │
       ┌──────────────┼──────────────┐
       ▼              ▼              ▼
   Business         Platform        Product
   Features         Targets         Apps
       │              │              │
       └──────────────┼──────────────┘
                      ▼
                   Modules
                      │
                      ▼
                 Source Sets
```

This is why KMP structure can initially feel more complicated than a traditional Android application.

There are multiple boundaries to represent.

---

# 5. The Main Parts of a KMP Project

A mature KMP project commonly contains some combination of:

| Part | Responsibility |
|---|---|
| Applications | Platform application entry points |
| Features | Business capabilities |
| Domain | Business rules and models |
| Data | Repositories and data access |
| Core | Shared cross-cutting capabilities |
| Platform | Platform-specific capabilities |
| Integrations | External systems and SDKs |
| Testing | Shared testing utilities |
| Build Logic | Gradle conventions |
| Tooling | Developer productivity tools |

Not every project needs every part.

---

# 6. The Simplest KMP Structure

A small project may start with:

```text
MyApp/
├── androidApp/
├── iosApp/
└── shared/
```

The idea is simple:

```text
Android Application
       │
       ▼
    Shared
       ▲
       │
iOS Application
```

This is perfectly reasonable for a small application.

The problem starts when `shared` becomes responsible for everything.

---

# 7. The "Everything in Shared" Problem

A large shared module may eventually look like:

```text
shared/
├── auth/
├── orders/
├── payments/
├── profile/
├── network/
├── database/
├── analytics/
├── security/
├── notifications/
├── utils/
└── platform/
```

Technically:

```text
It works.
```

Architecturally:

```text
Everything depends on everything.
```

The shared module becomes a container rather than a meaningful boundary.

---

# 8. From Shared Module to Meaningful Parts

A growing project can evolve into:

```text
MyApp/
├── androidApp/
├── iosApp/
│
├── features/
│   ├── auth/
│   ├── orders/
│   ├── payments/
│   └── profile/
│
├── core/
│   ├── network/
│   ├── database/
│   ├── security/
│   └── observability/
│
└── platform/
    ├── scanner/
    ├── storage/
    └── notifications/
```

Now the project structure communicates responsibility.

---

# 9. Parts Are Not Layers

This distinction is important.

A **layer** answers:

> What kind of responsibility is this?

A **part** answers:

> What meaningful area of the system does this belong to?

For example:

```text
Feature: Orders
```

may contain:

```text
Presentation
Domain
Data
```

So:

```text
Part
  └── Layers
```

is often a useful mental model.

---

# 10. Parts and Layers Together

Consider:

```text
features/orders/
│
├── presentation/
├── domain/
└── data/
```

Here:

```text
Part = Orders
```

and:

```text
Layers = Presentation / Domain / Data
```

This distinction becomes important as the project grows.

---

# 11. Part 1 Mental Model

Think about the project as:

```text
                 PROJECT
                    │
        ┌───────────┼───────────┐
        ▼           ▼           ▼
     FEATURE       CORE      PLATFORM
        │           │           │
     Layers       Layers      Adapters
```

The project is divided into meaningful parts first.

Then each part can have its own internal organization.

---

# 12. Applications

The application is the final composition.

For example:

```text
apps/
├── androidApp/
└── iosApp/
```

The application is responsible for:

- startup
- dependency composition
- platform configuration
- navigation setup
- lifecycle
- application-level configuration
- assembling features

It should not become the owner of business logic.

---

# 13. Application as Composition Root

A useful mental model:

```text
                 Application
                      │
             Composition Root
                      │
        ┌─────────────┼─────────────┐
        ▼             ▼             ▼
     Features        Core        Platform
```

The application decides:

```text
Which implementation?
Which configuration?
Which environment?
Which dependencies?
```

This is different from implementing business rules.

---

# 14. Android Application Part

An Android application may contain:

```text
androidApp/
├── src/
│   └── main/
│       ├── AndroidManifest.xml
│       └── kotlin/
│           └── MainActivity.kt
└── build.gradle.kts
```

Its job is primarily Android-specific application composition.

For example:

```kotlin
class MainActivity : ComponentActivity() {

    override fun onCreate(
        savedInstanceState: Bundle?
    ) {
        super.onCreate(savedInstanceState)

        // Compose application
        // dependency composition
        // Android-specific setup
    }
}
```

The activity should not become the location for shared business rules.

---

# 15. iOS Application Part

The iOS application may contain:

```text
iosApp/
├── ContentView.swift
├── AppDelegate.swift
└── ...
```

It can consume the shared KMP framework.

Conceptually:

```text
iOS Application
       │
       ▼
KMP Shared Modules
       │
       ▼
Business Capabilities
```

The iOS application remains responsible for iOS-specific lifecycle and composition.

---

# 16. Features

A feature represents a meaningful business capability.

Examples:

```text
Authentication
Orders
Payments
Profile
Inventory
Search
Notifications
```

A feature should answer:

> **What business capability does this part of the system own?**

For example:

```text
features/orders/
```

owns order-related behavior.

---

# 17. Feature Structure

A feature can contain:

```text
features/orders/
├── src/
│   ├── commonMain/
│   │   └── kotlin/
│   │       └── orders/
│   │           ├── presentation/
│   │           ├── domain/
│   │           └── data/
│   │
│   ├── commonTest/
│   ├── androidMain/
│   └── iosMain/
│
└── build.gradle.kts
```

The exact structure can vary.

The important point is ownership.

---

# 18. Feature Ownership

If the feature is:

```text
Orders
```

then the feature may own:

```text
Order
OrderRepository
OrderUseCase
OrderState
OrderApi
OrderMapper
OrderValidation
```

as long as those responsibilities are specific to Orders.

This creates a natural boundary.

---

# 19. What Should Not Automatically Go Into a Feature?

Avoid putting truly shared capabilities inside one feature just because that feature uses them first.

For example:

```text
orders/
└── NetworkClient
```

If Payments and Profile also need the same infrastructure, NetworkClient probably belongs elsewhere.

The question is:

> **Who owns the behavior?**

Not:

> **Who used it first?**

---

# 20. Core

Core contains capabilities shared across multiple parts of the system.

Examples:

```text
core/
├── network/
├── database/
├── security/
├── observability/
├── serialization/
└── testing/
```

Core should contain stable, reusable capabilities.

It should not become:

```text
random shared code
```

---

# 21. What Makes Something Core?

A capability is a candidate for core when:

```text
Multiple parts need it
+
Its responsibility is cross-cutting
+
Its API can be stable
+
It does not belong to one business feature
```

For example:

```text
NetworkClient
```

may be core.

But:

```text
OrderPricingCalculator
```

usually belongs to Orders.

---

# 22. Core Is Not "Common"

These names are often confused:

```text
common/
shared/
core/
utils/
```

They do not mean the same thing.

A better definition:

```text
Core
=
Intentional shared capabilities
```

rather than:

```text
Core
=
Things we don't know where to put
```

---

# 23. The Core Dumping Ground

A common anti-pattern:

```text
core/
├── DateUtils.kt
├── StringUtils.kt
├── OrderHelper.kt
├── PaymentHelper.kt
├── UserHelper.kt
├── Network.kt
├── RandomThing.kt
└── TemporaryFix.kt
```

This creates a dependency magnet.

Eventually:

```text
Everything → core
```

The module becomes impossible to reason about.

---

# 24. Better Core Structure

Prefer capability-based organization:

```text
core/
├── network/
├── security/
├── database/
├── observability/
└── testing/
```

Each part answers:

```text
What capability does this provide?
```

---

# 25. Platform

KMP allows common code to be shared while platform-specific code remains separate.

A platform part may contain:

```text
platform/
├── storage/
├── scanner/
├── notifications/
├── biometrics/
└── device/
```

These capabilities may require:

```text
Android SDK
iOS SDK
Native APIs
```

---

# 26. Platform Does Not Mean Android Only

A platform part may represent:

```text
Android
iOS
Desktop
Web
```

For example:

```text
platform/storage/
├── commonMain/
├── androidMain/
└── iosMain/
```

The platform boundary exists because the implementation differs.

---

# 27. Integrations

External systems deserve their own boundary.

Examples:

```text
integrations/
├── payments/
├── analytics/
├── identity/
├── maps/
└── messaging/
```

An integration protects the rest of the application from vendor-specific APIs.

---

# 28. Why Integrations Need Boundaries

Suppose the application uses:

```text
Payment SDK A
```

Without a boundary:

```text
Orders → SDK A
Payments → SDK A
Checkout → SDK A
Profile → SDK A
```

Now replacing the SDK becomes expensive.

With an integration boundary:

```text
Features
   │
   ▼
Payment Contract
   │
   ▼
Payment Integration
   │
   ▼
SDK A
```

The vendor is contained.

---

# 29. Testing as a Part

Testing is not simply a folder.

A mature project may contain:

```text
testing/
├── fake-network/
├── fake-database/
├── test-dispatchers/
├── fixtures/
└── assertions/
```

These utilities can be shared across modules.

But test helpers should remain test-oriented.

Do not accidentally make test infrastructure production dependencies.

---

# 30. Build Logic

As KMP projects grow, Gradle configuration can become repetitive.

A dedicated part can contain:

```text
build-logic/
├── kmp-library.gradle.kts
├── android-library.gradle.kts
├── testing.gradle.kts
└── publishing.gradle.kts
```

This allows modules to share build conventions.

---

# 31. Tooling

Large projects may also contain developer tooling:

```text
tools/
├── code-generation/
├── architecture-checks/
├── dependency-analysis/
└── reporting/
```

Tooling should be separate from application runtime code.

This keeps:

```text
Product Code
```

separate from:

```text
Developer Infrastructure
```

---

# 32. A Complete High-Level Structure

A mature KMP repository might look like:

```text
MyKmpProject/
│
├── apps/
│   ├── androidApp/
│   └── iosApp/
│
├── features/
│   ├── auth/
│   ├── orders/
│   ├── payments/
│   ├── profile/
│   └── search/
│
├── core/
│   ├── domain/
│   ├── network/
│   ├── database/
│   ├── security/
│   ├── observability/
│   └── testing/
│
├── platform/
│   ├── storage/
│   ├── scanner/
│   ├── notifications/
│   └── device/
│
├── integrations/
│   ├── payments/
│   ├── analytics/
│   └── identity/
│
├── build-logic/
│
├── tools/
│
├── gradle/
│
└── settings.gradle.kts
```

This is not a mandatory template.

It is a mental model.

---

# 33. Project Parts vs Gradle Modules

A part does not always have to be a separate Gradle module.

For example:

```text
features/orders/
```

can initially be a package inside:

```text
shared/
```

Later it may become:

```text
:features:orders
```

This is an important distinction.

```text
Architectural Boundary
        ↓
may become
        ↓
Gradle Module
```

when the benefits justify it.

---

# 34. Start with Logical Parts

Early in development:

```text
shared/
└── features/
    ├── orders/
    └── payments/
```

may be enough.

Later:

```text
:features:orders
:features:payments
```

can provide stronger isolation.

The architecture does not need to start fully modularized.

---

# 35. Packages Before Modules

A useful progression is:

```text
Package
   ↓
Clear Responsibility
   ↓
Repeated Need for Isolation
   ↓
Gradle Module
```

This avoids creating modules merely for organizational aesthetics.

---

# 36. When a Part Should Become a Module

Consider a separate module when you need:

- independent ownership
- dependency isolation
- build isolation
- independent testing
- reusable distribution
- platform separation
- clear public API
- independent evolution

For example:

```text
orders/
```

becoming:

```text
:features:orders
```

may make sense when Orders has its own team and meaningful boundaries.

---

# 37. When a Module Is Too Much

Avoid creating:

```text
:models
:interfaces
:constants
:helpers
:strings
```

simply because each contains a few files.

Module boundaries have costs:

```text
Gradle configuration
Build graph complexity
Navigation complexity
API management
Dependency management
```

A module should earn its existence.

---

# 38. The "One File, One Module" Mistake

This is an example of over-modularization:

```text
:order-model
:order-id
:order-status
:order-interface
:order-helper
```

The project may technically be modular.

But the architecture becomes harder to understand.

Prefer meaningful capabilities.

---

# 39. Part Boundaries Should Follow Change

A strong question is:

> **What changes together?**

If these usually change together:

```text
OrderApi
OrderRepository
OrderMapper
OrderUseCase
OrderState
```

they may belong to:

```text
Orders
```

If two pieces always change independently, a separate boundary may be useful.

---

# 40. Part Boundaries Should Follow Ownership

Another strong question:

> **Who owns this code?**

If:

```text
Orders Team
```

owns:

```text
Orders
```

then:

```text
features/orders/
```

is a natural boundary.

Ownership and architecture reinforce each other.

---

# 41. Part Boundaries Should Follow Business Capability

Business boundaries can be stronger than technical categories.

Compare:

```text
repositories/
models/
services/
controllers/
```

with:

```text
orders/
payments/
profile/
inventory/
```

The second structure makes it easier to understand business ownership.

---

# 42. Feature-Oriented vs Layer-Oriented Structure

Layer-oriented:

```text
shared/
├── presentation/
├── domain/
├── data/
└── network/
```

Feature-oriented:

```text
shared/
├── orders/
│   ├── presentation/
│   ├── domain/
│   └── data/
│
├── payments/
│   ├── presentation/
│   ├── domain/
│   └── data/
```

For larger products, feature-oriented organization often improves locality.

---

# 43. Why Feature Locality Matters

Suppose you need to change:

```text
Order Cancellation
```

With feature-oriented structure:

```text
features/orders/
```

contains most of the relevant code.

With layer-oriented structure:

```text
presentation/orders/
domain/orders/
data/orders/
```

the developer has to navigate multiple top-level areas.

Both can work.

The important principle is locality.

---

# 44. The Hybrid Structure

A scalable KMP project can combine both:

```text
features/
└── orders/
    ├── presentation/
    ├── domain/
    └── data/

core/
├── network/
├── security/
└── database/

platform/
├── storage/
└── notifications/
```

This gives:

```text
Business locality
+
Shared infrastructure boundaries
```

---

# 45. Parts and Source Sets

KMP introduces another dimension.

A module may contain:

```text
commonMain
commonTest
androidMain
androidUnitTest
iosMain
iosTest
```

Therefore:

```text
Module
   │
   ├── Common
   ├── Android
   └── iOS
```

The module is one architectural part.

Source sets represent platform-specific implementations inside that part.

---

# 46. Common Code as the Shared Contract

A typical module:

```text
orders/
├── commonMain/
├── androidMain/
└── iosMain/
```

can be viewed as:

```text
                 Orders
                   │
              commonMain
                   │
          ┌────────┴────────┐
          ▼                 ▼
     androidMain        iosMain
```

Common code contains what can be shared.

Platform code contains what cannot.

---

# 47. Parts and `expect` / `actual`

For platform-specific behavior:

```kotlin
expect class PlatformScanner {
    fun scan(): String
}
```

Android:

```kotlin
actual class PlatformScanner {
    actual fun scan(): String {
        // Android implementation
    }
}
```

iOS:

```kotlin
actual class PlatformScanner {
    actual fun scan(): String {
        // iOS implementation
    }
}
```

The architectural part remains:

```text
Scanner Capability
```

while implementation varies by platform.

---

# 48. Parts and Interfaces

An alternative is:

```kotlin
interface Scanner {
    suspend fun scan(): String
}
```

with implementations:

```text
AndroidScanner
IosScanner
```

This can be useful when dependency injection and multiple implementations are important.

The choice depends on the boundary.

---

# 49. Parts and Dependencies

Every part should have a deliberate dependency direction.

For example:

```text
Application
     ↓
Feature
     ↓
Core Contract
     ↑
Implementation
     ↓
Platform
```

The exact dependency graph depends on the architecture.

The important principle is:

> **A part should not depend on another part merely because the code is convenient to access.**

---

# 50. Parts and Dependency Ownership

For each dependency ask:

```text
Why does this part need it?
Who owns the dependency?
Can the dependency be narrower?
Is the dependency stable?
Could this dependency create a cycle?
```

These questions become increasingly important as the project grows.

---

# 51. Part Interfaces

A part should expose a small surface.

For example:

```text
features/orders
```

might expose:

```kotlin
interface Orders {
    suspend fun getOrder(
        id: OrderId
    ): Order
}
```

while keeping:

```text
OrderRepositoryImpl
OrderMapper
OrderApi
OrderDataSource
```

internal.

This is information hiding at the module level.

---

# 52. Public vs Internal

A module might contain:

```text
orders/
├── public/
│   └── Orders.kt
│
└── internal/
    ├── OrderRepositoryImpl.kt
    ├── OrderApi.kt
    └── OrderMapper.kt
```

Kotlin's:

```kotlin
internal
```

visibility can help enforce this boundary inside a module.

---

# 53. Part Contracts

A contract should represent:

```text
Capability
```

not:

```text
Implementation structure
```

Bad:

```kotlin
interface OrdersRepositoryWithRetrofitAndRoom
```

Better:

```kotlin
interface OrderRepository
```

The consumer should not care how the capability is implemented.

---

# 54. Part Boundaries and Models

Avoid exposing internal models unnecessarily.

For example:

```text
API DTO
```

should usually remain inside:

```text
data
```

and map into:

```text
Domain Model
```

This prevents external representations from becoming architecture-wide contracts.

---

# 55. Part Boundaries and DTO Leakage

Bad:

```text
Feature
   ↓
Backend DTO
```

Better:

```text
Feature
   ↓
Domain Model
   ▲
   │
Mapper
   ▲
DTO
```

The DTO belongs to the external data boundary.

---

# 56. Part Boundaries and UI Models

Likewise:

```text
UI
```

should not automatically consume:

```text
Database Entity
```

Prefer:

```text
Database Entity
      ↓
Domain Model
      ↓
UI State
```

Each part can evolve independently.

---

# 57. Part Boundaries and Security

Security-sensitive implementations should have strong boundaries.

For example:

```text
features/auth/
```

may depend on:

```text
core/security
```

but should not necessarily know:

```text
Keystore APIs
Keychain APIs
Encryption library internals
```

Those belong behind the security boundary.

---

# 58. Part Boundaries and Network

Similarly:

```text
Feature
```

should ideally not be tightly coupled to:

```text
Retrofit
OkHttp
Ktor
```

unless the architecture intentionally chooses that.

A boundary can be:

```text
Feature
   ↓
Repository
   ↓
Network Infrastructure
```

This makes network implementation replaceable.

---

# 59. Part Boundaries and Database

A feature should ideally depend on:

```text
Repository
```

rather than:

```text
SQLite
Room
SQLDelight
Core Data
```

directly.

The persistence implementation can remain behind the data boundary.

---

# 60. Part Boundaries and Platform SDKs

Avoid:

```kotlin
class OrderValidator {
    // Android SDK
}
```

when the validation rule is business logic.

Instead:

```text
commonMain
    ↓
OrderValidator

platform
    ↓
Android SDK
```

Platform dependencies should remain where they are genuinely required.

---

# 61. Part Boundaries and UI

A feature may have:

```text
common business logic
```

while UI remains:

```text
Android Compose
iOS SwiftUI
```

For example:

```text
Orders
├── commonMain
│   ├── domain
│   └── state
│
├── androidMain
│   └── Compose UI
│
└── iosMain
    └── SwiftUI bridge
```

This is a valid KMP architecture.

---

# 62. Part Boundaries and Shared UI

Alternatively, Compose Multiplatform may be used:

```text
Orders
└── commonMain
    └── Compose UI
```

with:

```text
Android
iOS
Desktop
```

consuming the same UI.

The part structure can remain the same.

Only the presentation sharing strategy changes.

---

# 63. Parts Should Reflect Change Frequency

Consider:

```text
Feature UI
```

may change frequently.

```text
Security Contract
```

may need to remain stable.

Putting them behind appropriate boundaries allows:

```text
High-change code
```

to evolve without unnecessarily disturbing:

```text
High-stability code
```

---

# 64. Parts Should Reflect Stability

A useful mental model:

```text
Frequently Changing
       │
       ▼
Features
       │
       ▼
Stable Contracts
       │
       ▼
Core
       │
       ▼
Platform Infrastructure
```

The exact order varies.

The principle is to avoid making stable areas depend directly on volatile implementation details.

---

# 65. Parts and Coupling

Two parts are coupled when:

```text
A cannot change without B
```

The goal is not:

```text
Zero coupling
```

because software needs relationships.

The goal is:

```text
Intentional coupling
```

For example:

```text
Orders → Payment Contract
```

may be intentional.

```text
Orders → Payment internal database table
```

is much stronger coupling.

---

# 66. Part Dependency Matrix

A simple matrix can help:

| Part | Depends On | Should Not Depend On |
|---|---|---|
| Application | Features, Core, Platform | Feature internals |
| Orders | Domain/Core | Payment internals |
| Payments | Core/Integration | Orders internals |
| Core | Stable abstractions | Features |
| Platform | Native SDKs | Business features |
| Integration | Vendor SDKs | UI |

The exact matrix depends on the product.

The important thing is that the relationships are deliberate.

---

# 67. Part Dependency Tree

A typical direction:

```text
                    Application
                         │
          ┌──────────────┼──────────────┐
          ▼              ▼              ▼
       Orders         Payments       Profile
          │              │              │
          └──────────────┼──────────────┘
                         ▼
                        Core
                         │
              ┌──────────┼──────────┐
              ▼          ▼          ▼
           Network    Security   Database
                         │
                         ▼
                     Platform
```

The actual dependency graph should be designed for the specific system.

---

# 68. Parts and Feature-to-Feature Dependencies

Sometimes:

```text
Orders
```

needs:

```text
Payments
```

Avoid automatically doing:

```text
Orders → Payments Implementation
```

Instead consider:

```text
Orders
   ↓
Payment Contract
   ▲
   │
Payments
```

or an event:

```text
Orders
   ↓
OrderCreated
   ↓
Payments
```

depending on the business relationship.

---

# 69. Parts and Shared Domain

Some domain concepts genuinely cross features.

For example:

```text
Money
Currency
CustomerId
OrderId
```

may be shared.

But avoid creating a massive:

```text
SharedDomain
```

containing every business model.

Shared domain concepts should be stable and genuinely shared.

---

# 70. Parts and Domain Ownership

Consider:

```text
Customer
```

used by:

```text
Orders
Payments
Profile
```

There are several possibilities:

```text
Identity owns Customer
```

and other features consume an appropriate contract.

This is often better than every feature owning a separate global Customer model.

---

# 71. Parts and Duplicate Models

Sometimes duplication is healthier than forced sharing.

For example:

```text
OrderCustomer
PaymentCustomer
```

may represent different business needs.

Avoid creating:

```text
UniversalCustomer
```

just to eliminate a few duplicate fields.

The right question is:

> **Do these models represent the same business concept in the same context?**

---

# 72. Parts and Bounded Contexts

A large system may have:

```text
Identity Context
Order Context
Payment Context
Inventory Context
```

Each context can have its own model.

```text
Identity.Customer
Orders.CustomerSnapshot
Payments.Payer
```

This can reduce accidental coupling.

---

# 73. Parts and Context-Specific Models

For example:

```kotlin
data class OrderCustomer(
    val id: CustomerId,
    val displayName: String
)
```

and:

```kotlin
data class PaymentCustomer(
    val id: CustomerId,
    val billingAddress: BillingAddress
)
```

These may intentionally differ.

Shared IDs can be enough without sharing the entire object.

---

# 74. Parts and Reuse

Reuse is valuable when:

```text
Meaning is shared
Behavior is shared
Lifecycle is shared
Ownership is shared
```

Reuse is harmful when:

```text
Different concepts are forced into one abstraction
```

The goal is not maximum reuse.

It is useful reuse.

---

# 75. Parts and the DRY Principle

DRY is often interpreted as:

> Don't repeat code.

A better architectural interpretation is:

> **Don't repeat knowledge that must remain consistent.**

Two similar pieces of code may legitimately be separate if they represent different business rules.

---

# 76. Parts and Duplication

Duplication can sometimes protect boundaries:

```text
Orders Model
```

and:

```text
Payments Model
```

may share:

```text
CustomerId
```

without sharing the entire model.

This avoids coupling unrelated contexts.

---

# 77. Parts and Shared Utilities

For small utilities:

```text
Date
Money
ID
Result
```

ask:

```text
Is this truly a cross-cutting concept?
Is the behavior stable?
Would sharing reduce duplication?
Will sharing create coupling?
```

Only then extract it.

---

# 78. Parts and the `utils` Folder

A large:

```text
utils/
```

is often a warning sign.

For example:

```text
utils/
├── DateUtils
├── StringUtils
├── NetworkUtils
├── PaymentUtils
├── OrderUtils
└── UserUtils
```

These utilities have unrelated responsibilities.

Prefer placing behavior near the capability that owns it.

---

# 79. Parts and Cohesion

A strong part has high cohesion.

For example:

```text
features/orders
```

contains things that belong together.

A weak part:

```text
common
```

contains unrelated things.

High cohesion makes:

```text
Understanding
Testing
Ownership
Refactoring
```

easier.

---

# 80. Parts and Coupling vs Cohesion

A useful architectural goal:

```text
High Cohesion
+
Low Unnecessary Coupling
```

Graphically:

```text
Part A
┌───────────────┐
│ related code  │
│ related code  │
│ related code  │
└───────────────┘

       │
       │ small stable API
       ▼

Part B
┌───────────────┐
│ related code  │
│ related code  │
└───────────────┘
```

---

# 81. Parts and Module Cohesion

A module should ideally answer one strong question.

Examples:

```text
What does this module do?

→ Orders

→ Security

→ Networking

→ Notifications
```

If the answer is:

```text
It contains things shared by several modules.
```

the boundary may be too vague.

---

# 82. Parts and Architecture Naming

Names matter.

Prefer:

```text
features/orders
core/security
integrations/payments
platform/scanner
```

over:

```text
module1
common2
sharedNew
misc
helpers
```

A developer should understand the architectural role from the name.

---

# 83. Parts and Naming Consistency

Choose a consistent convention.

For example:

```text
features/
core/
platform/
integrations/
apps/
```

Avoid mixing:

```text
feature/
features/
business/
domains/
modules/
```

unless each has a clearly different purpose.

---

# 84. Parts and Repository Navigation

A developer opening the repository should be able to answer quickly:

```text
Where are applications?
Where are features?
Where is shared infrastructure?
Where are platform adapters?
Where are external integrations?
Where is build logic?
```

A clear top-level structure makes onboarding faster.

---

# 85. Parts and GitHub Readability

A GitHub repository benefits from a readable tree:

```text
📦 project
├── 📱 apps
├── 🧩 features
├── 🧠 core
├── 🔌 platform
├── 🌐 integrations
├── 🛠️ build-logic
└── 🧪 tools
```

The structure itself becomes documentation.

---

# 86. Parts and README Architecture

A repository README can show:

```text
## Architecture

apps
  ↓
features
  ↓
core
  ↓
platform
```

Then explain each part.

This is much easier for new developers than discovering the architecture by browsing source files.

---

# 87. Parts and Gradle Settings

In a modular KMP project:

```kotlin
include(
    ":apps:androidApp",
    ":features:orders",
    ":features:payments",
    ":core:network",
    ":core:security",
    ":platform:storage"
)
```

The settings file becomes a high-level map of the repository.

---

# 88. Parts and Gradle Module Naming

A consistent naming convention helps:

```text
:apps:androidApp
:features:orders
:features:payments
:core:network
:core:security
:platform:storage
:integrations:payments
```

This is more informative than:

```text
:module1
:module2
:common
:shared
```

---

# 89. Parts and Gradle Configuration

A feature module may use:

```kotlin
plugins {
    alias(libs.plugins.kotlin.multiplatform)
}
```

and configure:

```kotlin
kotlin {
    androidTarget()
    iosX64()
    iosArm64()
    iosSimulatorArm64()

    sourceSets {
        val commonMain by getting
        val commonTest by getting
    }
}
```

The build configuration is part of the module boundary.

---

# 90. Parts and Convention Plugins

If every module repeats:

```text
KMP setup
Android setup
Testing setup
Compiler options
```

move the common configuration into build logic.

Then:

```text
Feature Module
      │
      ▼
KMP Convention
      │
      ▼
Standard Build
```

This keeps project parts consistent.

---

# 91. Parts and Source Set Structure

A feature module can look like:

```text
features/orders/
└── src/
    ├── commonMain/
    ├── commonTest/
    ├── androidMain/
    ├── androidUnitTest/
    ├── iosMain/
    └── iosTest/
```

This makes platform boundaries visible.

---

# 92. Parts and CommonMain

`commonMain` should contain code that can genuinely be shared.

Typical examples:

```text
Domain
Use Cases
Repository Contracts
Business Rules
Serialization Models
Shared State
```

when platform-independent.

---

# 93. Parts and Platform Source Sets

`androidMain` may contain:

```text
Android APIs
Android implementations
Android-specific UI
Android resources
```

`iosMain` may contain:

```text
iOS APIs
iOS implementations
iOS-specific integration
```

The boundary is explicit.

---

# 94. Parts and Platform Leakage

A common mistake:

```kotlin
commonMain/
└── AndroidContextProvider.kt
```

This defeats the purpose of common code.

Instead:

```text
commonMain
   ↓
Capability
   ↓
androidMain
```

The platform-specific mechanism stays on the platform side.

---

# 95. Parts and Dependency Injection

A feature may depend on:

```kotlin
interface SecureStorage
```

The application can provide:

```text
AndroidSecureStorage
```

or:

```text
IosSecureStorage
```

The architecture becomes:

```text
Feature
   ↓
Contract
   ↑
Platform Implementation
```

This allows the feature to remain portable.

---

# 96. Parts and the Composition Root

The application is a natural place to assemble:

```text
Feature implementations
Core services
Platform adapters
Integrations
```

Conceptually:

```kotlin
fun createApplicationGraph(): AppGraph {
    val network = createNetwork()
    val database = createDatabase()
    val security = createSecurity()

    return AppGraph(
        network = network,
        database = database,
        security = security
    )
}
```

The exact implementation depends on the DI approach.

---

# 97. Parts and Lifecycle

Different parts may have different lifecycles:

```text
Application
   │
   ├── Singleton services
   │
   ├── Feature scope
   │
   └── Screen scope
```

Avoid making everything global simply because it is convenient.

Ownership should reflect lifecycle.

---

# 98. Parts and Resources

Resources can also belong to parts.

For example:

```text
features/orders/
└── resources/
```

may contain:

```text
Order-specific strings
Images
Icons
Localization
```

This improves feature locality.

Platform-specific resource management can remain platform-specific.

---

# 99. Parts and Localization

A large application may have:

```text
orders/
payments/
profile/
```

with feature-specific localization needs.

Avoid creating one giant localization file containing every feature's text when the tooling and product requirements allow more localized ownership.

The principle is:

```text
Resource ownership follows capability ownership.
```

---

# 100. Parts and Design Systems

A design system is different from a business feature.

For example:

```text
design-system/
├── Button
├── TextField
├── Dialog
├── Card
└── Theme
```

This can be a shared UI capability.

But business components such as:

```text
OrderCard
PaymentSummary
```

may belong to their respective features.

---

# 101. Parts and UI Component Boundaries

A useful distinction:

```text
Generic UI
    → Design System

Business UI
    → Feature
```

For example:

```text
Button
    → Design System

OrderStatusCard
    → Orders
```

This prevents business logic from leaking into generic UI components.

---

# 102. Parts and Shared Presentation

Some presentation logic may be shared:

```text
OrderState
OrderAction
OrderReducer
```

while the visual implementation differs:

```text
Android Compose
iOS SwiftUI
```

The part remains:

```text
Orders
```

The platform presentation remains:

```text
Android / iOS
```

---

# 103. Parts and Navigation

Navigation can be treated as:

```text
Application-level concern
```

while feature destinations remain feature-owned.

For example:

```kotlin
sealed interface Destination {
    data class Order(
        val id: String
    ) : Destination
}
```

The application maps this destination to platform navigation.

---

# 104. Parts and Deep Links

Deep links should ideally resolve into:

```text
Semantic Destination
```

rather than directly constructing platform screens.

For example:

```text
myapp://orders/123
        ↓
Order(id = 123)
        ↓
Orders Feature
```

This keeps URL handling separate from business implementation.

---

# 105. Parts and Analytics

Analytics is usually cross-cutting.

A useful structure:

```text
core/observability/
```

or:

```text
integrations/analytics/
```

depending on whether the code represents:

```text
Generic analytics capability
```

or:

```text
Vendor-specific analytics
```

---

# 106. Generic Capability vs Vendor Integration

This distinction is important.

```text
Analytics
```

could be:

```kotlin
interface Analytics {
    fun track(event: AnalyticsEvent)
}
```

while:

```text
FirebaseAnalyticsAdapter
```

belongs to:

```text
integrations/
```

The feature depends on the capability.

The vendor stays behind the adapter.

---

# 107. Parts and Notifications

Similarly:

```text
NotificationService
```

may be a platform capability.

```text
FirebaseMessaging
APNs
Android NotificationManager
```

are implementation details.

The architecture can separate:

```text
Notification Contract
```

from:

```text
Platform / Vendor Implementation
```

---

# 108. Parts and File Storage

A generic capability:

```kotlin
interface FileStorage
```

can be implemented using:

```text
Android file APIs
iOS file APIs
Desktop file APIs
```

The business layer should not need to know which API is being used.

---

# 109. Parts and Secure Storage

Likewise:

```kotlin
interface SecureStorage
```

can hide:

```text
Android Keystore
iOS Keychain
```

This is a classic KMP platform boundary.

---

# 110. Parts and Connectivity

A common capability:

```kotlin
interface ConnectivityMonitor {
    val status: Flow<ConnectivityStatus>
}
```

can have platform implementations.

Features consume:

```text
ConnectivityStatus
```

rather than:

```text
ConnectivityManager
NWPathMonitor
```

---

# 111. Parts and Business Independence

The strongest parts are those that can answer:

```text
What business or technical capability do I own?
```

Examples:

```text
Orders
Payments
Security
Networking
Storage
```

Weak parts answer:

```text
Some files that are shared.
```

---

# 112. Parts and Architectural Boundaries

A part should hide unnecessary implementation details.

For example:

```text
Orders
 ├── public API
 └── internal implementation
```

The public API should be intentionally small.

This reduces coupling.

---

# 113. Parts and Internal Packages

A useful feature layout:

```text
orders/
├── api/
│   └── Orders.kt
│
└── internal/
    ├── data/
    ├── domain/
    └── presentation/
```

The exact naming is optional.

The principle is:

```text
Public boundary
+
Private implementation
```

---

# 114. Parts and API Surface

The larger the public surface:

```text
More consumers
+
More compatibility obligations
```

Therefore:

```text
Public API
    ↓
Keep small

Internal code
    ↓
Can evolve faster
```

---

# 115. Parts and Documentation

Each significant part should ideally have a small README.

For example:

```text
features/orders/README.md
```

could describe:

```markdown
# Orders

## Responsibility

Owns order lifecycle and order-related business rules.

## Public API

Orders

## Dependencies

core:network
core:database
core:security

## Platforms

Android
iOS
```

This turns the repository into self-documenting architecture.

---

# 116. Parts and Ownership Documentation

A part README can also specify:

```text
Owner
Criticality
Supported Platforms
Consumers
Public APIs
Migration Notes
```

This becomes valuable as teams increase.

---

# 117. Parts and Architecture Decision Records

If a part exists for a non-obvious reason, document the decision.

For example:

```text
ADR:
Payment SDK isolated in integrations/payments
```

Reason:

```text
Vendor replacement
Platform differences
Security isolation
```

This avoids repeating architectural debates.

---

# 118. Parts and Testing Boundaries

Each part should be testable within its boundary.

For example:

```text
Orders
├── commonTest
├── Android tests
└── iOS tests
```

Core:

```text
Network
├── unit tests
└── contract tests
```

Platform:

```text
Scanner
└── platform tests
```

This improves test ownership.

---

# 119. Parts and Fake Implementations

Shared testing can provide:

```kotlin
class FakePaymentGateway : PaymentGateway {
    override suspend fun authorize(
        request: PaymentRequest
    ): PaymentResult {
        return PaymentResult.Success
    }
}
```

The feature can test business behavior without using the real SDK.

This is another benefit of capability boundaries.

---

# 120. Parts and Dependency Substitution

If a part depends on:

```text
Interface
```

tests can substitute:

```text
Fake
Mock
Stub
Test implementation
```

The more explicit the boundary, the easier substitution becomes.

---

# 121. Parts and Build Performance

Project parts can also influence build performance.

A monolithic shared module:

```text
shared/
```

means:

```text
Small change
    ↓
Large module
    ↓
Potentially large recompilation
```

Meaningful modules can provide better isolation.

But excessive modularization can also increase build complexity.

The goal is balance.

---

# 122. Parts and Compilation Boundaries

A Gradle module can act as a compilation boundary.

For example:

```text
:features:orders
```

changing does not necessarily require recompiling unrelated modules.

This is one reason meaningful module boundaries can improve developer experience.

---

# 123. Parts and Build Isolation

A project might have:

```text
:core:network
:core:security
:features:orders
:features:payments
```

The dependency graph should avoid unnecessary relationships.

For example:

```text
Orders → Network
Payments → Network
```

is better than:

```text
Orders → Payments → Profile → Core
```

when those relationships are not business requirements.

---

# 124. Parts and Module Count

There is no universal ideal number of modules.

A project with:

```text
10 modules
```

can be poorly structured.

A project with:

```text
100 modules
```

can be well structured.

The important questions are:

```text
Are boundaries meaningful?
Are dependencies intentional?
Is ownership clear?
Is build performance acceptable?
```

---

# 125. Parts and Over-Modularization

Symptoms include:

```text
Too many tiny modules
Constant Gradle navigation
Long dependency chains
Hard-to-find implementations
Excessive interfaces
```

The solution may be consolidation.

Architecture should reduce complexity, not merely redistribute it.

---

# 126. Parts and Under-Modularization

Symptoms include:

```text
Giant shared module
Frequent merge conflicts
Slow builds
Unclear ownership
Cross-feature imports
Large public API
```

The solution may be extracting meaningful boundaries.

---

# 127. The Balance

Think of modularity as a curve:

```text
Too Few Modules
      │
      ▼
High Coupling

      ↓

Meaningful Modules
      │
      ▼
Good Isolation

      ↓

Too Many Modules
      │
      ▼
High Structural Complexity
```

The target is the middle.

---

# 128. Parts and the Rule of Meaningful Boundaries

A useful rule:

> **Create a part when it provides meaningful isolation, ownership, or reuse.**

Not merely because:

```text
There are many files.
```

---

# 129. Parts and Change Isolation

Imagine:

```text
Profile UI
```

changes.

Ideally:

```text
features/profile
```

contains most of the change.

The change should not require:

```text
Orders
Payments
Inventory
```

unless there is a genuine shared dependency.

This is locality.

---

# 130. Parts and Blast Radius

The blast radius of a change can be visualized as:

```text
Small Boundary

┌─────────────┐
│   Feature   │
└─────────────┘
      ↓
 Small Impact
```

Poor boundary:

```text
┌─────────────────────────────┐
│       Shared Everything     │
│ Orders Payments Profile ... │
└─────────────────────────────┘
              ↓
        Large Impact
```

Good parts reduce unnecessary propagation.

---

# 131. Parts and Team Boundaries

A natural architecture can align with teams:

```text
Orders Team
    ↓
features/orders

Payments Team
    ↓
features/payments

Platform Team
    ↓
core + platform
```

This alignment is not mandatory.

But when ownership and architecture align, coordination can become easier.

---

# 132. Parts and Team Autonomy

A feature team should ideally be able to:

```text
Develop
Test
Review
Deploy
Monitor
```

without modifying unrelated features.

Meaningful parts make this more achievable.

---

# 133. Parts and Shared Platform Teams

Platform teams can own:

```text
Network
Security
Storage
Observability
Build Logic
```

while feature teams own:

```text
Orders
Payments
Profile
```

This creates a useful separation:

```text
Platform Capabilities
        ↓
Feature Capabilities
        ↓
Application
```

---

# 134. Parts and Ownership Conflicts

If two teams believe they own:

```text
shared/network/
```

the boundary may be unclear.

Ownership should be explicit.

A part should ideally have:

```text
One accountable owner
```

even if many teams consume it.

---

# 135. Parts and Shared Ownership

Consumption can be many-to-one:

```text
Orders ─┐
Payments ─┼──→ Network
Profile ─┘
```

Ownership remains:

```text
Platform Team → Network
```

Consumers do not automatically become owners.

---

# 136. Parts and API Governance

A shared part should have rules for:

```text
API changes
Deprecation
Versioning
Migration
Documentation
```

This becomes important when the number of consumers grows.

---

# 137. Parts and Versioning

If:

```text
core:security
```

is consumed by multiple applications, changing its API may require coordination.

A controlled lifecycle can be:

```text
New API
   ↓
Deprecate old API
   ↓
Migrate consumers
   ↓
Remove old API
```

This is safer than breaking everyone at once.

---

# 138. Parts and Compatibility

Compatibility matters at several levels:

```text
Source compatibility
Binary compatibility
Behavior compatibility
Platform compatibility
Data compatibility
```

The larger the number of consumers, the more important compatibility becomes.

---

# 139. Parts and External Systems

External integrations should not dictate the internal architecture.

For example:

```text
Backend JSON
```

may contain:

```text
snake_case
status codes
vendor-specific values
```

The internal model can remain:

```text
Domain terminology
Strong types
Business semantics
```

The integration boundary translates between them.

---

# 140. Parts and Mappers

Mappers are useful when crossing boundaries:

```text
DTO
 ↓
Mapper
 ↓
Domain Model
 ↓
Mapper
 ↓
UI Model
```

Each representation serves its own part.

This reduces accidental coupling.

---

# 141. Parts and External SDKs

A third-party SDK may expose:

```text
VendorUser
VendorPayment
VendorSession
```

Do not automatically allow those types to become domain-wide models.

Instead:

```text
Vendor SDK
   ↓
Adapter
   ↓
Internal Model
```

The vendor remains replaceable.

---

# 142. Parts and Legacy Code

Existing applications may already contain:

```text
Legacy
```

and:

```text
Modern KMP
```

A migration boundary can look like:

```text
Legacy System
      │
      ▼
Adapter
      │
      ▼
KMP Capability
```

This allows gradual migration.

---

# 143. Parts and Incremental Migration

A large migration can proceed:

```text
Legacy Feature
      ↓
Define Boundary
      ↓
Create KMP Part
      ↓
Adapter
      ↓
Migrate Consumers
      ↓
Remove Legacy
```

This avoids requiring a complete rewrite.

---

# 144. Parts and Architecture Evolution

Project structure should evolve.

For example:

```text
Stage 1

shared/
```

then:

```text
Stage 2

shared/
├── features/
└── core/
```

then:

```text
Stage 3

features/
core/
platform/
```

then:

```text
Stage 4

apps/
features/
core/
platform/
integrations/
build-logic/
```

The architecture grows with the system.

---

# 145. Parts and the Cost of Premature Architecture

Do not create:

```text
apps/
platform/
integrations/
domain/
data/
presentation/
core/
tooling/
```

on day one if the application has:

```text
One screen
One feature
Two developers
```

Start with meaningful boundaries.

Extract when complexity justifies it.

---

# 146. Parts and Evolutionary Design

A good architecture can move:

```text
Package
   ↓
Module
   ↓
Shared Library
```

without changing the business concept.

For example:

```text
Orders
```

remains Orders regardless of whether it is:

```text
package
```

or:

```text
Gradle module
```

The architectural concept is more important than the physical representation.

---

# 147. Parts and Physical vs Logical Architecture

This distinction is extremely useful.

### Logical architecture

```text
Orders
Payments
Security
Network
```

### Physical architecture

```text
:features:orders
:features:payments
:core:security
:core:network
```

The physical structure should support the logical architecture.

It should not define the business architecture accidentally.

---

# 148. Parts and Refactoring Freedom

If the logical boundary is correct, the physical representation can change.

For example:

```text
Today:

shared/orders/

Tomorrow:

:features:orders
```

The business capability remains stable.

This gives the project room to evolve.

---

# 149. Parts and Architecture Intent

Before creating a part, write one sentence:

```text
This part exists to ______.
```

Examples:

```text
This part exists to own order lifecycle behavior.

This part exists to provide secure storage.

This part exists to isolate payment vendor integration.

This part exists to compose the Android application.
```

If the sentence is unclear, the boundary may be unclear.

---

# 150. Part Definition Checklist

For each part, define:

```text
Name
Purpose
Owner
Public API
Dependencies
Consumers
Platforms
Tests
Lifecycle
```

This simple checklist can prevent many architecture problems.

---

# 151. Example Part Definition

For Orders:

```text
Name:
Orders

Purpose:
Own order lifecycle and business rules.

Owner:
Orders Team

Public API:
Orders

Dependencies:
Network
Database

Consumers:
Android App
iOS App

Platforms:
Android
iOS

Tests:
commonTest
platform tests
integration tests
```

This is enough to make the boundary understandable.

---

# 152. Parts and Documentation Template

A reusable `README.md`:

```markdown
# Orders

## Responsibility

Owns order lifecycle and order-related business rules.

## Public API

- Orders
- OrderId

## Dependencies

- core:network
- core:database

## Consumers

- androidApp
- iosApp

## Platforms

- Android
- iOS

## Testing

- commonTest
- Android tests
- iOS tests
```

Small documentation can have a large impact at scale.

---

# 153. Common Mistake #1 — One Giant Shared Module

```text
shared/
    everything
```

### Why it happens

The project starts small.

### Why it hurts

Eventually:

```text
Everything depends on shared.
```

### Better approach

Extract meaningful capabilities when real coupling appears.

---

# 154. Common Mistake #2 — Module for Every File

```text
:order-id
:order-model
:order-helper
```

### Why it happens

Developers equate modularity with more modules.

### Why it hurts

The dependency graph becomes noisy.

### Better approach

Create modules around meaningful capabilities.

---

# 155. Common Mistake #3 — Giant Core

```text
core/
├── everything
```

### Why it happens

"Shared" code gets moved into Core.

### Why it hurts

Core becomes a dependency magnet.

### Better approach

Split Core by stable capability.

```text
core/network
core/security
core/database
```

---

# 156. Common Mistake #4 — Business Logic in Platform Code

Bad:

```text
androidMain/
└── OrderBusinessRules.kt
```

### Why it hurts

Business logic becomes platform-dependent.

### Better approach

Keep business rules in:

```text
commonMain
```

when they are platform-independent.

---

# 157. Common Mistake #5 — Platform SDK Leakage

Bad:

```text
commonMain
    ↓
AndroidContext
```

### Better approach

```text
commonMain
    ↓
Capability
    ↓
androidMain / iosMain
```

---

# 158. Common Mistake #6 — Feature-to-Feature Internals

Bad:

```text
Orders
   ↓
Payments internal class
```

### Better approach

```text
Orders
   ↓
Payment Contract
   ↑
Payments
```

The dependency should cross a public boundary.

---

# 159. Common Mistake #7 — Shared Global Models

Bad:

```text
models/
└── Everything.kt
```

### Why it hurts

Every feature becomes coupled to one representation.

### Better approach

Use:

```text
Domain models
Feature models
DTOs
UI models
```

where appropriate.

---

# 160. Common Mistake #8 — Global Utilities

Bad:

```text
utils/
    everything
```

### Better approach

Place behavior near the capability that owns it.

---

# 161. Common Mistake #9 — Vendor APIs Everywhere

Bad:

```text
Feature → Vendor SDK
```

### Better:

```text
Feature
   ↓
Contract
   ↓
Integration
   ↓
Vendor SDK
```

This makes vendor replacement easier.

---

# 162. Common Mistake #10 — Copying the Same Structure Everywhere

Do not force every project to have:

```text
20 modules
```

just because another project does.

The right structure depends on:

```text
Product size
Team size
Platforms
Business complexity
Release model
```

---

# 163. Common Mistake #11 — Mixing Logical and Physical Architecture

A developer may create:

```text
:common
```

because it sounds reusable.

But the logical responsibility is unclear.

Always define the capability first.

Then choose the physical module.

---

# 164. Common Mistake #12 — Architecture by Folder Count

This is not scalable:

```text
More folders
=
Better architecture
```

A project can have hundreds of folders and still be tightly coupled.

Architecture quality comes from:

```text
Boundaries
Dependencies
Ownership
Cohesion
```

---

# 165. Common Mistake #13 — No Ownership

A shared module with:

```text
20 consumers
```

and:

```text
No owner
```

is a long-term risk.

Every important part should have accountable ownership.

---

# 166. Common Mistake #14 — No Public API Strategy

If every class is public:

```text
Everything can be consumed.
```

Then every implementation becomes a potential contract.

Prefer:

```text
Small public API
+
Internal implementation
```

---

# 167. Common Mistake #15 — No Migration Path

Architecture changes happen.

If:

```text
Old API
```

must be replaced, plan:

```text
New API
Adapter
Migration
Deprecation
Removal
```

Do not assume every consumer can migrate immediately.

---

# 168. Common Mistake #16 — Over-Sharing Code

KMP makes sharing easy.

That does not mean everything should be shared.

Ask:

```text
Does sharing reduce meaningful duplication?
Does sharing preserve platform flexibility?
Does sharing create coupling?
```

---

# 169. Common Mistake #17 — Treating `commonMain` as a Dumping Ground

`commonMain` should not become:

```text
Everything that isn't Android
```

Instead:

```text
commonMain
=
genuinely platform-independent code
```

Platform-specific code belongs in the appropriate source set or behind an explicit abstraction.

---

# 170. Common Mistake #18 — Ignoring Build Architecture

A clean source tree can still have a terrible build graph.

Watch:

```text
Large modules
Deep dependency chains
Circular dependencies
Shared module bottlenecks
```

Project structure includes Gradle structure.

---

# 171. A Practical Decision Tree

When deciding where code belongs:

```text
                    New Code
                       │
                       ▼
              Is it business-specific?
                  /           \
                Yes            No
                │               │
                ▼               ▼
             Feature       Cross-cutting?
                              /      \
                            Yes       No
                            │          │
                            ▼          ▼
                           Core     Platform/
                                    Integration
```

Then ask:

```text
Does it need native APIs?
Does it belong to one feature?
Is it consumed by multiple parts?
Who owns it?
```

---

# 172. A Simpler Placement Rule

Use this mental model:

```text
Business capability
        ↓
Feature

Shared technical capability
        ↓
Core

Platform-specific capability
        ↓
Platform

External vendor/system
        ↓
Integration

Application composition
        ↓
Application

Build/developer infrastructure
        ↓
Build Logic / Tooling
```

This is a practical starting point.

---

# 173. KMP Project Structure Mental Model

The complete mental model:

```text
                         KMP PROJECT
                              │
        ┌─────────────────────┼─────────────────────┐
        ▼                     ▼                     ▼
   APPLICATIONS           FEATURES                CORE
        │                     │                     │
 Android / iOS          Business Capabilities   Shared Capabilities
                              │
                              ▼
                         PLATFORM
                              │
                    Native Implementations
                              │
                              ▼
                       INTEGRATIONS
                              │
                    External Systems / SDKs
```

And around all of them:

```text
BUILD LOGIC + TESTING + TOOLING
```

---

# 174. Parts vs Source Sets vs Layers

These three concepts should not be confused.

```text
PART
What capability or area?

LAYER
What responsibility inside that part?

SOURCE SET
Which platform can compile this implementation?
```

Example:

```text
Orders
│
├── Domain       ← Layer
├── Data         ← Layer
└── Presentation ← Layer
```

Inside the module:

```text
commonMain       ← Source set
androidMain      ← Source set
iosMain          ← Source set
```

At repository level:

```text
features/orders  ← Part
```

---

# 175. The Three-Dimensional KMP Model

KMP architecture can be visualized as three dimensions:

```text
                    PART
                     │
                     ▼
                  Orders
                     │
             ┌───────┼───────┐
             ▼       ▼       ▼
           Domain   Data   Presentation
             │
             ▼
          LAYER
             │
             ▼
       ┌─────┴─────┐
       ▼           ▼
  commonMain   platformMain
       │
       ▼
   SOURCE SET
```

This mental model prevents many structural mistakes.

---

# 176. A Small KMP Example

Suppose we build:

```text
Retail App
```

with:

```text
Orders
Payments
Profile
```

A sensible first structure:

```text
RetailApp/
├── androidApp/
├── iosApp/
│
├── features/
│   ├── orders/
│   ├── payments/
│   └── profile/
│
├── core/
│   ├── network/
│   └── database/
│
└── platform/
    └── secure-storage/
```

This is already enough to establish meaningful boundaries.

---

# 177. Orders Example

```text
features/orders/
├── src/
│   ├── commonMain/
│   │   └── orders/
│   │       ├── domain/
│   │       │   ├── Order.kt
│   │       │   └── OrderRepository.kt
│   │       │
│   │       ├── data/
│   │       │   ├── OrderRepositoryImpl.kt
│   │       │   ├── OrderApi.kt
│   │       │   └── OrderMapper.kt
│   │       │
│   │       └── presentation/
│   │           └── OrderViewModel.kt
│   │
│   ├── commonTest/
│   ├── androidMain/
│   └── iosMain/
│
└── build.gradle.kts
```

The exact arrangement can vary.

The important part is that Orders remains a coherent capability.

---

# 178. Payment Example

```text
features/payments/
├── src/
│   ├── commonMain/
│   │   └── payments/
│   │       ├── domain/
│   │       ├── data/
│   │       └── presentation/
│   │
│   ├── commonTest/
│   ├── androidMain/
│   └── iosMain/
│
└── build.gradle.kts
```

If a vendor SDK is required:

```text
integrations/payments/
```

can isolate the SDK.

---

# 179. Integration Example

```text
integrations/payments/
├── src/
│   ├── commonMain/
│   ├── androidMain/
│   └── iosMain/
└── build.gradle.kts
```

Conceptually:

```text
Payments Feature
       │
       ▼
Payment Contract
       │
       ▼
Payment Integration
       │
       ▼
Vendor SDK
```

---

# 180. Platform Example

```text
platform/secure-storage/
├── src/
│   ├── commonMain/
│   │   └── SecureStorage.kt
│   │
│   ├── androidMain/
│   │   └── AndroidSecureStorage.kt
│   │
│   └── iosMain/
│       └── IosSecureStorage.kt
│
└── build.gradle.kts
```

The contract is common.

The implementation is platform-specific.

---

# 181. Core Network Example

```text
core/network/
├── src/
│   ├── commonMain/
│   │   ├── HttpClientFactory.kt
│   │   └── NetworkConfig.kt
│   ├── androidMain/
│   └── iosMain/
│
└── build.gradle.kts
```

Feature APIs can use the shared networking capability without knowing platform details.

---

# 182. Part-Level Dependency Example

```text
:features:orders
        │
        ├──────────────► :core:network
        │
        ├──────────────► :core:database
        │
        └──────────────► :platform:secure-storage
```

Avoid:

```text
:features:orders
        │
        ├──────────────► :features:payments:internal
        ├──────────────► :features:profile:internal
        └──────────────► :core:everything
```

The first graph is intentional.

The second graph is a warning sign.

---

# 183. The Architecture Compass

When deciding where something belongs, ask five questions:

```text
1. What capability does this represent?

2. Who owns it?

3. Who consumes it?

4. What can change independently?

5. Does it depend on a platform or external system?
```

These five questions are often enough to find the right part.

---

# 184. Part Design Checklist

Before creating a new part:

```text
[ ] Clear responsibility
[ ] Clear owner
[ ] Clear public API
[ ] Clear consumers
[ ] Clear dependencies
[ ] Clear platform requirements
[ ] Clear testing strategy
[ ] Clear lifecycle
[ ] Clear reason for existence
```

If several answers are unclear, reconsider the boundary.

---

# 185. Final Takeaways

The first step toward a scalable KMP project is not creating dozens of Gradle modules.

It is learning to divide the system into **meaningful parts**.

Remember:

- Applications compose the system.
- Features represent business capabilities.
- Core contains intentional shared capabilities.
- Platform contains platform-specific mechanisms.
- Integrations isolate external systems and vendors.
- Build logic standardizes project configuration.
- Tooling supports developers without becoming runtime code.
- A part is a logical boundary before it becomes a Gradle module.
- Packages can establish boundaries before modules are necessary.
- Layers and parts are different concepts.
- Source sets and parts are different concepts.
- `commonMain` should contain genuinely shareable code.
- Platform APIs should remain behind platform boundaries.
- Feature ownership should be clear.
- Public APIs should be smaller than internal implementations.
- DTOs, entities, domain models, and UI models should not automatically become the same object.
- Shared code should be valuable shared code, not maximum shared code.
- Module count is not an architecture-quality metric.
- Meaningful boundaries are more important than folder count.
- Architecture should evolve as real complexity appears.

The core mental model is:

```text
                PROJECT
                   │
        ┌──────────┼──────────┐
        ▼          ▼          ▼
    FEATURES      CORE      PLATFORM
        │          │          │
        ▼          ▼          ▼
     BUSINESS   SHARED      NATIVE
    CAPABILITY CAPABILITY  IMPLEMENTATION
        │          │          │
        └──────────┼──────────┘
                   ▼
             APPLICATION
```

And inside a KMP part:

```text
                 PART
                   │
        ┌──────────┼──────────┐
        ▼          ▼          ▼
      DOMAIN      DATA   PRESENTATION
        │          │          │
        └──────────┼──────────┘
                   ▼
              SOURCE SETS
                   │
          ┌────────┼────────┐
          ▼        ▼        ▼
     commonMain androidMain iosMain
```

> [!NOTE]
> **A good project structure makes the architecture visible. A great project structure also makes the wrong dependency difficult to create.**

The objective is not to create the most complicated project tree.

The objective is to create a structure where:

```text
Responsibility is clear
        +
Ownership is clear
        +
Dependencies are intentional
        +
Platform boundaries are explicit
        +
Change remains localized
```

That is the foundation on which the next architectural decisions can safely be built.
