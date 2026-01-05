## Linux常用命令
### 一、文件与目录操作
查看文件内容
```bash
ls
ls -l      # 详细信息
ls -a      # 显示隐藏文件
ls -lh     # 文件大小人类友好
```
切换目录
```bash
cd /path/to/dir
cd ~       # 回家目录
cd -       # 返回上一个目录
```
创建/删除
```bash
mkdir test
mkdir -p a/b/c      # 递归创建

rm file.txt
rm -r dir           # 删除目录
rm -rf dir          # 强制删除（慎用！）
```
文件复制/移动/重命名
```bash
cp a.txt b.txt
cp -r dir1 dir2     # 复制目录

mv a.txt b.txt      # 重命名
mv file /tmp        # 移动
```
查看文件内容
```bash
cat file
head -n 20 file     # 前20行
tail -f log.txt     # 实时查看日志
less file           # 分页浏览（推荐）
```

### 二、文件搜索与文本处理
查找文件
```bash
find / -name "*.conf"
find . -type f -size +10M

```
查找字符串
```bash
grep "keyword" file
grep -r "keyword" .     # 递归搜索

```
文本编辑
```bash
nano file
vim file

```
### 三、系统信息查看
系统整体状况
```bash
top
htop               # 更美观（若已安装）
uptime
free -h
df -h              # 磁盘使用
du -h --max-depth=1

```
硬件信息
```bash
lscpu
lsblk
lspci
lsusb
dmidecode

```
发行版 & 内核
```bash
uname -a
cat /etc/os-release

```
### 四、网络命令
查看端口
```bash
ss -tlnp
netstat -tlnp
```
Ping 测试
```bash
ping 8.8.8.8
ping -c 4 www.google.com

```
下载工具
```bash
curl http://example.com
wget http://example.com/file

```
查看网络连接
```bash
ip a
ip r
ip link

```
### 五、权限相关
文件权限
```bash
chmod 755 file
chmod u+x script.sh

```
修改所有者
```bash
chown user file
chown user:group file

```
sudo 执行
```bash
sudo command

```
### 六、压缩解压
tar
```bash
tar -czvf a.tar.gz dir/
tar -xzvf a.tar.gz

```
zip / unzip
```bash
zip a.zip file1 file2
unzip a.zip

```
### 七、软件管理（根据发行版）
Debian / Ubuntu
```bash
apt update
apt install nginx
apt remove nginx

```
CentOS / RHEL
```bash
yum install httpd
dnf install nginx

```
### 八、服务管理（systemd）
```bash
systemctl start nginx
systemctl stop nginx
systemctl restart nginx
systemctl status nginx
systemctl enable nginx
systemctl disable nginx
```
### 九、进程管理
```bash
ps aux
ps -ef | grep nginx
kill 1234
kill -9 1234      # 强制

```
### 十、常用实用命令
```bash
history            # 历史命令
clear              # 清屏
watch -n 1 ss -tlnp
df -Th             # 查看磁盘类型
hostnamectl        # 查看/修改主机名

```


## Linux 命令常见问题
### ll 和 ls 的区别
### 如何查看某服务器端口是否可用
#### 1. 查看所有正在监听的端口（最常用）
- **`netstat -tuln`**: 显示所有 TCP/UDP 端口的监听状态（Listen），使用数字显示（-n），并显示进程ID (PID)和程序名（-l, -t, -u, -p）。
- **`ss -tuln`**: `ss` 是 `netstat` 的替代品，通常更快、更高效，功能类似。
```bash
netstat -tuln | grep <端口号>
ss -tuln | grep <端口号>
```
_如果输出有结果，表示该端口正在监听（可用）_。
#### 2. 查看指定端口被哪个进程占用
- **`lsof -i:<端口号>`**: 列出使用指定端口的进程信息。
    - **示例**: `lsof -i:80` 查 80 端口。
- **`fuser -v <端口号>/tcp`**: 显示使用该 TCP 端口的进程信息。
    - **示例**: `fuser -v 80/tcp` 查 80 端口。
- **`netstat -ntlp | grep <端口号>`**: 配合 `grep` 查找特定端口的占用进程。

#### 3. 检查防火墙是否开放（外部访问）
- **`firewall-cmd` (CentOS/RHEL)**:
    - 检查指定端口（如 22/tcp）是否已开放：`{Link: firewall-cmd --query-port=22/tcp https://cloud.tencent.com/developer/article/2053640}`。
    - 列出所有开放的端口：`firewall-cmd --list-ports`。
- **`ufw` (Ubuntu/Debian)**:
    - 查看状态和已开放端口：`ufw status`。

#### 4. 尝试连接（探测端口是否能通）
- **`telnet <IP地址> <端口号>`**: 尝试连接到目标 IP 和端口。
    - 如果显示 "Connected to..." 成功，表示端口开放且网络可达。
    - **示例**: `telnet 127.0.0.1 80`。
- **`nc -zv <IP地址> <端口号>`**: `netcat` 工具，检查端口连通性，`-z` 零I/O模式，`-v` 详细输出。
    - **示例**: `nc -zv localhost 8080`。

### 查找一个文件，有名称 
`find -name "xxx.txt"`
### 日志滚动查看 
1. **`tail -f` (推荐，最常用)**
    - **命令:** `tail -f /path/to/your/log/file` (例如 `tail -f /var/log/syslog`)。
    - **功能:** 显示文件的最后部分，并持续跟踪文件增长，新内容会实时显示在屏幕底部。
    - **退出:** 按 `Ctrl + C`。
2. **`less` (更灵活)**
    - **命令:** `less /path/to/your/log/file`。
    - **功能:** 允许你向前、向后翻页查看日志，非常适合大文件。
    - **进入实时模式:** 进去 `less` 后，按 `Shift + F` (大写F) 即可开始实时追随新内容。
    - **退出实时模式:** 按 `Ctrl + C`。
    - **退出 `less`:** 按 `q`。

### linux chmod 的参数含义 怎么用
#### 介绍
`chmod` 是 Linux 中用于**更改文件或目录权限**的命令，主要通过**数字模式**（三位八进制数）或**符号模式**（u/g/o/a + /-/rwx）来设置所有者（u）、所属组（g）、其他用户（o）的读（r）、写（w）、执行（x）权限，常用参数有 `-R` (递归) 用于目录，用法简洁，如 `chmod 755 filename` 或 `chmod u+x filename`
#### 参数含义
1. **用户角色**:
    - `u` (User): 文件所有者.
    - `g` (Group): 文件所属用户组.
    - `o` (Others): 其他用户 (不属于所有者也不属于所属组).
    - `a` (All): 所有用户 (u、g、o 的组合).
2. **操作符**:
    - `+`: 添加权限.
    - `-`: 移除权限.
    - `=`: 设置权限 (覆盖原有).
3. **权限**:
    - `r` (Read): 读权限 (数字为 4).
    - `w` (Write): 写权限 (数字为 2).
    - `x` (Execute): 执行权限 (数字为 1)
#### 用法
**1. 数字模式 (最常用)**  
用三位八进制数表示所有者(u)、所属组(g)、其他(o)的权限，数字是 r、w、x 权限之和 (4+2+1=7)。 
- **7 (111)**: rwx (读、写、执行)
- **6 (110)**: rw- (读、写)
- **5 (101)**: r-x (读、执行)
- **4 (100)**: r-- (读)
- **示例**:
    - `chmod 755 script.sh`: 所有者有 rwx，组和其他用户有 r-x。
    - `chmod 644 file.txt`: 所有者有 rw-，组和其他用户有 r--。
**2. 符号模式**  
结合用户角色、操作符和权限来精确修改。
- `chmod u+x script.sh`: 给所有者添加执行权限。
- `chmod go-w file.txt`: 从所属组和其他用户移除写权限。
- `chmod u=rw,go=rwx file.txt`: 设置所有者为读写，组和其他为读写执行。
**3. 特殊参数**
- `-R` (Recursion): 递归修改目录及子文件/子目录的权限.
    - `chmod -R 777 mydir/` (谨慎使用！).
- `-c`: 显示权限确实改变的文件。
- `-v`: 显示详细权限变更信息。
## Linux 操作相关
### 用过SSH吗，怎么设置免密登录？
首先在本地生成 SSH 密钥，使用命令：
```bash
ssh-keygen -t ed25519
# 或 ssh-keygen -t rsa -b 4096
```
然后将公钥复制到远程服务器上，
**方法1：ssh-copy-id**
```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub root@服务器IP
```
输入一次密码后，就绑定成功。
**方法2：手动上传（通用方法）**
1. 查看公钥内容：
	`cat ~/.ssh/id_ed25519.pub`
2. 登录远程服务器，打开：
	`mkdir -p ~/.ssh`
	`nano ~/.ssh/authorized_keys`
3. 粘贴公钥内容
4. 修改权限
```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```
否则 SSH 会拒绝公钥登录。
### 进程挂了怎么办-在 linux 环境挂了应该怎么处理
先判断进程挂掉的原因
查看进程是否还存在
查看系统资源是否导致进程挂掉
查看日志（最关键）
如果进程卡死，尝试杀掉






