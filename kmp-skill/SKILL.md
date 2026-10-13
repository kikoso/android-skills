---
name: kmp-skill
description: Enforces a clean, maintainable Kotlin Multiplatform (KMP) architecture for code shared between Android and iOS. Use when creating or modifying KMP modules, deciding what goes in commonMain vs platform source sets, adding multiplatform dependencies, using expect/actual, or migrating Android code to shared code.
---

# Android Gravity Skill: Kotlin Multiplatform

## Description
This skill enforces sound Kotlin Multiplatform practices: sharing business logic and data layers between Android and iOS, keeping platform code thin, and choosing libraries that actually work on every target.

## Instructions
When activated, you MUST adhere to the following rules:

### 1. What to Share
- **Share Logic, Not Platform Details:** Put domain models, business rules, use cases, repositories, networking, persistence and state holders in `commonMain`. Keep UI, permissions, sensors, background execution, push notifications and OS integrations in platform code unless a well-supported multiplatform library covers them.
- **UI Sharing Is a Decision, Not a Default:** Sharing UI with Compose Multiplatform is a product decision. Do not migrate a SwiftUI screen to Compose (or the other way round) unless the user asks for it.
- **No Android Imports in Common Code:** `commonMain` must never import `android.*`, `androidx.*` Android-only artifacts, `java.*` or platform frameworks. If code needs them, it belongs in `androidMain`/`iosMain` behind an abstraction.

### 2. Project Structure
- **Respect the Existing Layout:** Detect the current structure (e.g. a `shared` module, or `:core:*` / `:feature:*` KMP modules) and follow it. Do not reorganize modules without asking.
- **Default Hierarchy Template:** Use the default source set hierarchy (`commonMain`, `androidMain`, `iosMain`, `appleMain`, `nativeMain`...) instead of declaring intermediate source sets by hand.
- **Targets:** Declare only the targets the product ships (typically `androidTarget()` plus `iosArm64()`, `iosSimulatorArm64()`, and `iosX64()` only if Intel simulators are still needed).
- **Android Library Plugin:** For KMP library modules on recent AGP versions, use the Android-KMP library plugin (`com.android.kotlin.multiplatform.library`) with the `androidLibrary { }` block, instead of combining `com.android.library` with the Kotlin Multiplatform plugin. Check the project's AGP version before changing plugins.
- **Version Catalog:** Add every dependency and plugin through `gradle/libs.versions.toml`.

### 3. expect/actual vs Interfaces
- **Prefer Interfaces + DI:** For platform services (location, secure storage, analytics, clock, connectivity), define an `interface` in `commonMain` and provide platform implementations through dependency injection. This keeps the code testable with fakes in `commonTest`.
- **Use expect/actual Sparingly:** Reserve `expect`/`actual` for small, stateless primitives (e.g. a platform name, a UUID generator, a database driver factory). Prefer `expect fun` or `expect class` with a minimal API; never put business logic in `actual` implementations.

### 4. Libraries
Only add libraries that support every declared target. Typical choices:
- **Coroutines & Flow:** `kotlinx-coroutines-core`. Use `Dispatchers.Default` for CPU work. `Dispatchers.IO` exists on JVM and Native but not on JS/Wasm, so only use it in common code if those targets aren't declared, and prefer injecting dispatchers.
- **Serialization:** `kotlinx-serialization-json` (not Gson or Moshi).
- **Networking:** Ktor client in common code, with platform engines (`OkHttp` on Android, `Darwin` on iOS).
- **Persistence:** Room (KMP-enabled since 2.7, with the bundled SQLite driver) or SQLDelight. Follow whichever the project already uses.
- **Date & Time:** `kotlinx-datetime` (not `java.time`) in common code. Inject a `Clock` for testability.
- **Dependency Injection:** Hilt is Android-only. Use Koin, kotlin-inject, or manual constructor injection for shared code, keeping Hilt only in the Android app module if it already uses it.
- **ViewModels:** `androidx.lifecycle:lifecycle-viewmodel` supports KMP and can be used in `commonMain` for shared state holders.

### 5. Concurrency and State
- **Structured Concurrency:** Shared state holders own a `CoroutineScope` that the platform cancels (e.g. `viewModelScope` or an explicit `close()`/`clear()`), so no work outlives the screen.
- **Expose StateFlow:** Shared presentation logic exposes immutable state via `StateFlow<UiState>` and receives user actions as plain functions, mirroring unidirectional data flow on both platforms.
- **Memory Model:** The new Kotlin/Native memory manager is the default; do not use `freeze()` or legacy `@SharedImmutable`/`@ThreadLocal` workarounds.

### 6. Testing
- **Test in commonTest:** Write business-logic tests in `commonTest` with `kotlin.test` and `kotlinx-coroutines-test` (`runTest`), using fakes for platform interfaces. Use Turbine for Flows if the project has it.
- **Run Tests on Every Target:** Run both JVM/Android and iOS tests, e.g. `./gradlew :shared:allTests` or `:shared:testDebugUnitTest` plus `:shared:iosSimulatorArm64Test`. Never report shared code as working if only Android tests were run.

### 7. Build Hygiene
- **Keep iOS Builds Green:** After changing shared code, compile the iOS framework (e.g. `./gradlew :shared:linkDebugFrameworkIosSimulatorArm64`) or run the iOS tests; Android compiling alone does not prove the shared code is valid for iOS.
- **Explicit API (for SDK-like modules):** For modules consumed by other teams or exported to iOS, consider `explicitApi()` to keep the public surface intentional.

## Triggers
Activate this skill whenever:
- Creating or modifying a Kotlin Multiplatform module, `kotlin { }` targets or source sets.
- Writing code in `commonMain`, `androidMain`, `iosMain` or other KMP source sets.
- Adding dependencies to a KMP module or introducing `expect`/`actual` declarations.
- Moving Android code into shared code, or the user asks to share logic with iOS.
