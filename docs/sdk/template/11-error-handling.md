# 错误处理

<subtitle>捕获并处理模板构建过程中的各类异常。</subtitle>

## 配置环境

在使用 SDK 之前，请确保已配置 `AGENTBOX_API_KEY` 环境变量。

?> 您可以在 [控制台 API 密钥页面](https://console.ucloud.cn/modelverse/experience/api-keys) 获取您的密钥。

```bash
export AGENTBOX_API_KEY=your_api_key
```

SDK 提供了特定的错误类型：

```python
from ucloud_sandbox import Template, AuthError, BuildError, FileUploadError

try:
    Template.build(template, alias="my-template")
except AuthError as error:
    print(f"Authentication failed: {error}")
except FileUploadError as error:
    print(f"File upload failed: {error}")
except BuildError as error:
    print(f"Build failed: {error}")
```
