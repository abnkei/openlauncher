# Repository guidance

- Android application module: `app/`; application ID: `com.openlauncher.app`.
- UI uses Kotlin and Jetpack Compose. Follow existing patterns in `app/src/main/java/com/openlauncher/app/`.
- Build configuration: Android Gradle Plugin 9.1.1, Compose compiler plugin 2.2.10, Gradle wrapper 9.4.1, compile SDK 36.1, target SDK 36. Java source/target compatibility is 17.
- Existing source separates settings/data, models, services, viewmodel, UI screens/widgets, and utilities. Keep changes focused.
- Upstream has no automated test sources or CI configuration in the reviewed tree. Do not claim tests or builds passed without executing them.
- The wrapper script is `gradlew`; upstream does not include `gradlew.bat`. Windows builds require a suitable shell or a configured Gradle installation and Android SDK.
- Upstream's `Screenshots ` directory has a trailing space, which breaks normal Windows checkout. The current checkout uses cone-mode sparse checkout for `app`, `gradle`, and `docs`, retaining root files. Do not commit deletions of excluded assets.
- Keep downloaded APKs, ADB binaries, device logs, credentials, and machine-local configuration out of commits.
- Read `docs/K4811-HANDOFF-2026-10-19.md` before resuming the device investigation. Hardware compatibility and installation remain unverified.
