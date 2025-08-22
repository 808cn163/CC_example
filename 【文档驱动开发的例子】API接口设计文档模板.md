# 用户认证模块 API接口设计文档

## 执行摘要

本文档定义了用户认证模块的RESTful API接口规范，包括接口定义、请求响应格式、错误处理机制和集成示例。为前端和第三方系统提供统一的接口标准和集成指南。

## 1. API设计原则

### 1.1 设计规范
- **RESTful风格**：遵循REST架构规范，使用标准HTTP方法
- **统一格式**：所有接口采用统一的请求响应格式
- **版本控制**：通过URL路径进行版本管理
- **状态码规范**：合理使用HTTP状态码表示请求结果

### 1.2 命名约定
- **URL命名**：使用小写字母和连字符，名词复数形式
- **字段命名**：JSON字段使用蛇形命名法（snake_case）
- **枚举值**：使用大写字母和下划线

## 2. 通用规范

### 2.1 请求格式
**Base URL**: `https://api.example.com/v1`

**请求头**:
```http
Content-Type: application/json
Authorization: Bearer {access_token}
X-Request-ID: {uuid}
X-Client-Version: {version}
```

### 2.2 响应格式
**成功响应**:
```json
{
  "code": 200,
  "message": "success",
  "data": {},
  "timestamp": 1642694400,
  "request_id": "uuid"
}
```

**错误响应**:
```json
{
  "code": 40001,
  "message": "参数验证失败",
  "error": "INVALID_PARAMS",
  "details": {
    "field": "username",
    "reason": "用户名不能为空"
  },
  "timestamp": 1642694400,
  "request_id": "uuid"
}
```

### 2.3 状态码规范
| HTTP状态码 | 业务码 | 说明 | 场景 |
|-----------|-------|------|------|
| 200 | 20000 | 请求成功 | 正常业务响应 |
| 400 | 40001 | 参数错误 | 请求参数验证失败 |
| 401 | 40101 | 认证失败 | Token无效或过期 |
| 403 | 40301 | 权限不足 | 无访问权限 |
| 423 | 42301 | 账户锁定 | 登录失败次数过多 |
| 429 | 42901 | 请求过频 | 触发限流 |
| 500 | 50001 | 服务器错误 | 系统内部错误 |

## 3. 接口定义

### 3.1 用户登录
**接口路径**: `POST /auth/login`

**请求参数**:
```json
{
  "username": "john_doe",
  "password": "password123",
  "login_type": "password",
  "client_info": {
    "device_id": "device123",
    "ip_address": "192.168.1.100",
    "user_agent": "Mozilla/5.0..."
  }
}
```

**响应数据**:
```json
{
  "code": 20000,
  "message": "登录成功",
  "data": {
    "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refresh_token": "refresh_token_string",
    "expires_in": 3600,
    "token_type": "Bearer",
    "user_info": {
      "id": 12345,
      "username": "john_doe",
      "email": "john@example.com",
      "roles": ["user", "admin"],
      "permissions": ["user:read", "user:write"]
    }
  },
  "timestamp": 1642694400
}
```

**错误场景**:
```json
// 用户名或密码错误
{
  "code": 40102,
  "message": "用户名或密码错误",
  "error": "INVALID_CREDENTIALS"
}

// 账户被锁定
{
  "code": 42301,
  "message": "账户已锁定，请15分钟后重试",
  "error": "ACCOUNT_LOCKED",
  "details": {
    "lockout_duration": 900,
    "remaining_time": 850
  }
}
```

### 3.2 令牌验证
**接口路径**: `GET /auth/validate`

**请求头**:
```http
Authorization: Bearer {access_token}
```

**响应数据**:
```json
{
  "code": 20000,
  "message": "令牌有效",
  "data": {
    "user_id": 12345,
    "username": "john_doe",
    "roles": ["user", "admin"],
    "expires_at": 1642698000,
    "remaining_time": 3600
  }
}
```

### 3.3 权限检查
**接口路径**: `POST /auth/permissions/check`

**请求参数**:
```json
{
  "resource": "user_management",
  "action": "read",
  "context": {
    "department_id": 1001,
    "project_id": 2001
  }
}
```

**响应数据**:
```json
{
  "code": 20000,
  "message": "权限检查完成",
  "data": {
    "granted": true,
    "permission_source": "role",
    "matched_permissions": [
      {
        "resource": "user_management",
        "action": "read",
        "scope": "department"
      }
    ]
  }
}
```

### 3.4 刷新令牌
**接口路径**: `POST /auth/refresh`

**请求参数**:
```json
{
  "refresh_token": "refresh_token_string"
}
```

**响应数据**:
```json
{
  "code": 20000,
  "message": "令牌刷新成功", 
  "data": {
    "access_token": "new_access_token",
    "refresh_token": "new_refresh_token",
    "expires_in": 3600,
    "token_type": "Bearer"
  }
}
```

### 3.5 用户登出
**接口路径**: `POST /auth/logout`

**请求头**:
```http
Authorization: Bearer {access_token}
```

**响应数据**:
```json
{
  "code": 20000,
  "message": "登出成功",
  "data": {
    "logout_time": 1642694400
  }
}
```

### 3.6 获取用户权限
**接口路径**: `GET /auth/permissions`

**响应数据**:
```json
{
  "code": 20000,
  "message": "获取权限成功",
  "data": {
    "user_id": 12345,
    "permissions": [
      {
        "resource": "user_management",
        "actions": ["read", "write"],
        "source": "role",
        "role_name": "admin"
      },
      {
        "resource": "report",
        "actions": ["read"],
        "source": "direct",
        "expires_at": 1672531200
      }
    ],
    "roles": [
      {
        "id": 1,
        "name": "admin",
        "description": "系统管理员"
      }
    ]
  }
}
```

## 4. 错误码定义

### 4.1 认证类错误
| 错误码 | 错误标识 | 描述 | HTTP状态码 |
|-------|---------|------|-----------|
| 40101 | INVALID_TOKEN | 无效的访问令牌 | 401 |
| 40102 | INVALID_CREDENTIALS | 用户名或密码错误 | 401 |
| 40103 | TOKEN_EXPIRED | 访问令牌已过期 | 401 |
| 40104 | INVALID_REFRESH_TOKEN | 无效的刷新令牌 | 401 |
| 40105 | REFRESH_TOKEN_USED | 刷新令牌已使用 | 401 |

### 4.2 权限类错误  
| 错误码 | 错误标识 | 描述 | HTTP状态码 |
|-------|---------|------|-----------|
| 40301 | PERMISSION_DENIED | 权限不足 | 403 |
| 40302 | RESOURCE_ACCESS_DENIED | 资源访问被拒绝 | 403 |
| 40303 | ROLE_REQUIRED | 需要特定角色 | 403 |

### 4.3 业务类错误
| 错误码 | 错误标识 | 描述 | HTTP状态码 |
|-------|---------|------|-----------|
| 42301 | ACCOUNT_LOCKED | 账户已锁定 | 423 |
| 42302 | ACCOUNT_DISABLED | 账户已禁用 | 423 |
| 42901 | RATE_LIMIT_EXCEEDED | 请求频率超限 | 429 |

## 5. 客户端集成示例

### 5.1 JavaScript/TypeScript
```typescript
class AuthClient {
  private baseURL = 'https://api.example.com/v1';
  private accessToken: string | null = null;

  async login(username: string, password: string): Promise<LoginResponse> {
    const response = await fetch(`${this.baseURL}/auth/login`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        username,
        password,
        login_type: 'password'
      })
    });

    const result = await response.json();
    if (result.code === 20000) {
      this.accessToken = result.data.access_token;
      return result.data;
    }
    throw new AuthError(result.code, result.message);
  }

  async validateToken(): Promise<boolean> {
    if (!this.accessToken) return false;
    
    const response = await fetch(`${this.baseURL}/auth/validate`, {
      headers: {
        'Authorization': `Bearer ${this.accessToken}`
      }
    });

    const result = await response.json();
    return result.code === 20000;
  }
}
```

### 5.2 Python
```python
import requests
from typing import Optional, Dict, Any

class AuthClient:
    def __init__(self, base_url: str = "https://api.example.com/v1"):
        self.base_url = base_url
        self.access_token: Optional[str] = None

    def login(self, username: str, password: str) -> Dict[str, Any]:
        response = requests.post(
            f"{self.base_url}/auth/login",
            json={
                "username": username,
                "password": password,
                "login_type": "password"
            }
        )
        
        result = response.json()
        if result["code"] == 20000:
            self.access_token = result["data"]["access_token"]
            return result["data"]
        
        raise AuthError(result["code"], result["message"])

    def check_permission(self, resource: str, action: str) -> bool:
        if not self.access_token:
            return False
            
        response = requests.post(
            f"{self.base_url}/auth/permissions/check",
            headers={"Authorization": f"Bearer {self.access_token}"},
            json={"resource": resource, "action": action}
        )
        
        result = response.json()
        return result["code"] == 20000 and result["data"]["granted"]
```

### 5.3 Java
```java
@Component
public class AuthClient {
    private final RestTemplate restTemplate;
    private final String baseUrl = "https://api.example.com/v1";
    private String accessToken;

    public LoginResponse login(String username, String password) {
        LoginRequest request = LoginRequest.builder()
            .username(username)
            .password(password)
            .loginType("password")
            .build();

        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_JSON);
        HttpEntity<LoginRequest> entity = new HttpEntity<>(request, headers);

        ResponseEntity<ApiResponse<LoginData>> response = restTemplate.exchange(
            baseUrl + "/auth/login",
            HttpMethod.POST,
            entity,
            new ParameterizedTypeReference<ApiResponse<LoginData>>() {}
        );

        if (response.getBody().getCode() == 20000) {
            this.accessToken = response.getBody().getData().getAccessToken();
            return response.getBody().getData();
        }
        
        throw new AuthException(response.getBody().getCode(), response.getBody().getMessage());
    }
}
```

## 6. 调试和测试

### 6.1 Postman集合
```json
{
  "info": {
    "name": "用户认证API",
    "schema": "https://schema.getpostman.com/json/collection/v2.1.0/collection.json"
  },
  "item": [
    {
      "name": "用户登录",
      "request": {
        "method": "POST",
        "header": [
          {
            "key": "Content-Type",
            "value": "application/json"
          }
        ],
        "body": {
          "mode": "raw",
          "raw": "{\n  \"username\": \"{{username}}\",\n  \"password\": \"{{password}}\",\n  \"login_type\": \"password\"\n}"
        },
        "url": {
          "raw": "{{base_url}}/auth/login",
          "host": ["{{base_url}}"],
          "path": ["auth", "login"]
        }
      }
    }
  ]
}
```

### 6.2 性能测试脚本
```bash
# 登录接口压测
ab -n 1000 -c 10 -H "Content-Type: application/json" \
   -p login_data.json http://api.example.com/v1/auth/login

# 令牌验证接口压测
ab -n 5000 -c 50 -H "Authorization: Bearer $TOKEN" \
   http://api.example.com/v1/auth/validate
```

## 7. 版本兼容

### 7.1 API版本策略
- **v1**：当前版本，稳定支持
- **v2**：下一版本，向后兼容
- **废弃通知**：提前3个月通知API废弃

### 7.2 兼容性说明
- 新增字段：向后兼容，客户端可忽略
- 字段修改：通过新版本API提供
- 字段删除：提前通知并提供迁移指南

---

**文档版本**: v1.0  
**创建时间**: 2025-08-13  
**最后更新**: 2025-08-13  
**维护人员**: API设计团队