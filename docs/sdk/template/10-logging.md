---
title: "日志记录"
---


> 如何查看模板构建的日志

### 配置参数

在使用 SDK 之前，请确保配置以下环境变量：

- 获取 API Key: [https://console.ucloud.cn/modelverse/experience/api-keys](https://console.ucloud.cn/modelverse/experience/api-keys)

```bash
export AGENTBOX_API_KEY=your_api_key
```

您可以使用 SDK 检索构建日志。

## 默认日志记录器

我们提供了一个默认日志记录器，您可以使用它按级别过滤日志：

```python
from ucloud_sandbox import Template, default_build_logger

Template.build(
    template,
    alias="my-template",
    on_build_logs=default_build_logger(
        min_level="info",  # 要显示的最低日志级别
    )
)
```

## 自定义日志记录器

您可以自定义日志的处理方式：

```python
import sys
from ucloud_sandbox import Template

# 简单日志记录
on_build_logs=lambda log_entry: print(log_entry)

# 自定义格式化
def custom_logger(log_entry):
    time = log_entry.timestamp.isoformat()
    print(f"[{time}] {log_entry.level.upper()}: {log_entry.message}")

Template.build(template, alias="my-template", on_build_logs=custom_logger)

# 按日志级别过滤
def error_logger(log_entry):
    if log_entry.level in ["error", "warn"]:
        print(f"ERROR/WARNING: {log_entry}", file=sys.stderr)

Template.build(template, alias="my-template", on_build_logs=error_logger)
```

`on_build_logs` 回调接收具有以下属性的结构化 `LogEntry` 对象：

```python
from typing import Literal
from dataclasses import dataclass
from datetime import datetime

LogEntryLevel = Literal["debug", "info", "warn", "error"]

@dataclass
class LogEntry:
    timestamp: datetime
    level: LogEntryLevel
    message: str

    def __str__(self) -> str:  # 返回格式化的日志字符串
        pass


# 指示构建过程的开始
@dataclass
class LogEntryStart(LogEntry):
    level: LogEntryLevel = "debug"

# 指示构建过程的结束
@dataclass
class LogEntryEnd(LogEntry):
    level: LogEntryLevel = "debug"
```

除了 `LogEntry` 类型外，还有 `LogEntryStart` 和 `LogEntryEnd` 类型，分别指示构建过程的开始和结束。它们的默认日志级别是 `debug`，您可以像这样使用它们：

```python
from ucloud_sandbox import LogEntryStart, LogEntryEnd

if isinstance(log_entry, LogEntryStart):
    # 构建开始
    return

if isinstance(log_entry, LogEntryEnd):
    # 构建结束
    return
```
