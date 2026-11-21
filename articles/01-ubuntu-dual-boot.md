# 第一次安装 Ubuntu 双系统：从准备 U 盘到进入桌面

作者：Eating-King-Git。根据本人操作记录与官方文档整理，含 AI 辅助写作和核查。整理日期：2026-11-21。

我为后续 ROS 2 和机械臂开发，在一台 HP 笔记本上保留 Windows 11，另外安装了 Ubuntu 22.04.5。本文按实际过程整理，重点写三个容易卡住的地方：**容量显示不同、EFI 分区警告，以及安装后出现 BitLocker 恢复页面。**

这是 Ubuntu 22.04.5 安装经历，其他版本的界面可能不同；文中的磁盘编号是这台电脑的结果，操作时必须核对自己的设备。

## 1. 我用的电脑和安装方案是什么？

| 项目 | 本次配置 |
|---|---|
| 电脑 | HP Pavilion Aero 13-be1xxx |
| 原系统 | Windows 11 |
| 内置硬盘 | 标称 512 GB SSD，GPT 分区表 |
| 新系统 | Ubuntu 22.04.5 LTS Desktop，amd64 |
| 启动盘 | 64 GB U 盘，使用 Rufus 制作 |
| Ubuntu 空间 | 提前留出的连续 150 GiB 未分配空间 |
| 分区方案 | 新建 ext4 根分区 `/`，保留 Windows、EFI 和恢复分区 |

本次流程是：

**备份与留空间 → 制作启动 U 盘 → 从 U 盘启动 → 试用检查硬件 → 核对分区 → 安装 → 拔 U 盘重启 → 检查 Ubuntu → 回 Windows 验证。**

我选 22.04.5 是为了配合当时准备使用的开发环境。新项目应先核对目标软件支持的系统版本，不必照搬这个版本号。[Ubuntu 22.04.5 官方镜像](https://releases.ubuntu.com/22.04/)

## 2. 安装前要准备什么？

先备份重要文件，把 BitLocker 恢复密钥保存在**不依赖这台电脑启动**的位置，并确认自己能找到对应磁盘的密钥。恢复密钥是 48 位数字，用于解锁加密磁盘；它和页面显示的“密钥 ID”不是同一个东西。[Microsoft 恢复密钥说明](https://support.microsoft.com/en-us/windows/security/encryption/find-your-bitlocker-recovery-key)

我的安装前记录是：

- 已从原分区腾出连续的 150 GiB 未分配空间。
- 恢复密钥已另存到手机。
- Windows 快速启动已关闭。
- 启动 U 盘已制作完成。

**这里的“未分配空间”不是 D 盘里还剩多少可用容量。** 前者在分区表中独立显示为空闲区域，后者仍属于现有文件系统。本文不复述前期调整旧分区的过程，也不建议在 Ubuntu 安装器里凭盘符或容量猜测后删除分区。

如果安装器明确提示 BitLocker 阻止并存安装，应先退出，按官方流程处理加密状态并确认解密完成；**有恢复密钥不等于磁盘已解密，暂停保护也不等于关闭加密**。[Ubuntu 的 BitLocker 处理说明](https://ubuntu.com/desktop/docs/en/24.04/how-to/turn-off-bitlocker-in-windows/)

## 3. 启动 U 盘怎么做？

我使用 Rufus，将 Ubuntu 的 ISO 镜像写入 64 GB U 盘，当时采用 **GPT、UEFI、ISO 镜像模式**。这组设置对应我的 UEFI 电脑，并非所有旧电脑的通用设置。

操作顺序：

1. 从 Ubuntu 官方页面下载 Desktop amd64 镜像。
2. 打开 [Rufus 官网下载的工具](https://rufus.ie/en/)。
3. 在“设备”中确认选中的是准备使用的 U 盘，再选择 ISO。
4. 核对分区方案与目标启动模式，开始写入；出现 ISO/DD 选择时，本次选 ISO 模式。
5. 等待写入结束，再重启进入启动菜单。

制作过程会覆盖 U 盘现有数据。**把 ISO 文件普通复制进 U 盘，并不等于制作好了启动盘。**[Ubuntu 启动盘说明](https://ubuntu.com/desktop/docs/en/latest/how-to/create-a-bootable-usb-stick/)、[Rufus 操作示例](https://ubuntu.com/tutorials/create-a-usb-stick-on-windows)

## 4. 开机后选哪一个启动项？

在这台 HP 上，我进入启动菜单后按 **F9**，选择带 **UEFI** 的 USB 启动项，而不是 Windows Boot Manager。

接下来依次选择：

```text
UEFI USB 启动项
→ Try or Install Ubuntu
→ 中文（简体）
→ 试用 Ubuntu
```

先试用，是为了确认显示、Wi-Fi、触控板、亮度和声音能用。我实际检查这些项目后才进入安装。不同品牌的启动菜单按键可能不同，要以电脑说明和当前屏幕为准。

## 5. 为什么 512 GB 显示成 476.94 GiB，150 GiB 又变成约 161 GB？

这是计量单位不同：

- GB 按十进制计算：1 GB = 1,000,000,000 字节。
- GiB 按二进制计算：1 GiB = 1,073,741,824 字节。

因此，标称约 512 GB 的硬盘显示约 476.9 GiB；150 GiB 约等于 161.06 GB。我的安装器显示 **161062 MB 空闲空间**，与此前预留容量基本对应，不是突然多出一块硬盘。

我在 GParted 中确认内置 SSD 为 `/dev/nvme0n1`，再核对容量和分区布局；没有把约 64 GB 的 U 盘选为安装目标。

## 6. EFI 分区出现红色感叹号，是不是坏了？

我当时看到 EFI 分区旁边有红色警告，先打开“信息”查看提示，没有直接格式化或修复。

当时针对已经确认的 FAT32 EFI 分区，执行了只读检查：

```bash
sudo fsck.fat -n /dev/nvme0n1p1
```

**这是本机当时使用的设备名，不应直接照抄到另一台电脑。** 检查前要确认目标分区和挂载状态；`-n` 表示只检查、不向文件系统写入。不要擅自去掉它或换成自动修复参数。[Ubuntu 22.04 的 fsck.fat 手册](https://manpages.ubuntu.com/manpages/jammy/man8/fsck.fat.8.html)

当时记录的结果是：卷标信息一处为 `SYSTEM`、另一处为空，没有报告其他文件系统错误，末尾显示 `Leaving filesystem unchanged`。我保留原分区继续安装。

这次警告不能推广为“EFI 红色警告都可以忽略”。如果出现读取失败、文件链错误或不同输出，需要按实际问题处理。

## 7. 怎样把 Ubuntu 安装到预留空间？

在 Ubuntu 22.04.5 的“安装类型”页面，我选 **“其他选项”**，进入手动分区。

我的具体设置如下：

| 对象 | 本次处理 |
|---|---|
| 约 161062 MB 空闲空间 | 使用这块空间新建 ext4 分区，挂载点 `/` |
| 新建根分区 | 实际编号为 `/dev/nvme0n1p7`，勾选格式化 |
| 现有 EFI 分区 `p1` | 保留并复用，不勾选格式化 |
| Windows 与恢复分区 | 保留，不格式化、不删除 |
| 安装器中的引导器设备 | 确认为内置 SSD `/dev/nvme0n1` |

这次没有单独划分 `/home` 和 swap 分区。根分区用于系统和个人文件；是否启用交换文件，可在安装后用 `swapon --show` 核对，不能只凭分区方案认定已经启用。

点击“现在安装”后，还要看最终的写盘确认窗口。我当时确认它**只列出新建的第 7 分区将格式化为 ext4**，才继续。若窗口列出 Windows、EFI 或其他非目标分区要被格式化，就先返回重新核对。

接着选择时区、设置账户并等待安装完成。这次安装阶段取消了下载更新，系统更新放在进入 Ubuntu 后处理；这属于当时的安排，不是所有电脑必须如此。

## 8. 安装完成后什么时候拔 U 盘？

安装器提示完成后，先点击“现在重启”。等出现移除安装介质的提示，再拔掉 U 盘，按 Enter。

重启后进入 Ubuntu 桌面，只能说明首次启动成功。还应确认系统确实来自内置硬盘，而不是又回到了试用环境。

按 **Ctrl＋Alt＋T** 打开终端，逐条执行：

```bash
lsb_release -ds
```

```bash
uname -r
```

```bash
findmnt -no SOURCE,FSTYPE /
```

我的记录分别是：Ubuntu 22.04.5 LTS、内核 `6.8.0-149-generic`，以及根分区 `/dev/nvme0n1p7`、文件系统 ext4。**内核版本是当时记录，不是别人安装后必须一致的验收条件。**

## 9. 返回 Windows 时出现 BitLocker 恢复页，怎么办？

我安装 Ubuntu 后切回 Windows，确实遇到了 BitLocker 恢复页面；当时页面提示与安全启动策略变化有关。它不等于已经证明 Windows 文件丢失。

处理时先看页面上的**恢复密钥 ID**，在保存的记录中找到对应磁盘的密钥，再在电脑本机输入 48 位恢复密钥。若保存了多块磁盘的密钥，不要只凭“第一组、第二组”选择。[Microsoft 查找与匹配密钥说明](https://support.microsoft.com/en-us/windows/security/encryption/find-your-bitlocker-recovery-key)

引导或固件等变化可能触发恢复验证，因此本文不把“关闭 Secure Boot”列为安装必做步骤。[Microsoft BitLocker 恢复机制](https://learn.microsoft.com/en-us/windows/security/operating-system-security/data-protection/bitlocker/recovery-overview)

真正完成双系统验收，还应：

1. 拔掉安装 U 盘，分别启动 Ubuntu 和 Windows。
2. 登录 Windows，打开原有盘符，抽查重要文件。
3. 再回 Ubuntu，确认仍能从 SSD 正常启动。

本次保留的记录能确认 Ubuntu 已从 SSD 启动，并记录了 BitLocker 恢复页面；**没有保留 Windows 文件逐项复验完成的明确回执，因此不把“全部原文件完好”写成已证实结果。**

## 10. 刚开始用 Linux，先记哪些命令？

| 命令或快捷键 | 用途 |
|---|---|
| `pwd` | 显示当前所在目录 |
| `ls -lah` | 查看当前目录的文件，包括隐藏文件 |
| `cd ~` | 回到自己的主目录 |
| `mkdir -p ~/linux-practice` | 创建一个练习文件夹 |
| `lsblk -f` | 查看磁盘、分区和文件系统信息，不做修改 |
| `df -h /` | 查看根文件系统的容量与剩余空间 |
| `swapon --show` | 查看当前启用的交换空间 |
| `man ls` | 打开命令帮助；按 `q` 退出 |
| Tab | 补全命令或文件名，减少长名字输错 |
| Ctrl＋Shift＋V | 在终端中粘贴 |

复制命令时，只复制代码框里的内容，不要把 `用户名@电脑名:~$` 提示符一起复制。如果粘贴后出现 `<br/>` 或 `[网址](网址)`，先清掉再粘贴纯文本——这是我后来配置环境时实际遇到的错误。

Ubuntu 安装到这里，下一步才是网络更新和开发环境配置。我后续曾遇到 `security.ubuntu.com` 无法解析，说明进入桌面与软件更新成功是两项不同的检查；这部分留到网络排错篇展开。

---

[系列目录](../README.md) · [下一篇：第 02 篇](02-usb-reuse.md)
