# 连接存储桶
<subtitle>使用 s3fs 将 S3 兼容的对象存储桶挂载到沙箱本地目录，像操作本地文件一样读写远端对象。</subtitle>

## 配置环境

在使用 SDK 之前，请确保已配置 `AGENTBOX_API_KEY` 环境变量。

?> 您可以在 [控制台 API 密钥页面](https://console.ucloud.cn/modelverse/experience/api-keys) 获取您的密钥。

```bash
export AGENTBOX_API_KEY=your_api_key
```

---

## 功能说明

`s3fs` 是一个基于 FUSE 的工具，支持将兼容 S3 协议的 Bucket（如 UCloud US3、AWS S3、Cloudflare R2 等）挂载到本地目录，像使用本地文件系统一样直接操作对象存储中的对象。

沙箱默认运行 Debian 系统，可通过 `apt-get` 直接安装 s3fs。

---

## 通过 CLI 使用

### 1. 创建沙箱并安装 s3fs

```bash
# 创建沙箱并进入终端
ucloud-sandbox-cli sbx cr base

# 在沙箱终端中安装 s3fs
user@ucloud:~$ sudo apt-get update && sudo apt-get install -y s3fs

# 验证安装
user@ucloud:~$ s3fs --version
Amazon Simple Storage Service File System V1.90 (commit:unknown) with GnuTLS(gcrypt)
```

### 2. 配置密钥

创建密钥文件，格式为 `AccessKeyID:SecretAccessKey`。

?> 公私钥获取方式请参考对应云厂商的 S3 AccessKeyID 和 SecretAccessKey 说明。

```bash
# 写入凭证
user@ucloud:~$ sudo vim /root/.passwd-s3fs
# 文件内容格式：access-token:secret-access-token

# 设置权限
user@ucloud:~$ sudo chmod 600 /root/.passwd-s3fs
```

### 3. 挂载存储桶

```bash
# 创建挂载目录
user@ucloud:~$ mkdir /home/user/s3-data

# 挂载 S3 兼容存储桶, 沙箱可以使用乌兰察布内网地址http://internal.s3-cn-wlcb.ufileos.com或者其他 S3 公网地址。

user@ucloud:~$ sudo s3fs <bucket-name> /home/user/s3-data \
  -o url=<S3EndpointURL> \
  -o passwd_file=/root/.passwd-s3fs \
  -o dbglevel=info \
  -o curldbg,use_path_request_style,allow_other,nomixupload \
  -o retries=1 \
  -o multipart_size=8 \
  -o multireq_max=8 \
  -o parallel_count=32
```

### 4. 验证挂载

```bash
user@ucloud:~$ ls /home/user/s3-data/
gpt-oss-20b  qwen3-32b  ufile-log
```

### 5. 文件操作

挂载成功后，可以像操作本地文件一样读写存储桶中的对象。

```bash
# 上传文件（拷贝到挂载目录）
user@ucloud:~$ cp local-file.txt /home/user/s3-data/

# 下载文件（从挂载目录拷贝出来）
user@ucloud:~$ cp /home/user/s3-data/remote-file.txt ./

# 删除文件
user@ucloud:~$ rm /home/user/s3-data/remote-file.txt
```

### 6. 卸载存储桶

```bash
user@ucloud:~$ sudo umount /home/user/s3-data
```

---

## 通过 Python SDK 使用

以下是一个完整的示例，展示如何通过 SDK 在沙箱中安装 s3fs、挂载存储桶并进行文件操作。

```python
import os
import time
from ucloud_sandbox import Sandbox

# S3 配置（从环境变量读取）
S3_ACCESS_KEY = os.environ.get("S3_ACCESS_KEY", "")
S3_SECRET_KEY = os.environ.get("S3_SECRET_KEY", "")
S3_BUCKET = os.environ.get("S3_BUCKET", "bucket_name")
S3_ENDPOINT = os.environ.get("S3_ENDPOINT", "http://internal.s3-cn-wlcb.ufileos.com")
MOUNT_DIR = "/home/user/s3-data"

# 创建沙箱
sbx = Sandbox.create(timeout=120)

# 1. 安装 s3fs
result = sbx.commands.run(
    "sudo apt-get update && sudo apt-get install -y s3fs",
    timeout=120,
)
result = sbx.commands.run("s3fs --version")
print(f"s3fs 版本: {result.stdout.strip().splitlines()[0]}")

# 2. 配置 S3 凭证
sbx.files.write("/root/.passwd-s3fs", f"{S3_ACCESS_KEY}:{S3_SECRET_KEY}")
sbx.commands.run("sudo chmod 600 /root/.passwd-s3fs")

# 3. 创建挂载目录
sbx.files.make_dir(MOUNT_DIR)

# 4. 挂载存储桶
sbx.commands.run(
    f"sudo s3fs {S3_BUCKET} {MOUNT_DIR} "
    f"-o url={S3_ENDPOINT} "
    f"-o passwd_file=/root/.passwd-s3fs "
    f"-o dbglevel=info "
    f"-o curldbg,use_path_request_style,allow_other,nomixupload "
    f"-o retries=1 "
    f"-o multipart_size=8 "
    f"-o multireq_max=8 "
    f"-o parallel_count=32"
)

# 验证挂载
result = sbx.commands.run(f"df -h | grep s3fs")
print(f"挂载信息: {result.stdout.strip()}")

# 5. 文件操作
# 写入文件到存储桶
test_file = f"{MOUNT_DIR}/sdk_test_{int(time.time())}.txt"
sbx.files.write(test_file, "Hello from AgentBox SDK!")

# 读取存储桶中的文件
content = sbx.files.read(test_file)
print(f"读取内容: {content}")

# 列出挂载目录
result = sbx.commands.run(f"ls {MOUNT_DIR} | head -10")
print(f"目录内容:\n{result.stdout}")

# 删除测试文件
sbx.commands.run(f"rm {test_file}")

# 6. 卸载存储桶
sbx.commands.run(f"sudo umount {MOUNT_DIR}")

# 清理沙箱
sbx.kill()
```

---

## 参数说明

| 参数 | 说明 |
|------|------|
| `<bucket-name>` | 存储桶名称，不带域名后缀。例如 US3 空间名显示为 `test.cn-bj.ufileos.com`，则填 `test` |
| `-o url` | S3 API 端点地址。UCloud US3、Cloudflare R2 等非 AWS 服务必须指定 |
| `-o passwd_file` | 密钥文件路径 |
| `-o multipart_size=8` | 分片上传大小（MB），目前仅支持 8MB |
| `-o multireq_max=8` | 超过 8MB 的文件采用分片上传 |
| `-o parallel_count=32` | 并行操作数，可提高分片并发，建议不超过 128 |
| `-o allow_other` | 允许非 root 用户访问挂载目录 |
| `-o retries=1` | 错误重试次数 |

---

## 权限建议

?> 如果您希望普通用户（非 root）能够读写挂载目录，请在挂载命令中添加 `-o allow_other` 标志，并根据需要设置 `-o uid=<用户ID> -o gid=<组ID>`。

---

## 注意事项

!> 路径不符合 Linux 文件路径规范的对象，可以在控制台看到，但不会在挂载目录下显示。枚举文件清单会比较缓慢，建议直接使用指定具体文件的命令（如 `cp`、`rm`）。

---

## 性能参考

| 操作 | 吞吐量 |
|------|--------|
| 写入 | ~40 MB/s |
| 读取 | 最高 ~166 MB/s（与并发量相关） |
