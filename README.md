# LeeYe Init

LeeYe Init 是一套面向 Debian、Ubuntu 系统的 VPS 的中文交互式初始化脚本。它把系统更新、UFW、SSH、Fail2ban、Swap、BBR 和管理用户等常用操作集中到一个菜单中，每次只执行选择的模块。

当前脚本版本：`1.1.2`

## 功能

| 菜单 | 功能 | 主要行为 |
| --- | --- | --- |
| 1 | 系统更新与环境安装 | 刷新 APT 索引、执行普通软件包升级、安装必要环境或全能环境 |
| 2 | 防火墙 | 添加 TCP/UDP 放行规则、查看规则、按编号删除规则、启用 UFW |
| 3 | SSH 端口 | 生成或设置高位端口，分两阶段切换并验证监听状态 |
| 4 | Fail2ban | 按 SSH 的有效端口配置 `sshd` jail，设置封禁时间、统计窗口、重试次数和白名单 |
| 5 | 禁用密码登录 | 验证指定用户的公钥登录后，关闭 SSH 密码认证和键盘交互认证 |
| 6 | 时区 | 设置常用或自定义时区，检查并启用系统现有的时间同步服务 |
| 7 | Swap | 根据内存和磁盘空间给出建议值，创建、替换或删除脚本管理的 `/swapfile` |
| 8 | BBR | 检查当前内核能力，启用 `tcp_bbr`、`fq` 和对应的持久化配置 |
| 9 | 用户管理 | 创建或完善非 root 管理用户，配置密码、公钥及 sudo 验证方式 |
| 99 | 状态检查 | 汇总系统更新、UFW、SSH、Fail2ban、时区、Swap、BBR 和 sudo 组状态 |

脚本不提供一键初始化。完成一个模块后会返回主菜单，你可以根据当前服务器的情况决定下一步。


## 快速开始

在服务器上下载或克隆本仓库后执行：

```bash
cd leeye-init
chmod 700 leeye-init
sudo ./leeye-init
```

也可以直接交给 Bash 运行：

```bash
sudo bash leeye-init
```

首次进入交互菜单时，脚本会询问是否安装快捷命令。确认后会执行以下操作：

- 安装脚本到 `/usr/local/bin/leeye-init`
- 创建 `/usr/local/bin/ini`    

以后输入 `sudo ini` 即可打开主菜单。普通用户在交互式终端直接执行 `ini` 时，脚本也会尝试通过 `sudo` 提权。

## 命令行参数

安装快捷命令后可使用：

```bash
sudo ini                 # 打开交互菜单
sudo ini --dry-run       # 预览，不写入目标配置
sudo ini --check         # 只读检查当前状态
sudo ini --restore BATCH # 从指定备份中恢复配置
ini --help               # 查看帮助
```


## 建议操作顺序

1. 确认云厂商安全组、VNC 或控制台救援入口可用，并保持当前 SSH 会话。
2. 运行 `sudo ./leeye-init`或`sudo ini`，进入菜单，查看准备执行的命令和配置内容。
3. 通过菜单 1 更新系统并安装所需环境。
4. 如需修改 SSH 端口，先在云厂商安全组和 UFW 中放行新端口，再使用菜单 3。
5. 创建管理用户并配置 SSH 公钥，确认新用户可以登录后，再禁用密码认证。
6. 按需要配置 Fail2ban、时区、Swap 和 BBR，最后运行菜单 99 检查状态。




## 备份、日志与恢复

脚本把运行日志写入：

```text
/var/log/vps-init/<运行时间>-<进程号>.log
```

纳入事务的配置会保存到独立批次：

```text
/var/backups/vps-init/<运行标识>-<序号>-<模块>/
├── files/
├── manifest.tsv
└── transaction.status
```

查看可用批次：

```bash
sudo ls -1 /var/backups/vps-init
```

恢复指定批次：

```bash
sudo ini --restore <备份批次名称>
```

恢复前，脚本会核对批次名称、清单、备份摘要以及目标文件的当前摘要。目标文件在该批次完成后又被修改时，脚本会停止自动恢复，避免覆盖后续人工调整。恢复仍需输入 `RESTORE` 确认，并会重新校验或加载受影响的 SSH、UFW、Fail2ban、BBR、Swap 和 sudo 配置。

配置备份只覆盖该事务记录的文件与运行状态。APT 更新、软件包安装、用户创建、密码修改及已经执行的系统命令无法整体回滚。脚本发现未完成事务时只会提示批次和状态，不会在下次启动时自行回滚。

## 脚本管理的主要路径

| 路径 | 用途 |
| --- | --- |
| `/usr/local/bin/leeye-init` | 安装后的主脚本 |
| `/usr/local/bin/ini` | 快捷命令 |
| `/etc/ssh/sshd_config.d/00-vps-init-port.conf` | SSH 端口配置 |
| `/etc/ssh/sshd_config.d/00-vps-init-auth.conf` | SSH 认证配置 |
| `/etc/fail2ban/jail.d/vps-init-sshd.local` | Fail2ban SSH jail |
| `/etc/sudoers.d/90-vps-init-<用户名>` | 脚本管理的 sudo 免密码规则 |
| `/etc/modules-load.d/vps-init-bbr.conf` | BBR 模块加载配置 |
| `/etc/sysctl.d/99-vps-init-bbr.conf` | BBR 内核参数 |
| `/swapfile`、`/etc/fstab` | 脚本管理的 Swap 文件与持久化条目 |



在生产服务器上操作前，请先确认你有可用备份和独立的控制台登录方式。
