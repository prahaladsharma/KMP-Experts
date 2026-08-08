# Chapter 3 — Android vs Flutter vs React Native vs Compose vs KMP

## Part 4 — React Native

> **React Native took a different path: keep JavaScript at the center, but bring native mobile components into the application.**

React Native emerged from a different ecosystem than Android and Kotlin.

Its roots are in the web.

More specifically, it came from the React programming model and the JavaScript ecosystem.

The central idea was compelling:

> **Develop mobile applications using a familiar React and JavaScript/TypeScript model while still producing applications that interact with native platform capabilities.**

This approach made React Native particularly attractive to organizations that already had strong web engineering teams.

But to understand its strengths and trade-offs, we need to understand the architecture behind it.

---

# What Is React Native?

React Native is a framework for building native applications using JavaScript or TypeScript and the React programming model.

At a simplified level:

```text
              React Application
                     │
                     ▼
            JavaScript / TypeScript
                     │
                     ▼
              React Native
                     │
          ┌──────────┴──────────┐
          ▼                     ▼
      Android                 iOS
          │                     │
          ▼                     ▼
   Native Platform APIs   Native Platform APIs
```

The important point is that React Native is not simply a web application running inside a mobile browser.

It provides a model for building mobile applications that interact with native platform capabilities.

---

# The React Mental Model

React introduced a declarative approach to building user interfaces.

Instead of describing every individual UI operation, developers describe what the interface should look like for a particular state.

Conceptually:

```text
Application State
       │
       ▼
   React Tree
       │
       ▼
   UI Output
```

For example:

```tsx
function Profile({ name }) {
  return (
    <View>
      <Text>Hello, {name}</Text>
    </View>
  );
}
```

The syntax is different from Kotlin and Jetpack Compose, but the underlying idea is familiar:

> **UI is derived from state.**

This declarative model became one of React Native's biggest advantages for developers already familiar with React.

---

# React Native Is Not React for the Browser

This distinction is important.

A web React application typically renders into the browser's DOM.

```text
React
  │
  ▼
Browser
  │
  ▼
DOM
```

React Native follows a different path.

```text
React
  │
  ▼
React Native
  │
  ▼
Native Platform
```

Instead of rendering HTML elements such as:

```html
<div>
<button>
<input>
```

React Native applications use components such as:

```text
View
Text
Image
TextInput
ScrollView
Pressable
```

These components are designed for mobile application development.

---

# Figure 3.3 — React vs React Native

```text
                React

          ┌───────┴───────┐
          ▼               ▼
        Web          React Native
          │               │
          ▼               ▼
        DOM          Mobile Platform
```

> **Key Idea**
>
> React Native shares the React programming model, but its target is mobile application development rather than browser-based UI.

---

# The JavaScript / Native Boundary

One of the most important concepts when understanding React Native is the boundary between JavaScript and native code.

Historically, React Native applications relied heavily on communication between JavaScript and native platform code.

A simplified model looks like:

```text
       JavaScript / TypeScript
                  │
                  ▼
         React Native Runtime
                  │
                  ▼
        Native Communication
           ┌──────┴──────┐
           ▼             ▼
       Android           iOS
```

This architecture allowed JavaScript code to control native capabilities.

However, communication across that boundary became an important architectural consideration for performance-sensitive workloads.

---

# The Evolution of React Native's Architecture

It is important not to describe modern React Native solely using its original architecture.

React Native has evolved significantly.

The ecosystem introduced a newer architecture centered around technologies such as:

- JSI
- Fabric
- TurboModules
- Codegen

The goal was to reduce unnecessary overhead and provide a more direct and efficient interaction between JavaScript and native code.

A simplified modern view is:

```text
          JavaScript / TypeScript
                    │
                    ▼
                   JSI
                    │
        ┌───────────┴───────────┐
        ▼                       ▼
      Fabric               TurboModules
        │                       │
        ▼                       ▼
   Native UI              Native APIs
```

The architecture is considerably more sophisticated than the original bridge-centric model.

---

# Why This Evolution Matters

Framework architecture should always be evaluated based on the version and architecture actually being used.

It would be inaccurate to judge current React Native solely by limitations associated with older versions.

At the same time, understanding the historical architecture is useful because it explains why performance and native interoperability became such important areas of React Native's evolution.

The broader lesson is:

> **Cross-platform frameworks are not static technologies. Their architecture changes as developers discover new requirements.**

---

# React Native UI

React Native uses a component-based UI model.

A simplified component tree might look like:

```text
Screen
 │
 ├── View
 │    ├── Text
 │    └── Image
 │
 └── Pressable
```

Developers compose these building blocks into larger screens and features.

This model provides a familiar experience for React developers.

---

# Shared UI vs Native UI

React Native occupies an interesting position between fully custom rendering and completely separate native UI implementations.

The developer writes one component tree:

```text
             React Component

                   │

          ┌────────┴────────┐
          ▼                 ▼
       Android             iOS
          │                 │
          ▼                 ▼
     Native Platform   Native Platform
```

This means application developers can share a substantial amount of UI code.

At the same time, platform-specific behavior can be introduced when required.

---

# Platform-Specific Code Still Exists

Cross-platform doesn't mean platform-independent.

Consider a feature that behaves differently on Android and iOS.

React Native provides mechanisms for platform-specific implementations.

Conceptually:

```text
             Shared Feature
                   │
          ┌────────┴────────┐
          ▼                 ▼
     Android Logic       iOS Logic
```

This is important because real applications eventually encounter platform differences.

The question is not whether platform-specific code exists.

The question is how much of it the application needs.

---

# React Native and Native Modules

Applications frequently need capabilities beyond the framework's common APIs.

For example:

```text
Biometrics
Bluetooth
NFC
Camera
Payments
Health APIs
Background Processing
Device Management
```

Native modules allow developers to expose platform-specific capabilities to JavaScript or TypeScript.

The architecture becomes:

```text
             React Native App
                    │
                    ▼
              Shared API
                    │
          ┌─────────┴─────────┐
          ▼                   ▼
    Android Module       iOS Module
          │                   │
          ▼                   ▼
    Android APIs          iOS APIs
```

This provides flexibility.

It also means the application team may end up maintaining multiple implementations for platform-specific functionality.

---

# The JavaScript Ecosystem

One of React Native's strongest advantages is its relationship with the JavaScript ecosystem.

A company may already have:

```text
Web Team
   │
React
   │
TypeScript
   │
Node.js
```

Introducing React Native can create opportunities for developers to move between web and mobile projects more easily.

Shared knowledge can include:

- JavaScript / TypeScript
- React
- State management
- API patterns
- Testing tools
- Development practices

For organizations already invested heavily in the React ecosystem, this can be a significant advantage.

---

# React Native and Developer Experience

React Native provides a familiar component and development model for React developers.

A typical development cycle looks like:

```text
Change Code
    │
    ▼
Fast Feedback
    │
    ▼
Run Application
    │
    ▼
Inspect UI / State
    │
    ▼
Iterate
```

The exact development experience depends on the project's tooling and configuration.

But the underlying goal remains the same:

> Keep the feedback loop short enough that developers can experiment quickly.

---

# React Native Architecture

React Native does not prescribe one complete application architecture.

A production project may use:

```text
React Components
       │
       ▼
State Management
       │
       ▼
Domain / Business Logic
       │
       ▼
Services
       │
       ▼
API / Storage
       │
       ▼
Native Modules
```

Teams can choose different approaches for state management, networking, dependency organization, and domain modeling.

As with Flutter and Android, the framework does not automatically create good architecture.

Engineering discipline still matters.

---

# Where React Native Performs Well

React Native can be particularly attractive when:

### 🌐 The Organization Already Uses React

Existing React developers can transfer much of their knowledge into mobile development.

### 📱 Android and iOS Are Both Required

A shared application layer can reduce duplicated development.

### 🎨 A Shared UI Model Is Acceptable

Teams can maintain a largely shared component architecture.

### 🚀 Fast Product Development Matters

A single development model can accelerate feature delivery.

### 👥 JavaScript / TypeScript Hiring Is Easier

Organizations with strong web engineering teams may find React Native easier to staff.

---

# Where React Native Requires Careful Evaluation

React Native becomes more complicated when applications depend heavily on specialized native behavior.

Examples include:

- Advanced Bluetooth
- Industrial hardware
- Specialized background processing
- Deep operating-system integration
- Custom native rendering
- Platform-specific performance requirements

Again, this doesn't mean React Native cannot handle these scenarios.

It means the team must understand how much native code will ultimately be required.

---

# The Hidden Question: How Much Native Code Will We Need?

This is one of the most useful questions when evaluating React Native.

Imagine a product that begins with:

```text
90% Shared
10% Native
```

That may be an excellent fit.

But as requirements evolve, it might become:

```text
70% Shared
30% Native
```

Then:

```text
50% Shared
50% Native
```

At that point, the organization needs to evaluate whether the abstraction is still providing enough value.

The framework isn't necessarily failing.

The product requirements have changed.

---

# React Native and Existing Android Applications

Consider an organization with a mature Android application:

```text
Existing Android Application

        │
        ├── Kotlin
        ├── Jetpack
        ├── Native APIs
        ├── Existing Tests
        └── Existing CI/CD
```

Introducing React Native means introducing another major technology ecosystem.

The organization may need:

- JavaScript / TypeScript expertise
- React expertise
- React Native expertise
- Native integration expertise
- Additional build configuration

Migration can therefore become more complex than simply introducing shared business logic.

This is particularly important for large organizations with significant existing Kotlin investments.

---

# React Native vs Native Android

Using our evaluation framework:

| Dimension | Native Android | React Native |
|-----------|----------------|--------------|
| Android Platform Access | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| Native Android UX | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| Shared UI | ⭐ | ⭐⭐⭐⭐⭐ |
| Cross-Platform Development | ⭐ | ⭐⭐⭐⭐⭐ |
| React Ecosystem | ⭐ | ⭐⭐⭐⭐⭐ |
| Kotlin Ecosystem | ⭐⭐⭐⭐⭐ | ⭐⭐ |
| Android Hardware | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| Existing Android Migration | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| Platform Control | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| Shared Application Code | ⭐ | ⭐⭐⭐⭐⭐ |

These are architectural comparisons, not performance benchmarks.

The exact outcome depends on the application and its requirements.

---

# React Native vs Flutter

At first glance, Flutter and React Native appear very similar.

Both provide:

- Cross-platform development
- Shared application code
- Shared UI development
- Native platform integration
- Large developer ecosystems

But their rendering philosophies differ.

### Flutter

```text
Application
     │
     ▼
Flutter Widgets
     │
     ▼
Flutter Engine
     │
     ▼
Platform
```

### React Native

```text
React Components
       │
       ▼
React Native Architecture
       │
       ▼
Native Platform
```

This difference affects how UI, platform behavior, rendering, and native integration are approached.

---

# The Architectural Trade-Off

React Native makes a different trade from both native Android and Flutter.

```text
        React / JavaScript Ecosystem
                    │
                    ▼
             Shared UI Model
                    │
                    ▼
            Native Integration
                    │
                    ▼
       Platform-Specific Extensions
```

The benefit is a familiar development model for React teams.

The cost is that large applications may require careful management of the boundary between shared JavaScript/TypeScript code and native platform implementations.

---

# A Practical Example

Imagine a banking application.

The following logic could be shared:

```text
Authentication
Validation
Transaction Rules
Account Formatting
API Communication
State Management
```

But platform-specific capabilities may remain native:

```text
Android Biometrics
iOS Face ID
Android Notifications
iOS Notifications
Secure Storage
Platform Security APIs
```

The resulting architecture might look like:

```text
                 Shared React Native Layer

          ┌──────────┬───────────┬──────────┐
          │          │           │          │
      Business    Networking   State      UI
       Logic
          │
          ▼
   Platform Integration
      ┌──────┴──────┐
      ▼             ▼
   Android         iOS
```

The amount of platform-specific code ultimately depends on the product.

---

# The Most Important Lesson

React Native demonstrates another important point in the evolution of cross-platform development.

> **Sharing application development does not eliminate platform differences.**

It changes where those differences live.

Some teams may prefer this model.

Others may prefer a more native architecture.

Others may prefer sharing only business logic.

This is why framework comparisons must always return to the evaluation criteria from Part 1.

---

# 🧠 React Native in One Sentence

If we had to summarize React Native architecturally:

> **React Native allows teams to share a React-based application model across platforms while retaining mechanisms for native platform integration.**

Its biggest strength is the combination of:

```text
React
+
JavaScript / TypeScript
+
Cross-Platform Development
+
Native Integration
```

Its biggest architectural question is:

```text
How much of the application should remain
inside the shared React Native layer,
and how much should move into native code?
```

---

# Key Takeaways

> [!TIP]
> **React Native is particularly compelling for organizations already invested in React and JavaScript/TypeScript.**

The important points are:

- React Native uses JavaScript or TypeScript with the React programming model.
- It targets mobile platforms rather than browser DOM rendering.
- It allows substantial sharing of application and UI code.
- Modern React Native has evolved significantly beyond its original bridge-based architecture.
- Technologies such as JSI, Fabric, and TurboModules are important parts of its modern architecture.
- Native modules remain important for platform-specific capabilities.
- React Native benefits organizations with strong React expertise.
- Existing native applications require careful migration planning.
- Platform-specific code does not disappear; it moves into explicit native integration points.
- The right choice depends on the product rather than framework popularity.

---

## Looking Ahead

We have now examined two major cross-platform approaches:

**Flutter** emphasizes a shared application and rendering model.

**React Native** emphasizes a shared React-based development model with native platform integration.

The next technology takes a particularly interesting position for Kotlin and Android developers.

**Compose Multiplatform** starts from the declarative UI model many Android developers already know and extends it beyond Android.

That raises a natural question:

> **What happens when the Android UI model itself becomes multiplatform?**
````
