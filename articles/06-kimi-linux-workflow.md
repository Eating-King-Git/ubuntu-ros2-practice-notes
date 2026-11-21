# 在 Ubuntu 上使用 Kimi：从提问到环境配置和任务交接

作者：Eating-King-Git。根据本人操作记录与官方文档整理，含 AI 辅助写作和核查。整理日期：2026-11-21。

## 1. 网页里的 Kimi 和终端里的 Kimi 有什么区别？

只想解释报错、整理命令或讨论方案，可以在 Kimi 网页中提问；需要 AI 在自己的项目目录读取代码、修改文件和执行命令，则使用 **Kimi Code CLI**，并核对它实际获得的权限。[Kimi Code 官方介绍](https://www.kimi.com/code/docs/)

本系列的 Ubuntu 问题来自我的实际配置经历。本篇把其中可复用的提问、验证和交接方法整理为 Kimi 流程；安装与功能按 **2026-11-21 官方文档**核对，**没有把本轮写作描述为已经用 Kimi 在这台电脑重跑配置。**

流程是：

**检查命令是否已存在 → 安装并登录 → 进入正确项目 → 先检查再修改 → 验证输出 → 保存交接。**

## 2. 在 Ubuntu 上怎样安装？

先在普通终端检查，避免重复安装：

```bash
command -v kimi
```

如果已有命令，先核对：

```bash
kimi --version
type -a kimi
```

当前官方推荐的 Linux 安装命令是：

```bash
curl -fsSL https://code.kimi.com/kimi-code/install.sh | bash
```

它会下载并执行官方安装脚本。确认域名和来源后再运行，不要用陌生文章中的改写下载地址。安装后重新打开终端，执行 `kimi --version` 验证。

已有合适 Node.js 环境时，也可选择 npm 途径：

```bash
node --version
npm install -g @moonshot-ai/kimi-code
```

官方当前要求 npm 途径使用 **Node.js 22.19.0 或更高版本**；两种方式选一种即可。旧的 Python／uv `kimi-cli` 已进入迁移流程，新安装不再沿用旧教程中的安装路径。[官方安装指南](https://www.kimi.com/code/docs/kimi-code-cli/guides/getting-started.html)、[旧版迁移说明](https://www.kimi.com/code/docs/kimi-code-cli/guides/migration.html)

如果报 DNS 或连接超时，先处理网络；如果报权限错误，先检查安装目录。不把“重新装一遍”当作所有问题的解法。

## 3. 登录时要填写 API Key 吗？

不一定。在普通终端运行：

```bash
kimi
```

进入 Kimi 输入框后输入：

```text
/login
```

官方提供 Kimi Code OAuth 登录和 Kimi Platform API Key 等接入方式。使用 Kimi Code 时，可按页面完成账号与设备码授权，无需自行把密钥写进项目。是否有可用权益，以自己的账户显示为准；Kimi 网页能聊天，不直接等于所有编程服务权益都可用。[平台与模型说明](https://www.kimi.com/code/docs/kimi-code-cli/configuration/providers.html)

密码、设备码和密钥只在对应的官方认证流程使用，不发给 AI、不写进交接文件。也不要把不同平台的 Key 与服务地址混搭；遇到认证错误先核登录方式。

## 4. 命令应该输入在哪个窗口？

| 输入位置 | 输入内容 | 示例 |
|---|---|---|
| Ubuntu 普通终端 | Shell 命令 | `pwd`、`ls -lah`、`kimi` |
| Kimi 对话输入框 | 希望 AI 完成的任务 | “检查已有 ROS 环境，先别安装” |
| Kimi 斜杠命令 | 客户端内置操作 | `/login`、`/help`、`/sessions` |
| 程序的密码提示 | 本机认证输入 | sudo 密码通常不显示字符 |

初学时，自己要运行的命令放普通终端，给 AI 的任务放对话框，最容易分清。Kimi 也有 Shell 模式，但先把两种输入区分好，再使用快捷功能。[交互与输入](https://www.kimi.com/code/docs/kimi-code-cli/guides/interaction.html)

例如创建一个独立练习目录：

```bash
mkdir -p ~/projects/linux-lab
cd ~/projects/linux-lab
pwd
kimi
```

随后发给 Kimi：

```text
先确认当前工作目录，说明你准备读取哪些文件。
只检查已有环境，不安装软件、不修改配置。
```

网页提问时，如果没有连接本地工具，就由自己执行命令，再提供去敏后的结果；不能把聊天回答当成命令已经在自己的电脑运行。

## 5. 怎样让它先检查，再动手？

复杂任务可以从计划模式开始：

```bash
kimi --plan
```

已经进入交互界面时，可使用 `/plan`。当前官方说明中，计划模式优先调查并生成方案；权限可通过 `/permission` 查看。默认的 Always Ask 模式会对修改文件和执行命令等操作进行审批。[命令参考](https://www.kimi.com/code/docs/kimi-code-cli/reference/kimi-command.html)、[交互与审批说明](https://www.kimi.com/code/docs/kimi-code-cli/guides/interaction.html)、[斜杠命令说明](https://www.kimi.com/code/docs/kimi-code-cli/reference/slash-commands.html)

**计划模式、工具审批、Linux 用户权限是三件事。** 计划模式不等于操作系统级的只读沙箱，AI 的授权也不会自动让普通用户成为 root。

配置系统环境时，先看清具体要安装哪个包、修改哪个文件、如何回退，再决定是否执行。不为处理一次报错就关闭所有审批；需要 sudo 密码时，只在本人确认的本机终端中输入。

## 6. 怎样描述任务，才能避免重复安装？

与 AI 交互时，把现状、目标、限制和完成标准一起给出：

```text
我正在 Ubuntu 22.04 上配置 ROS 2 Humble。
先读项目中的 HANDOVER.md 和 ENVIRONMENT.md，
检查已有安装与工作区；已经完成的步骤不要重做。

本次只确认 talker/listener 能否通信。
先说明检查步骤，不连接机械臂，不启动会使能电机的节点。
如果缺依赖，指出具体包名、用途和安装命令，再处理授权。

每一步给出：实际命令、预期结果、实际结果、下一步。
结束时写下尚未解决的问题，不把“建议执行”记成“已执行”。
```

这是一份可直接改写的提问模板，不是环境已通过验证的凭证。收到回答后，仍要核对实际输出。

我在配置过程中最需要保留的习惯是：一次只解决一个明确问题。先确认 ROS 能加载，再处理 SDK；不要同时改网络、装依赖、编译驱动和连接真机，否则失败后很难定位。

## 7. AI 说“完成了”，还要核对什么？

| AI 的结论 | 应提供的证据 |
|---|---|
| 已安装 | 版本、包记录或可执行文件位置 |
| 已构建 | 实际构建退出码与目标文件 |
| 通信正常 | 接收端收到本次发送的消息 |
| 重启后可用 | 新终端或重启后的再次检查 |
| 已写交接 | 文件存在，内容与实际状态一致 |

例如它只给出一条安装命令，就还没有“完成安装”；只看到进程启动，也还不能说设备通信正常。让它把未知单独列出来，往往比反复追问“是不是好了”更有效。

Git 项目开始前看 `git status --short`，结束后看 `git diff`；原本就存在的修改要保留。没有 Git 的配置文件，先保存改前副本，再进行针对性修改。

## 8. 对话太长或换会话，怎样接续？

在交互界面中，`/compact` 可压缩上下文；`/sessions` 可选择历史会话。普通终端的 `kimi --continue` 继续当前工作目录最近一次会话。[会话与上下文](https://www.kimi.com/code/docs/kimi-code-cli/guides/sessions.html)

但压缩后的对话不应成为唯一记录。我采用三份项目文件：

- `AGENTS.md`：长期工作约定。
- `ENVIRONMENT.md`：系统、版本、依赖和验证入口。
- `HANDOVER.md`：这次完成、失败、未知与下一步。

Kimi 的 `/init` 可以分析项目并生成 `AGENTS.md`，生成后仍需检查，不能直接覆盖自己已有的规则。后两份是自己的文件约定，要在提示词中明确要求读取。[斜杠命令说明](https://www.kimi.com/code/docs/kimi-code-cli/reference/slash-commands.html)

新会话可以这样开始：

```text
先读 AGENTS.md、ENVIRONMENT.md 和 HANDOVER.md。
用三句话说明已完成什么、哪里还不确定、下一步做什么。
先核文件与当前环境，不把旧计划中的待办当作现在仍未完成。
```

跨电脑还需要迁移项目文件和依赖。能恢复聊天，不代表另一台电脑已经具有同样的软件和数据。

## 9. 本篇常用命令有哪些？

| 命令 | 用途 |
|---|---|
| `pwd` | 确认当前目录 |
| `type -a kimi` | 检查命令位置与是否有多份安装 |
| `kimi --version` | 记录实际版本 |
| `kimi --help` | 核对本机支持的参数 |
| `kimi --plan` | 从计划模式开始新会话 |
| `kimi --continue` | 继续当前目录最近的会话 |
| `git diff` | 查看实际文件变化 |
| `sed -n '1,100p' HANDOVER.md` | 读交接前100行 |

---

[系列目录](../README.md) · [上一篇：第 05 篇](05-terminal-network.md) · [下一篇：第 07 篇](07-kimi-connection-troubleshooting.md)
