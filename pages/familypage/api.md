# 饮食管理API接口文档

## 概述

饮食管理API提供饮食记录的创建、查询、更新、删除等功能，支持营养数据统计和每日饮食总结。所有接口都需要JWT认证。

**基础URL**: `http://your-domain.com/diets`

**注意**: 根据蓝图注册配置，路由前缀为 `/diets`

## 接口列表

### 1. 获取饮食记录列表

**接口说明**: 获取当前用户的饮食记录，支持日期和用餐时间筛选

**URL**: `GET /record_get_all/`

**请求头**:
```
Authorization: Bearer <jwt_token>
```

**查询参数**:
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| date | string | 否 | 特定日期 | YYYY-MM-DD格式 |
| meal_time | string | 否 | 用餐时间 | breakfast, lunch, dinner等 |
| days | int | 否 | 最近天数 | 默认7天 |

**请求示例**:
```
GET /diets/record_get_all/?date=2024-03-15&meal_time=lunch
```

**响应示例**:
```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "user_id": 123,
      "date": "2024-03-15",
      "meal_time": "lunch",
      "food_name": "米饭",
      "weight": 200.0,
      "protein": 5.0,
      "carbohydrates": 50.0,
      "fat": 1.0,
      "calories": 230.0,
      "created_at": "2024-03-15T12:30:00",
      "updated_at": "2024-03-15T12:30:00"
    }
  ],
  "total": 1
}
```

**状态码说明**:
- `200`: 获取成功
- `400`: 日期格式错误
- `500`: 服务器内部错误

### 2. 获取单个饮食记录

**接口说明**: 获取特定饮食记录的详细信息

**URL**: `GET /record_get/<record_id>`

**请求头**:
```
Authorization: Bearer <jwt_token>
```

**请求示例**:
```
GET /diets/record_get/1
```

**响应示例**:
```json
{
  "success": true,
  "data": {
    "id": 1,
    "user_id": 123,
    "date": "2024-03-15",
    "meal_time": "lunch",
    "food_name": "米饭",
    "weight": 200.0,
    "protein": 5.0,
    "carbohydrates": 50.0,
    "fat": 1.0,
    "calories": 230.0,
    "created_at": "2024-03-15T12:30:00",
    "updated_at": "2024-03-15T12:30:00"
  }
}
```

**状态码说明**:
- `200`: 获取成功
- `404`: 记录不存在或无权访问
- `500`: 服务器内部错误

### 3. 创建饮食记录

**接口说明**: 创建新的饮食记录

**URL**: `POST /record_create/`

**请求头**:
```
Authorization: Bearer <jwt_token>
Content-Type: application/json
```

**请求体**:
| 字段名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| date | string | 是 | 记录日期 | YYYY-MM-DD格式 |
| meal_time | string | 是 | 用餐时间 | breakfast, lunch, dinner等 |
| food_name | string | 是 | 食物名称 | - |
| weight | float | 是 | 食物重量 | 单位：克 |
| protein | float | 否 | 蛋白质含量 | 单位：克 |
| carbohydrates | float | 否 | 碳水化合物含量 | 单位：克 |
| fat | float | 否 | 脂肪含量 | 单位：克 |
| calories | float | 否 | 热量 | 单位：千卡 |

**请求示例**:
```json
{
  "date": "2024-03-15",
  "meal_time": "lunch",
  "food_name": "米饭",
  "weight": 200,
  "protein": 5,
  "carbohydrates": 50,
  "fat": 1
}
```

**响应示例**:
```json
{
  "success": true,
  "message": "饮食记录创建成功",
  "data": {
    "id": 1,
    "user_id": 123,
    "date": "2024-03-15",
    "meal_time": "lunch",
    "food_name": "米饭",
    "weight": 200.0,
    "protein": 5.0,
    "carbohydrates": 50.0,
    "fat": 1.0,
    "calories": 230.0,
    "created_at": "2024-03-15T12:30:00",
    "updated_at": "2024-03-15T12:30:00"
  }
}
```

**状态码说明**:
- `201`: 创建成功
- `400`: 缺少必需字段、日期格式错误
- `500`: 服务器内部错误

### 4. 更新饮食记录

**接口说明**: 更新饮食记录信息

**URL**: `PUT /record_update/<record_id>`

**请求头**:
```
Authorization: Bearer <jwt_token>
Content-Type: application/json
```

**请求体**:
| 字段名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| date | string | 否 | 记录日期 | YYYY-MM-DD格式 |
| meal_time | string | 否 | 用餐时间 | - |
| food_name | string | 否 | 食物名称 | - |
| weight | float | 否 | 食物重量 | - |
| protein | float | 否 | 蛋白质含量 | - |
| carbohydrates | float | 否 | 碳水化合物含量 | - |
| fat | float | 否 | 脂肪含量 | - |
| calories | float | 否 | 热量 | - |

**请求示例**:
```json
{
  "food_name": "白米饭",
  "weight": 250,
  "protein": 6
}
```

**响应示例**:
```json
{
  "success": true,
  "message": "饮食记录更新成功",
  "data": {
    "id": 1,
    "user_id": 123,
    "date": "2024-03-15",
    "meal_time": "lunch",
    "food_name": "白米饭",
    "weight": 250.0,
    "protein": 6.0,
    "carbohydrates": 50.0,
    "fat": 1.0,
    "calories": 280.0,
    "created_at": "2024-03-15T12:30:00",
    "updated_at": "2024-03-15T13:00:00"
  }
}
```

**状态码说明**:
- `200`: 更新成功
- `400`: 日期格式错误
- `404`: 记录不存在或无权修改
- `500`: 服务器内部错误

### 5. 删除饮食记录

**接口说明**: 删除饮食记录

**URL**: `DELETE /record_delete/<record_id>`

**请求头**:
```
Authorization: Bearer <jwt_token>
```

**请求示例**:
```
DELETE /diets/record_delete/1
```

**响应示例**:
```json
{
  "success": true,
  "message": "饮食记录删除成功"
}
```

**状态码说明**:
- `200`: 删除成功
- `404`: 记录不存在或无权删除
- `500`: 服务器内部错误

### 6. 获取当日饮食总结

**接口说明**: 获取指定日期的饮食营养汇总信息

**URL**: `GET /daily_summary`

**请求头**:
```
Authorization: Bearer <jwt_token>
```

**查询参数**:
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| date | string | 否 | 目标日期 | YYYY-MM-DD格式，默认今天 |

**请求示例**:
```
GET /diets/daily_summary?date=2024-03-15
```

**响应示例**:
```json
{
  "success": true,
  "data": {
    "date": "2024-03-15",
    "total_calories": 1850.0,
    "total_protein": 65.0,
    "total_carbohydrates": 250.0,
    "total_fat": 45.0,
    "record_count": 3,
    "meal_types": ["breakfast", "lunch", "dinner"],
    "recommended_calories": 2000,
    "calories_percentage": 92.5
  }
}
```

**状态码说明**:
- `200`: 获取成功
- `400`: 日期格式错误
- `500`: 服务器内部错误

### 7. 获取饮食统计信息

**接口说明**: 获取指定时间范围内的饮食统计信息

**URL**: `GET /statistics`

**请求头**:
```
Authorization: Bearer <jwt_token>
```

**查询参数**:
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| days | int | 否 | 统计天数 | 默认30天 |

**请求示例**:
```
GET /diets/statistics?days=60
```

**响应示例**:
```json
{
  "success": true,
  "data": {
    "period": "最近30天",
    "total_records": 45,
    "average_calories_per_day": 1850.5,
    "most_common_foods": [
      ["米饭", 15],
      ["鸡蛋", 12],
      ["鸡肉", 8]
    ],
    "meal_distribution": {
      "breakfast": 15,
      "lunch": 20,
      "dinner": 10
    },
    "nutrition_trends": [
      ["2024-03-15", 1850],
      ["2024-03-16", 1920],
      ["2024-03-17", 1780]
    ]
  }
}
```

**状态码说明**:
- `200`: 获取成功
- `500`: 服务器内部错误

### 8. 获取用餐时间列表

**接口说明**: 获取支持的用餐时间选项

**URL**: `GET /meal_times`

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
      "value": "breakfast",
      "label": "早餐"
    },
    {
      "value": "morning_snack",
      "label": "早间加餐"
    },
    {
      "value": "lunch",
      "label": "午餐"
    },
    {
      "value": "afternoon_snack",
      "label": "下午加餐"
    },
    {
      "value": "dinner",
      "label": "晚餐"
    },
    {
      "value": "supper",
      "label": "夜宵"
    },
    {
      "value": "other",
      "label": "其他"
    }
  ]
}
```

**状态码说明**:
- `200`: 获取成功
- `500`: 服务器内部错误

### 9. 获取AI饮食建议

**接口说明**: 基于本地Ollama获取个性化的饮食建议，根据用户选择的时段分析饮食记录

**URL**: `GET /ai_suggestion`

**请求头**:
```
Authorization: Bearer <jwt_token>
```

**查询参数**:
| 参数名 | 类型 | 必填 | 说明 | 可选值 |
|--------|------|------|------|--------|
| period | string | 否 | 分析时间段 | `today`(今天), `3days`(最近三天), `week`(最近一周) |

**请求示例**:
```
GET /diets/ai_suggestion?period=week
```

**响应示例（有饮食记录时）**:
```json
{
  "success": true,
  "data": {
    "period": "最近一周",
    "suggestion": "根据您最近一周的饮食记录分析，您的营养摄入情况总体良好。总热量摄入为12500.5千卡，平均每天1785.8千卡，处于合理范围。蛋白质摄入455.2克，比例适当。建议增加蔬菜水果摄入，提高食物多样性。",
    "summary": {
      "total_calories": 12500.5,
      "total_protein": 455.2,
      "total_carbohydrates": 1750.8,
      "total_fat": 350.3,
      "record_count": 21,
      "most_common_foods": [
        ["米饭", 7],
        ["鸡蛋", 5],
        ["鸡肉", 4],
        ["苹果", 3],
        ["牛奶", 2]
      ]
    },
    "record_count": 21
  }
}
```

**响应示例（无饮食记录时）**:
```json
{
  "success": true,
  "data": {
    "period": "最近一周",
    "suggestion": "最近一周没有找到饮食记录，请先记录您的饮食情况。",
    "summary": {},
    "record_count": 0
  }
}
```

**响应示例（Ollama服务不可用时）**:
```json
{
  "success": true,
  "data": {
    "period": "最近一周",
    "suggestion": "由于AI服务暂时无法使用，以下是基于最近一周饮食数据的分析：\n\n📊 营养概况：\n- 总热量：12500.5千卡\n- 蛋白质：455.2克\n- 碳水：1750.8克\n- 脂肪：350.3克\n\n💡 一般建议：\n1. 保持食物多样性，多吃蔬菜水果\n2. 注意控制总热量摄入\n3. 确保蛋白质摄入充足\n4. 合理安排三餐时间\n\n建议您咨询专业营养师获取更详细的个性化建议。",
    "summary": {
      "total_calories": 12500.5,
      "total_protein": 455.2,
      "total_carbohydrates": 1750.8,
      "total_fat": 350.3,
      "record_count": 21,
      "most_common_foods": [
        ["米饭", 7],
        ["鸡蛋", 5],
        ["鸡肉", 4]
      ]
    },
    "record_count": 21
  }
}
```

**状态码说明**:
- `200`: 获取成功
- `400`: 无效的时间段参数
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
  - 数值格式错误

- **401 Unauthorized**: 未授权
  - JWT token无效或过期

- **404 Not Found**: 资源不存在
  - 记录不存在
  - 用户无权访问该记录

- **500 Internal Server Error**: 服务器内部错误
  - 数据库操作失败
  - 未知的服务器错误

## 前端使用示例

### 1. 创建饮食记录
```javascript
// 创建午餐记录
const response = await uni.request({
  url: '/diets/record_create/',
  method: 'POST',
  header: {
    'Authorization': 'Bearer ' + token,
    'Content-Type': 'application/json'
  },
  data: {
    date: '2024-03-15',
    meal_time: 'lunch',
    food_name: '米饭',
    weight: 200,
    protein: 5,
    carbohydrates: 50,
    fat: 1
  }
});

if (response.data.success) {
  uni.showToast({
    title: '记录创建成功',
    icon: 'success'
  });
}
```

### 2. 获取今日饮食记录
```javascript
// 获取今日所有饮食记录
const response = await uni.request({
  url: '/diets/record_get_all/?date=2024-03-15',
  method: 'GET',
  header: {
    'Authorization': 'Bearer ' + token
  }
});

if (response.data.success) {
  this.dietRecords = response.data.data;
  this.totalRecords = response.data.total;
}
```

### 3. 获取饮食统计
```javascript
// 获取最近30天的饮食统计
const response = await uni.request({
  url: '/diets/statistics?days=30',
  method: 'GET',
  header: {
    'Authorization': 'Bearer ' + token
  }
});

if (response.data.success) {
  this.statistics = response.data.data;
  this.updateChartData();
}
```

### 4. 错误处理
```javascript
try {
  const response = await uni.request({
    url: '/diets/record_create/',
    method: 'POST',
    header: {
      'Authorization': 'Bearer ' + token,
      'Content-Type': 'application/json'
    },
    data: dietData
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

## 营养计算说明

### 热量自动计算
如果创建记录时未提供热量值，系统会根据营养数据自动计算：
- 蛋白质：4千卡/克
- 碳水化合物：4千卡/克  
- 脂肪：9千卡/克

**计算公式**: `热量 = 蛋白质×4 + 碳水化合物×4 + 脂肪×9`

### 推荐摄入量
每日饮食总结中的推荐热量值（2000千卡）为示例值，实际应用中可根据用户信息调整：
- 性别、年龄、体重、身高
- 活动水平
- 健康目标（减重、增肌、维持）

## 数据权限说明

### 记录权限控制
- **个人记录**: 用户只能查看和操作自己的饮食记录
- **数据隔离**: 不同用户之间的饮食记录完全隔离
- **权限验证**: 所有操作都会验证当前用户权限

### 数据安全
- 所有营养数据都经过数值验证
- 日期格式严格校验
- 用户输入数据会进行适当的清理和验证