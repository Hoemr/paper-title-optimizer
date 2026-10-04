# Paper Title Optimizer

> Make your paper **findable and citable** by AI answer engines — ChatGPT, Perplexity, Google AI Overviews — and literature indexes (arXiv, Semantic Scholar).

**Paper Title Optimizer** is an [agent skill](https://github.com/anthropics/skills) (`SKILL.md` + scripts + data) for [Claude Code](https://claude.com/claude-code), Codex, and ZCode that rewrites **academic paper titles and abstracts** for the age of AI search. Think SEO, but for papers — the literature calls it **Generative Engine Optimization (GEO)** (KDD 2024). 论文标题/摘要的 AI 检索优化。

![License](https://img.shields.io/badge/license-MIT-blue)

## Why

Reviewers are no longer the only readers of your title and abstract. When a researcher asks ChatGPT or Perplexity *"which paper should I read about X?"*, the answer is assembled by **lifting whole sentences** out of titles, abstracts, and related work. A title written only for a 2015-style human skimmer is invisible to that pipeline:

- AI engines retrieve by **semantic match against queries researchers actually type** — not by the internal jargon your paper coins.
- The GEO paper (KDD 2024) measured what moves the needle: **quoting +40%, statistics +30%, fluent readable prose +15–30% — and keyword stuffing is the only measured-harmful tactic.**
- Title/abstract are the only fields indexed by arXiv, Semantic Scholar, OpenAlex, and Crossref. If the word isn't there, the paper doesn't exist for that query.

## What the skill does

Six stages, five non-negotiable constraints, three layers:

1. **Stage 0 — Visibility baseline.** Before touching anything, probe whether 8–12 queries can actually retrieve your paper (arXiv backend built in; ChatGPT/Perplexity optional via API keys). No baseline, no way to tell better from worse later.
2. **Stage 1 — Claim→evidence map.** Every number in a rewrite must point to a table/section in the manuscript; missing ones are marked `[待补]`, never guessed. Includes **method-name collision checks** against the arXiv corpus (a colliding acronym makes AI confuse your results with someone else's).
3. **Stage 2 — Title & abstract rewrite (L1).** 2–3 title variants with explicit trade-offs; abstract rebuilt sentence-by-sentence for self-contained quotability.
4. **Stage 3 — Cross-community coverage (L2).** 8–12 queries written by *outsiders who don't know your jargon*, matched against the abstract in a hit/partial/miss matrix; capped at three vocabulary spaces so reviewers still know which community you belong to.
5. **Stage 4 — Term gate (L3).** Trending terms must pass four gates (real relationship, rising phase, not peaked, annualized data) before entering your abstract; rejected terms are reported with reasons, never silently dropped.
6. **Stage 5+6 — Body pointers & delivery.** Location-level suggestions for the manuscript body (never full rewrites), then a 9-item output contract.

## Evidence-based, with receipts

Most "optimization" advice is folklore. This skill ships its own data: `scripts/title-stats.mjs` analyzed **16,677 accepted titles** from ICLR 2025, NeurIPS 2024, ICML 2024, ACL 2025, and CVPR 2024 (via [papercopilot/paperlists](https://github.com/papercopilot/paperlists)). Findings baked into the rules:

- Only **1.8–2.8%** of accepted titles carry a quantitative number — numbers are a *differentiation* move, not a default.
- **Decimal percentages are near-extinct** in accepted titles (two-decimal cases: **0** across five conferences). Write `half the memory`, not `48.6% less memory`; keep the exact value for the abstract.
- Oral/award papers carry numbers 1.5–2.3× more often than posters — consistent with strong results enabling numbers, not numbers buying citations (selection effect, stated honestly).
- Accepted-title phrase pool (`references/title-word-pool.csv`) seeds subtitle word choices with real community vocabulary.

Every conclusion is reproducible: `node scripts/title-stats.mjs <venue-year>.json` on any year's data.

## Install

```bash
# Claude Code
mkdir -p ~/.claude/skills && cp -r paper-title-optimizer ~/.claude/skills/

# Codex
mkdir -p ~/.codex/skills && cp -r paper-title-optimizer ~/.codex/skills/

# cc-switch
cp -r paper-title-optimizer ~/.cc-switch/skills/

# Or drop the folder into your agent workspace's .skills/ directory
```

Scripts need Node 18+ (no third-party dependencies). `OPENAI_API_KEY` / `PERPLEXITY_API_KEY` are optional — without them the visibility probe reports `skipped` ("not measured" ≠ 0) instead of fabricating data.

## Usage

Just ask in natural language; the skill triggers on:

- "优化我的论文标题" / "摘要改写" / "让 ChatGPT 引用我的论文" / "论文标题太长/太空"
- "abstract rewrite" / "paper title optimization" / "make my paper findable in AI search"
- "GEO for papers" / "arXiv abstract SEO" / "how do I get my paper cited by AI"
- "check if my method name is taken"

## Repository layout

```
paper-title-optimizer/
├── SKILL.md                     # six-stage procedure + five hard constraints
├── references/
│   ├── title-patterns.md        # L1 title rules (structure, first words, colons, numbers)
│   ├── title-numbers.md         # empirical study: numbers in 16,677 accepted titles
│   ├── title-numbers-stats.json # the data snapshot behind the study
│   ├── title-word-pool.csv      # accepted-title phrase pool (per-10k frequencies)
│   ├── abstract-architecture.md # sentence-position template for quotable abstracts
│   ├── layer2-coverage.md       # cross-vocabulary query coverage method
│   ├── layer3-terms.md          # term-gate rules + term pool schema
│   ├── constraints.md           # the five non-negotiables, with rationale
│   └── term-pool.csv            # arXiv trend seed pool (demo; probe refreshes live)
├── scripts/
│   ├── arxiv-trend.mjs          # per-year arXiv phrase counts, rising/peaked/stale classification
│   ├── visibility-probe.mjs     # baseline: can 8–12 queries retrieve this paper?
│   └── title-stats.mjs          # reproduce the title-number study on any paperlists file
└── LICENSE
```

## Honest boundaries

This skill improves the *probability* of being retrieved and quoted; it does not guarantee citations, and it will tell you so. It refuses to fabricate numbers, inflate claims, or stuff keywords (the one tactic measured to hurt). If a target requires a stronger claim than the evidence supports, the correct output is *"this paper needs another experiment"* — not a snappier title.

## 中文说明

一个给 Claude Code / Codex / ZCode 用的论文标题与摘要优化 skill：把已完成的论文改写成在 AI 检索管道（ChatGPT、Perplexity、Google AI Overviews）和文献索引（arXiv、Semantic Scholar）里容易被召回、被整句引用的版本。内置六阶段流程（可见度基线 → 证据映射 → 标题摘要改写 → 跨词汇覆盖 → 术语闸门 → 交付）和五条硬约束（不编数字、不抬 claim、不挂无关热词、正文只给定位建议、没测到就说没测到）。规则不是拍脑袋：分析了五大会 16,677 篇录用标题后写成的，全部可复现。

## License

MIT — see [LICENSE](LICENSE).
