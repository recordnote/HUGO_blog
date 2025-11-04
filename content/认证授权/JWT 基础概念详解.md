---
title: JWT 基础概念详解
date: 2025-08-09T14:21:26+08:00
lastmod: 2025-08-09T14:21:26+08:00
author: Lin
avatar: /me/yy.jpg
cover: /img/jwt.png
images:
  - /img/jwt.png
categories:
  - 认证授权
tags:
  - 认证授权
weight: 1
---

# JWT 基础概念详解

## 什么是 JWT？

JWT（JSON Web Token）是一种基于 Token 的认证授权机制，也是目前最流行的跨域认证解决方案。它是一种规范化后的 JSON 结构 Token，自身包含了身份验证所需要的所有信息。

### 扩展理解

JWT 可以理解为一种"数字身份证"，它包含了持有者的身份信息并且带有防伪标识。与传统的 Session 认证不同，JWT 是自包含的，服务端不需要维护认证状态，这使得 JWT 在分布式系统和微服务架构中具有天然优势。

## JWT 的优势特点

### 核心优势

**无状态设计**

- 服务器不需要存储 Session 信息
- 提高了系统的可用性和伸缩性
- 大大减轻了服务端的压力
- 支持水平扩展，适合云原生架构

**符合现代架构**

- 更符合 RESTful API 的「无状态」原则
- 有效避免 CSRF 攻击（通常存储在 localStorage 中）
- 认证过程不涉及 Cookie，安全性更高
- 支持跨域认证，适合前后端分离项目

### 适用场景

- 分布式系统和微服务架构
- 移动端应用认证
- 单点登录（SSO）系统
- API 接口认证
- 第三方授权登录

## JWT 的三部分结构

JWT 由三部分组成，用点（.）分隔，格式为：`Header.Payload.Signature`

### 1. Header（头部）

包含元数据，定义签名算法和 Token 类型。

```
{
  "alg": "HS256",
  "typ": "JWT",
  "kid": "key-identifier-123"  // 密钥标识符，用于密钥轮换
}
```

**算法类型扩展：**

- HS256/HS384/HS512：对称加密算法，使用同一个密钥进行签名和验证
- RS256/RS384/RS512：非对称加密算法，使用私钥签名，公钥验证
- ES256/ES384/ES512：椭圆曲线数字签名算法

### 2. Payload（载荷）

存放实际需要传递的数据，包含三类声明：

**声明类型详解**

- **注册声明**：预定义的标准声明，建议使用但不强制
- **公有声明**：自定义的公开声明，需要在 IANA 注册避免冲突
- **私有声明**：项目特定的自定义声明，更符合业务需求

**常用注册声明详细说明**

```
{
  "iss": "https://api.example.com",     // 签发者，通常是域名或服务标识
  "sub": "user12345",                   // 主题，通常是用户ID
  "aud": "https://client.example.com",  // 接收方，指定可以使用该Token的客户端
  "exp": 1719690231,                    // 过期时间，Unix时间戳
  "nbf": 1719686631,                    // 生效时间，在此之前Token无效
  "iat": 1719686631,                    // 签发时间
  "jti": "a1b2c3d4e5f6",                // 唯一标识，防止重放攻击
  "scope": ["read", "write"]            // 权限范围
}
```

### 3. Signature（签名）

确保 Token 完整性的关键部分，计算公式：

```
HMACSHA256(base64UrlEncode(header) + "." + base64UrlEncode(payload), secret)
```

**签名的重要性：**

- 防止数据被篡改
- 验证消息的发送者
- 确保消息在传输过程中未被修改

## 完整 JWT 示例及解析

### 实际示例

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyLCJleHAiOjE1MTYyNDI2MjIsInNjb3BlIjpbInJlYWQiLCJ3cml0ZSJdfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

### 解码后内容

**Header:**

```
{
  "alg": "HS256",
  "typ": "JWT"
}
```

**Payload:**

```
{
  "sub": "1234567890",
  "name": "John Doe",
  "iat": 1516239022,
  "exp": 1516242622,
  "scope": ["read", "write"]
}
```

## 工作流程详解

### 标准认证流程

1. **用户登录**：客户端发送用户名密码到认证服务器
2. **验证凭证**：服务器验证用户身份信息
3. **生成JWT**：验证通过后，服务器生成包含用户信息的JWT
4. **返回Token**：将JWT返回给客户端
5. **存储Token**：客户端安全存储JWT（localStorage或HttpOnly Cookie）
6. **携带认证**：后续请求在Authorization头中携带JWT
7. **验证Token**：服务器验证JWT签名和有效期
8. **处理请求**：验证通过后执行相应业务逻辑

### 请求头示例

```
GET /api/user/profile HTTP/1.1
Host: api.example.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
Content-Type: application/json
```

## 安全最佳实践

### 存储安全策略

**客户端存储方案比较：**

- **localStorage**：避免CSRF攻击，但可能受到XSS攻击
- **HttpOnly Cookie**：防止XSS攻击，但需要考虑CSRF防护
- **Session Storage**：标签页生命周期，关闭后失效

**推荐方案：**

```
// 安全存储示例
const token = response.data.token;
localStorage.setItem('jwt_token', token);

// 请求时自动携带
axios.interceptors.request.use(config => {
  const token = localStorage.getItem('jwt_token');
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});
```

### 数据安全规范

**敏感信息处理：**

- 不要在Payload中存储密码、密钥等敏感信息
- 避免存储过多的用户个人信息
- 使用加密算法保护敏感声明（JWE）

**有效期管理：**

```
{
  "exp": Math.floor(Date.now() / 1000) + (60 * 60), // 1小时过期
  "iat": Math.floor(Date.now() / 1000)
}
```

### 技术实施要点

**密钥管理：**

- 使用强随机生成的密钥（至少256位）
- 定期轮换密钥，但确保旧密钥在有效期内仍可用
- 使用密钥管理服务（KMS）或硬件安全模块（HSM）

**算法选择建议：**

- 生产环境推荐使用RS256等非对称算法
- 避免使用已过时或不安全的算法（如HS1, none）

## 安全原理深度解析

### 签名验证机制

JWT 的安全性基于数字签名技术。验证过程：

1. **解析Token**：分割Header、Payload、Signature
2. **重新计算签名**：使用相同的密钥和算法计算签名
3. **比对验证**：比较计算出的签名与Token中的签名
4. **时效检查**：验证exp、nbf等时间声明

### 防篡改原理

```
// 签名验证伪代码
function verifyJWT(token, secret) {
  const [headerB64, payloadB64, signature] = token.split('.');
  
  // 重新计算签名
  const data = `${headerB64}.${payloadB64}`;
  const expectedSignature = HMACSHA256(data, secret);
  
  // 恒定时间比较，防止时序攻击
  return constantTimeCompare(signature, expectedSignature);
}
```

### 密钥安全体系

**多层次防护：**

- **应用层**：强密码策略和密钥轮换
- **系统层**：文件权限和访问控制
- **网络层**：传输加密和网络隔离
- **物理层**：硬件安全模块保护

## 高级应用场景

### 刷新令牌机制

```
{
  "access_token": "eyJhbGciOiJIUzI1NiIs...",
  "refresh_token": "eyJhbGciOiJIUzI1NiIs...",
  "expires_in": 3600,
  "token_type": "Bearer"
}
```

### 分布式会话管理

在微服务架构中，JWT可以作为服务间认证的凭证，减少中心化认证服务的压力。

### 权限精细控制

通过scope声明实现细粒度权限控制：

```
{
  "scope": [
    "user:read",
    "user:write", 
    "admin:users",
    "billing:invoices"
  ]
}
```

## 常见安全威胁及防护

### 1. 令牌泄露

- **威胁**：Token被窃取后可能被恶意使用
- **防护**：使用短期有效的Token，结合刷新机制

### 2. 算法混淆攻击

- **威胁**：攻击者修改算法为"none"绕过验证
- **防护**：在验证代码中明确指定允许的算法

### 3. 重放攻击

- **威胁**：截获Token后重复使用
- **防护**：使用jti声明和Token黑名单机制

### 4. 密钥破解

- **威胁**：弱密钥可能被暴力破解
- **防护**：使用强密钥，定期轮换

## 性能优化建议

### 令牌大小优化

- 避免在Payload中存储过多数据
- 使用简洁的声明名称
- 考虑使用压缩算法（如gzip）

### 验证性能

- 使用非对称算法时，缓存公钥验证结果
- 实现Token缓存机制，避免重复验证
- 使用CDN分发验证所需的公钥

## 总结

JWT作为一种现代化的认证解决方案，在正确实施的前提下，能够为应用提供安全、高效的身份认证机制。关键在于理解其安全原理，遵循最佳实践，并根据具体业务场景进行合理的架构设计。

**核心要点：**

- JWT是自包含的认证令牌，适合分布式系统
- 签名机制确保数据的完整性和真实性
- 合理设置有效期和存储方案是安全的关键
- 选择适当的加密算法和密钥管理策略
- 结合实际业务需求设计令牌的生命周期管理