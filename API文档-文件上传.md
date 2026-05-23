# 文件上传功能 API 文档

## 概述
本文档描述了寒家培训系统的文件上传功能相关API接口。

## 基础信息
- **Base URL**: `http://localhost:8888/api`
- **认证方式**: Token认证（需要在请求头中携带Token）
- **Content-Type**: `multipart/form-data`（上传接口）

---

## 接口列表

### 1. 文件上传

**接口地址**: `POST /api/file/upload`

**描述**: 上传文件到服务器，并保存附件信息到数据库

**请求参数**:
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| file | File | 是 | 要上传的文件 |
| stageId | int | 是 | 阶段ID |

**请求示例**:
```bash
curl -X POST http://localhost:8888/api/file/upload \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -F "file=@/path/to/file.pdf" \
  -F "stageId=1"
```

**响应示例**:
```json
{
  "code": 200,
  "message": "上传成功",
  "data": {
    "attachmentId": 1,
    "fileName": "document.pdf",
    "filePath": "/uploads/20260522/1234567890_abc123.pdf",
    "fileSize": "1.25MB",
    "uploadTime": "2026-05-22 10:30:00"
  }
}
```

**错误响应**:
- 400: 文件为空 / 文件大小超过限制 / 不支持的文件类型
- 401: 未登录或Token无效
- 500: 上传失败

**支持的文件类型**:
- 图片: JPEG, PNG, GIF, BMP
- 文档: PDF, Word (.doc, .docx), Excel (.xls, .xlsx)
- 文本: TXT

**文件大小限制**: 最大10MB

**存储规则**:
- 文件按日期分目录存储（格式：yyyyMMdd）
- 文件名格式：时间戳_UUID.扩展名
- 物理存储路径：`D:/project_files/hanjia/yyyyMMdd/`

---

### 2. 删除文件

**接口地址**: `DELETE /api/file/delete/{attachmentId}`

**描述**: 删除指定的附件（包括物理文件和数据库记录）

**路径参数**:
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| attachmentId | int | 是 | 附件ID |

**请求示例**:
```bash
curl -X DELETE http://localhost:8888/api/file/delete/1 \
  -H "Authorization: Bearer YOUR_TOKEN"
```

**响应示例**:
```json
{
  "code": 200,
  "message": "删除成功",
  "data": null
}
```

**错误响应**:
- 401: 未登录或Token无效
- 403: 无权删除此文件（只能删除自己的文件）
- 404: 附件不存在
- 500: 删除失败

**权限说明**:
- 用户只能删除自己上传的文件
- 删除操作会同时删除物理文件和数据库记录

---

### 3. 获取阶段附件列表

**接口地址**: `GET /api/file/list?stageId={stageId}`

**描述**: 获取指定阶段的所有附件列表

**查询参数**:
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| stageId | int | 是 | 阶段ID |

**请求示例**:
```bash
curl -X GET http://localhost:8888/api/file/list?stageId=1 \
  -H "Authorization: Bearer YOUR_TOKEN"
```

**响应示例**:
```json
{
  "code": 200,
  "message": "获取成功",
  "data": [
    {
      "attachmentId": 1,
      "fileName": "document.pdf",
      "filePath": "/uploads/20260522/1234567890_abc123.pdf",
      "fileType": "application/pdf",
      "fileSize": 1310720,
      "taskId": null,
      "userId": 1,
      "stageId": 1
    },
    {
      "attachmentId": 2,
      "fileName": "image.jpg",
      "filePath": "/uploads/20260522/1234567891_def456.jpg",
      "fileType": "image/jpeg",
      "fileSize": 524288,
      "taskId": null,
      "userId": 1,
      "stageId": 1
    }
  ]
}
```

**错误响应**:
- 401: 未登录或Token无效

---

### 4. 获取阶段详情（包含附件）

**接口地址**: `GET /api/stage/detail?stageId={stageId}&userId={userId}`

**描述**: 获取阶段的详细信息，包括任务列表、完成状态和附件列表

**查询参数**:
| 参数名 | 类型 | 必填 | 默认值 | 说明 |
|--------|------|------|--------|------|
| stageId | int | 是 | - | 阶段ID |
| userId | int | 否 | 1 | 用户ID |

**请求示例**:
```bash
curl -X GET http://localhost:8888/api/stage/detail?stageId=1&userId=1 \
  -H "Authorization: Bearer YOUR_TOKEN"
```

**响应示例**:
```json
{
  "code": 200,
  "message": "获取成功",
  "data": {
    "stage": {
      "stageId": 1,
      "stageName": "第一阶段",
      "stageStarttime": "2024-01-01",
      "stageEndtime": "2024-01-31"
    },
    "tasks": [
      {
        "taskId": 1,
        "taskTitle": "任务一",
        "taskDesc": "任务描述",
        "completed": 1
      }
    ],
    "taskStatusMap": {
      "1": 1,
      "2": 0
    },
    "attachments": [
      {
        "attachmentId": 1,
        "fileName": "document.pdf",
        "filePath": "/uploads/20260522/1234567890_abc123.pdf",
        "fileType": "application/pdf",
        "fileSize": 1310720,
        "taskId": null,
        "userId": 1,
        "stageId": 1
      }
    ],
    "totalTasks": 2,
    "completedTasks": 1
  }
}
```

---

## 配置说明

### application.properties 配置项

```properties
# 文件上传大小限制
spring.servlet.multipart.max-file-size=10MB
spring.servlet.multipart.max-request-size=10MB

# 文件存储路径
upload.filePath=D:/project_files/hanjia
```

### 文件存储结构

```
D:/project_files/hanjia/
├── 20260522/
│   ├── 1234567890_abc123.pdf
│   └── 1234567891_def456.jpg
├── 20260523/
│   └── 1234567892_ghi789.docx
└── ...
```

---

## 注意事项

1. **认证要求**: 所有接口都需要通过Token认证，请在请求头中携带有效的Token
2. **文件大小**: 单个文件不能超过10MB
3. **文件类型**: 仅支持常见的图片、文档和文本文件
4. **权限控制**: 用户只能查看和删除自己上传的文件
5. **文件命名**: 系统会自动生成唯一文件名，避免重名覆盖
6. **目录创建**: 系统会自动按日期创建子目录
7. **删除操作**: 删除附件时会同时删除物理文件和数据库记录

---

## 前端调用示例（JavaScript）

### 上传文件

```javascript
const formData = new FormData();
formData.append('file', fileInput.files[0]);
formData.append('stageId', 1);

fetch('http://localhost:8888/api/file/upload', {
  method: 'POST',
  headers: {
    'Authorization': 'Bearer YOUR_TOKEN'
  },
  body: formData
})
.then(response => response.json())
.then(data => {
  console.log('上传成功:', data);
})
.catch(error => {
  console.error('上传失败:', error);
});
```

### 获取附件列表

```javascript
fetch('http://localhost:8888/api/file/list?stageId=1', {
  method: 'GET',
  headers: {
    'Authorization': 'Bearer YOUR_TOKEN'
  }
})
.then(response => response.json())
.then(data => {
  console.log('附件列表:', data);
});
```

### 删除文件

```javascript
fetch('http://localhost:8888/api/file/delete/1', {
  method: 'DELETE',
  headers: {
    'Authorization': 'Bearer YOUR_TOKEN'
  }
})
.then(response => response.json())
.then(data => {
  console.log('删除成功:', data);
});
```

---

## 常见问题

### Q1: 上传失败，提示"文件大小超过限制"
**A**: 检查配置文件中的 `spring.servlet.multipart.max-file-size` 设置，确保文件大小不超过限制。

### Q2: 上传失败，提示"不支持的文件类型"
**A**: 检查文件的MIME类型是否在允许列表中。可以在 `FileUploadController.ALLOWED_TYPES` 中添加新的文件类型。

### Q3: 如何修改文件存储路径？
**A**: 修改 `application.properties` 中的 `upload.filePath` 配置项。

### Q4: 文件上传后如何访问？
**A**: 文件存储在配置的目录下，可以通过返回的 `filePath` 字段获取相对路径。如果需要提供下载功能，可以添加一个文件下载接口。

---

## 更新日志

- **2026-05-22**: 初始版本，实现文件上传、删除和查询功能
