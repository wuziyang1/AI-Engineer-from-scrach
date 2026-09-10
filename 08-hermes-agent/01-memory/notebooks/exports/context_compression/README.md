# Context Compression 流水线产物

```text
01_切开/          ← Split：system / head / middle / tail
02_middle摘要/    ← middle → 压缩prompt → 摘要 → 包装消息
03_拼接/          ← head + 摘要消息 + tail
```

讲解顺序：`01_切开` → `02_middle摘要` → `03_拼接`。
