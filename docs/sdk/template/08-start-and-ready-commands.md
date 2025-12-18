---
title: "启动和就绪命令"
---


> 定义沙盒运行的进程

### 配置参数

在使用 SDK 之前，请确保配置以下环境变量：

- 获取 API Key: [https://console.ucloud.cn/modelverse/experience/api-keys](https://console.ucloud.cn/modelverse/experience/api-keys)

```bash
export AGENTBOX_API_KEY=your_api_key
```

## 启动命令

启动命令允许您指定在生成自定义沙盒时**已经在运行**的命令。
这样，例如，您可以在沙盒内运行服务器或预置数据库，当您使用 SDK 生成沙盒时，它们已经完全准备就绪，用户在运行时零等待时间。

启动命令功能的目的是为了减少用户的等待时间，并在您生成沙盒时为用户准备好一切。

您可以在[此处](02-how-it-works.md)查看其工作原理。

## 就绪命令

就绪命令允许您指定一个命令，该命令将在创建[快照](02-how-it-works.md)之前确定**模板沙盒**的就绪状态。
它在一个无限循环中执行，直到返回成功的**退出代码 0**。
通过这种方式，您可以控制我们应该等待[启动命令](#启动命令)或任何系统状态多长时间。

## 用法

设置沙盒启动时运行的命令以及确定沙盒何时就绪的命令：

```python
from ucloud_sandbox import Template, wait_for_port, wait_for_timeout

template = Template()

# 同时设置启动命令和就绪命令
template.set_start_cmd("npm start", wait_for_port(3000))

# 仅设置就绪命令
template.set_ready_cmd(wait_for_timeout(10_000))
```

就绪命令用于确定沙盒何时准备好接受连接。

> 您每个模板只能调用这些命令一次。后续调用将抛出错误。

## 就绪命令辅助函数

SDK 为常见的就绪命令模式提供了辅助函数：

```python
from ucloud_sandbox import wait_for_port, wait_for_process, wait_for_file, wait_for_timeout

# 等待端口可用
wait_for_port(3000)

# 等待进程运行
wait_for_process("node")

# 等待文件存在
wait_for_file("/tmp/ready")

# 等待指定时间
wait_for_timeout(10_000)  # 10 秒
```
