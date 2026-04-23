# AGENTS.md

## Project Summary

This repository is a Kotlin-based JetBrains IntelliJ Platform plugin.

- Plugin id: `dev.sweep.assistant`
- Plugin display name patched by Gradle: `Self-Hosted Enterprise Updater`
- Version: `1.29.3`
- Main source: `src/main/kotlin/dev/sweep/assistant`
- Plugin descriptor: `src/main/resources/META-INF/plugin.xml`
- Protobuf schema: `src/main/proto/dev/sweep/assistant/data/sweepMessages.proto`
- Gradle build: `build.gradle.kts`
- Target platform currently used for builds: `intellijIdeaUltimate("2026.1")`

The plugin provides a Sweep assistant tool window, chat UI, code modification flows, autocomplete/edit helpers, MCP support, terminal/tool-call UI, and persistence via protobuf-backed message conversion.

## Current Build Baseline

This workspace has been adjusted to build against IntelliJ IDEA Ultimate 2026.1.

Important build settings:

- Kotlin Gradle plugin: `2.3.0`
- Kotlin serialization plugin: `2.3.0`
- IntelliJ Platform Gradle plugin: `2.2.0`
- Java toolchain: `JavaLanguageVersion.of(22)`
- Java bytecode target: `21`
- Kotlin JVM target: `JvmTarget.JVM_21`
- IntelliJ platform dependency: `intellijIdeaUltimate("2026.1")`
- Plugin compatibility patching: `sinceBuild = 261`, `untilBuild = 261.*`

Common build command:

```powershell
.\gradlew.bat buildPlugin --no-daemon --no-configuration-cache --console=plain
```

Known successful output:

```text
build/distributions/sweepai-1.29.3.zip
```

## Recent Compatibility Fixes

The original failure was:

```text
FileAnalysisException while analysing build/generated/source/proto/main/kotlin/.../CompletedToolCallKt.kt
IllegalArgumentException: source must not be null
```

That was caused by compiling protobuf Kotlin DSL generated files. The project uses Java-style generated protobuf builders such as `SweepMessages.ProtoMessage.newBuilder()`, not the Kotlin DSL functions like `completedToolCall { ... }`.

Current protobuf decision:

- Use `com.google.protobuf:protobuf-java:3.23.4`
- Generate and compile Java protobuf output only
- Keep `build/generated/source/proto/main/java` in Kotlin source dirs
- Do not generate or compile `build/generated/source/proto/main/kotlin`

This avoids the Kotlin compiler crash from `CompletedToolCallKt.kt`.

The 2026.1 migration also required:

- Upgrading Kotlin from `2.1.0` to `2.3.0`, because IDEA 2026.1 artifacts contain Kotlin metadata `2.3.0`.
- Replacing deprecated `kotlinOptions.jvmTarget = "..."` with the Kotlin 2.3 `compilerOptions { jvmTarget.set(...) }` DSL.
- Changing Java/Kotlin bytecode target from 17 to 21.
- Changing plugin compatibility from `241`/`253.*` to `261`/`261.*`.

Source compatibility fixes made for 2026.1 APIs:

- `src/main/kotlin/dev/sweep/assistant/actions/SweepProblemsAction.kt`
  - `ProblemNode.text`, `ProblemNode.line`, and `ProblemNode.severity` now need explicit getter calls: `getText()`, `getLine()`, `getSeverity()`.
- `src/main/kotlin/dev/sweep/assistant/components/MessagesComponent.kt`
  - `WriteIntentReadAction.compute` now takes one type argument, not two.
- `src/main/kotlin/dev/sweep/assistant/components/PromptBarPanel.kt`
  - `editor.virtualFile` is nullable under the newer API. Use `FileDocumentManager.getInstance().getFile(editor.document) ?: return` before calling `FileEditorManager.getEditors(...)`.

## Local Machine Notes

This Windows machine currently has:

- Default `java`: Corretto 17
- Installed JDKs under `C:\Users\95497\.jdks` include Corretto 17 and Corretto 22

The Gradle build uses the configured Java toolchain, so default `java -version` being 17 is acceptable as long as Gradle can find JDK 22.

`gradle.properties` was reduced for this machine:

```properties
kotlin.daemon.jvmargs=-Xmx2g
org.gradle.jvmargs=-Xmx2g -Dfile.encoding=UTF-8
org.gradle.workers.max=2
org.gradle.parallel=false
```

Reason: previous builds hit Windows pagefile/native memory pressure and Gradle daemon startup failures.

If Gradle fails with:

```text
java.net.BindException: Address already in use: bind
```

check for leftover Gradle processes:

```powershell
jps -l
```

Stop only Gradle-related processes (`GradleDaemon`, `GradleWrapperMain`) before retrying. Avoid killing unrelated Java processes such as the IDE or other applications.

## Known Warnings

`buildPlugin` currently succeeds but reports:

```text
The Kotlin Coroutines library must not be added explicitly to the project nor as a transitive dependency as it is already provided with the IntelliJ Platform
```

This is not blocking local packaging, but should be cleaned up before marketplace-style publication or strict verification.

The IntelliJ Platform 2026.1 dependency also prints many layout warnings about nonexistent classPath elements. These appeared during successful builds and were not blocking.

## Useful Commands

Compile Kotlin only:

```powershell
.\gradlew.bat compileKotlin --no-daemon --no-configuration-cache --console=plain
```

Build plugin zip:

```powershell
.\gradlew.bat buildPlugin --no-daemon --no-configuration-cache --console=plain
```

Run sandbox IDE:

```powershell
.\gradlew.bat runIde --no-daemon --no-configuration-cache --console=plain
```

Check IntelliJ platform dependency:

```powershell
.\gradlew.bat dependencies --configuration intellijPlatformDependency --no-daemon --no-configuration-cache --console=plain
```

## Windows Install Note

The `bin/install` script is Bash/macOS-oriented and installs into `~/Library/Application Support/JetBrains`. On Windows, install the generated zip manually:

1. Open IntelliJ IDEA.
2. Go to `Settings` -> `Plugins`.
3. Use `Install Plugin from Disk...`.
4. Select `build/distributions/sweepai-1.29.3.zip`.
5. Restart the IDE.

## Repository Caution

There were pre-existing unrelated local changes in `.idea/` and other files during this session. Do not revert files you did not intentionally edit. Before making further changes, inspect:

```powershell
git status --short
```

