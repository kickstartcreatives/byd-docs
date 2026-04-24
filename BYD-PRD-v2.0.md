# Before You Design
## Product Requirements Document
v2.0 · April 2026 · Women Build AI Build-A-Thon

Living working document — updated to reflect aligned planning between strategic and technical leads, and the completed prompt engineering sprint.

---

## 1. Product Overview

### 1.1 Problem Statement

AI design tools have collapsed the time it takes to build a polished website prototype from weeks to an afternoon. A founder can now generate a professional-looking site in under an hour. What these tools cannot do is strategy. A founder with unclear positioning, contradictory messaging, or a muddled business model will now produce a beautiful website that hides those problems rather than exposing them.

Before You Design fills that gap for service providers. It reads what a founder is already putting into the world, names the contradictions and the gold, asks a small number of sharp questions about what's changing, and produces a complete, validated strategic picture — and a highly detailed prompt ready for Claude Design or equivalent AI design tools.

### 1.2 One-Line Description

A tool that analyzes a founder's existing web presence and content, asks what they want to change, and produces a validated brand strategy and an AI-design-tool-ready prompt — so whatever they build next reflects real strategy instead of vibes.

### 1.3 Two Audiences, One Product

The tool serves two audiences with a single flow:

- **Founders of service-based businesses (primary):** Use the tool solo to get clarity before building their new site.
- **Web designers (white-label):** Embed their name and CTA at the output stage to run it as a lead-gen or client intake tool. A founder who completes the flow is a significantly warmer lead than a contact form submission. White-label is scoped to name, tagline, and CTA link — the app's color system does not change per designer.

### 1.4 Positioning

Before You Design owns the thinking layer that sits between "I have an idea" and "I have a website." It is not a brand generator, not a copywriting tool, and not a design tool. It is the strategy step that makes all of those tools produce better output.

---

## 2. Goals and Success Criteria

### 2.1 Hackathon Goals

- Demonstrate a working end-to-end flow: input → analysis → reconciliation → readout → validation → Aha! Moment → output prompt.
- Produce output quality that reads like a human strategist wrote it, not generic AI output.
- Be ready for the optional Shark Tank-style investor pitch on the strength of the business model.

### 2.2 Success Criteria for v1

- **Lean path under 10 minutes:** A founder who pastes a single URL and accepts the default readout can complete the full flow in under 10 minutes. Non-negotiable.
- **Depth path is optional, not default:** Founders who want to paste more content, carefully review each section, and edit with cascade can take 15–25 minutes. The tool rewards depth but does not require it.
- **Stage 1 analysis completes in under 30 seconds.** Sub-30 is a hard target, not aspirational.
- **The strategic readout correctly identifies at least one real gap or contradiction** in the existing brand/site.
- **The output prompt, when pasted into Claude Design, produces a site that is meaningfully more strategic** than what the founder would have generated without it.

### 2.3 Lean-Path Time Budget

| Stage | Lean time | Design requirement |
|---|---|---|
| Stage 0 — Input | ~30 sec | URL field prominent. Paste area optional. No required fields. |
| Stage 1 — Analysis | <30 sec | Single-call preferred. Streaming status messages mandatory. |
| Stage 2 — Questions | ~2 min | Single screen. 3 questions. Q1 required. Q2 and Q3 optional with Skip. Two-line-height text inputs. Next button at bottom. |
| Stage 2.5 — Reconciliation | <10 sec | Silent. Runs during transition to Stage 3. |
| Stage 3 — Readout | ~2–3 min | Skimmable. Bold lead-ins. Key insight per section highlighted. Full detail on expand. |
| Stage 4 — Validate | ~30 sec lean / ~2 min depth | Global "These all look great. Go." button prominent. Per-item checkboxes. Cascade fires once after all edits complete — not after each individual edit. |
| Stage 4.5 — Aha! Moment | ~1 min | Consolidated view. Single edit box. Emotional confirm button. Intentionally slows the user down. |
| Stage 5 — Scaffolding | Silent | Celebratory loading text. Runs between 4.5 and 6. |
| Stage 6 — Output | ~30–60 sec | Prominent copy-to-clipboard. Congratulatory design. Challenges-ahead notes. Designer CTA. |

**Lean-Path Total: ~8–10 minutes**

---

## 3. Product Pipeline Architecture

### 3.1 Pipeline Overview

The pipeline has seven stages. The user actively participates in four of them (Stage 0, Stage 2, Stage 4, Stage 4.5). Everything else happens internally.

The guiding principle: **analyze first, ask second, infer by default, show results fast.**

The pipeline is governed by a shared reference document (`pipeline_shared_reference.md`) that defines the eight analytical dimensions, gap taxonomy, severity scales, controlled vocabularies, and the drift rule. This file is prepended to every LLM prompt at call time.

### 3.2 Stage 1 Performance: Sub-30-Second Target

**Preferred approach: single call with input-aware prompt**

- JSON-only output. No prose. No chain-of-thought.
- Structured field names with controlled vocabularies — constrained fields generate faster than open-ended ones.
- Input-aware: extracts signal from whatever is present; marks missing dimensions as gaps rather than failing.
- Content trimmed before analysis. Target under ~30,000 tokens of input. Trim lower-priority sources in declared order (see Section 3.3, Stage 1). Never trim homepage or about page.
- Few-shot examples in prompt anchor output quality across input shapes.

**Fallback: parallel calls**

If single-call median latency exceeds 25 seconds:
- Call A: Positioning and voice (niche, USP, voice/authority, offer structure). ~10–15 sec.
- Call B: Coherence and gaps (cross-content alignment, contradictions, visual signals, gaps). ~10–15 sec.
- Both fire simultaneously. Total wall-clock = longest single call. JSON merged client-side.

**Streaming status messages are mandatory either way:**

"Reading your content…" → "Finding your voice and positioning…" → "Checking for gaps and contradictions…" → "Ready."

Messages must reflect actual work. No theater.

### 3.3 Stage Details

---

**STAGE 0 — Input Collection**

What the user does: Pastes any combination of content that represents their brand. All input shapes are valid.

Accepted input types:
- Website URL
- Bio or About text
- Social captions
- Blog posts or articles
- Podcast transcripts
- Video transcripts
- YouTube channel content
- Newsletter copy
- Anything they've written or published

UI notes:
- URL field is visually prominent — fastest path, most common input.
- Open paste area below, clearly labeled optional but encouraged.
- No login required. No required fields. Minimum viable input is a URL OR any text.
- UX principle: generous, not interrogative. This is the tool's first impression.

---

**STAGE 1 — Silent Analysis**

What happens: Single comprehensive LLM call analyzes whatever the user provided. Produces structured JSON across eight dimensions. Nothing shown to the user yet.

Prompt: `stage_1_analysis_prompt.md` (v3.2). Prepended with `pipeline_shared_reference.md` at call time.

**The eight analytical dimensions** (defined in shared reference):
1. `niche_and_buyer`
2. `usp_and_differentiation`
3. `offer_structure`
4. `voice_and_authority`
5. `brand_coherence` (cross-cutting)
6. `visual_signals`
7. `origin_story_signal`
8. `business_model_soundness`

**Gap taxonomy** (defined in shared reference):
- **Brand gap:** Enough material to assess a dimension, and it's weak, missing, or contradictory. Real problem with the brand.
- **Input gap:** Not enough material to assess. Data limitation, not a critique.

Both are flagged in `all_gaps_index`. Never conflated.

**Business idea validation:** Stage 1 gently flags fundamentally unviable business models without performing full market analysis. This is a coherence check only — not market sizing.

**Content prioritization when input is large:**
1. Homepage (USP, voice, positioning)
2. About page (origin story, authority posture)
3. Services / offers page (offer structure, pricing logic)
4. Sales page if present
5. Recent blog posts
6. Social captions
7. Navigation and site structure

Output: Structured JSON with `all_gaps_index` — a flat list of every gap across all dimensions, typed and rated. Stage 2.5 iterates this list.

**Minimum viable input:** Under ~200 words, every dimension returns `not_assessable`. Do not fabricate.

**Finding IDs:** Must come from controlled vocabulary in shared reference. Stability across runs is required — Stage 2.5 cross-references by `finding_id`.

Anti-Wrapper Note: The extraction schema, gap taxonomy, inference rules, `finding_id` controlled vocabulary, and `all_gaps_index` roll-up verification are application IP, not replicable by pasting into Claude.ai.

---

**STAGE 2 — What's Changing (single screen)**

What the user does: Answers up to three questions about what's shifting. Single screen. First question is required. Two and three are optional.

UI pattern:
- Single screen (not two screens)
- Two-line-height text inputs — signals "a sentence or two" without feeling like a form
- Red asterisk on Q1 (required). No asterisk on Q2 and Q3.
- Next button at bottom — enabled as soon as Q1 has any content

**Question set (final):**

**Q1 (Required):** Why a new site? Why now?
- Placeholder: *"e.g. I've pivoted from done-for-you to group coaching and my site still sells the old thing — or — Honestly it just looks dated. But also I'm not sure my messaging lands anymore."*

**Q2 (Optional):** How do you want your new site to feel?
- Placeholder: *"e.g. warm, friendly, open, peaceful, professional, polished, lux, clean, inviting"*

**Q3 (Optional):** Is there anything about your current positioning you want to move away from?
- Placeholder: *"e.g. I don't want to be the 'affordable option' anymore — or — I'm tired of the hustle-y, sales-heavy voice. I want something quieter."*

Logic: Q2 and Q3 may be empty. Stage 2.5 handles null answers gracefully — defaults to inference from Stage 1 rather than failing.

---

**STAGE 2.5 — Reconciliation (silent)**

What happens: Silent LLM call runs during the transition from Stage 2 to Stage 3. Takes Stage 1 JSON + Stage 2 answers, reconciles them, and produces structured JSON that Stage 3 consumes. Latency target: under 10 seconds.

Prompt: `stage_2_reconciliation_prompt.md` (v1.2). Prepended with `pipeline_shared_reference.md` at call time.

The reconciliation engine does four things:
1. Classifies each answer's shift type, depth, and alignment with Stage 1.
2. Synthesizes forward intent — what the founder actually said they want, in direct paraphrase.
3. Reconciles intent against Stage 1 — alignments where they match, contradictions where they conflict. Recorded as facts, not verdicts.
4. Routes every Stage 1 gap. **Default: infer.** Ask only for true evidence voids or high-severity contradictions only the founder can resolve.

**Gap routing:**
- `infer` (default) — draw a defensible conclusion from Stage 1 + Stage 2, record the inference in `stage3_routing_rationale`, trust Stage 3 to disclose and validate.
- `ask` — reserved for true evidence voids or major contradictions the founder must resolve. High bar.
- `defer` — low-severity gaps that don't materially change the readout.

**Key output fields:**
- `gap_routing[].stage3_routing_rationale` — most important field. When routing to `infer`, states the specific inference Stage 3 should make. Good: *"Group coaching pivot plus stated repricing supports inferring a single premium-tier ladder."* Bad: *"Direction is clear but structure not specified."*
- `flags_for_stage3.narrative_recommendation` — one sentence, directive not interpretive, tells Stage 3 what to lead with and what to infer.
- `flags_for_stage3.about_page_gap_remains` — flags whether origin story is still unresolved after reconciliation.

**"Where You're Headed" coherence check:** Stage 2.5 verifies that the forward intent maps coherently to Stage 2 answers before handing off to Stage 3. No disconnect allowed.

**Drift rule** (governs Stage 1 and Stage 2.5): Any phrasing that would appear verbatim in the Stage 3 user-facing readout is out of scope. No finished-strategist voice. No coach voice. No metaphors. Structured claims with evidence only.

Anti-Wrapper Note: The cascade dependency map, gap routing logic, and `narrative_recommendation` generation with drift prevention are application architecture, not prompt wrappers.

---

**STAGE 3 — Strategic Readout**

What happens: LLM writes the polished user-facing strategic readout, consuming Stage 1 JSON + Stage 2 raw answers + Stage 2.5 reconciliation JSON. This is the first thing the founder sees. It should feel like a smart strategist who did their homework before the meeting.

Readout sections (labeled, structured):
- **USP:** What the brand actually stands for and why it wins.
- **Niche:** Who this is for, specifically. Flags if too broad.
- **Offer Structure:** What's being sold and whether the ladder is coherent.
- **Voice and Tone:** What the brand sounds like, authority level, market tension.
- **Visual Direction:** Style, mood, and whether visuals support positioning.
- **Origin Story:** Outward Journey, Inner Journey, The Turn — drafted from available material or inferred from pivot narrative. Flagged for validation if inferred.
- **Brand Coherence:** Where what they claim and what they demonstrate line up — and where they don't. Named clearly, not softened. Prioritized by impact.
- **Where You're Headed:** Synthesis of what the new site needs to do differently, based on Stage 2 answers. Must map coherently to what the founder actually said.

**Gap fill logic:** AI infers missing content wherever a defensible inference exists. Asks a targeted follow-up only when inference is genuinely not defensible. The most common missing piece is the origin story — infer from any available Outward Journey or pivot narrative before routing to ask.

**Tone requirement:** Direct without being harsh. Names contradictions clearly but frames them as opportunities, not failures. This is a prompt engineering priority, not a UX afterthought.

---

**STAGE 4 — Review & Confirm**

What the user does: Reviews the strategic picture and responds to each section. Three distinct section states. Global lean-path button. Cascade fires once after all edits complete.

**Global approve button (lean path):** "These all look great. Go. →" — prominent, at the top of the screen. Bypasses all individual review. Total time ~30 seconds.

**Per-item review (depth path):** Each section has a checkbox. Cascade fires once — after the founder completes all edits, not after each individual one. This prevents frustrating wait times mid-session.

**Three section states:**

1. **Correct** — Standard approve checkbox. Edit button available.

2. **Contradiction flagged** — Orange flag badge. Follow-up question: "What's your idea to fix this?" Text input. Skip option available.

3. **Missing info** — Blue badge. Text input to write it in. Three escape buttons: "Make your best guess" / "Use a placeholder" / "I'm not sure."

**Cascade logic (fires on edit, once):**
- USP change → rechecks voice, offer ladder coherence, homepage promise
- Niche change → rechecks ideal client definition, offer positioning, visual direction
- Origin story change → rechecks About↔Home alignment, services page pain validation
- Cascade updates shown with brief explanation of what changed and why
- Fallback if running long: flag stale sections rather than auto-propagate

Anti-Wrapper Note: The cascade dependency map and batch-edit-then-propagate architecture are application logic, not a prompt.

---

**STAGE 4.5 — The Aha! Moment**

What happens: New screen inserted between Stage 4 and Stage 5. Intentionally slows the user down to create an emotional beat of validation and confidence before the output generates.

Content:
- Consolidated view of everything — original analysis plus all corrections the founder just made.
- Corrected sections are visually marked (e.g., "Corrected" badge, orange border).
- Origin story shown in full if generated or inferred.
- Single edit box at bottom: "One last tweak? Type it here and we'll update your strategy."

Confirm button: "Yep, that's amazing. Build my prompt →"

Design goal: The founder should feel that they've just seen themselves clearly — possibly for the first time. This is not a form. It is a moment.

---

**STAGE 5 — Content Scaffolding (silent)**

What happens: Internal page-by-page scaffolding using the confirmed strategic picture as governing constraint. The user sees engaging, celebratory loading text — not a spinner.

Loading text sequence: "Building your strategy…" → "Creating the money machine for you…" → "Almost there…"

Pages scaffolded:
- **Homepage:** Headline derived from The Turn. Hero copy. Benefit statements. Process overview. Social proof placement. CTA.
- **About page:** Outward Journey, Inner Journey, The Turn — structured to anchor to homepage promise and validate services page pain.
- **Services page:** Offer descriptions framed around pain points surfaced in Stage 3. Niche qualifiers. CTA logic.
- **Contact page:** Intake framing consistent with brand voice.

**Coherence check:** Before generating output, final pass: does the About page story connect to the homepage promise? Does the homepage promise connect to what the services page is actually selling?

---

**STAGE 6 — Output**

Primary output: One complete prompt ready to paste into Claude Design. Contains everything the design tool needs to produce a site that converts, not just looks good.

**Prompt contents:**
- Positioning and USP statement
- Ideal client definition with pain points
- Voice and tone rules with examples of what to say and what to avoid
- Visual direction: style, mood, color signals, photography direction
- Page-by-page structure with copy direction, headlines, and CTA logic
- About page: full Outward Journey, Inner Journey, The Turn narrative
- Coherence rules: explicit instructions for how pages reference and reinforce each other

**Screen design:** Highly congratulatory. Copy-to-clipboard button is the primary action. "Show full prompt" expands the preview. Includes a short "challenges ahead" section — a few constructive notes about things to watch for in the design output.

**Designer white-label CTA** (v1 scope): Appears at the bottom of Stage 6 only. Designer name, tagline, and "Book a call" button. The app's visual design does not change per designer — only this component is personalized.

---

## 4. Role Division

| Area | Owner |
|---|---|
| Stage 1 prompt authorship | Diane (strategic lead) |
| Stage 2.5 prompt authorship | Diane (strategic lead) |
| Stage 3 readout tone and prompt | Diane (strategic lead) |
| Stage 6 output prompt structure | Diane (strategic lead) |
| Daily output quality review | Diane (strategic lead) |
| Copy pass across all screens | Diane (strategic lead) |
| Next.js scaffolding and app architecture | Barb (technical lead) |
| Supabase schema and persistence | Barb (technical lead) |
| Anthropic API integration | Barb (technical lead) |
| URL fetching (Firecrawl) | Barb (technical lead) |
| Vercel deployment | Barb (technical lead) |
| Cascade dependency mapping | Barb (technical lead) |
| UI implementation | Barb (technical lead) |
| Prompt iteration and testing | Shared |
| End-of-day checkpoint review | Shared |
| Daily sync (WhatsApp + Zoom) | Shared |

---

## 5. Prompt Engineering Architecture

### 5.1 Prompt Files

| File | Version | Purpose |
|---|---|---|
| `pipeline_shared_reference.md` | 1.0 | Single source of truth for dimensions, vocabularies, gap taxonomy, severity scales, drift rule. Prepended to every LLM call at runtime. |
| `stage_1_analysis_prompt.md` | 3.2 | Analyzes founder input. Produces eight-dimension JSON with `all_gaps_index`. |
| `stage_2_reconciliation_prompt.md` | 1.2 | Reconciles Stage 2 answers against Stage 1 JSON. Routes gaps. Produces structured input for Stage 3. |
| Stage 3 readout prompt | TBD | Writes the user-facing strategic readout. Only stage that produces prose — all others produce JSON. |
| Stage 6 output prompt | TBD | Generates the Claude Design–ready output prompt from validated strategy + scaffolding. |

### 5.2 Prompt Governance

- Every prompt receives `pipeline_shared_reference.md` as a preamble at call time.
- `finding_id` values in Stage 1 must come from the controlled vocabulary in the shared reference. Stability across runs is required — Stage 2.5 cross-references by `finding_id`.
- The drift rule governs all stages except Stage 3: no finished-strategist voice, no metaphors, no coach voice in JSON-producing stages.
- All changes to any prompt or the shared reference must pass the relevant tests in `pipeline_test_protocols.md` before merging.

### 5.3 Testing

Test protocols are defined in `pipeline_test_protocols.md`. Key tests:

- **Stage 2.5 `narrative_recommendation` test** — Three input shapes (aesthetic-only, committed shift, contradiction). Run before any Stage 2.5 prompt change.
- **Stage 1 `finding_id` stability test** — Same input, three runs, verify IDs are byte-identical across runs.
- **Golden I/O regression pairs** — Baseline inputs/outputs captured before the build. All prompt changes must reproduce within tolerance.

---

## 6. Scope

### 6.1 In Scope for v1

- URL fetch and analysis
- Paste-in content analysis (social captions, blog posts, articles, video transcripts, any text)
- Single comprehensive analysis pass with structured JSON output across eight dimensions
- Business idea validation (coherence check, not market sizing)
- Stage 2.5 reconciliation — silent, runs in under 10 seconds
- Single-screen "what's changing" form (3 questions, Q1 required)
- Strategic readout with labeled sections
- Per-item approval with three states (correct / contradiction / missing)
- Global one-click approve (lean path)
- Cascade edit logic — fires once after all edits complete
- Aha! Moment screen with consolidated view and single edit box
- Celebratory loading text on Stage 5
- Internal content scaffolding (page-by-page, not user-facing)
- Claude Design–ready output prompt
- Challenges-ahead notes on output screen
- Designer white-label CTA at Stage 6 (name, tagline, link)

### 6.2 Deferred to v2

- Direct social media API integration / scraping
- PDF upload for existing brand guides
- User accounts and authentication
- True multi-turn conversational analysis
- Advanced designer customization (interface theming, analytics, paid tier)
- Full white-label color system per designer
- Visual brand prompt generation for image tools (Midjourney, etc.)
- Slug-based shareable URLs for designer white-label

---

## 7. Technical Approach

### 7.1 Build Method

Vibe-coded build using Claude Code as the primary builder. No manual coding. Claude API is the intelligence layer throughout.

### 7.2 AI Layer

Latest Claude model via Anthropic API. Confirm model string in Anthropic docs on Day 1 — do not hardcode an older version. Stage 1 is the core IP and primary prompt engineering priority. Output quality — strategic writing that reads like a human strategist — is the product's key differentiator.

### 7.3 Key Technical Decisions

| Decision | Resolution |
|---|---|
| Stack | Next.js (App Router) + Tailwind + Supabase + Vercel + Anthropic API. Claude Code as builder. |
| URL fetching | Firecrawl (primary) for JS-rendered pages. Simple fetch + Cheerio as fallback for static sites. Confirm Firecrawl handles Squarespace on Day 1. Fallback: manual paste-in. |
| Analysis output | Structured JSON across eight dimensions with controlled vocabularies. Not prose. Feeds all downstream stages. |
| Cascade logic | Batch edit: fires once after all edits complete. Dependency map declared at schema level. Simplified fallback: flag stale sections rather than auto-propagate if running long. |
| Persistence | Supabase. Minimal schema: sessions table (slug, designer_config, analysis_result, created_at). No founder accounts. |
| Auth | None in v1. |
| Streaming | Streaming API responses with incremental status messages. Stage 1: ~30 seconds. Stage 5: celebratory loading text. Both mandatory. |
| Prompt architecture | `pipeline_shared_reference.md` prepended to every LLM call at runtime. Stage-specific prompts are self-contained beyond that. |
| Config | Brand values (name, CTA copy, CTA link) stored in a single config object from day one — never hardcoded. White-label in v2 is then a config swap, not a refactor. |

---

## 8. Risks and Mitigations

| Risk | Mitigation |
|---|---|
| Scope creep | Pipeline is locked. All deferred features documented. No additions during build without explicit decision. |
| Tone calibration | Readout must be direct without being harsh. Drift rule governs all JSON stages. Stage 3 tone is a prompt engineering priority, not a polish task. |
| Analysis quality | Stage 1 is the make-or-break moment. Strategic lead reviews AI output daily starting Day 2. |
| `finding_id` drift | Controlled vocabulary in shared reference locks IDs. Stability test in `pipeline_test_protocols.md` catches drift before it breaks Stage 2.5 cross-references. |
| Stage 1 latency miss | Single-call preferred. Parallel-call fallback confirmed during Day 1 spike if needed. |
| Variable input quality | Stage 1 explicitly handles all input shapes — URL only, bio only, social only, mixed. Minimum viable input returns `not_assessable` everywhere rather than fabricating. |
| Cascade logic running long | Batch-edit approach (fires once after all edits) reduces cascade complexity. Simplified fallback: flag stale sections if implementation runs past 6pm Sunday. |
| Lean path exceeding 10 min | If Day 7 testing shows lean path over 10 min: cut Stage 2 to Q1 only, make Stage 4 one-click by default, trim Stage 3 readout. Do not sacrifice Stage 1 quality. |
| Narrative recommendation drift | `narrative_recommendation` test in `pipeline_test_protocols.md` must run before any Stage 2.5 prompt change. Three test shapes with pass/fail criteria defined. |
| Aha! Moment feeling like a form | Single edit box only. No section re-review. Confirm button copy is emotional, not functional. Design goal: revelation, not administration. |

---

## 9. Build Plan

Event runs April 24 – May 1, 2026. Submission deadline: May 1 at 3:00 PM EST.

### 9.1 Capacity Model

6-day effective build. Weekend carries the weight.

- **Fri Apr 24 (Kickoff):** 2–3 hrs evening. Scaffold only.
- **Sat–Sun Apr 25–26 (Weekend):** 8–10 hrs each day. Heavy build. Both leads fully available.
- **Mon–Tue Apr 27–28:** 2–3 hrs evening each. Medium scope.
- **Wed Apr 29:** 2–3 hrs evening. Lighter scope.
- **Thu Apr 30 (Polish):** 2–3 hrs evening. No new features. Bug fixes, copy pass, demo prep only.
- **Fri May 1 (Submission):** Morning only. Submit by 2:00 PM EST.

### 9.2 Day-by-Day Plan

**Day 1 — Fri Apr 24 (Scaffold only)**
- Scaffold Next.js + Tailwind + Supabase client
- Wire Firecrawl to URL input form. Confirm scraping returns clean output.
- Confirm Claude model string before hardcoding
- EOD: URL input renders, Firecrawl returns content, deployed to Vercel staging. No AI calls yet.
- Diane: Identify 10 test sites spanning input shapes. Clarify cascade dependency map.

**Day 2 — Sat Apr 25 (Stage 1 + Stage 3)**
- Connect Anthropic API. Run Stage 1 against 3 real URLs. Review together on Zoom.
- Iterate Stage 1 until output quality passes strategic lead's review. Core IP day.
- Build Stage 0 input UI (URL + paste area)
- Build Stage 3 readout UI with labeled sections
- Wire streaming status messages to Stage 1 call
- EOD: paste URL → see a real strategic readout with accurate loading states.

**Day 3 — Sun Apr 26 (Stage 2 + Stage 2.5 + Stage 4 + Aha! Moment)**
- Build Stage 2 single-screen form (3 questions, Q1 required)
- Build and wire Stage 2.5 reconciliation call
- Build Stage 4 three-state review UI (correct / contradiction / missing)
- Implement cascade — batch fire after all edits complete
- Build Stage 4.5 Aha! Moment screen
- Cascade fallback if bogging down by 6pm: flag stale sections, do not auto-propagate
- EOD: full flow input → readout → validate → Aha! Moment working end-to-end.

**Day 4 — Mon Apr 27 (Stage 5 scaffolding)**
- Stage 5 internal content scaffolding (page-by-page)
- Coherence check pass — About ↔ Home ↔ Services
- Strategic lead reviews scaffolding quality on 2 test cases
- EOD: Stages 0–5 work end-to-end.

**Day 5 — Tue Apr 28 (Stage 6 output)**
- Stage 6 output prompt generator
- Output page UI — copyable prompt, challenges-ahead notes, designer CTA
- Strategic lead paste-tests output into Claude Design. Make-or-break test.
- EOD: full founder flow works end-to-end.
- ⚠ No new feature work after tonight.

**Day 6 — Wed Apr 29 (Testing + designer CTA)**
- Wire designer white-label CTA component (name, tagline, link)
- End-to-end testing day. Run golden I/O regression tests.
- Fix anything that breaks the lean path.

**Day 7 — Thu Apr 30 (Polish)**
- Error states — failed scrape, failed API call, invalid URL, empty input
- Mobile responsiveness pass
- Copy pass — every label, button, loading message sounds like the product
- Test with 3–5 real users. Time lean-path completion. Over 10 min = design problem, not a polish issue.
- Strategic lead output quality review on 10 diverse founder URLs
- Final Vercel deployment
- EOD: working, deployed, tested. Logic frozen.

**Day 8 — Fri May 1 (Submission)**
- Demo recording
- Shark Tank pitch prep — business model story, dual-audience framing
- Final smoke test on 3 fresh URLs. No logic changes.
- **SUBMIT BY 2:00 PM EST.**

---

## 10. Open Items

- Stage 3 readout prompt: to be authored by strategic lead and tested against Stage 2.5 output before Day 2.
- Stage 6 output prompt: to be authored and tested before Day 5.
- Golden I/O regression pairs: to be captured on Day 2 after Stage 1 prompt is locked.
- Firecrawl plan B: finalized after Day 1 spike. Fallback is manual paste-in.
- Claude Design output format: findings from pre-hackathon research feed Stage 6 prompt structure.

---

## 11. The Honest MVP

This is a 6-day effective build. The weekend carries the weight — Stages 1, 2, 2.5, 3, 4, and 4.5 all land by Sunday April 26. Weekday evenings are for integration and finishing. Polish day is non-negotiable. Submission day is not a build day.

The make-or-break factors are prompt quality and the Aha! Moment. Strategic lead reviews AI output every single day starting Day 2. If the output reads like a chatbot, the product fails regardless of the engineering. If the Aha! Moment feels like a form, the product fails regardless of the analysis quality.

— End of PRD —
