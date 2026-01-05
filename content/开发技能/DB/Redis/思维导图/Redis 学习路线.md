---

mindmap-plugin: basic

---

# basic

## 概念基础
- redis 是什么
- 为什么用 Redis
- 应用于哪些场景

## 数据结构
- 5种基础类型
	- String
	- Hash
	- List
	- Set
	- ZSet
- 3种特殊类型
	- HyperLogLog
	- Bitmap
	- Geo
- Stream 类型（v5.0）
- 对象机制
	- redisObject
	- 对象共享
	- 对象淘汰
- 底层数据结构

## 核心知识
- 持久化
	- RDB
	- AOF
	- 混合模式（4.0）
- 订阅/发布
	- 基于频道（Channel）
	- 基于模式（pattern）
- 事件机制
	- 文件事件
		- NIO，Reactor模型
	- 时间事件
- 事务
	- 标准的事务执行
	- CAS操作实现乐观锁

## 高可用 | 可扩展
- 哨兵机制（Redis Sentinel）
- 分片技术（Redis Cluster）