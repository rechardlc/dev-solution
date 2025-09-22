# API网关设计架构说明文档

## 概述

本项目采用统一网关设计模式，基于安全性要求，实现了单一接口、统一POST请求和接口加密的安全架构。

## 设计背景

由于安全性要求，系统采用以下设计原则：

1. **单一接口入口**：所有API请求通过统一网关处理
2. **统一POST请求**：仅支持POST方法，提高安全性
3. **双重加密机制**：SM2 + SM4国密加密算法
4. **请求白名单**：部分接口可豁免加密处理

## 核心架构组件

### 1. 网关配置 (GatewayConfig)

```typescript
interface GatewayConfig {
  baseUrl: string;           // 网关基础URL
  timeout: number;           // 请求超时时间
  appKey: string;            // 应用密钥
  debug: boolean;            // 调试模式
  sm2PrivateKey: string;     // SM2私钥
  sm2PublicKey: string;      // SM2公钥
}
```

### 2. 加密策略

#### SM2加密 (非对称加密)
- **用途**：加密关键信息（service路径、SM4密钥和IV）
- **实现**：前后端各自生成密钥对，交换公钥
- **特点**：使用对方公钥加密，自己私钥解密

#### SM4加密 (对称加密)
- **用途**：加密请求体数据
- **密钥管理**：动态生成16字节密钥和IV
- **安全性**：每次请求使用新的密钥和IV

### 3. 路径管控

#### 允许路径 (ALLOWED_PATHS)
```typescript
const ALLOWED_PATHS = [
  '/gather/toMethod',      // 主要网关入口
  '/gather/proxyUploads',  // 文件上传代理
  '/systemDictionary/queryPage'  // 系统字典查询
];
```

#### 白名单路径 (WHITE_LIST)
```typescript
const WHITE_LIST = ['/api/health'];  // 健康检查等无需加密的接口
```

## 请求处理流程

### 请求拦截器流程

1. **URL预处理**：去除前导斜杠，标准化路径
2. **参数合并**：将params和data参数合并，data优先级更高
3. **路径转换**：转换为安全的网关路径（随机c1-c5）
4. **调试模式**：开发环境下在URL中附加原始路径
5. **IV生成**：为每个请求生成新的16字节IV
6. **方法验证**：强制使用POST方法
7. **请求头设置**：添加必要的安全头信息
8. **路径验证**：检查是否在允许路径列表中
9. **服务加密**：使用SM2加密service路径
10. **数据加密**：非白名单接口使用SM4加密请求体
11. **密钥加密**：使用SM2加密SM4的key和iv
12. **超时设置**：配置请求超时时间

### 响应拦截器流程

1. **状态检查**：处理HTTP状态码（如401未授权）
2. **密钥解密**：使用SM2解密响应头中的key和iv
3. **数据解密**：非白名单接口使用SM4解密响应数据
4. **结果返回**：返回解密后的JSON数据

## 安全特性

### 1. 时间戳验证
- 防止重放攻击
- 时间窗口：10分钟内有效
- 服务器时间同步要求

### 2. 动态加密
- 每次请求生成新的SM4密钥和IV
- SM2加密保护SM4密钥传输
- 双重加密确保数据安全

### 3. 路径白名单
- 严格控制可访问的网关路径
- 支持白名单机制豁免特定接口
- 运行时路径验证

### 4. 请求头验证
必需的请求头字段：
- `service`: 目标服务路径（SM2加密）
- `appKey`: 应用标识密钥
- `timeStamp`: 请求时间戳
- `tokenKey`: 用户会话令牌
- `key`: SM4加密密钥（SM2加密）
- `iv`: SM4加密向量（SM2加密）

## 使用示例

### 基本请求
```typescript
import request from './fetch';

// 加密请求示例
const response = await request('/api/user/profile', {
  method: 'POST',
  data: {
    userId: 123,
    fields: ['name', 'email']
  }
});

// 自定义网关路径
const response = await request('/api/sensitive/data', {
  method: 'POST',
  data: { query: 'secret' }
}, '/gather/toMethod/c3');
```

### 调试模式
在开发环境中，设置 `NUXT_PUBLIC_DEBUG=true` 可以：
- 跳过加密步骤
- 在URL中显示原始路径
- 输出详细的调试信息

## 环境配置

### 必需的环境变量
```bash
NUXT_PUBLIC_GATEWAY_BASE_URL=http://localhost:8080
NUXT_PUBLIC_GATEWAY_SM2_FRONT_PRIVATE_KEY=your_private_key
NUXT_PUBLIC_GATEWAY_SM2_BACK_PUBLIC_KEY=backend_public_key
NUXT_PUBLIC_GATEWAY_TIMEOUT=10000
NUXT_PUBLIC_APP_KEY=your_app_key
NUXT_PUBLIC_DEBUG=false
```

## 类型定义

### RequestOptions
```typescript
type RequestOptions = {
  method: 'POST';
  headers?: { [key: string]: unknown };
  body?: string | Record<string, any>;
  data?: Record<string, any>;
  timeout?: number;
  [key: string]: unknown;
};
```

### ResponseData
```typescript
type ResponseData = {
  data: unknown;
  message: string;
};
```

## 错误处理

### 常见错误类型
1. **请求头验证失败**：缺少必需头信息
2. **时间戳过期**：请求时间超出允许窗口
3. **路径不被允许**：请求的网关路径未在白名单中
4. **加密/解密失败**：国密算法处理异常
5. **未授权访问**：401状态码处理

### 错误处理策略
- 详细的错误日志记录
- 友好的错误信息提示
- 自动重试机制（适用场景）
- 降级处理方案

## 性能考虑

### 优化策略
1. **密钥复用**：会话期间适当复用SM4密钥
2. **并发处理**：支持多个请求并发加密
3. **缓存机制**：环境配置和公钥缓存
4. **异步处理**：加密解密操作异步化

### 监控指标
- 请求加密/解密耗时
- 网关响应时间
- 错误率统计
- 并发请求数量

## 扩展性

### 支持的扩展点
1. **新增白名单路径**：修改 `WHITE_LIST` 配置
2. **自定义加密算法**：实现新的加密方法
3. **请求头扩展**：添加新的安全头字段
4. **中间件集成**：与其他安全中间件集成

### 版本兼容性
- 向后兼容旧版本API
- 渐进式升级策略
- 协议版本协商机制

## 最佳实践

### 开发建议
1. **密钥管理**：生产环境使用安全的密钥管理系统
2. **日志安全**：避免在日志中记录敏感信息
3. **错误处理**：不要在错误信息中暴露内部实现细节
4. **性能测试**：定期进行加密性能基准测试

### 安全建议
1. **定期轮换**：定期更换SM2密钥对
2. **网络安全**：结合HTTPS传输层安全
3. **访问控制**：实施严格的API访问控制
4. **安全审计**：定期进行安全代码审查

---

**文档版本**: 1.0  
**最后更新**: 2025年9月22日  
**维护者**: 前端开发团队
