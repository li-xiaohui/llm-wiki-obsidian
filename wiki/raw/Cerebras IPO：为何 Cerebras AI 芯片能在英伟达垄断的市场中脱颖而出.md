为何 Cerebras AI 芯片能在英伟达垄断的市场中脱颖而出：万字长文详解Cerebras技术设计

本文来自对Semianalysis 《Cerebras — Faster Tokens Please》一文的深度解读，聚焦Cerebras的底层技术，原报告 54 页，本文共 7775字。

今年最大的 IPO，Cerebras 正式登陆纳斯达克了！市场热度非常高！可以说是SpaceX之前最值得关注的IPO公司了。

这家公司股票代码 CBRS，IPO 定价为每股 185 美元，高于此前 150–160 美元的发行区间，发行 3000 万股，募资约 55.5 亿美元，完全摊薄估值约 564 亿美元。按照 Reuters 的报道，这是 2026 年迄今为止全球最大 IPO，订单需求超过发行股票数量的 20 倍，说明市场对 AI 基础设施资产的热情依然非常高。

Cerebras 受到关注，不只是因为它赶上了 AI IPO 情绪，更因为它走了一条和 NVIDIA GPU 完全不同的路线：把整片晶圆做成一颗芯片，用超大 SRAM 和极高内存带宽，专门服务 fast inference，也就是高速推理。它的核心赌注是，在模型智能达到一定门槛后，用户会愿意为更快的 token 生成速度付更高价格。

更关键的是，Cerebras 已经拿到了 OpenAI 这张超级大单：OpenAI 承诺购买 750MW AI 推理算力容量，并保留额外购买 1.25GW 的选择权，潜在总规模可达到 2GW。

但问题也在这里：Cerebras 真的能靠晶圆级芯片重塑 AI 推理市场吗？它的 SRAM 架构到底强在哪里，又为什么会在大模型、长上下文和高并发场景中遇到瓶颈？OpenAI 的大单到底是商业化拐点，还是把公司深度绑定在单一客户上的高风险赌注？

## 

Cerebras 的核心产品：晶圆级引擎（WSE）

Cerebras 的核心技术赌注，是突破单颗芯片的光刻掩模限制。不再把一片晶圆切割成很多颗独立芯片，而是把整片晶圆做成一颗巨型芯片。 这个设计试图解决两类问题：第一，摩尔定律放缓后，单颗芯片继续靠制程缩小获得算力提升越来越难；第二，传统芯片受 reticle limit，也就是光刻机单次曝光面积限制，单颗硅片面积通常不能无限扩大。文中提到的单个 reticle pattern 面积极限约为 858 平方毫米，而 Cerebras 直接绕过这个单芯片面积上限，把多个 reticle 曝光区域缝合成一整片晶圆级芯片。

这颗整片晶圆级芯片，就是 Cerebras 的核心产品：Wafer Scale Engine，简称 WSE，中文可以翻译为晶圆级引擎。

[

](https://x.com/xingpt/article/2055177203328139318/media/2055170076475813888)

Cerebras WSE-3 核心参数

- 晶体管数量：约 4 万亿
    
- 硅片面积：约 46,225 平方毫米
    
- 结构：12 列 × 7 行，共 84 个相同 reticle stepping / die 区块
    
- 片上 SRAM：44GB
    
- SRAM 带宽：约 21PB/s
    
- 片外 I/O 带宽：约 150GB/s
    
- 宣传口径稀疏 FP16 算力：125PFLOPS
    
- 更接近实际使用的 dense FP16 算力：约 15.625PFLOPS
    

WSE 由整片晶圆上的 84 个完全相同的光刻单元组成，呈 12 列 × 7 行网格排布。这些单元在普通晶圆上本来会被切割成独立芯片，但 Cerebras 通过跨划片槽布线，把它们连接成一整块硅片。每颗 WSE 中，约 50% 的硅片面积用于 SRAM 单元，剩余约 50% 用于计算核心、路由和其他逻辑。

这里的核心创新，是把计算单元和内存放在同一片硅上。传统 GPU、TPU 或其他 XPU 通常需要把计算芯片、HBM、封装基板、互联网络组合起来，数据经常要跨芯片、跨封装甚至跨节点移动。每一次移动都会消耗功耗、增加延迟并占用昂贵的互联资源。WSE 的目标是尽量让数据留在晶圆内部，用极高的片上 SRAM 带宽来减少外部数据搬运。

从小核心到巨型晶圆

层级拆解：核心（Core）→裸片（Die）→晶圆（WSE-3）

- 单核心：10700 个核心
    
- 单裸片：90 万个核心
    
- 整片晶圆：84 个裸片
    

[

](https://x.com/xingpt/article/2055177203328139318/media/2055170165021822976)

传统 GPU、XPU 想扩大总算力，通常要依赖先进封装和高速网络。例如 GPU 通过 HBM 获得高带宽内存，通过 NVLink、NVSwitch、InfiniBand 或以太网把多颗 GPU 连接起来。这个路径的好处是扩展灵活，坏处是数据搬运链路更长，系统复杂度和成本更高。Cerebras 的路径相反：它把尽可能多的计算和 SRAM 都塞进一片晶圆内部，用晶圆内部的 2D mesh fabric 完成数据流动。这个架构在片内数据流上非常强，但也会带来后文反复讨论的片外 I/O 瓶颈。

芯片规格对比（核心参数）：

[

](https://x.com/xingpt/article/2055177203328139318/media/2055170489166000128)

Cerebras 已迭代至第三代产品 WSE-3，采用台积电 5 纳米工艺制造。单颗 WSE-3 集成 44GB SRAM。这个数字对 SRAM 来说非常大。普通高端处理器的片上 SRAM 通常只有数十 MB 到数百 MB；即便 Groq 这类同样主打 SRAM 架构的 LPU，单芯片 SRAM 规模也远低于 WSE。文中提到 Groq LPU3 约 500MB SRAM，而 WSE-3 是 44GB，容量高出接近两个数量级。

SRAM 的优势是极高带宽和低延迟。WSE-3 的 44GB SRAM 可以提供约 21PB/s 带宽。PB/s 是 petabyte per second，也就是每秒拍字节级别带宽。相比 HBM 的 TB/s 级别带宽，SRAM 带宽高出很多。WSE-3 的带宽之所以极高，是因为整片晶圆上分布着大量 SRAM bank，多个存储单元的带宽可以在片内聚合。

但需要避免一个误解：WSE-3 的强项主要是内存带宽，不是单位面积算力。Cerebras 宣传 WSE-3 有 125PFLOPS FP16 算力，但这是稀疏算力口径，假设 8:1 非结构化稀疏。换成 dense FP16 口径，实际约为 15.625PFLOPS。这个数字绝对值不低，但考虑到 WSE-3 用了整片晶圆，单位硅片面积算力并不突出。原因在于 WSE 的单个核心刻意设计得较小，这有利于良率修复和故障绕行，但也降低了类似 GPU 大型矩阵阵列那样的面积计算密度。

WSE 的最大短板是片外网络带宽。单片 WSE 对外 I/O 约 150GB/s，远低于 NVIDIA GPU 的 scale-up 带宽，也远低于 WSE 自身的片上 SRAM 带宽。这个差距很关键：在单片晶圆内部跑低算术强度 decode 时，WSE 很强；一旦模型太大、上下文太长、需要多片晶圆协作，数据就必须进出晶圆，150GB/s 会很快成为硬瓶颈。后文的流水线并行、I/O 岛屿问题和光互连方案，都是围绕这个瓶颈展开。

## 

SRAM 架构芯片

WSE 最突出的优势是SRAM 容量—— 与 Groq LPU 一致，WSE 属于SRAM 架构加速器：将更多硅片面积用于超高速 SRAM，作为模型权重、键值缓存（KV Cache）的主存储。

主流 GPU、ASIC（TPU、Trainium）则采用HBM 存储权重与 KV Cache—— 虽也集成 SRAM，但容量远小于 WSE。

SRAM 替代 HBM 的利弊：

- 优势：带宽更高、延迟更低、单用户 token 输出速度更快
    
- 劣势：容量更小、单位 bit 成本更高、单位瓦和单位美元可承载的总内存更低
    
- 结果：更适合低 batch、高交互、decode-heavy 场景；较不适合超大模型、超长上下文、高并发吞吐场景
    

可以用一个简单类比理解：SRAM 像一辆极快的跑车，响应速度非常快，但载客量有限；HBM 像一辆大巴，速度不一定极限，但可以承载更多乘客，也更适合规模化运营。对 AI 推理服务商来说，单用户最快速度很重要，但总吞吐、并发能力和每百万 token 成本往往同样重要。

HBM、DDR5、GDDR7、LPU SRAM 对比

- HBM：GPU/TPU 的核心高带宽内存，单 stack 容量可达数十 GB，多 stack 聚合后单封装容量可达数百 GB
    
- DDR5：容量大、成本相对低，适合 CPU 主存和 KV Cache offload，但带宽和延迟不如 HBM/SRAM
    
- GDDR7：常见于图形和部分加速卡，带宽高于 DDR，但系统形态和容量扩展方式不同于 HBM
    
- LPU SRAM：Groq 等 SRAM machine 使用的片上高速 SRAM，单芯片容量较小，但延迟和带宽优秀
    
- WSE SRAM：单片 WSE-3 达到 44GB，是 SRAM 口径下的极大容量，但和多 stack HBM GPU 相比仍然偏小
    

[

](https://x.com/xingpt/article/2055177203328139318/media/2055170605243408384)

尽管 WSE-3 的44GB SRAM远超普通芯片，但仅略高于单颗HBM3E 12-Hi 堆叠（36GB）。而当前主流加速器普遍采用8 堆叠 HBM，单 GPU/TPU 封装容量达288GB—— 是 WSE SRAM 容量的 6.5 倍。为何 Cerebras AI 芯片能在英伟达垄断的市场中脱颖而出

业内对 DRAM 的需求激增，核心原因是 AI 系统设计追求最大化内存容量—— 充足内存可实现：

1. 容纳更大模型（更多参数）
    
2. 服务更多并发请求（更多用户，更大 KV Cache）
    
3. 支持更长上下文窗口（单请求更长序列，更大 KV Cache）
    

推理服务商的核心竞争力，正是上述三点 —— 这也是 GPU 内存容量持续提升的原因。更重要的是，内存容量不受限于单颗封装：工作负载可跨芯片分片，通过扩展架构聚合全局内存。因此，网络带宽成为所有 AI 硬件厂商的核心竞争战场 ——Cerebras 除外，其主动接受了低网络带宽的权衡，并围绕该限制设计产品。

受限于单片晶圆内存容量，Cerebras 扩展多晶圆集群的空间极小。低 I/O 带宽虽非致命缺陷，但仍是 WSE-3 设计的核心短板，制约其业务爆发式增长。

即便如此，Cerebras 已步入高速增长通道——OpenAI 合作是关键转折点：2026-2028 年，Cerebras 需交付的服务器总量，将超过成立以来的总和。订单激增已体现在台积电晶圆产能上：2026 年起，台积电每季度为 OpenAI 订单提升 Cerebras 晶圆产能。预计未来几年，Cerebras 营收将迎来爆发式增长，OpenAI 是唯一核心增长引擎。

推理服务商的核心竞争力，正是上述三点。更大的模型、更长的上下文和更多并发，都会直接消耗内存容量。GPU 内存容量持续提升，不是偶然，而是推理经济性的必然结果。

更重要的是，HBM 系统的内存容量不受限于单颗封装。一个大模型可以切分到多颗 GPU 上，通过 NVLink、NVSwitch 或其他 scale-up fabric 把多颗 GPU 的 HBM 聚合使用。这样一来，网络带宽就成为 AI 硬件竞争的核心战场。NVIDIA、Google、AWS、AMD 都在通过芯片间互联提高可扩展性。Cerebras 的路线更特殊，它主动接受较低片外 I/O，把重点放在片内 SRAM 和片内数据流上。

受限于单片晶圆的内存容量，Cerebras 想通过多片晶圆扩展大模型时，空间会明显受限。低 I/O 带宽并非致命缺陷，因为特定模型和特定 workload 仍然可以很好地匹配 WSE。但它确实是 WSE-3 的核心短板，也限制了 Cerebras 从快 token 走向大规模通用推理平台的速度。

即便如此，Cerebras 已经进入高速增长通道。OpenAI 合作是关键转折点。2026 到 2028 年，Cerebras 需要交付的服务器总量，可能超过公司成立以来交付总量的一个数量级。订单激增已经反映在台积电晶圆投片需求中，Cerebras 需要逐季提高 WSE 产出，以满足 OpenAI 部署节奏。 未来几年，Cerebras 营收大概率会快速放大，而 OpenAI 是最核心增长引擎。

## 

Cerebras 核心自研技术

能走到今天，Cerebras 攻克了从硅片、系统到软件的全链路技术难题。相比多数初创加速器厂商，Cerebras 拥有大量独家硬件技术，晶圆级芯片的大胆设计，是行业巨头难以复制的壁垒。

核心自研技术

1. 跨裸片布线与路由：利用晶圆划片槽作为晶圆内数据总线，连接所有裸片。普通晶圆的划片槽是切割区域，用于将晶圆分割为单颗芯片。
    
2. 冗余设计与故障路由：晶圆级芯片良率极低（接近光刻掩模尺寸的芯片良率普遍低于 50%），核心冗余 + 故障路由是量产关键。WSE 集成97 万个核心，启用 90 万个；单颗核心刻意缩小尺寸，提升良率。 量产难点：每批次晶圆定制上层金属掩模—— 针对批次内缺陷区域，定制专属布线。单批次掩模成本大幅增加，接近台积电晶圆成本。 原因：批次内工艺差异远小于批次间，定制掩模可最大化晶圆良率。 结果：晶圆级良率接近 100%，台积电生产的晶圆几乎全部可组装为量产服务器。
    
3. 供电与散热系统：核心难题是单颗晶圆输入 25 千瓦电力（下一代功率更高）。定制化供电方案由 Vicor 提供，25 千瓦电力最终转化为热量，需专用散热系统 ——CS 服务器的供电 + 散热组件，称为引擎模块（Engine Block），是 Cerebras 独有的核心组件。
    

[

](https://x.com/xingpt/article/2055177203328139318/media/2055170934101913600)

尽管技术成就卓越，WSE 架构仍存在三大技术瓶颈，制约技术路线图与服务能力：

散热设计与冷却难题

核心挑战：46225 平方毫米晶圆散热 25 千瓦，平均热流密度50 瓦 / 平方厘米（不含热点）。

- 风冷淘汰：3D 均热板方案（类似 H100 服务器）扩展至 21.5 厘米晶圆后，超出毛细极限，冷却液无法回流，失效。
    
- CS-3 散热方案：定制液冷堆叠架构，与英伟达标准单相对流直触芯片方案完全不同。
    
- 散热定制化：全定制散热方案，与晶圆协同设计。硅片与下方 PCB 受热膨胀率不同，21.5×21.5 厘米晶圆的膨胀差足以导致封装冷板、晶圆 - PCB 连接器、组装工具开裂。因此，冷板、连接器、组装工具均需从零设计。
    
- 引擎模块结构：四层堆叠 —— 冷板、晶圆、柔性连接器、PCB；冷却歧管贴合冷板背面。
    

[

](https://x.com/xingpt/article/2055177203328139318/media/2055170985079488512)

机架级散热瓶颈

流量差异：

- 英伟达 GB200 NVL72：1.5 升 / 分钟 / 千瓦
    
- Cerebras WSE-3：4 升 / 分钟 / 千瓦（25 千瓦≈100 升 / 分钟）
    
- 影响：需更大水泵、更粗管道、超大冷却分配单元（CDU）、高流量快速接头。
    
- 下一代规划：CS-4 目标将流量降至1.5-1.7 升 / 分钟 / 千瓦，适配行业标准基础设施。
    

冷却合作伙伴与进水温度

- 合作伙伴：LiquidStack（2026 年 3 月被特灵科技收购），联合开发单相对流 CDU，适配 CS-3 流量与压力需求。
    
- 进水温度：Cerebras 俄克拉荷马工厂采用5℃冷水机组，换热后21℃进入引擎模块；英伟达 NVL72 支持45℃进水，可全年自然冷却。Cerebras 需依赖冷水机组，设施成本更高。
    

## 

晶圆级芯片的制胜场景

要理解 Cerebras 的极致内存带宽优势，需从LLM 推理性能优化视角分析 —— 芯片本质是工具，核心指标是算术强度（FLOPs / 字节，每传输 1 字节数据可执行的浮点运算次数），直接决定芯片性能上限。

[

](https://x.com/xingpt/article/2055177203328139318/media/2055171140403044352)

核心结论

- WSE-3 最优场景：低算术强度（AI<10）、内存带宽受限的解码任务（批量 = 1、单用户），此时可发挥21PB/s 带宽优势，算力上限达15.625 PFLOPS。
    
- HBM 芯片最优场景：高算术强度（AI>1000）、算力受限的预填充、批量解码任务。
    
- 硬件 - 软件协同：未来适配 WSE-3 的模型（如 GPT-5.3-Codex-Spark/gpt-oss-120B），将针对性优化低算术强度算子，实现性能最大化。
    

## 

Cerabas的瓶颈

WSE 架构给了 Cerebras 极高 SRAM 带宽和 fast token 能力，但同时牺牲了内存容量、联网扩展性和大模型长上下文吞吐能力。

Cerebras 的 SRAM 带宽极高，所以单用户快速生成 token 很强。但 SRAM 容量小、单位容量贵。WSE-3 只有 44GB SRAM，一旦遇到大模型、长上下文、高并发，模型权重和 KV Cache 很快会把 SRAM 吃满。

HBM GPU 的逻辑相反。单用户极速 decode 可能不如 Cerebras，但 HBM 容量大得多，可以容纳更大模型、更多 KV Cache、更长上下文和更多并发用户。对推理服务商来说，总吞吐和每百万 token 成本往往比单用户极限速度更重要。

Cerebras 最大的优势来自 SRAM，但 SRAM 的制程缩放已经接近停滞，这会让它未来的容量扩展越来越困难。

WSE-1 到 WSE-2，SRAM 从 18GB 增加到 40GB，提升很大；但 WSE-2 到 WSE-3，SRAM 只从 40GB 增加到 44GB。这个变化说明，先进制程继续推进时，逻辑晶体管还能增加，但 SRAM 容量很难同步提升。

这对 Cerebras 特别关键，因为它的商业价值很大程度来自“把大量 SRAM 放在一整片晶圆上”。如果 SRAM 密度不再明显提高，Cerebras 想增加片上内存，主要只能继续牺牲计算面积，或者走更复杂的 3D 堆叠和 wafer-on-wafer bonding。

相比之下，Groq 这种较小芯片可以通过封装层面的 Z 方向堆叠增加 SRAM tile，灵活性更高。Cerebras 也可以探索类似方向，但把整片晶圆级芯片再与另一片存储晶圆键合，技术复杂度远高于普通芯片堆叠。

这一节后半段进一步引出另一个大问题：I/O。

WSE 的片上带宽巨大，但晶圆外带宽只有 150GB/s。对单片晶圆内部的数据流来说，这可能够用；但一旦模型太大，需要多片晶圆协同，或者需要频繁把数据搬进搬出 WSE，off-wafer bandwidth 就会成为硬瓶颈。

WSE 为什么对外 I/O 很难做大？

根本原因不是 Cerebras 没有意识到 I/O 重要性，而是 wafer-scale 架构本身有几何约束。WSE 的成立依赖重复一致的 reticle pattern。每个 reticle 都必须具有相同逻辑、相同内存、相同布线和相同边缘 pin assignment。只有这样，相邻 die 的东西南北边缘才能对齐，跨 die 2D mesh fabric 才能稳定工作。

但 I/O 的需求天然不均匀。只有晶圆边缘能直接连到外部世界，中间区域即使放 SerDes，也很难真正把信号引出晶圆。如果只在外围 reticle 放 PHY，会破坏所有 reticle 必须相同的要求；如果在每个 reticle 都放 PHY，大量 PHY 会被困在晶圆内部，既占面积，又无法对外通信。

此外，高速 PHY/SerDes 本身面积大、功耗高，还包含模拟电路。它们会带来供电噪声和电磁干扰，需要 guard region 进行隔离。把大量 PHY 放入 reticle 内部，会在 2D mesh fabric 中挖出很多“洞”，迫使片内互连绕路，增加延迟并降低有效带宽。

这就是“带宽是几何问题”的含义。WSE 像一个内部交通极强、边境口岸很少的岛。岛内 2D mesh 和 SRAM 带宽极强，但进出岛的数据通道有限。这会限制多晶圆协同、模型分片、KV Cache 迁移和外部系统解耦。

Cerebras 的潜在解法是 photonic interconnect wafer，也就是光互连晶圆。通过 hybrid bonding 把光互连晶圆接到 WSE 上，让数据从 Z 方向进出，而不只依赖晶圆边缘。这个方向很大胆，但技术风险也很高，因为它同时涉及 wafer-on-wafer bonding、热管理、光纤耦合、CPO 可靠性和光器件温度敏感性。

所以这一节的核心判断是：Cerebras 的 I/O 瓶颈不是普通工程堆料问题，而是 wafer-scale 的结构性几何问题。要真正突破，需要 3D/光互连这种级别的架构跃迁。

[

](https://x.com/xingpt/article/2055177203328139318/media/2055171302676398080)

## 

流水线并行方案的必然性

Cerebras 的低 I/O 和有限 SRAM 让它在大模型推理中几乎只能选择 pipeline parallelism。

如果一个大模型的 weights 放不进单片 WSE-3 的 44GB SRAM，就必须拆到多片晶圆上。由于 CS-3 对外 I/O 只有约 150GB/s，不能像 GPU 集群那样频繁做高带宽 collective，也不能把大 tensor 来回 streaming。所以最现实的做法是按层切模型，让每片晶圆保留自己负责的权重，只在 stage 之间传较小的 activation。

但这会带来新的成本。晶圆数量越多，pipeline stage 越多，需要更多 in-flight microbatch 才能填满流水线；每个 microbatch 又需要自己的 KV Cache，会进一步消耗 SRAM。模型越大，晶圆间 activation 传输次数越多，decode 延迟也越容易累积。

所以说，这种生产环境用法和晶圆的设计初衷有冲突。Cerebras 最强的场景是小 batch、低延迟、fast token。但为了跑大模型，它被迫用 pipeline、多 microbatch 和多晶圆切分，这会削弱它最擅长的低 batch 高交互优势。

## 

大模型参数测算

[

](https://x.com/xingpt/article/2055177203328139318/media/2055171344267169792)

从表格中可以看出，最近的 KV Cache 压缩技术，比如 DeepSeek 发表的相关技术，可能会显著缓解 Cerebras 在长上下文服务中的问题。

不过，慢 I/O 的问题并没有完全消失。

第一，KV Cache 在芯片上和芯片外之间的搬运时间仍然相当大，通常是数毫秒级别。

这会影响 TTFT，也就是 first token 生成时间。

同时，它也会让系统更难实现高利用率，因为 KV Cache 的存储和传输会牵涉 batching、pipelining 和 latency hiding 等问题。

第二，activation transfer 的固定 I/O 延迟必须按照承载一个模型实例所使用的晶圆数量来支付。

这会成为 TPOT 中的固定开销，并且会随着承载该模型所需晶圆数量线性增长。

关键结论是：Cerebras 虽然很快，但在数据进出晶圆时要付出很大的延迟成本。

因此，它们的 cost-to-performance ratio，也就是成本性能比，或者 perf per Joule，也就是每焦耳性能，取决于它们能在多大程度上隐藏或最小化这部分延迟。

这个问题在实践中的难度，可能可以从 Cerebras Inference Cloud 上提供的模型列表里看出一些线索。

目前最大的生产模型是 GPT-OSS，总参数量只有 120B。

Cerebras 的真正机会来自“模型架构与硬件特性匹配”。如果未来模型像 DeepSeek V4-Pro 一样，使用 MoE、低 active params、强 KV 压缩、足够并发，那么 WSE 的 fast decode 价值会被放大；如果模型是高参数 dense、长上下文、高 KV 压力，HBM GPU 集群仍然更有优势

## 

Cerebras的 OpenAI Deal细节

OpenAI 交易几乎重塑了 Cerebras 的商业前景，但也让 Cerebras 对单一大客户高度绑定。

OpenAI 通过三层机制和 Cerebras 绑定：第一是 750MW 推理算力采购协议，并且有扩展到 2GW 的选择权；第二是 10 亿美元营运资金贷款；第三是几乎免费行权的 warrant。对 Cerebras 来说，这笔交易带来巨额 backlog、融资支持和 IPO 叙事。对 OpenAI 来说，它获得了 fast inference capacity 和潜在股权价值获利。

但这不是一笔简单的客户采购。OpenAI 既是客户，也是债权人，也是潜在大股东。Cerebras 的未来收入、偿债安排、股权稀释和数据中心扩张，都和 OpenAI 的需求绑定。关系成功，Cerebras 收入爆发；关系出问题，贷款可能提前到期，执行压力会非常大。

技术上，OpenAI 真正买的是 fast token 能力，特别是像 GPT-5.3-Codex-Spark 这种 120B 蒸馏模型在 Cerebras 上可以跑到极高交互速度。但文章提醒，Spark 不是完整 GPT-5.3-Codex，而是小得多的蒸馏模型。Cerebras 当前更适合服务高速度、小到中等规模模型，而不是直接承载最前沿、超大参数、超长上下文模型。

投资逻辑上，这一节的关键判断是：Cerebras 的上行空间取决于 OpenAI 是否认为“稍微落后一代但非常快的 token”值得长期溢价付费。如果 120B 级别模型很快能接近 GPT-5.5 质量，那么 Cerebras 的 fast token 市场可能成立；如果用户只愿意为最强模型付费，Cerebras 的价值就会受限于模型容量和长上下文瓶颈。

具体的交易细节如下：

[

](https://x.com/xingpt/article/2055177203328139318/media/2055171392321314816)

[

](https://x.com/xingpt/article/2055177203328139318/media/2055171432267870208)

## 

与OpenAI交易的收益分析

这一节其实在回答两个问题：Cerebras 赚不赚钱，OpenAI 买得划不划算。

对 Cerebras 来说，这笔交易看起来非常好。CS-3 每小时 TCO 约 23.35 美元，而 OpenAI 隐含租金是每小时 41.96 美元。中间的差额意味着 Cerebras 可以获得很高项目回报率。更关键的是，Cerebras 用的是自己的硬件，利润不需要像 GPU 云厂商那样大量流向 NVIDIA。

[

](https://x.com/xingpt/article/2055177203328139318/media/2055171602632097792)

这也是 Cerebras 和普通 GPU neocloud 的根本差别。GPU 云厂商的核心成本是买 NVIDIA GPU，NVIDIA 吃掉硬件毛利；Cerebras 自己设计并托管 CS-3，如果系统成本控制得住，租金收益就会更多留在自己账上。

对 OpenAI 来说，经济性没有那么夸张。5.3 Codex-Spark 在 agentic coding workload 下，估算每百万 token 成本 0.19 美元，收入 0.30 美元，推理毛利率约 35%。这个水平可以接受，但不算特别高，尤其和真正前沿模型的高价 token 相比。

所以 OpenAI 后续要让这笔交易更有价值，有两条路：一是提高收入端，让更高质量的小模型在 Cerebras 上跑出 fast token，并收取溢价；二是降低成本端，通过模型架构和 Cerebras 硬件协同优化，让 workload 更贴合 WSE 的低算术强度优势。

因此，OpenAI 交易对 Cerebras 是一个高 IRR、高确定性的大订单；对 OpenAI 则是一次 fast inference 的战略下注。它是否真正划算，取决于 Cerebras 上能跑的模型质量，以及 OpenAI 能否把 fast token 卖出足够高的溢价。

## 

AWS：OpenAI之外的其他可能性

虽然 OpenAI 是 Cerebras 最重要的发展，但 Amazon 在 2026 年 3 月宣布与 Cerebras 建立的合作关系，将为 Cerebras 提供另一条增长路径。

OpenAI 交易是 Cerebras 自己托管 CS-3，然后把算力容量卖给 OpenAI； AWS 会把 WSE 部署到自己的数据中心，用来支持 Amazon Bedrock 的推理服务。两者的收入确认和商业模式不同：OpenAI 偏云租赁收入，AWS 偏硬件销售收入。

作为合作的一部分，WSE 将用于 decode，而 Trainium 将用于 prefill 节点。

技术上，AWS 方案非常符合前文对 Cerebras 的判断：Cerebras 最适合 decode，因为 decode 受内存带宽和延迟约束；Trainium 更适合 prefill，因为 prefill 更偏计算密集。把两者拆开，各自做擅长的环节，就是典型 PD disagg 架构。

值得注意的是，到目前为止，Trainium（Trainium：AWS 自研 AI 训练/推理芯片） 最大的客户是 Anthropic，也就是通过 Project Rainier 使用 Trainium。

由于 Anthropic 是 Trainium 最大客户，而 Claude fast mode 需求可能继续增长，Anthropic 将来很可能也会通过 AWS Bedrock 间接或直接使用 Trainium + Cerebras 的组合

无论如何，Cerebras 很可能都会通过 Bedrock 服务参与 Claude token 的服务。