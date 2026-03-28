# 日历管理API接口文档

## 概述

日历管理API提供日历事件的创建、查询、更新、删除等功能，支持个人事件和家庭事件管理，包含重复事件处理。所有接口都需要JWT认证。

**基础URL**: `http://your-domain.com/calendar`

**注意**: 根据蓝图注册配置，路由前缀为 `/calendar`

## 接口列表

### 1. 创建日历事件

**接口说明**: 创建新的日历事件，支持个人事件和家庭事件

**URL**: `POST /`

**请求头**:
```
Authorization: Bearer <jwt_token>
Content-Type: application/json
```

**请求体**:
| 字段名 | 类型 | 必填 | 说明 | 约束 |
|--------|------|------|------|------|
| title | string | 是 | 事件标题 | - |
| date | string | 是 | 事件日期 | YYYY-MM-DD格式 |
| description | string | 否 | 事件描述 | - |
| color | string | 否 | 事件颜色 | 十六进制颜色码，默认#8CA9AD |
| recurring_type | string | 否 | 重复类型 | no_recurring, recurring_weekly, recurring_monthly, recurring_yearly |
| recurring_rule | string | 否 | 重复规则 | - |
| recurring_end_date | string | 否 | 重复结束日期 | YYYY-MM-DD格式 |
| family_id | int | 否 | 家庭ID | 创建家庭事件时必填 |

**请求示例**:
```json
{
  "title": "家庭聚餐",
  "date": "2024-03-15",
  "description": "周末家庭聚餐",
  "color": "#FF6B6B",
  "recurring_type": "no_recurring",
  "family_id": 1
}
```

**响应示例**:
```json
{
  "success": true,
  "message": "日历事件创建成功",
  "data": {
    "id": 1,
    "title": "家庭聚餐",
    "description": "周末家庭聚餐",
    "color": "#FF6B6B",
    "date": "2024-03-15",
    "recurring_type": "no_recurring",
    "recurring_rule": null,
    "recurring_end_date": null,
    "family_id": 1,
    "created_by": 123,
    "is_active": true,
    "created_at": "2024-03-11T10:30:00",
    "updated_at": "2024-03-11T10:30:00"
  }
}
```

**状态码说明**:
- `200`: 创建成功
- `400`: 缺少必需字段、日期格式错误、无效的重复类型
- `403`: 无权在此家庭创建事件
- `500`: 服务器内部错误

### 2. 获取日历事件列表

**接口说明**: 获取当前用户相关的日历事件（包括个人事件和家庭事件）

**URL**: `GET /`

**请求头**:
```
Authorization: Bearer <jwt_token>
```

**查询参数**:
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| start_date | string | 否 | 开始日期 |
| end_date | string | 否 | 结束日期 |
| type | string | 否 | 事件类型 | all, personal, family |

**请求示例**:
```
GET /calendar/?start_date=2024-03-01&end_date=2024-03-31&type=all
```

**响应示例**:
```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "title": "家庭聚餐",
      "description": "周末家庭聚餐",
      "color": "#FF6B6B",
      "date": "2024-03-15",
      "recurring_type": "no_recurring",
      "family_id": 1,
      "created_by": 123,
      "is_active": true,
      "created_at": "2024-03-11T10:30:00",
      "updated_at": "2024-03-11T10:30:00"
    }
  ],
  "total": 1
}
```

**状态码说明**:
- `200`: 获取成功
- `400`: 日期格式错误
- `500`: 服务器内部错误

### 3. 获取单个事件详情

**接口说明**: 获取单个日历事件的详细信息

**URL**: `GET /<id>`

**请求头**:
```
Authorization: Bearer <jwt_token>
```

**请求示例**:
```
GET /calendar/1
```

**响应示例**:
```json
{
  "success": true,
  "data": {
    "id": 1,
    "title": "家庭聚餐",
    "description": "周末家庭聚餐",
    "color": "#FF6B6B",
    "date": "2024-03-15",
    "recurring_type": "no_recurring",
    "family_id": 1,
    "created_by": 123,
    "is_active": true,
    "created_at": "2024-03-11T10:30:00",
    "updated_at": "2024-03-11T10:30:00"
  }
}
```

**状态码说明**:
- `200`: 获取成功
- `403`: 无权访问此事件
- `404`: 事件不存在
- `500`: 服务器内部错误

### 4. 更新日历事件

**接口说明**: 更新日历事件信息

**URL**: `PUT /<id>`

**请求头**:
```
Authorization: Bearer <jwt_token>
Content-Type: application/json
```

**请求体**:
| 字段名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| title | string | 否 | 事件标题 |
| description | string | 否 | 事件描述 |
| color | string | 否 | 事件颜色 |
| date | string | 否 | 事件日期 |
| recurring_type | string | 否 | 重复类型 |
| recurring_rule | string | 否 | 重复规则 |
| recurring_end_date | string | 否 | 重复结束日期 |
| is_active | boolean | 否 | 是否激活 |

**请求示例**:
```json
{
  "title": "家庭聚餐（更新）",
  "description": "周末家庭聚餐，地点改为餐厅",
  "color": "#4ECDC4"
}
```

**响应示例**:
```json
{
  "success": true,
  "message": "日历事件更新成功",
  "data": {
    "id": 1,
    "title": "家庭聚餐（更新）",
    "description": "周末家庭聚餐，地点改为餐厅",
    "color": "#4ECDC4",
    "date": "2024-03-15",
    "recurring_type": "no_recurring",
    "family_id": 1,
    "created_by": 123,
    "is_active": true,
    "created_at": "2024-03-11T10:30:00",
    "updated_at": "2024-03-11T11:00:00"
  }
}
```

**状态码说明**:
- `200`: 更新成功
- `400`: 日期格式错误、无效的重复类型
- `403`: 无权修改此事件
- `404`: 事件不存在
- `500`: 服务器内部错误

### 5. 删除日历事件

**接口说明**: 删除日历事件

**URL**: `DELETE /<id>`

**请求头**:
```
Authorization: Bearer <jwt_token>
```

**请求示例**:
```
DELETE /calendar/1
```

**响应示例**:
```json
{
  "success": true,
  "message": "日历事件删除成功"
}
```

**状态码说明**:
- `200`: 删除成功
- `403`: 无权删除此事件
- `404`: 事件不存在
- `500`: 服务器内部错误

### 6. 获取今日事件

**接口说明**: 获取今日的所有事件

**URL**: `GET /today`

**请求头**:
```
Authorization: Bearer <jwt_token>
```

**响应示例**:
```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "title": "家庭聚餐",
      "description": "周末家庭聚餐",
      "color": "#FF6B6B",
      "date": "2024-03-15",
      "recurring_type": "no_recurring",
      "family_id": 1,
      "created_by": 123,
      "is_active": true,
      "created_at": "2024-03-11T10:30:00",
      "updated_at": "2024-03-11T10:30:00"
    }
  ],
  "date": "2024-03-15"
}
```

**状态码说明**:
- `200`: 获取成功
- `500`: 服务器内部错误

### 7. 获取即将发生事件

**接口说明**: 获取未来一段时间内即将发生的事件

**URL**: `GET /upcoming`

**请求头**:
```
Authorization: Bearer <jwt_token>
```

**查询参数**:
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| days | int | 否 | 未来天数，默认7天 |

**请求示例**:
```
GET /calendar/upcoming?days=14
```

**响应示例**:
```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "title": "家庭聚餐",
      "description": "周末家庭聚餐",
      "color": "#FF6B6B",
      "date": "2024-03-15",
      "recurring_type": "no_recurring",
      "family_id": 1,
      "created_by": 123,
      "is_active": true,
      "created_at": "2024-03-11T10:30:00",
      "updated_at": "2024-03-11T10:30:00"
    }
  ],
  "period": "2024-03-15 至 2024-03-29"
}
```

**状态码说明**:
- `200`: 获取成功
- `500`: 服务器内部错误

### 8. 获取已过期事件

**接口说明**: 获取已过期的日历事件

**URL**: `GET /overdue`

**请求头**:
```
Authorization: Bearer <jwt_token>
```

**响应示例**:
```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "title": "家庭聚餐",
      "description": "周末家庭聚餐",
      "color": "#FF6B6B",
      "date": "2024-03-10",
      "recurring_type": "no_recurring",
      "family_id": 1,
      "created_by": 123,
      "is_active": true,
      "created_at": "2024-03-11T10:30:00",
      "updated_at": "2024-03-11T10:30:00"
    }
  ],
  "total": 1
}
```

**状态码说明**:
- `200`: 获取成功
- `500`: 服务器内部错误

### 9. 获取重复类型列表

**接口说明**: 获取支持的重复事件类型列表

**URL**: `GET /recurring-types`

**请求头**:
```
Authorization: Bearer <jwt_token>
```

**响应示例**:
```json
{
  "success": true,
  "data": [
    {
      "value": "no_recurring",
      "label": "不重复"
    },
    {
      "value": "recurring_weekly",
      "label": "每周重复"
    },
    {
      "value": "recurring_monthly",
      "label": "每月重复"
    },
    {
      "value": "recurring_yearly",
      "label": "每年重复"
    }
  ]
}
```

**状态码说明**:
- `200`: 获取成功
- `500`: 服务器内部错误

### 10. 获取日历统计信息

**接口说明**: 获取日历事件的统计信息

**URL**: `GET /statistics`

**请求头**:
```
Authorization: Bearer <jwt_token>
```

**查询参数**:
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| days | int | 否 | 统计天数，默认30天 |

**请求示例**:
```
GET /calendar/statistics?days=60
```

**响应示例**:
```json
{
  "success": true,
  "data": {
    "period": "最近30天",
    "total_events": 15,
    "personal_events": 8,
    "family_events": 7,
    "recurring_stats": {
      "no_recurring": 12,
      "recurring_weekly": 2,
      "recurring_monthly": 1
    },
    "daily_stats": [
      ["2024-03-15", 1],
      ["2024-03-16", 2],
      ["2024-03-17", 1]
    ],
    "upcoming_count": 3,
    "today_count": 0
  }
}
```

**状态码说明**:
- `200`: 获取成功
- `500`: 服务器内部错误

## 错误处理

所有接口都使用统一的错误响应格式：

```json
{
  "error": "错误描述信息"
}
```

### 常见错误码说明

- **400 Bad Request**: 请求参数错误
  - 缺少必需字段
  - 日期格式错误
  - 无效的重复类型

- **401 Unauthorized**: 未授权
  - JWT token无效或过期

- **403 Forbidden**: 权限不足
  - 无权访问其他家庭的事件
  - 无权修改/删除其他用户创建的事件

- **404 Not Found**: 资源不存在
  - 事件不存在
  - 用户不存在

- **500 Internal Server Error**: 服务器内部错误
  - 数据库操作失败
  - 未知的服务器错误

## 前端使用示例

### 1. 创建日历事件
```javascript
// 创建个人事件
const response = await uni.request({
  url: '/calendar/',
  method: 'POST',
  header: {
    'Authorization': 'Bearer ' + token,
    'Content-Type': 'application/json'
  },
  data: {
    title: '个人会议',
    date: '2024-03-15',
    description: '重要会议',
    color: '#4ECDC4'
  }
});

if (response.data.success) {
  uni.showToast({
    title: '事件创建成功',
    icon: 'success'
  });
}
```

### 2. 获取事件列表
```javascript
// 获取本月事件
const response = await uni.request({
  url: '/calendar/?start_date=2024-03-01&end_date=2024-03-31&type=all',
  method: 'GET',
  header: {
    'Authorization': 'Bearer ' + token
  }
});

if (response.data.success) {
  this.events = response.data.data;
}
```

### 3. 错误处理
```javascript
try {
  const response = await uni.request({
    url: '/calendar/',
    method: 'POST',
    header: {
      'Authorization': 'Bearer ' + token,
      'Content-Type': 'application/json'
    },
    data: eventData
  });
  
  if (response.data.success) {
    // 处理成功
  }
} catch (error) {
  uni.showToast({
    title: error.data.error || '操作失败',
    icon: 'error'
  });
}
```

## 权限说明

### 事件权限控制
- **个人事件**: 创建者可以查看、修改、删除
- **家庭事件**: 同一家庭的所有成员可以查看，但只有创建者可以修改、删除

### 重复事件处理
- 支持四种重复类型：不重复、每周重复、每月重复、每年重复
- 重复事件可以设置结束日期
- 重复规则支持自定义配置