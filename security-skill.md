# Android Gravity Skill: Security

## Description
This skill enforces strict, comprehensive security practices during Android development, ensuring sensitive information is protected both at rest and in transit.

## Instructions
When activated, you MUST adhere to the following rules:

### 1. Secrets and Credentials
- **Never Commit Secrets:** Do not commit API keys, tokens, passwords, or any `*.jks`/`*.keystore` files to version control.
- **Use local.properties:** Store development secrets in `local.properties` and read them via `build.gradle.kts` (using `BuildConfig`). Verify `local.properties` is in `.gitignore`.
- **Review Before Staging:** Always review file diffs for hardcoded secrets, IP addresses, or sensitive internal URLs before staging them for a commit.

### 2. Data Storage
- **Encrypt Sensitive Data:** Never store PII (Personally Identifiable Information), tokens, or passwords in plain text in `SharedPreferences` or SQLite.
- **Use AndroidX Security:** Use `EncryptedSharedPreferences` for key-value pairs and `SQLCipher` for Room/SQLite databases when handling sensitive data.
- **Internal Storage:** Prefer internal storage over external storage for app-specific data to leverage Android's sandbox isolation.

### 3. Network Security
- **HTTPS Only:** Ensure all network communication uses HTTPS. Cleartext traffic should be disabled by default (which is standard from Android 9+).
- **Network Security Configuration:** Utilize `network_security_config.xml` to enforce certificate pinning and restrict cleartext traffic.
- **Validate Certificates:** Never implement custom `TrustManager`s that bypass SSL validation.

### 4. Logging and Telemetry
- **Scrub Logs:** Do not log sensitive user data (PII), credentials, or full API responses in `Logcat` or crash reporting tools (like Crashlytics).
- **Use Timber/Custom Loggers:** Ensure logging is disabled or stripped in release builds (e.g., using ProGuard rules or conditional `BuildConfig.DEBUG` checks).

### 5. App Components & IPC
- **Exported Components:** Explicitly declare `android:exported="false"` for Activities, Services, and BroadcastReceivers unless they absolutely need to be accessible to other apps.
- **Intent Validation:** Always validate incoming `Intent` data and extras to prevent injection or unexpected behavior.

### 6. Code Obfuscation
- **Enable R8/ProGuard:** Ensure `isMinifyEnabled = true` and `isShrinkResources = true` in release builds to obfuscate code and remove unused resources, making reverse engineering more difficult.

## Triggers
Activate this skill whenever:
- Modifying configuration files (`build.gradle.kts`, `AndroidManifest.xml`, `network_security_config.xml`).
- Implementing network clients, local databases, or `SharedPreferences`.
- Adding logging or crash reporting.
- The user requests a security review or mentions handling sensitive user data.