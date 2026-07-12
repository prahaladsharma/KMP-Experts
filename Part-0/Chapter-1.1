## Why Sharing Code Was Never the Real Goal

At this point, it might sound like the obvious solution is to share as much code as possible.

Interestingly, that isn't what the software industry learned over the last decade.

Many cross-platform frameworks promised exactly that:

> "Write once. Run everywhere."

On paper, it sounded perfect.

Write a single application.

Compile it for multiple platforms.

Reduce development cost.

Ship features faster.

In reality, engineering teams discovered that sharing *everything* often introduced a different set of problems.

The user experience started feeling less native.

Platform-specific capabilities became harder to access.

Applications behaved differently from what users expected on each operating system.

As projects grew larger, maintaining a fully shared UI became increasingly complex.

Eventually, many engineering teams reached the same conclusion.

The objective isn't to share **everything**.

The objective is to share **the right things**.

That distinction is one of the most important ideas in Kotlin Multiplatform.

---

## Understanding the Layers of a Mobile Application

Imagine opening your favorite shopping application.

You see:

- Product images
- Search bar
- Shopping cart
- Checkout button

This is the part users interact with.

But behind every screen lies an entire software system.

A realistic mobile application looks more like this.


                    Mobile Application

 ┌─────────────────────────────────────────┐
 │                 UI Layer                │
 │                                         │
 │  Compose • SwiftUI • XML • UIKit        │
 └─────────────────────────────────────────┘

 ┌─────────────────────────────────────────┐
 │          Presentation Layer             │
 │                                         │
 │ ViewModel • State • Events              │
 └─────────────────────────────────────────┘

 ┌─────────────────────────────────────────┐
 │            Domain Layer                 │
 │                                         │
 │ UseCases • Business Rules               │
 └─────────────────────────────────────────┘

 ┌─────────────────────────────────────────┐
 │             Data Layer                  │
 │                                         │
 │ Repository • API • Database             │
 └─────────────────────────────────────────┘

 ┌─────────────────────────────────────────┐
 │        Platform Infrastructure          │
 │                                         │
 │ Files • Camera • Bluetooth              │
 └─────────────────────────────────────────┘
```

Every layer has different responsibilities.

Not every layer should be shared.

Understanding this separation is the key to understanding Kotlin Multiplatform.

---

## Which Layers Actually Depend on the Platform?

Let's examine every layer one by one.

### User Interface

The UI is naturally platform-specific.

Android users expect Material Design.

iPhone users expect Human Interface Guidelines.

Buttons look different.

Navigation behaves differently.

Animations feel different.

Even gestures are different.

Trying to force identical user interfaces across platforms often produces an application that feels foreign everywhere.

For this reason, keeping the UI native usually provides the best user experience.

---

### Business Rules

Now consider a different example.

A banking application requires passwords containing:

- At least eight characters
- One uppercase letter
- One lowercase letter
- One number
- One special character

Should Android validate those rules differently from iOS?

Of course not.

The business requirement is identical.

The platform is irrelevant.

The same applies to:

- Discount calculation
- EMI eligibility
- Reward points
- Shipping rules
- Tax calculation
- Coupon validation
- Inventory availability

These are business rules.

Business rules belong to the business—not to Android or iOS.

---

### Networking

Suppose the backend exposes the following endpoint.

```
POST /login
```

Android sends:

```json
{
  "email": "john@example.com",
  "password": "password123"
}
```

Should iOS send a different request?

No.

The backend expects exactly the same payload.

The response is also identical.

```json
{
  "accessToken":"...",
  "refreshToken":"..."
}
```

Networking logic doesn't depend on the platform.

Only the HTTP client implementation changes internally.

The business workflow remains identical.

---

### Repository Layer

Repositories coordinate data.

They decide:

- Fetch from network
- Read from cache
- Store in database
- Return domain model

These decisions don't depend on Android.

They don't depend on iOS.

Repositories are excellent candidates for sharing.

---

### Domain Models

Consider a Product.

```
Product

id

title

price

rating

stock

description
```

Should Android have different product information than iOS?

Obviously not.

The product exists independently of any operating system.

Domain models represent business concepts.

Business concepts don't change because the application is running on a different device.

---

### Validation

Imagine validating an email address.

Should Android accept

```
john@example.com
```

while iOS rejects it?

That would be disastrous.

Validation must remain consistent across every platform.

Sharing validation logic eliminates an entire category of production bugs.

---

## The Cost of Duplication

Developers often think duplication means writing more code.

The reality is much more expensive.

Imagine a single business rule.

```
Password must contain
one uppercase letter.
```

In a traditional architecture, that single rule creates multiple responsibilities.

```
Requirement

      │

      ▼

Android Implementation

      │

Android Unit Tests

      │

Android Code Review

      │

Android Maintenance

────────────────────────────

iOS Implementation

      │

iOS Unit Tests

      │

iOS Code Review

      │

iOS Maintenance
```

One requirement.

Two implementations.

Two test suites.

Two reviews.

Two maintenance paths.

Now multiply this by hundreds of business rules accumulated over several years.

Suddenly, maintaining consistency becomes one of the biggest engineering challenges in the organization.

---

## The Hidden Tax Every Team Pays

Most companies don't realize they're paying this cost.

It doesn't appear on a balance sheet.

It isn't reported by analytics.

Yet every sprint feels its impact.

Consider a simple login enhancement.

Without shared business logic, the timeline often looks like this.

Business Requirement

        │

Android Development

        │

Android Code Review

        │

Android QA

        │

────────────────────────────

iOS Development

        │

iOS Code Review

        │

iOS QA

        │

────────────────────────────

Regression Testing

        │

Release
```

Every feature passes through two parallel engineering pipelines.

The larger the organization becomes, the more synchronization is required.

Meetings increase.

Documentation increases.

Cross-platform validation increases.

Release coordination becomes increasingly complex.

Engineering velocity gradually slows—not because developers are less productive, but because duplication creates operational overhead.

---

## Figure 1.2 — Feature Development Without Shared Business Logic

**Illustration Specification (Final Book Diagram)**

**Title**

Feature Development Before Kotlin Multiplatform

```
                New Feature

                     │

        ┌────────────┴────────────┐

        ▼                         ▼

 Android Team               iOS Team

        │                         │

 Design Implementation     Design Implementation

        │                         │

 Business Logic            Business Logic

        │                         │

 Unit Testing              Unit Testing

        │                         │

 Bug Fixes                 Bug Fixes

        │                         │

 Release                   Release
```

**Observation**

Every engineering activity is duplicated.

The organization doesn't build one feature.

It builds two independent implementations of the same feature.

---

## Looking at the Problem Differently

Instead of asking:

> "How can Android and iOS share code?"

Let's ask a different question.

> "Which parts of this application actually belong to the business?"

This subtle change completely transforms the discussion.

Businesses don't own Android.

Businesses don't own iOS.

Businesses own:

- Pricing rules
- Customer validation
- Loyalty programs
- Authentication policies
- Payment workflows
- Order processing
- Shipping calculations

These concepts exist regardless of platform.

Whether a customer places an order from an Android phone, an iPhone, a desktop browser, or a smartwatch, the business rules remain exactly the same.

That realization is the foundation of Kotlin Multiplatform.

JetBrains didn't begin with the idea of sharing platforms.

They began with the idea of sharing **business knowledge**.

---

## A Thought Experiment

Imagine tomorrow your company decides to build four applications.

- Android
- iOS
- Desktop
- Web

Should the discount calculation now exist four times?

Should inventory validation exist four times?

Should coupon rules exist four times?

Should authentication logic exist four times?

Clearly not.

Adding more platforms shouldn't multiply business logic.

The business itself hasn't changed.

Only the number of user interfaces has changed.

Once you start viewing software through this lens, the architectural direction becomes much clearer.

Instead of asking:

> "Which platform owns this code?"

you begin asking:

> "Does this code represent the business, or does it represent the platform?"

That single question is one of the most valuable architectural filters you'll use throughout this book.

It not only explains why Kotlin Multiplatform exists—it also explains why certain code belongs in shared modules while other code should remain platform-specific.

The distinction between **business code** and **platform code** is the foundation upon which the rest of Kotlin Multiplatform is built.
````
