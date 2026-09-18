# POD Miner

## An autonomous, self-funding inference business

*Version 0.1 — August 2026*

---

## Abstract

POD Miner is a software agent that runs a compute business with no human in
the loop. It earns USDC by serving open-model inference on the **UsePod**
marketplace and spends that USDC renting GPU capacity on **Vast.ai**. It holds
its own Solana wallet, makes its own treasury decisions under an open,
community-auditable operating skill, and is designed to be profit-seeking from
the first box it rents. This paper describes the architecture, the economics,
the capital rules, and the governance model.

## 1. The business

Inference is a commodity market with a simple structure: buyers route to the
cheapest eligible provider, prices are capped by centralized alternatives, and
reliability compounds through reputation. Margins come from one place —
**serving more tokens per dollar of rented compute than the price floor
implies**. POD Miner is built around that single equation:

```
profit = 0.8 × price × tokens_served − box_cost
```

Every design decision in this paper follows from maximizing utilization of the
cheapest reliable hardware that fits a competitive model.

## 2. Architecture — one agent, two runtimes

POD Miner is **one agent** — one Solana wallet, one ClawPump identity — with
two runtimes that share a single operating procedure:

| Component | Holds | Does |
|---|---|---|
| **POD Miner Cloud** (hosted runtime, clawpump.tech) | Solana wallet (ClawPump custody) | Decides. Funds. Verifies on-chain. Reports. |
| **POD Miner Edge** (Hermes runtime, VPS) | A **revocable** ClawPump API key — wallet-tool access via MCP, *not* the private key | Drives the executor; hunts bargains; enforces discipline; reaches what the cloud runtime can't (shell, SSH, boxes). |
| **Executor** (service on the VPS) | Vast API key | Enrolls hosts, rents boxes, installs the UsePod agent, wires config. |
| **Ops wallet** (hot keypair on the VPS) | One bond + gas at most | Signs the on-chain bond instruction. |
| **UsePod** | Marketplace | Routes demand; pays 80% of billed tokens. |

The custody invariant: **the wallet's private key never leaves ClawPump.**
Edge can *request* sends through the MCP tool ceiling (plain transfers, swaps)
using its revocable API key, but it cannot sign arbitrary transactions, and
the key can be killed from the dashboard without touching the treasury.
Cloud keys (Vast) live only on the executor; the runtime that can move money
never holds them. A compromise of any single component cannot drain the
treasury.

## 3. Machine economics

UsePod's market rules shape everything:

- A provider's listed price is **capped at the cheapest centralized price** for
  that model.
- Routing picks the **cheapest eligible provider**; reputation (uptime,
  latency) only breaks ties.
- An offline or throttled host earns nothing and damages the reputation that
  future routing depends on.

Therefore the selection policy is:

1. **Minimum VRAM class, never above it.** Idle VRAM earns nothing.
   Classes: `s` = 3–8B models (16GB), `m` = MoE 30–35B active-3B or 27–32B
   dense Q4 (24GB), `l` = 70B Q4 (48GB).
2. **Cheapest reliable box in class.** Hard floors: reliability ≥ 98.5%,
   inbound bandwidth ≥ 500 Mbps, disk ≥ 1500 MB/s. The floors are never
   lowered to find a cheaper box.
3. **Continuous bargain hunting.** The fleet scans the market every 30 minutes
   and flags offers below 0.7× the class median. Bargains justify *scale-out*
   when earnings cover them — never churn of an existing host.
4. **Utilization is the real risk.** At the 35B-class floor
   ($0.05/$0.10 per million tokens in/out), a $0.13/hr box breaks even near
   600 tokens/second sustained. Below that it bleeds slowly; above it prints.

## 4. Capital and the bond

UsePod requires a **$50 USDC bond per host**, posted on-chain as a
`deposit_usdc` instruction carrying a unique per-host deposit code. The rules
that govern this capital:

- **One bond per machine.** Bonds are never shared, reused, or pooled.
- **Refund path**: retire the host, then a mandatory **90-day cooldown**. An
  unretired host forfeits. Retire → destroy → calendar the refund.
- **Churn is capital-negative.** Swapping machines locks a second $50 while
  the first sits in cooldown. Hosts are keepers, not experiments.
- The ops wallet never holds more than one bond plus gas. The treasury lives
  in POD Miner's own wallet; withdrawals from UsePod flow back to it.

## 5. Autonomy boundaries

POD Miner operates under a published skill — a written operating procedure the
community can read and improve. Its hard limits:

- It can only send funds to the ops wallet (host operations) or an explicit
  withdrawal destination.
- It cannot rent above a model's VRAM class, lower reliability floors, or
  invent payment flows.
- If a send would exceed its balance, it stops and reports
  `INSUFFICIENT_FUNDS`.

The skills are versioned and developed in the open:
[github.com/fraserbrownirl/podminer-skills](https://github.com/fraserbrownirl/podminer-skills).

## 6. Token — $PODM

**$PODM is live** on Solana, launched via ClawPump's pump.fun integration on
2026-09-18:

```
mint: CdmjEpps6wHfYQM9z4goZMdCTZrPgdzguM5CNa7C3LM1
```

Verify the mint before any trade — treat any other "POD Miner" token as
counterfeit. The official links are this repository and the ops console.

The token's relationship to the business:

- **Buyback and burn.** Host profits (USDC earnings minus Vast costs) fund
  open-market $PODM buys that are burned. The console's Economics panel tracks
  the cumulative bought-and-burned total publicly.
- **Transparent treasury.** POD Miner's wallet
  (`APx5DT1CiQ3HRbJgS59Ms6anyrADePqdLhgCEW1XJuoc`) and the ops wallet are
  public; every bond, withdrawal, and burn is on-chain.
- **Open operations.** The skills that govern the agent's decisions are this
  repository — the community can audit and improve the exact procedures the
  agent follows.

## 7. Risks

- **Demand risk**: utilization below break-even bleeds the treasury slowly.
  Mitigation: small cost basis, ruthless box selection, scale only on earnings.
- **Marketplace risk**: UsePod pricing rules, caps, or bond terms can change.
  Mitigation: the skills are parameterized, not hard-coded.
- **Counterparty risk**: Vast hosts can go offline. Mitigation: reliability
  floors, reputation-weighted selection, fast destroy-and-replace (after
  retiring on UsePod).
- **Key risk**: mitigated by the separation architecture in §2.

## 8. Roadmap

1. **Now**: first m-class host live and earning; console observability;
   skills repo public.
2. **Next**: competitive MoE model serving (35B-A3B class) via llama.cpp;
   automated earnings → scale-out loop driven by the bargain stream.
3. **Later**: multi-host fleet with per-host P&L; token launch against proven
   revenue; community governance of the operating skills.

---

*POD Miner is an experiment in autonomous business. Its operating procedures
are open source; contributions are welcome.*
