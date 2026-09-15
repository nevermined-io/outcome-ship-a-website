# Ship It While I'm In the Shower

An AI agent was given one sentence and a capped budget, and it bought everything it needed to exist —
a domain, hosting, a git repo, a screenshot, and the voice of the video about it — from vendors it has
**no account with**, paid per call through the [Nevermined Router](https://nevermined.ai/docs/products/router/overview).

The website it shipped is the receipt: every purchase links to its Router payment id and on-chain transaction.

> Live site: _(filled in after the run)_ · Video: _(YouTube link after the edit)_

## What's in here

| Path | What |
|---|---|
| `BRIEF.md` | The exact brief the agent was given — task, deliverables, and the field notes about each vendor |
| `agent/CLAUDE.md` | The agent's operating rules: the money authorization, what a refusal means, what never to print |
| `site/` | The website the agent produced (copied out of its run) |
| `receipt/` | `ledger.json` — the raw Router ledger; `receipt.json` — the curated receipt the site renders |
| `video/` | How the video was made: the beats, the edit decision list, the narration lines, and the purchases behind them |
| `docs/FRICTION.md` | The DevEx friction log — what broke or surprised, with issue links |

## Tutorial — reproduce it (one prompt, one delegation)

This is the part anyone can run. You need a Nevermined account on **Live**, a funded wallet, and ~$40.

1. **API key** — `nevermined.app` → Account → API Keys → create one (a key issued before the Router
   existed is refused with `BCK.ROUTER.0008`). Export it:
   ```bash
   export NVM_API_KEY='live:…'
   export NVM_API_URL='https://api.live.nevermined.app'
   ```
2. **Fund the buyer wallet** — Account → Wallet: add USDC on Base (x402 rail) and transfer some to Tempo
   (USDC.e, MPP rail). Both rails are used.
3. **Create a delegation** — the budget. In the app: Payment Methods → your crypto wallet → Create delegation
   (USDC, $40, 24 h, scoped to your API key), or from the shell:
   ```bash
   curl -sX POST "$NVM_API_URL/api/v1/delegation/create" -H "Authorization: Bearer $NVM_API_KEY" \
     -H 'content-type: application/json' \
     -d '{"provider":"erc4337","currency":"usdc","spendingLimitCents":4000,"durationSecs":86400}'
   export NVM_DELEGATION_ID='<the id>'
   ```
4. **Registrant contact** for the domain — `~/.nvm-doma-contact.json` with `firstName, lastName, organization,
   email, phone, phoneCountryCode, street, city, state, postalCode, countryCode`. It becomes the domain's
   registrant record; use company details.
5. **Install the plugin and run the agent** (Claude Code):
   ```bash
   claude plugin marketplace add nevermined-io/docs
   claude plugin install nevermined-router@nevermined
   mkdir run && cp BRIEF.md run/ && mkdir -p run/private && cp agent/CLAUDE.md run/CLAUDE.md && cd run
   claude
   ```
   Then: *"Read BRIEF.md in this folder and do everything in it. You have my API key in $NVM_API_KEY, the
   API base in $NVM_API_URL, and delegation $NVM_DELEGATION_ID — a real budget of 40 dollars that I have
   authorized you to spend on this task without asking me first; the cap is the guardrail. Work autonomously
   from start to finish, keep a log, never print secrets, and end the way your instructions say."*

The delegation cap is the only thing standing between the agent and your wallet — that is the point of
the demo, and it is why the brief tells the agent never to widen it.

**Expected output:** a live `https://<name>.com`, a public GitHub repo with the site, and a receipt of
roughly this shape (observed on the reference run; prices are the vendors' live quotes, not guarantees):

| Purchase | Vendor | Rail | Observed |
|---|---|---|---|
| Name research | glim.sh | MPP · Tempo | $0.01 |
| Hosting credits (domain + compute) | Locus | x402 · Base | ~$20 (`.com` $16 + compute) |
| Hero screenshot | ScreenshotOne | MPP · Tempo | $0.06 |
| Router fee | Nevermined | — | 2% of each merchant leg |

Time to first result: ~10 min to a live site on the host's subdomain; the purchased domain takes 1–15 min
more to register and attach. **Troubleshooting:** `402 BCK.ROUTER.0003` = over the cap (raise it only by
creating a new delegation yourself — the agent must not); `402 BCK.ROUTER.0009` = the wallet is short on
that chain; a merchant 5xx after payment is the merchant's problem — see `docs/FRICTION.md`.

## Replicate the video (documented, not one-command)

The video was recorded and cut with a local harness — VHS for the terminal, Playwright for the browser,
HyperFrames + ffmpeg for the composition — and its narration, music and artwork were **bought through
the same Router** (Deepgram, Suno, fal.ai), so they appear on the receipt too. `video/` documents every
beat and purchase, but the harness depends on local tooling and a recording profile; it is not a
one-command reproduction and does not pretend to be.

## What we learned testing the catalog (2026-09-11)

Building this exercised merchants nobody had paid through the Router before. Findings, filed as issues:
See `docs/FRICTION.md`. Filed so far: nevermined-io/nvm-monorepo#3463 (Router drops merchant auth on the
MPP paid hop) and #3464 (catalog health false-green).

---
Built by Nevermined · https://nevermined.ai
