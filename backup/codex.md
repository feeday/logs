### 常用链接

- 网页使用：[https://chatgpt.com/codex/cloud/](https://chatgpt.com/codex/cloud/)
- 用量查询：[https://chatgpt.com/codex/cloud/settings/analytics#usage](https://chatgpt.com/codex/cloud/settings/analytics#usage)
- 工具下载：[https://chatgpt.com/zh-Hans-CN/download](https://chatgpt.com/zh-Hans-CN/download/)
- 插件安装：[VS Code 插件安装页](https://marketplace.visualstudio.com/items?itemName=openai.chatgpt)
- 使用说明：[https://developers.openai.com/codex/ide](https://developers.openai.com/codex/ide)


安装插件后，在 VS Code 侧边栏打开 Codex，使用 ChatGPT 账号登录即可。

#### 登录端口异常或冲突

如果登录时出现以下报错：

```text
Sign-in failed: failed to start login server: 以一种访问权限不允许的方式做了一个访问套接字的尝试。 (os error 10013)
```

可能是 Windows 端口分配异常或端口被排除。请使用**管理员身份**打开 PowerShell，依次执行：

```powershell
net stop winnat
netsh interface ipv4 show excludedportrange protocol=tcp
net start winnat
netsh interface ipv4 show excludedportrange protocol=tcp
```

然后重新登录。重启 WinNAT 服务可能会暂时影响 WSL、Docker 或虚拟化网络连接。

### Linux 服务器

- 官方安装说明：[Codex CLI](https://developers.openai.com/codex/cli)

安装后，在项目目录中运行 `codex`，首次使用时按提示登录 ChatGPT 账号或配置 API Key。

## 会员价格与用量

| 套餐 | 价格 | 适合场景 |
| --- | --- | --- |
| Free | `$0/月` | 体验和简单任务，用量较少 |
| Go | `$8/月` | 轻量编程任务 |
| Plus | `$20/月` | 每周进行几次集中开发 |
| Pro | `$100/月` 起 | 高频开发，可选择相对 Plus 更高的用量档位 |

Codex 用量不是固定的“每天多少次”。官方按滚动 5 小时窗口计算，部分套餐还可能有每周限制。以 `GPT-5.3-Codex` 为例，Plus 通常可用：

| 类型 | Plus 参考用量 |
| --- | --- |
| 本地消息 | 每 5 小时约 `30-150` 条 |
| 云端任务 | 每 5 小时约 `10-60` 个 |
| GitHub 代码审查 | 每 5 小时约 `20-50` 次 |

简单修改消耗较少；大型项目、长时间任务、较多上下文、图片生成和 Fast 模式会更快消耗额度。达到套餐额度后，部分用户可以购买 Credits 继续使用。使用 API Key 时，费用独立计算，按 Token 用量扣费。

价格和限额会调整，请以官方页面为准：

- [Codex 价格与用量说明](https://developers.openai.com/codex/pricing)
- [ChatGPT 套餐价格](https://chatgpt.com/pricing)
- [Codex Credits 计费说明](https://help.openai.com/en/articles/20001106-codex-rate-card)
