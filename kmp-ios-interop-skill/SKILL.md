---
name: kmp-ios-interop-skill
description: Enforces a Swift-friendly public API for Kotlin Multiplatform code consumed from iOS. Use when exposing shared Kotlin classes, suspend functions, Flows, sealed classes, enums or exceptions to Swift, configuring the iOS framework export, or reviewing how iOS code calls into the shared module.
---

# Android Gravity Skill: KMP iOS Interop

## Description
This skill ensures that shared Kotlin code is pleasant and safe to use from Swift. Kotlin/Native exports an Objective-C-compatible header, so some Kotlin features translate poorly; the public API must be designed with the Swift caller in mind.

## Instructions
When activated, you MUST adhere to the following rules:

### 1. Design the Public Surface for Swift
- **Small, Intentional API:** Only the types iOS actually needs should be public in the exported module. Mark everything else `internal`. A large public surface bloats the framework header and slows down iOS builds.
- **Facade per Feature:** Expose one entry point per feature (e.g. `TripsPresenter`, `CheckInService`) rather than leaking repositories, DAOs or DTOs to Swift.
- **Avoid Kotlin-Only Constructs in Public APIs:** Default arguments are not visible from Swift (all parameters must be passed), and generics on interfaces and functions are partially erased. Provide explicit overloads or simple types where Swift callers need them.
- **Naming:** Use `@ObjCName` (or `@ShouldRefineInSwift` / `@HiddenFromObjC` for wrappers) when the generated Swift name is awkward or clashes.

### 2. Suspend Functions and Flows
- **Suspend Functions:** Swift sees them as async functions (Swift 5.5+) or completion handlers. They can only throw the exception types declared with `@Throws`; any other exception crashes the app. Annotate every public `suspend` function that can fail with `@Throws(CancellationException::class, <DomainException>::class)`, or return a result type instead of throwing.
- **Flows Need a Bridge:** A `Flow` is exposed as a plain interface without generics in Swift. Use the bridging approach the project already uses: SKIE (Swift `AsyncSequence`), KMP-NativeCoroutines, or a hand-written wrapper with `subscribe(onEach:onComplete:)` that returns a cancellable handle. Do not add a second bridging library.
- **Cancellation:** Every subscription exposed to Swift must be cancellable, and cancelling it must cancel the underlying coroutine. Document who owns the scope.
- **Main Thread:** State observed by SwiftUI must be delivered on the main thread (`Dispatchers.Main`), or the Swift side must explicitly hop to `@MainActor`.

### 3. Types and Errors
- **Sealed Classes:** Without SKIE, sealed classes become class hierarchies that Swift can't `switch` over exhaustively. Prefer SKIE's exhaustive enums, or provide simple `when`-free accessors (e.g. `isLoading`, `errorMessage`) on UI state for Swift.
- **Enums:** Kotlin enums become classes in Swift (not Swift enums) unless SKIE is used. Keep them simple, and avoid relying on `values()` ordering in Swift.
- **Collections and Primitives:** `List`/`Map` map to `NSArray`/`NSDictionary` (`Array`/`Dictionary` in Swift). Nullable primitives (`Int?`, `Boolean?`) become `KotlinInt?`/`KotlinBoolean?` boxes; avoid them in hot public APIs.
- **Time and Money:** Do not expose `kotlinx.datetime.Instant` or big-decimal types directly if Swift needs to compute with them; expose epoch milliseconds or formatted strings, or provide conversion helpers.
- **Errors:** Model expected failures as data (a sealed result or error state), not exceptions. Reserve exceptions for programming errors.

### 4. Framework Configuration
- **Static vs Dynamic:** Follow the project's existing `isStatic` choice; switching can break the Xcode integration.
- **Exporting Dependencies:** Only `export(...)` modules whose types must appear in the Swift API, and keep `transitiveExport` off.
- **Integration Method:** Respect the existing integration (Xcode build phase with `embedAndSignAppleFrameworkForXcode`, SPM via XCFramework, or CocoaPods). Don't switch methods without asking.
- **Bitcode/Arch:** Build for the architectures the app ships (`iosArm64`, `iosSimulatorArm64`); don't add targets nobody uses.

### 5. Verification
- **Compile the Framework:** After changing anything public, build the framework (e.g. `./gradlew :shared:linkDebugFrameworkIosSimulatorArm64`) and inspect the generated header (`build/bin/iosSimulatorArm64/debugFramework/<Name>.framework/Headers/<Name>.h`) for the Swift-visible names.
- **Call It From Swift:** When possible, build the iOS app or a Swift test that calls the changed API. Report clearly if only the Kotlin side was compiled.
- **Breaking Changes:** Treat renames and signature changes of public shared APIs as breaking changes for the iOS team; call them out in the PR description.

## Triggers
Activate this skill whenever:
- Adding or changing `public` declarations in a KMP module that is exported to iOS.
- Exposing `suspend` functions, `Flow`/`StateFlow`, sealed classes, enums or exceptions to Swift.
- Modifying the iOS framework configuration (`binaries.framework`, `export`, `isStatic`, XCFramework, CocoaPods or SPM setup).
- Reviewing Swift code that calls into the shared module, or the user reports crashes or awkward APIs on the iOS side.
