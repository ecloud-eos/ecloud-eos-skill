# 移动云 EOS 操作参考

本文档记录操作方式的详细参数定义，供执行操作时查阅。

**环境设置**：首次使用请运行 `scripts/setup.sh`，详见 `SKILL.md` 首次使用章节。

## 方式一：scripts/eos_node.mjs 命令参考

脚本位于 `scripts/eos_node.mjs`，依赖 `@aws-sdk/client-s3` 和 `@aws-sdk/s3-request-presigner`（`npm install @aws-sdk/client-s3 @aws-sdk/s3-request-presigner`）。
所有凭证通过环境变量读取。输出 JSON 格式。

### 可用操作

| 操作 | 命令 | 说明 |
|------|------|------|
| list-buckets | `node scripts/eos_node.mjs list-buckets` | 列出所有存储桶 |
| create-bucket | `node scripts/eos_node.mjs create-bucket --bucket <bucket>` | 创建桶 |
| delete-bucket | `node scripts/eos_node.mjs delete-bucket --bucket <bucket>` | 删除桶 |
| empty-bucket | `node scripts/eos_node.mjs empty-bucket --bucket <bucket>` | 清空桶（删除所有文件） |
| set-bucket | `node scripts/eos_node.mjs set-bucket --bucket <bucket>` | 切换存储桶（更新环境变量） |
| get-bucket-acl | `node scripts/eos_node.mjs get-bucket-acl` | 查询桶访问权限 |
| set-bucket-acl | `node scripts/eos_node.mjs set-bucket-acl --acl <acl>` | 设置桶访问权限 |
| set-referer | `node scripts/eos_node.mjs set-referer --referers <list> [--type <type>]` | 设置防盗链 |
| get-referer | `node scripts/eos_node.mjs get-referer` | 查询防盗链 |
| delete-referer | `node scripts/eos_node.mjs delete-referer` | 删除防盗链 |
| set-cors | `node scripts/eos_node.mjs set-cors --allowed-origins <list> --allowed-methods <methods> [--allowed-headers <headers>] [--expose-headers <headers>] [--max-age <seconds>]` | 设置跨域规则（CORS） |
| get-cors | `node scripts/eos_node.mjs get-cors` | 查询跨域规则（CORS） |
| delete-cors | `node scripts/eos_node.mjs delete-cors` | 删除跨域规则（CORS） |
| upload-object | `node scripts/eos_node.mjs upload-object --file <path> --key <key>` | 上传本地文件 |
| download-object | `node scripts/eos_node.mjs download-object --key <key> --output <path>` | 下载文件到本地 |
| copy-object | `node scripts/eos_node.mjs copy-object --source-key <src> --dest-key <dest> [--dest-bucket <bucket>]` | 复制文件（同一桶内或跨桶） |
| list-objects | `node scripts/eos_node.mjs list-objects [--prefix <prefix>] [--max-keys <n>] [--marker <marker>]` | 列出文件（支持分页） |
| delete-object | `node scripts/eos_node.mjs delete-object --key <key>` | 删除文件 |
| head-object | `node scripts/eos_node.mjs head-object --key <key>` | 查看文件元信息 |
| exists-object | `node scripts/eos_node.mjs exists-object --key <key>` | 判断文件是否存在 |
| get-object-acl | `node scripts/eos_node.mjs get-object-acl --key <key>` | 查询文件访问权限 |
| set-object-acl | `node scripts/eos_node.mjs set-object-acl --key <key> --acl <acl>` | 设置文件访问权限 |
| generate-url | `node scripts/eos_node.mjs generate-url --key <key> [--expires <n>]` | 生成文件共享外链 |

### 返回格式

成功时 `success: true`，退出码 0；失败时 `success: false`，退出码 1。

### 操作说明

#### list-buckets - 列出所有存储桶

**参数：**
无参数

**返回示例：**
```json
{
  "success": true,
  "action": "list-buckets",
  "count": 3,
  "buckets": [
    {
      "name": "my-bucket",
      "creationDate": "2025-03-13T10:00:00.000Z"
    },
    {
      "name": "test-bucket",
      "creationDate": "2025-03-14T15:30:00.000Z"
    },
    {
      "name": "backup-bucket",
      "creationDate": "2025-03-15T08:45:00.000Z"
    }
  ],
  "message": "找到 3 个存储桶"
}
```

**说明：**
- 列出当前账户下的所有存储桶
- 返回每个桶的名称和创建时间
- 按创建时间排序

**使用场景：**
```bash
# 查看所有可用的存储桶
node scripts/eos_node.mjs list-buckets

# 根据返回的桶名选择要操作的桶
node scripts/eos_node.mjs set-bucket --bucket my-bucket
```

#### create-bucket - 创建桶

**参数：**
- `--bucket <bucket>`（必需）：要创建的桶名称

**返回示例：**
```json
{
  "success": true,
  "action": "create-bucket",
  "bucket": "my-new-bucket",
  "message": "桶创建成功"
}
```

**说明：**
- 桶名需要手动指定，不使用环境变量中的 `EOS_BUCKET`
- 桶名必须符合移动云命名规则（通常格式：name-appid）
- 桶名创建后不能修改
- 桶名在全局范围内必须唯一

#### delete-bucket - 删除桶

**参数：**
- `--bucket <bucket>`（必需）：要删除的桶名称

**返回示例：**

桶为空，删除成功：
```json
{
  "success": true,
  "action": "delete-bucket",
  "bucket": "my-old-bucket",
  "message": "桶删除成功"
}
```

桶不为空，删除失败：
```json
{
  "success": false,
  "action": "delete-bucket",
  "error": "桶 my-old-bucket 不为空，无法删除。请先删除桶内的所有文件后再尝试删除桶。"
}
```

**说明：**
- 桶名需要手动指定，不使用环境变量中的 `EOS_BUCKET`
- **桶必须为空才能删除**
- 如果桶不为空，系统会返回错误提示，不会执行删除操作
- 删除桶前需要先手动删除桶内的所有文件
- 删除操作不可逆，请谨慎操作

#### empty-bucket - 清空桶（删除所有文件）

**参数：**
- `--bucket <bucket>`（必需）：要清空的桶名称

**返回示例：**

成功清空桶：
```json
{
  "success": true,
  "action": "empty-bucket",
  "bucket": "my-bucket",
  "deletedCount": 150,
  "failedCount": 0,
  "message": "清空桶完成，成功删除 150 个文件"
}
```

部分文件删除失败：
```json
{
  "success": true,
  "action": "empty-bucket",
  "bucket": "my-bucket",
  "deletedCount": 148,
  "failedCount": 2,
  "message": "清空桶完成，成功删除 148 个文件，失败 2 个",
  "errors": [
    {
      "key": "images/photo1.jpg",
      "error": "Access Denied"
    },
    {
      "key": "documents/report.pdf",
      "error": "Lock conflict"
    }
  ]
}
```

桶已为空：
```json
{
  "success": true,
  "action": "empty-bucket",
  "bucket": "my-bucket",
  "deletedCount": 0,
  "message": "桶已为空，无需删除文件"
}
```

**说明：**
- 桶名需要手动指定，不使用环境变量中的 `EOS_BUCKET`
- 使用分页方式获取桶内所有文件，确保不会遗漏
- 逐个删除文件，记录成功和失败的数量
- 如果删除失败，会在返回结果中包含错误信息
- 清空操作不可逆，请谨慎操作

**使用场景：**
```bash
# 场景1：准备删除桶前清空
node scripts/eos_node.mjs empty-bucket --bucket my-old-bucket
node scripts/eos_node.mjs delete-bucket --bucket my-old-bucket

# 场景2：批量清理测试数据
node scripts/eos_node.mjs empty-bucket --bucket test-bucket
```

#### set-bucket - 切换存储桶（更新环境变量）

**参数：**
- `--bucket <bucket>`（必需）：要切换到的桶名称

**返回示例：**

切换成功：
```json
{
  "success": true,
  "action": "set-bucket",
  "bucket": "my-new-bucket",
  "message": "已切换到指定存储桶",
  "note": "请重新打开终端使环境变量生效"
}
```

切换失败：
```json
{
  "success": false,
  "action": "set-bucket",
  "error": "切换存储桶失败: ..."
}
```

**说明：**
- 此命令会直接更新环境变量中的 `EOS_BUCKET` 值
- 不修改配置文件
- 会验证新桶是否可访问
- 需要重新打开终端使环境变量生效

**使用场景：**
```bash
# 切换到另一个存储桶
node scripts/eos_node.mjs set-bucket --bucket target-bucket

# 切换成功后，所有操作将使用新的存储桶
node scripts/eos_node.mjs list-objects
node scripts/eos_node.mjs upload-object "file.jpg" "images/file.jpg"
```

#### get-bucket-acl - 查询桶访问权限

**参数：**
无参数，使用环境变量中的 `EOS_BUCKET`

**返回示例：**
```json
{
  "success": true,
  "action": "get-bucket-acl",
  "bucket": "mybucket",
  "data": {
    "Owner": {
      "DisplayName": "owner",
      "ID": "owner-id"
    },
    "Grants": [
      {
        "Grantee": {
          "Type": "CanonicalUser",
          "DisplayName": "owner",
          "ID": "owner-id"
        },
        "Permission": "FULL_CONTROL"
      }
    ]
  }
}
```

#### set-bucket-acl - 设置桶访问权限

**参数：**
- `--acl <acl>`（必需）：权限类型，支持以下值：
  - `private` - 私有：仅所有者有完全控制权限
  - `public-read` - 公共读：所有者有完全控制权限，所有人可读
  - `public-read-write` - 公共读写：所有者有完全控制权限，所有人可读可写
- 注意：使用环境变量中的 `EOS_BUCKET`

**返回示例：**
```json
{
  "success": true,
  "action": "set-bucket-acl",
  "bucket": "mybucket",
  "acl": "private",
  "message": "桶权限设置成功"
}
```

#### set-referer - 设置防盗链

**参数：**
- `--referers <list>`（必需）：防盗链域名列表，用逗号分隔
  - 使用 `*` 表示允许所有域名（相当于关闭防盗链）
  - 使用 `https://example.com` 表示只允许该域名访问
  - 可以指定多个域名，用逗号分隔
- `--type <type>`（可选）：防盗链类型，默认为 `allow`
  - `allow` - 白名单模式：只允许指定域名访问
  - `deny` - 黑名单模式：拒绝指定域名访问

**返回示例：**

白名单模式：
```json
{
  "success": true,
  "action": "set-referer",
  "bucket": "my-bucket",
  "type": "allow",
  "referers": [
    "https://example.com",
    "https://www.example.com"
  ],
  "message": "防盗链设置成功"
}
```

黑名单模式：
```json
{
  "success": true,
  "action": "set-referer",
  "bucket": "my-bucket",
  "type": "deny",
  "referers": [
    "https://evil.com"
  ],
  "message": "防盗链设置成功"
}
```

**说明：**
- 防盗链功能通过检查 HTTP Referer 头来实现
- 白名单模式：只允许指定域名的网站访问桶中的资源
- 黑名单模式：拒绝指定域名的网站访问桶中的资源
- 使用 `*` 作为 Referer 时，相当于关闭防盗链（允许所有域名访问）
- 防盗链设置只影响通过 HTTP Referer 的访问，不影响其他访问方式（如直接访问、预签名 URL 等）

**使用场景：**
```bash
# 场景1：只允许自己的网站访问资源
node scripts/eos_node.mjs set-referer --referers "https://mywebsite.com,https://www.mywebsite.com" --type allow

# 场景2：禁止特定盗链网站
node scripts/eos_node.mjs set-referer --referers "https://leech-site.com" --type deny

# 场景3：关闭防盗链（允许所有域名）
node scripts/eos_node.mjs set-referer --referers "*"
```

#### get-referer - 查询防盗链

**参数：**
无参数（使用配置文件中的桶名）

**返回示例：**

已设置防盗链：
```json
{
  "success": true,
  "action": "get-referer",
  "bucket": "my-bucket",
  "type": "allow",
  "referers": [
    "https://example.com",
    "https://www.example.com"
  ],
  "message": "防盗链查询成功"
}
```

未设置防盗链：
```json
{
  "success": true,
  "action": "get-referer",
  "bucket": "my-bucket",
  "message": "未设置防盗链"
}
```

**说明：**
- 查询当前桶的防盗链设置
- 返回防盗链类型和域名列表
- 如果未设置防盗链，会返回相应提示

**使用示例：**
```bash
# 查询防盗链设置
node scripts/eos_node.mjs get-referer
```

#### delete-referer - 删除防盗链

**参数：**
无参数（使用配置文件中的桶名）

**返回示例：**

删除成功：
```json
{
  "success": true,
  "action": "delete-referer",
  "bucket": "my-bucket",
  "message": "防盗链已删除"
}
```

未设置防盗链：
```json
{
  "success": false,
  "action": "delete-referer",
  "error": "该桶未设置防盗链，无需删除"
}
```

**说明：**
- 删除桶的防盗链设置
- 删除后，所有域名都可以通过 HTTP Referer 访问桶中的资源
- 如果桶未设置防盗链，会返回错误提示
- 删除操作不可逆，请谨慎操作

**使用示例：**
```bash
# 删除防盗链
node scripts/eos_node.mjs delete-referer --bucket my-bucket
```

#### set-cors - 设置跨域规则（CORS）

**参数：**
- `--allowed-origins <list>`（必需）：允许的源（Origin），用逗号分隔
  - 使用 `*` 表示允许所有源
  - 使用 `https://example.com` 表示只允许该域名
  - 可以指定多个域名，用逗号分隔
- `--allowed-methods <methods>`（必需）：允许的 HTTP 方法，用逗号分隔
  - 支持的方法：GET、POST、PUT、DELETE、HEAD 等
- `--allowed-headers <headers>`（可选）：允许的请求头，用逗号分隔，默认为 `*`
- `--expose-headers <headers>`（可选）：暴露的响应头，用逗号分隔
- `--max-age <seconds>`（可选）：缓存时间（秒），默认 3600（1小时）

**返回示例：**
```json
{
  "success": true,
  "action": "set-cors",
  "bucket": "my-bucket",
  "cors": {
    "CORSRules": [
      {
        "AllowedOrigins": [
          "https://example.com",
          "https://www.example.com"
        ],
        "AllowedMethods": [
          "GET",
          "POST",
          "PUT",
          "DELETE",
          "HEAD"
        ],
        "AllowedHeaders": [
          "*"
        ],
        "ExposeHeaders": [
          "ETag",
          "Content-Length"
        ],
        "MaxAgeSeconds": 3600
      }
    ]
  },
  "message": "跨域规则设置成功"
}
```

**说明：**
- CORS（跨域资源共享）用于控制哪些网站可以跨域访问桶中的资源
- `AllowedOrigins`：指定允许的源域名
- `AllowedMethods`：指定允许的 HTTP 方法
- `AllowedHeaders`：指定允许的请求头
- `ExposeHeaders`：指定允许浏览器访问的响应头
- `MaxAgeSeconds`：指定浏览器缓存 CORS 规则的时间
- 设置后，指定的网站可以通过 AJAX 等方式访问桶中的资源

**使用场景：**
```bash
# 场景1：只允许自己的网站访问
node scripts/eos_node.mjs set-cors --allowed-origins "https://mywebsite.com" --allowed-methods "GET,POST" --max-age 3600

# 场景2：允许多个网站访问
node scripts/eos_node.mjs set-cors --allowed-origins "https://site1.com,https://site2.com,https://site3.com" --allowed-methods "GET,POST,PUT,DELETE,HEAD" --allowed-headers "*" --max-age 7200

# 场景3：允许所有网站访问（测试用）
node scripts/eos_node.mjs set-cors --allowed-origins "*" --allowed-methods "GET,POST,PUT,DELETE,HEAD" --allowed-headers "*" --max-age 3600

# 场景4：允许所有方法并暴露更多响应头
node scripts/eos_node.mjs set-cors --allowed-origins "https://mywebsite.com" --allowed-methods "GET,POST,PUT,DELETE,HEAD,OPTIONS" --allowed-headers "*" --expose-headers "ETag,Content-Length,Content-Type" --max-age 3600
```

#### get-cors - 查询跨域规则（CORS）

**参数：**
无参数（使用配置文件中的桶名）

**返回示例：**

已设置跨域规则：
```json
{
  "success": true,
  "action": "get-cors",
  "bucket": "my-bucket",
  "cors": {
    "CORSRules": [
      {
        "AllowedOrigins": [
          "https://example.com"
        ],
        "AllowedMethods": [
          "GET",
          "POST"
        ],
        "AllowedHeaders": [
          "*"
        ],
        "ExposeHeaders": [],
        "MaxAgeSeconds": 3600
      }
    ]
  },
  "message": "跨域规则查询成功"
}
```

未设置跨域规则：
```json
{
  "success": true,
  "action": "get-cors",
  "bucket": "my-bucket",
  "cors": null,
  "message": "未设置跨域规则"
}
```

**说明：**
- 查询当前桶的 CORS 配置
- 返回完整的 CORS 规则信息
- 如果未设置 CORS，会返回相应提示

**使用示例：**
```bash
# 查询 CORS 设置
node scripts/eos_node.mjs get-cors
```

#### delete-cors - 删除跨域规则（CORS）

**参数：**
无参数（使用配置文件中的桶名）

**返回示例：**

删除成功：
```json
{
  "success": true,
  "action": "delete-cors",
  "bucket": "my-bucket",
  "message": "跨域规则已删除"
}
```

**说明：**
- 删除桶的 CORS 配置
- 删除后，所有网站都无法通过跨域方式访问桶中的资源（除非其他策略允许）
- 删除操作不可逆，请谨慎操作

**使用示例：**
```bash
# 删除 CORS 规则
node scripts/eos_node.mjs delete-cors
```

#### upload-object - 上传本地文件

**参数：**
- `--file <path>`（必需）：本地文件路径
- `--key <key>`（可选）：存储桶内的目标路径，默认为文件名

**返回示例：**
```json
{
  "success": true,
  "action": "upload-object",
  "key": "remote/path/file.jpg",
  "filePath": "/path/to/file.jpg",
  "size": 1048576,
  "message": "文件上传成功"
}
```

**说明：**
- 支持大文件上传（超过 500MB 自动使用分片上传）
- 自动处理 Windows 路径分隔符（反斜杠转换为正斜杠）
- 支持位置参数：第一个参数是文件路径，第二个参数是 key

**使用示例：**
```bash
# 使用参数上传
node scripts/eos_node.mjs upload-object --file /path/to/file.jpg --key remote/path/file.jpg

# 使用位置参数上传
node scripts/eos_node.mjs upload "/path/to/file.jpg" "remote/path/file.jpg"
```

#### download-object - 下载文件到本地

**参数：**
- `--key <key>`（必需）：存储桶内的文件路径
- `--output <path>`（可选）：本地保存路径，默认为文件名

**返回示例：**
```json
{
  "success": true,
  "action": "download-object",
  "key": "remote/path/file.jpg",
  "outputPath": "file.jpg",
  "message": "文件下载成功"
}
```

**说明：**
- 自动处理 Windows 路径分隔符（反斜杠转换为正斜杠）

**使用示例：**
```bash
# 下载文件
node scripts/eos_node.mjs download-object --key remote/path/file.jpg --output /path/to/save/file.jpg
```

#### copy-object - 复制文件

**参数：**
- `--source-key <src>`（必需）：源文件路径
- `--dest-key <dest>`（必需）：目标文件路径
- `--dest-bucket <bucket>`（可选）：目标桶名称，默认为当前桶（环境变量中的 EOS_BUCKET）

**返回示例：**

同一桶内复制：
```json
{
  "success": true,
  "action": "copy-object",
  "sourceKey": "images/photo.jpg",
  "sourceBucket": "my-bucket",
  "destKey": "backup/photo.jpg",
  "destBucket": "my-bucket",
  "message": "文件复制成功"
}
```

跨桶复制：
```json
{
  "success": true,
  "action": "copy-object",
  "sourceKey": "documents/report.pdf",
  "sourceBucket": "source-bucket",
  "destKey": "documents/report.pdf",
  "destBucket": "target-bucket",
  "message": "文件复制成功"
}
```

**说明：**
- 支持同一桶内复制文件
- 支持跨桶复制文件
- 源文件必须存在
- 如果目标文件已存在，会被覆盖
- 自动处理 Windows 路径分隔符（反斜杠转换为正斜杠）

**使用场景：**
```bash
# 场景1：同一桶内复制（备份文件）
node scripts/eos_node.mjs copy-object --source-key "images/photo.jpg" --dest-key "backup/photo.jpg"

# 场景2：跨桶复制（数据迁移）
node scripts/eos_node.mjs copy-object --source-key "data/file.csv" --dest-key "data/file.csv" --dest-bucket "backup-bucket"

# 场景3：复制到不同目录
node scripts/eos_node.mjs copy-object --source-key "temp/file.txt" --dest-key "documents/file.txt"
```

#### list-objects - 列出文件（支持分页）

**参数：**
- `--prefix <prefix>`（可选）：只列出以此前缀开头的文件
- `--max-keys <n>`（可选）：每页返回的最大文件数，默认 100，最大 100
- `--marker <marker>`（可选）：用于获取下一页的标记（从上一次请求的 `nextMarker` 获取）

**返回示例：**

第一页请求：
```json
{
  "success": true,
  "action": "list-objects",
  "prefix": "",
  "maxKeys": 100,
  "marker": null,
  "nextMarker": "file100.jpg",
  "isTruncated": true,
  "count": 100,
  "data": {
    "Contents": [
      {
        "Key": "file1.jpg",
        "LastModified": "2025-03-13T10:00:00.000Z",
        "Size": 1024
      }
      // ... 更多文件
    ]
  }
}
```

**分页使用方法：**
1. 第一次调用不提供 `--marker`
2. 如果返回的 `isTruncated` 为 `true`，说明还有更多文件
3. 使用返回的 `nextMarker` 作为下一次请求的 `--marker` 参数
4. 重复直到 `isTruncated` 为 `false`

**示例：**
```bash
# 第一页
node scripts/eos_node.mjs list-objects --max-keys 100

# 第二页（使用第一页返回的 nextMarker）
node scripts/eos_node.mjs list-objects --max-keys 100 --marker "file100.jpg"
```

#### delete-object - 删除文件

**参数：**
- `--key <key>`（必需）：存储桶内的文件路径

**返回示例：**
```json
{
  "success": true,
  "action": "delete-object",
  "key": "remote/path/file.jpg",
  "message": "文件删除成功"
}
```

**说明：**
- 自动处理 Windows 路径分隔符（反斜杠转换为正斜杠）
- 删除操作不可逆，请谨慎操作

**使用示例：**
```bash
# 删除文件
node scripts/eos_node.mjs delete-object --key remote/path/file.jpg
```

#### head-object - 查看文件元信息

**参数：**
- `--key <key>`（必需）：存储桶内的文件路径

**返回示例：**
```json
{
  "success": true,
  "action": "head-object",
  "key": "remote/path/file.jpg",
  "data": {
    "ContentLength": 1048576,
    "ContentType": "image/jpeg",
    "LastModified": "2025-03-13T10:00:00.000Z",
    "ETag": "\"abc123def456\""
  }
}
```

**说明：**
- 自动处理 Windows 路径分隔符（反斜杠转换为正斜杠）
- 不会下载文件内容，只返回元信息

**使用示例：**
```bash
# 查看文件元信息
node scripts/eos_node.mjs head-object --key remote/path/file.jpg
```

#### exists-object - 判断文件是否存在

**参数：**
- `--key <key>`（必需）：存储桶内的文件路径

**返回示例：**

文件存在时：
```json
{
  "success": true,
  "action": "exists-object",
  "key": "remote/path/file.jpg",
  "exists": true,
  "message": "文件存在"
}
```

文件不存在时：
```json
{
  "success": true,
  "action": "exists-object",
  "key": "remote/path/file.jpg",
  "exists": false,
  "message": "文件不存在"
}
```

#### get-object-acl - 查询文件访问权限

**参数：**
- `--key <key>`（必需）：存储桶内的文件路径

**返回示例：**
```json
{
  "success": true,
  "action": "get-object-acl",
  "key": "remote/path/file.jpg",
  "data": {
    "Owner": {
      "DisplayName": "owner",
      "ID": "owner-id"
    },
    "Grants": [
      {
        "Grantee": {
          "Type": "CanonicalUser",
          "DisplayName": "owner",
          "ID": "owner-id"
        },
        "Permission": "FULL_CONTROL"
      }
    ]
  }
}
```

#### set-object-acl - 设置文件访问权限

**参数：**
- `--key <key>`（必需）：存储桶内的文件路径
- `--acl <acl>`（必需）：权限类型，支持以下值：
  - `private` - 私有：仅所有者有完全控制权限
  - `public-read` - 公共读：所有者有完全控制权限，所有人可读
  - `public-read-write` - 公共读写：所有者有完全控制权限，所有人可读可写

**返回示例：**
```json
{
  "success": true,
  "action": "set-object-acl",
  "key": "remote/path/file.jpg",
  "acl": "public-read",
  "message": "文件权限设置成功"
}
```

#### generate-url - 生成文件共享外链

**参数：**
- `--key <key>`（必需）：存储桶内的文件路径
- `--expires <n>`（可选）：过期时间，单位秒，默认 3600 秒（1小时）

**返回示例：**
```json
{
  "success": true,
  "action": "generate-url",
  "key": "remote/path/file.jpg",
  "url": "https://eos-anhui-1.cmecloud.cn/mybucket/remote/path/file.jpg?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=xxx&X-Amz-Date=20250313T000000Z&X-Amz-Expires=3600&X-Amz-SignedHeaders=host&X-Amz-Signature=xxx",
  "expiresIn": 3600,
  "expiresAt": "2025-03-13T01:00:00.000Z",
  "message": "文件共享外链生成成功"
}
```

**说明：**
- 生成的外链包含签名和过期时间
- 外链仅在 `expiresIn` 指定的秒数内有效
- 过期后外链将无法访问
- 请勿将包含敏感信息的文件生成公共外链

---

**参数：**
- `--key <key>`（必需）：存储桶内的文件路径
- `--expires <n>`（可选）：过期时间，单位秒，默认 3600 秒（1小时）

**返回示例：**
```json
{
  "success": true,
  "action": "generate-url",
  "key": "remote/path/file.jpg",
  "url": "https://eos-anhui-1.cmecloud.cn/mybucket/remote/path/file.jpg?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=xxx&X-Amz-Date=20250313T000000Z&X-Amz-Expires=3600&X-Amz-SignedHeaders=host&X-Amz-Signature=xxx",
  "expiresIn": 3600,
  "expiresAt": "2025-03-13T01:00:00.000Z",
  "message": "文件共享外链生成成功"
}
```

**说明：**
- 生成的外链包含签名和过期时间
- 外链仅在 `expiresIn` 指定的秒数内有效
- 过期后外链将无法访问
- 请勿将包含敏感信息的文件生成公共外链

---
