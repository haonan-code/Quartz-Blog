## 问题1：执行 git push 时，终端报远端意外挂掉了错误：

```
枚举对象: 35, 完成.
对象计数中: 100% (35/35), 完成.
使用 4 个线程进行压缩
压缩对象中: 100% (30/30), 完成.
写入对象中: 100% (30/30), 341.28 KiB | 976.00 KiB/s, 完成.
总共 30 （差异 10），复用 0 （差异 0）
Connection to github.com closed by remote host.
fatal: 远端意外挂断了
fatal: 远端意外挂断了
```

解决：该问题应该是提交的改动内容大小超过了git设置的Buffer大小，使用下列命令对git的Buffer扩容即可。

```bash
git config http.postBuffer 100000000
git config ssh.postBuffer 100000000
```

## 问题2：Jetbrains 系列 IDE 使用 Undo Commit，Revert Commit，Drop Commit区别

|               | 是否删除对代码的修改 |    是否删除Commit记录    | 是否会新增Commit记录 |
| :-----------: | :--------: | :----------------: | :-----------: |
|  Undo Commit  |     不会     | 未 Push 会，已 Push 不会 |      不会       |
| Revert Commit |     会      |         不会         |       会       |
|  Drop Commit  |     会      | 未 Push 会，已 Push 不会 |      不会       |
|               |            |                    |               |

### 一、Undo Commit

**适用情况：代码修改完了，已经Commit了，但是还未push，然后发现还有地方需要修改，但是又不想增加一个新的Commit记录。这时可以进行Undo Commit，修改后再重新Commit。**

如果已经进行了Push，线上的Commit记录还是会存在的

简单来说，就是撤销了你Commit的这个动作。详细解释下：

1. 首先，对项目进行了代码修改，然后进行 commit 操作
	![pict1](Pasted%20image%2020241213144223.png)
2. 确认 Commit 之后（未进行push）
   
    ![](Pasted%20image%2020241213150756.png)
    
3. 进行 Undo Commit 操作
   
    ![](Pasted%20image%2020241213150903.png)
    
4. 执行后和未 Commit 之前完全一样
   
    ![](Pasted%20image%2020241213151135.png)
    

### 二、[Revert](https://so.csdn.net/so/search?q=Revert&spm=1001.2101.3001.7020) Commit

**会新建一个 Revert “xxx Commit”的Commit记录，该记录进行的操作是将"xxx Commit"中对代码进行的修改全部撤销掉。**

1. 首先，对项目进行了代码修改，然后进行 commit 操作
   
    ![](Pasted%20image%2020241213151235.png)
    
    Commit 之后：
    
    ![](Pasted%20image%2020241213151248.png)
    
2. 进行 Revert Commit
   
    ![](Pasted%20image%2020241213151349.png)
    
    执行成功后：
    
    可以看到，新增了 Commit 记录【Revert “测试 Revert Commit”】，该记录中将【测试 “Revert Commit”】中对代码进行的修改删除了。
    
    ![](Pasted%20image%2020241213151407.png)
    
    ![](Pasted%20image%2020241213151419.png)
    

### 三、Drop Commit（慎用）

**未push的Commit记录:**

**会删除Commit记录，同时Commit中对代码进行的修改也会全部被删除**

**已push的Commit记录:**

**区别在于线上的Commit记录不会被删除**

1. 修改代码，然后进行 commit
   
    ![](Pasted%20image%2020241213151433.png)
    
    ![](Pasted%20image%2020241213151444.png)
    
2. 进行 drop commit 操作后
   
    commit 记录被删除，代码修改也被删除
    
    ![](Pasted%20image%2020241213151515.png)
    
3. 已 push 的 commit 记录
   
    ![](Pasted%20image%2020241213151525.png)

## 问题3：rebase 和 merge 区别
### 定义：
- **Merge（合并）**：
	- 将两个或多个分支的历史合并，创建一个新的 **合并提交**（merge commit），保留所有分支的提交历史。
	- 通常用于将特性分支（feature branch）合并到主分支（如 main）
- **Rebase（变基）**：
	- 将一个分支的提交“移动”到另一个分支的最新提交之上，改写提交历史，使其看起来像是线性历史。
	- 常用于清理提交历史或将主分支的更新应用到特性分支
### 直观图：
- **Merge**：
```text
A --- B --- C (main)
 \         /
  D --- E (feature)
```
合并后：
```text
A --- B --- C --- F (main, F 是合并提交)
 \         /
  D --- E (feature)
```
- **Rebase**：
```text
A --- B --- C (main)
 \
  D --- E (feature)
```
变基后：
```text
A --- B --- C --- D' --- E' (feature)
```

> 通俗理解：先更新了最新 main 的“地基”，再把自己建好的“房子”搬上去
### 优缺点：
**Merge**：
优点：
- **保留完整历史**：分支的上下文和历史清晰，适合追踪谁做了什么。
- **安全**：不会改写提交历史，适合多人协作和已推送的分支。
- **简单**：冲突只需解决一次，操作直观。
缺点：
- **历史复杂**：频繁合并可能导致提交历史杂乱，包含大量合并提交。
- **可读性降低**：长期项目中，分支分叉和合并提交可能难以追溯。
**Rebase**：
优点：
- **简洁历史**：线性历史更易阅读，适合追求干净提交记录的项目。
- **便于审查**：提交历史像单人开发，方便代码审查和日志分析。
- **同步更新**：将主分支的最新提交应用到特性分支，减少后续合并冲突。
缺点：
- **改写历史**：对已推送的提交执行 rebase 会导致协作问题，需强制推送（git push --force）。
- **冲突复杂**：可能需要逐个提交解决冲突，操作繁琐。
- **风险较高**：误操作可能丢失提交，需谨慎使用。

### 流程图
![[Pasted image 20250608185106.png]]







































