# Security Research | Mohsin Arif ([@mhsn1](https://github.com/mhsn1))

Security researcher focused on **smart contracts / web3** (Solidity, EVM, DeFi) and **AI/ML systems** (web/AppSec, LLM security). This repository collects my vulnerability disclosures and technical writeups.

I report responsibly and privately, back every finding with a runnable proof-of-concept, and care about the line between a *bug class* and a real, reachable *impact*.

## Disclosures & writeups

| Date | Target | Class | Outcome | Writeup |
|------|--------|-------|---------|---------|
| 2026-10 | [Pipecat](https://github.com/pipecat-ai/pipecat) (Daily) | Path traversal / trust boundary | Docs clarified & credited ([PR #6034](https://github.com/pipecat-ai/pipecat/pull/6034)) | [Read →](./writeups/2026-10-pipecat-flowconfig-include-trust-boundary.md) |

*Additional findings are under coordinated disclosure and will be added here once the vendors publish their advisories.*

## Practical / CTF writeups

| Platform | Room / Target | Focus | Writeup |
|----------|---------------|-------|---------|
| TryHackMe | Rabbit Hole | Second-order SQL injection, MySQL `processlist` abuse, SSH foothold | [Read →](./writeups/tryhackme-rabbithole.md) |

Automation scripts for the above live in [`writeups/rabbithole-scripts/`](./writeups/rabbithole-scripts).

## How I work
Responsible, private disclosure first. A dangerous function is not a vulnerability until untrusted input can **provably reach it** across a boundary the project defends — so I reproduce every finding end-to-end against the real software (not a hand-built harness) before reporting, and I include a control test that isolates the single condition responsible for the bug. I verify reachability at default settings, scope impact honestly, and accept a maintainer's correct classification gracefully.

## Focus areas
- **Web / AppSec** — SSRF, path traversal / LFI, IDOR & broken access control, auth bypass.
- **LLM / AI systems** — prompt injection, guardrail/sandbox bypass, red-teaming RAG & agent systems; SSRF & code-execution in LLM tooling.
- **Smart contract / DeFi** — access control, accounting & invariant bugs, oracle/price-feed logic; PoCs with Foundry.

## Links
- GitHub: [@mhsn1](https://github.com/mhsn1) · TryHackMe: [mhsnarf](https://tryhackme.com/p/mhsnarf) · Portfolio: [mhxmllc.com](https://www.mhxmllc.com)
