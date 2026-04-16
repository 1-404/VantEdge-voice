VantEdge Voice
A local-first, mathematically private voice OS layer for Android.
Privacy is not a promise. It's architecture.

What It Does
VantEdge Voice is a voice operating system that sits between your mic and your Android apps. You speak. It understands. It invokes any Android intent — and then it stops.
No data leaves your device unless you explicitly ask for it. No accounts. No tracking. No cloud by default.
Core Capabilities
On-Device Wake Word & STT

Fixed "Hey VantEdge" wake word, fully on-device via openWakeWord

Why fixed? One auditable pattern. Fully open-source, fully verifiable.
Voice training (opt-in, heavily suggested): Say commands a few times for dramatically better accuracy on your voice, accent, and environment. Here's how we keep it safe: extract voice hashes and acoustic features, discard raw audio immediately, train locally on-device, never transmit voice data. You get better accuracy. We never hear you.
Custom wake words? Roadmap for v4+, after the core architecture is solid.


Distributed mic support — route voice through multiple LAN devices, pick the best one
Three-tier voice processing: wake word → fixed vocabulary commands → complex queries on demand
Confidence scoring, no mystery — you see exactly how confident we are

Intent System — Zero Data Leaving VantEdge

Fire standard Android intents to any app (navigation, calls, music, third-party automation)
VantEdge does not know which app answered or what happened next
Privacy guarantee is mathematical, not a promise — Android OS mediates everything
Intent handlers are separate apps; VantEdge structurally cannot see them

Token-Based Capability Model — Permission & Delegation OS

This is where VantEdge stops being a "voice assistant" and becomes infrastructure.
Five-tier token hierarchy (server, hub, node, guest, NFC) — all anonymous, all cryptographic
Permissions encoded as tokens, not user accounts
Hub/node example: boss's phone = hub, crew phones = nodes with scoped intent allowlists
NFC handoff: one tap, instant permission grant, no pairing, no accounts, no setup

"Hey VantEdge, give Johnson materials log access" + tap = done
Johnson's phone gets a scoped token. Knows exactly what he can do. Token expires when you revoke it.
This is the physical embodiment of capability-based security. No UI friction. No identity lookup. Just capability.


Single-use tokens, short TTLs, revocation on the fly

The juice: you're not managing users or accounts. You're delegating capabilities. That's the paradigm shift.

Distributed Mic Architecture (coming v2)

Multiple LAN devices act as mic nodes
Each mic scores its own audio quality and proximity
Hub selects best source automatically — no user thinking about it
Consensus wake word detection across mics — catch misses, reject false positives
Old Android phones, ESP32 boards, Raspberry Pis all work as nodes
Note: WiFi RTT proximity estimation in testing — may refine in later versions

LAN-First, Cloud-Optional

Hub/node communication over LAN only — no phone home by default
Self-hosted Ollama on your server? Route complex queries there
External LLM? Opt-in, explicit, user's responsibility


Worksite Access Control (MVP feature)

Crew member wants the job? They install VantEdge on their own phone.
During onboarding, they grant you device admin permissions — for that project only.
You issue a scoped token: materials log, time tracking, navigation, job comms. Nothing else.
VantEdge controls WiFi and mobile data — crew can't browse, can't install apps, can't access their personal data during work.

Breaks: Token includes break windows (e.g., 12-1pm, 3-3:15pm). During breaks, WiFi/data automatically re-enable. Crew can check messages, use their phone normally.
End of shift: Token expires → WiFi/data revert to normal user control, full phone access restored.


No hardware to manage. No phones to distribute. Just capability distribution.
This is the token model solving contractor workflow: lock down access for a job, release it when done, allow breathing room during breaks.


What This Actually Is

Not a voice assistant. (That's boring. Everyone's building those.)
A voice-first permission and delegation operating system.
You build the assistant on top of it. Skills are just Android intent handlers. VantEdge is the plumbing that mediates permissions, enforces least privilege, handles delegation, and makes it all work offline with zero accounts.
The voice part? That's just how you talk to it.
The real magic is the capability model underneath. That's what lets you:

Delegate to crew members with a tap
Revoke access instantly
Never track anyone
Never create an account
Never send data to the cloud
Run entirely on your own hardware

You get a beautiful assistant later. Today, you're building the OS that makes it possible.

Architecture Highlights
Voice Pipeline (Three Tiers)
Tier 1 — Wake Word

Porcupine alternative: openWakeWord, fully on-device, fixed phrase
Runs on every mic node simultaneously
Consensus detection across nodes = smarter than a single mic

Tier 2 — Fixed Vocabulary

Registered commands only ("Add expense", "Log time", "Call Johnson")
Quantized INT8 model, tiny, runs on NNAPI hardware acceleration
Faster and more accurate than general STT on known vocabulary
This is the daily driver for contractors and home automation

Tier 3 — Complex Queries

Loads on demand for unknown intents
Ollama on your LAN server preferred; on-device fallback
User expects a pause ("give me a second") — no surprise

Token Hierarchy (All Anonymous)
Server token    → subscription state (Cloudflare KV, not your data)
Hub token       → LAN master device, scoped permissions
Node token      → crew/family device, issued by hub, limited to hub's intent allowlist
Guest token     → temporary access, expires on LAN departure or explicit revoke
NFC token       → instant permission grant via tap, single-use or time-limited
All tokens are cryptographic capability objects, not identity documents. No usernames. No email. No tracking.
Intent System — Mathematical Privacy
You say: "Call Johnson"
       ↓
VantEdge parses intent: {action: CALL, contact: Johnson}
       ↓
Fires Android intent to phone app (outbound = standard broadcast)
       ↓
Phone app handles the call
       ↓
Phone app binds back via VantEdge's custom permission gate
       ↓
Sends confirmation: {success: true, contact: Johnson}
       ↓
VantEdge receives via bound service, logs confirmation, stops.
The architecture: Outbound intents are broadcast (anyone can see). Return data comes back through a bound service protected by a custom signature-level permission — only apps VantEdge explicitly trusts can bind and reply.
This means: yes, data returns (for confirmation, logging, UI feedback). But the return path is mathematically gated. Only pre-approved apps can communicate back. VantEdge cannot see what apps did — only that they completed the action.
The privacy guarantee still holds: Android OS mediates everything. VantEdge structurally cannot invoke untrusted apps or receive data from them.
Hub/Node Example
Boss: "Hey VantEdge, give Johnson site access"
      *tap phones*
Johnson: "VantEdge site access granted — materials log, time tracking, navigate"

What Johnson sees:
- Scoped token in encrypted storage
- Intent allowlist (only these commands work)
- No identity
- Permissions expire when token expires

What the hub sees:
- Node joined
- Token hash (not the token itself)
- No personal data

What the server sees:
- Nothing. Ever.

Current Status
MVP (targeting Q2 2026):

On-device wake word (openWakeWord)
Android SpeechRecognizer + command-logic fuzzy matching
Intent firing to standard apps
Confidence scoring and unknown-intent handling
Token-based permissions (scoped intents, device admin)
Worksite access control (WiFi/mobile data lockdown per job)
Subscription model (no Google Play Billing, ever)

V1 (Q3 2026):

Hub/node architecture
Token-based device state management
NFC handoff for crew onboarding
Multi-site token management

V2 (Q4 2026):

Distributed mic architecture
Consensus wake word detection
Receipt OCR, basic NLP
Self-hosted Ollama routing

V3+ (2027):

Community contributed skills and domain models
GPS mileage (opt-in)
Desktop (only if users request)



- Integration Points
- Home Assistant
Voice-first control via standard intents. No special HA integration needed — use Android intent system.
- ESPHome
Mic nodes can run on ESP32-S3 boards. Same mic scoring protocol, same LAN coordination.
- Third-Party Skills

Build as separate Android apps. Implement Android intent handlers. VantEdge fires intents and stops. Privacy is mathematical.

Your Own Infrastructure

Ollama on a LAN server? Route complex queries there. WireGuard for untrusted networks? Built in.

Design Principles

Mathematical, Not a Promise

Privacy is enforced by architecture, not policy. You don't have to trust us — you can verify the code.

No Accounts. Ever.

Tokens replace identity. No usernames, emails, or tracking.

Local First. Cloud Optional.

Default = fully offline. Cloud is opt-in, explicit, user's responsibility.

Tinkerer-Friendly

Built for people who plug in old Android phones, Raspberry Pis, and ESP32 boards around the house.

Open Where It Matters

Core plumbing (voice-module, command-logic) is public. Competitive advantage (voice-processing pipeline) stays private until post-launch.

Why We're Building This

The Home Assistant and Rhasspy communities demand privacy-first voice. They want to own their data. They're tired of closed ecosystems.
We're building the infrastructure. Contractors, small business owners, and HA enthusiasts will build the features.

Feedback Welcome

We're designing this in public. Architecture feedback, security review, use-case discussion — all valuable.

Contact: dev@vantedgevoice.app

GitHub: github.com/1-404

Status: Coming soon. Want to help shape it?

Quick Links

Full Documentation — architecture deep-dives, integration guides, security considerations
Token System Explained — how capability tokens work
Distributed Mic Protocol — consensus wake word, mic scoring
Third-Party Skills — build intent handlers, privacy guarantees


VantEdge Voice

Mathematical. Not a promise.

intent.fire()

// we stop here

Stop the snoop.
