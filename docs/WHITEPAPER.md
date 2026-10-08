# POD Miner

POD Miner runs three compute businesses with one agent, one Solana wallet, and one public API.

*Version 0.3 — October 2026*

---

## Abstract

POD Miner finds models people want, stands servers up under them, and sells the inference. It does that on three lines:

1. **Niche and obliterated models.** The agent hunts Hugging Face for in-demand obliterated and niche weights, then finds servers that can serve them. The community buys that inference on a paid API or over x402.
2. **Dolphin and POD.** The agent hunts server deals that fit Dolphin's latest offerings, mines POD on those boxes, and can turn the profit into defense of the $PODM price.
3. **Mainstream inference, rebated.** POD Miner offers mainstream models at hyper-competitive prices because $PODM fees come back to users as rebates.

The operating procedure is open. Rents are add-only: a new host never replaces a live one.

## 1. Niche and obliterated models

Most inference catalogs serve the same few names. POD Miner looks where they do not.

The agent searches Hugging Face for obliterated and niche models that already have demand. A model only moves forward when a server can actually hold it: the VRAM class, reliability, bandwidth, and disk have to fit. The agent then stands that server up and publishes the model to the community.

Buyers pay in either of two ways:

- **Paid API.** OpenAI-compatible calls at `https://api.pod-miner.com/v1`, settled in USDC credits.
- **x402.** An agent pays per request for the same inference, without a prepaid key.

A new model is a new host. It is never a reason to destroy one that is already serving.

## 2. Dolphin, POD, and the $PODM defense

Dolphin publishes offerings that need specific machines. POD Miner's agent watches the server market for the best deals that fit those offerings: the cheapest reliable hardware in the right class, not the first cheap box.

Those hosts mine **POD**. Profit from that mining is capital the business can convert into defending the **$PODM** price — open-market support for the token, paid for by the compute the agent actually ran.

The Dolphin path is one agent action. Cloud pays the executor. Edge follows the Dolphin skill. The box is Vast credit, not an on-chain rent.

## 3. Mainstream inference and the rebate

The third line is the opposite of scarce weights: mainstream models, priced to win.

POD Miner can price that inference below a naive cost stack because **$PODM fees are rebates for users**:

> Pod Miner distributes 50% of its fees to users as a rebate every day. For example, if 24hr fees were $350 and users had spent $500 on inference, $175 will be distributed by the Pod Miner ClawPump agent straight to the users wallets.

The rebate is the pricing strategy. Users pay a hyper-competitive rate, and half of the fees return to the wallets that spent.

## 4. Architecture

One agent. The wallet's private key stays in ClawPump custody.

| Piece | Does |
|---|---|
| **Cloud** (clawpump.tech) | Decides and pays. Holds the wallet `APx5DT1CiQ3HRbJgS59Ms6anyrADePqdLhgCEW1XJuoc`. |
| **Edge** (Hermes on the VPS) | Follows the skills and does the box work. A revocable key, not the seed. |
| **Executor** | Holds the Vast key. Rents hosts and wakes Dolphin. |
| **Public API** | `https://api.pod-miner.com/v1` — paid keys, ClawPump agents, and consumer credits. |

Earlier marketplace enrollment is not this design. The three lines above are the business.

## 5. How a host is chosen

The same floors apply on every line:

1. **Minimum VRAM that fits the model.** Idle VRAM is waste. Classes stay coupled to the weights: do not put a small model on a card bought for a larger one.
2. **Cheapest box that clears the floors.** Reliability ≥ 98.5%, inbound bandwidth ≥ 500 Mbps, disk ≥ 1500 MB/s. The floors are not lowered to chase a price.
3. **Bargains add a host.** A scan can justify another box when that line can pay for it. It never justifies destroying a live host.
4. **Destroy is a separate operation.** "New model" and "better deal" mean add. They do not mean retire.

## 6. $PODM

**$PODM** is live on Solana:

```
mint: CdmjEpps6wHfYQM9z4goZMdCTZrPgdzguM5CNa7C3LM1
```

Verify the mint before any trade. Treat any other "POD Miner" token as counterfeit. The token is where the three lines meet:

- **Rebates.** Fees on mainstream inference return to users, as in §3.
- **Defense.** POD mining profit can be converted into support for the $PODM price, as in §2.
- **Treasury.** The agent wallet and the mint are public. Rebates and buys that defend the price are on-chain events.

The skills that govern the agent's decisions are open: [github.com/fraserbrownirl/podminer-skills](https://github.com/fraserbrownirl/podminer-skills). This paper and [`llms.txt`](https://github.com/fraserbrownirl/podminer-skills/blob/main/docs/llms.txt) are MIT-licensed in [`docs/`](https://github.com/fraserbrownirl/podminer-skills/tree/main/docs).

## 7. Risks

- **A niche model can be wanted and still not fill a box.** Mitigation: small cost basis, and a server only when the weights and the deal both fit.
- **Dolphin's offerings change.** A box that fit yesterday can be the wrong shape tomorrow. Mitigation: hunt against the latest offering, and do not churn a host that is still earning.
- **Rebates need fees.** A price with no volume returns nothing. Mitigation: mainstream models stay on the catalog people already call.
- **A rented host can go offline.** Mitigation: reliability floors. Destroy is never implied by a new box.

## 8. What is public

- Ops console: [pod-miner.com](https://www.pod-miner.com)
- Credits and a `pk-cm-` key: [pod-miner.com/app](https://www.pod-miner.com/app)
- Inference: `https://api.pod-miner.com/v1`
- Skills: [github.com/fraserbrownirl/podminer-skills](https://github.com/fraserbrownirl/podminer-skills)
- This paper: [docs/WHITEPAPER.md](https://github.com/fraserbrownirl/podminer-skills/blob/main/docs/WHITEPAPER.md)
- Operator file: [docs/llms.txt](https://github.com/fraserbrownirl/podminer-skills/blob/main/docs/llms.txt)

---

*The procedures are open. Fix the class of mistake, not the instance.*
