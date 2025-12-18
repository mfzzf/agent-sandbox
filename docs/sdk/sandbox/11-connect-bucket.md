---
title: "连接存储桶"
---

# 连接存储桶
<subtitle>将外部对象存储挂载为沙箱内的本地盘，实现大文件的高效存取。</subtitle>

!> **提示**：目前挂载外部存储桶的功能即将上线。以下文档供提前参考。

## 配置环境

在使用 SDK 之前，请确保已配置 `AGENTBOX_API_KEY` 环境变量。

?> 您可以在 [控制台 API 密钥页面](https://console.ucloud.cn/modelverse/experience/api-keys) 获取您的秘钥。

```bash
export AGENTBOX_API_KEY=your_api_key
```

您可以利用 FUSE 技术将流行的云端对象存储（如 UCloud US3, AWS S3, GCS 等）直接挂载到沙箱的目录树中。

---

## Google Cloud Storage (GCS)

### 1. 准备模板
您需要创建一个安装了 `gcsfuse` 的自定义沙箱模板。

```python
from ucloud_sandbox import Template

template = (
    Template()
    .from_image("base-image:latest")
    .apt_install(["gnupg", "lsb-release"])
    .run_cmd('echo "deb [signed-by=/usr/share/keyrings/cloud.google.asc] https://packages.cloud.google.com/apt gcsfuse-$(lsb_release -c -s) main" | sudo tee /etc/apt/sources.list.d/gcsfuse.list')
    .run_cmd("curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo tee /usr/share/keyrings/cloud.google.asc")
    .apt_install(["gcsfuse"])
)
```

### 2. 挂载操作
在运行时初始化挂载点并调用 `gcsfuse`。

```python
from ucloud_sandbox import Sandbox

sandbox = Sandbox.create(template_id="your-custom-template-id")
sandbox.files.make_dir("/home/user/bucket")

# 写入凭证文件
sandbox.files.write("key.json", "your-gcp-key-json-content")

# 执行挂载命令
sandbox.commands.run(
    "sudo gcsfuse --key-file /home/user/key.json <bucket-name> /home/user/bucket"
)
```

---

## Amazon S3 / UCloud US3

对于兼容 S3 协议的存储，推荐使用 `s3fs`。

### 1. 挂载示例

```python
from ucloud_sandbox import Sandbox

sandbox = Sandbox.create(template_id="template-with-s3fs")
sandbox.files.make_dir("/home/user/s3-data")

# 写入 S3 凭证 (AccessKey:SecretKey)
sandbox.files.write("/root/.passwd-s3fs", "AK_ID:SK_KEY")
sandbox.commands.run("sudo chmod 600 /root/.passwd-s3fs")

# 挂载 S3 桶
# 如果是 UCloud US3 或 R2，需通过 -o url 指定 API 端点
sandbox.commands.run("sudo s3fs <bucket-name> /home/user/s3-data -o allow_other")
```

---

## 权限建议

?> 如果您希望普通用户（非 root）能够读写挂载目录，请在挂载命令中添加 `-o allow_other` 标志，并根据需要设置 `-file-mode=777 -dir-mode=777`。