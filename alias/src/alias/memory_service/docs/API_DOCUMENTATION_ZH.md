# 用户画像服务 API 文档

## 概览

用户画像服务（User Profiling Service）是一个用于用户画像的独立服务，提供用户记忆管理、行为记录与任务管理等能力。该服务基于 FastAPI 构建，支持异步操作与后台任务处理。

**服务信息：**

- 服务名称：User Profiling Service
- 版本：0.1.0
- Base URL：`http://localhost:8000`

## 认证与错误处理

### 错误响应格式

所有错误响应遵循统一格式：

```json
{
  "error_code": "ERROR_CODE",
  "message": "Error description",
  "details": {
    "errors": [
      {
        "field": "field_path",
        "message": "field_error_message",
        "type": "error_type"
      }
    ]
  },
  "timestamp": "2024-01-01T12:00:00"
}
```

### 常见错误码

| Error Code | Status Code | 说明 |
|------------|-------------|------|
| `VALIDATION_ERROR` | 400/422 | 请求数据校验失败 |
| `MISSING_REQUIRED_FIELD` | 400 | 缺少必填字段 |
| `EMPTY_STRING_FIELD` | 400 | 字段为空字符串 |
| `USER_NOT_FOUND` | 404 | 用户不存在 |
| `TASK_NOT_FOUND` | 404 | 任务不存在 |
| `MEMORY_SERVICE_ERROR` | 503 | 记忆服务错误 |
| `SERVICE_UNAVAILABLE` | 503 | 服务不可用 |
| `INTERNAL_SERVER_ERROR` | 500 | 服务内部错误 |

## API 接口

### 1. 健康检查

#### GET /health

检查服务健康状态。

**响应示例：**

```json
{
  "status": "healthy",
  "service": "user_profiling_service",
  "mem0_available": true,
  "memory_utils_available": true
}
```

### 2. 记忆管理（Memory Management）

#### POST /alias_memory_service/user_profiling/add

向记忆服务发送内容，系统会处理并存储。该接口会提交后台任务并返回 `submit_id`。

**请求参数：**

```json
{
  "uid": "user_id",
  "content": [
    {
      "role": "user",
      "content": "User message content"
    },
    {
      "role": "assistant",
      "content": "Assistant response content"
    }
  ]
}
```

**响应示例：**

```json
{
  "status": "submit success",
  "submit_id": "uuid-string"
}
```

#### POST /alias_memory_service/user_profiling/clear

清空指定用户的全部记忆。

**请求参数：**

```json
{
  "uid": "user_id"
}
```

**响应示例：**

```json
{
  "status": "submit success",
  "submit_id": "uuid-string"
}
```

#### POST /alias_memory_service/user_profiling/retrieve

从记忆服务中检索相关信息。

**请求参数：**

```json
{
  "uid": "user_id",
  "query": "query content"
}
```

**响应示例：**

```json
{
  "status": "success",
  "uid": "user_id",
  "query": "query content",
  "data": {
    "candidates": {
      "results": [
        {
          "id": "b904ba2c-3f87-4050-981e-b5fe5afe1ad0",
          "memory": "Likes watching sci-fi movies",
          "hash": "904b4e33b7721425dee845bff17b08fa",
          "metadata": null,
          "score": 0.5977808,
          "created_at": "2025-07-20T23:11:00.221851-07:00",
          "updated_at": null,
          "user_id": "test_user_basic"
        }
      ],
      "relations": null
    },
    "profiling": {},
    "user_info": {}
  }
}
```

**响应字段说明：**

- `candidates`：记忆检索的搜索结果
  - `results`：与 query 匹配的记忆条目列表
    - `id`：记忆条目唯一标识
    - `memory`：记忆文本内容
    - `hash`：记忆内容的 hash
    - `metadata`：元信息（可为 null）
    - `score`：相关性得分（0-1，越高越相关）
    - `created_at`：创建时间
    - `updated_at`：更新时间（可为 null）
    - `user_id`：用户 ID
  - `relations`：关系信息（可为 null）
- `profiling`：画像数据
- `user_info`：用户信息

#### POST /alias_memory_service/user_profiling/show_all

展示指定用户的全部记忆。

**请求参数：**

```json
{
  "uid": "user_id"
}
```

**响应示例：**

```json
{
  "status": "success",
  "uid": "user_id",
  "data": {
    "results": [
      {
        "id": "b904ba2c-3f87-4050-981e-b5fe5afe1ad0",
        "memory": "Likes watching sci-fi movies",
        "hash": "904b4e33b7721425dee845bff17b08fa",
        "metadata": null,
        "score": 0.5977808,
        "created_at": "2025-07-20T23:11:00.221851-07:00",
        "updated_at": null,
        "user_id": "test_user_basic"
      }
    ],
    "relations": null
  }
}
```

#### POST /alias_memory_service/user_profiling/show_all_user_profiles

展示指定用户的全部画像（profiles）。

**请求参数：**

```json
{
  "uid": "user_id"
}
```

**响应示例：**

```json
{
  "status": "success",
  "data": [
    {
      "pid": "profile_id",
      "uid": "user_id",
      "content": "Profile content text",
      "metadata": {
        "session_id": "session_id",
        "is_confirmed": 0
      }
    }
  ]
}
```

**响应字段说明：**

- `pid`：画像 ID
- `uid`：用户 ID
- `content`：画像内容
- `metadata`：画像元信息
  - `session_id`：会话 ID（可为 null）
  - `is_confirmed`：确认状态（0：未确认，1：已确认）

### 3. 工具记忆（Tool Memory）

#### POST /alias_memory_service/tool_memory/retrieve

根据 query 检索工具记忆，用于获取与工具使用相关的记忆信息。

**请求参数：**

```json
{
  "uid": "user_id",
  "query": "web_search,write_file"
}
```

**响应示例：**

```json
{
  "status": "success",
  "data": {
    "results": [
      {
        "id": "memory_id",
        "memory": "Tool usage information",
        "score": 0.8
      }
    ],
    "relations": null
  }
}
```

### 4. 行为记录（Action Recording）

#### POST /alias_memory_service/record_action

记录用户行为，支持多种 action 类型。该接口会提交后台任务并返回 `submit_id`。

**请求参数：**

```json
{
  "uid": "user_id",
  "session_id": "session_id",
  "action_type": "LIKE",
  "message_id": "message_id",
  "reference_time": "2024-01-01T12:00:00",
  "data": {}
}
```

**支持的 Action 类型：**

**反馈类：**

- `LIKE` - 点赞
- `DISLIKE` - 点踩
- `CANCEL_LIKE` - 取消点赞
- `CANCEL_DISLIKE` - 取消点踩

**收藏类：**

- `COLLECT_TOOL` - 收藏工具
- `UNCOLLECT_TOOL` - 取消收藏工具
- `COLLECT_SESSION` - 收藏会话
- `UNCOLLECT_SESSION` - 取消收藏会话

**对话类：**

- `START_CHAT` - 开始对话
- `FOLLOWUP_CHAT` - 追问对话
- `BREAK_CHAT` - 中断对话

**编辑/操作类：**

- `EDIT_ROADMAP` - 编辑 roadmap
- `EDIT_FILE` - 编辑文件
- `EXECUTE_SHELL_COMMAND` - 执行 shell 命令
- `BROWSER_OPERATION` - 浏览器操作

**任务类：**

- `TASK_STOP` - 任务停止（数据会存入 tool_memory）

**注意：**

- `action_type` 字段既可以传枚举字符串，也可以使用旧版字段 `action` 以保持兼容。
- `data` 字段可包含与 action 类型相关的数据结构，例如：
  - 反馈/收藏类：`ChangeRecord`（包含 `previous` 与 `current`）
  - 对话类：`QueryRecord`（包含 `query`）
  - 操作类：`OperationRecord`（包含 `operation_type` 与 `operation_data`）
  - roadmap 编辑：`Roadmap`（包含 `content` 与 `metadata`）

**响应示例：**

```json
{
  "status": "submit success",
  "submit_id": "uuid-string"
}
```

### 5. 画像直接操作（Direct Profile Operations）

#### POST /alias_memory_service/user_profiling/direct_add_profile

直接新增用户画像。

**请求参数：**

```json
{
  "uid": "user_id",
  "content": "Profile content text"
}
```

**响应示例：**

```json
{
  "status": "success",
  "uid": "user_id",
  "pid": "profile_id",
  "data": {
    "results": [
      {
        "id": "profile_id",
        "memory": "Profile content text",
        "user_id": "user_id"
      }
    ]
  }
}
```

#### POST /alias_memory_service/user_profiling/direct_delete_by_profiling_id

按 Profiling ID 删除画像。

**请求参数：**

```json
{
  "uid": "user_id",
  "pid": "Profiling ID"
}
```

**响应示例：**

```json
{
  "status": "success",
  "uid": "user_id",
  "pid": "Profiling ID",
  "data": {
    "deleted": true
  }
}
```

#### POST /alias_memory_service/user_profiling/direct_update_profile

按 Profiling ID 更新画像内容。

**请求参数：**

```json
{
  "uid": "user_id",
  "pid": "Profiling ID",
  "content_before": "original profiling content",
  "content_after": "updated profiling content"
}
```

**响应示例：**

```json
{
  "status": "success",
  "uid": "user_id",
  "pid": "Profiling ID",
  "data": {
    "updated": true
  }
}
```

#### POST /alias_memory_service/user_profiling/direct_confirm_profile

确认画像（将 `is_confirmed` 置为 1）。

**请求参数：**

```json
{
  "uid": "user_id",
  "pid": "Profiling ID"
}
```

**响应示例：**

```json
{
  "status": "success",
  "data": {
    "pid": "profile_id",
    "uid": "user_id",
    "content": "Profile content text",
    "metadata": {
      "session_id": "session_id",
      "is_confirmed": 1
    }
  }
}
```

### 6. 任务管理（Task Management）

#### GET /alias_memory_service/task_status/{submit_id}

获取指定任务状态。

**路径参数：**

- `submit_id`：任务提交 ID

**响应示例：**

```json
{
  "submit_id": "uuid-string",
  "status": "completed",
  "data": {
    "status": "completed",
    "result": {},
    "created_at": "2024-01-01T12:00:00",
    "completed_at": "2024-01-01T12:05:00"
  }
}
```

#### GET /alias_memory_service/all_tasks

获取所有被跟踪的任务（用于调试/监控）。

**响应示例：**

```json
{
  "status": "success",
  "data": {
    "task-id-1": {
      "status": "completed",
      "result": {},
      "created_at": "2024-01-01T12:00:00"
    },
    "task-id-2": {
      "status": "running",
      "created_at": "2024-01-01T12:10:00"
    }
  }
}
```

#### GET /alias_memory_service/tasks_by_date/{date_str}

获取指定日期的所有任务。

**路径参数：**

- `date_str`：日期字符串（YYYY-MM-DD）

**响应示例：**

```json
{
  "status": "success",
  "date": "2024-01-01",
  "data": [
    {
      "submit_id": "uuid-string",
      "status": "completed",
      "created_at": "2024-01-01T12:00:00"
    }
  ]
}
```

#### GET /alias_memory_service/tasks_by_date_range

获取日期范围内的所有任务。

**查询参数：**

- `start_date`：开始日期（YYYY-MM-DD，必填）
- `end_date`：结束日期（YYYY-MM-DD，必填）

**响应示例：**

```json
{
  "status": "success",
  "start_date": "2024-01-01",
  "end_date": "2024-01-07",
  "data": [
    {
      "submit_id": "uuid-string",
      "status": "completed",
      "created_at": "2024-01-01T12:00:00"
    }
  ]
}
```

#### GET /alias_memory_service/storage_stats

获取任务文件存储统计信息。

**响应示例：**

```json
{
  "status": "success",
  "data": {
    "total_tasks": 100,
    "completed_tasks": 95,
    "failed_tasks": 3,
    "running_tasks": 2,
    "storage_size_mb": 15.5
  }
}
```

## 任务状态（Task Status）

可能的任务状态包括：

- `running` - 运行中
- `completed` - 已完成
- `failed` - 已失败

## 使用示例

### Python 客户端示例

```python
import asyncio
import aiohttp
import json

async def add_memory(uid: str, content: list):
    async with aiohttp.ClientSession() as session:
        url = "http://localhost:8000/alias_memory_service/user_profiling/add"
        data = {
            "uid": uid,
            "content": content
        }

        async with session.post(url, json=data) as response:
            result = await response.json()
            return result

async def retrieve_memory(uid: str, query: str):
    async with aiohttp.ClientSession() as session:
        url = "http://localhost:8000/alias_memory_service/user_profiling/retrieve"
        data = {
            "uid": uid,
            "query": query
        }

        async with session.post(url, json=data) as response:
            result = await response.json()
            return result

async def check_task_status(submit_id: str):
    async with aiohttp.ClientSession() as session:
        url = f"http://localhost:8000/alias_memory_service/task_status/{submit_id}"

        async with session.get(url) as response:
            result = await response.json()
            return result

async def record_action(uid: str, session_id: str, action_type: str):
    async with aiohttp.ClientSession() as session:
        url = "http://localhost:8000/alias_memory_service/record_action"
        data = {
            "uid": uid,
            "session_id": session_id,
            "action_type": action_type
        }

        async with session.post(url, json=data) as response:
            result = await response.json()
            return result

# Usage example
async def main():
    # Add memory
    content = [
        {"role": "user", "content": "I like sci-fi movies"},
        {"role": "assistant", "content": "Sci-fi movies are interesting! Which one do you like best?"}
    ]

    result = await add_memory("user123", content)
    submit_id = result["submit_id"]

    # Check task status
    while True:
        status = await check_task_status(submit_id)
        if status["status"] in ["completed", "failed"]:
            print(f"Task completed, status: {status['status']}")
            break
        await asyncio.sleep(5)

    # Retrieve memory
    retrieve_result = await retrieve_memory("user123", "What type of movies do I like")
    print(f"Retrieved memories: {retrieve_result}")

# Run example
asyncio.run(main())
```

### cURL 示例

```bash
# Health check
curl -X GET "http://localhost:8000/health"

# Add memory
curl -X POST "http://localhost:8000/alias_memory_service/user_profiling/add" \
  -H "Content-Type: application/json" \
  -d '{
    "uid": "user123",
    "content": [
      {"role": "user", "content": "I like sci-fi movies"},
      {"role": "assistant", "content": "Sci-fi movies are interesting!"}
    ]
  }'

# Retrieve memory
curl -X POST "http://localhost:8000/alias_memory_service/user_profiling/retrieve" \
  -H "Content-Type: application/json" \
  -d '{
    "uid": "user123",
    "query": "What type of movies do I like"
  }'

# Show all memory
curl -X POST "http://localhost:8000/alias_memory_service/user_profiling/show_all" \
  -H "Content-Type: application/json" \
  -d '{
    "uid": "user123"
  }'

# Show all user profiles
curl -X POST "http://localhost:8000/alias_memory_service/user_profiling/show_all_user_profiles" \
  -H "Content-Type: application/json" \
  -d '{
    "uid": "user123"
  }'

# Record action
curl -X POST "http://localhost:8000/alias_memory_service/record_action" \
  -H "Content-Type: application/json" \
  -d '{
    "uid": "user123",
    "session_id": "session_id",
    "action_type": "LIKE",
    "message_id": "message_id"
  }'

# Retrieve tool memory
curl -X POST "http://localhost:8000/alias_memory_service/tool_memory/retrieve" \
  -H "Content-Type: application/json" \
  -d '{
    "uid": "user123",
    "query": "web_search,write_file"
  }'

# Check task status
curl -X GET "http://localhost:8000/alias_memory_service/task_status/{submit_id}"

# Get all tasks
curl -X GET "http://localhost:8000/alias_memory_service/all_tasks"

# Get tasks by date
curl -X GET "http://localhost:8000/alias_memory_service/tasks_by_date/2024-01-01"

# Get tasks by date range
curl -X GET "http://localhost:8000/alias_memory_service/tasks_by_date_range?start_date=2024-01-01&end_date=2024-01-07"

# Get storage stats
curl -X GET "http://localhost:8000/alias_memory_service/storage_stats"
```

## 重要说明

1. **异步操作**：多数记忆相关操作（add、clear、record_action）为异步，返回 `submit_id`；应通过 task status 接口查询完成状态。
2. **错误处理**：所有接口都包含较完整的错误处理机制，会返回详细错误信息。
3. **数据校验**：所有请求都会进行数据校验，确保必填字段存在且格式正确。
4. **会话管理**：行为记录功能需要有效的 session ID 才能检索会话内容。
5. **记忆服务依赖**：该服务依赖 mem0ai 记忆服务，请确保其可用。
6. **日志**：所有操作会写入日志文件，便于调试与监控。
7. **工具记忆**：`TASK_STOP` action 类型会将数据存入 tool_memory，而不是 user_profiling。
8. **画像确认**：使用 `direct_confirm_profile` 将画像标记为已确认（`is_confirmed=1`）。

## 部署说明

服务默认监听 `0.0.0.0:8000`，可以通过以下方式启动：

```bash
python main.py
```

或使用 uvicorn：

```bash
uvicorn main:app --host 0.0.0.0 --port 8000
```

也可通过环境变量指定端口：

```bash
export USER_PROFILING_SERVICE_PORT=8000
uvicorn main:app --host 0.0.0.0 --port $USER_PROFILING_SERVICE_PORT
```

该服务支持 CORS，可从任意来源访问。

