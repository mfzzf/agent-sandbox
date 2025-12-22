# SDK 快速开始
<subtitle>几分钟内完成安装并创建您的第一个沙箱。</subtitle>

本指南将带您完成 UCloud Sandbox SDK 的快速入门，包括 CLI 安装、创建第一个沙箱、命令执行、文件操作以及构建自定义模板。

---

## 1. 环境配置

在使用 SDK 之前，请确保已配置 `AGENTBOX_API_KEY` 环境变量。

?> 您可以在 [控制台 API 密钥页面](https://console.ucloud.cn/modelverse/experience/api-keys) 获取您的密钥。

```bash
export AGENTBOX_API_KEY=your_api_key
```

---

## 2. 安装 CLI

确保您的系统中已安装 Node.js 环境。

```bash
npm i -g @ucloud-sdks/ucloud-sandbox-cli
```

安装完成后，可以通过以下命令验证是否成功：

```bash
ucloud-sandbox-cli --help
```

> [!TIP]
> 更多 CLI 功能请参阅 [CLI 完整指南](/agent-sandbox/docs/cli/cli.md)。

---

## 3. 创建第一个沙箱

使用 Python SDK 快速创建沙箱：

```python
from ucloud_sandbox import Sandbox

# 创建沙箱并设置存活时间为 60 秒
sandbox = Sandbox.create(timeout=60)

# 获取沙箱详细信息
info = sandbox.get_info()
print(info)

# 使用完毕后立即销毁
sandbox.kill()
```

!> 注意：超时的沙箱将由系统自动回收并清理。建议在业务流程结束时手动调用 `kill()` 方法释放资源。

> [!NOTE]
> 更多沙箱生命周期管理请参阅 [沙箱生命周期](/agent-sandbox/docs/sdk/sandbox/01-lifecycle.md)。

---

## 4. 执行命令

`commands.run()` 是与沙箱交互最直接的方式。您可以像操作本地终端一样执行任意合法命令：

```python
from ucloud_sandbox import Sandbox

sandbox = Sandbox.create()

# 执行命令
result = sandbox.commands.run('ls -la /home/user')

# 解析结果
if result.exit_code == 0:
    print(f"Success:\n{result.stdout}")
else:
    print(f"Error (Exit {result.exit_code}):\n{result.stderr}")

sandbox.kill()
```

> [!TIP]
> 对于长时间运行的命令，请参考 [后台运行命令](/agent-sandbox/docs/sdk/commands/03-run-commands-in-background.md)。

---

## 5. 文件操作

每个沙箱都拥有独立的文件系统，您可以轻松进行读写操作：

```python
from ucloud_sandbox import Sandbox

sandbox = Sandbox.create()

# 写入文件
sandbox.files.write("hello.txt", "UCloud Sandbox is awesome!")

# 读取文件
content = sandbox.files.read("hello.txt")
print(content)  # 输出: UCloud Sandbox is awesome!

# 列出目录
files = sandbox.files.list("/home/user")
for f in files:
    print(f.name, f.type)

sandbox.kill()
```

?> **默认根目录**：绝大部分操作默认在 `/home/user` 下进行。

> [!NOTE]
> 更多文件操作请参阅 [文件系统概述](/agent-sandbox/docs/sdk/filesystem/01-overview.md)。

---

## 6. 构建自定义模板

模板（Template）是沙箱的蓝图，允许您预装软件、配置环境变量、预置文件。

### 方式一：使用 CLI（推荐）

```bash
# 初始化模板项目
ucloud-sandbox-cli template init

# 构建模板
ucloud-sandbox-cli template build --name my-agent-env
```

### 方式二：使用 Python SDK

**编写模板定义：**

```python
from ucloud_sandbox import Template, wait_for_timeout

template = (
    Template()
    .from_base_image()  # 使用官方预置的基础镜像
    .set_envs({
        "APP_VERSION": "1.0.0",
        "DEBUG": "true"
    })
    .set_start_cmd("echo 'Environment is ready'", wait_for_timeout(5_000))
)
```

**构建并发布：**

```python
from ucloud_sandbox import Template, default_build_logger

Template.build(
    template,
    alias="my-agent-env",
    cpu_count=2,
    memory_mb=1024,
    on_build_logs=default_build_logger(),
)
```

### 使用自定义模板

```python
from ucloud_sandbox import Sandbox

# 使用模板别名创建沙箱
sbx = Sandbox.create(template="my-agent-env")

# 检查环境变量
result = sbx.commands.run("echo $APP_VERSION")
print(f"Version: {result.stdout}")  # 输出: Version: 1.0.0
```

> [!TIP]
> 模板别名是您全局唯一的标识符。更多模板功能请参阅 [模板完整指南](/agent-sandbox/docs/sdk/template/01-quick-start.md)。

---

## 7. 完整示例

以下是一个完整的工作流程示例：

```python
from ucloud_sandbox import Sandbox

# 创建沙箱
sandbox = Sandbox.create(timeout=300)
print(f"Sandbox created: {sandbox.sandbox_id}")

# 执行命令
result = sandbox.commands.run("python --version")
print(f"Python version: {result.stdout}")

# 写入并执行 Python 脚本
sandbox.files.write("script.py", """
import os
print("Hello from UCloud Sandbox!")
print(f"Working directory: {os.getcwd()}")
""")

result = sandbox.commands.run("python script.py")
print(result.stdout)

# 清理资源
sandbox.kill()
print("Sandbox destroyed")
```

---

## 下一步

- [沙箱生命周期管理](/agent-sandbox/docs/sdk/sandbox/01-lifecycle.md) - 了解超时设置与运行监控
- [命令执行详解](/agent-sandbox/docs/sdk/commands/01-overview.md) - 深入了解命令执行功能
- [文件系统操作](/agent-sandbox/docs/sdk/filesystem/01-overview.md) - 完整的文件操作指南
- [模板工作原理](/agent-sandbox/docs/sdk/template/02-how-it-works.md) - 深入理解模板机制
- [E2B 兼容模式](/agent-sandbox/docs/sdk/e2b-compatibility.md) - 使用 E2B SDK 接入
