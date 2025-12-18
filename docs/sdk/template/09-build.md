---
title: "构建"
---


> 如何构建模板

### 配置参数

在使用 SDK 之前，请确保配置以下环境变量：

- 获取 API Key: [https://console.ucloud.cn/modelverse/experience/api-keys](https://console.ucloud.cn/modelverse/experience/api-keys)

```bash
export AGENTBOX_API_KEY=your_api_key
```

## 构建并等待完成

`build` 方法构建模板并等待构建完成。它返回构建信息，包括模板 ID 和构建 ID。

```python
from ucloud_sandbox import Template, default_build_logger

build_info = Template.build(
    template,
    alias="my-template",  # 模板别名（必填）
    cpu_count=2,  # CPU 核心数
    memory_mb=2048,  # 内存（MB）
    skip_cache=False,  # 配置跳过缓存（文件除外）
    on_build_logs=default_build_logger(),  # 日志回调接收 LogEntry 对象
    api_key="your-api-key",  # 覆盖 API 密钥
    domain="your-domain",  # 覆盖域名
)

# build_info 包含: BuildInfo(alias, template_id, build_id)
```

## 后台构建

`build_in_background` 方法启动构建过程并立即返回，而无需等待完成。当您想要触发构建并稍后检查其状态时，这很有用。

```python
from ucloud_sandbox import Template

build_info = Template.build_in_background(
    template,
    alias="my-template",
    cpu_count=2,
    memory_mb=2048,
)

# 立即返回: BuildInfo(alias, template_id, build_id)
```

## 检查构建状态

使用 `get_build_status` 检查使用 `build_in_background` 启动的构建的状态。

```python
from ucloud_sandbox import Template

status = Template.get_build_status(
    build_info,
    logs_offset=0,  # 可选：获取日志的偏移量
)

# status 包含构建状态和日志
```

## 示例：带有状态轮询的后台构建

```python
import time
from ucloud_sandbox import Template

# 在后台启动构建
build_info = Template.build_in_background(
    template,
    alias="my-template",
    cpu_count=2,
    memory_mb=2048,
)

# 轮询构建状态
logs_offset = 0
status = "building"

while status == "building":
    build_status = Template.get_build_status(
        build_info,
        logs_offset=logs_offset,
    )

    logs_offset += len(build_status.log_entries)
    status = build_status.status.value

    for log_entry in build_status.log_entries:
        print(log_entry)

    # 等待一小段时间后再次检查状态
    time.sleep(2)

if status == "ready":
    print("Build completed successfully")
else:
    print("Build failed")
```
