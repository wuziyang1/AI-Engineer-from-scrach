# 步骤 1 · 切开（Split）

对应逻辑图：`messages → Split → head / middle / tail`

- protect_first_n = 3
- protect_last_n = 2
- head=3 | middle=12 | tail=2

| 文件 | 含义 |
|------|------|
| `system_prompt.md` | head 里的 system / MEMORY |
| `head.md` | protect head（原样保留） |
| `middle.md` | 待摘要的中间轮次 |
| `tail.md` | protect tail（原样保留） |
