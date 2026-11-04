# Agent conventions

给 Codex 的读取约定。先 `git pull`，再按下面的顺序读文件。答案以仓库内容为准，不要为了补全笔记去搜索 X。

## 读取顺序

1. `index/topics.json`：主题目录。
2. `index/latest.json`：最近笔记的指针。
3. `research/<topic-slug>/README.md`：该主题的范围。
4. `research/<topic-slug>/notes/`：单篇笔记。需要时按文件名中的日期从新到旧读。

## 笔记格式

路径：`research/<topic-slug>/notes/YYYY-MM-DD-<短标题>.md`

YAML front matter 字段：

- `topic`：主题 slug，与文件夹名一致
- `date`：`YYYY-MM-DD`
- `query`：这次调研的问题
- `sources`：URL 列表。可以是 X 帖子，也可以是其他网页
- `summary`：依据本篇笔记写出的简短摘要

正文包含三块：

- 发现（findings）
- 引用 / 摘录（quotes / excerpts），每条注明对应的 source URL
- 后续问题（follow-ups）

## 约束

- 不要编造文件里没有的事实。
- 索引与笔记不一致时，以笔记文件为准，索引只当作指针。
- `research/_example-topic/` 是布局示例，不是研究结论。
- 没有 X 链接并不表示笔记无效。来源可以是 X 或其他网络调研。
