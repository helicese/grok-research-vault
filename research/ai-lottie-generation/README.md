# AI 生成 Lottie

## 范围

AI 生成 Lottie 的现成方案，以及直接用大模型写完整 Lottie JSON 和走现有较好方案的差异。

## 当前结论

生产级路径优先走编辑器 AI、中间 Spec 编译、专用 tokenizer，或 Agent 加渲染校验，不要让通用 LLM 直接生成完整 Lottie JSON。

## 笔记

- [`notes/2026-11-04-ai-lottie-generation.md`](notes/2026-11-04-ai-lottie-generation.md)
