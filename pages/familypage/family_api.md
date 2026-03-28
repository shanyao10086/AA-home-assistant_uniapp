# 家庭管理API接口文档

## 概述

家庭管理API提供家庭创建、成员管理等功能，支持家庭的创建、成员添加、成员删除和成员列表查询。所有接口都需要JWT认证。

**基础URL**: `http://your-domain.com/families`

**注意**: 根据蓝图注册配置，路由前缀为 `/families`

## 接口列表

### 1. 创建家庭

**接口说明**: 创建新的家庭，创建者自动成为家庭管理员

**URL**: `POST /create`

**请求头**:
```
Authorization: Bearer <jwt_token>
Content-Type: application/json
```

**请求体**:
| 字段名 | 类型 | 必填 | 说明 | 约束 |
|--------|------|------|------|------|
| name | string | 是 | 家庭名称 | 1-64字符 |
| description | string | 否 | 家庭描述 | - |

**请求示例**:
```json
{
  "name": "幸福之家",
  "description": "我的温馨家庭"
}
```

**响应示例**:
```json
{
  "success": true,
  "message": "家庭创建成功",
  "data": {
    "id": 1,
    "name": "幸福之家",
    "description": "我的温馨家庭",
    "family_admin_id": 123,
    "family_admin_name": "张三",
    "family_member_num": 1,
    "family_members": ["123"],
    "member_details": [
      {
        "id": 123,
        "nickname": "张三",
        "avatar_url": "https://wx.qlogo.cn/mmopen/vi_32/xxx",
        "role": "admin",
        "family_id": 1,
        "family_name": "幸福之家",
        "is_family_admin": true,
        "created_at": "2024-03-11T10:30:00",
        "last_login_at": "2024-03-11T10:30:00"
      }
    ],
    "created_at": "2024-03-11T10:30:00",
    "updated_at": "2024-03-11T10:30:00"
  }
}
```

**前端使用示例**:
```javascript
// 创建家庭
const response = await uni.request({
  url: '/families/create',
  method: 'POST',
  header: {
    'Authorization': 'Bearer ' + token,
    'Content-Type': 'application/json'
  },
  data: {
    name: '幸福之家',
    description: '我的温馨家庭'
  }
});

if (response.data.success) {
  uni.showToast({
    title: '家庭创建成功',
    icon: 'success'
  });
  // 跳转到家庭管理页面
  uni.navigateTo({
    url: '/pages/family/index'
  });
}
```

**错误情况**:
- `400`: 用户已经创建了家庭或已经加入了其他家庭
- `400`: 家庭名称为空或超过64字符
- `404`: 用户不存在

### 2. 获取家庭成员列表

**接口说明**: 获取当前用户所属家庭的所有成员列表

**URL**: `GET /members`

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
      "id": 123,
      "nickname": "张三",
      "avatar_url": "https://wx.qlogo.cn/mmopen/vi_32/xxx",
      "role": "admin",
      "family_id": 1,
      "family_name": "幸福之家",
      "is_family_admin": true,
      "created_at": "2024-03-11T10:30:00",
      "last_login_at": "2024-03-11T10:30:00"
    },
    {
      "id": 124,
      "nickname": "李四",
      "avatar_url": "https://wx.qlogo.cn/mmopen/vi_32/yyy",
      "role": "member",
      "family_id": 1,
      "family_name": "幸福之家",
      "is_family_admin": false,
      "created_at": "2024-03-10T09:00:00",
      "last_login_at": "2024-03-10T09:00:00"
    }
  ],
  "total": 2
}
```

**前端使用示例**:
```javascript
// 获取家庭成员列表
const response = await uni.request({
  url: '/families/members',
  method: 'GET',
  header: {
    'Authorization': 'Bearer ' + token
  }
});

if (response.data.success) {
  this.familyMembers = response.data.data;
  this.memberCount = response.data.total;
}
```

**特殊处理**:
- 如果用户没有加入任何家庭，返回空数组和提示信息

### 3. 添加家庭成员

**接口说明**: 添加新的家庭成员（仅家庭管理员可操作）

**URL**: `POST /add_member`

**请求头**:
```
Authorization: Bearer <jwt_token>
Content-Type: application/json
```

**请求体**:
| 字段名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| user_openid | string | 是 | 目标用户的微信OpenID |

**请求示例**:
```json
{
  "user_openid": "o1234567890abcdefghijklmnopqrstuvwxyz"
}
```

**响应示例**:
```json
{
  "success": true,
  "message": "成员添加成功",
  "data": {
    "family": {
      "id": 1,
      "name": "幸福之家",
      "description": "我的温馨家庭",
      "family_admin_id": 123,
      "family_admin_name": "张三",
      "family_member_num": 2,
      "family_members": ["123", "124"],
      "member_details": [
        {
          "id": 123,
          "nickname": "张三",
          "avatar_url": "https://wx.qlogo.cn/mmopen/vi_32/xxx",
          "role": "admin",
          "family_id": 1,
          "family_name": "幸福之家",
          "is_family_admin": true,
          "created_at": "2024-03-11T10:30:00",
          "last_login_at": "2024-03-11T10:30:00"
        },
        {
          "id": 124,
          "nickname": "李四",
          "avatar_url": "https://wx.qlogo.cn/mmopen/vi_32/yyy",
          "role": "member",
          "family_id": 1,
          "family_name": "幸福之家",
          "is_family_admin": false,
          "created_at": "2024-03-10T09:00:00",
          "last_login_at": "2024-03-10T09:00:00"
        }
      ],
      "created_at": "2024-03-11T10:30:00",
      "updated_at": "2024-03-11T11:00:00"
    },
    "added_member": {
      "id": 124,
      "nickname": "李四",
      "avatar_url": "https://wx.qlogo.cn/mmopen/vi_32/yyy",
      "role": "member",
      "family_id": 1,
      "family_name": "幸福之家",
      "is_family_admin": false,
      "created_at": "2024-03-10T09:00:00",
      "last_login_at": "2024-03-10T09:00:00"
    }
  }
}
```

**前端使用示例**:
```javascript
// 添加家庭成员
const response = await uni.request({
  url: '/families/add_member',
  method: 'POST',
  header: {
    'Authorization': 'Bearer ' + token,
    'Content-Type': 'application/json'
  },
  data: {
    user_openid: 'o1234567890abcdefghijklmnopqrstuvwxyz'
  }
});

if (response.data.success) {
  uni.showToast({
    title: '成员添加成功',
    icon: 'success'
  });
  // 刷新家庭成员列表
  this.loadFamilyMembers();
}
```

**错误情况**:
- `400`: 目标用户已经是家庭成员或已加入其他家庭
- `400`: 家庭成员已满（最多10人）
- `403`: 当前用户不是家庭管理员
- `404`: 目标用户不存在

### 4. 删除家庭成员

**接口说明**: 删除家庭成员（仅家庭管理员可操作）

**URL**: `POST /remove_member`

**请求头**:
```
Authorization: Bearer <jwt_token>
Content-Type: application/json
```

**请求体**:
| 字段名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| user_id | int | 是 | 目标用户的ID |

**请求示例**:
```json
{
  "user_id": 124
}
```

**响应示例**:
```json
{
  "success": true,
  "message": "成员删除成功",
  "data": {
    "family": {
      "id": 1,
      "name": "幸福之家",
      "description": "我的温馨家庭",
      "family_admin_id": 123,
      "family_admin_name": "张三",
      "family_member_num": 1,
      "family_members": ["123"],
      "member_details": [
        {
          "id": 123,
          "nickname": "张三",
          "avatar_url": "https://wx.qlogo.cn/mmopen/vi_32/xxx",
          "role": "admin",
          "family_id": 1,
          "family_name": "幸福之家",
          "is_family_admin": true,
          "created_at": "2024-03-11T10:30:00",
          "last_login_at": "2024-03-11T10:30:00"
        }
      ],
      "created_at": "2024-03-11T10:30:00",
      "updated_at": "2024-03-11T11:30:00"
    },
    "removed_member_id": 124
  }
}
```

**前端使用示例**:
```javascript
// 删除家庭成员
const response = await uni.request({
  url: '/families/remove_member',
  method: 'POST',
  header: {
    'Authorization': 'Bearer ' + token,
    'Content-Type': 'application/json'
  },
  data: {
    user_id: 124
  }
});

if (response.data.success) {
  uni.showToast({
    title: '成员删除成功',
    icon: 'success'
  });
  // 刷新家庭成员列表
  this.loadFamilyMembers();
}
```

**错误情况**:
- `400`: 目标用户不是家庭成员
- `400`: 不能删除家庭管理员
- `403`: 当前用户不是家庭管理员
- `404`: 目标用户不存在

## 错误处理

所有接口都使用统一的错误响应格式：

```json
{
  "error": "错误描述信息"
}
```

**常见错误码**:
- `400`: 请求参数错误（字段缺失、格式错误等）
- `401`: 未授权（JWT无效）
- `403`: 权限不足（非管理员操作）
- `404`: 资源不存在（用户、家庭不存在）
- `500`: 服务器内部错误

## 权限说明

### 用户角色
- **admin**: 家庭管理员，拥有所有权限
- **member**: 普通成员，只能查看家庭信息

### 权限矩阵
| 操作 | admin | member |
|------|-------|--------|
| 创建家庭 | ✅ | ✅（未加入家庭时） |
| 查看成员列表 | ✅ | ✅ |
| 添加成员 | ✅ | ❌ |
| 删除成员 | ✅ | ❌ |

## 前端集成建议

### 1. 封装API请求
```javascript
class FamilyAPI {
  constructor(token) {
    this.baseURL = '/families';
    this.token = token;
  }
  
  async request(url, options = {}) {
    const response = await uni.request({
      url: this.baseURL + url,
      header: {
        'Authorization': 'Bearer ' + this.token,
        'Content-Type': 'application/json',
        ...options.header
      },
      ...options
    });
    
    if (response.data.error) {
      throw new Error(response.data.error);
    }
    
    return response.data;
  }
  
  // 创建家庭
  async createFamily(data) {
    return this.request('/create', {
      method: 'POST',
      data
    });
  }
  
  // 获取成员列表
  async getMembers() {
    return this.request('/members', {
      method: 'GET'
    });
  }
  
  // 添加成员
  async addMember(userOpenid) {
    return this.request('/add_member', {
      method: 'POST',
      data: { user_openid: userOpenid }
    });
  }
  
  // 删除成员
  async removeMember(userId) {
    return this.request('/remove_member', {
      method: 'POST',
      data: { user_id: userId }
    });
  }
}
```

### 2. 错误处理
```javascript
// 统一错误处理
try {
  const result = await familyAPI.createFamily({
    name: '幸福之家',
    description: '我的温馨家庭'
  });
  // 处理成功结果
} catch (error) {
  uni.showToast({
    title: error.message,
    icon: 'error'
  });
}
```

### 3. 权限检查
```javascript
// 检查用户权限
function checkAdminPermission(user) {
  if (!user.is_family_admin) {
    uni.showToast({
      title: '只有家庭管理员可以执行此操作',
      icon: 'error'
    });
    return false;
  }
  return true;
}

// 使用权限检查
if (checkAdminPermission(this.currentUser)) {
  // 执行管理员操作
}
```

### 4. 数据同步
```javascript
// 监听家庭成员变化
watch: {
  familyMembers: {
    handler(newMembers) {
      this.updateMemberCount(newMembers.length);
      this.updateAdminStatus(newMembers);
    },
    deep: true
  }
},

methods: {
  // 更新成员数量
  updateMemberCount(count) {
    this.memberCount = count;
  },
  
  // 更新管理员状态
  updateAdminStatus(members) {
    const currentUser = this.currentUser;
    const isAdmin = members.some(member => 
      member.id === currentUser.id && member.is_family_admin
    );
    this.isCurrentUserAdmin = isAdmin;
  }
}
```

## 使用流程

### 1. 创建家庭流程
```javascript
// 1. 检查用户是否已加入家庭
const userInfo = await getUserInfo();
if (userInfo.family_id) {
  // 用户已加入家庭，跳转到家庭管理页面
  uni.navigateTo({
    url: '/pages/family/manage'
  });
  return;
}

// 2. 创建家庭
const result = await familyAPI.createFamily({
  name: '我的家庭',
  description: '温馨的家庭'
});

// 3. 创建成功后更新用户信息
await updateUserInfo();
```

### 2. 添加成员流程
```javascript
// 1. 扫描二维码获取用户OpenID
const scanResult = await uni.scanCode();
const userOpenid = scanResult.result;

// 2. 添加成员
try {
  const result = await familyAPI.addMember(userOpenid);
  uni.showToast({
    title: '添加成功',
    icon: 'success'
  });
} catch (error) {
  uni.showToast({
    title: error.message,
    icon: 'error'
  });
}
```

### 3. 管理成员流程
```javascript
// 1. 获取成员列表
const members = await familyAPI.getMembers();

// 2. 显示成员列表
this.familyMembers = members.data;

// 3. 删除成员（长按操作）
async onMemberLongPress(member) {
  if (!this.isCurrentUserAdmin) {
    uni.showToast({
      title: '只有管理员可以删除成员',
      icon: 'error'
    });
    return;
  }
  
  uni.showModal({
    title: '确认删除',
    content: `确定要删除成员 ${member.nickname} 吗？`,
    success: async (res) => {
      if (res.confirm) {
        try {
          await familyAPI.removeMember(member.id);
          uni.showToast({
            title: '删除成功',
            icon: 'success'
          });
        } catch (error) {
          uni.showToast({
            title: error.message,
            icon: 'error'
          });
        }
      }
    }
  });
}
```