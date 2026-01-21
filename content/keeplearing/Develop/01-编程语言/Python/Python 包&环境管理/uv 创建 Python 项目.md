### Python 版本管理
#### 配置 python 环境

uv 安装后查看可安装的 python 环境列表：
```bash
uv python list
```
运行这条命令后可查看本地已安装的 python 环境以及可安装的 python 版本
![](Pasted%20image%2020250326105653.png)


### 初始化项目
#### 1. 使用 uv 创建（推荐）
可直接通过 uv 创建项目：
```bash
uv init 项目名称 --python 3.13.2 
```
指定的 python 版本为项目可支持的最低版本，即 >= 3.13.2，可在 pyproject.toml 文件中查看：
![](Pasted%20image%2020250326110728.png)

项目创建后的目录：
![](Pasted%20image%2020250326110038.png)
uv 会自动给项目创建上述文件，.git 文件需自行绑定远程仓库，.gitignore 文件需自行配置忽略文件和目录


#### 2. 手动创建
手动创建项目文件夹，然后使用`uv init` 初始化 python 项目

### 创建虚拟环境
创建的虚拟环境 python 版本应该满足 pyproject.toml 文件的要求：
```bash
uv venv -p 3.13.2
```
创建后激活虚拟环境：
```bash
source .venv/bin/activate
```
退出虚拟环境：
```bash
deactivate
```

### 安装库
#### 通过 uv add 进行安装库，速度快（推荐）
```bash
uv add "fastapi[all]"
```
运行 `uv add <package>`时，uv 会自动：
- 安装新包
- 更新 `pyproject.toml`
- 更新 `uv.lock`
#### 兼容 pip 的命令安装
```bash
uv pip install "fastapi[all]"
```
上述命令的行为类似于 `pip install`，只会安装包到当前环境，不会修改 pyproject.toml 和 uv.lock 文件。
##### 如果需要更新依赖文件：
如果使用 `uv pip install` 安装了某些包，然后想让 `uv.lock` 反映这些更改，需要手动运行：
```bash
uv pip compile
```
然后再运行：
```bash
uv sync
```
让环境完全匹配锁文件

### 同步依赖
拉取其他项目时，可以使用`uv sync` 同步 Python 项目依赖，使项目的虚拟环境严格匹配 `pyproject.toml` 和 `uv.lock` 文件中指定的版本。
#### 使用场景
- 确保依赖一致性
	当你在 `pyproject.toml` 中定义了依赖项，并生成了 `uv.lock` 文件，`uv sync` 可以确保本地环境安装的依赖与 `uv.lock` 中锁定的版本完全匹配。
- 团队协作
	在团队项目中，开发者可以使用 `uv sync` 让每个人的开发环境保持一致，避免因依赖版本不同导致的问题。
- 部署环境管理
	在生产环境、CI/CD 流程或 Docker 容器中使用 `uv sync` 来快速安装并锁定依赖，而不依赖 `pip install`。

#### 常见用法
```bash
uv sync
```
该命令会读取 `uv.lock` 文件，并安装或删除依赖，使当前环境与锁文件匹配。
如果没有 `uv.lock`，则可以先运行：
```bash
uv pip compile
```
来生成 `uv.lock`，然后再执行 `uv sync`。

### 运行
单个脚本文件可使用下列命令执行：
```bash
uv run xxxx.py
```
像 `fastapi` 这种库可能有特定的命令启动项目：
```bash
fastapi dev main.py --host 0.0.0.0 --port 8000
```





