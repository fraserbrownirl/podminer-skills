# POD Miner Skills

Open operating skills for **POD Miner** — an autonomous, self-funding compute
business. POD Miner is a [ClawPump](https://clawpump.tech) agent that earns
USDC serving inference on [UsePod](https://usepod.ai) and spends it renting
GPU boxes on [Vast.ai](https://vast.ai), with the goal of becoming a
self-sustaining commercial inference provider.

These files are the agent's actual operating procedures — the instructions
that decide which machines to rent, how to price, when to scale, and how
capital (bonds) is handled. **They are open on purpose: the community is
invited to improve them.**

## The skills

| Skill | Runs on | Purpose |
|---|---|---|
| [`clawpump/usepod-host/SKILL.md`](clawpump/usepod-host/SKILL.md) | POD Miner (ClawPump cloud agent) | Money side: fund the ops wallet, verify bonds on-chain, check earnings, withdraw, report. Holds the Solana wallet; never touches Vast keys or SSH. |
| [`hermes/usepod-host/SKILL.md`](hermes/usepod-host/SKILL.md) | Hermes (VPS operator agent) | Box side: drive the executor API, pick machines by size class and reliability floors, hunt bargains, enforce bond discipline. Never touches the wallet. |

## Design principles encoded in these skills

- **Secret separation** — the agent that holds money never holds cloud keys;
  the agent that touches cloud keys never holds money.
- **Minimum VRAM class** — rent the cheapest reliable box that fits the target
  model; idle VRAM earns nothing.
- **Cheapest-eligible wins** — UsePod routing picks the cheapest provider at or
  under the centralized price cap; reputation only breaks ties.
- **Bond discipline** — $50 USDC per host, one bond per machine, 90-day
  cooldown after retirement. Bargains trigger scale-out, never churn.
- **Judgment is not generatable** — skills state procedures and floors; they
  do not fake per-item judgment.

## Contributing

Pull requests welcome. Useful directions:

- Better host-selection economics (utilization models, demand signals)
- Model-class definitions as the open-model landscape moves
- New marketplace adapters beyond UsePod
- Bond/treasury automation hardening
- Post-mortems: if the agent made a bad call, fix the *class* of mistake in
  the skill, not the instance

Keep edits **general** — rules that transfer, not thresholds tuned to one
machine. See each skill's frontmatter for versioning.

## Links

- Whitepaper and ops console: see the main POD Miner project
- [UsePod docs](https://docs.usepod.ai) · [ClawPump](https://clawpump.tech) · [Vast.ai](https://vast.ai)
