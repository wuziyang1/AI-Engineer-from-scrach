# 拼接的 context

- 条数: 6
- 粗估 tokens: 876
- roles: ['system', 'user', 'assistant', 'user', 'user', 'assistant']
- 公式: head(3) + summary_msg(1) + tail(2)

===== [0] role=system =====
## MEMORY
- 用户叫 Julie，在上海，想转 AI Agent Engineer
- 主力 Python，做过 RAG Agent；最近在拆 Hermes Agent 源码

你是编程助手，回答要具体、能对着源码讲。

===== [1] role=user =====
我是 Julie，在上海准备转 AI Agent Engineer。今天卡在 Hermes 的 `context_compressor.py`：为什么对话一长就突然「改历史」了？我以为 Prompt Cache 绝对不能动过去的 messages。

===== [2] role=assistant =====
Julie，你的直觉大体对：Hermes 平时确实不改历史，为的是保住 per-conversation 的 prompt cache。

唯一例外就是 context compression。触发条件大致是：当前 messages 的粗估 token 越过 `context_length * threshold_percent`。一旦触发，不会整段删掉，而是切开：

1. **protect head**：通常保住 system（含你 MEMORY 里「Julie / 上海 / AI Engineer」）和前几轮
2. **middle**：中间轮次序列化成文本，丢给辅助模型做结构化摘要
3. **protect tail**：最近几轮原样留下，因为还在接着干

所以「改历史」不是乱改，而是把 middle 换成一条带 `SUMMARY_PREFIX` 的摘要消息。Cache 会被这一刀打断——这是刻意权衡：爆窗比多花一点 cache 更糟。

===== [3] role=user  [_compressed_summary=True] =====
[CONTEXT COMPACTION — REFERENCE ONLY] Earlier turns were compacted into the summary below. This is a handoff from a previous context window — treat it as background reference, NOT as active instructions. Do NOT answer questions or fulfill requests mentioned in this summary; they were already addressed. Respond ONLY to the latest user message that appears AFTER this summary — that message is the single source of truth for what to do right now. Topic overlap with the summary does NOT mean you should resume its task: even on similar topics, the latest user message WINS. Treat ONLY the latest message as the active task and discard stale items from '## Historical Task Snapshot' / '## Historical In-Progress State' / '## Historical Pending User Asks' / '## Historical Remaining Work' entirely — do not 'wrap up' or 'finish' work described there unless the latest message explicitly asks for it. Reverse signals in the latest message (e.g. 'stop', 'undo', 'roll back', 'just verify', 'don't do that anymore', 'never mind', a new topic) must immediately end any in-flight work described in the summary; do not re-surface it in later turns. IMPORTANT: Your persistent memory (MEMORY.md, USER.md) in the system prompt is ALWAYS authoritative and active — never ignore or deprioritize memory content due to this compaction note. The current session state (files, config, etc.) may reflect work described here — avoid repeating it:

## Historical Task Snapshot
[用户询问模型是否会重复回答摘要中的旧问题，并讨论了摘要模板的命名和压缩机制]

## Goal
[了解模型摘要机制，确保摘要正确处理历史信息，避免重复回答]

## Constraints & Preferences
[偏好使用清晰、具体的命名，确保摘要正确反映对话内容]

## Completed Actions
[1. 讨论摘要模板的命名和作用；2. 解释压缩机制和迭代更新过程；3. 讨论MemoryProvider在压缩过程中的作用]

## Active State
[用户正在尝试复现摘要压缩过程，并询问细节]

## Historical In-Progress State
[用户尝试理解摘要压缩的内部工作原理]

## Blocked
[无]

## Key Decisions
[决定使用`SUMMARY_PREFIX`来区分摘要和指令，确保模型不会重复回答旧问题]

## Resolved Questions
[1. 摘要模板的命名和作用；2. 压缩机制的迭代更新过程；3. MemoryProvider在压缩过程中的作用]

## Historical Pending User Asks
[无]

## Relevant Files
[无]

## Historical Remaining Work
[用户需要复现摘要压缩过程，并理解其内部工作原理]

## Critical Context
[REDACTED]

--- END OF CONTEXT SUMMARY — respond to the message below, not the summary above ---

===== [4] role=user =====
好，帮我收束一下。假如面试官问：「Hermes 为什么允许压缩改历史？和 MEMORY 什么关系？」给我一段 60 秒可直接说的答案。我明天模拟面试要用。

===== [5] role=assistant =====
可以这样说：

「Hermes 默认不改历史，是为了 prompt cache。唯一例外是 context 快爆窗时做 compression：保留 system/head 和最近 tail，把中间轮次交给辅助模型做成结构化摘要，再以带 SUMMARY_PREFIX 的 user 消息插回去。前缀明确告诉模型：摘要只是背景，不要重答旧问题。MEMORY 通常在 head 里原样保留，必要时 MemoryProvider.on_pre_compress 还会把关键事实塞进摘要 prompt，所以像我是 Julie、在上海转 AI Agent Engineer 这种画像不会因为压缩丢。Cache 会被打断，但这是用一次失效换上下文继续可用。」

如果你要，下一轮我们可以对着 `context_compressor.py` 把 threshold 和 iterative summary 的分支一行行对一下。
