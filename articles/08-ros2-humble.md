# ROS 2 Humble 装好了，然后呢？用验证确认环境可用

作者：Eating-King-Git。根据本人操作记录与官方文档整理，含 AI 辅助写作和核查。整理日期：2026-11-21。

适用环境：Ubuntu 22.04、ROS 2 Humble、Bash。我的历史环境是 Ubuntu 22.04.5、Python 3.10.12，安装了 ROS Desktop。本文把当时已经完成的检查和读者可以使用的最小验证分开说明。

## 1. 安装命令没有报错，为什么还要验证？

因为“软件包存在”和“实际能用”之间还有几步：终端是否加载环境、程序能否启动、两个节点能否通信、自己的代码能否构建、图形界面是否正常。

我当时的记录包含 C++ 与 Python 节点收发、最小工作区构建、RViz 启动，以及重启后的复查。判断环境是否可用，要看这些实际输出；AI 助手给出“安装完成”的文字，不能替代终端中的验证结果。

因此，我会按下面的顺序验收：

```text
系统与软件版本 → 当前终端环境 → 本机消息收发
→ 自己的工作区 → RViz → 新终端与重启后复查
```

这些检查通过后，能说电脑端开发基础可用了；还不能据此说机械臂、相机或另一台电脑已经连通。

## 2. 第一步看哪些信息？

先打开普通 Ubuntu 终端，执行：

```bash
cat /etc/os-release
uname -m
python3 --version
dpkg-query -W ros-humble-desktop ros-humble-rclpy ros-humble-rviz2
```

前两项确认系统版本和处理器架构，最后一项确认软件包及其版本。没有安装 Desktop 而只装 ROS Base 的电脑，查不到 Desktop 或 RViz 不代表整个 ROS 安装失败；应先确认自己选择的安装类型。

Humble 的官方 Ubuntu 二进制安装面向 Jammy 22.04；Desktop 包含 RViz 和示例，ROS Base 更精简。不要因为搜索结果里出现了另一个 ROS 发行版，就把不同版本的软件源和包名混在一起。[ROS 2 官方安装文档源码](https://github.com/ros2/ros2_documentation/blob/humble/source/Installation/Ubuntu-Install-Debs.rst)

我当时记录的 rclpy 是 3.3.22、RViz2 是 11.2.29。这些数字用于定位那次环境，读者不必为了“和文章一样”降级已经正常工作的包。

## 3. 新终端里提示 `ros2: command not found`，怎么办？

先检查是否加载了环境：

```bash
source /opt/ros/humble/setup.bash
command -v ros2
printenv ROS_DISTRO
```

最后一行应输出 `humble`。`source` 的作用是把软件位置等配置加载到**当前终端**；在一个窗口执行，另一个已经打开的窗口不会自动继承。

新手可以先每次手动加载，确认没有混用多个 ROS 版本，再考虑是否写入 `.bashrc`。自己的工作区也有环境文件：先加载系统 ROS，再加载已经构建成功的工作区。官方把这种关系称为底层环境与覆盖环境；理解为“先有公共工具，再加入自己的包”即可。[ROS 2 官方环境配置文档源码](https://github.com/ros2/ros2_documentation/blob/humble/source/Tutorials/Beginner-CLI-Tools/Configuring-ROS2-Environment.rst)

不要把文章中的项目专用 `verify-xxx.sh` 当成 ROS 自带命令。我原来用过自定义检查脚本，本文用公开的基础命令说明同一检查思路。

## 4. 怎样确认两个节点真的在通信？

在两个专门用于练习的终端中分别运行以下命令。括号让设置只在子 shell 中生效，退出示例后不会长期改变当前终端配置。

终端 A：

```bash
(
  source /opt/ros/humble/setup.bash
  export ROS_LOCALHOST_ONLY=1
  export ROS_DOMAIN_ID=41
  ros2 run demo_nodes_cpp talker
)
```

终端 B：

```bash
(
  source /opt/ros/humble/setup.bash
  export ROS_LOCALHOST_ONLY=1
  export ROS_DOMAIN_ID=41
  ros2 run demo_nodes_py listener
)
```

这里的 41 只是本次练习选择的相同域编号；若本机已有程序使用这个域，另选一个未使用的有效编号，并同时修改两端。`ROS_LOCALHOST_ONLY=1` 把发现范围限制在本机，适合先排除跨机网络因素。

预期结果是 A 持续发布，B 持续打印收到的 `Hello World`，计数随时间变化。确认后分别按 `Ctrl+C` 结束。

这组示例同时检查了 C++、Python 和消息通信；不是仅看到一个程序启动就算完成。官方安装文档也使用这组不同语言的示例进行验证。[ROS 2 官方示例说明](https://github.com/ros2/ros2_documentation/blob/humble/source/Installation/Ubuntu-Install-Debs.rst)

我历史验证中的本机限制只用于测试进程，没有把所有未来终端永久锁在本机。后续要跨电脑通信，应重新检查两端域编号、网卡与发现条件。

## 5. 发送端正常，接收端没消息，先查什么？

先对照两个窗口：是否都加载 Humble，域编号是否相同，是否一端只允许本机而另一端在远端，进程是否仍在运行。再在使用相同环境的第三个终端查看：

```bash
source /opt/ros/humble/setup.bash
export ROS_LOCALHOST_ONLY=1
export ROS_DOMAIN_ID=41
ros2 node list
ros2 topic list
ros2 topic info /chatter
ros2 topic echo /chatter --once
```

`topic list` 看到名字，只能说明图中发现了这个话题；`echo --once` 真正收到一条消息，才补上了数据传输证据。检查完成后关闭这个练习终端。

如果图中没有节点，先回到进程与环境；有话题但没有数据，再看发布是否持续、消息类型和通信配置。不要一看到空列表就重装 ROS，更不要顺手修改机械臂网络。

如果启动时直接提示找不到 `demo_nodes_cpp` 或 `demo_nodes_py`，收发测试其实还没开始。先用 `ros2 pkg list` 检查示例包是否存在，再对照自己安装的是 Desktop 还是精简环境，按官方安装说明补齐所需示例。缺少一个示例包与消息通信失败，是两种不同的问题，记录错误时要分开。

## 6. 示例通过了，还需要构建自己的工作区吗？

需要。预编译示例正常，不代表本机编译器、依赖声明和工作区组织已经没有问题。

工作区可以理解为一个项目目录，通常有 `src` 放源码；构建后产生 `build`、`install` 和 `log`。我当时构建了最小测试包，并在重启后复查通过。读者可以按官方 colcon 教程创建练习工作区，不需要先引入庞大的机械臂工程。[ROS 2 官方 colcon 教程源码](https://github.com/ros2/ros2_documentation/blob/humble/source/Tutorials/Beginner-Client-Libraries/Colcon-Tutorial.rst)

下面两条只适用于**已经按教程放好源码**的练习工作区根目录：

```bash
colcon list
colcon build --symlink-install
```

第一条应列出预期的包，第二条应有成功的构建汇总。随后另开一个新终端，加载系统环境和该工作区的 `install/setup.bash`，查询或运行刚构建的练习程序。

构建失败时保留第一条真正的错误和对应日志。最后的“某包失败”只是汇总；常见原因可能是缺依赖、声明没写全或选错源码版本，下一篇就有实际例子。

## 7. RViz 能打开，是否说明机器人模型正确？

先加载 Humble，再执行：

```bash
rviz2
```

能打开窗口，只证明图形界面启动这一层。空白网格很正常，因为还没有配置模型、坐标系和传感器数据。后续显示模型时，需要进一步确认 Fixed Frame、变换关系和数据来源。

我的历史记录确认 RViz 可用，图形栈能进行硬件加速；那时只有笔记本内置摄像头被识别，目标深度相机的型号与连接方式仍未确定。所以“RViz 可用”不能写成“视觉抓取环境已完成”。

## 8. 为什么还要重启一次？

因为某些成功来自临时终端变量或尚未完成的系统更新。我的记录中，后续重启完成、待重启标记消失，ROS、Python、C++ 和 RViz 复查仍通过，才把这套电脑环境记为可继续开发。

读者至少应另开新终端，重复环境和消息收发检查；若刚完成需要重启的系统更新，再在方便时重启复查。记录系统、关键包版本、源码提交和检查结果，能帮助下次判断是代码变化还是环境变化。

最后保留一张简短验收表即可：

| 检查 | 通过的表现 | 还没有证明什么 |
|---|---|---|
| 环境 | `ROS_DISTRO` 为 Humble | 所有项目依赖齐全 |
| 收发 | listener 连续收到消息 | 跨电脑、真机通信可用 |
| 构建 | 练习包成功并能被找到 | 厂商 SDK 全部接口兼容 |
| RViz | 窗口正常启动 | 模型准确、现场避障有效 |
| 复查 | 新终端或重启后仍通过 | 今后所有更新均无影响 |

## Linux 常用命令小板块

| 命令 | 这篇里的用途 |
|---|---|
| `source 文件` | 把环境设置加载到当前 shell |
| `command -v ros2` | 查看实际找到的命令位置 |
| `printenv ROS_DISTRO` | 检查当前 ROS 发行版 |
| `dpkg-query -W 包名` | 查已安装包及版本 |
| `ros2 topic info 话题名` | 查看话题类型和端点数量 |
| `Ctrl+C` | 结束当前前台示例程序 |

---

[系列目录](../README.md) · [上一篇：第 07 篇](07-kimi-connection-troubleshooting.md) · [下一篇：第 09 篇](09-sdk-python-cpp.md)
