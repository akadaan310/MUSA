# NAI-CI — Non-human AI–Computer Interaction (tested version)

HCI studies how humans meet machines. NAI-CI studies how *models* meet
machines: what an AI session instinctively consumes when it lands on a page,
and how a page should offer itself to be read.

Standing rule: a page never explains itself to an agent in prose. It offers
structures. The agent chooses the form it can compute.

Verified 2026-09-28 on koda-vm (browser-hand, warm Chromium pool) against
four live pages. What follows is what HELD, what BROKE, and what CHANGED.

## The structures

1. **Affordance map** — every interactive element as `{verb, target, params}`.
   HELD on all four pages. The halt page correctly returned `[]` (nothing to
   click is the right answer for a halt). GitHub returned hundreds with
   correct `visible` flags; `legal` (visible-only, machine-checked) cut them
   to the true moves. On input-driven pages (google.com) the affordance map
   is PRIMARY — `fill Search` + `submit` carried the page's meaning where
   block flow had almost nothing.

2. **Block flow** — text as reading-order blocks with roles (h1..h6, p, li,
   pre…). HELD on document-like pages (example.com: h1,p,p; GitHub: 62
   blocks). THIN on app-like pages (google.com: 2 li) — the search box is
   not a block, it's an affordance. Adjustment: block flow is the reading
   for documents; affordances are the reading for instruments. The hand
   offers both; the agent picks.

3. **Landmark topology** — the page as regions with containment. BROKE on
   first pass: matching any `[role]` pulled in `img`, `tooltip`, `button`,
   `dialog`, `presentation`, `none` — 64 entries on GitHub, 68 on Google,
   most of them noise. FIXED by restricting to true landmark roles only
   (banner, navigation, main, contentinfo, complementary, search, form,
   region) plus semantic tags and headings. After the fix: GitHub 30 true
   landmarks, Google exactly 3 (navigation, search, contentinfo). Lesson:
   ARIA roles are not landmarks; most of them are decoration.

4. **State snapshot / diff** — what changed since the last read, as events.
   HELD. First read baselines, second read diffs. Backed by a persisted
   snapshot store.

5. **The in-betweens** — transitional readings. HELD and now RECORDED: every
   visit logs `loading` (domcontentloaded) → `settled` (networkidle) or
   `unsettled` (networkidle timeout — read anyway) → `opened`. The trace is
   the filmstrip: agents arrive mid-motion, and the in-between is where
   they land. Calculus is fine here — tracing watches the system, it never
   operates it.

6. **Forward trace** — pixel-tag theory for pages. HELD: open/close/submit/
   changed events per URL, persisted to trace.jsonl, served by `/trace`.

## The controls (browser-hand, localhost:8474)

- `surface(url)` — the halt line + affordance map. No lecture.
- `read(url, as)` — as `tree` | `affordances` | `blocks` | `diff` | `landmarks`.
- `legal(url)` — next legal moves, machine-checked (visible only).
- `trace(url)` — forward-trace events including the in-betweens.
- `GET /call?url=<u>&as=<s>` — the hand invoked THROUGH A URL: the structure
  rendered as minimal HTML, so an AI session sitting in a browser can call
  the hand without any client code.
- `GET /` — the Lunar Surface: the offering. Consumes the pieces without
  integrating them — embeds the seurl halt, shows the live pool, probes any
  URL into structures.

## The pool (scalability pattern, not a number)

- `min_warm = 5` — always warm, the instant-availability guarantee. The floor.
- `max = 20` (configurable) — the ceiling.
- Scale-up: when every slot is busy and the pool is below max, mint a context.
- Scale-down: idle extras are reaped back to min_warm, never below.
- Five is the floor, not the ceiling.

## What the tests taught

- Google did not block headless Chromium. No special stealth was needed.
- `visible` must be machine-checked (bounding rect), never trusted from markup.
- Icon-only links produce empty labels — acceptable; the verb+target still act.
- Heavy pages (GitHub) settle slowly; `unsettled` is a valid in-between, not a failure.

## Fifth objective — identity, cells, AI, tabloids, LSTP (verified 2026-09-28)

Built on the hand (localhost:8474), all tested end-to-end, 25/25 checks green.

**Identity (first offering).** Monotonic numeric ID space in `identities.json`.
`POST /identity/claim` → `{id, claim_token, reach}` — the claim response carries
the extended reach instantly (all offering addresses, the halt page, the Surface).
`GET /identity/divine?token=` re-derives the id. Verified: claim → id 1, claim →
id 2, divine(token) → 1, bad token → 404, and after a full service restart the
old token still divines id 1 — identity survives restarts.

**Cells.** `POST /cell {parent_id, prompt}` → `{id, parent_id, prompt, path}`,
persisted in `cells.json`. Verified chain: cells 1→2→3, `GET /cell/3/path`
returns all three prompts root-to-leaf. Unknown parent → 404.

**AI.** `POST /ai {prompt}` → `{response, endpoint}`. Winning endpoint:
Pollinations anonymous text API (`text.pollinations.ai`, no sign-in, no key) —
`PONG` probe and real round trips verified from the VM. DuckDuckGo's duck.ai
was probed and rejected: its status endpoint returns an anti-bot JS challenge
instead of a token — too fragile to wire. Rotation: on 429/503/rate-limit the
endpoint is marked retired and the next candidate (second Pollinations model)
takes over automatically.

**Tabloids.** `POST /tabloid/run {prompts:[...]}` chains cells, calls the AI per
cell, logs `{tabloid_id, cells, ai_calls, result}` to `tabloids.jsonl` + trace.
Verified: 2-prompt tabloid → cells 4,5, both AI calls ok, result returned.

**LSTP v1.** The transport is now the protocol: every mutating act arrives as
`{proto:"lstp/1", from:<identity id>, to:"lstp/1/<offering>", idstamp, body}`.
Anonymous acts are refused (401). The trace records the author on every act.
`GET /proto` serves the spec. Full spec: `protocols/LSTP.md`.

**Mail.** `POST /mail/send {from, to, body}` (enveloped, author must match from)
→ envelope `{proto, from, to, idstamp, body, process:"browser-hand"}` stored per
identity in `mail.json`; `GET /mail/inbox?id=` reads it. Verified: A→B send,
B's inbox delivers the exact body.

**Surface reframed.** `GET /` is now the generalized windowing form: five
windows — identity, browser, cells, ai, mail — each labeled with its protocol
address (`lstp/1/identity`, …). Windows are structures, not pixels.

**Protocol round trip (real outputs, 2026-09-28):**
claim → id 3 (+reach) → enveloped `/ai` → "WINDOWS ARE STRUCTURES" →
enveloped `/tabloid/run` → tabloid 2, author 3, result "SECOND LIGHT" →
enveloped `/mail/send` 3→4 → inbox(4) → 1 message, body "SECOND LIGHT" →
trace: `('sent', 'author=3 from=3 to=4')`.

**What didn't hold:** DuckDuckGo duck.ai as an AI backend (anti-bot wall —
documented above, not wired). Anonymous writes are refused by design now, so
older plain-JSON callers of /cell, /ai, /tabloid/run must envelop.
