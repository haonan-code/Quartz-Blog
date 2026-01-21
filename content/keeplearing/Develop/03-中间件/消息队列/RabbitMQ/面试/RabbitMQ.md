**架构图：**
```nginx
Producer → Exchange → Queue → Consumer
```
- `Exchange` 控制路由策略（direct, fanout, topic）
- 队列可以绑定多个 exchange，实现灵活路由
**使用场景：**

| 场景        | 描述             |
| --------- | -------------- |
| 多消费者不同需求  | 一个消息分发给不同队列    |
| 邮件/短信通知系统 | 各模块可独立处理       |
| RPC 异步调用  | 使用 reply_to 模式 |



RabbitMQ 使用过延迟消息吗?
RabbitMQ 重复消费问题,展开说说?

