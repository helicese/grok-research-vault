# grok-research-vault

Grok Bot 的研究笔记仓库。笔记可以来自 X，也可以来自其他网页调研。Codex 通过 `git pull` 拉取后直接阅读这些文件，不必自己去搜索 X。

## 目录

| 路径 | 作用 |
| --- | --- |
| `AGENTS.md` | Codex 的读取约定 |
| `index/topics.json` | 主题索引 |
| `index/latest.json` | 最近笔记指针 |
| `research/<topic-slug>/README.md` | 单个主题的范围说明 |
| `research/<topic-slug>/notes/` | 该主题下按日期存放的笔记 |
| `research/_example-topic/` | 布局示例，不是真实研究 |

`research/.gitkeep` 用来在没有其他内容时仍跟踪 `research/` 目录。目前已有示例主题，该文件一并保留。

## Grok Bot 如何写笔记

1. 用小写连字符选定主题 slug，在 `research/<topic-slug>/` 下维护 `README.md`。
2. 每次调研新增 `notes/YYYY-MM-DD-<短标题>.md`。
3. 笔记使用 YAML front matter：`topic`、`date`（`YYYY-MM-DD`）、`query`、`sources`（URL 列表）、`summary`。
4. 正文写发现、引用或摘录、后续问题。`sources` 里可以是 X 帖子链接，也可以是其他网页。
5. 同步更新 `index/topics.json` 与 `index/latest.json`，让索引指向真实文件。

## Codex 如何使用

```bash
git pull
```

然后按这个顺序读：

1. `AGENTS.md`
2. `index/topics.json` 与 `index/latest.json`
3. 对应主题的 `research/<topic-slug>/README.md`，以及 `notes/` 下的笔记

只依据仓库里的文件作答。`research/_example-topic/` 是示例，不要当成真实研究。
