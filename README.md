# Human Context Protocol (HCP)

**v0.1 — co-designed by Kevin and Claude, April 2026. Open for argument.**

> An open standard for how AI tools communicate with humans — designed around human constraints, not AI capabilities.

---

## The problem

MCP gave AI a clean way to talk to tools. The reverse — AI talking to humans — is currently solved with notifications that interrupt regardless of stakes, buttons that put the cost of the AI's confidence model on the user, engagement metrics that turn human attention into a resource to mine, and per-app preference panels that don't compose. The result is software that talks at humans, not with them. Tools claim to surface "what's important" but can't tell what important means to *this* person, on *this* day, in *this* project. Builders reach for the same anti-patterns over and over because design conventions encourage it.

HCP is a standard for doing this differently.

## What HCP is

HCP is two things at once.

A **runtime contract** that defines how AI systems should communicate with the humans they work for — what's allowed, what's required, what's forbidden — gated by a human-declared profile.

A **design specification** that AI assistants read and apply when they help build software. The deployed product may have no AI in it. But if Claude (or any AI dev tool) helped design it, HCP-shaped behavior shipped with the architecture. This is HCP's adoption story: the standard travels through development, not just runtime.

Both halves serve the same goal: **the human can always tell what's true, what's uncertain, what's being asked of them, and what the stakes are.** Action follows from clarity, not persuasion. HCP is an explicit rejection of engagement-as-metric.

## What HCP is not

It's not a UI framework, a notification spec, or a tone-and-voice guide. It's not a settings file — the profile encodes values that translate into operational rules, not just preferences. It's opinionated but not a moral framework: every opinion in HCP is operationalized into a testable rule. If a rule can't be tested, it's not in the spec.

## Design center

Three principles anchor HCP. Every layer below answers to these.

**Calibrated.** Every claim ships with confidence and provenance. The human can always tell what's known, inferred, guessed, or asked.

**Actionable.** The human always knows what (if anything) they're being asked to do, by when, with what stakes, and what happens if they ignore it.

**Respectful.** The protocol enforces attention budget, reversibility disclosure, and not crying wolf. The cost of the AI's uncertainty is not paid by the human in clicks.

## Two halves

HCP has two halves, and the hard one is the first.

**The human-declared contract** — a portable, machine-readable description of how a person wants their tools to represent them. What they care about, what counts as important, when they're interruptible, what stakes warrant escalation, what to suppress, what values their tools should embody when acting in their name. This is the half people find hard to write, because we don't know what we want until we're being bombarded with what we don't.

**The AI-emitted communication** — structured messages from AI to human, gated by the contract. The shape is mostly schema. The work is in the gating.

Both halves co-evolve. The human declares what they can articulate; the AI infers from behavior what they can't; the inferred contract is visible and editable in plain language; conflicts surface as questions, never as buttons.

## Four layers

### 1. Profile — the contract

The human-declared (and AI-inferred) communication contract. A profile encodes:

- **Salience signals** — what makes something important to me (named entities, dollar thresholds, anomaly magnitude, project tags, who's involved, recency-of-touch)
- **Attention modes** — states I'm in (deep work, triage, EOD, idle, on-call) and what each admits
- **Aggregation rules** — what's live vs queued for digest vs dropped, and at what cadence
- **Stakes-to-channel mapping** — blockers interrupt; FYI goes to digest; decisions due today push, decisions due this week land at the top of next standup
- **Suppression and decay** — mute X until Y; I no longer care about Z; resolved things drop off
- **Operational values** — principles translated into testable rules ("I value not being manipulated" → "no urgency framing without a real deadline; no auto-escalation of pending items")

Profiles are scoped, cascading, and pin-able.

**Cascading.** Profiles cascade along two axes.

*Within a person:* global → context (e.g., a mode like "deep work" or "sim-racing") → project. Most-specific wins by default. A small set of override-from-above primitives let outer scopes pin non-negotiable rules — e.g., "always block destructive operations on production systems" — that no inner scope may silently relax.

*Across parties:* products carry their own HCP — the operator's contract for how the product should communicate with its users, and what values the product itself represents. When an end-user consumes the product, their personal HCP layers on top of the product HCP. Override semantics are the same machinery: the product HCP defines what users can override and what's pinned. AI integrations inside the product honor both — operator baseline plus consumer overlay.

*Worked example.* `me` → `me when sim-racing` → `my sim-racing product` → `videos produced through the product`. The first two are personal scopes. The third is a product whose HCP encodes the operator's ideals (clean racing, no clickbait, respect for the consumer's focus state during a race) and inherits from the operator's sim-racing context. The fourth is content carrying both the product's ideals and the end-creator's personal HCP. At the integration point — the product, where Claude lives — the consumer's HCP overlays the product's HCP, overriding what the product allows and honoring what it pins. The output (the videos) is a multi-author artifact whose representation is governed by both contracts.

**Privacy.** Personal, project, and product profiles can be private. The AI consuming a private profile may not leak it upward, downward, or sideways. Patterns inferred inside a private scope may not promote elsewhere without explicit human action.

### 2. Emission — what AI sends

Every AI-to-human message is an emission. HCP defines three primitives.

**Statement.** A claim about the world or the work. Required metadata: confidence, provenance (file read, search, inference, memory), scope (what it does and doesn't cover), verifiability (how the human can check without rebuilding the work).

**Request.** An ask for input or decision. Required metadata: the decision being asked, the options, the default if no answer, reversibility, time-sensitivity.

**Update.** A state change in ongoing work. Required metadata: what changed, what's pending, what's blocked, what attention (if any) is required.

Every emission is classified by **ack-cost**.

**Silent.** The AI proceeds; continued human engagement is the acknowledgment. Reversion, restatement, or redirection is the rejection. Most emissions live here.

**Pending.** Flagged for ambient attention; lives in a sidebar, digest, or dashboard until acted on or aged out. No friction unless the human chooses to engage.

**Blocking.** The AI waits for explicit confirmation. Reserved for irreversible-or-expensive decisions. Tools may not escalate ack-cost for engagement.

### 3. Resolution — what reaches the human

How a profile plus an emission decides whether to surface at all, through which channel, at what priority, in what aggregation, with what level of detail. Resolution is deterministic given a profile and an emission, but the profile may include hooks that ask the AI to use judgment in ambiguous cases. Tools must be able to explain *why* they surfaced something when asked — "this matches your salience rule X under attention mode Y."

### 4. Conformance — how tools earn the right to claim compliance

This is the layer that makes HCP a design spec and not just a runtime contract. AI assistants helping build software read the conformance layer and apply it during development. The shipped feature is HCP-shaped even if no LLM is in the runtime path.

Conformance includes required patterns (every notification declares its ack-cost; every claim ships with confidence and provenance; every surfacing decision is explainable), forbidden anti-patterns (named below), reference implementations (libraries that gate emissions against profiles correctly so most tools don't reimplement), and conformance tests (compliant and non-compliant feature behavior).

## Anti-patterns (named, forbidden)

Naming what HCP forbids gives the standard teeth. Claiming HCP compliance means not doing these.

**Engagement-as-metric.** Optimizing for clicks, opens, time-in-app, or any measure that turns human attention into something to maximize. HCP measures whether a human acted on what they decided was important. It does not measure whether the human kept looking.

**Fake urgency.** Escalating ack-cost or using urgency framing without a corresponding real-world deadline or stake. Urgency in HCP must be explainable: "this is urgent because X happens at Y."

**Consent laundering.** UI flows where the easy path is acceptance and the friction is opt-out. HCP-compliant features default to the more restrictive, more private, less attention-consuming option.

**Buttons-everywhere.** Putting the cost of the AI's confidence model on the human via gratuitous explicit confirmation. Acknowledgment is ambient by default; explicit confirmation is expensive currency, spent only when reversibility is poor and stakes are real.

**Shadow engagement.** Covertly tracking behavior to build a profile the user can't see or edit. The inferred contract is always visible to the human, in plain language, and editable.

## The acknowledgment strategy

Acknowledgment is ambient, not gestural. Tools infer ack from behavior — the human kept working, restated, redirected, ignored, returned to the topic, or reverted the change. Buttons are reserved for high-stakes irreversible decisions. The AI's working model of the human's contract is always visible and editable in plain language. Conflicts between the AI's inference and the human's declaration surface as plain-language questions, not modal dialogs.

## What's next

This is v0.1 — settled enough to write down, not settled enough to schema-ize.

Next moves, in priority order: a concrete example profile (a real one, lived-in, with values translated into rules); a concrete example feature designed against HCP, with conformance shown; expansion of the five anti-patterns with examples and counter-examples; wire format and SDK shape, only after the prose spec is stable.

Open for argument.
