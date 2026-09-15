# How the video was made

Three chapters, recorded separately and assembled with an edit decision list (`edit.json`).

| Chapter | Surface | What is real |
|---|---|---|
| 1 · A human sets the budget | Browser (`nevermined.app`, Live) — Playwright-driven, login typed by a human | The login, the wallet balances, the delegation creation. The on-ramp and bridge dialogs are **shown and closed** — the wallet was already funded. |
| 2 · The agent goes shopping | Terminal — a real Claude Code session recorded with VHS | Everything: the plugin install, every purchase, every refusal. The API key is exported from the OS keyring inside a hidden setup block and never appears on screen. |
| 3 · The receipt is the website | Browser — the live domain, then the delegation's spent/cap row | The site the agent deployed, unedited. |

Purchases made **for the video itself**, through the same Router and delegation, and listed on the site's receipt:

| What | Vendor (catalog slug) | Rail |
|---|---|---|
| Narration lines | Deepgram Aura-2 (`deepgram-via-mpp`, `/deepgram/speak`, ~$0.023/line) | MPP · Tempo |
| Music bed | Suno (`suno-mpp`) | MPP · Tempo |
| Title card / thumbnail | fal.ai (`fal-ai-mpp`) | MPP · Tempo |

Files: `beats/` (the Playwright beat scripts and the VHS tape, with ids blanked), `edit.json` (the EDL),
`narration.md` (every spoken line), `purchases.jsonl` (the ledger rows behind the narration, music and art).

The harness itself (VHS 0.11, Playwright on Chrome, HyperFrames + ffmpeg composition, a recording browser
profile) lives outside this repo and is not a one-command reproduction — this folder documents the process.
