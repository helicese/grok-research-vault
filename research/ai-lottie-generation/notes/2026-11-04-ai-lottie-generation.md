---
topic: ai-lottie-generation
date: 2026-11-04
query: AI 生成 Lottie 的现成方案；直接用大模型生成 Lottie JSON vs 基于现有较好方案的差异；答疑解惑
sources:
  - https://docs.lottiefiles.com/en/format/lottie-json/specification
  - https://lottiefiles.com/ai
  - https://lottiefiles.com/tools/prompt-to-lottie
  - https://www.lottielab.com/figma-to-lottie
  - https://github.com/openvglab/omnilottie
  - https://ar5iv.labs.arxiv.org/html/2603.02138
  - https://www.easylottie.com/image-to-lottie
  - https://vizgpt.ai/text-to-lottie
  - https://www.recraft.ai/features/image-converter/png-to-lottie
  - https://github.com/diffusionstudio/lottie
  - https://raw.githubusercontent.com/diffusionstudio/lottie/main/skills/text-to-lottie/SKILL.md
  - https://lottie.github.io/validator/
  - https://x.com/ssijak/status/2015379039939117127
  - https://x.com/GroqLLC/status/1877722528828797015
  - https://x.com/carmenansio/status/1883862596157399085
  - https://x.com/Abmankendrick/status/1942496281382822252
  - https://x.com/dorukkavcioglu/status/2073787052600995901
  - https://x.com/konstipaulus/status/2086138543236067410
  - https://x.com/LottieFiles/status/2095331854530555995
  - https://x.com/LottieFiles/status/2117801751520964885
  - https://x.com/fahdmirza/status/2033270476450242703
  - https://x.com/CatAccept/status/2029076477405454614
  - https://x.com/reallynattu/status/2079950671324094792
  - https://x.com/ridd_design/status/1932160566895784295
summary: >
  Lottie 是矢量动画 JSON。生产级 AI 路径应优先走「编辑器 AI / 中间 Spec 编译 / 专用 tokenizer / Agent+渲染校验」，
  而不是让通用 LLM 直接生成完整 Lottie JSON。学术基准上裸 LLM 成功率常接近 0–十几%，失败多为 schema 幻觉、
  属性挂错、空层与单位错误导致白屏；LottieFiles Copilot/MCP、Lottielab Magic Animator、EasyLottie、
  OmniLottie、diffusionstudio text-to-lottie 是当前更稳的方向。
---

# AI 生成 Lottie：方案与「直接 LLM」差异

## Findings

### Lottie 本身
- 矢量动画的 JSON 描述（图层、形状、关键帧等），源自 AE/Bodymovin 生态，跨端播放。出处：LottieFiles Format Spec。
- 单位陷阱：opacity 0–100，scale 百分比，颜色 0–1。出处：同上。

### 现成方案（按类型）
1. **编辑器内 AI**：LottieFiles Motion Copilot / Prompt to Lottie / MCP（官网 AI 页；X 上 Groq、设计师 demo、官方 MCP 演示）。
2. **设计稿→动效**：Lottielab Magic Animator（Figma→可编辑动画→Lottie；2025 年中发布讨论热）。
3. **中间表示再编译**：EasyLottie（TimelineSpec）、vizGPT（motion spec）——产品页明确「不直接让 AI 写 Lottie JSON」。
4. **专训模型**：OmniLottie（CVPR 2026；Lottie tokenizer；文/图/视频→JSON）。
5. **Agent Skill + 本地校验**：diffusionstudio/lottie（`npx skills add`；Skottie 验帧；X 上宣称 GitHub 5k+ stars）。
6. **资产准备**：Recraft 矢量化后导出 Lottie。

### 直接 LLM 生成：失败模式
- Schema 幻觉、属性错位、空 layers、缺 fill/stroke、o/s 单位错、关键帧非法。出处：OmniLottie 附录失败分类；X @ssijak 吐槽。
- 定量：论文 Real Text-to-Lottie 上 OmniLottie 远高于 GPT-5 / 通用 VLM。出处：arXiv 2603.02138 Table 1。

### 直接 LLM vs 好方案
- 可靠性、可编辑性、品牌保真、控制力：好方案胜；裸 LLM 仅适合 PoC。
- 根因：Lottie JSON 高度嵌套且充满「格式噪声」；好方案要么缩短表示（tokenizer/spec），要么在工具里改并校验。

## 推荐（前端/AI）
- 生产：Copilot/MCP 或 text-to-lottie skill 或 Magic Animator / EasyLottie。
- 自研：学 OmniLottie 的表示与渲染反馈，别迷信「更大模型一次写对」。
- 强交互：Rive 或 Lottie state machines。

## Follow-ups
- [ ] 实测 LottieFiles Prompt to Lottie vs Motion Copilot 同 prompt 质量
- [ ] 在 Codex 上安装 diffusionstudio/lottie，跑一套品牌 SVG→Lottie
- [ ] 跟踪 LottieFiles「3 new AI products」（X 2026-11-04 预告）
- [ ] 若需要视频→矢量动画，评测 OmniLottie video-to-lottie 本地效果与成本
