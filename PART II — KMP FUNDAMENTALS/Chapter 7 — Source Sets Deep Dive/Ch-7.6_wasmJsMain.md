# Chapter 7 — Source Sets Deep Dive

## Part 6 — `wasmJsMain`

> **`wasmJsMain` is the source-set boundary for Kotlin/Wasm JavaScript-interoperable code, allowing KMP applications to share logic while targeting the Web through WebAssembly.**

As Kotlin Multiplatform expands beyond mobile and desktop, the Web becomes an important target.

Kotlin/Wasm allows Kotlin code to be compiled to WebAssembly, with JavaScript interoperability available where browser or JavaScript APIs are required.

A project targeting WebAssembly may contain a structure such as:

```text
shared/
└── src/
    ├── commonMain/
    ├── commonTest/
    ├── androidMain/
    ├── iosMain/
    ├── desktopMain/
    └── wasmJsMain/
```

The architectural relationship can be visualized as:

```text
                         commonMain
                             │
                    Shared application logic
                             │
        ┌────────────────────┼────────────────────┐
        ▼                    ▼                    ▼
   androidMain           iosMain            wasmJsMain
        │                    │                    │
   Android APIs          Apple APIs        Web/Wasm APIs
                                                   │
                                              Browser
                                                   │
                                           JavaScript interop
```

The goal is not to move the entire application into `wasmJsMain`.

The goal is to keep genuinely reusable logic in `commonMain` and place WebAssembly-specific behavior at the Web boundary.

---

# 1. What Is `wasmJsMain`?

`wasmJsMain` is a Kotlin Multiplatform source set associated with a WebAssembly target that interoperates with JavaScript.

It is useful for code that requires capabilities specific to:

```text
Kotlin/Wasm
+
JavaScript interoperability
+
Browser environment
+
Web platform APIs
```

Examples can include:

```text
Browser APIs
DOM integration
JavaScript interop
Web-specific storage
Browser events
Web-specific application bootstrap
```

The exact source-set hierarchy depends on the Kotlin Multiplatform targets configured by the project.

---

# 2. Why Does `wasmJsMain` Matter?

Traditional mobile KMP architecture often looks like:

```text
commonMain
    ├── Android
    └── iOS
```

Modern multiplatform applications can expand:

```text
                    commonMain
                         │
       ┌─────────┬───────┼────────┬──────────┐
       ▼         ▼       ▼        ▼          ▼
    Android     iOS    Desktop   Web/Wasm   Other
```

The Web introduces a different execution environment.

A browser provides:

```text
DOM
Window
Document
Storage
Events
Web APIs
JavaScript
```

These capabilities should not leak into `commonMain`.

That is where `wasmJsMain` becomes useful.

---

# 3. Kotlin/Wasm in the KMP Model

A simplified compilation model is:

```text
Kotlin source
     │
     ▼
Kotlin compiler
     │
     ▼
WebAssembly
     │
     ▼
Browser / Web runtime
     │
     ├── Web APIs
     └── JavaScript interoperability
```

The important architectural boundary is:

```text
Shared Kotlin logic
        ↓
Kotlin/Wasm target
        ↓
Browser environment
```

---

# 4. `commonMain` vs `wasmJsMain`

The distinction is similar to the other source sets.

### `commonMain`

Good candidates:

```text
Domain models
Business rules
Use cases
Validation
Shared state
Networking abstractions
Repository interfaces
Serialization
Application logic
```

### `wasmJsMain`

Good candidates:

```text
Browser APIs
DOM integration
JavaScript interop
Browser storage integration
Web-specific event handling
Web-specific bootstrap
Web platform adapters
```

The question is always:

> **Does this code require the WebAssembly/Web environment?**

If not, keep it common.

---

# 5. The Browser Is a Platform

One of the most important concepts is that the browser itself is a platform.

A browser provides capabilities that do not exist in the same form on Android, iOS, or desktop:

```text
DOM
Window
Document
Location
History
Web Storage
Fetch
Web Workers
Browser events
JavaScript APIs
```

These capabilities belong at the Web boundary.

Conceptually:

```text
commonMain
     │
     ▼
Web capability
     │
     ▼
wasmJsMain
     │
     ▼
Browser API
```

---

# 6. WebAssembly Does Not Mean "No JavaScript"

A common misconception is:

> "If the application uses WebAssembly, JavaScript disappears."

That is not the right mental model.

A browser application may involve:

```text
Kotlin
   ↓
WebAssembly
   ↓
Browser runtime
   ↓
JavaScript/Web APIs
```

JavaScript interoperability can still be important because browsers expose many capabilities through Web APIs and JavaScript interfaces.

---

# 7. JavaScript Interoperability

Web applications frequently need to communicate with JavaScript.

Examples include:

```text
Browser APIs
Existing JavaScript libraries
Third-party Web SDKs
DOM APIs
Browser events
JavaScript callbacks
```

When such integration is required, it belongs at the Web-specific boundary.

That usually means:

```text
wasmJsMain
```

rather than:

```text
commonMain
```

---

# 8. Keep JavaScript Types Out of `commonMain`

Avoid designing shared business APIs around browser-specific JavaScript types.

For example, a domain model should not require:

```text
JavaScript DOM object
```

to represent a business concept.

Instead:

```text
Browser event
    ↓
wasmJsMain adapter
    ↓
Shared application event
    ↓
commonMain
```

This keeps the shared layer portable.

---

# 9. DOM Integration

The DOM is one of the most important Web platform APIs.

It represents the structure of a Web page:

```text
Document
 └── HTML
      └── Body
           ├── Header
           ├── Main
           └── Footer
```

If your Kotlin/Wasm application directly interacts with DOM APIs, that integration belongs at the Web boundary.

For example:

```text
wasmJsMain
    ↓
DOM adapter
    ↓
Application state
```

---

# 10. Compose Multiplatform and Wasm

Kotlin/Wasm is particularly interesting for Compose Multiplatform.

A Compose-based application can potentially share:

```text
UI
State
View models
Business logic
Navigation
Networking
```

across multiple targets.

Conceptually:

```text
                    common/shared UI
                           │
              ┌────────────┼────────────┐
              ▼            ▼            ▼
           Android        iOS        Web/Wasm
                                         │
                                    wasmJsMain
                                         │
                                      Browser
```

The exact supported APIs and project configuration depend on the versions and libraries used.

---

# 11. Shared UI Does Not Mean Shared Browser APIs

Suppose a shared Compose UI needs to download a file.

The user-facing action can be shared:

```text
DownloadReport
```

But the actual browser download mechanism is Web-specific.

Therefore:

```text
commonMain
    ↓
DownloadReport
    ↓
DownloadService
    ↑
    │
wasmJsMain
    ↓
Browser download API
```

This is the correct separation.

---

# 12. Browser Storage

Web applications commonly use browser-managed storage.

Examples include:

```text
localStorage
sessionStorage
IndexedDB
```

These are browser capabilities.

If an application uses them directly, the integration belongs at the Web boundary.

The shared application can depend on an abstraction:

```kotlin
interface KeyValueStorage {
    suspend fun get(key: String): String?
    suspend fun put(key: String, value: String)
    suspend fun remove(key: String)
}
```

The Web implementation can use an appropriate browser storage mechanism.

---

# 13. `localStorage`

`localStorage` provides simple browser-side key/value storage.

It can be useful for:

```text
Preferences
Small configuration values
Non-sensitive UI state
Feature settings
```

However, it should not automatically be treated as a secure credential store.

A shared abstraction can hide the browser implementation.

---

# 14. `sessionStorage`

`sessionStorage` is associated with the browser session.

It can be useful for temporary state such as:

```text
Temporary navigation information
Session-level UI preferences
Short-lived application state
```

Again, the shared layer should depend on a capability rather than directly on browser storage APIs.

---

# 15. IndexedDB

For larger structured client-side data, Web applications may use:

```text
IndexedDB
```

This can be useful for:

```text
Offline data
Caches
Large structured datasets
Local application state
```

If the chosen persistence solution is Web-specific, its implementation belongs near:

```text
wasmJsMain
```

---

# 16. Browser Cookies

Cookies are another Web-specific mechanism.

They may participate in:

```text
Authentication
Session management
Preferences
Tracking
```

Cookie handling should be carefully designed, especially for security-sensitive data.

The application architecture can isolate cookie access behind a Web-specific adapter.

---

# 17. Browser Networking

Networking can often remain shared when the selected KMP HTTP stack supports the required targets.

For example:

```text
commonMain
    ↓
API client
    ↓
Repository
```

The target-specific networking implementation may be handled by the selected HTTP engine.

Do not move the entire networking layer to `wasmJsMain` simply because the application runs in a browser.

---

# 18. Browser Fetch

At the Web boundary, browser networking may involve:

```text
Fetch API
```

The browser imposes its own rules around:

```text
CORS
Credentials
Headers
Cookies
Security policies
```

These concerns should be handled at the Web/platform layer where appropriate.

---

# 19. CORS

Cross-Origin Resource Sharing is a browser security mechanism.

For example:

```text
Web app
   ↓
https://app.example
   ↓
API
   ↓
https://api.example
```

The browser may enforce CORS rules.

This is not a business-logic concern.

Your shared application can know:

```text
Request failed
```

while the Web-specific environment determines whether the request is permitted.

---

# 20. Browser URL

A browser application may need access to:

```text
Current URL
Path
Query parameters
Hash
Origin
```

These are Web-specific concepts.

A shared navigation model can be:

```text
Route(
    path = "/orders/123"
)
```

while `wasmJsMain` handles:

```text
Browser URL
    ↓
Route
```

---

# 21. Browser History

The browser provides history navigation:

```text
Back
Forward
Push state
Replace state
```

A shared navigation abstraction can remain platform-neutral.

The browser-specific implementation can integrate with:

```text
History API
```

from the Web boundary.

---

# 22. Deep Linking on the Web

A Web application may receive:

```text
https://example.com/orders/123
```

The Web-specific layer can extract:

```text
/orders/123
```

and translate it into a shared route.

Then:

```text
commonMain
    ↓
OrderDetailsRoute
```

can handle the application behavior.

---

# 23. Browser Events

Web applications receive events such as:

```text
click
keydown
resize
scroll
visibilitychange
online
offline
```

These are platform events.

The Web layer can translate them into application-level events.

For example:

```text
Browser online event
        ↓
wasmJsMain
        ↓
ConnectivityChanged(true)
        ↓
commonMain
```

---

# 24. Online and Offline Detection

A Web application may need to know whether the browser reports network connectivity.

The platform layer can expose:

```kotlin
interface ConnectivityMonitor {
    val isOnline: StateFlow<Boolean>
}
```

The Web implementation can observe browser events.

The synchronization logic can remain shared.

---

# 25. Browser Visibility

Browsers can notify applications when a page becomes:

```text
Visible
Hidden
```

This can matter for:

```text
Polling
Animations
Refresh behavior
Resource usage
Analytics
```

The browser event belongs to the Web boundary.

The application's reaction can remain common.

---

# 26. Browser Window

The browser `window` object exposes many capabilities.

Examples:

```text
Location
Timers
Screen information
Events
Navigation
Storage
```

Do not expose the entire browser window object to shared business logic.

Instead create narrow abstractions around the exact capability required.

---

# 27. Browser Timers

The browser provides timer mechanisms.

A shared application may need:

```text
Retry
Timeout
Polling
Delayed actions
```

Prefer multiplatform coroutine mechanisms for shared business behavior when possible.

Use browser-specific timer APIs only when the Web environment genuinely requires them.

---

# 28. Web Workers

For certain workloads, Web Workers can move processing away from the main browser thread.

Potential use cases include:

```text
Large calculations
Data processing
Parsing
CPU-intensive operations
```

Worker integration is Web-specific.

The application can expose:

```text
BackgroundProcessor
```

while `wasmJsMain` provides the browser implementation.

---

# 29. Browser Main Thread

UI work must respect browser execution constraints.

A Web application needs to avoid blocking the main UI thread with expensive synchronous work.

Shared coroutine-based architecture can help structure asynchronous operations.

The Web-specific layer should integrate with browser execution requirements.

---

# 30. Browser APIs as Platform Dependencies

A useful rule:

```text
Browser API
    ↓
wasmJsMain
```

Examples:

```text
window
document
localStorage
sessionStorage
history
location
navigator
browser events
```

Do not allow those APIs to spread across every shared module.

---

# 31. Browser File Upload

A Web application may allow users to upload files.

The browser provides:

```text
File
FileList
File input
Drag-and-drop
```

The Web-specific layer can translate the browser file into a shared representation.

For example:

```text
Browser File
    ↓
wasmJsMain
    ↓
UploadedFile
    ↓
commonMain
```

---

# 32. Browser File Download

The reverse flow is:

```text
commonMain
    ↓
Generated content
    ↓
wasmJsMain
    ↓
Browser download
```

This is a clean platform boundary.

---

# 33. Drag and Drop on the Web

Web applications can support:

```text
Drag file
Drop file
Read file
Process file
```

The browser event is Web-specific.

The resulting application command can remain shared:

```text
ImportFile
```

---

# 34. Clipboard on the Web

Browser clipboard APIs can support:

```text
Copy text
Paste text
```

A shared abstraction can look like:

```kotlin
interface Clipboard {
    suspend fun copy(text: String)
    suspend fun paste(): String?
}
```

The Web implementation handles browser permissions and APIs.

---

# 35. Browser Permissions

Some Web APIs require user permission.

Examples can include:

```text
Clipboard
Notifications
Camera
Microphone
Location
```

Permission handling is part of the Web environment.

The shared layer should ideally receive a meaningful result:

```text
Granted
Denied
Unavailable
```

rather than depending on browser-specific objects.

---

# 36. Browser Notifications

Web applications can use browser notifications when permitted.

A shared abstraction might be:

```text
NotificationService
```

with the Web implementation handling:

```text
Notification API
Permission state
Browser limitations
```

---

# 37. Browser Geolocation

If an application needs location:

```text
commonMain
    ↓
LocationProvider
    ↑
    │
wasmJsMain
    ↓
Browser Geolocation API
```

The domain should not know about browser permission dialogs or JavaScript callback objects.

---

# 38. Camera and Microphone

Browser applications can access media devices when permitted.

The integration involves:

```text
Camera
Microphone
Media streams
Browser permissions
```

This is clearly a platform boundary.

The shared application can work with an abstraction such as:

```text
MediaCapture
```

without depending directly on browser APIs.

---

# 39. Web APIs and Security

Browser APIs operate inside a security model.

Examples include:

```text
Same-origin policy
CORS
Permissions
Secure contexts
Content Security Policy
Sandboxing
```

KMP architecture should respect these constraints.

A multiplatform architecture cannot remove browser security rules.

---

# 40. Authentication in the Browser

Authentication can be shared conceptually:

```text
Login
Logout
Session
Token refresh
User state
```

But browser-specific authentication may involve:

```text
Cookies
Redirects
Browser storage
OAuth/OIDC callbacks
Popup windows
```

Those integrations belong near:

```text
wasmJsMain
```

---

# 41. OAuth Redirect Flow

A typical browser authentication flow may look like:

```text
Application
    ↓
Open authentication page
    ↓
User signs in
    ↓
Redirect to application
    ↓
wasmJsMain receives URL
    ↓
Shared authentication layer
```

The redirect parsing can be platform-specific.

The authentication state can remain shared.

---

# 42. Token Storage

Do not automatically put authentication tokens into browser storage simply because it is convenient.

Security requirements should determine the mechanism.

The architecture should provide a narrow abstraction:

```text
CredentialStore
```

and choose an appropriate Web implementation.

---

# 43. Browser Lifecycle

Mobile developers are used to:

```text
Activity
Fragment
App lifecycle
```

Desktop developers may think in:

```text
Window
Application
Process
```

The browser has different lifecycle concepts:

```text
Page load
Visibility
Navigation
Page freeze
Page restore
Unload
```

These events should be adapted into application-level lifecycle signals where needed.

---

# 44. Browser Lifecycle Adapter

For example:

```text
Browser visibilitychange
        ↓
wasmJsMain
        ↓
AppLifecycle.Background
        ↓
commonMain
```

The shared application does not need to understand the raw DOM event.

---

# 45. Browser Resize

Window resizing is a Web/UI concern.

For example:

```text
Browser resize
    ↓
wasmJsMain
    ↓
WindowSizeChanged
    ↓
Shared UI/state
```

The actual browser event handling belongs at the platform boundary.

---

# 46. Responsive Design

A shared Compose UI can react to:

```text
Window width
Window height
Screen density
Input mode
```

The UI layer can adapt presentation.

Business logic should not depend on:

```text
1024px
768px
Desktop browser
Mobile browser
```

unless that information is genuinely part of the application requirement.

---

# 47. Browser Input

Web applications can receive:

```text
Mouse
Keyboard
Touch
Pointer
Gamepad
```

The UI toolkit may abstract many of these.

When direct browser integration is necessary, keep it in:

```text
wasmJsMain
```

or the appropriate UI/platform layer.

---

# 48. Web Accessibility

Browser accessibility includes:

```text
Keyboard navigation
Screen readers
ARIA
Focus
Semantic HTML
Contrast
Reduced motion
```

If the UI toolkit provides the appropriate abstraction, use it.

Direct DOM accessibility manipulation belongs at the Web boundary.

---

# 49. Browser Resource Loading

Web applications may load:

```text
Images
Fonts
JavaScript resources
WebAssembly modules
Configuration
```

The mechanism is Web-specific.

Shared code should ideally work with logical resources rather than raw browser resource objects.

---

# 50. Browser Cache

Browsers may cache:

```text
HTTP resources
Application assets
Service-worker-managed data
```

Application caching strategy should be separated from browser cache implementation.

A shared repository can define:

```text
Cache policy
```

while the Web layer integrates with the available Web mechanisms.

---

# 51. Service Workers

Service workers can enable:

```text
Offline behavior
Caching
Background operations
Push notifications
```

They are part of the Web platform architecture.

When used, keep service-worker-specific integration outside common business logic.

---

# 52. Offline-First Web Applications

A KMP application targeting Web can still use the same conceptual architecture:

```text
UI
 ↓
Use Case
 ↓
Repository
 ↓
Local Data
 ↓
Network
```

The Web-specific persistence implementation may use browser storage.

The business synchronization rules can remain shared.

---

# 53. WebAssembly and Performance

WebAssembly can provide advantages for certain workloads, but it is not a magic solution for every Web performance problem.

Performance depends on:

```text
Application architecture
Rendering
Data volume
Interop frequency
Browser behavior
Network latency
Startup cost
Memory usage
```

Use measurement rather than assumptions.

---

# 54. JavaScript Interop Has a Cost

Crossing between Kotlin/Wasm and JavaScript boundaries can introduce complexity.

Therefore:

```text
Shared computation
```

should remain in Kotlin where appropriate, while JavaScript interop should be concentrated around the APIs that actually require it.

A clean boundary makes the integration easier to reason about.

---

# 55. Avoid Excessive Interop

Avoid designs where the application constantly performs:

```text
Kotlin
  ↔ JavaScript
  ↔ Kotlin
  ↔ JavaScript
```

for every small operation.

Instead:

```text
Kotlin shared logic
        ↓
Web adapter
        ↓
Browser API
```

Perform the necessary interop at a well-defined boundary.

---

# 56. WebAssembly Module Boundaries

A useful mental model is:

```text
Shared Kotlin application
        │
        ▼
Kotlin/Wasm
        │
        ▼
Browser integration layer
        │
        ▼
Web APIs
```

This makes it clear where platform-specific responsibilities live.

---

# 57. Web-Specific Dependency Injection

Suppose shared code requires:

```text
Clipboard
Storage
NotificationService
LocationProvider
```

The Web application can provide:

```text
WasmClipboard
WasmStorage
WasmNotificationService
WasmLocationProvider
```

The shared application receives them through dependency injection.

---

# 58. Web Composition Root

The Web entry point is an important place for dependency composition.

Conceptually:

```text
wasmJsMain
    ↓
Create Web services
    ↓
Create shared application
    ↓
Start Web UI
```

This prevents platform setup from leaking into the domain.

---

# 59. Web Testing

Testing should be divided into:

```text
commonTest
    ↓
Shared behavior

wasmJsTest
    ↓
Web/Wasm-specific behavior
```

The exact test source-set names and support depend on the configured targets.

The architectural distinction remains useful.

---

# 60. Common Tests for Web Applications

Test in `commonTest` where possible:

```text
Validation
Use cases
Repository behavior
State transitions
Business rules
Serialization
Domain logic
```

These tests provide high-value coverage without requiring a browser.

---

# 61. Web-Specific Tests

Web-specific tests can validate:

```text
Browser storage adapter
URL handling
Browser event adapters
Web APIs
JavaScript interoperability
Web-specific integration
```

These tests belong near the Web implementation.

---

# 62. Browser Environment Differences

A Web application can behave differently across:

```text
Chrome
Firefox
Safari
Edge
```

Browser compatibility should be considered separately from KMP source-set architecture.

Do not assume:

```text
WebAssembly support
```

means:

```text
Every browser API behaves identically.
```

---

# 63. Browser Support Strategy

A production Web application should validate:

```text
Target browsers
Required Web APIs
WebAssembly support
JavaScript interoperability requirements
UI toolkit support
Third-party library support
```

The KMP target is only one part of the compatibility story.

---

# 64. Web and Desktop Are Different

It is tempting to think:

```text
desktopMain
wasmJsMain
```

are almost the same because both can run on computers.

They are not.

Desktop applications generally have:

```text
Direct file system access
Process access
Native window management
System tray
OS integration
```

Browsers operate inside a sandbox and expose:

```text
Web APIs
Permissions
Security boundaries
Browser storage
DOM
```

The source sets reflect these differences.

---

# 65. Web and Android Are Different

Android applications have:

```text
Activity
Service
BroadcastReceiver
Android storage
Android permissions
```

The browser has:

```text
Page
Web APIs
Browser permissions
Storage sandbox
DOM
```

Do not design common code around either platform's lifecycle.

Create shared application abstractions where the behavior truly overlaps.

---

# 66. Web and iOS Are Different

iOS provides:

```text
UIKit/SwiftUI
Keychain
Apple lifecycle
Native file APIs
Apple frameworks
```

Web provides:

```text
DOM
Browser storage
Web security model
JavaScript APIs
```

Again, isolate the platform differences.

---

# 67. A Cross-Platform Capability Model

A mature KMP application may look like:

```text
                    commonMain
                         │
        ┌────────────────┼────────────────┐
        │                │                │
        ▼                ▼                ▼
   Android APIs      iOS APIs        Web APIs
        │                │                │
   androidMain        iosMain        wasmJsMain
```

The shared layer defines capabilities.

Each target provides the implementation.

---

# 68. Example: Storage

```text
                 Storage
                    │
          ┌─────────┼─────────┐
          ▼         ▼         ▼
      Android      iOS       Web
          │         │         │
       androidMain iosMain wasmJsMain
```

The business layer remains unaware of:

```text
SharedPreferences
UserDefaults
localStorage
IndexedDB
```

unless those details are explicitly required by the architecture.

---

# 69. Example: Notifications

```text
             NotificationService
                     │
          ┌──────────┼──────────┐
          ▼          ▼          ▼
       Android       iOS        Web
          │          │          │
      Native API  Apple API  Browser API
```

Same capability.

Different implementation.

---

# 70. Example: URL Opening

```text
commonMain
    ↓
UrlLauncher
    ↑
    │
┌───┼─────────────┐
│   │             │
▼   ▼             ▼
Android iOS    wasmJsMain
│   │             │
▼   ▼             ▼
Native Browser Browser
```

This is the KMP source-set model in practice.

---

# 71. Avoid Browser-Specific Business Rules

Do not write:

```text
if browser:
    validate order
else:
    validate order differently
```

unless the business requirement genuinely differs.

Browser-specific behavior should normally stay around:

```text
Presentation
Platform APIs
Security
Storage
Lifecycle
Integration
```

---

# 72. `wasmJsMain` as a Boundary

The most useful mental model is:

```text
commonMain
    │
    │ Shared intent
    ▼
Platform abstraction
    ▲
    │
wasmJsMain
    │
    ▼
Browser / WebAssembly environment
```

This is similar to:

```text
androidMain
iosMain
desktopMain
```

but with Web-specific constraints.

---

# 73. What Should Stay in `commonMain`?

A typical Web-capable KMP project may keep:

```text
Domain
Data models
Use cases
Repositories
Networking contracts
Serialization
Shared UI
State management
Validation
Business rules
```

in:

```text
commonMain
```

whenever the selected libraries support the required target.

---

# 74. What Belongs in `wasmJsMain`?

Typical examples include:

```text
Browser APIs
JavaScript interop
DOM adapters
Browser storage
Browser clipboard
Browser notifications
Browser URL handling
Browser lifecycle
Web-specific entry point
Web-specific integrations
```

The exact list depends on the application.

---

# 75. What Should Not Automatically Move to `wasmJsMain`?

Do not move:

```text
Repository
Use Case
Domain model
Business validation
Networking
State management
```

to `wasmJsMain` simply because the Web target uses them.

Only move code when it has a real Web-specific dependency.

---

# 76. Source-Set Decision Example

Suppose you need:

```text
Save user preferences
```

First ask:

```text
Can a multiplatform storage library provide this?
```

If yes:

```text
commonMain
```

If the implementation must directly use:

```text
localStorage
```

then:

```text
wasmJsMain
```

is a natural place for the adapter.

---

# 77. Source-Set Decision Example: URL

Requirement:

```text
Open an external URL.
```

Shared contract:

```text
UrlLauncher
```

Web implementation:

```text
wasmJsMain
```

Android implementation:

```text
androidMain
```

iOS implementation:

```text
iosMain
```

The feature itself remains shared.

---

# 78. Source-Set Decision Example: Browser Notification

Requirement:

```text
Show notification.
```

Shared:

```text
NotificationService
```

Web:

```text
wasmJsMain
```

The notification API itself is not part of the domain.

---

# 79. Source-Set Decision Example: Business Validation

Requirement:

```text
Order quantity cannot be negative.
```

This is not Web-specific.

It belongs in:

```text
commonMain
```

There is no reason to duplicate it for:

```text
Android
iOS
Desktop
Web
```

---

# 80. Source-Set Decision Example: Browser Resize

Requirement:

```text
React to browser width changes.
```

This is Web/UI-specific.

It belongs near:

```text
wasmJsMain
```

or the appropriate shared UI/platform integration layer.

The business domain should not depend directly on browser resize events.

---

# 81. Source-Set Decision Example: Authentication

Requirement:

```text
Refresh authentication session.
```

The core session logic may be:

```text
commonMain
```

The Web-specific cookie or browser redirect integration may be:

```text
wasmJsMain
```

This results in a smaller and more reusable platform layer.

---

# 82. Architecture With Web

A complete KMP architecture may look like:

```text
┌──────────────────────────────────────────┐
│                commonMain                │
│                                          │
│ Domain                                   │
│ Use Cases                                │
│ Repositories                             │
│ Shared State                             │
│ Shared UI                                │
│ Platform Contracts                       │
└─────────────────────┬────────────────────┘
                      │
        ┌─────────────┼──────────────┐
        ▼             ▼              ▼
   androidMain      iosMain      wasmJsMain
        │             │              │
   Android APIs    Apple APIs    Browser APIs
                                      │
                               JavaScript/Wasm
```

The source sets define the platform boundary.

---

# 83. Web-Specific Architecture Should Be Intentional

Do not add `wasmJsMain` code simply because the folder exists.

Every platform-specific class should answer:

> **Which Web capability requires this class?**

For example:

```text
BrowserClipboard
→ Clipboard API

BrowserStorage
→ Web Storage

BrowserLocation
→ Geolocation API
```

This makes the architecture understandable.

---

# 84. Documentation Matters

When introducing Web-specific integrations, document:

```text
Why the code is Web-specific
Which browser API it uses
What the shared abstraction is
What limitations exist
What permissions are required
```

This prevents future developers from accidentally moving Web-specific APIs into common code.

---

# 85. Dependency Management

When adding a Web-specific dependency, verify:

```text
Supported targets
Kotlin version compatibility
Wasm compatibility
Browser compatibility
JavaScript interop requirements
```

A library supporting JVM does not automatically support Kotlin/Wasm.

A library supporting JavaScript does not automatically provide the same API for Kotlin/Wasm.

Target compatibility must be checked explicitly.

---

# 86. Wasm Dependency Selection

Before adding a dependency to:

```text
commonMain
```

ask:

```text
Does it support all of my targets?
```

For example:

```text
Android ✓
iOS ✓
Desktop ✓
Wasm ✓
```

If one target is unsupported, the dependency may require:

```text
Platform-specific source set
```

or a different architecture.

---

# 87. Keep the Web Layer Thin

A healthy Web platform layer often looks like:

```text
wasmJsMain
    ├── BrowserStorage
    ├── BrowserClipboard
    ├── BrowserNotification
    ├── BrowserUrlLauncher
    └── WebApplicationEntryPoint
```

while:

```text
commonMain
    ├── Domain
    ├── UseCases
    ├── Repositories
    ├── State
    └── Shared UI
```

contains the majority of reusable behavior.

---

# 88. Thin Platform, Strong Core

The ideal architecture can be summarized as:

```text
        Strong shared core
               │
               ▼
       Small platform layer
               │
               ▼
       Native platform APIs
```

This is one of the strongest architectural benefits of KMP.

---

# 89. The Web Boundary Is Not a Limitation

Some developers see platform-specific source sets as duplication.

They are not.

A platform source set is a controlled boundary.

It prevents:

```text
Browser APIs
```

from contaminating:

```text
Business logic
```

while still allowing the application to use the Web platform fully.

---

# 90. `wasmJsMain` and Architectural Freedom

The goal of KMP is not:

```text
One source set for everything.
```

The goal is:

```text
Maximum meaningful reuse
+
Native platform capability
+
Clear boundaries
```

`wasmJsMain` supports exactly that principle for WebAssembly/Web applications.

---

# 91. Final Mental Model

Think of `wasmJsMain` as the Web adapter layer:

```text
                    commonMain
                         │
                   Shared intent
                         │
                  Platform contract
                         │
                         ▼
                    wasmJsMain
                         │
              ┌──────────┼──────────┐
              ▼          ▼          ▼
             DOM      Browser APIs  JS
              │          │          │
              └──────────┼──────────┘
                         ▼
                    Web Platform
```

The shared layer defines **what the application needs**.

The Web layer defines **how the browser provides it**.

---

# Chapter Takeaways

> [!TIP]
> **Use `wasmJsMain` for WebAssembly/Web-specific production code, especially when browser or JavaScript APIs are required. Keep business logic and genuinely multiplatform functionality in `commonMain`.**

Remember:

1. `wasmJsMain` is associated with Kotlin/Wasm JavaScript-interoperable targets.
2. It provides a useful Web-specific source-set boundary.
3. The exact source-set hierarchy depends on the configured KMP targets.
4. The browser should be treated as a platform.
5. Web APIs are platform capabilities.
6. Browser APIs should not leak into `commonMain`.
7. JavaScript interoperability belongs at the Web boundary when required.
8. DOM integration is Web-specific.
9. Browser storage is Web-specific.
10. `localStorage` is a browser capability.
11. `sessionStorage` is a browser capability.
12. IndexedDB is a browser capability.
13. Browser cookies are Web-specific.
14. Browser URL handling is Web-specific.
15. Browser history integration is Web-specific.
16. Browser lifecycle events are Web-specific.
17. Browser visibility events can be adapted into shared lifecycle signals.
18. Browser resize events belong near the Web/UI boundary.
19. Browser clipboard integration can belong in `wasmJsMain`.
20. Browser notifications can be implemented in `wasmJsMain`.
21. Browser geolocation is a Web-specific capability.
22. Camera and microphone access require Web-specific integration.
23. Browser permissions are platform concerns.
24. Browser security policies are platform concerns.
25. CORS is a browser concern, not business logic.
26. Web authentication redirects are platform-specific.
27. Browser credential storage should be designed with security in mind.
28. Shared authentication state can remain in `commonMain`.
29. Shared networking can remain common when the chosen stack supports Wasm.
30. Do not move the entire networking layer to `wasmJsMain` without a target-specific reason.
31. Browser file upload is platform-specific.
32. Browser file download is platform-specific.
33. Browser drag-and-drop integration is platform-specific.
34. Web Workers are Web-specific.
35. Browser main-thread constraints should influence Web architecture.
36. WebAssembly does not mean JavaScript interoperability disappears.
37. JavaScript interop should be concentrated at clear boundaries.
38. Avoid excessive Kotlin/Wasm ↔ JavaScript boundary crossings.
39. Compose Multiplatform can share UI across supported targets.
40. Shared UI does not require shared browser APIs.
41. Browser APIs should be wrapped behind focused abstractions when shared code needs them.
42. Prefer `Clipboard` over exposing a browser clipboard object.
43. Prefer `Storage` over exposing `localStorage` directly.
44. Prefer `UrlLauncher` over exposing browser navigation objects.
45. Prefer `NotificationService` over exposing browser notification APIs.
46. Keep domain models independent of browser objects.
47. Keep use cases independent of DOM types.
48. Keep repository contracts independent of browser APIs.
49. Translate browser events into application-level events.
50. Use dependency injection to provide Web implementations.
51. The Web entry point is a natural composition root.
52. Browser-specific services can be created at application startup.
53. Shared business logic should remain testable without a browser.
54. `commonTest` should cover shared behavior.
55. Web-specific tests should validate Web integrations.
56. Browser compatibility is separate from KMP source-set design.
57. Different browsers may expose different API behavior.
58. Verify browser support requirements before shipping.
59. Verify Wasm compatibility of dependencies.
60. JVM support does not imply Kotlin/Wasm support.
61. JavaScript support does not automatically imply identical Kotlin/Wasm APIs.
62. Check target support before placing dependencies in `commonMain`.
63. Keep Web-specific dependencies in the narrowest valid source set.
64. Do not force browser APIs into `commonMain`.
65. Do not duplicate business logic in `wasmJsMain`.
66. Business validation belongs in `commonMain`.
67. Shared state belongs in `commonMain` when target-independent.
68. Shared use cases belong in `commonMain` when target-independent.
69. Shared domain models belong in `commonMain`.
70. Browser storage implementations can belong in `wasmJsMain`.
71. Browser URL implementations can belong in `wasmJsMain`.
72. Browser notification implementations can belong in `wasmJsMain`.
73. Browser lifecycle adapters can belong in `wasmJsMain`.
74. Web-specific entry-point code belongs at the Web boundary.
75. `wasmJsMain` is not a replacement for `commonMain`.
76. `wasmJsMain` is a platform boundary.
77. A thin Web layer is usually easier to maintain.
78. A strong shared core should contain the majority of reusable application behavior.
79. The platform layer should adapt native capabilities to shared contracts.
80. Avoid one giant `WebService` abstraction.
81. Prefer focused platform capabilities.
82. Source sets communicate architecture through project structure.
83. Place code at the highest source set where its dependencies allow it.
84. Move code downward only when a real platform dependency exists.
85. WebAssembly and desktop are different execution environments.
86. Browser sandboxing is fundamentally different from desktop OS access.
87. Browser APIs should be treated as platform dependencies.
88. Native Web capabilities should not be hidden merely for the sake of reuse.
89. KMP is about meaningful reuse, not eliminating every platform-specific line.
90. **The best KMP Web architecture shares stable application logic while isolating browser and JavaScript-specific behavior.**

---

# Final Thought

`wasmJsMain` represents an important shift in the KMP journey.

The application is no longer limited to:

```text
Android + iOS
```

It can move toward:

```text
Android + iOS + Desktop + Web
```

without forcing every platform into the same implementation.

The architecture becomes:

```text
                         commonMain
                             │
                    Shared application
                             │
                    Platform contracts
                             │
        ┌────────────┬──────┼──────┬────────────┐
        ▼            ▼      ▼      ▼            ▼
   androidMain    iosMain desktopMain wasmJsMain
        │            │      │          │
     Android       Apple  Desktop    Browser
                                      │
                              WebAssembly + JS
```

The principle remains consistent:

> **Share the behavior that is truly common. Isolate the APIs that belong to the Web.**

That is how Kotlin Multiplatform can extend one application architecture across mobile, desktop, and Web without turning the shared layer into a collection of platform-specific assumptions.
