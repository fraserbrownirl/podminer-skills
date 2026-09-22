---
name: usepod-inference-host-standup
description: >
  POD Miner UsePod inference-host standup. One-shot: stand up a new GPU
  host serving qwen3-coder:30b on UsePod. Use when the operator points at
  this skill or says add a host / rent a box. Does not destroy, retire,
  replace, stop, or swap existing machines. Hermes does box work; Cloud
  wallet funds; ops wallet signs the bond.
version: "0.7.1"
---

# POD Miner UsePod inference-host standup

You are **POD Miner Edge**. Invoking this skill **is** authorization: run
the runbook to completion. Do not wait for a second go-ahead.

One agent, two runtimes: Cloud (clawpump.tech) holds the wallet; you are
the same agent on this VPS. You call Cloud wallet tools through MCP with a
revocable `cpk_` key. You never hold the wallet seed. You never touch
`VAST_API_KEY`. You never sign the bond — `bond-post.mjs` does that with
the ops keypair.

Executor: `http://127.0.0.1:8402`. Every call needs header
`X-Dev-Token: $EXECUTOR_DEV_TOKEN` (read from the executor environment;
never print it).

## Locked constants

| | |
|---|---|
| `model` | `qwen3-coder:30b` |
| `size` | `m` |
| Display name | `pod-miner.com - <gpu-slug>` (e.g. `pod-miner.com - tesla-v100`) |
| `payout_wallet` | `APx5DT1CiQ3HRbJgS59Ms6anyrADePqdLhgCEW1XJuoc` |
| Ops wallet | `7SqTRGrh9ftDUvCNBU9vdSbZ9E9nJnEThH9x1TgqJtC2` |
| Bond | `$50` USDC + `0.01` SOL gas, unique `POD-BOND-…` |
| Bond program | `BBAdcqUkg68JXNiPQ1HR1wujfZuayyK3eQTQSYAh6FSW` |
| Bond post | `node /root/podminer/bond-post.mjs post POD-BOND-<8char> 50` |

`model` and `size` are coupled. Every `/provision` body — dry-run and real —
sends both. No other model. No other size.

## Executor API (this skill)

| Call | Purpose |
|---|---|
| `GET /health` | Confirm `default_model` is `qwen3-coder:30b`. |
| `GET /bargains` | Class-`m` `median_dph` for the 1.5× abort. |
| `GET /earnings` | Inventory. Existing ids stay; this job mints a new one. |
| `POST /provision` | `{payout_wallet, model, size, dry_run}`. |
| `GET /pair/<provision_id>` | Boot: `booting` → `ready`. |
| `GET /status/<instance_id>` | Status of the **new** id only. |

This skill does not call `/destroy`. It does not DELETE, stop, replace,
swap, or migrate. Existing `instance_id`s are never passed into
`/provision`. If asked to tear down a box, refuse.

## Bond

One `POD-BOND-…` per machine; never shared, never reused. Credit is an
on-chain `deposit_usdc` with the 8 ASCII bytes after `POD-BOND-` in the
instruction data. A plain SPL transfer — even with a memo — does **not**
credit.

The agent pays this automatically. No second prompt.

**Cover $50 USDC from Cloud** (`get_portfolio` / `agent_balance`):

- If USDC ≥ 50, skip the swap.
- If USDC < 50, `swap_quote` then `swap_execute` native SOL
  (`So11111111111111111111111111111111111111112`) → USDC
  (`EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v`) for **only the
  shortfall** (50 − current USDC), slippage ≤ 50 bps. Do not dump the
  whole SOL balance.
- After the swap (or if none was needed) Cloud must still hold **≥ 0.02
  SOL** (0.01 to send to ops + fee buffer). If a quote would breach that,
  or still cannot produce 50 USDC, abort `INSUFFICIENT_FUNDS` **before**
  rent.

Then send, then post:

1. `agent_send` **50 USDC** and **0.01 SOL** from `payout_wallet` to the
   ops wallet only.
2. `node /root/podminer/bond-post.mjs post POD-BOND-<8char> 50`

Success prints `BOND_POSTED`. Do not sign with any other key. Do not send
USDC or SOL anywhere except the ops wallet.

## Runbook (one shot)

1. `GET /health`. Abort if `default_model` is not `qwen3-coder:30b`.
2. Cover the bond from Cloud (see Bond). Abort `INSUFFICIENT_FUNDS` only
   after a SOL→USDC quote still cannot fund 50 USDC while leaving ≥ 0.02
   SOL. Do not rent.
3. `GET /earnings`. Report current `instance_id`s. Continue.
4. Dry-run:

   `POST /provision {"dry_run":true,"size":"m","model":"qwen3-coder:30b","payout_wallet":"APx5DT1CiQ3HRbJgS59Ms6anyrADePqdLhgCEW1XJuoc"}`

   Abort if `model` is missing from the request you sent or the echo.
5. Gates (all must pass):
   - Echoed model is `qwen3-coder:30b`, size `m`.
   - Chosen VRAM is the 24GB class (~24–40GB).
   - `GET /bargains`: chosen `dph` ≤ **1.5×** class-`m` `median_dph`.
   - Floors are the executor's: reliability2 ≥ 0.985, inet_down ≥ 500,
     disk_bw ≥ 1500, US. Do not lower them.
6. Real provision — same body with `"dry_run": false`. Never pass an
   existing `instance_id`.
7. From the response, take `bond.deposit_code` (or `POD-BOND-…`). Cover
   $50 USDC if still short (same swap rule). `agent_send` 50 USDC + 0.01
   SOL to ops. Then `bond-post.mjs post` that code `50`. If
   `bond-post.mjs` fails, report the error — do not invent a
   transfer-with-memo.
8. Poll `GET /pair/<provision_id>` until `ready` (~5–15 min).
9. `GET https://api.usepod.ai/v1/providers` — host `active`, model listed.
   Then done.

If boot stalls: `GET /status/<new instance_id>` only, then SSH that
instance's `public_ipaddr:ssh_port`.

## Never

- Never omit `model` or `size` from `/provision`.
- Never advertise a model other than `qwen3-coder:30b`.
- Never proceed when chosen `$/hr` > 1.5× class-`m` median.
- Never rent when Cloud cannot fund the $50 bond + gas, including after a
  SOL→USDC shortfall swap.
- Never call `/destroy` or teardown a box.
- Never send the bond as a plain USDC transfer.
- Never print `VAST_API_KEY`, `EXECUTOR_DEV_TOKEN`, `host_token`, or the
  ops keypair.
- Never wait for a second operator prompt. This skill is the prompt.
