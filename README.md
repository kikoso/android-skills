# Android & Gravity Skills

This directory contains specialized skills for Android & Gravity to enforce best practices during software development.

## Available Skills

1. [Security Skill](./android-security-skill/SKILL.md) - Comprehensive rules for managing secrets, network security, data encryption, logging, and component safety.
2. [Git Operations Skill](./git-skill/SKILL.md) - Enforces branch strategies, conventional commits, ignore rules, and demands explicit confirmation before pushing.
3. [Background Location Skill](./background-location-skill/SKILL.md) - Rules for permissions, foreground services, Doze and OEM battery restrictions, data quality, privacy, and how to verify that tracking survives in the background.
4. [Kotlin Test Skill](./kotlin-test-skill/SKILL.md) - Writes deterministic unit, coroutine/Flow and Compose tests, with an edge-case checklist (time, money, location, connectivity, lifecycle) and regression-test-first bug fixing.
5. [Kotlin Multiplatform Skill](./kmp-skill/SKILL.md) - What to share in `commonMain`, project structure, interfaces vs expect/actual, multiplatform libraries, concurrency, and testing on every target.
6. [KMP iOS Interop Skill](./kmp-ios-interop-skill/SKILL.md) - Designing a Swift-friendly API: `@Throws` on suspend functions, bridging Flows, sealed classes and enums, framework export, and verifying from the iOS side.

*To activate these skills in an agent prompt, reference the corresponding markdown file instructions.*
