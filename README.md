# Kixo Android SDK — Release Repo

This repo serves built AAR + POM artefacts for the Kixo Android SDK.
The development source lives in a private Bitbucket repo; published
release artefacts land here in a Maven-2 layout.

## Add to your project

`settings.gradle.kts`:

```kotlin
dependencyResolutionManagement {
  repositories {
    google()
    mavenCentral()
    maven { url = uri("https://raw.githubusercontent.com/kixoio/kixo-android-sdk/main/repo") }
  }
}
```

`app/build.gradle.kts`:

```kotlin
implementation("io.kixo:kixo-android-sdk:0.1.0-alpha7")
```

Available versions are listed in `repo/io/kixo/kixo-android-sdk/`.

## Quick start

```kotlin
class MyApp : Application() {
  override fun onCreate() {
    super.onCreate()
    Kixo.configure(this, "kx_proj_…", "kx_key_…")
  }
}

Kixo.track("checkout_started", mapOf("plan" to "pro"))
Kixo.identify("user_123", mapOf("email" to "alex@example.com"))
Kixo.markGoal("subscribed", value = 9.99, currency = "USD")
```

## Status

Pre-release. APIs may change. Contact paul@kixo.io to enroll your
project for early access.
