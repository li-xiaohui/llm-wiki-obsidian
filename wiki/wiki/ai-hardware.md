# AI Hardware

**Summary**: The AI hardware landscape in 2026 features intense competition between Nvidia (dominant in GPUs), Cerebras (wafer-scale alternative), and Huawei (China's domestic champion), with geopolitics increasingly shaping market access.

**Sources**: Cerebras IPO Ushering in a New Era of AI Hardware.md, Cerebras IPO：为何 Cerebras AI 芯片能在英伟达垄断的市场中脱颖而出.md, Nvidia's Future in China Remains Unclear After Trump-Xi Summit.md, South Korea's world-beating stock rally stumbles as global funds sell.md

**Last updated**: 2026-05-16

---

## Key Players

### Nvidia
The world's leading AI chip maker. Its H200 chip was approved for sale to China in December 2025 but no sales have materialized. Nvidia CEO Jensen Huang has warned that China's shift to domestic hardware will erode US influence over AI development. (source: Nvidia's Future in China Remains Unclear After Trump-Xi Summit.md)

### Cerebras
Went public in May 2026 at a ~$56B valuation. Its Wafer Scale Engine (WSE-3) uses an entirely different architecture from GPUs: a full-wafer SRAM chip with 44GB on-chip memory at 21 PB/s bandwidth. Marketing claims 125 PFLOPS (sparse); actual dense FP16 is ~15.6 PFLOPS. Excels at fast single-user token generation (decode) but limited by 150 GB/s off-chip I/O and 44GB SRAM capacity — largest production model is only 120B params. Anchored by an OpenAI deal (750MW + 1.25GW option) and an AWS Bedrock partnership (WSE for decode, Trainium for prefill). (source: Cerebras IPO Ushering in a New Era of AI Hardware.md, Cerebras IPO：为何 Cerebras AI 芯片能在英伟达垄断的市场中脱颖而出.md)

### Huawei
China's domestic chip champion. DeepSeek announced in May 2026 that its latest AI model was optimized to run on Huawei chips — a milestone in China's push for technological self-sufficiency. Beijing is actively directing Chinese companies to use Huawei over Nvidia. (source: Nvidia's Future in China Remains Unclear After Trump-Xi Summit.md)

### Samsung & SK Hynix
Dominant in memory semiconductors (HBM for AI). Together they account for nearly half the Kospi Index and drove two-thirds of its near-90% rally in 2025-2026. Their stock performance is a proxy for global AI hardware demand. (source: South Korea's world-beating stock rally stumbles as global funds sell.md)

## Architecture Paradigms: SRAM vs HBM

The AI hardware market is splitting into two memory architecture approaches:

| | SRAM (Cerebras, Groq) | HBM (Nvidia, AMD, Google TPU) |
|---|---|---|
| Bandwidth | 21 PB/s (WSE-3) | ~10 TB/s (B200) |
| Capacity | 44GB (WSE-3) | 288GB (8x HBM3E) |
| Best for | Low-batch fast decode | Large models, long context, high throughput |
| Cost/bit | Very high | Lower |
| Scalability | Limited by off-chip I/O | Scale-out via NVLink/InfiniBand |

The market is evolving toward prefill/decode disaggregation — using HBM chips for compute-heavy prefill and SRAM chips for bandwidth-bound decode. The AWS Bedrock architecture (Trainium prefill + Cerebras decode) is an early example. (source: Cerebras IPO：为何 Cerebras AI 芯片能在英伟达垄断的市场中脱颖而出.md)

## Geopolitical Dimensions

AI hardware has become a key vector of US-China competition:

- US export controls aim to slow China's AI progress
- China responds with self-sufficiency push and rare earth export controls
- The market is bifurcating into US-allied and China-domestic ecosystems
- Nvidia faces the risk of permanently losing the China market

(source: Nvidia's Future in China Remains Unclear After Trump-Xi Summit.md)

## Related pages

- [[cerebras]]
- [[nvidia-china]]
- [[china-chip-self-sufficiency]]
- [[south-korea-stock-market]]
