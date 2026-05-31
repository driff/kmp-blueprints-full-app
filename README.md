# kmp-blueprint

A Kotlin Multiplatform + Compose Multiplatform starter targeting **Android, iOS, and Desktop (JVM)** from one shared codebase. Use it as the starting point for new KMP apps.

## What's included

- **`:shared`** — KMP module with Compose Multiplatform UI. Common, Android, iOS, and JVM (desktop) source sets are wired up.
- **`:androidApp`** — Android host (Compose `MainActivity`).
- **`:desktopApp`** — Compose Desktop `singleWindowApplication`.
- **`iosApp/`** — XcodeGen-managed iOS app hosting `ComposeUIViewController`.
- Gradle 9.4.1 wrapper, JDK 25 toolchain, AGP 9 with the built-in Kotlin plugin.
- Detekt + ktlint wired up.
- Clean Architecture package skeleton inside `:shared` (`core/`, `domain/`, `data/`, `presentation/`, `di/`).

## Prerequisites

| Tool | Version | Install |
|---|---|---|
| JDK | 25 LTS (Temurin) | `sdk install java 25.0.3-tem` (SDKMAN) — pinned via `kotlin.jvmToolchain(25)` |
| Gradle | 9.4.1 (via wrapper) | bundled — `./gradlew` |
| Android SDK | API 36 (Baklava) | `sdkmanager` or the `android` CLI |
| Xcode | 26.4.1+ | App Store. Required only for the iOS target. |
| XcodeGen | 2.45.4+ | `brew install xcodegen` — regenerates `iosApp/iosApp.xcodeproj` from `iosApp/project.yml` |

### One-time setup

```bash
echo "sdk.dir=$HOME/Library/Android/sdk" > local.properties
./gradlew --version
```

## Running the project

### Android

```bash
./gradlew :androidApp:assembleDebug
```

### iOS

```bash
(cd iosApp && xcodegen generate)   # only after editing project.yml
xcodebuild \
  -project iosApp/iosApp.xcodeproj \
  -scheme iosApp \
  -configuration Debug \
  -destination 'platform=iOS Simulator,name=iPhone 17,OS=latest' \
  -derivedDataPath iosApp/build/derivedData \
  CODE_SIGNING_ALLOWED=NO build
```

Or open `iosApp/iosApp.xcodeproj` in Xcode and press ⌘R — the project's Run Script phase invokes `:shared:embedAndSignAppleFrameworkForXcode`.

### Desktop

```bash
./gradlew :desktopApp:run
./gradlew :desktopApp:packageDistributionForCurrentOS
```

### Tests & static analysis

```bash
./gradlew detekt
./gradlew ktlintCheck
./gradlew :shared:jvmTest
./gradlew :shared:iosSimulatorArm64Test
```

## Project layout

```
kmp-blueprint/
├── settings.gradle.kts                ← declares :shared, :androidApp, :desktopApp
├── build.gradle.kts                   ← root + Detekt/ktlint config
├── gradle/libs.versions.toml          ← single source of truth for versions
├── config/detekt/detekt.yml
├── shared/
│   └── src/commonMain/kotlin/com/example/kmpblueprint/
│       ├── core/        framework-agnostic primitives
│       ├── domain/      pure Kotlin: models, repositories, use cases
│       ├── data/        Ktor / SQLDelight / OIDC implementations
│       ├── presentation/ Compose UI + ViewModels
│       └── di/          Koin modules
├── androidApp/                        Compose root host
├── iosApp/                            XcodeGen project + Swift entry point
│   ├── project.yml                    XcodeGen source of truth (edit this)
│   └── iosApp.xcodeproj/              committed; regenerate with `xcodegen generate`
└── desktopApp/                        Compose Desktop singleWindowApplication
```

## Using this blueprint for a new project

Search-and-replace the following placeholders. They are intentionally generic — keep the casing consistent in each location.

| Placeholder | Where it appears | Replace with |
|---|---|---|
| `kmp-blueprint` | `settings.gradle.kts` (`rootProject.name`), this README | your gradle project name (kebab-case) |
| `com.example.kmpblueprint` | namespaces, applicationId, bundle id, all `package` decls, source dirs `com/example/kmpblueprint/` | your package (e.g. `com.acme.app`) |
| `KMP Blueprint` | Android `android:label`, iOS `CFBundleDisplayName`, desktop window title, greeting string | your app's display name |
| `KmpBlueprint` | `Theme.KmpBlueprint` (Android style), Compose Desktop `packageName` | identifier form of the display name (no spaces) |

After renaming, run `./gradlew help` and `(cd iosApp && xcodegen generate)` to confirm everything still resolves.

## Technical notes

- **Single `:shared` module, Clean Architecture as packages.** A multi-module split is overkill for an MVP. One module + package-level boundaries gives most of the same enforcement.
- **Compose Multiplatform 1.10.x.** CMP 1.8.x calls AGP APIs removed in AGP 9.
- **AGP 9 + the built-in Kotlin plugin.** The legacy `org.jetbrains.kotlin.android` plugin is intentionally omitted from `:androidApp`; `:shared` uses `com.android.kotlin.multiplatform.library`.
- **iOS via XcodeGen.** `iosApp/project.yml` is the editable source. The generated `.xcodeproj` is committed so contributors can open Xcode without installing XcodeGen first.
- **`CADisableMinimumFrameDurationOnPhone = true`** in `iosApp/project.yml` is required: Compose Multiplatform 1.10 throws at launch on iPhones with high refresh-rate displays otherwise.
