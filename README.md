# A2A-T Agent Registry Center

A2A-T智能体AgentCard注册中心，提供RESTful API服务用于管理和查询AI Agent卡片信息。

## 项目概述

本项目是一个基于FastAPI的Agent注册中心服务，实现了A2A-T协议规范，提供Agent的注册、查询和管理功能。服务支持TLS/SSL安全连接、流量控制、审计日志等企业级特性。

## 核心功能

### 1. Agent注册与查询
- **注册Agent**: 通过POST接口注册新的Agent，支持唯一性校验（name + organization）
- **查询Agent**: 支持精确查询，按name和organization字段过滤
- **数据持久化**: 使用JSON文件存储Agent数据，支持自动加载和保存

### 2. 安全特性
- **TLS/SSL支持**: 支持双向TLS认证，支持CRL证书吊销列表
- **密码套件转换**: 支持IANA到OpenSSL密码套件格式转换
- **客户端认证**: 支持基于证书的客户端身份验证

### 3. 流量控制
- **限流机制**: 基于IP的请求频率限制（注册/查询独立配置）
- **并发控制**: 支持注册和查询的并发数限制
- **连接限制**: 最大连接数控制
- **超时控制**: 请求处理超时控制

### 4. 数据验证
- **AgentCard验证**: 基于Pydantic的完整数据模型验证
- **字段长度限制**: 对各字段设置最大长度限制
- **格式校验**: URL格式、字符集等格式验证
- **安全字符检查**: 防止注入攻击的字符过滤

### 5. 审计与日志
- **操作审计**: 记录所有关键操作（注册、查询、服务启动等）
- **日志管理**: 基于loguru的结构化日志

## 项目结构

```
registry-center/
├── agent_registry/          # 核心注册服务模块
│   ├── server.py           # FastAPI服务器，提供REST API接口
│   ├── core.py             # 核心注册逻辑，Agent存储和管理
│   ├── start.py            # 服务启动入口，SSL配置
│   ├── config.py           # 配置常量定义
│   ├── persistence.py       # 数据持久化（JSON文件）
│   ├── middleware.py        # 中间件（连接限制、超时）
│   ├── cipher_converter.py  # 密码套件格式转换
│   ├── registry_instance.py # 注册中心单例
│   └── model/              # 数据模型
│       └── validated_agentcard.py  # AgentCard验证模型
├── common/                  # 公共模块
│   ├── custom/             # 自定义处理器机制
│   │   ├── custom_handle.py      # 处理器注册表
│   │   └── interface_type.py     # 接口类型枚举
│   ├── util/               # 工具类
│   │   ├── authenticate_util.py  # 认证工具
│   │   ├── cipher_util.py       # 加密工具
│   │   ├── config_util.py       # 配置工具
│   │   └── conf_util.py         # 配置对象
│   ├── cert/               # 证书相关
│   └── log/                # 日志相关
├── etc/                    # 配置文件
│   ├── server.conf         # 服务器配置
│   ├── server.properties   # 属性配置
│   └── log_config.conf     # 日志配置
├── bin/                    # 启动脚本
│   ├── start.sh            # 启动脚本
│   └── stop.sh             # 停止脚本
├── tests/                  # 测试代码
│   └── test_register.py    # 注册功能测试
├── requirements.txt        # Python依赖
└── README.md              # 项目说明
```

## API接口

### 注册Agent
```
POST /rest/a2a-t/v1/agents/register
Content-Type: application/json

请求体：AgentCard对象
响应：201 Created (成功) / 409 Conflict (重复) / 400 Bad Request (数据无效)
```

### 查询Agent
```
GET /rest/a2a-t/v1/agents/query?name={name}&organization={organization}

参数：
- name: Agent名称（可选）
- organization: 组织名称（可选）

响应：AgentCard列表
```

## 配置说明

主要配置项（etc/server.conf）：
- `connection.max`: 最大连接数
- `connection.timeout`: 连接超时时间
- `flowcontrol.ratelimit.register`: 注册接口限流
- `flowcontrol.ratelimit.query`: 查询接口限流
- `flowcontrol.parallelism.register`: 注册并发数
- `flowcontrol.parallelism.query`: 查询并发数
- `agent.num.max`: 最大Agent数量
- `tls.version`: TLS版本
- `tls.cipher`: 密码套件

## 依赖项

- a2a-sdk>=0.3.2: A2A协议SDK
- fastapi>=0.115.11: Web框架
- uvicorn>=0.34.0: ASGI服务器
- loguru>=0.7.3: 日志库
- cryptography~=46.0.5: 加密库
- limits~=4.0.0: 限流库
- starlette~=0.50.0: ASGI工具包

## 启动方式

```bash
# 使用启动脚本
./bin/start.sh

# 或直接运行Python模块
python -m agent_registry.start
```

## 扩展机制

项目提供了自定义处理器机制，通过`HandlerRegistry`可以扩展以下功能：
- **AUTHENTICATE**: 认证处理器
- **AUDIT**: 审计处理器
- **INSERT**: 插入处理器
- **QUERY**: 查询处理器
- **DECRYPT**: 解密处理器

通过继承`BaseHandler`并实现`handle`方法，可以自定义这些接口的实现。

## 安全特性

1. **数据安全**：
   - 敏感数据文件权限设置为600
   - 支持数据加密存储
   - 文件大小限制（100MB）

2. **网络安全**：
   - 强制TLS/SSL加密传输
   - 支持客户端证书认证
   - 支持CRL证书吊销

3. **应用安全**：
   - 输入数据严格验证
   - 防止注入攻击
   - 请求大小限制
   - URL长度限制

## 版本信息

当前版本：2.0.0