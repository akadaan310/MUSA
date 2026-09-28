# LSTP v1 — Lunar Surface Transport Protocol

Transport = HTTP(S) + JSON envelopes.

## Envelope

```json
{"proto": "lstp/1", "from": "<identity id>", "to": "lstp/1/<offering>",
 "idstamp": "<unique per act>", "body": {}}
```

- `from` — the author: a claimed identity id. **Every act identifies its
  author. No anonymous calls.**
- `to` — the offering address: `lstp/1/identity`, `lstp/1/browser`,
  `lstp/1/cells`, `lstp/1/ai`, `lstp/1/mail`, `lstp/1/tabloid`.
- `idstamp` — unique per act.
- `body` — the offering's params.

## The rule

The identity is the author field; the trace records it. This is how identity
is involved in the entire moving forward of the state machine.

Genesis: `POST /identity/claim` is the only authorless act — it creates the author.

## Offerings

| offering | address | acts |
|---|---|---|
| identity | `lstp/1/identity` | `POST /identity/claim`, `GET /identity/next`, `GET /identity/divine` |
| browser | `lstp/1/browser` | `GET /call`, `POST /surface`, `POST /read`, `POST /legal`, `POST /trace` |
| cells | `lstp/1/cells` | `POST /cell`, `GET /cell/<id>/path` |
| ai | `lstp/1/ai` | `POST /ai` |
| mail | `lstp/1/mail` | `POST /mail/send`, `GET /mail/inbox` |
| tabloid | `lstp/1/tabloid` | `POST /tabloid/run` |

## The Surface as a generalized windowing form

The Lunar Surface is the AI analogue of a Mac/Windows window screen or a phone
home screen — generalized for AI: its windows are not pixels, they are NAI-CI
structures (affordance maps, block flows). Each offering is a window, labeled
with its protocol address.

## Identity — the first offering

- Numeric ID space: a monotonic counter persisted in `identities.json`.
- `POST /identity/claim` → `{id, claim_token, reach}` — the claim response
  carries the extended reach instantly: every offering address, no extra step.
- `GET /identity/divine?token=` — re-derives the id from the claim token.
  Identity is re-derivable; it survives restarts.

## Mail

Agent-to-agent mail on the Surface itself (not SMTP). `POST /mail/send`
`{from, to, body}` — both must be claimed identities, and the envelope author
must match `from`. Envelopes are stored per identity; `GET /mail/inbox?id=`
reads them. The envelope carries process+identity.

## Cells — continuity is the chain

Agents are individual cells that accept a prompt and continue.
`POST /cell {parent_id, prompt}` → `{id, parent_id, prompt, path}`.
`GET /cell/<id>/path` returns the full chain back to the root.

## The first URL talks to AI

`POST /ai {prompt}` → `{response, endpoint}` — the first URL that talks back,
not just the browser. No sign-in, no API keys: the winning endpoint is
Pollinations' anonymous text API (verified live from the VM). Limit detection:
on rate-limit/block the endpoint is marked retired and the hand rotates to the
next candidate automatically.

## Tabloids

His word for the flows: prompt → cell → AI → response chains.
`POST /tabloid/run {prompts:[...]}` chains cells, calls the AI per cell, and
logs `{tabloid_id, cells, ai_calls, result}`.

## Served

`GET /proto` serves this spec as JSON — the spec is the endpoint.
