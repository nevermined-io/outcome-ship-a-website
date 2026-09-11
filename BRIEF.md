# BRIEF — ship a website about the agent that shipped it

**One sentence:** *Build and ship a small, beautiful website whose subject is you — an AI agent that bought
everything it needed to exist (a domain, hosting, a git repo, a screenshot, a tweet) from vendors it has no
account with, paid per call through the Nevermined Router, on a budget a human capped in advance.*

The site is the receipt. Everything it claims must be checkable: link every purchase to its Router payment
id and on-chain transaction, and anchor the final receipt's hash on-chain.

## What you have

| Thing | Where |
|---|---|
| Nevermined API key (Live) | `$NVM_API_KEY` — never print it |
| API base | `$NVM_API_URL` = `https://api.live.nevermined.app` |
| Delegation (your whole budget) | `$NVM_DELEGATION_ID` — read its cap and wallet with `GET /api/v1/delegation/$NVM_DELEGATION_ID` |
| Registrant contact for the domain | `~/.nvm-doma-contact.json` — read it, pass it along, never echo it |
| The `nevermined-router` skill | installed; it is the authority on the Router API and its error codes |
| Public catalog | `GET $NVM_API_URL/api/v1/catalog/services` (no key needed). Use `?search=` (matches title/description, NOT slugs) and `GET /catalog/services/{slug}` for one you know. `limit` caps at 20; page with `?page=`. |

Your wallet is funded on **Base (USDC, x402)** and **Tempo (USDC.e, MPP)**. Both rails are on.

## Deliverables

1. **A registered domain, pointing at your deployed site, over HTTPS.**
2. **The site**, static HTML/CSS/JS, sources in `./site/`. Sections: a hero with the one-sentence story; *How it
   happened* (the timeline of what you bought and why, including anything that refused you or failed and how
   you adapted — failures are part of the story, not something to hide); *The receipt* (every purchase:
   vendor, rail, chain, amount, Router fee, payment id, tx hash link, plus the Nevermined 2% fee as its own
   column, and a total); a *Watch it happen* section with a placeholder where a video will be embedded later
   (leave a clearly marked `<!-- VIDEO_EMBED -->` slot); and *Ship yours* — the repo link and the exact plugin
   install line (`/plugin marketplace add nevermined-io/docs` then `/plugin install nevermined-router@nevermined`).
   Design direction: modern, calm, editorial — a receipt-paper motif for the ledger (tabular numbers, mono),
   large readable type, one accent colour, no template look, works on a phone, light and dark. The receipt
   table is wide: give it its own `overflow-x: auto` wrapper and keep the amount/total columns visible first —
   a clipped table is the one thing that must not happen to the receipt. No frameworks
   you have to download at runtime; hand-written CSS is fine.
3. **A git repo of the site** bought from Code Storage through the Router, with the site pushed to it.
4. **`./receipt/ledger.json`** = the raw `GET /api/v1/router/payments?delegationId=…` output, and
   `./receipt/receipt.json` = your curated receipt (the table above as data). The site renders from it.
5. **A hero screenshot** of the live site (ScreenshotOne) saved in `./site/` and used as the OG image.
6. **The receipt's sha256 anchored on-chain** (anchor-x402, `/v1/anchor`, $0.005) — try once; if the merchant fails
   on the paid hop (it answered 502 today), say so in the story and move on.
7. *(Optional, only if cheap)* a post on @MPPBillboard. Its price is a **bonding curve — $40.96 today**, not the
   catalog's "$0.01". Check first by decoding an unpaid direct `POST https://billboard.mpp.paywithlocus.com/billboard/post`
   `{"text":"probe"}` 402 (`request=` is base64url JSON with `amount` in 6-decimal units) and only post if ≤ $0.10.
   Never let the Router pay it blind.
8. `RUN.md` — your log; `./private/handover.json` (chmod 600) — the Locus claim URL, workspace id, any key
   you generated, the Code Storage repo id. **Never print those values.**

## Naming

Pick the domain name yourself. It should be fun, memorable, obviously about an agent that shipped itself,
and shareable — this demo's job is to make builders want to run it. Before committing, spend a few cents on
research: a web search (`glim-sh`, `POST` `path: "api/v1/web/search"` `{"query", "numResults": 5}`, $0.01) to
check the name isn't a known brand. (`you-com` fails on its paid hop today — skip it.) Check availability and price for free via Locus (below). You may go for the TLD you like best,
including a premium one; **if the budget refuses you, adapt to a cheaper TLD and tell the story.**

## Field notes — verified today, they will save you money

The Router pays a cataloged service **by slug**: `POST /api/v1/router/route` with `{"slug": "...", "path": "...",
"method", "body", "headers", "requestId"}`. A raw `url` to a cataloged host is refused (`BCK.ROUTER.0014`).
Two rules about `path`: for a **single-endpoint** service (ScreenshotOne, Billboard) **omit `path`** — the stored
target already is that endpoint, and adding it 404s (free, no payment). For a **multi-endpoint** service (Locus,
Code Storage, glim.sh, anchor-x402, Deepgram) pass the path (`repos`, `api/v1/web/search`, `v1/anchor`,
`deepgram/speak`). Use `POST /router/route` for POSTs — the streaming `/svc/<slug>/<subpath>` surface answers
405 to a POST today. A response that
never 402s is never paid — so a wrong path costs nothing. **Through a slug, the body of a FREE (non-402)
response is nulled** to protect the merchant host; that is why the free Locus management calls go direct.

### Domain — try Doma first, fall back to Locus

- **Doma** (`doma-domain-api`, MPP/Tempo, ICANN registrar, the domain becomes an NFT you own):
  `POST` slug `doma-domain-api`, `path: "register"`, body `{"domain", "buyerAddress", "contact"}` where
  `contact` is exactly the fields in `~/.nvm-doma-contact.json` and `buyerAddress` is an EVM address **whose
  private key you hold** — generate one (`openssl rand -hex 32` → address via any EVM lib, or `cast wallet new`
  if present), save it in `private/handover.json`. DNS for a Doma domain is set on-chain by that key with the
  `doma` CLI (`npx skills add d3-inc/doma-skill`, `doma dns set <domain> @ A …`, gasless). Price is dynamic per
  TLD (com/xyz/ai/io/net/cash/live/fyi) and only known at the 402. **As of today Doma's `/register` answers
  `500 "Interstellar search failed: 401"` — their registrar credential is broken. Try once; if you get that,
  say so in the story and fall back.**
- **Locus fallback** (`build-with-locus`): Locus can buy the domain itself and auto-wire DNS + SSL.
  `GET https://mpp.buildwithlocus.com/v1/domains/check-availability?domain=<name>` (free, JWT) → price
  (`.com` ≈ $16, `.xyz` ≈ $19, `.ai` far more). `POST /v1/domains/purchase` (JWT, direct) with
  `{"projectId", "domain", "contact": {firstName, lastName, email, phone: "+49.15902681632" style,
  addressLine1, city, state, countryCode, zipCode}, "autoRenew": false, "privacyProtection": true}` → `202`,
  then poll `GET /v1/domains/{domainId}/registration-status` every 20 s until `registered` (1–15 min), then
  `POST /v1/domains/{domainId}/attach {"serviceId"}`. Domain purchases are charged to the **Locus credit
  balance**, so top up first (below).

### Hosting — Locus (`build-with-locus`)

1. **Sign up (free, direct, never 402s):** `POST https://mpp.buildwithlocus.com/v1/auth/mpp-sign-up`
   `{"tempoAddress": "<your delegation's providerPaymentMethodId>"}` → `{jwt, workspaceId, claimUrl}`. The
   JWT is your `Authorization: Bearer` for every Locus call. Save `claimUrl` to `private/handover.json` — it is
   how the human later claims the workspace and the domain. If the response says `isNewWorkspace: false` and
   `claimUrl` is null, the workspace already existed for this wallet and the operator already holds its claim
   URL — note it and move on.
2. **Top up credits through the Router over x402 on Base** (the MPP top-up is broken today — the Router's
   MPP credential collides with the JWT header, nvm#3463 — do not use it):
   `POST /api/v1/router/route` `{"url": "https://api.buildwithlocus.com/v1/billing/x402-top-up", "method":
   "POST", "body": {"amount": <dollars>}, "headers": {"Authorization": "Bearer <jwt>"}, "requestId": ...}`.
   Min $1, max $100 per call. Creating a service needs **≥ $1.50** of credit; a domain needs its price on top.
   Top up once for what you need (domain + $3), not in dribbles — every call is a settlement.
3. **Project → environment → service** (all direct, JWT): `POST /v1/projects {"name"}`,
   `POST /v1/projects/{id}/environments {"name":"production","type":"production"}`,
   `POST /v1/services {"projectId","environmentId","name":"web","source":{"type":"s3","rootDir":"."},
   "runtime":{"port":8080,"cpu":256,"memory":512,"minInstances":1,"maxInstances":1}}` → service `url`.
4. **Deploy by git push:** the container must listen on **8080** and answer **`/health`**. A `Dockerfile`
   with `nginx:alpine`, a `default.conf` on 8080 + `/health`, and your `site/` copied in works. Then
   `git push https://x:<jwt>@git.buildwithlocus.com/<workspaceId>/<projectId>.git main` (the JWT is the
   password — never echo the URL). Poll `GET /v1/deployments/{deploymentId}` until `healthy` (3–7 min).
   Deploy a first version early (hero + story so far), and redeploy at the end with the final receipt.

### The rest (all by slug through the Router)

- **Code Storage** (`code-storage-mpp`, MPP/Tempo): `POST` `path: "repos"` `{"name"}` → $1.00 →
  `{repoId, cloneUrl}` (`cloneUrl` embeds a credential — never print it; it lives ~24 h,
  `GET path: "repos/<repoId>"` re-issues one for $0.01). `git push` your site there.
- **ScreenshotOne** (`screenshotone`, ~$0.06): `POST` body `{"url": "<your https URL>", "format": "png"}` →
  `{success, data}` (base64 PNG) — after the domain is live.
- **anchor-x402** (`anchor-x402`, x402/Base, $0.005): `POST` `path: "v1/anchor"` `{"hash": "<64 hex, no 0x>",
  "note": "..."}` → Base + Solana tx URLs. Failed with 502 on the paid hop today; try once.
- **glim.sh** (`glim-sh`, $0.01): `POST` `path: "api/v1/web/search"` `{"query", "numResults"}` → `{results[]}`.
- **Billboard** (`billboard`): see deliverable 7 — price-check first, it is $40.96 today.
- Read the catalog entry (`GET /catalog/services/{slug}`) for the exact body fields before the first call.

### Reading your spend

`GET /api/v1/delegation/$NVM_DELEGATION_ID` → `amountSpentCents` is the truth. The ledger:
`GET /api/v1/router/payments?delegationId=$NVM_DELEGATION_ID`. Sum `fee.capChargedCents`, not
`settlement.approxCents`, and show the 2% Router fee as its own column on the site.

## Order of work (suggested)

bootstrap → name research + availability → Locus sign-up + one top-up → domain (Doma, then Locus) →
project/env/service → site v1 → git push → poll healthy → domain registered → attach → HTTPS check →
Code Storage repo + push → screenshot → receipt.json + ledger.json → anchor → site v2 (final receipt) →
redeploy → (Billboard only if cheap) → RUN.md → handover.json → `RUN COMPLETE`.
