# Virtual Steering Controller — Product Definition

> Working title: **DriveDeck**. This is a placeholder and must pass trademark, domain, Play Store, App Store, GitHub, and social-handle checks before launch.

## 1. Document purpose

This document defines the product we intend to build: an Android phone application that becomes a responsive steering wheel and general-purpose controller for Windows PC games, supported by a Windows receiver, product website, and a small cloud control plane. It is the product source of truth for vision, audience, scope, commercial model, success metrics, launch, and post-launch direction.

Related documents:

- `Tech-Stack.md`: selected technologies and engineering standards.
- `System-architecture.md`: component boundaries, protocols, data flows, security, and deployment.
- `Project-execution.md`: twelve-week implementation and release plan.

## 2. Product thesis

Dedicated racing wheels deliver excellent control but are expensive, occupy desk space, and are unnecessary for casual players. Keyboards offer poor analog steering. Standard controllers are better but still require additional hardware. Modern phones already contain responsive touchscreens, accelerometers, gyroscopes, vibration motors, networking, and high-refresh-rate displays.

DriveDeck turns that existing hardware into a polished PC controller. A player installs the mobile app and Windows receiver, pairs them on the same network, chooses a game profile, calibrates once, and plays.

The commercial opportunity is not merely “phone sends gyro data.” The product wins through:

1. A trustworthy Windows installation.
2. Pairing that works without typing IP addresses.
3. Stable and measurable low latency.
4. Excellent calibration and game-specific defaults.
5. Controls designed for eyes-off use.
6. Compatibility with games that expect standard Xbox input.
7. A fair free tier and a compelling one-time Pro upgrade.

## 3. Vision, mission, and positioning

### Vision

Make responsive PC driving controls accessible to anyone who owns a smartphone.

### Mission

Deliver the fastest, easiest-to-set-up, and most customizable phone-to-PC gaming controller, without requiring cloud streaming or dedicated hardware.

### Positioning statement

For PC players who want analog steering without buying a wheel, DriveDeck turns an Android phone into a precise, configurable controller. Unlike basic remote-control utilities, it provides automatic pairing, guided calibration, game-tested profiles, connection diagnostics, and a premium interface built specifically for gameplay.

### Product principles

- **Local first:** live input stays on the local network or USB connection.
- **Latest input wins:** stale controller events are discarded, never queued.
- **Playable before customizable:** sensible presets must work before users open advanced settings.
- **Trust is a feature:** signed installers, transparent permissions, no hidden background behaviour.
- **Graceful degradation:** packet loss or cloud unavailability must not make the controller unsafe or unusable.
- **Fair monetization:** core evaluation is free; permanent local features are available through a lifetime purchase.
- **Original product:** achieve functional parity where useful, but do not copy another product’s name, code, art, store assets, copywriting, or exact interface.

## 4. Target users

### Primary persona: casual PC racer

- Plays Forza, GTA, BeamNG, Euro Truck Simulator, Need for Speed, or similar games.
- Uses a keyboard and wants analog steering.
- Is unwilling or unable to purchase a physical racing wheel.
- Values fast setup more than advanced simulation telemetry.

Primary job: “Let me steer naturally with my phone in under five minutes.”

### Secondary persona: budget simulation player

- Wants 360°, 540°, 900°, or greater rotation behaviour.
- Owns a phone mount or attaches the phone to a basic wheel.
- Adjusts curves, centre offset, dead zones, pedals, clutch, handbrake, and gears.

Primary job: “Give me more precision than a keyboard or thumbstick without expensive equipment.”

### Secondary persona: customizable-controller user

- Plays open-world, arcade, flight, farming, or accessibility-oriented games.
- Wants custom buttons, axes, touchpads, gyro aiming, and per-game layouts.

Primary job: “Let me create a controller that matches my game and physical comfort.”

### Future persona: creator and community expert

- Publishes layouts, calibration guides, and game profiles.
- Builds a following around racing configurations.
- Needs profile versioning, attribution, ratings, and sharing.

### Initial geographic focus

- English-language global launch.
- Pricing localized for the UK, US, EU, India, Brazil, and other high-volume Android markets.
- Documentation initially in English; localization follows usage data.

## 5. Jobs to be done

| Situation | User need | Product response |
|---|---|---|
| Keyboard steering is binary | Smooth analog turning | Tilt, wheel, and gyro modes |
| Physical wheel is unaffordable | Low-cost alternative | Free starter tier and lifetime Pro |
| Receiver setup is confusing | Guided installation | Pairing wizard, QR code, automatic discovery |
| Settings vary per game | Known-good defaults | Tested game profiles and calibration presets |
| Wi-Fi quality varies | Diagnose lag | RTT, jitter, packet-loss, and sensor-age monitor |
| Touch buttons are hard to find | Eyes-off operation | Large zones, haptics, layout lock, edge anchoring |
| Users change games | Quick switching | Automatic or one-tap profile selection |
| Cloud is unavailable | Continue playing | Offline entitlement grace and local profiles |

## 6. Product surfaces

### 6.1 Android application

Responsible for sensors, touch controls, controller rendering, profile editing, pairing, connection diagnostics, haptics, purchases, and optional account features.

### 6.2 Windows receiver

Responsible for discovery, secure sessions, packet ingestion, input normalization, virtual-controller output, force-feedback capture, game profiles, diagnostics, updates, and Windows integration.

### 6.3 Website

Responsible for product marketing, Windows downloads, documentation, troubleshooting, compatibility status, privacy policy, terms, release notes, and support.

### 6.4 Cloud control plane

Responsible for accounts, entitlements, device limits, profile backup, public profile metadata, release manifests, feature flags, support diagnostics with consent, and aggregate analytics. It is not part of the gameplay data path.

## 7. MVP scope

The three-month objective is a high-quality Android and Windows public beta, followed by a production candidate. “MVP” means commercially testable, not disposable.

### Mobile MVP

- Android 10+ target, subject to final device testing.
- Landscape controller experience.
- Tilt steering using fused sensor data.
- Touch steering wheel with auto-centre and configurable rotation.
- Analog throttle and brake.
- Digital clutch, handbrake, gear up/down, pause, camera, and configurable buttons.
- Steering dead zone, saturation, sensitivity, linearity, smoothing, inversion, centre offset, and rotation range.
- Automatic LAN receiver discovery.
- QR/PIN pairing and remembered trusted PCs.
- Connection status, battery warning, RTT, jitter, and packet-loss indicators.
- Calibration wizard with live axis visualization.
- Built-in game profiles.
- Basic layout editing, saving, duplication, reset, import, and export.
- Haptic feedback for UI actions and supported game rumble events.
- Free and Pro entitlement states.
- Privacy-respecting diagnostics and crash reporting with consent.

### Windows MVP

- Windows 10 and 11 support.
- Signed x64 installer for release.
- First-run dependency and virtual-device setup.
- Local discovery advertisement.
- Pairing approval and trusted-device management.
- Low-latency UDP input receiver.
- Standard Xbox-compatible controller output.
- Calibration and live input tester.
- Game profile selection and per-game mappings.
- Local logs with exportable support bundle.
- Firewall guidance and automated rule creation where permitted.
- Minimize-to-tray and optional start-on-login.
- Safe automatic update checks with signed manifests.

### Website MVP

- Landing page.
- Windows download page with checksums and signing information.
- Getting-started guide.
- Pairing and calibration guide.
- Supported-game matrix.
- Troubleshooting for firewall, discovery, latency, drivers, and controllers.
- Pricing, FAQ, privacy policy, terms, refund guidance, and contact route.

### Backend MVP

- Anonymous installation identifier.
- Optional email or social account.
- Google Play purchase verification.
- Entitlement API and offline grace policy.
- Device registration and revocation.
- Private profile sync for paid cloud plan, if schedule permits.
- Release manifest and minimum-supported-version policy.
- Aggregate funnel and reliability telemetry without collecting gameplay content.

## 8. Explicit non-goals for the first release

- iOS support.
- macOS or Linux receiver.
- Cloud gaming or video/audio streaming.
- Internet relay for controller packets.
- Kernel-driver development from scratch.
- Competitive esports certification.
- Full community marketplace.
- Multiplayer with many phones on one PC, beyond experimental support.
- Game automation, anti-recoil, cheating features, or anti-cheat bypassing.
- Guaranteed compatibility with every PC game.
- Full racing telemetry dashboards.

## 9. Feature roadmap

### Release 1.1

- USB tethering/direct USB transport.
- Advanced layout editor.
- Automatic executable-to-profile selection.
- More game profiles.
- Input recording for diagnostics.
- Better accessibility controls.

### Release 1.2

- Full gamepad mode.
- Gyro mouse and touch-camera mode.
- Cloud profile sync.
- Shareable profile links.
- Desktop overlay for calibration and connection health.

### Release 2.0

- iOS client.
- Community profile library.
- Multi-controller sessions.
- Telemetry-based dashboards for supported games.
- Creator attribution and moderation.
- Alternative maintained virtual-device backends.

## 10. UX requirements

### Onboarding success path

1. User opens the mobile app and sees a concise explanation.
2. The app asks only for permissions that are immediately needed.
3. It detects whether required motion sensors exist.
4. It directs the user to install the Windows receiver.
5. The receiver displays a QR code and six-digit PIN.
6. The phone discovers the receiver and scans/enters the code.
7. Both sides establish an authenticated trusted-device relationship.
8. The receiver confirms virtual controller availability.
9. A calibration wizard guides neutral position, full steering, and pedals.
10. The user selects a game and runs an input test.
11. The app transitions into distraction-free controller mode.

Target: at least 70% of users who install both applications complete their first connection.

### Visual direction

- Graphite/near-black base with one distinctive electric accent.
- Strong typography and high contrast.
- Deliberate, short transitions; no decorative animation during input.
- Large touch targets and safe-area awareness.
- Controls must remain legible under different screen sizes and aspect ratios.
- Avoid using colour as the only status signal.
- Provide high-contrast and reduced-motion options.

### Control ergonomics

- Lock layout during play.
- Adjustable opacity, size, position, haptic strength, and handedness.
- Edge-anchored pedals and buttons.
- Optional touch guards to prevent palm input.
- Screen-awake mode with battery/thermal warning.
- Immediate neutralization when app loses focus unless the user selects a safe alternative.

## 11. Monetization strategy

### Recommended model

Use freemium + lifetime Pro + optional recurring cloud membership.

| Tier | Indicative price | Value |
|---|---:|---|
| Free | £0 | Core tilt/wheel, basic pedals, two profiles, basic calibration |
| Pro Lifetime | £19.99 launch target | All permanent local modes, unlimited profiles, advanced curves, editor, USB when released, no ads |
| Pro Cloud | £2.99/month or £19.99/year | Sync, device backup, premium profile updates, community features, telemetry history |
| Founder offer | £9.99–£14.99 once | Time-limited early supporter lifetime Pro |

Prices are hypotheses. Validate through Play Store experiments and user interviews. Localize rather than simply converting GBP.

### Monetization rules

- Gameplay must never be interrupted by advertising.
- If ads are tested, restrict them to low-frequency menu placements in the free tier.
- Do not require a recurring fee for functionality that is entirely local and permanently delivered.
- Subscriptions must fund continuing value such as sync, community content, ongoing premium profiles, and telemetry storage.
- Provide entitlement grace when the device is offline.
- Restore purchases reliably.
- Explain trial, renewal, cancellation, and refund terms plainly.
- Verify purchases server-side; never trust a client boolean.
- Use Google Play Billing for in-app digital functionality where required by store policy.

### Conversion moments

- After the user completes a successful session, not before proof of value.
- When saving a third profile.
- When selecting an advanced curve or editor feature.
- When enabling cloud sync.
- After a seven-day trial demonstrates premium value.

### Revenue scenarios

These are planning models, not promises.

| Scenario | Installs | Lifetime conversion | Annual conversion | Approx. gross |
|---|---:|---:|---:|---:|
| Conservative | 20,000 | 2% at £19.99 | 0.5% at £19.99 | ~£10,000 |
| Base | 100,000 | 4% at £19.99 | 1% at £19.99 | ~£100,000 |
| Strong | 500,000 | 5% at £19.99 | 2% at £19.99 | ~£700,000 |

Gross values exclude store fees, taxes, refunds, payment costs, marketing, infrastructure, code signing, support, and currency effects.

## 12. Go-to-market strategy

### Pre-launch

- Recruit 50–100 closed testers across different Android phones, routers, and games.
- Publish engineering updates and latency demonstrations.
- Create side-by-side keyboard-versus-phone gameplay clips.
- Establish Discord or another community support channel only if it can be actively moderated.
- Contact small racing, BeamNG, ETS2, and budget-gaming creators.
- Build a transparent compatibility matrix.

### Launch assets

- 15–30 second setup demonstration.
- Real uncut latency demonstration.
- Separate videos for Forza, BeamNG, Assetto Corsa, and ETS2.
- Store screenshots showing pairing, steering, calibration, profiles, and diagnostics.
- Search-optimized landing pages for “use phone as steering wheel for PC” and specific games.

### Growth loops

- Shareable game profiles.
- In-app request for rating after multiple successful sessions.
- Referral link for Founder pricing.
- Community voting on the next supported game.
- Public release notes and reliability improvements.

## 13. Success metrics

### North-star metric

Weekly successful gameplay sessions of at least ten minutes.

### Activation funnel

| Metric | Initial target |
|---|---:|
| Mobile install → receiver download | 40% |
| Receiver installed → paired | 70% |
| Paired → completed calibration | 80% |
| Calibrated → first 10-minute session | 70% |
| Day-7 retention | 20%+ |
| Trial → paid | 5%+ |

### Engineering service-level objectives

- Crash-free mobile sessions: ≥99.5% beta, ≥99.8% production.
- Crash-free receiver sessions: ≥99.8%.
- Median local network RTT on healthy Wi-Fi: <10 ms.
- 95th percentile receiver packet processing: <2 ms.
- Pairing success on supported configurations: ≥95%.
- No stale control held beyond fail-safe timeout.

## 14. Legal, privacy, and trust

- Perform naming and trademark checks before adopting the final brand.
- Use only original graphics, interface layouts, text, and code.
- Audit all third-party licences, especially virtual-controller drivers.
- Publish privacy policy and terms before closed testing expands.
- Collect the minimum data necessary.
- Never collect the game’s screen, keystrokes outside mapped controls, unrelated device files, or network traffic.
- Encrypt account and profile traffic in transit.
- Make diagnostics opt-in where they could include detailed device/network information.
- Provide account and cloud-data deletion.
- Clearly explain local-network discovery and firewall permissions.
- Avoid claims such as “zero latency,” “works with every game,” or “anti-cheat safe.”

## 15. Major risks and mitigations

| Risk | Impact | Mitigation |
|---|---|---|
| Windows installer is flagged | Severe funnel loss | Code signing, reputation building, checksums, transparent documentation |
| Virtual-controller dependency becomes obsolete | Compatibility loss | Backend abstraction, licence audit, alternate implementation spike |
| Router blocks discovery | Pairing failure | QR/manual IP fallback, hotspot and firewall guidance |
| Sensor quality varies | Poor steering | Capability detection, calibration, device presets, adaptive filtering |
| Excess smoothing adds lag | Bad control feel | Measure sensor age; expose presets; cap filter delay |
| Scope exceeds 12 weeks | Missed launch | Freeze MVP at week 2; defer cloud/community extras |
| Subscription rejection | Weak revenue | Lifetime Pro as primary offer; subscription only for ongoing value |
| Support burden grows | Slow development | Diagnostic bundle, compatibility matrix, guided troubleshooting |
| Cheating misuse | Store/reputation risk | No automation or bypass tools; responsible mappings and policies |

## 16. Definition of MVP success

The MVP is successful when a new user can install both applications, pair without entering an IP address, calibrate, launch one of the supported games, play for thirty minutes with stable steering, recover safely from a disconnect, and understand the upgrade offering—without developer assistance.

## 17. Product decisions still requiring validation

- Final brand name and identity.
- Minimum Android version based on test-device coverage.
- Virtual-controller backend and redistribution/licensing approach.
- Exact free-tier limits and regional prices.
- Whether cloud sync belongs in launch or version 1.1.
- Initial supported-game list based on access to games and testers.
- Code-signing certificate and installer technology.
- Whether accounts are optional for lifetime purchase restoration beyond Play.

