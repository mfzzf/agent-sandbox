# 命令执行
<subtitle>在沙箱控制台中远程执行 Shell 命令并获取执行结果。</subtitle>

## 配置环境

在使用 SDK 之前，请确保已配置 `AGENTBOX_API_KEY` 环境变量。

?> 您可以在 [控制台 API 密钥页面](https://console.ucloud.cn/modelverse/experience/api-keys) 获取您的秘钥。

```bash
export AGENTBOX_API_KEY=your_api_key
```

`commands.run()` 是与沙箱交互最直接的方式。您可以像操作本地终端一样，在沙箱中执行任意合法的命令。

## 基础用法

调用 `run` 方法将阻塞当前进程直到命令执行结束，并返回包含标准输出（stdout）、标准错误（stderr）及退出代码的结果对象。

```python
from ucloud_sandbox import Sandbox

# 1. 创建沙箱
sandbox = Sandbox.create()

# 2. 执行简单命令
result = sandbox.commands.run('ls -la /home/user')

# 3. 解析结果
if result.exit_code == 0:
    print(f"Success:\n{result.stdout}")
else:
    print(f"Error (Exit {result.exit_code}):\n{result.stderr}")
```

## 注意事项

?> **默认工作目录**：命令默认在 `/home/user` 路径下执行。如果需要更改目录，可以在命令中使用 `cd` 或者通过 `working_dir` 参数指定。

!> **超时控制**：长时间运行的命令可能会受到 API 连接超时的限制。对于长任务，建议参考 [后台运行命令](/agent-sandbox/docs/sdk/commands/03-run-commands-in-background.md)。
