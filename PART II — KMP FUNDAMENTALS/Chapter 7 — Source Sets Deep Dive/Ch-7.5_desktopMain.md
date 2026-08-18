# Chapter 7 — Source Sets Deep Dive

## Part 5 — `desktopMain`

> **`desktopMain` gives desktop platforms a dedicated place for platform-specific behavior while keeping genuinely reusable application logic in `commonMain`.**

Kotlin Multiplatform is not limited to Android and iOS.

When a project targets desktop platforms, desktop-specific code needs its own source-set boundary. Depending on the targets and project configuration, this is commonly represented through:

```text
desktopMain
```

A simplified KMP source-set structure can look like:

```text
shared/
└── src/
    ├── commonMain/
    ├── commonTest/
    ├── androidMain/
    ├── iosMain/
    ├── desktopMain/
    └── desktopTest/
```

The architectural relationship becomes:

```text
                         commonMain
                             │
                  Shared application logic
                             │
              ┌──────────────┼──────────────┐
              ▼              ▼              ▼
         androidMain       iosMain      desktopMain
              │              │              │
         Android APIs     Apple APIs     Desktop APIs
```

The important idea is simple:

> **Desktop-specific implementation belongs at the desktop boundary; application behavior that does not depend on a platform should remain shared.**

---

# 1. What Is `desktopMain`?

`desktopMain` is a source-set convention used to organize production code shared by desktop targets.

It is particularly useful when a KMP project targets platforms such as:

```text
JVM desktop applications
Windows
macOS
Linux
```

The exact target configuration depends on the project.

The source set provides a place for implementation that is:

```text
Desktop-specific
+
Not necessarily Android-specific
+
Not necessarily iOS-specific
```

For example:

```text
Desktop file system
Desktop window integration
Desktop-specific process APIs
Desktop keyboard/mouse integrations
Desktop-specific persistence
Desktop application integrations
```

can potentially live in `desktopMain`.

---

# 2. Why `desktopMain` Exists

Imagine an application that shares:

```text
Business rules
Networking
Serialization
Repositories
Use cases
Domain models
State management
```

across:

```text
Android
iOS
Desktop
```

But the desktop application needs a native file-system integration.

The architecture can become:

```text
                       commonMain
                           │
                      FileStorage
                           │
             ┌─────────────┼─────────────┐
             ▼             ▼             ▼
        androidMain      iosMain      desktopMain
             │             │             │
        Android file    iOS files     Desktop files
        APIs            APIs          APIs
```

The common application does not need to know which operating system is providing the storage.

---

# 3. Desktop Does Not Mean "Windows Only"

A common mistake is assuming:

```text
desktopMain = Windows code
```

That is not necessarily true.

Depending on the target configuration, desktop code may be shared across multiple desktop targets.

For example:

```text
desktopMain
    │
    ├── JVM desktop logic
    │
    ├── macOS desktop logic
    ├── Windows desktop logic
    └── Linux desktop logic
```

The exact hierarchy depends on the KMP targets defined by the project.

If behavior is specific to only one operating system, a more specific source set or target-specific source set may be more appropriate.

---

# 4. `desktopMain` vs `commonMain`

The key distinction is dependency scope.

### `commonMain`

Use it for code that can run across all intended targets.

Examples:

```text
Business rules
Domain models
Use cases
Validation
Serialization
Shared networking
Shared repositories
Shared state
```

### `desktopMain`

Use it for code that requires desktop-specific capabilities.

Examples:

```text
Desktop file access
Desktop process integration
Desktop-specific UI integration
Desktop keyboard/mouse behavior
Desktop-specific OS integration
```

The decision should always be based on capability dependency.

---

# 5. A Simple Decision Rule

Ask:

> **Can this implementation compile and behave correctly on Android, iOS, and desktop without platform-specific APIs?**

If yes:

```text
commonMain
```

If it requires desktop APIs:

```text
desktopMain
```

If it is only required by one desktop operating system:

```text
OS-specific source set
```

or an appropriate target-specific implementation.

---

# 6. Desktop Architecture

A typical architecture can look like:

```text
┌──────────────────────────────────────┐
│              commonMain              │
│                                      │
│ Domain                               │
│ Use Cases                            │
│ Repositories                         │
│ Shared Models                        │
│ Shared State                         │
└───────────────────┬──────────────────┘
                    │
             Platform boundary
                    │
┌───────────────────┴──────────────────┐
│             desktopMain              │
│                                      │
│ Desktop adapters                     │
│ File system integration              │
│ Desktop services                     │
│ Native desktop integrations          │
└──────────────────────────────────────┘
```

This gives desktop functionality a clear boundary.

---

# 7. Desktop UI and `desktopMain`

Desktop UI can be built using different technologies.

For example:

```text
Compose Multiplatform
Java Swing
JavaFX
Native desktop integration
```

When using Compose Multiplatform, much of the UI can often remain in shared code.

For example:

```text
commonMain
    ↓
Shared Compose UI
    ↓
Desktop target
```

But desktop-specific integrations can still live in:

```text
desktopMain
```

This is an important distinction.

---

# 8. Shared UI vs Desktop UI

A desktop application may have:

```text
Shared UI
+
Desktop-specific behavior
```

For example:

```text
Shared:
    LoginScreen
    Dashboard
    ProductList
    Settings

Desktop-specific:
    Window management
    Keyboard shortcuts
    File dialogs
    System tray
    Native menus
```

The shared UI remains reusable.

The desktop-specific behavior stays at the platform boundary.

---

# 9. Compose Multiplatform and `desktopMain`

A common desktop KMP project may contain:

```text
commonMain
    ├── shared UI
    ├── state
    ├── view models
    └── business logic

desktopMain
    ├── desktop entry point
    ├── desktop-specific integrations
    └── desktop platform services
```

The desktop application can then use:

```text
common UI
+
desktop implementation
```

without duplicating the entire application.

---

# 10. Desktop Entry Point

Desktop applications generally require an application entry point.

For a Compose Multiplatform desktop application, this often includes a JVM `main` function.

Conceptually:

```kotlin
fun main() {
    application {
        Window(
            onCloseRequest = ::exitApplication
        ) {
            App()
        }
    }
}
```

The exact implementation depends on the project and Compose Multiplatform version.

The important architectural point is:

```text
Desktop application bootstrap
→ desktop boundary
```

while:

```text
App()
→ can remain shared
```

when the UI is designed for sharing.

---

# 11. Desktop Application Bootstrap

The bootstrap layer is responsible for things such as:

```text
Creating the application
Creating windows
Initializing desktop services
Registering system integrations
Starting shared UI
```

A conceptual flow is:

```text
Desktop main()
      ↓
Create platform services
      ↓
Initialize shared application
      ↓
Launch shared UI
```

---

# 12. Desktop Window Management

Window behavior is often desktop-specific.

Examples:

```text
Window size
Window position
Multiple windows
Window title
Minimize/maximize
Window state
Native window events
```

The application logic should not depend directly on the window implementation.

Instead:

```text
commonMain
    ↓
Window capability
    ↓
desktopMain
    ↓
Desktop window APIs
```

---

# 13. Multiple Windows

Desktop applications commonly support multiple windows.

For example:

```text
Main Window
    │
    ├── Settings Window
    ├── Reports Window
    └── Details Window
```

Window creation and management can remain at the desktop application boundary.

Shared business state can still remain in:

```text
commonMain
```

This separation prevents window management from contaminating business logic.

---

# 14. Desktop File System

The file system is one of the most common desktop-specific capabilities.

The application might need:

```text
Open file
Save file
Read directory
Write file
Delete file
Watch file changes
```

Common:

```kotlin
interface FileStorage {
    suspend fun read(path: String): ByteArray
    suspend fun write(path: String, data: ByteArray)
}
```

Desktop:

```text
desktopMain
    ↓
JVM / desktop file APIs
```

Android and iOS can provide their own implementations.

---

# 15. File Picker

A desktop application may provide:

```text
File → Open
File → Save
```

The native file chooser is platform-specific.

A shared application can express:

```text
OpenDocument
```

while the desktop layer handles:

```text
Native file chooser
```

The architecture becomes:

```text
commonMain
    ↓
DocumentPicker
    ↓
desktopMain
    ↓
Desktop file dialog
```

---

# 16. Desktop Paths

File paths can vary between operating systems.

For example:

```text
Windows:
C:\Users\...

macOS:
/Users/...

Linux:
/home/...
```

Avoid hardcoding path assumptions in common business logic.

A platform abstraction can expose:

```text
ApplicationDirectories
```

with values appropriate for the desktop platform.

---

# 17. Application Data Directory

An application may need directories for:

```text
Cache
Configuration
Logs
Database
Downloads
Temporary files
```

Desktop code can determine the appropriate location.

Common code can depend on:

```kotlin
interface AppDirectories {
    val dataDirectory: Path
    val cacheDirectory: Path
}
```

The desktop implementation resolves those directories appropriately.

---

# 18. Desktop Environment Variables

Desktop applications may interact with:

```text
Environment variables
System properties
User home directory
Operating system information
Process information
```

These are platform/environment concerns.

They should remain outside business logic.

---

# 19. Desktop Process Integration

Some applications need to:

```text
Launch another process
Open a system application
Execute a command
Inspect process information
```

Such capabilities are desktop-specific.

A shared abstraction might be:

```kotlin
interface ProcessLauncher {
    suspend fun launch(command: String)
}
```

The desktop implementation can use the appropriate JVM or operating-system APIs.

---

# 20. Desktop URL Handling

Desktop applications may open URLs using the operating system.

Common:

```kotlin
interface UrlLauncher {
    fun open(url: String): Boolean
}
```

Desktop:

```text
desktopMain
    ↓
Desktop browse/open API
```

Mobile implementations can use their respective platform APIs.

---

# 21. Desktop Clipboard

Clipboard functionality is another platform boundary.

Common:

```kotlin
interface Clipboard {
    fun copy(text: String)
    fun paste(): String?
}
```

Desktop:

```text
desktopMain
    ↓
Desktop/JVM clipboard APIs
```

Android and iOS use their own implementations.

---

# 22. Keyboard Shortcuts

Keyboard input is particularly important for desktop applications.

Examples:

```text
Ctrl/Cmd + S
Ctrl/Cmd + F
Ctrl/Cmd + Z
Escape
Arrow keys
Function keys
```

The shared UI may define an action:

```text
SaveDocument
```

while desktop-specific input handling maps:

```text
Ctrl/Cmd + S
```

to that action.

---

# 23. Platform-Specific Shortcut Mapping

Different desktop platforms may have different conventions.

For example:

```text
Windows/Linux:
Ctrl + S

macOS:
Command + S
```

The business action is the same:

```text
Save
```

The input mapping belongs at the platform/UI boundary.

---

# 24. Mouse and Pointer Behavior

Desktop applications may require:

```text
Hover
Right-click
Mouse wheel
Drag and drop
Pointer capture
```

These are generally presentation or platform concerns.

They should not become part of domain logic.

For example:

```text
Drag product
    ↓
UI interaction
    ↓
MoveProduct command
    ↓
commonMain
```

The business layer receives the command rather than raw mouse events.

---

# 25. Desktop Drag and Drop

Drag-and-drop may interact with:

```text
Files
Folders
Text
Images
External applications
```

The desktop integration can translate the native event into a shared command.

Conceptually:

```text
Native drag/drop event
        ↓
desktopMain
        ↓
FileDropped(path)
        ↓
commonMain
```

---

# 26. System Tray

Desktop applications sometimes run in the system tray.

Examples:

```text
Background utilities
Messaging applications
Monitoring tools
Synchronization clients
```

System tray integration is platform-specific.

The shared application may expose:

```text
ApplicationState
```

while desktop code handles:

```text
Tray icon
Tray menu
Tray actions
```

---

# 27. Desktop Notifications

Desktop notifications differ from mobile notifications.

Common:

```kotlin
interface NotificationService {
    fun notify(
        title: String,
        message: String
    )
}
```

Desktop:

```text
desktopMain
    ↓
Desktop notification mechanism
```

Android and iOS can implement the same capability using their own notification systems.

---

# 28. Desktop Auto-Start

Some desktop applications need to start automatically with the operating system.

This may involve:

```text
Windows startup
macOS login items
Linux desktop configuration
```

This is clearly outside shared business logic.

If required, isolate it in platform-specific desktop code.

---

# 29. Desktop System Integration

Desktop applications can integrate deeply with the operating system.

Examples:

```text
System tray
File associations
URL schemes
Notifications
Clipboard
Drag and drop
Native menus
Window controls
Startup
```

These capabilities are valuable precisely because desktop platforms expose them.

KMP should not hide them.

It should isolate them.

---

# 30. Desktop Native Menus

Desktop applications often have:

```text
File
Edit
View
Window
Help
```

menus.

The menu action itself may map to shared commands:

```text
NewDocument
SaveDocument
Undo
Redo
```

The desktop-specific menu implementation can remain at the platform/UI boundary.

---

# 31. File Associations

An application may be registered to open:

```text
.pdf
.myapp
.json
.custom files
```

Registration is operating-system specific.

The shared application can process the document once it receives it.

The desktop layer handles:

```text
Operating system
    ↓
Application launch/open event
    ↓
Document
    ↓
commonMain
```

---

# 32. Desktop Deep Linking

Desktop applications can also support URLs or custom schemes.

For example:

```text
myapp://orders/123
```

Desktop code receives the URL.

Shared code parses the route:

```text
orders/123
```

and determines the destination.

This keeps URL registration separate from navigation logic.

---

# 33. Desktop Persistence

Desktop applications may use:

```text
SQLite
Files
Preferences
Embedded databases
```

When the selected database is multiplatform-compatible, persistence can often remain shared.

When the implementation is desktop-specific, isolate it in:

```text
desktopMain
```

---

# 34. Desktop Database Drivers

A KMP database abstraction may use platform-specific drivers.

Conceptually:

```text
commonMain
    ↓
Database abstraction
    ↓
desktopMain
    ↓
Desktop/JVM database driver
```

This is another example of separating:

```text
Database behavior
```

from:

```text
Platform driver
```

---

# 35. Desktop Networking

Networking is often shareable.

If a multiplatform HTTP client supports the desktop target:

```text
commonMain
```

can contain:

```text
API service
Repository
Serialization
Authentication
Error handling
```

The HTTP engine may use a desktop-specific implementation underneath.

That does not require moving the entire network layer to `desktopMain`.

---

# 36. Desktop Dependency Decisions

Before placing a library in `desktopMain`, ask:

```text
Does this library really require desktop APIs?
```

If yes:

```text
desktopMain
```

If it supports all required targets:

```text
commonMain
```

This prevents unnecessary source-set fragmentation.

---

# 37. Desktop Logging

Logging can be shared through an abstraction:

```kotlin
interface Logger {
    fun debug(message: String)
    fun error(message: String)
}
```

Desktop can provide:

```text
DesktopLogger
```

which may integrate with:

```text
JVM logging
Console logging
Application logging
```

The common layer remains independent of the logging backend.

---

# 38. Desktop Configuration

Applications often need configuration such as:

```text
API endpoint
Feature flags
Environment
User preferences
Local paths
```

The configuration contract can remain shared.

The source of configuration may be platform-specific.

For example:

```text
commonMain
    ↓
ConfigurationProvider
    ↓
desktopMain
    ↓
Desktop configuration files / environment
```

---

# 39. Desktop Security

Desktop security can involve:

```text
Credential storage
Operating-system key stores
File permissions
Certificate stores
Process permissions
```

These should be isolated from common business logic.

A shared abstraction can represent the capability:

```text
SecureStorage
```

while the desktop implementation chooses the appropriate mechanism.

---

# 40. Desktop Authentication

Authentication flow can remain shared:

```text
Login
Refresh token
Logout
Session state
```

The desktop-specific part may be:

```text
Secure token storage
Browser-based authentication
OS credential integration
```

Therefore:

```text
Authentication logic → commonMain
Desktop authentication integration → desktopMain
```

---

# 41. Browser-Based Login

Some desktop applications authenticate through a browser.

The flow may be:

```text
Desktop app
    ↓
Open browser
    ↓
User authenticates
    ↓
Redirect/callback
    ↓
Desktop app
    ↓
Shared authentication flow
```

Opening the browser and receiving the desktop callback are platform concerns.

Token processing can remain shared.

---

# 42. Desktop Environment Detection

Some applications behave differently on:

```text
Windows
macOS
Linux
```

If behavior truly differs, keep the distinction at the appropriate platform boundary.

For example:

```text
desktopMain
    ↓
OS detection
    ↓
Windows implementation
macOS implementation
Linux implementation
```

Do not scatter operating-system checks throughout business logic.

---

# 43. Avoid This Pattern

Avoid:

```kotlin
if (os == "Windows") {
    // ...
} else if (os == "macOS") {
    // ...
} else {
    // ...
}
```

throughout the application.

Instead isolate the variation:

```text
OperatingSystemService
```

or a platform-specific implementation hierarchy.

This keeps the rest of the application clean.

---

# 44. Desktop Source Sets Can Become More Specific

A large project may eventually need:

```text
desktopMain
    │
    ├── Windows-specific
    ├── macOS-specific
    └── Linux-specific
```

The exact source-set hierarchy depends on the targets and Gradle/KMP configuration.

The architectural principle remains:

> **Put code at the narrowest source-set boundary that can legitimately support it.**

---

# 45. Narrowest Valid Boundary

Consider:

```text
commonMain
    ↓
desktopMain
    ↓
Windows-specific
```

If code works everywhere:

```text
commonMain
```

If it works across desktop targets:

```text
desktopMain
```

If it only works on Windows:

```text
Windows-specific
```

This prevents unnecessary duplication.

---

# 46. Desktop and Dependency Injection

Dependency injection can connect desktop implementations to shared services.

For example:

```text
commonMain
    ↓
SecureStorage
Logger
Clipboard
FileStorage
    ▲
    │
desktopMain
    ↓
DesktopSecureStorage
DesktopLogger
DesktopClipboard
DesktopFileStorage
```

The shared feature code does not need to know the implementation.

---

# 47. Desktop Composition Root

The desktop application can create:

```text
DesktopPlatformServices
```

and inject them into the shared application.

Conceptually:

```text
fun main() {

    val services = DesktopPlatformServices(
        storage = DesktopSecureStorage(),
        logger = DesktopLogger(),
        clipboard = DesktopClipboard()
    )

    startApplication(services)
}
```

The exact implementation varies by architecture.

The important idea is the dependency direction.

---

# 48. Keep Shared Code Testable

If common code directly calls:

```text
File
Process
System
Desktop APIs
```

testing becomes harder.

Instead:

```text
commonMain
    ↓
Interface
    ↓
Fake implementation
```

This allows common tests to run without a desktop environment.

---

# 49. Desktop Integration Tests

Desktop-specific integrations should be tested separately.

For example:

```text
FileStorage
Clipboard
ProcessLauncher
NotificationService
```

can have desktop-specific tests.

Common tests should focus on:

```text
Business behavior
Contracts
State transitions
Use cases
```

---

# 50. Desktop Tests and Environment

Desktop tests may depend on:

```text
Operating system
File system
Display environment
Permissions
Native services
```

Therefore, not every desktop test is equivalent to a pure unit test.

Keep environment-sensitive tests clearly separated.

---

# 51. CommonTest vs Desktop Test

A useful split is:

```text
commonTest
    ↓
Shared business behavior

desktopTest
    ↓
Desktop-specific behavior
```

For example:

```text
commonTest:
    OrderUseCaseTest

desktopTest:
    DesktopFileStorageTest
```

This keeps test responsibilities clear.

---

# 52. Desktop UI Testing

Compose Multiplatform desktop applications may require UI-specific testing strategies.

Keep business logic tests independent of UI.

A strong test pyramid is:

```text
             UI tests
                ▲
                │
        Desktop integration
                ▲
                │
       Shared unit tests
                ▲
                │
        Domain / use cases
```

The majority of business tests should remain platform-independent.

---

# 53. Desktop Performance

Desktop applications may have different performance characteristics from mobile applications.

For example:

```text
More memory
Larger displays
More powerful CPUs
Long-running sessions
Large files
Large datasets
```

Shared business logic can remain common, but desktop-specific optimizations may belong in:

```text
desktopMain
```

when they genuinely depend on the platform.

---

# 54. Large File Handling

Desktop applications may work with large files.

Instead of loading everything into memory:

```text
Large file
    ↓
Streaming
```

may be more appropriate.

If the implementation relies on desktop/JVM APIs, that implementation belongs at the desktop boundary.

The shared abstraction can remain platform-neutral.

---

# 55. Desktop Concurrency

Desktop applications can run:

```text
Background processing
File operations
Network operations
Long-running tasks
```

Shared coroutine-based business logic can often remain in:

```text
commonMain
```

Platform-specific thread or executor integration should remain at the platform boundary.

---

# 56. Desktop Long-Running Tasks

For example:

```text
Export 500,000 records
```

The business operation can remain shared:

```text
ExportDataUseCase
```

Desktop can provide:

```text
Desktop file destination
Progress UI
Native file chooser
```

This separates the work from the environment.

---

# 57. Desktop and JVM APIs

Many desktop KMP applications use JVM-based desktop targets.

That means desktop code may interact with:

```text
java.io
java.nio
java.net
java.awt
Swing
JVM system properties
```

when appropriate for the configured target.

These APIs should not leak into `commonMain`.

---

# 58. JVM Does Not Mean Common

A very important distinction:

```text
JVM API
≠
Multiplatform API
```

For example:

```text
java.io.File
```

is not available in:

```text
commonMain
```

just because Android also runs on a JVM-derived environment.

If a dependency is JVM-specific:

```text
desktopMain
```

is a possible location for desktop JVM code, depending on the project's target structure.

---

# 59. Avoid JVM Leakage

Do not create a shared abstraction like:

```kotlin
fun loadFile(file: java.io.File)
```

if the API is intended to be used by:

```text
Android
iOS
Desktop
```

Instead use a platform-neutral model.

For example:

```kotlin
data class FileReference(
    val path: String
)
```

or a suitable multiplatform representation.

---

# 60. Desktop and Native APIs

Not all desktop platforms are identical.

A project may use:

```text
JVM desktop
```

or other native desktop targets depending on its architecture.

Therefore, avoid assuming that every API available in one desktop environment is universally available to every desktop target.

Always design the source-set hierarchy according to the actual targets.

---

# 61. Desktop Resource Handling

Desktop applications may need:

```text
Images
Fonts
Configuration files
Templates
Icons
Localization files
```

Resource handling depends on the UI toolkit and project configuration.

Shared resources can be placed in shared mechanisms where supported.

Desktop-specific resources should remain with the desktop application or desktop source set.

---

# 62. Desktop Localization

The same semantic model can be shared:

```text
MessageKey.NetworkError
```

The desktop UI can resolve the appropriate localized resource.

The business layer should not depend on desktop resource identifiers.

---

# 63. Desktop Accessibility

Desktop applications may require:

```text
Keyboard navigation
Screen readers
Focus management
High contrast
Font scaling
Accessible labels
```

Much of this is UI-toolkit specific.

Keep business semantics shared and platform/UI accessibility implementation near the presentation layer.

---

# 64. Desktop UI State

UI state can remain common:

```kotlin
data class DashboardUiState(
    val loading: Boolean,
    val items: List<Item>,
    val error: String?
)
```

Desktop UI consumes it.

Android UI can consume it.

iOS UI can consume it.

The state itself does not need to become desktop-specific.

---

# 65. Desktop Navigation

Navigation logic can often remain common.

For example:

```text
Home
Settings
Details
Reports
```

The desktop UI can render those destinations.

Desktop-specific navigation may include:

```text
Multiple windows
External links
Native dialogs
Menus
```

Those belong at the desktop boundary.

---

# 66. Desktop Dialogs

Native dialogs can include:

```text
Open file
Save file
Print
System permissions
Native confirmation dialogs
```

The business layer should express intent:

```text
SelectFile
ConfirmDelete
```

The desktop layer chooses the appropriate UI.

---

# 67. Printing

Printing is another desktop-oriented capability.

Common:

```text
PrintDocument
```

Desktop:

```text
Native print APIs
```

This is a good candidate for a platform abstraction.

---

# 68. Desktop PDF Generation

If PDF generation uses a multiplatform library:

```text
commonMain
```

may be appropriate.

If it depends on a JVM-only library:

```text
desktopMain
```

may be required.

Again, the dependency determines the boundary.

---

# 69. Desktop Export

Suppose the application exports:

```text
CSV
JSON
PDF
Excel-compatible files
```

The transformation logic may be common.

The destination selection may be desktop-specific.

For example:

```text
commonMain
    ↓
GenerateCsv()
    ↓
desktopMain
    ↓
Save file dialog
```

---

# 70. Desktop Application Updates

Auto-update mechanisms vary across:

```text
Windows
macOS
Linux
```

They are platform/application deployment concerns.

Do not place update installation logic inside shared domain code.

Shared code can expose:

```text
UpdateAvailable
```

while desktop-specific code handles:

```text
Download
Install
Restart
```

---

# 71. Desktop Crash Reporting

Crash-reporting SDKs may be:

```text
Multiplatform
JVM-specific
Desktop-specific
OS-specific
```

Choose the appropriate source set based on actual target support.

The application-level reporting contract can remain common.

---

# 72. Desktop Analytics

Analytics events can remain common:

```text
UserLoggedIn
OrderCreated
ReportExported
```

Desktop can provide an analytics implementation.

This lets the same product analytics model work across platforms.

---

# 73. Desktop Feature Flags

Feature flag evaluation can remain common:

```text
isFeatureEnabled("new-dashboard")
```

The desktop application can provide:

```text
Configuration
Environment
Remote values
```

when required.

The evaluation logic should remain shared whenever possible.

---

# 74. Desktop Offline Mode

Offline behavior can often remain common.

For example:

```text
Offline
    ↓
Read local data
    ↓
Queue changes
    ↓
Sync later
```

The actual scheduling or file-system integration may require:

```text
desktopMain
```

but the synchronization business rules can remain in:

```text
commonMain
```

---

# 75. Desktop Sync

A good separation is:

```text
commonMain
    ↓
SyncEngine
```

Desktop:

```text
desktopMain
    ↓
Scheduler / lifecycle integration
```

The same sync engine can potentially be reused elsewhere.

---

# 76. Desktop System Services

A mature desktop platform layer might provide:

```text
Logger
Clipboard
FileStorage
FilePicker
NotificationService
UrlLauncher
ProcessLauncher
DeviceInfo
ApplicationDirectories
```

These are platform capabilities.

They should not be scattered throughout the shared codebase.

---

# 77. Avoid One Giant `DesktopService`

It may be tempting to create:

```kotlin
interface DesktopService {
    fun openFile()
    fun copy()
    fun notify()
    fun launchProcess()
    fun getDeviceInfo()
    ...
}
```

This can become difficult to maintain.

Prefer focused abstractions:

```text
FilePicker
Clipboard
NotificationService
ProcessLauncher
DeviceInfoProvider
```

This makes dependencies explicit.

---

# 78. Dependency Direction

The desired dependency direction is:

```text
desktopMain
      │
      ▼
commonMain abstractions
```

not:

```text
commonMain
      │
      ▼
desktopMain implementation
```

The shared layer defines what it needs.

The desktop layer fulfills those requirements.

---

# 79. Dependency Inversion

For example:

```kotlin
class ExportReportUseCase(
    private val fileStorage: FileStorage
)
```

The use case knows:

```text
FileStorage
```

It does not know:

```text
java.io.File
```

or:

```text
Windows API
```

or:

```text
macOS API
```

That is dependency inversion applied to KMP.

---

# 80. Desktop-Specific UI Should Stay at the Edge

The deeper the application layer goes, the less platform-specific it should become.

A useful mental model is:

```text
                    Platform specificity
                           ↑
                    Desktop UI / APIs
                           │
                    desktopMain
                           │
                    Shared adapters
                           │
                    commonMain
                           │
                    Domain logic
                           ↓
                    Platform independent
```

The domain should be the most portable part of the system.

---

# 81. A Practical Example

Imagine a desktop warehouse application.

Shared:

```text
Product
Inventory
ScanProductUseCase
UpdateInventoryUseCase
WarehouseRepository
SyncInventoryUseCase
```

Desktop-specific:

```text
USB scanner integration
Desktop file export
Native notifications
Keyboard shortcuts
Window management
```

The architecture can be:

```text
commonMain
    ↓
Warehouse business logic
    ↓
Platform contracts
    ↓
desktopMain
    ↓
Desktop integrations
```

This is a realistic KMP use case.

---

# 82. Desktop Hardware Integration

Desktop applications may interact with:

```text
Barcode scanners
USB devices
Serial ports
Printers
External displays
Specialized peripherals
```

These integrations are usually platform-specific.

The business layer should expose the capability it needs.

For example:

```kotlin
interface BarcodeScanner {
    fun scans(): Flow<String>
}
```

The desktop implementation handles the actual device.

---

# 83. Shared Scanner Events

The desktop scanner can produce:

```text
BarcodeScanned("123456")
```

The shared application can then process:

```text
Find product
Validate product
Update state
Sync inventory
```

This is a strong example of using desktop-specific hardware without moving business logic into `desktopMain`.

---

# 84. Desktop Printing Integration

A warehouse application might need:

```text
PrintLabel
PrintInvoice
PrintShippingDocument
```

The document generation can be shared.

The printer integration can remain:

```text
desktopMain
```

This keeps the platform-specific part small.

---

# 85. Desktop File Export

The application can generate:

```text
CSV
JSON
PDF
```

in shared code if supported.

Then:

```text
desktopMain
```

can handle:

```text
Choose destination
Write file
Open containing folder
```

The responsibility remains cleanly separated.

---

# 86. Desktop Source Sets and Architecture Boundaries

The deeper lesson is that source sets are architectural boundaries.

```text
commonMain
    ↓
Shared contract
    ↓
desktopMain
    ↓
Native desktop implementation
```

The folder structure communicates the architecture to every developer working on the project.

---

# 87. Common Mistake: Put Everything in `desktopMain`

A developer may think:

> "This is a desktop application, so everything can go into `desktopMain`."

That defeats the purpose of KMP.

If the following code does not depend on desktop:

```text
Business logic
Validation
Networking
Models
Use cases
Repository contracts
State
```

keep it shared.

---

# 88. Common Mistake: Force Everything into `commonMain`

The opposite mistake is equally dangerous.

Developers may try to make every API common even when it is clearly desktop-specific.

Forcing:

```text
Native file dialogs
System tray
Desktop process APIs
Native window management
```

into `commonMain` creates unnecessary abstractions or platform leaks.

The right solution is a clean boundary.

---

# 89. Common Mistake: Confusing Desktop With JVM

A desktop target may be JVM-based, but:

```text
desktopMain
```

is an architectural source-set concept.

Do not automatically expose every JVM API to the entire desktop application.

Ask whether the API is:

```text
Required by desktop?
```

or merely:

```text
Convenient because the target happens to use JVM.
```

---

# 90. Common Mistake: OS Checks Everywhere

Avoid:

```text
if Windows
if macOS
if Linux
```

throughout the application.

Centralize OS-specific behavior.

For example:

```text
DesktopFileSystem
    ├── WindowsFileSystem
    ├── MacFileSystem
    └── LinuxFileSystem
```

when the behavior truly differs.

---

# 91. Common Mistake: Platform Objects in Domain Models

Avoid:

```kotlin
data class Document(
    val file: java.io.File
)
```

if the model belongs to shared code.

Prefer:

```kotlin
data class Document(
    val id: String,
    val name: String
)
```

and keep platform file handles outside the domain.

---

# 92. Common Mistake: Platform APIs in Use Cases

Avoid:

```kotlin
class ExportReportUseCase {
    fun execute(file: java.io.File) {
        // ...
    }
}
```

in common code.

Prefer:

```kotlin
class ExportReportUseCase(
    private val fileStorage: FileStorage
)
```

The desktop implementation handles the native file system.

---

# 93. Common Mistake: Duplicate Desktop Business Logic

Do not create:

```text
Android order logic
iOS order logic
Desktop order logic
```

when the business rules are identical.

Instead:

```text
commonMain
    ↓
OrderUseCase
```

with only the platform-specific capability implementations separated.

---

# 94. When `desktopMain` Should Be Large

A desktop application with deep OS integration may legitimately have significant code in:

```text
desktopMain
```

Examples:

```text
Professional design software
IDE
Warehouse desktop client
Media application
Developer tool
File manager
POS application
```

The objective is not to minimize `desktopMain` at all costs.

The objective is to keep its responsibility clear.

---

# 95. When `desktopMain` Should Be Small

If the application is mostly:

```text
Shared Compose UI
Shared business logic
Shared networking
Shared persistence
```

then `desktopMain` may only contain:

```text
Entry point
Window configuration
A few platform services
```

That is perfectly healthy.

---

# 96. Source Set Granularity

Think of source sets as layers of specificity:

```text
                 commonMain
                     │
               More specific
                     ▼
                desktopMain
                     │
               Even more specific
                     ▼
            OS/target-specific
```

Move code downward only when necessary.

Keep code as high in the hierarchy as its dependencies allow.

---

# 97. A Useful Architecture Test

Ask three questions:

### Question 1

Does the code depend on business concepts?

```text
→ commonMain
```

### Question 2

Does it depend on desktop capabilities?

```text
→ desktopMain
```

### Question 3

Does it depend on one specific desktop OS?

```text
→ OS-specific boundary
```

This simple test solves many source-set decisions.

---

# 98. `desktopMain` as an Adapter Layer

The best mental model is:

```text
commonMain
    │
    │ "I need to open a file."
    ▼
FilePicker
    ▲
    │
desktopMain
    │
    │ "I know how to show the desktop dialog."
    ▼
Native desktop API
```

The shared layer defines the need.

The desktop layer knows the implementation.

---

# 99. Desktop and Multiplatform Thinking

KMP architecture becomes easier when you stop asking:

> "How can I make desktop code common?"

and start asking:

> "Which parts of the desktop application are actually platform-independent?"

That shift leads to better boundaries.

---

# 100. Final Desktop Mental Model

The complete picture is:

```text
                         commonMain
                             │
                 Shared business capability
                             │
                    Platform abstraction
                             │
             ┌───────────────┼───────────────┐
             ▼               ▼               ▼
        androidMain        iosMain       desktopMain
             │               │               │
        Android APIs     Apple APIs      Desktop APIs
                                             │
                              ┌──────────────┼──────────────┐
                              ▼              ▼              ▼
                           Windows         macOS          Linux
```

The exact source-set hierarchy depends on the targets configured by the project, but the architectural principle remains the same.

> **`desktopMain` is where desktop-specific capability meets shared application architecture.**

---

# Chapter Takeaways

> [!TIP]
> **Use `desktopMain` for desktop-specific production code. Keep shared business behavior, models, use cases, and genuinely multiplatform infrastructure in `commonMain`.**

Remember:

1. `desktopMain` provides a desktop-specific source-set boundary.
2. It is useful for desktop-targeted KMP applications.
3. It can contain code shared by multiple desktop targets.
4. It is not automatically equivalent to Windows code.
5. Desktop-specific APIs belong at the desktop boundary.
6. Common business logic should remain in `commonMain`.
7. Shared models should generally remain in `commonMain`.
8. Shared use cases should generally remain in `commonMain`.
9. Shared validation should generally remain in `commonMain`.
10. Shared networking can remain in `commonMain` when the selected stack supports the targets.
11. Shared persistence can remain in `commonMain` when the selected stack supports the targets.
12. Desktop file-system integration is a common `desktopMain` responsibility.
13. Native file dialogs belong at the desktop boundary.
14. Desktop window management belongs at the desktop boundary.
15. Desktop keyboard shortcuts belong at the UI/platform boundary.
16. Desktop mouse and pointer integrations belong at the UI/platform boundary.
17. Desktop clipboard integration can belong in `desktopMain`.
18. Desktop notifications can be platform-specific.
19. System tray integration is platform-specific.
20. Native desktop menus are platform-specific.
21. File associations are platform-specific.
22. Desktop deep-link registration is platform-specific.
23. Process launching is platform-specific.
24. Desktop environment access is platform-specific.
25. Desktop application directories are platform-specific.
26. Desktop hardware integrations are platform-specific.
27. Barcode scanner integrations can be isolated in `desktopMain`.
28. Printer integrations can be isolated in `desktopMain`.
29. Native file export destinations can be handled in `desktopMain`.
30. Desktop background scheduling should remain at the platform boundary.
31. Shared synchronization logic can remain in `commonMain`.
32. JVM APIs should not leak into `commonMain`.
33. A JVM-based desktop target does not make JVM APIs multiplatform.
34. Verify library target support before choosing a source set.
35. A multiplatform library can often remain in `commonMain`.
36. A desktop-only library may belong in `desktopMain`.
37. An OS-specific dependency may require an even narrower source-set boundary.
38. Avoid scattering Windows/macOS/Linux checks throughout business logic.
39. Centralize OS-specific behavior.
40. Use abstractions for desktop capabilities when shared code needs them.
41. Prefer focused interfaces over one giant desktop service.
42. Keep dependency direction from platform implementations toward common abstractions.
43. Use dependency injection to provide desktop implementations.
44. Keep native desktop objects away from domain models.
45. Keep platform-specific errors away from domain APIs where possible.
46. Translate platform API results into shared models when appropriate.
47. Keep desktop construction close to the desktop composition root.
48. Common tests should focus on shared behavior.
49. Desktop tests should validate desktop-specific integrations.
50. Environment-sensitive tests should be treated differently from pure unit tests.
51. Desktop UI can be shared when using a multiplatform UI toolkit.
52. Desktop-specific UI integrations can remain in `desktopMain`.
53. Window management does not need to become business logic.
54. Keyboard shortcuts should map to shared actions rather than domain-level key events.
55. Native dialogs should express user intent to the shared application.
56. Desktop file paths should not leak into shared domain models.
57. Desktop process handles should not leak into shared domain models.
58. Desktop system services should be isolated behind capability interfaces.
59. Shared code should depend on capabilities rather than implementations.
60. Do not move an entire feature to `desktopMain` just because one dependency is desktop-specific.
61. Move only the platform-dependent capability when possible.
62. Do not force clearly desktop-specific APIs into `commonMain`.
63. Do not duplicate business logic between desktop and mobile.
64. Source sets communicate architectural boundaries.
65. The narrowest valid source set is usually the best home for platform-specific code.
66. Keep code in `commonMain` when its dependencies allow it.
67. Move code to `desktopMain` when it genuinely requires desktop capabilities.
68. Move code to an OS-specific boundary when behavior is limited to one operating system.
69. `desktopMain` should not be judged by its size alone.
70. A desktop-heavy application may legitimately have substantial desktop-specific code.
71. A simple shared Compose application may need very little `desktopMain`.
72. The purpose of `desktopMain` is clear separation, not maximum code reduction.
73. Desktop is a platform with valuable native capabilities.
74. KMP should isolate those capabilities rather than eliminate them.
75. **The best desktop KMP architecture shares what is stable and isolates what is native.**

---

# Final Thought

A strong KMP desktop architecture does not attempt to turn desktop development into mobile development.

It recognizes the strengths of the desktop platform while sharing the logic that genuinely benefits from reuse.

The model is:

```text
                         commonMain
                             │
                    Shared application
                             │
                     Platform contract
                             │
                             ▼
                        desktopMain
                             │
                  Desktop implementation
                             │
             ┌───────────────┼───────────────┐
             ▼               ▼               ▼
          Windows          macOS           Linux
```

The shared layer defines **what the application needs**.

The desktop layer defines **how the desktop platform provides it**.

That separation keeps the codebase portable without pretending that every platform behaves the same way.

> **Share the logic that should be shared. Keep desktop power where desktop belongs.**
