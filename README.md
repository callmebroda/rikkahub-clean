> [!IMPORTANT]
> **This is a modified build of RikkaHub — not the official app.**
>
> This repository is a personal fork of [rikkahub/rikkahub](https://github.com/rikkahub/rikkahub)
> with **Google Firebase (Analytics and Crashlytics) removed** and the first-party update ping
> disabled. Download the APK only from
> [this repository's Releases](https://github.com/callmebroda/rikkahub-clean/releases/tag/notrackers).
>
> Issues with this build belong here, not in the upstream project.

## 🔧 Differences from upstream RikkaHub

Based on upstream commit `2435dbd` ("chore: bump version to 3.3.1", 2026-11-21) — RikkaHub
3.3.1, versionCode 198.

| | Upstream `rikkahub/rikkahub` | This fork (`rikkahub-clean`) |
|---|---|---|
| Analytics | `firebase-analytics` sends 5 event names: `ai_send_message`, `ai_edit_message`, `ai_regenerate_at_message`, `ai_tool_approval`, `ai_tool_answer` | **Removed.** No analytics SDK, no events |
| Crash reporting | `firebase-crashlytics`, enabled by default with no in-app opt-out | **Removed** |
| Build requirement | `app/google-services.json` must exist (upstream CI writes it from a secret) | **Not needed** — a plain clone builds |
| Version check | `GET https://updates.rikka-ai.com/` on first launch, sending `RikkaHub <version> #<versionCode>` in the User-Agent | **Disabled** — the app never contacts that host |
| Signing | Upstream release key | Fresh RSA-4096 key generated inside CI on every run |
| Build pipeline | Local Android Studio, or upstream CI with keystore + Google Services secrets | `.github/workflows/build-notrackers.yml` in this repo, no secrets required |

### Exactly what changed in the source

38 lines deleted across 6 files, plus one function body:

| File | Change |
|---|---|
| `gradle/libs.versions.toml` | Removed the `google-services`, `firebase-bom` and `firebase-crashlytics` versions, the three Firebase library aliases and the two Gradle plugin aliases |
| `build.gradle.kts` | Removed the `google-services` and `firebase-crashlytics` plugin declarations |
| `app/build.gradle.kts` | Removed both plugin aliases and the Firebase dependency block |
| `app/src/main/java/me/rerere/rikkahub/di/AppModule.kt` | Removed the `Firebase.crashlytics` and `Firebase.analytics` Koin singletons |
| `app/src/main/java/me/rerere/rikkahub/di/ViewModelModule.kt` | Removed the `analytics = get()` injection into `ChatVM` |
| `app/src/main/java/me/rerere/rikkahub/ui/pages/chat/ChatVM.kt` | Removed the `FirebaseAnalytics` constructor parameter and the 5 `logEvent` calls |
| `app/src/main/java/me/rerere/rikkahub/utils/UpdateChecker.kt` | `checkUpdate()` now returns immediately instead of calling `updates.rikka-ai.com` |

### What was deliberately left alone

- Everything else is upstream code, unmodified: features, UI, providers, tools, permissions.
- The package id is still `me.rerere.rikkahub`, so this build **conflicts with an installed
  official RikkaHub** — uninstall one before installing the other.
- The app still talks to the AI providers you configure, and to `sponsors.rikka-ai.com` for the
  sponsor list on the About page. Those are the only first-party endpoints left.
- No other tracker SDK was ever present upstream: no AdMob, AppsFlyer, Adjust, Umeng, Bugly,
  Sentry, PostHog, Amplitude or Mixpanel.

### Installing and updating

Every CI run generates a new signing key, so a newer build cannot be installed over an older one.
Uninstall the previous copy first. If you want in-place updates, add your own keystore as a
base64 repository secret and change the "Generate signing key" step to decode it instead of
creating a new key.

### Licence and modification notice (AGPL-3.0)

Upstream RikkaHub is licensed under the GNU Affero General Public License v3.0. This is a
modified version of that work, produced and published under the same licence, with the complete
corresponding source in this repository. The modifications are the Firebase removal and the
update-check change listed above, made on 2026-11-24. No upstream code was relicensed and no
authorship of unchanged code is claimed.

---

*Everything below is the upstream README, kept for reference.*

---

<div align="center">
  <img src="docs/icon.png" alt="App Icon" width="100" />
  <h1>RikkaHub</h1>

A native Android LLM chat client that supports switching between different providers for
conversations 🤖💬

Click to join our Discord server 👉 [【RikkaHub】](https://discord.gg/9weBqxe5c4)

[简体中文](README_ZH_CN.md) | [繁體中文](README_ZH_TW.md) | English
</div>

<div align="center">
  <img src="docs/img/poster.png" alt="RikkaHub" width="800" />
</div>

## 🚀 Download

🔗 [Download the tracker-free APK from this repository's Releases](https://github.com/callmebroda/rikkahub-clean/releases/tag/notrackers)

> The official app is published at [rikka-ai.com/download](https://rikka-ai.com/download) and
> [Google Play](https://play.google.com/store/apps/details?id=me.rerere.rikkahub). Those builds
> still include Firebase Analytics and Crashlytics.

> [!WARNING]
> There are many forked versions of RikkaHub. Issues with forks are unrelated to RikkaHub, so please use forks with caution to avoid privacy leaks or excessive permission requests.

## 💖 Sponsors

| Sponsor | Description |
|:---:|:---|
| <img src="docs/sponsors/sta1n.png" alt="STA1N API" width="50" /><br /><b><a href="https://sta1n.ai">STA1N API🍪</a></b> | 感谢<a href="https://sta1n.ai">STA1N API</a>对本项目的赞助！STA1N API 是一家专注于 RP（角色扮演）的 API 提供商。 |
| <img src="docs/sponsors/api-mart.png" alt="APIMart" width="50" /><br /><b><a href="https://go.apimart.ai/gh-rikkahub">APIMart</a></b> | Thanks to APIMart for sponsoring this project! APIMart is a low-cost API platform for AI image & video generation — GPT-Image-2 from $0.006/image, 160+ images per dollar. One async API covers both image and video: submit a task, get an ID, fetch results via polling or callback. Batch tens of thousands of images without timeouts, switch models without changing code. Pay-as-you-go with no monthly fee — <a href="https://go.apimart.ai/gh-rikkahub">sign up here</a> to get started. |
| <img src="docs/sponsors/suixiang.jpg" alt="随想AI中转" width="50" /><br /><b><a href="https://sui-xiang.com">随想AI中转</a></b> | 感谢<a href="https://sui-xiang.com">随想AI中转</a>对本项目的赞助！随想AI中转 是一家可靠高效的 API 中继服务提供商，提供 Claude、Codex、Gemini 等的中继服务。注重隐私的中转站·无数据倒卖·无模型掺水，隐私，透明，极速售后。新账户注册每日签到就送 0.5 元测试额度，充值额度 1:1，无需订阅，按量付费。多线路冗余、跨区域容灾、自动故障切换，长链路 SSE 不中断。99.9% 可用性，关键调用从不掉队。 |
| <img src="docs/sponsors/ztest.png" alt="真测 ztest.ai" width="50" /><br /><b><a href="https://ztest.ai">真测 ztest.ai</a></b> | 感谢<a href="https://ztest.ai">真测 ztest.ai</a>对本项目的赞助！真测 ztest.ai 是一个 AI 中转站模型检测平台，检测结果数据全公开，23 项探针覆盖协议、身份、能力、内容完整性、安全性、性能六大维度，交叉印证识别伪造与降级。作为独立第三方验证平台，实时监测 AI 中转站的模型真实性、响应质量与服务可用性。 |
| <img src="docs/sponsors/maru.png" alt="MaruCode" width="50" /><br /><b><a href="https://api.muteki.site/register?aff=Rikkahub&promo=Rikkahub">MaruCode</a></b> | <b><a href="https://api.muteki.site/register?aff=Rikkahub&promo=Rikkahub">MaruCode</a></b> 是一家偶尔做做慈善的小破站 API，自营号池，主要提供 Codex、Claude Code、GPT Image 等主流模型，支持 Websocket 协议，明码标价(Codex 0.25x, CC 1.5x)，透明汇率(1:1)，<a href="https://api.muteki.site/register?aff=Rikkahub&promo=Rikkahub">新用户注册送 2 刀</a>。<a href="https://images-2.muteki.site">生图工作台🖼️</a> |
| <img src="docs/sponsors/naapi.png" alt="NaAPI 钠API" width="50" /><br /><b><a href="https://naapi.cc/register?aff=rikkahub">NaAPI 钠API</a></b> | 感谢<a href="https://naapi.cc/register?aff=rikkahub">NaAPI</a>（naapi.cc）对本项目的赞助！NaAPI 是一个兼容 OpenAI 的 LLM 网关，以优惠价格稳定提供 OpenAI、Anthropic、Google 等厂商的 300+ 款模型。只需一个 Base URL 和一个 Key 即可接入；多线路冗余、自动故障切换，长链路 SSE 不中断；严格按量付费，充值 1:1，无需订阅，无隐藏加价。 |

## ✨ Features

- 🎨 Material 3 Expressive design
- 📦 Workspace: a proot-based Linux agent environment
- 📱 Phone Use: let the AI operate your phone
- 🎬 Media generation (image and video)
- 🔄 Multiple AI Provider Support: custom API / URL / models (all OpenAI, Google, Anthropic compatible api)
- 🖥️ Web access for multi-platform use
- 🛠️ MCP and Skills support
- 📝 Markdown Rendering (with code highlighting, Latex formulas, tables, Mermaid)
- 🔍 Search capabilities (Exa, Tavily, Zhipu, LinkUp, Brave, Perplexity, etc.)
- 🧩 Prompt variables (model name, time, etc.)
- 🤳 QR code export and import for providers
- 🤖 Agent customization
- 🧠 File-based memory
- 📝 AI Translation
- 🌐 Custom HTTP request headers and request bodies
- 💌 Silly Tavern character card import

## ✨ Development

> [!IMPORTANT]
> This project does not accept pull requests (PRs).

This project is developed using [Android Studio](https://developer.android.com/studio).

Technology stack documentation:

- [Kotlin](https://kotlinlang.org/) (Development language)
- [Koin](https://insert-koin.io/) (Dependency Injection)
- [Jetpack Compose](https://developer.android.com/jetpack/compose) (UI framework)
- [DataStore](https://developer.android.com/topic/libraries/architecture/datastore) (Preference data
  storage)
- [Room](https://developer.android.com/training/data-storage/room) (Database)
- [Coil](https://coil-kt.github.io/coil/) (Image loading)
- [Material 3 Expressive](https://m3.material.io/) (UI design)
- [Navigation 3](https://developer.android.com/guide/navigation/navigation-3) (Navigation)
- [Okhttp](https://square.github.io/okhttp/) (HTTP client)
- [kotlinx.serialization](https://github.com/Kotlin/kotlinx.serialization) (JSON serialization)

> [!TIP]
> You need a `google-services.json` file at `app` folder to build the app.

## 💰 Donate

* [Ko-fi](https://ko-fi.com/reovodev)
* [爱发电](https://afdian.com/a/reovo)

## ⭐ Star History

If you like this project, please give it a star ⭐

<a href="https://www.star-history.com/?type=date&repos=re-ovo%2Frikkahub">
 <picture>
   <source media="(prefers-color-scheme: dark)" srcset="https://api.star-history.com/chart?repos=re-ovo/rikkahub&type=date&theme=dark&legend=top-left&sealed_token=qSytWeq7LkzQQViTjK0MYlvvA_qkfuwjOxOqgbRpLUZZwok5rO6LXhpVL7Mq-q3o89BfKpzE7g66BCy18H6eiqTsD8czD0J-HejLqmHy-npcvCTHu11wZw" />
   <source media="(prefers-color-scheme: light)" srcset="https://api.star-history.com/chart?repos=re-ovo/rikkahub&type=date&legend=top-left&sealed_token=qSytWeq7LkzQQViTjK0MYlvvA_qkfuwjOxOqgbRpLUZZwok5rO6LXhpVL7Mq-q3o89BfKpzE7g66BCy18H6eiqTsD8czD0J-HejLqmHy-npcvCTHu11wZw" />
   <img alt="Star History Chart" src="https://api.star-history.com/chart?repos=re-ovo/rikkahub&type=date&legend=top-left&sealed_token=qSytWeq7LkzQQViTjK0MYlvvA_qkfuwjOxOqgbRpLUZZwok5rO6LXhpVL7Mq-q3o89BfKpzE7g66BCy18H6eiqTsD8czD0J-HejLqmHy-npcvCTHu11wZw" />
 </picture>
</a>

## 📄 License

This project is licensed under the [GNU Affero General Public License v3.0](LICENSE) (AGPL-3.0).
