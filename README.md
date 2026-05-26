# NPU Tools — 多服务器 NPU 状态监控工具

通过 SSH 并发查询多台服务器的 NPU 状态 · 飞书机器人实时推送 · 本地终端快捷查询 · 状态图片自动生成

---

🔍 核心能力

- SSH 并发查询多台服务器 NPU 状态，支持自动重试
- 智能解析 `npu-smi info` 输出，精确区分空闲 / 占用 / 异常
- 终端彩色表格输出，中英文混排自动对齐
- 自动生成状态汇总图片（PNG），飞书内直观展示
- 飞书 WebSocket 长连接，消息触发即时响应

🔌 使用方式

| 场景 | 方案 |
|------|------|
| 本地终端查询 | `--local` 模式，输出彩色表格到 stdout |
| 飞书机器人 | WebSocket 模式，发送消息即可获取 NPU 状态图片 |

🖥️ 运行模式

| 模式 | 命令 | 说明 |
|------|------|------|
| 本地模式 | `python3 npu_status_query.py --local` | 查询后输出到终端并退出，无需飞书配置 |
| 飞书模式 | `python3 npu_status_query.py` | 启动 WebSocket 长连接，通过飞书消息触发查询 |

---

## 快速开始

### 安装依赖

```bash
pip install -r requirements.txt
```

### 配置

编辑 `config.yaml`：

```yaml
servers:
  - host: "192.168.25.212"
    port: 22
    username: "root"
    password: "YOUR_SERVER_PASSWORD"
  - host: "192.168.25.213"
    port: 22
    username: "root"
    password: "YOUR_SERVER_PASSWORD"

feishu:
  app_id: "YOUR_FEISHU_APP_ID"
  app_secret: "YOUR_FEISHU_APP_SECRET"
```

### 本地模式

```bash
python3 npu_status_query.py --local
```

输出示例：

```
==================================================================================
  服务器            空闲卡                      占用卡                      状态
----------------------------------------------------------------------------------
  192.168.25.212    7卡 (1,2,3,4,5,6,7)         1卡 (0)                     有空闲
  192.168.25.218    2卡 (4,5)                   6卡 (0,1,2,3,6,7)           有空闲
----------------------------------------------------------------------------------
  总计              9 卡空闲                    7 卡占用
==================================================================================
```

退出码说明：

| 退出码 | 含义 |
|--------|------|
| 0 | 全部成功且有空闲卡 |
| 1 | 部分服务器查询失败 |
| 2 | 无空闲卡 |
| 3 | 全部服务器查询失败 |

### 飞书机器人模式

```bash
python3 npu_status_query.py
```

启动后连接飞书 WebSocket，用户发送 `npu`、`/npu`、`npu status` 或 `查看npu` 即可触发查询，机器人自动返回 NPU 状态图片。

#### 飞书应用配置

1. 前往 [飞书开放平台](https://open.feishu.cn/app) 创建应用
2. 开启 **机器人** 能力
3. 在 **凭证与基础信息** 中获取 `app_id` 和 `app_secret`
4. 配置消息权限：`im:message`（发送消息）
5. 使用 WebSocket 模式部署

> 未配置飞书信息时，仅可使用 `--local` 模式。

---

## 依赖说明

| 包名 | 用途 |
|------|------|
| PyYAML | 解析 config.yaml 配置文件 |
| requests | HTTP 请求（获取 Token、上传图片） |
| Pillow | 生成 NPU 状态汇总图片 |
| lark-oapi | 飞书 SDK（消息收发、WebSocket 长连接） |
| paramiko | SSH 连接服务器执行命令 |

---

## 项目结构

```
npu-tools/
├── npu_status_query.py   # 主程序
├── config.yaml           # 服务器与飞书配置
└── requirements.txt      # Python 依赖
```
