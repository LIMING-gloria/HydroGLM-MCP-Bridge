# WorkBuddy 接入

本项目采用 **本地 stdio MCP**。WorkBuddy 官方连接器规范支持这一传输方式，核心配置字段为 `type`、`command`、`args` 和可选的 `env`。[官方规范](https://open.workbuddy.cn/docs/connector)

以下步骤按官方配置格式整理，未进行 WorkBuddy 桌面端实测。它是自定义 MCP 接入，不是已经上架 WorkBuddy 连接器市场的安装包。

## 步骤

1. 安装 Python 3.10+，下载本项目，在目录中运行 `python3 configure_key.py --agent HydroGLM`。
2. 在同一设备上生成配置：

   ```bash
   python3 generate_config.py --client workbuddy --output local.workbuddy.mcp.json
   ```

   Windows 使用 `py -3 generate_config.py --client workbuddy --output local.workbuddy.mcp.json`。

3. 打开 WorkBuddy 中的自定义 MCP 服务器配置入口，将生成文件中的 `hydroglm_bridge` 合并进已有 `mcpServers`。若采用表单方式填写，选择 stdio，分别填入生成文件里的 `command` 和 `args`。不要填瀚海 API URL。
4. 保存、启用并重新加载服务器，在工具列表确认 `ask_hydroglm`。
5. 直接问“水库汛限水位有什么作用？”，由宿主按相关专业任务自动选用工具，并通过真实调用记录确认。日常无需点名工具；首次验证或未调用时见 [连接验证与排障](使用说明.md#连接验证与排障)。

界面名称可能随版本变化，配置入口参考 [WorkBuddy MCP 使用指南](https://www.codebuddy.ai/docs/zh/workbuddy/From-Beginner-to-Expert-Guide/Function-Description/MCP-Guide)。以实际版本提供的 MCP 编辑器为准。

## 配置示意

```json
{
  "mcpServers": {
    "hydroglm_bridge": {
      "type": "stdio",
      "command": "/absolute/path/to/python3",
      "args": ["-X", "utf8", "/absolute/path/to/HydroGLM-MCP-Bridge/server.py"]
    }
  }
}
```

上面的绝对路径仅作示意，优先使用生成器取得真实路径。凭据保存在该 Python 进程所属用户的配置目录。若 WorkBuddy 把 MCP 放到隔离环境运行，应在那个环境中准备 Python、脚本及凭据。

## 等待时间

API 等待上限为 600 秒。WorkBuddy 规范中的 `timeout` 是**连接超时，单位毫秒**，默认 30000；不能将 `timeout=660` 理解为等待工具 660 秒。样例没有设置该字段。当前规范不足以证明这个连接字段也控制单次生成等待，因此不在这里承诺完整的 600 秒调用窗口。长请求是否提前终止，需要在所用 WorkBuddy 版本中验证。

如果准备上架 WorkBuddy 连接器市场，还需要其要求的元信息、分发和审核流程；GitHub 源码发布与市场上架是两个步骤。本仓库先交付可配置的本地 MCP 服务。
