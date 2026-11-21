# Ubuntu、Kimi 与 ROS 2 实践笔记

从 Windows 与 Ubuntu 双系统安装开始，逐步整理桌面使用、终端网络、Kimi 协作、ROS 2 环境和 A3 机械臂接入。共 11 篇，采用问题—答案、操作流程和 Linux 命令表的形式，可以按顺序读，也可以直接查当前问题。

作者：Eating-King-Git。根据本人操作记录与官方文档整理，含 AI 辅助写作和核查。整理日期：2026-11-21。

## 文章目录

| 篇目 | 主要解决的问题 |
|---|---|
| [01 · 第一次安装 Ubuntu 双系统](articles/01-ubuntu-dual-boot.md) | 准备启动盘、核对分区、EFI 警告与 BitLocker 恢复页面 |
| [02 · 启动 U 盘恢复普通存储](articles/02-usb-reuse.md) | 容量变小、格式化失败，什么时候该停止操作 |
| [03 · 双系统安装与启动排错](articles/03-boot-troubleshooting.md) | 区分启动菜单、试用系统、安装器和已安装系统 |
| [04 · 中文输入、AppImage 与命令复制](articles/04-desktop-input-appimage.md) | 输入源、文件名、执行权限、FUSE 与粘贴格式 |
| [05 · 浏览器能上网，终端却下载失败](articles/05-terminal-network.md) | DNS、代理、TLS 和 APT 分层排查 |
| [06 · 在 Ubuntu 上使用 Kimi](articles/06-kimi-linux-workflow.md) | 网页与命令行、安装登录、任务拆分、权限和交接 |
| [07 · Kimi 连接失败怎么查](articles/07-kimi-connection-troubleshooting.md) | 最小请求、网络与认证、会话恢复和有效报错记录 |
| [08 · ROS 2 Humble 环境验证](articles/08-ros2-humble.md) | C++／Python 收发、工作区、RViz 与重启复查 |
| [09 · Python 与 C++ SDK 依赖排查](articles/09-sdk-python-cpp.md) | 虚拟环境、版本来源、CMake、JsonCpp 与 ROS 依赖 |
| [10 · Wi-Fi 上网、有线连接设备](articles/10-dual-network.md) | 双网卡、静态地址、默认路由、TUN 与协议验证 |
| [11 · A3 状态、RViz 与运动预览](articles/11-a3-rviz-preview.md) | 真实反馈、模型转换、运动学对照与虚拟预览 |

## 怎样使用这些笔记？

电脑环境以历史使用的 Ubuntu 22.04.5 与 ROS 2 Humble 为主。版本号用于说明当时的条件，不要求读者降级到相同版本；命令里的设备名、目录和示例参数要先核对。

文章内分别说明已有操作记录和补充指南。第 02 篇没有完整的 U 盘恢复成功回执；第 06—07 篇是按 Kimi 官方文档编写的使用与排错指南，没有重新在 Ubuntu 上实测 Kimi。各篇保留可用于 Kimi 的提问方法，示例提示词不冒充历史聊天原文。

机器人部分按“环境、网络、反馈、显示、预览、实际运动”分别验收。已有虚拟预览不代表完整真机运动验收、自主抓取或现场避障已经完成；Windows 文件复验、中文输入等记录不完整的项目，也在相应文章中注明。

这里只发布整理后的文章及官方来源链接，不附原始聊天、私人网络配置、账户凭据或厂商软件包。缺失的原始截图用文字说明，没有补造操作截图。
