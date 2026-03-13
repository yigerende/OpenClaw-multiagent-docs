# OpenClaw 配置多飞书机器人完整指南

> 创建时间：2026-03-06  
> 适用版本：OpenClaw 2026.3.1+

---

## 🎯 目标

配置 N 个飞书机器人对应 N 个独立 Agent，实现多助手并行工作。

---

## ✅ 步骤 1：在飞书创建机器人

**每个机器人需要：**

1. 打开 https://open.feishu.cn/ 创建企业自建应用
2. 进入 **应用功能 → 机器人** → 启用机器人
3. 设置机器人名称和头像
4. **关键配置（事件与回调）：**
   - 订阅方式：选择 **"使用长连接接收事件/回调"**
   - 订阅事件：添加 **im:message**（接收消息事件）
   - 保存配置

5. 获取凭证（应用管理 → 凭证与基础信息）：
   - AppID（cli_xxx）
   - AppSecret

---

## ✅ 步骤 2：创建 Agent

```bash
openclaw agents add <agent-id> --workspace "C:\Users\Administrator\.openclaw\workspace-<agent-id>"
```

**示例：**
```bash
openclaw agents add it-tech --workspace "C:\Users\Administrator\.openclaw\workspace-it-tech"
```

> 💡 模型默认使用 `qwen3.5-plus`，无需额外指定

**验证：**
```bash
openclaw agents list
```

应看到：
```
- main (default)
- it-tech
```

---

## ✅ 步骤 3：修改 openclaw.json 配置

编辑 `C:\Users\Administrator\.openclaw\openclaw.json`

**修改 `channels.feishu` 部分：**

tools部分的sessions和agentToAgent要加上去

accounts加上去，accounts下面的main和agent-2这是你上面创建的agent的名称要对应

```json
"channels": {
  "feishu": {
    "enabled": true,
    "domain": "feishu",
    "connectionMode": "websocket",
    "dmPolicy": "open",
    "allowFrom": ["*"],
    "groupPolicy": "open",
    "requireMention": false,
    "userAccessToken": "你的 userAccessToken",
    "userRefreshToken": "你的 userRefreshToken",
     "tools": {
        "sessions": {
          "visibility": "all"
        },
        "agentToAgent": {
          "enabled": true,
          "allow": [
            "main",
            "it-tech",
            "company",
            "hr-admin"
          ]
        }
      },
    "accounts": {
      "main": {
        "appId": "cli_xxx1",
        "appSecret": "xxx1",
        "botName": "机器人 1 名称"
      },
      "agent-2": {
        "appId": "cli_xxx2",
        "appSecret": "xxx2",
        "botName": "机器人 2 名称"
      }
      // 更多机器人...
    }
  }
},
"bindings": [
  {
    "agentId": "main",
    "match": {
      "channel": "feishu",
      "accountId": "main"
    }
  },
  {
    "agentId": "agent-2",
    "match": {
      "channel": "feishu",
      "accountId": "agent-2"
    }
  }
  // 更多绑定...
]
```

**关键配置说明：**

| 配置项 | 位置 | 说明 |
|--------|------|------|
| `connectionMode: "websocket"` | 外层 | 长连接模式 |
| `dmPolicy: "open"` | 外层 | 私聊无需配对 |
| `groupPolicy: "open"` | 外层 | 群聊无需@ |
| `requireMention: false` | 外层 | 群聊无需@机器人 |
| `userAccessToken` | 外层 | 飞书用户 Token（保留原值） |
| `userRefreshToken` | 外层 | 飞书刷新 Token（保留原值） |
| `accounts.xxx` | 内层 | 每个机器人的配置 |
| `bindings` | 顶层 | 路由规则（agentId 对应 accountId） |

**⚠️ 注意事项：**
- `accounts` 下的 key（如 `main`、`it-tech`）必须与 `bindings` 中的 `accountId` 对应
- `bindings` 中的 `agentId` 必须是已创建的 Agent
- 配置完成后**不要运行** `openclaw doctor --fix`，可能会破坏配置

---

## ✅ 步骤 4：重启 Gateway

```bash
openclaw gateway restart
```

**等待启动完成，查看日志确认：**

```
[feishu] starting feishu[main] (mode: websocket)
[feishu] starting feishu[it-tech] (mode: websocket)
[feishu] feishu[main]: WebSocket client started
[feishu] feishu[it-tech]: WebSocket client started
```

---

## ✅ 步骤 5：验证配置

```bash
openclaw agents list
openclaw agents bindings
```

应看到：
```
- main (default)
- it-tech

Routing bindings:
- main <- feishu accountId=main
- it-tech <- feishu accountId=it-tech
```

---

## ✅ 步骤 6：飞书测试

1. 在飞书找到对应的机器人
2. 发送消息测试
3. 查看日志确认收到消息：
   ```
   feishu[xxx]: received message from ...
   feishu[xxx]: dispatching to agent ...
   ```

**可选：直接测试 Agent（不通过飞书）**
```bash
openclaw agent --agent it-tech --message "你好，请回复一个测试消息"
```

---

## ⚠️ 常见坑和解决方案

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| **机器人收不到消息** | 飞书长连接配置未生效 | 关闭长连接 → 保存 → 重新开启 → 保存 |
| **只有一个机器人能回复** | bindings 配置错误 | 检查 agentId 和 accountId 是否对应 |
| **Gateway 启动失败** | 端口被占用 | `openclaw gateway stop` 后再 start |
| **doctor 破坏配置** | 自动修复会改结构 | 配置完成后不要运行 doctor --fix |
| **飞书显示"未检测到连接"** | 长连接未正确配置 | 在飞书后台重新开关长连接 |

---

## 💡 最佳实践

1. **先测试 Agent** - 用 `openclaw agent --agent xxx --message "测试"` 验证 Agent 正常
2. **再配置飞书** - 确认 Agent 没问题后再配置飞书长连接
3. **一次配置一个** - 不要同时配置多个，容易混淆问题
4. **保留备份** - 修改 openclaw.json 前先备份

---

## 📝 快速参考命令

```bash
# 创建 Agent
openclaw agents add <id> --workspace "<path>"

# 查看 Agent 列表
openclaw agents list

# 查看路由绑定
openclaw agents bindings

# 测试 Agent（不通过飞书）
openclaw agent --agent <id> --message "测试"

# 重启 Gateway
openclaw gateway restart

# 查看日志
Get-Content "C:\Users\ADMINI~1\AppData\Local\Temp\2\openclaw\openclaw-2026-03-06.log" -Tail 50
```

---

**祝配置顺利！🦞**