---
name: kotlin-test-skill
description: Writes and reviews Kotlin/Android unit and UI tests that target edge cases, not just the happy path. Use when adding tests, fixing a bug (regression test first), testing coroutines/Flows or Compose UI, or when the user asks for test coverage of domain logic such as time, money, location or state.
---

# Android Gravity Skill: Kotlin Tests

## Description
This skill writes deterministic, readable tests for Kotlin and Android code, with a strong focus on the edge cases where production bugs actually happen: time, money, location, connectivity, lifecycle and concurrency.

## Instructions
When activated, you MUST adhere to the following rules:

### 1. Follow the Project First
- **Match the Existing Stack:** Before writing tests, inspect `build.gradle.kts`, `libs.versions.toml` and existing tests. Use the framework already in use (JUnit4 or JUnit5, Kotest, MockK, Turbine, Robolectric, Truth/AssertK). Do not introduce a new testing library without asking.
- **Match Conventions:** Mirror the package of the class under test, the existing naming style, and the existing fakes/test fixtures. Reuse test helpers instead of duplicating them.
- **Right Source Set:** Pure JVM logic goes in `src/test`. Code that needs a real device or the Android framework goes in `src/androidTest` (or `src/test` with Robolectric if the project already uses it).

### 2. Test Design
- **One Behavior per Test:** Each test verifies a single behavior. Name it as a sentence: ``fun `check-out after midnight bills the departure day`()``.
- **Arrange / Act / Assert:** Keep the three phases visible. No logic (loops, conditionals) inside assertions.
- **Fakes over Mocks:** Prefer simple fakes for repositories, data sources and clocks. Use mocks (MockK) only for interactions that cannot be observed otherwise, and never mock the class under test or value types.
- **Test Behavior, Not Implementation:** Assert on outputs and observable state, not on private methods or call order that is not part of the contract.

### 3. Determinism
- **Inject Time:** Never call `System.currentTimeMillis()`, `Instant.now()` or `LocalDate.now()` in code under test. Inject a `java.time.Clock` (or `kotlinx.datetime.Clock`) and use a fixed or controllable clock in tests.
- **Inject Dispatchers:** Never hardcode `Dispatchers.IO`/`Default` in classes under test; inject a `CoroutineDispatcher`. Replace `Dispatchers.Main` with a `MainDispatcherRule` (JUnit4) or an equivalent extension (JUnit5).
- **No Real Sleeps or Network:** No `Thread.sleep`, real delays, real HTTP calls or shared global state. Use virtual time and fakes (or MockWebServer for HTTP client tests).
- **Fixed Locale and Time Zone:** When formatting is involved, set the `Locale` and `ZoneId` explicitly in the test.

### 4. Coroutines and Flows
- **Use `runTest`:** Use `kotlinx-coroutines-test` with `runTest` and `StandardTestDispatcher`. Control virtual time with `advanceTimeBy` / `advanceUntilIdle` / `runCurrent`.
- **Test Flows with Turbine** (if available): use `flow.test { awaitItem(); ...; cancelAndIgnoreRemainingEvents() }`. Assert the full sequence of emitted states, including loading and error states.
- **Cancellation and Errors:** Cover cancellation (the job is cancelled mid-work and resources are released) and exceptions (they surface as the expected error state, not as a crash or a silent swallow).

### 5. Compose UI
- **Use Semantics:** With `createComposeRule()`, find nodes by text, content description or test tag (`onNodeWithText`, `onNodeWithContentDescription`, `onNodeWithTag`), not by position.
- **Test State, Not Pixels:** Drive the composable with explicit state and assert what is displayed and which callbacks fire. Screenshot tests only if the project already uses them.
- **Accessibility Checks:** Verify that interactive elements have a content description or text, so TalkBack can announce them.

### 6. Edge-Case Checklist
For each unit under test, go through this list and write tests for every item that applies (state the ones you skip and why):
- **Time:** midnight crossing, DST transitions (spring forward / fall back), time zone change during a session, month/year boundaries, leap day, clock moving backwards.
- **Money:** use minor units or `BigDecimal` (never `Double`), rounding rules, zero and negative amounts, caps and discounts, currency mismatch.
- **Location:** no fix, poor accuracy, sudden jumps, stale fixes, mock locations, crossing a region or country border, GPS lost mid-session.
- **Connectivity:** offline, timeout, retry exhausted, partial or duplicate server response, request succeeds after the app gave up.
- **Lifecycle:** process death and restore (`SavedStateHandle`), configuration change, app started from a notification or deep link.
- **Concurrency:** double taps / duplicate events, concurrent updates, cancellation in the middle of an operation.
- **Inputs:** empty, null, maximum sizes, unexpected enum values or unknown fields from the backend.

### 7. Bug Fixes: Regression Test First
- When fixing a bug, first write a test that reproduces it and confirm it **fails** for the right reason. Then fix the code and confirm it passes.

### 8. Run and Report Honestly
- **Run the Tests:** After writing tests, run them (e.g. `./gradlew testDebugUnitTest --tests "<fully.qualified.TestClass>"`, or `connectedDebugAndroidTest` for instrumented tests) and report the actual result.
- **Never Weaken Tests to Pass:** Do not delete assertions, loosen expectations, add `@Ignore`, or change production behavior just to make a test green. If a test exposes a real bug, report it as a finding with the failing output.
- **Summarize Coverage:** End with a short list of the behaviors and edge cases now covered and the ones that still are not.

## Triggers
Activate this skill whenever:
- The user asks to add, fix or review tests, or to increase coverage.
- Fixing a bug (write the regression test first).
- Adding or changing domain logic involving time, money, location, state machines or retries.
- Writing ViewModels, repositories, use cases, Flows or Compose screens.
