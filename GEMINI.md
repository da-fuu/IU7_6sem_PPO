# TimeTracker - Comprehensive Project Guide

TimeTracker is a modular Kotlin application for personal time management. It features a modern Android UI and a versatile CLI that runs on both standard JVMs and Android devices.

## 1. Architectural Principles

The project follows **Clean Architecture** to ensure testability, scalability, and separation of concerns.

### 1.1 Module Structure
- **`:domain`**: The core of the application.
  - **Models**: Business entities (`TimeRecord`, `Category`, `User`).
  - **Repositories**: Interface definitions for data access.
  - **Use Cases**: Encapsulated business logic (e.g., `StartTimerUseCase`, `AddManualRecordUseCase`).
  - **Constraint**: Must remain a pure Kotlin module with no platform-specific dependencies.
- **`:data`**: Data persistence and external service integration.
  - **Database**: Room implementation with `BundledSQLiteDriver`.
  - **Entities**: Room-specific data models.
  - **Mappers**: Conversion logic between Entities and Domain models.
  - **Repositories**: Concrete implementations of domain interfaces.
- **`:app`**: The primary Android application.
  - **UI**: Jetpack Compose-based screens.
  - **ViewModels**: Manage UI state and interact with Use Cases.
  - **Session**: `SessionManager` provides reactive authentication state.
- **`:tech_ui`**: Technical UI (CLI).
  - **Controller**: Orchestrates CLI commands.
  - **Runtime**: Supports JVM and Android (via `app_process`).

### 1.2 Data Flow
1.  **UI/CLI** interacts with a **ViewModel/Controller**.
2.  **ViewModel/Controller** triggers a **Use Case**.
3.  **Use Case** enforces business rules and calls **Repository**.
4.  **Repository** performs CRUD operations via **Room DAOs**.
5.  State updates propagate back via `StateFlow` or `Result` objects.

## 2. Development Conventions

### 2.1 Coding Style
- **NO COMMENTS**: Code must be self-explanatory. Comments of any kind are strictly prohibited.
- **Naming**: Prefer verbosity over ambiguity (e.g., `getTimeRecordsByUserId` vs `getRecords`).
- **Reactive State**: Always use `StateFlow` or `SharedFlow` for asynchronous state updates.
- **Immutability**: Prefer `val` and immutable data classes.

### 2.2 Error Handling & Feedback
- **Business Logic**: Return `Result<T>` from use cases to handle success and failure explicitly.
- **UI Feedback**: display errors to the user using Android `Toast`. Never ignore exceptions or silently fail in the UI layer.
- **Logging**: Use SLF4J for technical auditing. Business rule violations (like short sessions) should be logged as `WARN`.

### 2.3 Dependency Injection
- **Framework**: Koin.
- **Scope**: Definitions are centralized in `TimeTrackerApp` (Android) and `TechUiApp` (CLI).

## 3. Build & Execution Commands

### 3.1 Common Tasks
- **Run All Tests**: `./gradlew test`
- **Lint Check**: `./gradlew lint`
- **Clean Project**: `./gradlew clean`

### 3.2 Android App (`:app`)
- **Build Debug APK**: `./gradlew :app:assembleDebug`
- **Install on Device**: `./gradlew :app:installDebug`

### 3.3 CLI - JVM Mode (`:tech_ui`)
- **Build Executable JAR**: `./gradlew :tech_ui:jar` (Output: `dist/tech_ui.jar`)
- **Run JAR**: `java -jar dist/tech_ui.jar [args]`

### 3.4 CLI - Android Mode (`:tech_ui`)
- **Build Dexed JAR**: `./gradlew :tech_ui:dex` (Output: `dist/timetracker-dex.jar` and `dist/lib/`)
- **Deploy to Device**:
  ```bash
  adb shell mkdir -p /data/local/tmp/timetracker/
  adb push dist/timetracker-dex.jar /data/local/tmp/timetracker/
  adb push dist/lib /data/local/tmp/timetracker/
  ```
- **Execute on Device**:
  ```bash
  export CLASSPATH="/data/local/tmp/timetracker/timetracker-dex.jar"
  app_process -Dslf4j.provider=org.slf4j.simple.SimpleServiceProvider /data/local/tmp/timetracker com.ppo.tech_ui.TechUiAppKt [args]
  ```

## 4. Key CLI Commands Reference
- `register <login> <name> <pass> [USER/ADMIN]`
- `login <login> <pass>`
- `categories`: List all categories.
- `create-category <name> <color_hex>`
- `start <category_uuid>`: Start timer.
- `stop`: Stop active timer.
- `add-manual <category_uuid> <start_iso> <end_iso> <desc>`: Supports flexible ISO-8601 formats.
- `records`: List time history.
- `update-desc <record_uuid> <new_description>`: Update record description and save old version to history.
- `desc-history <record_uuid>`: View all previous descriptions for a record.
- `rollback-desc <record_uuid> <version_uuid>`: Revert description to a specific historical version.

## 5. Persistence & Configuration
- **Root Directory**: `Downloads/TimeTracker/` (created automatically).
- **Database File**: `db/database.db` (within root).
- **Configuration**: `config.properties`.
  - `min.duration.minutes`: Minimum duration for a valid record.
  - `db.path`: Custom path for the database.
  - `log.level`: SLF4J log level (INFO, DEBUG, etc.).
- **Session**: `session.txt` stores the current User UUID.
