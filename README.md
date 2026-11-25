<div align="center">
  <img src="docs/icon.png" alt="App Icon" width="100" />
  <h1>rikkahub-clean</h1>
  <p><b>A personal build repository for clean, tracker-free builds of RikkaHub.</b></p>
</div>

> [!IMPORTANT]
> **This is not the official RikkaHub app and not a supported distribution.**
>
> This repository exists for one purpose: to produce an APK of
> [RikkaHub](https://github.com/rikkahub/rikkahub) with **Google Firebase (Analytics and
> Crashlytics) removed** and the first-party update ping disabled. There are no added features,
> no upstream pull requests and no support. Issues with this build belong here — never in the
> upstream project.
>
> **APK:** [Releases → `notrackers`](https://github.com/callmebroda/rikkahub-clean/releases/tag/notrackers)
>
> **Builds are manual.** Nothing here builds on a schedule or on a push. Upstream code is synced
> automatically, but an APK is only produced when the build workflow is started by hand.

## What this repository is

- A **personal build repo**, not a maintained fork or a competing distribution. It is upstream
  RikkaHub plus two small patches, built by GitHub Actions on demand.
- The source tree is otherwise **byte-for-byte upstream**. The tracker removal is not committed
  to the code; it is applied from [`patches/`](patches/) at build time, so upstream updates can
  never silently undo it.
- No feature work happens here. If you want new features, they come from upstream.
- **Building is manual.** The only automatic job is the upstream sync, which merges code and
  checks the patches — it never runs a Gradle build.
- `README_ZH_CN.md` and `README_ZH_TW.md` are untouched upstream files and still describe the
  official app. This English README is the accurate description of this build.

## Differences from upstream RikkaHub

Tracks upstream `master` automatically: the daily sync merges it, so the app behaviour here is
whatever upstream currently ships (RikkaHub 3.3.1, versionCode 198 at the time of writing). The
exact upstream base of any build is visible in
[the commit history](https://github.com/callmebroda/rikkahub-clean/commits/master).

| | Upstream `rikkahub/rikkahub` | This build |
|---|---|---|
| Analytics | `firebase-analytics` sends 5 event names: `ai_send_message`, `ai_edit_message`, `ai_regenerate_at_message`, `ai_tool_approval`, `ai_tool_answer` | **Removed.** No analytics SDK, no events |
| Crash reporting | `firebase-crashlytics`, enabled by default with no in-app opt-out | **Removed** |
| Build requirement | `app/google-services.json` must exist (upstream CI writes it from a secret) | **Not needed** — a plain clone builds |
| Version check | `GET https://updates.rikka-ai.com/` on first launch, sending `RikkaHub <version> #<versionCode>` in the User-Agent | **Disabled** — the app never contacts that host |
| Signing | Upstream release key | Fresh RSA-4096 key generated inside CI on every run |

### Files the patches touch

| Patch | Files | Change |
|---|---|---|
| `patches/0001-remove-firebase.patch` | `gradle/libs.versions.toml`, `build.gradle.kts`, `app/build.gradle.kts`, `di/AppModule.kt`, `di/ViewModelModule.kt`, `ui/pages/chat/ChatVM.kt` | Drops the `google-services` and `firebase-crashlytics` Gradle plugins, the three Firebase dependencies, the two Koin singletons, the `analytics = get()` injection, the `FirebaseAnalytics` constructor parameter and the 5 `logEvent` calls. 38 lines deleted |
| `patches/0002-disable-update-ping.patch` | `utils/UpdateChecker.kt` | `checkUpdate()` returns immediately instead of calling `updates.rikka-ai.com` |

### What is deliberately left alone

- Everything else is upstream code, unmodified: features, UI, providers, tools, permissions.
- The package id is still `me.rerere.rikkahub`, so this build **conflicts with an installed
  official RikkaHub** — uninstall one before installing the other.
- The app still talks to the AI providers you configure, and to `sponsors.rikka-ai.com` for the
  sponsor list on the About page. Those are the only first-party endpoints left.
- No other tracker SDK was ever present upstream: no AdMob, AppsFlyer, Adjust, Umeng, Bugly,
  Sentry, PostHog, Amplitude or Mixpanel.

## How the changes survive upstream updates

Three mechanisms, in order of importance:

1. **The patches are not in the source tree.** They live in `patches/` and are applied by the
   build workflow at run time. Merging upstream can therefore never conflict with them or drop
   them, because the tracked source never contains those edits.
2. **Upstream is synced automatically.** [`sync-upstream.yml`](.github/workflows/sync-upstream.yml)
   runs daily at 03:00 UTC (08:30 IST) and on manual dispatch: it merges
   `rikkahub/rikkahub@master` into this repo's `master`, pushes, and then **dry-runs the patches
   to see whether they still apply**. It never builds an APK.
   `README.md` is resolved in favour of this fork (it is this fork's own document); any other
   conflict stops the sync with an error rather than guessing.
3. **A broken patch is caught before it reaches a build.** The sync dry-runs every patch and
   goes red, naming the file, if one no longer fits the synced code — so you find out from the
   daily run, not from a failed build you paid attention-minutes for. The build repeats the same
   check, then greps the whole tree for `com.google.firebase` / `com.google.gms` /
   `google-services`; if anything survives, the run fails and no APK is published.

The intended rhythm is: the sync merges upstream and reports whether the patches still fit → if a
patch went stale you update it → then you start the build yourself, once.

### If a patch stops applying

Upstream edited the same lines the patch touches. You will see it as a red **Sync upstream
RikkaHub** run naming the patch (and, if you build anyway, as a failed build). To fix it:

```sh
git clone https://github.com/callmebroda/rikkahub-clean.git
cd rikkahub-clean
git remote add upstream https://github.com/rikkahub/rikkahub.git
git fetch upstream master && git merge upstream/master

git apply --3way patches/0001-remove-firebase.patch   # or 0002-…
# resolve the conflicts by hand, keeping the intent: no Firebase, no update ping
git diff > patches/0001-remove-firebase.patch         # overwrite that one patch only
git add -A && git commit -m "patches: update for upstream <commit>" && git push
```

The two patches touch disjoint files, so regenerating one at a time with a clean tree keeps the
other intact. Then re-run the build workflow.

## How a build is produced

[`build-notrackers.yml`](.github/workflows/build-notrackers.yml) — **manual only, never scheduled,
never triggered by the upstream sync or by a normal commit.** A build starts when someone taps
*Run workflow*, or when a tag named `build-*` is pushed (`git tag build-2026-11-25 && git push
origin build-2026-11-25`), which is just another deliberate way to ask for one. An APK exists
only because a build was asked for:

1. Checkout, JDK 21, Node 22, pnpm 11, `pnpm install` in `web-ui/` (the `:web` module builds a
   React SPA during `preBuild`).
2. Apply `patches/*.patch`, then verify no Firebase reference remains.
3. Generate a throwaway RSA-4096 signing key and write `local.properties`.
4. `./gradlew assembleGithubRelease` — arm64-v8a, x86_64 and a universal APK.
5. Upload the APKs as a workflow artifact and publish them to the `notrackers` release.

No repository secrets are needed. One build uses roughly 20-30 runner-minutes. (For the record,
GitHub does not meter Actions minutes on public repositories — the monthly allowance applies to
private ones — but keeping builds manual is still the right default here: an APK only appears
when it is actually wanted.)

### Installing and updating

Every run generates a new signing key, so a newer build cannot be installed over an older one.
Uninstall the previous copy first. If you want in-place updates, add your own keystore as a
base64 repository secret and change the "Generate signing key" step to decode it instead of
creating a new key.

## Licence and modification notice (AGPL-3.0)

Upstream RikkaHub is licensed under the GNU Affero General Public License v3.0. This is a
modified version of that work, produced and published under the same licence, with the complete
corresponding source in this repository — the modifications are the two patches in `patches/`,
applied on 2026-11-24. No upstream code was relicensed and no authorship of unchanged code is
claimed.

## Upstream project

- Repository: [rikkahub/rikkahub](https://github.com/rikkahub/rikkahub)
- Upstream README: [README](https://github.com/rikkahub/rikkahub/blob/master/README.md)
- Official downloads: [rikka-ai.com/download](https://rikka-ai.com/download),
  [Google Play](https://play.google.com/store/apps/details?id=me.rerere.rikkahub) — those builds
  still contain Firebase Analytics and Crashlytics.
