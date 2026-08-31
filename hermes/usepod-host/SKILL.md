---
name: usepod-host
description: >
  Operate POD Miner's UsePod inference hosts through the executor on this VPS.
  Use when provisioning a GPU box, hunting bargains, checking host status or
  earnings, retiring a host, or destroying an instance. Hermes does box work
  only — money moves are POD Miner's, bond signing is the ops wallet's.
version: "0.3.0"
---

# UsePod Host Operations (Hermes side)

You run the **box work** for POD Miner's inference business. The executor on
this VPS (`http://127.0.0.1:8402`) holds `VAST_API_KEY` and does all Vast
calls. You never touch the Vast key directly, never hold POD Miner's Solana
wallet, and never sign the bond transaction (the ops wallet keypair does that).

Auth: every executor call needs header `X-Dev-Token: $EXECUTOR_DEV_TOKEN`
(read it from the executor's environment, never print it).

## Executor API

| Call | Purpose |
|---|---|
| `GET /health` | Liveness + config. |
| `GET /bargains` | Cheapest + below-median offers per size class (s/m/l). |
| `POST /provision {payout_wallet, model?, size?, dry_run?}` | Enrolls host (API, no dashboard), rents cheapest reliable box in class, returns `provision_id`, `instance_id`, `bond`. Always `dry_run: true` first. |
| `GET /pair/<provision_id>` | Poll boot: `status: booting → ready`, bond details. |
| `GET /status/<instance_id>` | Vast instance state. |
| `POST /destroy/<instance_id>` | Destroys the box. **Retire on UsePod first** (see bond rules). |

## Machine economics (why selection works this way)

We earn **80% of billed tokens**. Listed price is **capped at the cheapest
centralized price** for the model, and routing picks the **cheapest eligible
provider** — reputation (uptime/latency) only breaks ties. Therefore:

1. Check the live floor before choosing a model:
   `curl -s https://api.usepod.ai/v1/providers` — match or undercut it, or
   pick a model with demand and no live self-hosted supply.
2. Rent the **minimum VRAM class** for the target model — idle VRAM earns
   nothing:
   - `s` = 3–8B models → 16GB (tiny price caps; floor play)
   - `m` = MoE 30–35B-A3B / 27–32B dense Q4 → 24GB (**commercial sweet spot**:
     3B-active MoE decodes fast with mid-model quality)
   - `l` = 70B Q4 → 48GB
3. Cheapest reliable box in class. Floors are hard: reliability2 ≥ 0.985,
   inet_down ≥ 500 Mbps, disk_bw ≥ 1500 MB/s, US region. An offline or
   throttled host earns $0 and burns reputation.
4. Utilization is the real risk: at the 35B-A3B floor ($0.05/$0.10 per 1M) a
   $0.13/hr box breaks even near ~600 tok/s sustained aggregate. Below that it
   bleeds slowly; above it prints.

## Bond rules (capital discipline)

- Bond is **$50 USDC per enrolled host** — every enrollment mints a unique
  `POD-BOND-…` code. **One bond per machine; never shared, never reused.**
- The bond is an on-chain `deposit_usdc` instruction on program
  `BBAdcqUkg68JXNiPQ1HR1wujfZuayyK3eQTQSYAh6FSW` with the deposit code in the
  instruction data (8 ASCII bytes after `POD-BOND-`). A plain SPL transfer —
  even with a memo — does **not** credit. The ops wallet signs it; POD Miner
  funds the ops wallet (50 USDC + ~0.01 SOL for gas).
- Refund path: **retire the host, then a mandatory 90-day cooldown**. An
  unretired host **forfeits** the bond. So: retire on UsePod → destroy the
  Vast box → calendar the refund at +90 days. Never destroy before retiring.
- Churn is capital-negative: swapping machines locks a second $50 while the
  first sits in cooldown. Bargains trigger **scale-out** (add a host when
  earnings cover it), never replacement.

## Bargain hunting (periodic)

A system cron runs `/bargains` every 30 min and appends to
`/var/log/podminer-bargains.log`. Check that log when asked about deals, or
call `GET /bargains` live. A bargain = ≤ 0.7× the class median $/hr. Report
deals with: class, offer_id, GPU, VRAM, $/hr vs class median, reliability.
Recommend scale-out only if current hosts are earning near capacity.

## Provisioning runbook

1. `POST /provision {"dry_run": true, "size": "m"}` → report the chosen host
   (id, GPU, VRAM, $/hr, reliability) and get go-ahead.
2. Real provision with POD Miner's payout wallet. Executor enrolls the host
   first (returns `bond.deposit_code`) — relay the bond code + amounts to the
   conductor so POD Miner can fund the ops wallet.
3. Poll `GET /pair/<provision_id>` until `ready` (box boot + model pull takes
   ~5–15 min depending on model size and host disk).
4. Bond posted → host goes `active` in `/v1/providers`. Verify it lists with
   our pricing before declaring done.
5. If boot stalls: `GET /status/<instance_id>`, then SSH is available via the
   instance's `public_ipaddr:ssh_port` if needed.

## Never

- Never rent above the model's VRAM class ("bigger just in case" = burned margin).
- Never lower the reliability floors to find a cheaper box.
- Never destroy an instance whose host hasn't been retired on UsePod.
- Never print or exfiltrate `VAST_API_KEY`, `EXECUTOR_DEV_TOKEN`, `host_token`,
  or the ops wallet key.
- Never improvise a pairing/bond flow outside this doc — the enroll API +
  `deposit_usdc` path above is the only correct one.
