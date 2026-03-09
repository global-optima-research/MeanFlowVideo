MeanFlow Update：

**DisCa: Accelerating Video Diffusion Transformers with Distillation-Compatible Learnable Feature Caching（Tencent Hunyuan + SJTU, arXiv:2602.05449）**

针对大型视频DiT（如HunyuanVideo），提出**Distillation-Compatible Learnable Feature Caching (DisCa)**。传统training-free特征缓存（TeaCache、PAB等）在step-distilled模型上失效（因为步数稀疏导致相邻步特征差异巨大）。他们用**轻量可学习神经网络predictor**（代替手工Taylor展开）来预测高维特征演化。

同时，针对MeanFlow在视频大模型上蒸馏不稳定（JVP数值误差 + 过于aggressive），提出**Restricted MeanFlow**：限制平均速度采样范围 + 剪枝高压缩比样本，实现更stable的few-step蒸馏。最终在视频生成上达到**11.8×加速**，几乎无损质量（结构语义、时序、物理合理性全面领先）。

把MeanFlow作为step-distillation基线，但发现它在视频上“太激进”导致崩溃 → 提出Restricted变体使其更适合大模型视频；然后把改进后的蒸馏模型与learnable caching结合，形成“training-aware + training-free”互补框架。

**LARGE SCALE DIFFUSION DISTILLATION VIA SCORE-REGULARIZED CONTINUOUS-TIME CONSISTENCY (rCM)（NVIDIA + Tsinghua, ICLR 2026, arXiv:2510.08431）**

首次把连续时间一致性模型（sCM/MeanFlow）**规模化到10B+参数的大型文本到图像/视频模型**（Cosmos-Predict2、Wan2.1等，5秒视频）。发现纯sCM/MeanFlow在细粒度细节上质量崩（error accumulation + mode-covering），提出**rCM**：把score distillation（reverse divergence, mode-seeking）作为long-skip regularizer加到sCM（forward divergence）上。

额外贡献：开发FlashAttention-2兼容的JVP kernel，支持BF16+context parallelism。最终1~4步生成，质量匹敌DMD2但多样性更好，加速15~50×，无GAN调参。

rCM就是“MeanFlow/sCM风格的连续时间一致性”在大型视频上的scaling版 + 正则化改进，解决MeanFlow在真实大模型上的质量瓶颈。

1-step 已可接受，2~4-step 达到几乎无损高质量

**MEANFLOW-ACCELERATED MULTIMODAL VIDEO-TO-AUDIO SYNTHESIS VIA ONE-STEP GENERATION（MF-MJT）（Wuhan Univ + Xiaomi + SWUFE, arXiv:2509.06389）**

把MeanFlow直接用于**视频到音频（VTA）合成**（多模态联合训练框架MMAudio backbone）。用平均速度取代瞬时速度，实现**原生一步生成**（无需蒸馏）。额外提出**CFG-scaled机制**（自适应标量重缩放无条件预测），解决一步生成下CFG容易overshoot的问题。同时支持TTA（文本到音频）零样本迁移。实验显示VTA/TTA上2~500×加速，感知质量、语义对齐、时序同步均不下降。

核心创新就是“把MeanFlow塞进VTA网络”，并针对音频任务特化CFG，证明MeanFlow在跨模态（视频→音频）上也天然适合one-step。

**FlowerDance: MeanFlow for Efficient and Refined 3D Dance Generation（Renmin Univ + Tsinghua + Wuhan Univ, arXiv:2511.21029）**

音乐到3D舞蹈生成任务。提出**FlowerDance**框架：

- 生成策略：MeanFlow + Physical Consistency Constraints（重建、速度、3D关节位置损失，把轨迹拉回人体运动流形）。
- 模型架构：BiMamba backbone + Channel-Level Cross-Modal Fusion（参数free，非自回归）。
支持motion editing（时衰软掩码）。在AIST++和FineDance上SOTA（质量+效率双赢），推理速度和显存远超MEGA、Bailando、EDGE等。

论文明确“FlowerDance combines MeanFlow with Physical Consistency Constraints”，把MeanFlow当成“舞蹈编排的物理直观建模工具”，用它把多步ODE压到few-step，同时加物理约束解决舞蹈特有的不真实运动问题。

**DECOUPLED MEANFLOW: TURNING FLOW MODELS INTO FLOW MAPS FOR ACCELERATED SAMPLING（KAIST, arXiv:2510.24474）**

提出**Decoupled MeanFlow (DMF)**：无需改架构，就能把任意预训练Flow模型（SiT等）直接转成Flow Map模型。只需让DiT最后几块（decoder）条件化在**下一个timestep r**上（encoder仍只看当前t）。结合增强训练技巧，1~4步生成。在ImageNet 256/512上：1-step FID 2.16/2.12（SOTA），4-step FID 1.51/1.68（几乎追平原Flow模型，但>100×快）。证明“先训Flow再转DMF”比从零训Flow Map更高效。

**架构级改进版MeanFlow**。MeanFlow是“Flow Map”的代表工作（平均速度），但原版要求全程条件化r。DMF把MeanFlow的“decouple encoder-decoder”思想落地，实现“零架构改动”兼容任意预训练Flow模型，被视为MeanFlow在实际部署上的实用进化。

**ALPHAFLOW: UNDERSTANDING AND IMPROVING MEANFLOW MODELS（Snap Inc. + UMich, arXiv:2510.20771）**

发现其目标可分解为“trajectory flow matching”和“trajectory consistency”两部分，梯度强烈负相关导致优化冲突和慢收敛。提出**α-Flow**统一框架（涵盖Shortcut Model、MeanFlow、trajectory FM），用**curriculum策略**（从trajectory FM平滑退火到MeanFlow）解耦冲突。vanilla DiT从零训练，在ImageNet 256上：α-Flow-XL/2+ 1-NFE FID 2.58、2-NFE FID 2.15（均优于原MeanFlow），建立vanilla DiT backbone下的新SOTA。

**理论分析+系统性改进**。论文直接以“Understanding and Improving MeanFlow Models”为副标题，剖析其内在缺陷，并给出更优的α-Flow家族（本质是MeanFlow的“平滑版”），性能全面超越。

TODO：

rCM待改进：

1. FlashAttention-2 JVP kernel，在BF16 + 大模型（14B）下，JVP仍会产生“substantially larger numerical errors” → 用更稳定的JVP近似
2. 1-step distill “blurry textures + unstable object geometry + object interpenetration” ,Restricted MeanFlow的保守采样范围
3. 扩展到Autoregressive Video Diffusion，在已有的 autoregressive video diffusion 模型上，用 rCM/MeanFlow 风格做 forward-divergence distillation
