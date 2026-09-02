# Virtual Steering Controller — Technical Stack

## 1. Purpose

This document selects the implementation technologies for a premium Android-to-Windows virtual steering controller. Choices prioritize deterministic input handling, maintainability, secure distribution, testability, and a realistic twelve-week delivery window.

## 2. Repository strategy

Use a monorepo for shared specifications and coordinated releases without forcing every component into one language.

```text
virtual-steering-controller/
├── apps/
│   ├── android-controller/
│   ├── windows-receiver/
│   ├── web/
│   └── admin/
├── services/
│   └── control-plane-api/
├── packages/
│   ├── protocol-spec/
│   ├── profile-schema/
│   ├── design-tokens/
│   └── test-vectors/
├── infrastructure/
├── docs/
├── scripts/
└── .github/workflows/
```

Use a protocol specification and generated test vectors as the cross-language contract. Do not attempt to share runtime UI code between Kotlin, Rust, and TypeScript.

## 3. Android controller stack

### Language and build

| Concern | Selection | Rationale |
|---|---|---|
| Language | Kotlin | Native access to sensors, networking, lifecycle, haptics, billing |
| Build | Gradle Kotlin DSL | Typed build configuration and Android ecosystem standard |
| UI | Jetpack Compose | Declarative UI, custom canvas controls, modern state handling |
| Architecture | Modular clean architecture + unidirectional data flow | Separates real-time engine from screens and services |
| Async | Kotlin Coroutines + Flow | Lifecycle-aware concurrent streams |
| DI | Hilt | Standardized construction and test substitution |
| Persistence | Room + DataStore | Profiles/history in Room; simple preferences in DataStore |
| Serialization | Kotlinx Serialization | Compact control-plane messages and profile schemas |
| Networking | Native UDP/DatagramChannel + TLS HTTP client | Direct control over real-time channel; secure cloud API |
| HTTP | OkHttp | Mature TLS, interceptors, and observability |
| QR | CameraX + ML Kit barcode scanner or ZXing | Secure pairing without manual addresses |
| Billing | Google Play Billing Library | Store-compliant products and subscriptions |
| Crash reporting | Sentry Android or Firebase Crashlytics | Symbolicated failures and release health |
| Analytics | PostHog or Firebase Analytics, minimized | Funnel measurement with strict event taxonomy |

### Android modules

```text
:app
:core:model
:core:protocol
:core:network
:core:sensors
:core:controls
:core:profiles
:core:billing
:core:designsystem
:feature:onboarding
:feature:pairing
:feature:controller
:feature:calibration
:feature:profiles
:feature:layouteditor
:feature:settings
:feature:diagnostics
```

The `:core:sensors`, `:core:network`, and `:core:controls` modules must not depend on Compose. The gameplay engine must be testable without rendering the interface.

### Sensor implementation

- Prefer game rotation vector where appropriate; fall back to gyroscope + accelerometer fusion.
- Use monotonic elapsed real-time timestamps, never wall-clock time.
- Capture at the best safe rate supported by the device, then publish controller state at a controlled rate.
- Compensate for display/device orientation.
- Provide stationary bias calibration.
- Detect missing gyro hardware and gracefully offer touch-wheel mode.
- Implement filters as swappable strategies: raw, low-pass, complementary/fused, and configurable curve.
- Measure filter-induced age so smoothing cannot silently add excessive delay.
- Avoid allocations inside high-frequency sample and packet loops.

### UI performance rules

- Keep live controller state outside broad application recomposition.
- Use `Canvas` or custom drawing for the steering wheel and telemetry.
- Use stable immutable UI state for screens and dedicated atomic/flow state for controls.
- Target the device refresh rate where reasonable while keeping input sampling independent of rendering.
- Profile frame time, dropped frames, thermal behaviour, and battery drain on low/mid/high-tier devices.

## 4. Windows receiver stack

### Core selections

| Concern | Selection | Rationale |
|---|---|---|
| Core language | Rust stable | Memory safety, predictable native performance, compact service |
| Async runtime | Tokio | UDP/TCP, timers, cancellation, task supervision |
| Desktop shell | Tauri 2 | Native Rust backend with a modern web UI and smaller footprint than Electron |
| UI | React + TypeScript + Vite | Fast desktop interface development and component ecosystem |
| State | Zustand or reducer-based local state | Lightweight UI state; backend remains authoritative |
| Styling | Tailwind CSS + shared tokens | Consistent premium interface without runtime CSS complexity |
| Local storage | SQLite via SQLx | Profiles, trusted devices, settings, migrations |
| Logging | `tracing` + rolling file appender | Structured diagnostics with session correlation |
| Errors | `thiserror` for libraries, `anyhow` at application boundary | Typed domain errors and contextual top-level handling |
| Serialization | `serde` | Profiles, IPC DTOs, control-plane messages |
| Metrics | In-process histograms/counters | RTT, jitter, packet age, drops, processing time |
| Installer | WiX Toolset or MSIX after spike | Reliable installation, repair, upgrade, uninstall |
| Updates | Signed update manifest and artifact verification | Prevent malicious or partial updates |

### Rust crate layout

```text
windows-receiver/
├── crates/
│   ├── receiver-app
│   ├── receiver-core
│   ├── protocol
│   ├── discovery
│   ├── pairing
│   ├── transport
│   ├── input-pipeline
│   ├── virtual-device
│   ├── force-feedback
│   ├── profiles
│   ├── diagnostics
│   └── updater
└── ui/
```

`virtual-device` defines a trait such as `VirtualControllerBackend`. Concrete driver integrations remain behind it. The remainder of the product must not import a driver-specific SDK.

### Virtual controller decision

The receiver should initially emulate an Xbox-compatible controller because games widely recognize XInput. Before implementation freeze, complete a legal and technical spike covering:

- Driver maintenance status.
- Redistribution licence.
- Driver signing and install behaviour.
- Windows 10/11 compatibility.
- XInput and DirectInput coverage.
- Force-feedback support.
- Anti-cheat implications; never claim universal compatibility.
- Clean uninstall and upgrade.
- Alternative backend feasibility.

ViGEmBus may be used for an MVP only after this review. It is archived and therefore must not become an unreplaceable architectural dependency.

### Windows UI responsibilities

- First-run setup and health checks.
- Pairing QR/PIN.
- Connected-device and input visualization.
- Profile selection/editing.
- Driver, firewall, and update status.
- Diagnostics export.
- Start-on-login and tray behaviour.

The webview UI must not process controller packets. All real-time work remains in Rust.

## 5. Real-time protocol stack

### Transport split

| Channel | Transport | Purpose |
|---|---|---|
| Discovery | mDNS/DNS-SD plus UDP fallback | Find receivers on a LAN |
| Pairing | TCP/TLS or authenticated HTTP on LAN | Exchange identity and session material |
| Input state | UDP | Latest steering/pedal/button state |
| Reliable events | Reliable LAN channel | Profile transfer, critical configuration, capability exchange |
| Force feedback | UDP with sequence/timestamp | Timely rumble/haptic commands |
| Cloud API | HTTPS/TLS | Entitlements, accounts, profiles, releases |

### Protocol format

- Define protocol version, capabilities, packet layout, units, endianness, error handling, and upgrade rules in `packages/protocol-spec`.
- Use fixed-size binary gameplay packets.
- Quantize axes consistently, preferably signed/unsigned 16-bit values.
- Use a bitmask for buttons.
- Include session ID, sequence number, monotonic timestamp, flags, and authentication tag.
- Keep packets well below MTU; never depend on fragmentation.
- Authenticate packets after pairing to reject unrelated LAN injection.
- Generate golden packet test vectors consumed by Kotlin and Rust tests.

Protocol evolution rules:

- Major versions may break packet format.
- Minor capabilities are negotiated.
- Unknown flags are ignored only when specified as safe.
- Receiver rejects malformed lengths before parsing fields.
- Old clients receive a human-readable upgrade message on the reliable channel.

## 6. Web and admin stack

| Concern | Selection |
|---|---|
| Framework | Next.js with TypeScript |
| Styling | Tailwind CSS and shared design tokens |
| Components | Accessible headless components such as Radix primitives |
| Content | MDX for documentation and release notes |
| Forms | React Hook Form + Zod |
| Testing | Vitest + React Testing Library + Playwright |
| Hosting | Cloudflare or managed Node platform based on API needs |
| Downloads | Object storage + CDN with signed release metadata |

The marketing site and documentation can live in one app. The admin interface should be separately authorized even if it shares packages.

## 7. Backend/control-plane stack

### Recommended initial implementation

Use a modular monolith rather than microservices.

| Concern | Selection | Notes |
|---|---|---|
| API language | TypeScript/NestJS or Rust/Axum | Choose based on team speed; TypeScript is faster for a solo 12-week build |
| API style | REST with OpenAPI | Simple clients and generated contracts |
| Database | PostgreSQL | Accounts, entitlements, devices, profiles, audit records |
| ORM | Prisma if TypeScript; SQLx if Rust | Migrations must be committed and reviewed |
| Cache/rate limit | Redis only when needed | Avoid operational complexity during MVP |
| Auth | Managed OIDC/passwordless provider or audited library | Do not build password crypto from scratch |
| Object storage | S3-compatible | Receiver builds, support attachments, optional profile assets |
| Email | Transactional provider | Verification and critical account notices only |
| Jobs | Database-backed worker initially | Purchase rechecks, cleanup, release processing |
| Observability | OpenTelemetry + Sentry/log platform | Correlated API traces and errors |

### Backend modules

- Identity and sessions.
- Installations and devices.
- Billing verification and entitlements.
- Profiles and versions.
- Releases and update channels.
- Feature flags.
- Analytics ingestion with allowlisted events.
- Support and consented diagnostics.
- Administration and audit log.

## 8. Data storage decisions

### Mobile local data

- Paired receiver identities.
- User preferences.
- Local profiles and layouts.
- Cached entitlements with expiry/grace metadata.
- Limited diagnostic history.
- No raw long-term sensor recording by default.

### Receiver local data

- Trusted mobile public identities.
- Profiles and executable associations.
- Driver/setup health.
- Update preferences.
- Rotating logs with size and retention limits.

### Cloud data

- Account and verified identities.
- Purchase tokens stored or transformed according to platform guidance.
- Entitlements and transaction history.
- Registered devices.
- Private synced profiles.
- Consent and privacy state.
- Minimal aggregate analytics.
- Admin audit records.

## 9. Security stack and standards

- Threat-model pairing, LAN spoofing, replay, account theft, entitlement tampering, update compromise, and malicious profile payloads.
- Generate device keypairs using platform secure storage where feasible.
- Pair with an ephemeral challenge displayed on the receiver.
- Derive per-session keys; rotate them on reconnect.
- Authenticate every live input packet; encryption is optional for performance only after threat analysis, but authentication is mandatory.
- Rate-limit discovery and pairing attempts.
- Bind receiver services only to intended interfaces.
- Verify signed update manifests and artifact hashes.
- Never execute commands embedded in profiles.
- Validate all profile schemas and cap payload sizes.
- Store mobile secrets in Android Keystore.
- Use Windows Credential Manager/DPAPI for receiver secrets.
- Use dependency scanning, secret scanning, SBOM generation, and release provenance.

## 10. Testing stack

### Android

- JUnit 5 or platform-compatible JUnit.
- Turbine for Flow testing.
- MockK where a fake cannot be used.
- Compose UI tests.
- Macrobenchmark and Baseline Profiles.
- Physical-device matrix for sensors, refresh rates, and thermal testing.

### Rust receiver

- Unit tests and property tests with `proptest`.
- Tokio paused-time tests for timeout logic.
- Fuzz protocol decoders.
- Integration tests using a fake virtual-controller backend.
- Criterion microbenchmarks for packet decode and pipeline processing.
- Windows VM and physical-machine installer tests.

### Cross-component

- Golden packet vectors shared between Kotlin and Rust.
- Network impairment tests: delay, jitter, reordering, duplication, and loss.
- Long-duration soak tests.
- Compatibility tests per supported game and Windows version.
- Purchase sandbox tests, restore, refund, expiry, grace, and offline cases.
- Playwright website and download-flow tests.

## 11. CI/CD and release engineering

Use GitHub Actions or equivalent with separate pipelines:

1. Pull request: formatting, lint, unit tests, protocol compatibility, dependency audit.
2. Android beta: signed internal App Bundle, release notes, test distribution.
3. Receiver beta: reproducible build, signing, installer, hashes, malware scan submission workflow.
4. Web/API: migration check, preview deployment, integration tests, production approval.
5. Coordinated release: publish version compatibility matrix and signed manifest.

Version independently but publish compatible sets:

- Android semantic version.
- Receiver semantic version.
- Protocol major/minor.
- Profile schema version.
- Backend API version.

Protect signing keys in CI secrets or a hardware-backed signing service. Never expose them to pull-request builds.

## 12. Code-quality standards

- Kotlin: ktlint, detekt, explicit API for core modules.
- Rust: rustfmt, Clippy with warnings denied in CI, cargo-audit, cargo-deny.
- TypeScript: strict mode, ESLint, Prettier, no unchecked `any` in domain contracts.
- Conventional commits or another consistent change format.
- Architecture decision records for irreversible choices.
- Required tests for protocol, billing, pairing, and fail-safe changes.
- No log statements containing auth tokens, pairing secrets, purchase tokens, or complete IP histories.

## 13. Observability taxonomy

### Local real-time metrics

- Sensor sample age.
- Packet publication interval.
- RTT and one-way estimate where reliable.
- Jitter.
- Packet loss, reordering, and duplicate rate.
- Receiver decode time.
- Virtual-device write time.
- End-to-end synthetic loopback time.

### Product analytics events

- Onboarding started/completed.
- Receiver download opened.
- Pairing attempted/succeeded/failed with coarse reason.
- Calibration completed.
- Gameplay session started/completed with duration bucket.
- Profile selected.
- Paywall viewed.
- Trial/purchase/restoration outcome.

Do not upload raw controller movement or button sequences as analytics.

## 14. Developer environment

Recommended prerequisites:

- Android Studio stable and matching JDK.
- Rust stable via rustup.
- Node.js current LTS, not experimental current releases.
- pnpm with locked version through Corepack.
- PostgreSQL through Docker for local backend development.
- Windows 10/11 physical test machine or VM with access to required virtual-device setup.
- At least three Android physical devices spanning low, medium, and high performance.

Provide one bootstrap script per operating system, `.env.example` files containing no secrets, seeded development data, and a fake virtual-controller backend for contributors who cannot install the Windows driver.

## 15. Technology decision gates

Resolve these during week 1–2:

1. Virtual controller backend and licensing.
2. WiX versus MSIX installer.
3. TypeScript versus Rust control-plane API.
4. Managed authentication provider.
5. Error/analytics providers and privacy implications.
6. Code-signing certificate procurement.
7. Minimum Android/Windows versions.
8. CDN/object-storage vendor.

No technology should be adopted solely because it is fashionable. Each selection must improve latency, delivery speed, safety, or maintainability.

