---
topic: ai-lottie-generation
date: 2026-11-05
query: "AI 如何生成 Lottie：背景、方案特性、例子对比与选型建议（面向零基础）"
sources:
  - https://medium.com/airbnb-engineering/introducing-lottie-4ff4a0afac0e
  - https://lottiefiles.com/blog/working-with-lottie-animations/lottie-vs-gif
  - https://lottie.github.io/lottie-spec/1.0
  - https://dotlottie.io/spec
  - https://dotlottie.io/spec/1.0
  - https://dotlottie.io/spec/2.0
  - https://www.kalex.studio/blog/lottie-vs-gif-vs-mp4-vs-webm-web-animation-formats
  - https://rive.app/blog/rive-as-a-lottie-alternative
  - https://github.com/airbnb/lottie/blob/master/after-effects.md
  - https://lottiefiles.com/blog/working-with-lottie-animations/introducing-motion-copilot-on-lottie-creator
  - https://arxiv.org/html/2604.11792v1
  - https://lottiefiles.com/ai
  - https://docs.lottiefiles.com/en/creator/13_ai-tools/motion-copilot
  - https://lottiefiles.com/tutorials/lottie-creator/talk-to-lottie-creator-using-motion-copilot-0QCYWw5uIX3
  - https://lottiefiles.com/tools/prompt-to-lottie
  - https://vizgpt.ai/text-to-lottie
  - https://lottiefiles.com/tools/svg-to-lottie
  - https://www.lottielab.com/lottie/svg-to-lottie
summary: "Lottie 是矢量动画指令 JSON；通用大模型零样本几乎吐不出可渲染文件。今天最稳路径是 LottieFiles Creator 的 Motion Copilot / Prompt to Lottie / SVG→Lottie，产品管线应校验与多端冒烟，而不是裸 LLM 默写 JSON。"
---

## 发现（findings）

# AI 生成 Lottie 调研报告（面向小白）

> 调研日期：2026-11-05（Asia/Shanghai）  
> 方法：Tavily Research（model=`pro`）+ 官方页抽取核实 + 补充搜索；X 社区检索噪声大，仅作旁证说明。  
> 约定：带脚注 URL 的为**来源事实**；标为「分析」的是综合判断，非原文照搬。

---

## 1. 给小白的背景：Lottie 是什么？

### 1.1 一句话定义

**Lottie** 最初由 Airbnb 开源：它是一套能在 iOS / Android / React Native（以及后来的 Web 等）上**实时播放 After Effects 动画**的库；动画数据来自 After Effects 插件 **Bodymovin** 导出的 **JSON** 文件，而不是把每一帧都存成像素图。  
来源：Airbnb 工程博客 [Introducing Lottie](https://medium.com/airbnb-engineering/introducing-lottie-4ff4a0afac0e)；统一站点 [lottie.airbnb.tech](https://lottie.airbnb.tech)。

可以把它想成：

| 类比 | GIF / 视频 | Lottie |
|---

## 0. 一句话结论

**今天要「真可用的 Lottie」：优先走 LottieFiles 编辑器生态（Motion Copilot / Prompt to Lottie / SVG→Lottie），再用 Creator 精修；不要把「让通用大模型直接吐 JSON」当生产主路径。**

理由很短：Lottie 是严格的矢量动画指令格式，零样本大模型在公开研究里基本吐不出可渲染文件；官方 AI 工具是在编辑器里改关键帧/出草稿，产物才稳。有现成 SVG 时先转 Lottie 通常比纯文本更贴品牌；视频/GIF 转 Lottie 要当心变成「嵌帧」、丢掉矢量优势。强交互动效另看 Rive。

（细节与出处见下文。）

---

---|------------|--------|
| 存什么 | 每一帧像素（或压缩视频帧） | 「画什么形状、怎么动、何时动」的指令 |
| 放大 | 糊 | 矢量路径通常清晰（纯矢量时） |
| 体积 | 常偏大 | 同类 UI 动效通常更小 |
| 改颜色/交互 | 难 | JSON 可被代码改色、控播放 |

（上表中体积与清晰度对比的典型表述见 LottieFiles 博客 [Lottie vs GIF](https://lottiefiles.com/blog/working-with-lottie-animations/lottie-vs-gif)。）

### 1.2 文件长什么样？三种常见「外壳」

1. **`.json`（标准 Lottie JSON）**  
   - 规范要求用 JSON 组织，顶层是 Animation 对象，并有机器可读 Schema。  
   - 常见字段包括：`v`（版本）、`fr`（帧率）、`ip`/`op`（入出点）、`w`/`h`、`assets`、`layers` 等——**不是随便一段 JSON**。  
   来源：[Lottie Spec 1.0](https://lottie.github.io/lottie-spec/1.0)；[LottieFiles JSON 规格说明](https://docs.lottiefiles.com/en/format/lottie-json/specification)。

2. **`.lottie`（dotLottie）**  
   - 开源打包格式：把一个或多个 Lottie 动画及图片/字体等资源打进**压缩 ZIP**；扩展名 `.lottie`，需含 `manifest.json`。  
   - 官方称相对原始 JSON 往往更小；还可带主题、状态机等（视版本而定）。  
   来源：[dotLottie Spec](https://dotlottie.io/spec)、[v1.0](https://dotlottie.io/spec/1.0)、[v2.0](https://dotlottie.io/spec/2.0)；LottieFiles [Lottie vs GIF 文中对 dotLottie 的说明](https://lottiefiles.com/blog/working-with-lottie-animations/lottie-vs-gif)。

3. **`.tgs`（Telegram 动画贴纸）**  
   - Telegram 官方说明：动画贴纸是以 `.tgs` 存的 **Lottie 向量动画**，由 rlottie 播放。  
   - 实务上常被描述为 gzip 压缩的 Lottie JSON，并有严格限制（如尺寸、时长、体积、不支持部分 AE 特性）。第三方工具页亦复述了常见约束；**若做贴纸，以 Telegram 官方要求为准并自行实测**。  
   来源：[Telegram Stickers API](https://core.telegram.org/api/stickers)；格式说明参考 [fileformat.com TGS](https://docs.fileformat.com/compression/tgs)。

**最小心智模型（分析）：**  
设计师在时间轴上摆图层 → 导出成「脚本式 JSON」→ App/网页用播放器读脚本并实时画出来。

### 1.3 怎么播放？

| 平台 | 常见运行时 | 说明 |
|------|------------|------|
| Web | [lottie-web](https://github.com/airbnb/lottie-web) | 解析 Bodymovin JSON，在浏览器渲染 |
| iOS | [lottie-ios](https://github.com/airbnb/lottie-ios) | Apple 平台原生渲染 |
| Android | [lottie-android](https://github.com/airbnb/lottie-android) | Android 原生渲染 |
| 多端 / `.lottie` | [LottieFiles Runtimes](https://lottiefiles.com/runtimes) | 宣称支持 Web、Android、iOS 及 React / Vue / Flutter 等 |

开发侧通常：把 JSON 或 `.lottie` 放进工程 → 调用 SDK 的 `play` / `loop` / 进度控制。

### 1.4 常见用途

- App / 网站：**加载动画、成功勾选、空状态插画、按钮微交互、引导页**  
- 营销落地页 hero 区轻量动效  
- Telegram 等平台的向量贴纸（`.tgs`）  

适合「图标/插画式、可矢量化」的运动；**不适合**把电影级实拍或重度粒子特效硬塞进 Lottie（分析；格式对比讨论见 [kalex.studio 格式对比文](https://www.kalex.studio/blog/lottie-vs-gif-vs-mp4-vs-webm-web-animation-formats)）。

### 1.5 和 GIF / 视频 / SVG·CSS / Rive 差在哪？

| 对比对象 | 关键差异（来源事实摘要） |
|----------|--------------------------|
| **GIF** | GIF 是光栅、逐帧像素；Lottie 多为矢量指令。LottieFiles 称多 MB 的 GIF 常可对应「很小的 Lottie」；GIF 几乎到处能播，但体积大、难编辑、难交互。[Lottie vs GIF](https://lottiefiles.com/blog/working-with-lottie-animations/lottie-vs-gif) |
| **MP4 / WebM** | 视频更适合复杂影像；Lottie 更适合 UI/插画动效与代码可控播放。系统化官方「谁赢」规范不足，选型看内容类型。 |
| **CSS / 手写 SVG 动画** | CSS 无额外播放器，但复杂时间轴、角色级运动更吃力；Lottie 常由 AE/编辑器可视化制作，且可跨 Web+原生。第三方对比文：[Lottie vs SVG](https://lottiemax.com/learn/lottie-vs-svg-animation)、[Lottie vs CSS](https://lottiewizard.com/lottie-vs-css-animation)（偏二手，作辅助） |
| **Rive** | Rive 强在状态机、骨骼/网格与运行时输入；Lottie 交互多依赖外部代码控时间轴。Rive 官方称未压缩 Lottie JSON 常比等价 `.riv` 大许多，且 iOS 渲染路径不同。[Rive vs Lottie](https://rive.app/blog/rive-as-a-lottie-alternative) |
| **Spine** | Spine JSON 主要序列化骨骼/插槽/皮肤/动画（[Spine JSON 格式](https://en.esotericsoftware.com/spine-json-format)）；与 Lottie「AE 导出 UI 插画动效」工作流不同，不宜简单等同。 |

### 1.6 传统制作工作流（至今仍是「高质量」基准）

```
After Effects 做动画
    → Bodymovin（或 LottieFiles for AE 插件）导出 JSON
    → 用 lottie-web / lottie-ios / lottie-android / dotLottie runtime 播放
```

- Bodymovin：Hernan Torrisi 创建的 AE 插件，导出 JSON + 附带 Web 播放器；Airbnb 在其上扩展到原生平台。  
  来源：[airbnb/lottie after-effects.md](https://github.com/airbnb/lottie/blob/master/after-effects.md)  
- LottieFiles for AE：在导出能力之上增加预览、性能/体积检查等（见 [Bodymovin or LottieFiles for AE](https://lottiefiles.com/blog/working-with-lottie-animations/bodymovin-or-lottiefiles-for-after-effects)）。  
- **注意：** AE 里很多特效/表达式在 Lottie 播放器上**不完全支持**，导出前要对照各端支持表（Airbnb / LottieFiles 均维护支持特性表）。

---

## 2. 问题定义：为什么有人想用 AI 生成 Lottie？

**来源可支撑的动机（产品定位推断，非调查统计）：**

1. **门槛**：动效设计师贵且稀缺；产品/运营/前端也想要「加载成功勾、空状态」这类资产。Lottie Creator 明确面向新手与资深用户，并用 Motion Copilot 做文本驱动创建。[Lottie Creator](https://lottiefiles.com/lottie-creator)  
2. **速度与变体**：同一图标要多套配色、时长、循环/非循环；AI 辅助改关键帧、出草稿更快（Motion Copilot 2.0 博客描述可从一句提示搭出可编辑场景）。[Introducing Motion Copilot 2.0](https://lottiefiles.com/blog/working-with-lottie-animations/introducing-motion-copilot-on-lottie-creator)  
3. **管线化**：希望「提示词 → 可播放 JSON → 进 App」，而不是每人装一套 AE。第三方如 VizGPT 自称面向产品/营销/前端团队。[vizGPT AI Generate Lottie](https://vizgpt.ai/usecases/ai-generate-lottie)  

**分析：** ROI 数字（省多少人天、单价）公开一手材料不足；下文不编造价格。选型应自己试官方免费档并核算商用授权。

---

## 3. 可行性：大模型能不能「直接吐」合法 Lottie JSON？

### 3.1 难点（结构事实）

Lottie JSON 是深层嵌套：图层、形状组、贝塞尔路径、变换、关键帧时间、缓动曲线、资源引用……字段类型/层级稍错，播放器就不渲染。  
来源：规格文档如上；学术工作亦以此为出发点。

### 3.2 研究结论（较强证据）

论文 **LottieGPT: Tokenizing Vector Animation for Autoregressive Generation** 明确写到：

> *Since zero-shot LLM / VLM cannot generate any valid renderable Lottie JSON files, we do not present these results…*  
> 并称 few-shot 下多数 LLM 输出仍常无法正确渲染；多数 VLM 生成的 JSON 不符合标准。

来源：[arXiv HTML 2604.11792v1](https://arxiv.org/html/2604.11792v1)（文内 §10 等）。

**分析（据此）：**  
「打开 ChatGPT，一句中文 → 复杂可商用 Lottie」**今天不能当成可靠默认路径**。个案成功（社区博客/视频教模型修 JSON）只能说明「在强约束、小场景、人工校验下偶尔可行」，不能推翻论文里的 zero-shot 失败观察。

### 3.3 更靠谱的技术路线（分析 + 产品事实）

| 路线 | 含义 | 是否真 Lottie |
|------|------|----------------|
| A. 编辑器内 AI | 提示词驱动 Creator 画布改图层/关键帧，再导出 | 是（官方导出） |
| B. Prompt→Lottie 一键工具 | 网页输入提示直接下载动画 | 宣称是；需自测播放器 |
| C. 中间格式管线 | 提示→SVG/矢量 → 加动画/转 Lottie；或→视频再转 | 视转换质量；光栅嵌入则丢矢量优势 |
| D. 专用/代理模型 | 如论文 LottieGPT；或 agent 写 JSON + validate/autofix | 目标是真 JSON；成熟度不一 |
| E. 裸 LLM 吐 JSON | 无 schema 约束、无校验 | 高风险，论文显示 zero-shot 基本无效 |

---

## 4. 方案分类与特性

### 4.1 传统 AE + Bodymovin（非 AI，对照基准）

| 维度 | 说明 |
|------|------|
| 输入/输出 | AE 工程 → JSON / 再包 `.lottie` |
| 是否合法 Lottie | 是（在支持特性范围内） |
| 可控性 | 最高（时间轴、缓动、图层精细调） |
| 质量上限 | 高（受播放器特性限制） |
| 成本/门槛 | AE 授权 + 动效技能；周期长 |
| 适用 | 品牌级、复杂叙事、要像素级打磨 |

### 4.2 编辑器内 AI：LottieFiles Creator + Motion Copilot（当前最稳）

**是什么：** 浏览器里的 Lottie Creator；**Motion Copilot** 用自然语言创建/编辑场景、图形、动画、主题、交互；结果在画布上可预览、可撤销、可手改。另有 **Prompt to Vector / Themes / State Machines**，以及把 Claude/Cursor 等接到编辑器的 **Lottie Creator MCP / WebMCP**。  

来源：  
- [lottiefiles.com/ai](https://lottiefiles.com/ai)  
- [Motion Copilot 文档](https://docs.lottiefiles.com/en/creator/13_ai-tools/motion-copilot)（文档标注更新于 2026-10-08）  
- [Lottie Creator](https://lottiefiles.com/lottie-creator)  
- [Motion Copilot 2.0 介绍博文](https://lottiefiles.com/blog/working-with-lottie-animations/introducing-motion-copilot-on-lottie-creator)  
- 教程示例提示如 “move the square from left to right”：[Talk to Lottie Creator using Motion Copilot](https://lottiefiles.com/tutorials/lottie-creator/talk-to-lottie-creator-using-motion-copilot-0QCYWw5uIX3)

| 维度 | 说明 |
|------|------|
| 输入/输出 | 文本（及选中图层）→ 可编辑动画 → 导出 Lottie / dotLottie |
| 是否合法 Lottie | **是**（走官方编辑器导出） |
| 可控性 | 高：AI 出草稿 + 人工精修；可用 `@`/图层名定点改 |
| 质量 | 适合 UI 微交互、简单场景；复杂叙事仍需人 |
| 成本/门槛 | 网页即可；具体套餐/配额**本稿未核价，请查官网** |
| 适用 | 学习、快速出素材、设计团队日常 |

### 4.3 一键 Prompt → Lottie

**LottieFiles Prompt to Lottie**  
- 文案：输入提示即可生成可下载使用的动画 Lottie；示例提示如 “flower breathe”“success sparkle check”。  
- 同页也引导复杂需求改用 Motion Copilot / Prompt to Vector / MCP。  
来源：[Prompt to Lottie](https://lottiefiles.com/tools/prompt-to-lottie)

**VizGPT Text-to-Lottie**  
- 自称文本→Lottie JSON，可在编辑器继续改；示例含纸飞机穿云、礼花、三点加载等。  
- **证据等级：产品自述为主，缺独立评测。**  
来源：[vizgpt.ai/text-to-lottie](https://vizgpt.ai/text-to-lottie)

| 维度 | 说明 |
|------|------|
| 输入/输出 | 一句提示 → 下载 JSON/动画 |
| 是否合法 Lottie | 官方工具一般可播；第三方需用标准播放器自测 |
| 可控性 | 偏低（生成后再进编辑器才能细调） |
| 质量 | 简单 icon/微交互较好；复杂定制存疑 |
| 成本/门槛 | 极低学习成本；商用条款自查 |
| 适用 | 「先有一个能动的东西」的草稿阶段 |

### 4.4 转换管线：SVG / 视频 / GIF → Lottie

**SVG → Lottie（推荐的转换入口）**  
- [LottieFiles SVG to Lottie](https://lottiefiles.com/tools/svg-to-lottie)：上传 SVG（页称最大 20MB）转 Lottie，强调矢量清晰、跨端、轻量。  
- 另有 [Lottielab SVG to Lottie](https://www.lottielab.com/lottie/svg-to-lottie)、[IconScout SVG to Lottie](https://iconscout.com/converter/svg-to-lottie) 等第三方转换页。  
- 开源：[Koding-net/lottie-tools](https://github.com/Koding-net/lottie-tools)（Lottie↔SVG 工具集，自称可客户端运行）。

**视频 / GIF → Lottie**  
- LottieFiles 提供汇总入口 [Video to Lottie](https://lottiefiles.com/tools/video-to-lottie)，下挂 GIF/MP4/MOV/WebM/PNG 序列等转换。  
- **分析/风险：** 光栅源转 Lottie 时常变成「把帧嵌进 Lottie」——文件可变大，且失去「真矢量可改色/无限缩放」的核心优势。IconikAI 博文亦持类似观点（二手）：[Lottie Icon Animation Generator](https://www.iconikai.com/blog/lottie-animation-icon-generator)。  
- **本稿未做并排实测**；若必须用视频源，请导出后检查：是否仍是 shape 图层、体积、各端播放。

| 维度 | SVG→Lottie | 视频/GIF→Lottie |
|------|------------|-----------------|
| 输入/输出 | 静态/动画 SVG → Lottie | 光栅序列/视频 → Lottie |
| 真矢量？ | 通常更好 | 风险高（可能嵌图） |
| 可控性 | 中（依赖源 SVG 结构） | 低 |
| 适用 | 已有矢量资产时优先 | 仅有成片、临时兼容 |

还可配合：**Prompt → Vector（SVG）→ 再动画/转 Lottie**（LottieFiles AI 工具矩阵中的 Prompt to Vector / Raster to Vector，见 [AI 工具页](https://lottiefiles.com/ai)）。

### 4.5 开源 / Agent / 专用模型路线

**Diffusion Studio「Text to Lottie」**  
- Changelog 称 agent 直接按 Lottie 规格写图层、形状、关键帧、缓动；适合短单场景、给 SVG 加动画、数据驱动图（如 K 线图示例）。  
- 宣称生成 JSON 可用于 lottie-web / RN / iOS / Android / Flutter，并可进 AE；并称开源（页内指向 GitHub，**具体仓库 URL 以该页「View the code」链接为准，本稿未单独克隆核实**）。  
来源：[diffusion.studio/changelog/text-to-lottie](https://diffusion.studio/changelog/text-to-lottie)

**kin3o（GitHub）**  
- 自述为 AI-powered Lottie 生成 CLI：`prompt → generate → validate → autofix → .json/.lottie`；强调 Lottie JSON 适合 AI 生成，而 Rive 的二进制 `.riv` 难被 LLM 直接生成。  
来源：[github.com/affromero/kin3o](https://github.com/affromero/kin3o)

**LottieGPT（论文）**  
- 将生成建模为结构化代码/token，而非像素视频；对比商业 LLM 与视频模型；并报告 zero-shot LLM 无法产出可渲染 JSON。  
来源：[arXiv](https://arxiv.org/html/2604.11792v1)  
- **分析：** 研究原型/方法论文，**不等于**已有稳定面向大众的 SaaS；集成前需看是否有公开权重/API（本稿未找到可商用官方产品页）。

**社区「教 Claude/Codex 写 Lottie」**  
- 例如 Design Huddle 帮助文：[How to create and validate Lottie files with Claude or Codex](https://help.designhuddle.com/how-to-create-and-validate-lottie-files-with-claude-or-codex)；个人博客/YouTube 教程亦有。  
- **证据等级：操作指南/个案**；务必加 schema 校验与播放器回归。

### 4.6 非 Lottie 替代：何时考虑 Rive？

若需求是**复杂交互状态机、运行时输入、骨骼网格**，Rive 往往更合适；但它**不是** Lottie，工具链与 AI「直接生成文本格式」友好度不同（kin3o README 亦对比二者）。  
来源：[Rive vs Lottie](https://rive.app/blog/rive-as-a-lottie-alternative)；[kin3o](https://github.com/affromero/kin3o)

---

## 5. 具体例子（产品 / 项目 + URL + 一句话）

| 名称 | URL | 文档/实测说法摘要 |
|------|-----|-------------------|
| Motion Copilot | https://docs.lottiefiles.com/en/creator/13_ai-tools/motion-copilot | 在 Creator 内用对话创建/编辑 scene、artwork、animation、themes、interactivity；预览后再应用 |
| Lottie Creator | https://lottiefiles.com/lottie-creator | 浏览器制作并导出 Lottie/dotLottie；FAQ 称可用 Motion Copilot 把提示变成可编辑关键帧 |
| Prompt to Lottie | https://lottiefiles.com/tools/prompt-to-lottie | 「Type a prompt and get an animated Lottie」；给简单示例提示 |
| Prompt to Vector | 见 https://lottiefiles.com/ai | 文本→矢量图，再动画 |
| SVG to Lottie | https://lottiefiles.com/tools/svg-to-lottie | 官方在线转换，强调矢量与跨端 |
| Video to Lottie 入口 | https://lottiefiles.com/tools/video-to-lottie | 提供 GIF/MP4/MOV/WebM/PNG 等转换入口 |
| VizGPT Text-to-Lottie | https://vizgpt.ai/text-to-lottie | 自述文本生成可部署 JSON（待独立验证） |
| Diffusion Studio Text-to-Lottie | https://diffusion.studio/changelog/text-to-lottie | Agent 写完整规格 JSON；偏数据驱动/短场景 |
| kin3o | https://github.com/affromero/kin3o | CLI：生成→校验→修复；可出交互 `.lottie` |
| LottieGPT | https://arxiv.org/html/2604.11792v1 | 专用生成方法；记录通用 LLM zero-shot 失败 |
| Bodymovin 工作流 | https://github.com/airbnb/lottie/blob/master/after-effects.md | AE→JSON 传统权威路径 |
| Airbnb 介绍文 | https://medium.com/airbnb-engineering/introducing-lottie-4ff4a0afac0e | Lottie 起源与定位 |
| lottie-web / ios / android | 对应 GitHub airbnb 仓库 | 播放侧事实标准 |

**教程级例子（官方）：**  
提示 “move the square from left to right” / “fade in the text” → Motion Copilot 生成关键帧（[教程页](https://lottiefiles.com/tutorials/lottie-creator/talk-to-lottie-creator-using-motion-copilot-0QCYWw5uIX3)）。  
这比「让 LLM 默写整份 JSON」更符合当前产品能力边界。

---

## 6. 对比表（总览）

| 方案 | 输入 | 输出 | 真·可播放 Lottie？ | 可控性 | 质量上限 | 门槛 | 最适场景 |
|------|------|------|-------------------|--------|----------|------|----------|
| AE + Bodymovin | AE 工程 | JSON | ✅（特性范围内） | 极高 | 高 | 高 | 品牌/复杂动效 |
| Creator + Motion Copilot | 自然语言 + 画布 | Lottie/dotLottie | ✅ 官方导出 | 高 | 中高 | 低–中 | **默认推荐主力** |
| Prompt to Lottie（LF） | 一句提示 | 动画文件 | ✅（宣称/官方） | 中低 | 中（简单） | 极低 | 秒出草稿 |
| VizGPT 等第三方 T2L | 提示 | JSON | ⚠️ 需自测 | 中低 | 不明 | 低 | 试验 |
| SVG→Lottie | SVG | Lottie | ✅ 常见 | 中 | 取决于 SVG | 低 | 已有矢量 |
| 视频/GIF→Lottie | 光栅 | Lottie | ⚠️ 可能嵌帧 | 低 | 常损矢量优势 | 低 | 仅有成片时 |
| Agent/CLI（kin3o 等） | 提示 | JSON/.lottie | ⚠️ 靠校验 | 中（可编程） | 场景受限 | 中（工程） | 实验管线 |
| 裸 LLM 吐 JSON | 提示 | 文本 | ❌ 常失败 | 低 | 低 | 低 | **不推荐主路径** |
| Rive | Rive 编辑器 | `.riv` | 非 Lottie | 高（交互） | 高（交互向） | 中 | 强交互 UI |

---

## 7. 最终建议（可执行）

### 7.1 场景 A：学习了解（对 Lottie 一窍不通）

1. 读 Airbnb 介绍文 + 打开任意小 JSON，对照 `layers` / `fr` / `ip`/`op` 字段（[规格](https://docs.lottiefiles.com/en/format/lottie-json/specification)）。  
2. 在 [Lottie Creator](https://lottiefiles.com/lottie-creator) 用资源库拖一个 loader，导出，用 [Web Player](https://lottiefiles.com/tools/web-player) 或官方 runtime 播一下。  
3. 再用 Motion Copilot 对**已有形状**说「向右移动并循环」，体会「AI 改关键帧 ≠ AI 瞎写 JSON」。  
4. （可选）装 AE 试用 + Bodymovin，理解「专业上限」在哪。

### 7.2 场景 B：快速出营销 / UI 素材

**推荐顺序（分析）：**

1. **首选** LottieFiles：**Prompt to Lottie** 出草稿 → 不满意就进 **Creator + Motion Copilot** 精修 → 导出 `.lottie`。  
2. 若设计侧已有 **SVG/图标**：**SVG to Lottie** 或 Prompt to Vector → 再动画，通常比纯文本更贴品牌。  
3. 不要优先走「视频转 Lottie」除非只有成片；转完检查是否仍是矢量、体积是否爆炸。  
4. 第三方一键站可作对照实验，**上线前**必须用目标端 SDK 播一遍并确认授权。

### 7.3 场景 C：产品内嵌「AI → Lottie」管线

**不要**把「调用通用 Chat API 让它返回 JSON」当唯一架构。

更稳的三段式（分析）：

```
提示词 / 模板参数
  → 受约束生成（Creator MCP / 自建 agent + JSON Schema 校验 + 自动修复）
  → 人工或规则化精修（色板、时长、无障碍对比度）
  → CI：lottie 播放器冒烟（Web + 至少一端移动）
```

- 短期可产品化：**对接 Lottie Creator MCP / 其即将开放的 API（官网称 API early access，以官网为准）**，让模型「操作编辑器」而不是「默写文件」。  
- 中期研发：参考 kin3o / Diffusion Studio 的 validate→autofix；关注 LottieGPT 一类专用模型是否开源落地。  
- 若核心是**强交互**，评估 **Rive**，而不是硬凹 Lottie。  
- Telegram 贴纸另加 `.tgs` 约束检查。

### 7.4 一句话总推荐（分析）

> **今天要「真 Lottie」：用 LottieFiles 编辑器生态（Motion Copilot / Prompt to Lottie / SVG 转换）做主路径；用校验过的 agent 做实验；把裸 LLM 直接生成 JSON 当成不可靠的旁路。**

---

## 8. 证据缺口（诚实清单）

以下内容**查到的公开材料不足以下定论**，故不编造：

1. 各家 SaaS 的准确价格、商用授权、API 配额（请直接看 LottieFiles / VizGPT 定价页）。  
2. GPT / Claude / Gemini 在同一测试集上的 Lottie 生成成功率横评（论文有定性结论，但不是你的业务测试集）。  
3. Video→Lottie 是否在典型素材上保持「真矢量」的系统实测。  
4. LottieGPT 是否已有可调用的官方推理 API / 权重发布渠道。  
5. X 上「AI 生成 Lottie 格式」的高质量从业者讨论——本次检索多为同名角色画师或无关动画工具帖，**旁证价值低**。

---

## 9. 出处列表

### 9.1 来源事实（按主题）

**起源与播放**  
- https://medium.com/airbnb-engineering/introducing-lottie-4ff4a0afac0e  
- https://lottie.airbnb.tech  
- https://github.com/airbnb/lottie-web  
- https://github.com/airbnb/lottie-ios  
- https://github.com/airbnb/lottie-android  
- https://github.com/airbnb/lottie/blob/master/after-effects.md  
- https://lottiefiles.com/runtimes  

**格式**  
- https://lottie.github.io/lottie-spec/1.0  
- https://docs.lottiefiles.com/en/format/lottie-json/specification  
- https://dotlottie.io/spec  
- https://dotlottie.io/spec/1.0  
- https://dotlottie.io/spec/2.0  
- https://core.telegram.org/api/stickers  
- https://docs.fileformat.com/compression/tgs  

**对比**  
- https://lottiefiles.com/blog/working-with-lottie-animations/lottie-vs-gif  
- https://rive.app/blog/rive-as-a-lottie-alternative  
- https://www.kalex.studio/blog/lottie-vs-gif-vs-mp4-vs-webm-web-animation-formats  
- https://en.esotericsoftware.com/spine-json-format  

**AI 产品与工具**  
- https://lottiefiles.com/ai  
- https://lottiefiles.com/lottie-creator  
- https://docs.lottiefiles.com/en/creator/13_ai-tools/motion-copilot  
- https://lottiefiles.com/blog/working-with-lottie-animations/introducing-motion-copilot-on-lottie-creator  
- https://lottiefiles.com/tools/prompt-to-lottie  
- https://lottiefiles.com/tools/svg-to-lottie  
- https://lottiefiles.com/tools/video-to-lottie  
- https://lottiefiles.com/tutorials/lottie-creator/talk-to-lottie-creator-using-motion-copilot-0QCYWw5uIX3  
- https://vizgpt.ai/text-to-lottie  
- https://vizgpt.ai/usecases/ai-generate-lottie  
- https://diffusion.studio/changelog/text-to-lottie  
- https://github.com/affromero/kin3o  
- https://github.com/Koding-net/lottie-tools  
- https://arxiv.org/html/2604.11792v1  

**其他转换/编辑器（补充）**  
- https://www.lottielab.com/lottie/svg-to-lottie  
- https://iconscout.com/converter/svg-to-lottie  
- https://www.svgator.com/tools  
- https://help.designhuddle.com/how-to-create-and-validate-lottie-files-with-claude-or-codex  

### 9.2 本文分析（非原文）

- 「裸 LLM 不适合做生产主路径」——综合 LottieGPT 实验表述与 Lottie schema 复杂度。  
- 「快速出素材优先 LF 官方 AI + SVG 转换」——基于官方能力描述与转换是否保持矢量的工程常识。  
- 「产品管线应 validate + 多端冒烟，而不是只信模型」——工程建议。  
- 「视频转 Lottie 慎用」——基于光栅/矢量差异及二手行业表述，**缺系统实测**。  
- 对比表中的「质量上限/门槛」列为相对判断，非厂商评分。

---

## 10. 调研元数据

| 项 | 值 |
|----|-----|
| Tavily Research | model=`pro`，status=completed，request_id=`f5ab7e18-4a55-4baa-a3f5-37dee3d96846` |
| 补充 | `tavily_extract` 核实 LF/Airbnb/arXiv/VizGPT/Rive 等；`tavily_search` 补充 Motion Copilot、转换工具、Bodymovin |
| X | `search_posts_all` 一次；结果与「Lottie 动画格式 + AI」相关度低，未采信为产品能力证据 |
| 输出文件 | `/workspace/ai-lottie-research-v2.md` |
| 未做 | 未安装 Tavily CLI；未向 GitHub vault 提交；未实测各 SaaS 付费档 |


## 引用 / 摘录（quotes / excerpts）

- LottieGPT：「Since zero-shot LLM / VLM cannot generate any valid renderable Lottie JSON files…」— https://arxiv.org/html/2604.11792v1
- Airbnb：「Lottie is a mobile library for Android and iOS that parses Adobe After Effects animations exported as JSON…」— https://medium.com/airbnb-engineering/introducing-lottie-4ff4a0afac0e
- Motion Copilot docs：文本驱动、在 Creator 内生成可编辑关键帧 — https://docs.lottiefiles.com/en/creator/13_ai-tools/motion-copilot

## 后续问题（follow-ups）

- 实测 LottieFiles 各 AI 工具免费档导出质量与商用授权。
- Video→Lottie 在典型营销素材上是否仍保持矢量优势。
- LottieGPT 权重/推理 API 是否开源落地。
- 若强交互，做 Rive vs Lottie 产品侧对比原型。
