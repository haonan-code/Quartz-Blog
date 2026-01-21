一句话总结：带目录的说明书，或者说是一种渐进式披露提示词的机制

skills 将提示词分为三个部分
![PixPin_2026-01-13_10-23-54.png](https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/20260113102411385.png)

AI 使用 skills时，只是先把目录加载进提示词，然后根据需要再决定是否查阅正文与附件。
这样比起传统的Prompt或者MCP的方式，Agent Skills的最大好处是大幅降低了token的消耗与提示词的复杂度

Agent Skills 调用过程
![PixPin_2026-01-13_10-32-51.png](https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/20260113103254011.png)

Agent Skills 在有指令层和资源层情况下的调用过程
![PixPin_2026-01-13_10-39-10.png](https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/20260113103936294.png)

Agent Skills 与 MCP

|              | 侧重点  | 类比      | Token 消耗 | 核心主体        | 编写难度 |
| ------------ | ---- | ------- | -------- | ----------- | ---- |
| Agent Skills | 提示词  | 带目录的说明书 | 低        | Markdown 文件 | 低    |
| MCP          | 工具调用 | 标准化工具箱  | 高        | 软件包         | 高    |

Agent Skills 与 MCP 协作调用：
SKills 负责披露提示词，MCP 负责具体的工具调用

## 概念
Agent Skill：大模型可以随时翻阅的说明文档
## 基本用法

## 高级用法
### Reference

### Script

## 与MCP比较
































