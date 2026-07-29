# Xtrainer Toolkit

Dobot Xtrainer 双臂机械臂控制工具。在不依赖 ROS 的情况下，直接通过 TCP/Dashboard 协议（端口 29999）控制机械臂的上电、解锁、清除错误、示教拖拽等操作。
> [!TIP]
> 同样适用于Nova2等支持越疆TCP_IP远程控制接口v3的机械臂

## 功能

- **CLI 命令模式** — 单条命令控制双臂
- **TUI 交互模式** — 终端图形界面，实时显示双臂状态（RobotMode + ErrorID）
- **一键启停** — 锁定→清除错误→上电(轮询等待)→解锁，每步间隔 1 秒
- **上电轮询** — 发送 PowerOn 后每秒查询 RobotMode，直到离开"未上电"状态（最长 60 秒）
- **连接状态轮询** — TUI 中每 2 秒自动查询双臂 RobotMode 和 GetErrorID
- **操作互斥** — busy 锁防止并发操作
- **静态编译** — 基于 musl 静态链接，1.3MB，零外部依赖

## 安装

从 [Releases](https://github.com/kt286/xtrainer_toolkit/releases) 下载预编译二进制，或自行编译：

```bash
# 需要 Rust 工具链 + musl target
rustup target add x86_64-unknown-linux-musl
cargo build --release --target x86_64-unknown-linux-musl
# 产物: target/x86_64-unknown-linux-musl/release/xtrainer_toolkit
```

## 配置

首次运行自动生成 `~/.config/xtk/config.json`：

```json
{
  "arm_left": { "name": "Arm1 (Left)", "ip": "192.168.5.1", "port": 29999 },
  "arm_right": { "name": "Arm2 (Right)", "ip": "192.168.5.2", "port": 29999 }
}
```

或通过 `-c` 指定配置：

```bash
xtk -c /path/to/config.json <command>
```

## CLI 使用

```bash
# 单条命令
xtk power-on --arm left       # 左臂上电（含轮询等待）
xtk enable --arm both         # 双臂解锁
xtk clear-error               # 双臂清除错误
xtk start-drag --arm right    # 右臂示教
xtk stop-drag                 # 双臂停止示教
xtk disable --arm both        # 双臂锁定

# 一键启停
xtk quick-start               # 锁定→清错→上电→解锁
xtk quick-stop                # 停教→锁定

# TUI
xtk                           # 进入 TUI 交互模式
```

### 可用命令

| 命令 | 说明 |
| ------ | ------ |
| `power-on` | 机械臂上电（等待完成） |
| `enable` | 机械臂解锁/使能 |
| `disable` | 机械臂去使能/锁定 |
| `clear-error` | 清除错误状态 |
| `start-drag` | 开启示教模式 |
| `stop-drag` | 停止示教模式 |
| `quick-start` | 一键启动 |
| `quick-stop` | 一键停止 |

所有命令支持 `-a/--arm left|right|both`。

## TUI 快捷键

```
1 上电  2 解锁  3 清错  4 示教  5 停教  6 锁定
! @     #       $       %       ^           ← 按住 Shift 操作右臂

s 一键启动    x 一键停止    q 退出    Ctrl+C 强制退出
```

TUI 面板实时显示：

- 连接状态（✓/✗）
- RobotMode（1-11，带中文标签）
- 错误信息（GetErrorID 解析）
- 最后查询时间

## 协议

基于 [TCP/IP 远程控制接口文档 V3](https://www.dobot.cn/) Dashboard 协议：

- 端口：**29999**
- 命令：ASCII 文本（如 `PowerOn()`）
- 响应：以 `;` 结尾
- 状态查询：`RobotMode()`（1-11），`GetErrorID()`

### RobotMode 值

| Code | 含义 | 颜色 |
| ------ | ------ | ------ |
| 1 | 初始化 | 🟡 |
| 2 | 抱闸松开 | 🟡 |
| 3 | 未上电 | 🔴 |
| 4 | 未使能 | 🔴 |
| 5 | 使能就绪 | 🟢 |
| 6 | 拖拽中 | 🟣 |
| 7 | 运行中 | 🩵 |
| 8 | 录制中 | 🔵 |
| 9 | ⚠ 报警 | 🔴 |
| 10 | 暂停 | 🟡 |
| 11 | 点动中 | 🟡 |

## License

MIT
