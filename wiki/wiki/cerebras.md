# Cerebras

**Summary**: Cerebras Systems is an AI hardware company built around wafer-scale computing. It IPO'd on Nasdaq in May 2026 at ~$56B valuation, anchored by a landmark OpenAI deal for up to 2GW of inference capacity. Its WSE-3 chip excels at fast token generation but faces structural I/O and memory capacity limitations.

**Sources**: Cerebras IPO Ushering in a New Era of AI Hardware.md, Cerebras IPO：为何 Cerebras AI 芯片能在英伟达垄断的市场中脱颖而出.md

**Last updated**: 2026-05-16

---

## Company Overview

Cerebras Systems, founded in 2015 in Sunnyvale, California, builds computer systems for complex AI deep learning applications. The company was founded by Andrew Feldman (CEO), Gary Lauterbach, Michael James, Sean Lie, and Jean-Philippe Fricker. (source: Cerebras IPO Ushering in a New Era of AI Hardware.md)

## Wafer Scale Engine Technology

Cerebras's core innovation is its proprietary Wafer Scale Engine (WSE). Unlike conventional chips cut from silicon wafers, the WSE uses nearly an entire 300mm wafer to make a single chip. Rather than cutting a wafer into many individual dies, Cerebras stitches all dies together via cross-scribe-line wiring, creating one monolithic chip from 84 identical reticle blocks arranged in a 12x7 grid. (source: Cerebras IPO：为何 Cerebras AI 芯片能在英伟达垄断的市场中脱颖而出.md)

### WSE-3 Specifications

| Spec | Value | Context |
|------|-------|---------|
| Transistors | ~4 trillion | |
| Die area | ~46,225 mm² | Full 300mm wafer |
| Structure | 84 identical reticle blocks (12x7) | |
| On-chip SRAM | 44GB | ~50% of silicon area |
| SRAM bandwidth | ~21 PB/s | vs. HBM at TB/s level |
| Off-chip I/O | ~150 GB/s | Key bottleneck |
| Sparse FP16 | 125 PFLOPS | Marketing figure, 8:1 sparsity |
| Dense FP16 | ~15.625 PFLOPS | Actual usable compute |
| Cores | 900,000 active (970,000 total) | Small cores for yield |
| Power | 25 kW per wafer | |

(source: Cerebras IPO：为何 Cerebras AI 芯片能在英伟达垄断的市场中脱颖而出.md)

### SRAM vs HBM Architecture

Cerebras is an SRAM-architecture accelerator — it uses massive on-chip SRAM (44GB) as primary storage for model weights and KV cache, achieving 21 PB/s bandwidth. This is fundamentally different from GPU/TPU designs that use HBM.

**Advantages**: Much higher bandwidth, lower latency, faster per-user token generation
**Disadvantages**: Lower total capacity (44GB vs 288GB per GPU with 8x HBM stacks), higher cost per bit, limited scalability

Best analogy: SRAM is a sports car (extremely fast, limited passengers); HBM is a bus (not as fast, but carries many more passengers and is better for scale operations). (source: Cerebras IPO：为何 Cerebras AI 芯片能在英伟达垄断的市场中脱颖而出.md)

### Critical Bottleneck: Off-Chip I/O

The WSE's off-chip bandwidth of ~150 GB/s is the structural weakness. This is a geometric constraint, not an engineering problem:

- Only wafer-edge reticles can connect to the outside world
- All 84 reticles must be identical (required for cross-die mesh fabric), so I/O cannot be added selectively
- High-speed SerDes/PHY blocks are large, power-hungry, and create electromagnetic interference "holes" in the 2D mesh
- The wafer is like "an island with incredibly strong internal traffic but very few border crossings"

Potential solution: photonic interconnect wafer bonded to the WSE via hybrid bonding, enabling data to exit through the Z-axis rather than only wafer edges. This is technically high-risk. (source: Cerebras IPO：为何 Cerebras AI 芯片能在英伟达垄断的市场中脱颖而出.md)

### SRAM Scaling Problem

SRAM density scaling is stalling:
- WSE-1 → WSE-2: 18GB → 40GB (large jump)
- WSE-2 → WSE-3: 40GB → 44GB (near-stagnation)

As process nodes advance, logic transistor density improves but SRAM density does not keep pace. This limits Cerebras's future ability to increase on-chip memory without sacrificing compute area. (source: Cerebras IPO：为何 Cerebras AI 芯片能在英伟达垄断的市场中脱颖而出.md)

## Proprietary Engineering

Key self-developed technologies:

1. **Cross-die wiring**: Uses wafer scribe lanes as data buses connecting all 84 dies
2. **Redundancy & fault routing**: 970K total cores, 900K active. Custom upper-metal masks per wafer batch to route around defects — nearly 100% wafer-level yield
3. **Power & cooling**: 25kW per wafer via custom Vicor power delivery. Custom liquid cooling at 5°C inlet (vs. Nvidia NVL72 supports 45°C natural cooling). Four-layer stack: cold plate, wafer, flex connector, PCB. Coolant flow 4L/min/kW (vs. Nvidia 1.5L/min/kW)

(source: Cerebras IPO：为何 Cerebras AI 芯片能在英伟达垄断的市场中脱颖而出.md)

## Optimal Use Cases

WSE-3 excels at low arithmetic intensity, memory-bandwidth-bound decode tasks (batch=1, single user). It achieves 21 PB/s bandwidth advantage for fast token generation on small-to-medium models.

**Best for**: Low-batch, high-interactivity, decode-heavy inference (e.g., coding assistants, conversational AI)
**Worst for**: Ultra-large models, long contexts, high-concurrency throughput (where HBM GPU clusters dominate)

Largest production model currently: ~120B parameters (GPT-OSS). (source: Cerebras IPO：为何 Cerebras AI 芯片能在英伟达垄断的市场中脱颖而出.md)

## Customers & Proven Deployments

### Tier 1 — Production at Scale

- **OpenAI**: Running GPT-5.3-Codex-Spark (120B distilled model) for fast inference. 750MW committed with option to 2GW. Accounts for ~86% of revenue. (source: Cerebras IPO：为何 Cerebras AI 芯片能在英伟达垄断的市场中脱颖而出.md)
- **AWS Bedrock** (March 2026): WSE deployed in AWS data centers as decode engine, paired with Trainium for prefill. Anthropic/Claude likely to use this path for fast inference. (source: Cerebras IPO：为何 Cerebras AI 芯片能在英伟达垄断的市场中脱颖而出.md)
- **Meta**: Powers the Llama API — Llama 4 on Cerebras generates tokens up to 18x faster than GPU alternatives. (source: web research, Cerebras company filings)

### Tier 2 — Enterprise & Research

- **GlaxoSmithKline**: Epigenomics research using Cerebras systems. (source: web research)
- **AstraZeneca**: Reduced training time from 2 weeks on GPU cluster to 2 days on CS-1. (source: web research)
- **Mayo Clinic**: Medical diagnostics and personalized medicine applications. (source: web research)
- **TotalEnergies**: Energy sector AI applications. (source: web research)
- **G42 (UAE)**: "Condor Galaxy" AI supercomputer network — sovereign AI deployment. (source: web research)

### Tier 3 — Fragmented

Government, enterprise, and cloud customers making up ~14% of revenue. (source: web research, S-1 filing)

### Customer Concentration Risk

Cerebras is heavily concentrated on OpenAI. The other customers are real but small. This is the company's primary business risk — OpenAI is simultaneously customer, creditor, and potential equity holder.

### Proven Sweet Spots

1. **Fast inference on distilled/medium models** (120B class) — coding assistants, conversational AI
2. **Scientific computing** where training time reduction matters — pharma, energy
3. **Sovereign AI** where physical footprint and data locality matter (G42/UAE)

### Not Yet Proven

- Running frontier-scale models (1T+ params)
- Long-context workloads
- High-concurrency production serving at GPU-competitive cost
- The 44GB SRAM cap and 150GB/s I/O bottleneck are structural constraints limiting general-purpose viability

## IPO (May 2026)

Cerebras began trading on the Nasdaq on May 15, 2026 under ticker CBRS:

- Priced at $185/share, opened at $350/share
- Raised $5.55 billion
- Implied valuation: ~$56.43 billion (fully diluted)
- Offering oversubscribed ~20x
- Largest global IPO in 2026 to date (pre-SpaceX)

(source: Cerebras IPO Ushering in a New Era of AI Hardware.md)

## Financial Performance

In 2025, Cerebras reported $510 million total revenue (up 76% YoY), swinging to $237.8 million net income from a net loss of $481.6 million prior year. (source: Cerebras IPO Ushering in a New Era of AI Hardware.md)

## OpenAI Deal (Transformative)

OpenAI has a three-layered relationship with Cerebras:

1. **Capacity purchase**: 750MW inference compute, with option to expand to 1.25GW additional (up to 2GW total)
2. **Working capital loan**: $1 billion from OpenAI to Cerebras
3. **Warrants**: Near-free exercise price — OpenAI is a potential major shareholder

**Economics**:
- CS-3 TCO: ~$23.35/hour
- OpenAI implied rent: ~$41.96/hour
- High project IRR for Cerebras (unlike GPU neoclouds where Nvidia captures hardware margin)
- OpenAI running GPT-5.3-Codex-Spark (a 120B distilled model, not the full GPT-5.3)
- Estimated cost per million tokens: $0.19, revenue: $0.30, ~35% inference gross margin for OpenAI

**Risk**: OpenAI is simultaneously customer, creditor, and potential equity holder. Revenue, debt repayment, and equity dilution are all tied to one relationship. (source: Cerebras IPO：为何 Cerebras AI 芯片能在英伟达垄断的市场中脱颖而出.md)

## AWS Partnership (March 2026)

- WSE deployed in AWS data centers for Amazon Bedrock inference
- Architecture: Cerebras WSE for decode, AWS Trainium for prefill (PD disaggregation)
- Revenue model: hardware sales (vs. OpenAI which is cloud rental)
- Anthropic (Trainium's largest customer via Project Rainier) likely to use Cerebras via Bedrock for Claude fast inference

(source: Cerebras IPO：为何 Cerebras AI 芯片能在英伟达垄断的市场中脱颖而出.md)

## Related pages

- [[ai-hardware]]
- [[nvidia-china]]
- [[thesis-nvidia]]
