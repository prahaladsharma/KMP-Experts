# Chapter 5 — Creating Your First KMP Project

## Part 5 — Running Android

> **The first Android run is where the KMP project stops being a folder structure and becomes a real application.**

The project has been created.

The generated files are understood.

The folder structure is clear.

The build works.

Now it is time to run the Android application.

For an Android developer, this part will feel familiar at first:

```text
Android Studio
      │
      ▼
Device / Emulator
      │
      ▼
Run
      │
      ▼
Application
```

But there is an important difference.

The Android application is no longer isolated.

It consumes code from the KMP shared module:

```text
                    Android App
                         │
                         ▼
                    Shared Module
                         │
                         ▼
                    commonMain
```

That relationship is what we want to prove.

---

# 1. What Are We Trying to Prove?

The goal of the first Android run is not to build a complete application.

We want to verify four things:

```text
✓ Android project is configured
✓ Shared module is connected
✓ Android target compiles
✓ Application can run on an Android device
```

The flow is:

```text
commonMain
    │
    ▼
KMP Shared Module
    │
    ▼
Android Target
    │
    ▼
androidApp
    │
    ▼
APK
    │
    ▼
Android Device / Emulator
```

If this works, the Android side of our first KMP project is alive.

---

# 2. Before Running

Make sure the project has already passed the basic build stage.

A good sequence is:

```text
Project Created
      │
      ▼
Generated Files Inspected
      │
      ▼
Build Successful
      │
      ▼
Android Environment Ready
      │
      ▼
Run Android
```

Do not introduce multiple new dependencies immediately before the first run.

The cleaner the project is, the easier it is to understand what is happening.

---

# 3. Android Environment

For the Android target, you need a functioning Android development environment.

Typically this includes:

```text
Android Studio
Android SDK
Android Build Tools
Android Emulator or Physical Device
JDK supported by your project/toolchain
```

The exact supported versions change over time.

Use the versions recommended by the current Kotlin Multiplatform and Android tooling documentation for the project you are creating.

---

# 4. Android Device or Emulator

You need somewhere to run the application.

You can use:

```text
Android Emulator
```

or:

```text
Physical Android Device
```

Conceptually:

```text
                 Android Build
                       │
                       ▼
                    APK
                       │
             ┌─────────┴─────────┐
             ▼                   ▼
          Emulator           Physical Device
```

For a first project, an emulator is usually the simplest reproducible option.

---

# 5. Create or Select an Emulator

If you are using Android Studio, open the device management tools and create an emulator if you don't already have one.

Choose an Android device profile and a system image appropriate for your development environment.

For example:

```text
Device:
Pixel-class device

System Image:
Android version supported by your project
```

The exact device and Android version are not important for learning KMP.

What matters is that:

```text
ADB
+
Android SDK
+
Emulator
```

are working correctly.

---

# 6. Verify the Device

Before running the application, verify that Android Studio can see the device.

You can also check through the command line:

```bash
adb devices
```

A healthy result should show a connected device or emulator.

Conceptually:

```text
Developer Machine
       │
       ▼
      ADB
       │
       ▼
Android Device
```

If the device is not visible, fix the Android environment before troubleshooting KMP.

---

# 7. Running From Android Studio

The simplest approach is to select the Android application run configuration.

Conceptually:

```text
Run Configuration
       │
       ▼
androidApp
       │
       ▼
Select Device
       │
       ▼
Run
```

The exact UI labels can change between Android Studio versions and project templates.

The important part is that you are running the **Android application target**, not just compiling the shared library.

---

# 8. What Happens When You Press Run?

A simple mental model is:

```text
                 Run
                  │
                  ▼
           Gradle Build Tasks
                  │
                  ▼
          Compile Shared Code
                  │
                  ▼
          Compile Android Code
                  │
                  ▼
              Package APK
                  │
                  ▼
            Install on Device
                  │
                  ▼
               Launch App
```

The real task graph is more detailed.

But this is the flow you should remember.

---

# 9. The Shared Code Is Included

Suppose your shared module contains:

```kotlin
class Greeting {

    fun message(): String {
        return "Hello from Kotlin Multiplatform!"
    }
}
```

The Android application can consume this code.

Conceptually:

```text
androidApp
    │
    ▼
shared
    │
    ▼
commonMain
```

The important point is:

> **Android is not copying the source file into the Android module. It is consuming the shared module through the Gradle build.**

---

# 10. A Simple Android Screen

A minimal Android screen might call the shared code.

For example:

```kotlin
@Composable
fun GreetingScreen(
    greeting: Greeting
) {
    Text(
        text = greeting.message()
    )
}
```

The exact UI implementation depends on the project template.

The architecture is what matters:

```text
Android UI
    │
    ▼
Greeting
    │
    ▼
commonMain
```

---

# 11. The Runtime Flow

Once the application is installed:

```text
User
 │
 ▼
Android App
 │
 ▼
Android UI
 │
 ▼
Shared Kotlin Code
 │
 ▼
Result
 │
 ▼
Android UI
```

The user doesn't need to know whether the business logic originated in:

```text
androidApp
```

or:

```text
shared
```

The application simply runs.

---

# 12. What KMP Does Not Do

KMP does not make Android disappear.

You still have:

```text
Android Activity
Android lifecycle
Android SDK
Android resources
Android permissions
Android packaging
```

The difference is that some of the logic can now be shared.

Think:

```text
Android
   +
Shared Kotlin
```

not:

```text
Android replaced by KMP
```

---

# 13. Native Android Is Still Native Android

If you choose native Android UI, you can continue using:

```text
Jetpack Compose
```

or:

```text
Views
```

along with:

```text
AndroidX
Lifecycle
Navigation
Material
```

The KMP shared module sits behind the Android application boundary.

Conceptually:

```text
              Android Application
                      │
          ┌───────────┴───────────┐
          ▼                       ▼
       Android UI             Shared KMP
          │                       │
          └───────────┬───────────┘
                      ▼
                   Result
```

---

# 14. The First Shared Feature

Let's create a very small example.

Imagine:

```text
Greeting
```

in:

```text
shared/src/commonMain/
```

Example:

```kotlin
class Greeting {

    fun message(): String {
        return "Hello from Kotlin Multiplatform!"
    }
}
```

Then Android consumes it.

This gives us our first proof that:

```text
commonMain
      ↓
Android
```

actually works.

---

# 15. Why Start With Something Small?

Imagine starting with:

```text
Authentication
+
Database
+
Networking
+
Dependency Injection
+
Navigation
+
Analytics
```

If the Android application fails, there are too many possible causes.

Instead:

```text
Greeting
    │
    ▼
Android
```

If that works, then add complexity one step at a time.

This is controlled engineering.

---

# 16. Verify Shared Code Is Really Shared

There is an important distinction between:

```text
Android-only code
```

and:

```text
commonMain code consumed by Android
```

Put the simple business logic in:

```text
shared/src/commonMain/
```

Then call it from:

```text
androidApp/
```

The structure should make the ownership obvious:

```text
shared/
└── src/
    └── commonMain/
        └── kotlin/
            └── com.example.app/
                └── Greeting.kt

androidApp/
└── src/
    └── main/
        └── kotlin/
            └── com.example.app/
                └── MainActivity.kt
```

---

# 17. The First Android Architecture

For our first project:

```text
             Android App
                  │
                  ▼
            Android UI
                  │
                  ▼
             Shared KMP
                  │
        ┌─────────┴─────────┐
        ▼                   ▼
      Domain              Data
```

Keep it simple.

The purpose of the first run is to understand the connection.

---

# 18. Running From the Command Line

You can also use Gradle from the terminal.

The exact task names depend on the project template and current tooling.

Start by discovering the available tasks:

```bash
./gradlew tasks
```

On Windows PowerShell:

```powershell
.\gradlew.bat tasks
```

Look for Android application and build-related tasks.

This is preferable to assuming a task name copied from another KMP project will exist in yours.

---

# 19. Why Task Discovery Matters

KMP tooling evolves.

A project created today may not expose exactly the same task names as a project created with an older Kotlin or Android Gradle Plugin version.

Therefore:

```text
Current Project
      │
      ▼
Discover Tasks
      │
      ▼
Choose Correct Task
```

is more reliable than:

```text
Old Tutorial
      │
      ▼
Copy Command
      │
      ▼
Build Failure
```

---

# 20. Android Studio vs Command Line

Both approaches are useful.

### Android Studio

Best for:

```text
Quick development
Debugging
Logcat
Device interaction
Breakpoints
UI inspection
```

### Command Line

Best for:

```text
Automation
CI
Build scripts
Reproducibility
Understanding Gradle
```

A strong Android/KMP developer should be comfortable with both.

---

# 21. Debugging the First Run

If the application does not start, separate the problem into layers.

```text
Run Failed
    │
    ├── Device?
    │
    ├── Android SDK?
    │
    ├── Gradle?
    │
    ├── Compilation?
    │
    ├── Installation?
    │
    └── Runtime?
```

This prevents random troubleshooting.

---

# 22. Layer 1 — Device

Check:

```text
Is the emulator running?
Is the physical device connected?
Is USB debugging enabled?
Does ADB see the device?
```

Use:

```bash
adb devices
```

If there is no device:

```text
KMP is not the problem yet.
```

Fix the Android environment first.

---

# 23. Layer 2 — Build

If Gradle fails:

```text
Build failed
```

look at:

```text
Which task failed?
Which module?
Which source set?
Which dependency?
```

For example:

```text
commonMain
     │
     ▼
Dependency Error
     │
     ▼
Android Compilation
     │
     ▼
Build Failure
```

The failure may originate in shared code even though you are launching Android.

---

# 24. Layer 3 — Installation

Suppose:

```text
Build → PASS
```

but:

```text
Install → FAIL
```

Now investigate Android device state:

```text
Storage
ADB
Package installation
Existing application state
Device API level
Signing
```

This is no longer primarily a KMP source-code problem.

---

# 25. Layer 4 — Runtime

Suppose:

```text
Build → PASS
Install → PASS
Launch → FAIL
```

Now use Android debugging tools.

Look at:

```text
Logcat
Stack trace
Application startup
Dependency initialization
Activity lifecycle
```

The distinction is important:

```text
Build Error
≠
Install Error
≠
Runtime Error
```

Each requires a different investigation.

---

# 26. Logcat

For Android runtime issues, Logcat remains one of your most useful tools.

Conceptually:

```text
Android App
     │
     ▼
Runtime Event
     │
     ▼
Logcat
     │
     ▼
Stack Trace / Message
     │
     ▼
Root Cause
```

If shared code throws an exception, the Android application can surface that exception through the normal Android runtime.

---

# 27. Debugging Shared Kotlin Code

This is where KMP becomes especially interesting for Android developers.

You can debug code that lives in:

```text
commonMain
```

while running the Android application.

Conceptually:

```text
Android App
     │
     ▼
Shared Kotlin
     │
     ▼
Breakpoint
     │
     ▼
Debug
```

The exact debugger experience depends on the IDE, Kotlin tooling, and project configuration.

But the important idea is that shared code is still normal Kotlin code participating in a platform compilation.

---

# 28. Set a Breakpoint in `commonMain`

Suppose:

```kotlin
class Greeting {

    fun message(): String {
        return "Hello from Kotlin Multiplatform!"
    }
}
```

Place a breakpoint inside:

```text
message()
```

Then run the Android application in debug mode.

The execution path should be conceptually:

```text
Android UI
    │
    ▼
Greeting.message()
    │
    ▼
Breakpoint
    │
    ▼
Return Value
    │
    ▼
Android UI
```

This is a powerful mental shift.

Shared code is not an abstract idea.

It is executable code.

---

# 29. Inspecting Variables

When the debugger stops in shared code, inspect:

```text
Parameters
Local Variables
Object State
Call Stack
Return Values
```

This is exactly the kind of debugging you already do in native Android development.

The difference is where the source file lives:

```text
commonMain
```

rather than:

```text
androidApp
```

---

# 30. A Shared Use Case Example

Consider:

```kotlin
class CalculateCartTotalUseCase {

    fun execute(
        prices: List<Double>
    ): Double {
        return prices.sum()
    }
}
```

The Android UI might call:

```text
CheckoutScreen
       │
       ▼
CalculateCartTotalUseCase
       │
       ▼
commonMain
       │
       ▼
Total
       │
       ▼
CheckoutScreen
```

The same use case could later be consumed by iOS.

That is the real value of KMP.

---

# 31. The Android Run Proves Only One Target

After a successful Android run, remember:

```text
Android → PASS
```

does not mean:

```text
iOS → PASS
```

It means:

```text
Android Target
     │
     ▼
Successfully Built and Ran
```

The iOS target still needs independent validation.

This is one of the most important habits in multiplatform development.

---

# 32. Don't Declare "Multiplatform Works" Yet

A common mistake is:

> "The Android app runs, so KMP is working."

More accurately:

> **The Android target of the KMP project is working.**

A true multiplatform milestone requires:

```text
Android
   +
iOS
```

and eventually:

```text
Shared Behavior
works correctly
on both
```

---

# 33. Android-Specific Code

Sometimes Android really needs platform-specific code.

For example:

```text
Android Context
```

or:

```text
Android-specific system service
```

That code can live in:

```text
androidMain
```

or the Android application module depending on its responsibility.

Conceptually:

```text
commonMain
     │
     ▼
Abstraction
     ▲
     │
androidMain
     │
     ▼
Android API
```

---

# 34. Don't Force Android Code Into Common

Suppose you need:

```text
Context
```

Do not try to make:

```text
commonMain
```

know about `Context`.

Instead:

```text
Shared Logic
      │
      ▼
Platform Abstraction
      ▲
      │
Android Implementation
      │
      ▼
Android Context
```

This preserves the shared boundary.

---

# 35. Android UI Still Owns Android Concerns

For example:

```text
Permission Dialog
```

may belong naturally to Android UI.

Similarly:

```text
Activity Result
```

may belong to Android.

The shared layer can expose the business requirement:

```text
"Permission required"
```

while Android decides how that requirement is presented.

---

# 36. Example: Permission Boundary

Instead of:

```text
commonMain
    │
    ▼
Android Permission API
```

prefer:

```text
commonMain
    │
    ▼
Permission Requirement
    │
    ▼
Android UI
    │
    ▼
Android Permission API
```

The shared layer describes behavior.

The platform layer handles platform mechanics.

---

# 37. Running a Real Feature

Once the greeting works, create a slightly more realistic feature.

For example:

```text
Product List
```

Shared:

```text
Product
ProductRepository
GetProductsUseCase
```

Android:

```text
ProductScreen
```

Architecture:

```text
                Product Screen
                     │
                     ▼
             GetProductsUseCase
                     │
                     ▼
             ProductRepository
                     │
                     ▼
                 commonMain
```

This is closer to how a production application will behave.

---

# 38. The Android Data Flow

A simplified flow:

```text
User
 │
 ▼
Android UI
 │
 ▼
ViewModel / State Holder
 │
 ▼
Use Case
 │
 ▼
Repository
 │
 ▼
Network / Database
 │
 ▼
Result
 │
 ▼
State
 │
 ▼
Android UI
```

The important part is:

```text
Use Case
Repository
Business Rules
```

can live in shared code when the behavior is genuinely common.

---

# 39. What Stays Android-Specific?

Potentially:

```text
Activity
Compose Screen
Android Navigation
Android Permissions
Android Notifications
Android Services
Android-specific SDK integrations
```

The exact boundary depends on the architecture.

KMP is not about maximizing the percentage of code in `commonMain`.

It is about sharing the right code.

---

# 40. A Better Metric Than "Percentage Shared"

Avoid saying:

```text
"We share 90% of our code."
```

without context.

A better question is:

```text
Which responsibilities are shared?
Which responsibilities remain native?
Why?
```

For example:

```text
Shared:
✓ Domain
✓ Networking
✓ Repository
✓ Validation

Android:
✓ Android UI
✓ Android permissions

iOS:
✓ iOS UI
✓ Apple-specific integrations
```

That tells a much more useful architectural story.

---

# 41. First Android Run Checklist

Use this checklist:

```text
[ ] Android SDK is configured
[ ] Emulator or device is available
[ ] ADB sees the device
[ ] Project syncs successfully
[ ] Gradle build succeeds
[ ] Android run configuration is available
[ ] Android application installs
[ ] Application launches
[ ] Shared code is executed
[ ] Breakpoint in commonMain can be investigated
[ ] Logcat is available
```

If all of these work, the Android side is ready for the next stage.

---

# 42. Common Beginner Mistakes

### Mistake 1 — Running Before Building

Start with:

```text
Build
```

then:

```text
Run
```

This gives you a clean baseline.

---

### Mistake 2 — Debugging KMP When ADB Is Broken

If:

```bash
adb devices
```

doesn't show a device, fix the Android environment first.

---

### Mistake 3 — Putting Android APIs in `commonMain`

If the code requires:

```text
Context
Activity
Android SDK
```

it should not casually move into shared code.

---

### Mistake 4 — Assuming Android Success Means Multiplatform Success

It doesn't.

You have validated:

```text
Android target
```

not:

```text
Every KMP target
```

---

### Mistake 5 — Adding Infrastructure Too Early

Don't add:

```text
Database
DI
Analytics
Authentication
```

before the first shared feature can run.

---

### Mistake 6 — Using `clean` After Every Failure

First identify the failure.

Then decide whether a clean build is actually required.

---

# 43. The First Successful Android Run

The milestone looks like this:

```text
                 KMP Project
                      │
                      ▼
                Shared Module
                      │
                      ▼
                 Android Target
                      │
                      ▼
                   Gradle
                      │
                      ▼
                     APK
                      │
                      ▼
               Android Device
                      │
                      ▼
                  App Running
```

Now the project is no longer just configuration.

It is executable.

---

# 44. What We Have Proven

After the first successful run:

```text
✓ Android environment works
✓ Gradle can build the Android target
✓ Shared module is connected
✓ commonMain can participate in Android compilation
✓ Android application can consume shared code
✓ Application can run on a real Android runtime
```

That is a meaningful milestone.

---

# 45. What We Have Not Proven

We have not yet proven:

```text
iOS compilation
iOS runtime
iOS UI integration
Apple-specific dependencies
Cross-platform behavior
Shared logic on both platforms
```

Those need separate validation.

---

# 46. The Complete Android Flow

Keep this diagram in mind:

```text
                         Developer
                             │
                             ▼
                       Android Studio
                             │
                             ▼
                        Run Command
                             │
                             ▼
                           Gradle
                             │
               ┌─────────────┴─────────────┐
               ▼                           ▼
           androidApp                  shared
               │                           │
               │                    ┌──────┴──────┐
               │                    ▼             ▼
               │               commonMain    androidMain
               │                    │             │
               └────────────────────┴─────────────┘
                             │
                             ▼
                      Android Compilation
                             │
                             ▼
                            APK
                             │
                             ▼
                    Emulator / Device
                             │
                             ▼
                         Application
```

This is the path from source code to a running Android application.

---

# 47. The Deeper Lesson

Running Android in a KMP project should not feel like:

```text
"Nothing changed from Android."
```

Nor should it feel like:

```text
"Everything is different."
```

The correct understanding is:

```text
Android development remains Android development.
```

KMP adds:

```text
A shared Kotlin layer
```

between the platform application and the platform-specific implementation.

---

# Chapter Takeaways

> [!TIP]
> **Your first Android run is the first proof that the KMP architecture is connected to a real platform.**

Remember:

1. Build the project before making significant changes.
2. Make sure the Android SDK and device/emulator are working.
3. Verify the device with `adb devices` when troubleshooting device connectivity.
4. Run the Android application target, not just the shared module.
5. The Android app consumes the shared KMP module through the Gradle build.
6. `commonMain` code can participate in Android compilation.
7. `androidMain` contains Android-specific shared-module implementation.
8. Android UI remains Android-specific when using a native UI architecture.
9. KMP does not replace Android APIs, Android Studio, or the Android toolchain.
10. A successful Android run validates the Android target only.
11. Android success does not prove that the iOS target works.
12. Use small shared features to validate the architecture.
13. Keep platform APIs behind clear platform boundaries.
14. Use Logcat and normal Android debugging techniques for runtime failures.
15. Shared Kotlin code can be debugged while running the Android application.
16. Don't assume a copied Gradle task name exists in your current project.
17. Discover current tasks with the project's Gradle configuration.
18. Don't use `clean` as a universal fix.
19. Measure sharing by responsibilities, not by an arbitrary percentage.
20. The first successful Android run establishes the first real runtime milestone of the KMP project.

---

# Final Mental Model

When you press **Run** for Android, don't think:

```text
Run
 ↓
APK
 ↓
Done
```

Think:

```text
                         Android Run
                              │
                              ▼
                            Gradle
                              │
                    ┌─────────┴─────────┐
                    ▼                   ▼
                androidApp           shared
                    │                   │
                    │          ┌────────┴────────┐
                    │          ▼                 ▼
                    │      commonMain       androidMain
                    │          │                 │
                    └──────────┴─────────────────┘
                               │
                               ▼
                        Android Compilation
                               │
                               ▼
                              APK
                               │
                               ▼
                       Android Device
                               │
                               ▼
                           Running App
```

And remember the bigger picture:

```text
                         KMP
                          │
              ┌───────────┴───────────┐
              ▼                       ▼
           Android                   iOS
              │                       │
        Native Android          Native Apple
          Application            Application
              ▲                       ▲
              │                       │
              └────── Shared KMP ─────┘
```

> **KMP does not replace Android development. It gives Android and iOS a shared Kotlin layer while allowing each platform to remain native where it needs to be. The first Android run is where that idea becomes something you can actually execute, debug, and observe.**
