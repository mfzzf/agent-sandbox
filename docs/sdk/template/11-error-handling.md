# 错误处理

> 处理模板中的错误

### 配置参数

在使用 SDK 之前，请确保配置以下环境变量：

- 获取 API Key: [https://console.ucloud.cn/modelverse/experience/api-keys](https://console.ucloud.cn/modelverse/experience/api-keys)

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
