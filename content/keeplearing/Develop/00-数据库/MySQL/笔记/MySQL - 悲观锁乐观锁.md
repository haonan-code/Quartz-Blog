> 写写冲突（Write-Write Conflict）是数据库并发控制中最常见的问题之一，尤其是在多个事务**同时对同一数据进行修改**时。为了解决这类问题，常见方案有两种：

## 悲观锁
### 核心思想：
> 悲观地认为冲突一定会发生，所以在操作数据之前就加锁，别人不能动。

### 实现方式（MySQL/InnoDB）：
- 利用数据库的行级锁（例如`SELECT ... FOR UPDATE`）
- 在读数据时就加上排他锁，**阻塞其他事务对该行的读写**

### 举例
```sql
BEGIN;
SELECT * FROM product WHERE id = 1 FOR UPDATE;
UPDATE product SET stock = stock - 1 WHERE id = 1;
COMMIT;
```

- `FOR UPDATE` 会锁住这行数据
- 其他事务想修改这行，就会**等待当前事务提交或回滚**
- 这就是“悲观”控制，防止别人同时修改

### 优点
- 冲突概率高时很稳妥，**安全可靠**
- 避免脏写
### 缺点
- 并发低时也加锁，可能导致**性能下降**
- 有可能产生死锁
- 对高并发业务不友好

## 乐观锁
### 核心思想
> 乐观地认为冲突不会发生，操作时不加锁，但在提交时做“版本校验”来判断有没有冲突。

### 实现方式
- 在数据表中加一个 **版本号字段 `version`** 或 **时间戳字段**
- 读取时获取当前版本号
- 更新时加上 **`WHERE version = ?`** 条件
- 如果没人改，更新成功；如果有人改过，`UPDATE` 失败 → 说明冲突了

### 举例
表结构
```sql
CREATE TABLE product (
  id INT,
  stock INT,
  version INT,
  PRIMARY KEY (id)
);
```
操作流程：
1. 查询：
```sql
SELECT stock, version FROM product WHERE id = 1;
-- 得到 stock = 100, version = 3
```

2. 更新：
```sql
UPDATE product
SET stock = 99, version = version + 1
WHERE id = 1 AND version = 3;
```

- 如果 `version = 3` 成立，就更新成功（说明没人动）
- 如果版本不对，`UPDATE` 返回 0 行，说明有并发冲突 → 可以重试

### 优点
- **无锁操作**，性能高
- 非阻塞，适合**读多写少、高并发场景**
### 缺点
- 编码复杂（要手动加字段、逻辑）
- **冲突多时重试成本高**
- 不适合写多的场景

## 总结
- **悲观锁：** 直接锁住数据，防止其他写操作发生
- **乐观锁：** 允许其他操作，但在提交阶段判断是否有冲突，冲突就回滚/重试