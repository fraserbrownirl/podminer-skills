---
name: usepod-inference-host-standup
description: >
  POD Miner UsePod inference-host standup. One-shot: stand up a new GPU
  host on UsePod. Default next product is homura:30b (Homura-30B). Use when
  the operator points at this skill or says add a host / rent a box. Does
  not destroy, retire, replace, stop, or swap existing machines. Hermes
  does box work; Cloud wallet funds; ops wallet signs the bond.
version: "0.8.0"
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

## Model options

UsePod routes by **canonical model id**, not “feels like the same 30B.”
Pick one row. The operator naming a row overrides the default.

| Option | Role | Pull | Advertise on UsePod | Size | List price |
|---|---|---|---|---|---|
| **`homura:30b`** | **Next box (default).** HYRE exclusive. No centralized cap. Dense ~17 GB Q4. | `hf.co/hyrelabs/Homura-30B-GGUF:Q4_K_M` then `ollama cp` → `homura:30b` | **`homura-30b` only** | `m` | Match live `homura-30b` listings (HYRE was `$0.15` / `$0.45`). Check `/v1/providers` at dry-run. |
| **`qwen3-coder:30b`** | Already live on Tesla `51382627`. Catalog commodity, price-capped. | Ollama library `qwen3-coder:30b` | `qwen3-coder-30b` **and** `qwen3-coder-30b-a3b-instruct` at the same price | `m` | Relay floor (~`$0.0056` / `$0.0216`, 90% off centralized). |
| llama 8B | Never on `m`. That was the V100 mistake. | — | — | `s` only | Executor rejects `llama3.1:8b` + `size: m`. |

Uncapped exclusives (Homura) can earn; capped catalog Qwen is presence. Do
not install Homura on the existing Tesla. Do not add a second Qwen box
unless the operator names that row.

Before dry-run, `GET https://api.usepod.ai/v1/providers` and record the
cheapest enabled `homura-30b` listing. List at that price, or up to 10%
under. Do not 99%-off an exclusive.

## Locked constants (this standup)

| | |
|---|---|
| `model` | `homura:30b` |
| `size` | `m` |
| Display name | `pod-miner.com - <gpu-slug>` |
| `payout_wallet` | `APx5DT1CiQ3HRbJgS59Ms6anyrADePqdLhgCEW1XJuoc` |
| Ops wallet | `7SqTRGrh9ftDUvCNBU9vdSbZ9E9nJnEThH9x1TgqJtC2` |
| Bond | `$50` USDC + `0.01` SOL gas, unique `POD-BOND-…` |
| Bond program | `BBAdcqUkg68JXNiPQ1HR1wujfZuayyK3eQTQSYAh6FSW` |
| Bond post | `node /root/podminer/bond-post.mjs post POD-BOND-<8char> 50` |

`model` and `size` are coupled. Every `/provision` body — dry-run and real —
sends both. Existing Tesla (`51382627`, Qwen) stays.

## Executor API (this skill)

| Call | Purpose |
|---|---|
| `GET /health` | Confirm executor up. `default_model` should be `homura:30b`. |
| `GET /bargains` | Class-`m` `median_dph` for the 1.5× abort. |
| `GET /earnings` | Inventory. Existing ids stay; this job mints a new one. |
| `POST /provision` | `{payout_wallet, model, size, dry_run}`. |
| `GET /pair/<provision_id>` | Boot: `booting` → `ready`. |
| `GET /status/<instance_id>` | Status of the **new** id only. |
| `POST /actions` | Public ticker. One line per step. Never secrets. |

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

1. `GET /health`. Executor must be up. Prefer `default_model` `homura:30b`.
   Still send `model` in every `/provision` body.
2. Cover the bond from Cloud (see Bond). Abort `INSUFFICIENT_FUNDS` only
   after a SOL→USDC quote still cannot fund 50 USDC while leaving ≥ 0.02
   SOL. Do not rent.
3. `GET /earnings`. Report current `instance_id`s. Tesla `51382627` stays.
   Continue.
4. `GET https://api.usepod.ai/v1/providers`. Note live `homura-30b` prices.
5. Dry-run:

   `POST /provision {"dry_run":true,"size":"m","model":"homura:30b","payout_wallet":"APx5DT1CiQ3HRbJgS59Ms6anyrADePqdLhgCEW1XJuoc"}`

   Abort if `model` is missing from the request you sent or the echo.
6. Gates (all must pass):
   - Echoed model is `homura:30b`, size `m`.
   - Chosen VRAM is the 24GB class (~24–40GB). Homura Q4 is ~17 GB.
   - `GET /bargains`: chosen `dph` ≤ **1.5×** class-`m` `median_dph`.
   - Floors are the executor's: reliability2 ≥ 0.985, inet_down ≥ 500,
     disk_bw ≥ 1500, US. Do not lower them.
7. Real provision — same body with `"dry_run": false`. Never pass an
   existing `instance_id`.
8. From the response, take `bond.deposit_code` (or `POD-BOND-…`). Cover
   $50 USDC if still short (same swap rule). `agent_send` 50 USDC + 0.01
   SOL to ops. Then `bond-post.mjs post` that code `50`. If
   `bond-post.mjs` fails, report the error — do not invent a
   transfer-with-memo.
9. Poll `GET /pair/<provision_id>` until `ready` (~5–15 min). Homura GGUF
   pull is ~17 GB; allow longer than Qwen if the box is slow.
10. `GET https://api.usepod.ai/v1/providers` — new host `active`, model
    `homura-30b` listed. Tesla Qwen listings unchanged. Then done.

If boot stalls: `GET /status/<new instance_id>` only, then SSH that
instance's `public_ipaddr:ssh_port`.

## Live actions (pod-miner.com)

After every runbook step, POST a one-line event so the public ticker
moves. Never include tokens, keys, `host_token`, `enrollment_code`, or
`X-Dev-Token` in `detail`. A missed event is not a reason to abort.

```
curl -s -X POST http://127.0.0.1:8402/actions \
  -H "X-Dev-Token: $EXECUTOR_DEV_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"agent":"edge","action":"<step>","detail":"<one line>","status":"ok"}'
```

`action` examples: `health`, `inventory`, `dry-run`, `gate`, `provision`,
`swap`, `send`, `bond-post`, `ready`. On failure use `"status":"fail"`.

## Never

- Never omit `model` or `size` from `/provision`.
- Never advertise Homura under a second id (no hf.co path on the
  marketplace). One GGUF, one canonical `homura-30b`.
- Never replace or destroy `51382627` as part of this standup.
- Never proceed when chosen `$/hr` > 1.5× class-`m` median.
- Never rent when Cloud cannot fund the $50 bond + gas, including after a
  SOL→USDC shortfall swap.
- Never call `/destroy` or teardown a box.
- Never send the bond as a plain USDC transfer.
- Never print `VAST_API_KEY`, `EXECUTOR_DEV_TOKEN`, `host_token`, or the
  ops keypair.
- Never wait for a second operator prompt. This skill is the prompt.
