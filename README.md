# Kixo Android SDK

Product analytics, push, session replay, and AI-driven funnel analysis
for Android apps. Every public API lives behind a single `object Kixo`
in package `io.kixo.sdk`. Kotlin 2.0+, minSdk 24, JVM 17.

This repo serves the built AAR + POM artefacts in a Maven-2 layout —
add the repo to your Gradle config and pull the SDK like any other
dependency. Source code, issue tracker, and contribution flow live
separately; this is the **distribution** repo only.

- **Latest version**: `0.1.1`
- **Package**: `io.kixo.sdk` · **Entry point**: `io.kixo.sdk.Kixo`
- **Maven coords**: `io.kixo:kixo-android-sdk:0.1.1`
- **Available versions**: see [`repo/io/kixo/kixo-android-sdk/`](repo/io/kixo/kixo-android-sdk/)
- **Docs**: <https://docs.kixo.io/docs/sdk/android>

---

## Install

In your `settings.gradle.kts`, add the Kixo Maven repository alongside
`google()` and `mavenCentral()`:

```kotlin
dependencyResolutionManagement {
    repositories {
        google()
        mavenCentral()
        maven {
            url = uri("https://raw.githubusercontent.com/kixoio/kixo-android-sdk/main/repo")
        }
    }
}
```

In your `app/build.gradle.kts`:

```kotlin
dependencies {
    implementation("io.kixo:kixo-android-sdk:0.1.1")
}
```

Internet permission is bundled in the SDK manifest — no extra
`uses-permission` declarations needed for basic ingest.

---

## Configure

The SDK is a singleton object. Import it by its fully-qualified name
and call `configure` once from your `Application.onCreate`:

```kotlin
package com.example.myapp

import android.app.Application
import io.kixo.sdk.Kixo

class MyApp : Application() {
    override fun onCreate() {
        super.onCreate()
        Kixo.configure(
            context   = this,
            projectId = "kx_proj_YOUR_PROJECT_ID",
            apiKey    = "kx_key_YOUR_API_KEY",
        )
    }
}
```

Register the `Application` subclass in `AndroidManifest.xml`:

```xml
<application
    android:name=".MyApp"
    ...>
</application>
```

Get your `projectId` + `apiKey` from <https://dashboard.kixo.io> →
Settings → SDK keys.

---

## Send custom events

Every public API on `Kixo` is safe to call from any thread. Pre-config
calls buffer (capped at 50) and replay onto the queue when `configure`
lands — call sites in global singletons that initialise before your
`Application.onCreate` finishes are fine.

```kotlin
import io.kixo.sdk.Kixo
import io.kixo.sdk.PushProvider
import io.kixo.sdk.StandardEvent

// Custom event with properties
Kixo.track("checkout_started", mapOf(
    "plan" to "pro",
    "discount_pct" to 15,
))

// Tie events to a known user
Kixo.identify(
    userId = "user_123",
    traits = mapOf("email" to "alex@example.com", "plan" to "pro"),
)

// Set a property attached to every subsequent event in the session
Kixo.setSuperProperty("ab_variant", "treatment_b")

// Mark a conversion goal
Kixo.markGoal(
    name     = "subscribed",
    value    = 9.99,
    currency = "USD",
    properties = mapOf("plan" to "pro"),
)

// Typed standard event (drives the revenue dashboard)
Kixo.trackPurchase(
    amount     = 19.99,
    currency   = "USD",
    productId  = "sku_pro_annual",
    quantity   = 1,
)

// Add a Sentry-style breadcrumb (rides along with the next crash)
Kixo.addBreadcrumb(
    category = "ui",
    message  = "user tapped retry",
    data     = mapOf("attempt" to 2),
)

// Push token registration (FCM, HMS, or APNs)
Kixo.setPushToken("fcm_token_here", PushProvider.FCM)

// Force-flush before backgrounding (asynchronous)
Kixo.flush()

// Health snapshot for debug screens
val health = Kixo.diagnostics()
android.util.Log.d("Kixo", "lifecycle=${health.lifecycleState} " +
    "buffered=${health.queue.bufferedEventCount} " +
    "retry=${health.queue.retryTier}")
```

For the complete public API (every method, every parameter), see the
[Android docs](https://docs.kixo.io/docs/sdk/android).

---

## For AI coding agents (Claude / Cursor / OpenAI Codex)

This section is a copy-pasteable system-prompt extension. Drop it
into `CLAUDE.md`, `.cursor/rules/`, or your agent's system message so
the agent integrates Kixo correctly without round-tripping for clarification.

```
# Kixo Android SDK integration rules

- Package: `io.kixo.sdk`. Entry point: singleton `object Kixo`.
  Every code sample MUST start with `import io.kixo.sdk.Kixo`
  (plus any extra named type referenced — `KixoConfiguration`,
  `PushProvider`, `StandardEvent`, `SubscriptionInterval`,
  `KixoDiagnostics`).
- Install via Gradle. Add this Maven repo to `settings.gradle.kts`:
  `maven { url = uri("https://raw.githubusercontent.com/kixoio/kixo-android-sdk/main/repo") }`
  Then `implementation("io.kixo:kixo-android-sdk:0.1.1")` in
  `app/build.gradle.kts`. minSdk 24, targetSdk 35, JVM 17.
- Initialise from `Application.onCreate` with
  `Kixo.configure(this, "kx_proj_…", "kx_key_…")`. The SDK
  auto-tracks lifecycle, screens (Activity + Fragment),
  taps/gestures, crashes, sessions, push, and (when enabled)
  session replay. No manual screen-view calls needed in pure
  View-binding apps; Compose Navigation requires calling
  `Kixo.screen("name")` from a `LaunchedEffect(route)` inside
  the `NavHost`.
- Send custom events with `Kixo.track(name, props)`. Mark
  conversion goals with `Kixo.markGoal(name, value?, currency?)`.
  Identify a user with `Kixo.identify(userId, traits)`. Use the
  typed `StandardEvent` sealed class for revenue (Purchase,
  SubscriptionStart, etc.) — those drive built-in dashboards.
- Push: register the FCM/HMS/APNs token with
  `Kixo.setPushToken(token, PushProvider.FCM)` from the
  `FirebaseMessagingService.onNewToken` callback. Manually log
  delivery with `Kixo.logPushReceived/Opened/Dismissed/Action`
  from your push handler — there is no automatic AppDelegate
  swizzle equivalent on Android.
- Consent: `Kixo.optOut()` / `Kixo.optIn()` /
  `Kixo.grantConsent()` / `Kixo.revokeConsent()`. State persists
  across process restarts. Until `Kixo.grantConsent()` is called,
  the SDK ships only the bare minimum (lifecycle, version registry).
- Debug: `Kixo.diagnostics()` returns a `KixoDiagnostics` snapshot
  with `lifecycleState`, queue buffered count, retry tier, and
  paused state. Use this to answer "why aren't events landing?"
  before reaching for logcat. `Kixo.flush()` forces the next batch
  immediately.
- The SDK NEVER crashes the host. Every failure path swallows + logs
  to logcat with tag `Kixo`. If you see no SDK logs at all on cold
  launch, the most likely cause is `Kixo.configure` not being called.
- Ingest is HTTPS-only and fully managed by Kixo — no host or
  endpoint configuration needed.
```

---

## Support

Open an issue at <https://github.com/kixoio/kixo-android-sdk/issues>
or email <support@kixo.io>.
