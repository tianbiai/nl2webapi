# .NET API Agent 工作流程

## 📌 总体概述

根据用户需求自动生成 API 设计文档和代码，进行 .NET 10 微服务架构的后端 API 开发工作。

## 🔴 核心原则

### 文档驱动开发（Document-Driven Development）

1. **spec.md 是项目唯一权威文档**

   - 各服务根目录 `projects/项目名/{ServiceName}/spec.md`：服务级完整功能说明书，使用 `rules/service-spec-template.md` 模板
   - spec.md 中应包含每个 Controller 的功能描述
   - 严禁代码与文档不一致
2. **开发必须按顺序进行**

   - 先生成各服务根目录 `projects/项目名/{ServiceName}/spec.md` → 再进行开发 → 代码变更时同步更新文档
   - 严禁跳过 spec.md 直接编写代码
3. **需求变更管理**

   - 任何新增/调整需求必须先更新对应服务的根目录 spec.md（`projects/项目名/{ServiceName}/spec.md`）
   - 严禁在不更新文档的情况下直接开发新功能
4. **代码与文档双向一致性**

   - 若发现代码实现与 spec.md 不一致，应主动指出并要求基于代码实现更新对应的 spec.md
   - 确保文档始终反映真实的系统状态

### 技术规范遵循

**在开始任何工作前必须完整阅读 `skills/` 目录下的技能文档！**

1. **dotnet-architect** ⭐⭐⭐ - .NET 架构设计和代码审查（需求分析、技术选型、架构决策）
2. **net-microservice-generator** ⭐⭐⭐ - .NET 微服务项目架构和生成规范（项目框架生成）
3. **net-api-efcore-developer** ⭐⭐⭐ - 所有 .NET API 和 EF Core 开发的最高优先级规范（API 和数据库开发）
4. **net-cache-use** ⭐⭐ - 缓存类库创建与使用规范（为业务实体添加缓存功能）
5. **net-database-bulkcopy** ⭐⭐ - PostgreSQL 批量数据操作规范（大数据量导入/更新/同步场景）
6. **net-background-job** ⭐⭐ - 后台循环任务开发规范（定时任务、数据同步等场景）

---

## 🔄 工作流程总览

```
用户输入需求 → 需求分析与澄清 → 各服务根目录 spec.md（projects/项目名/{ServiceName}/spec.md） → 项目框架生成 → MVP 开发验证 → 详细功能开发 → 服务验证 → 交付
```

---

## 📄 阶段零：需求分析与澄清

**核心原则**：主动沟通，充分理解，避免误解

### 需求确认清单

使用 `AskUserQuestion` 工具确认以下要素：

| 要素     | 说明             | 示例                                          |
| -------- | ---------------- | --------------------------------------------- |
| 服务名称 | 服务唯一标识     | 用户服务、订单服务、支付服务                  |
| 核心目标 | 解决什么问题     | 提供用户认证和授权功能                        |
| 业务领域 | 所属业务模块     | 用户管理、订单处理、支付结算                  |
| 功能清单 | 包含哪些功能模块 | 用户注册、登录、信息管理、权限控制            |
| 数据实体 | 需要哪些数据表   | 用户表、角色表、权限表                        |
| 接口需求 | 需要哪些 API     | POST /api/users/register、GET /api/users/{id} |
| 认证方式 | API 认证方式     | Bearer Token、Basic Auth、无需认证            |

### 🏗️ 架构要素确认（重点）

**⚠️ 重要**：微服务架构设计直接影响系统可扩展性和维护性，必须重点确认以下架构要素：

| 架构要素           | 说明                     | 示例                           |
| ------------------ | ------------------------ | ------------------------------ |
| **服务拆分** | 如何拆分微服务           | 按业务领域（用户、订单、支付） |
| **数据隔离** | 数据库归属策略           | 每个服务独立数据库             |
| **通信方式** | 服务间通信方式           | HTTP REST API（暂不考虑 MQ）   |
| **认证服务** | 是否需要 IdentityService | 统一认证授权服务               |
| **通用组件** | 需要哪些通用类库         | Common、Cache、工具类          |
| **技术约束** | 技术限制和规范要求       | .NET 10、PostgreSQL、EF Core   |

**💡 提示**：

- 如果用户不清楚服务拆分方式，可提供 2-3 种拆分方案供选择
- 数据隔离策略应考虑业务边界和数据一致性要求
- 认证服务通常作为独立服务统一管理用户身份

### ⚠️ 常见误区

- ❌ 直接基于模糊需求开始开发
- ❌ 假设用户没有提到的功能就是不需要
- ❌ 仅凭猜测做出技术决策
- ❌ 忽视微服务拆分的合理性
- ✅ 主动提问澄清所有不确定点
- ✅ 确认边界条件和特殊情况
- ✅ 与用户确认对需求的理解是否准确
- ✅ 评估服务拆分的必要性

## 阶段一：生成完整服务功能说明书

**前置条件**：需求分析与澄清完成后

### 核心任务

按照 `rules/service-spec-template.md` 模板生成服务级 spec.md：

**文件路径**：`projects/项目名/{ServiceName}/spec.md`

### 技术栈规范（不可变）

**⚠️ 重要：必须严格遵守以下技术栈，严禁随意更改！**

**核心框架**：

- **.NET 版本**：10
- **数据库**：PostgreSQL
- **ORM**：Entity Framework Core（Code First）
- **架构模式**：微服务架构

**项目模板**：

- **Web API 服务**：使用 `ThirdNet.Core.WebApiService` 模板
- **认证服务（包含了认证模块的Web API 服务）**：使用 `ThirdNet.Core.IdentityService` 模板
- **类库项目**：使用标准 `classlib` 模板

### 服务结构规范

项目以projects作为根目录。

**项目标准目录结构**：

projects/                           # 根目录
└── 项目名/                         # 具体项目
    ├── Tools/                      # 通用工具类库
    │   ├── {项目名}.Common/
    │   └── {项目名}.Cache/
    ├── IdentityService/            # 认证服务
    │   ├── spec.md                 # 服务级 spec.md（包含所有 Controller 功能描述）
    │   ├── IdentityService.slnx
    │   ├── IdentityService.Api/
    │   │   ├── Controllers/                    # 默认 Controllers 文件夹
    │   │   │   ├── XxxController.cs
    │   │   │   └── YyyController.cs
    │   │   ├── [Category1]Controllers/         # 分类 Controllers 文件夹（与 Controllers 平级）
    │   │   │   └── ZzzController.cs
    │   │   └── [Category2]Controllers/         # 分类 Controllers 文件夹（与 Controllers 平级）
    │   │       └── AaaController.cs
    │   └── IdentityService.Database/
    └── {ServiceName}/              # 业务微服务
        ├── spec.md                 # 服务级 spec.md（包含所有 Controller 功能描述）
        ├── {ServiceName}.slnx
        ├── {ServiceName}.Api/
        │   ├── Controllers/                    # 默认 Controllers 文件夹
        │   │   ├── XxxController.cs
        │   │   └── YyyController.cs
        │   ├── [Category1]Controllers/         # 分类 Controllers 文件夹（与 Controllers 平级）
        │   │   └── ZzzController.cs
        │   └── [Category2]Controllers/         # 分类 Controllers 文件夹（与 Controllers 平级）
        │       └── AaaController.cs
        └── {ServiceName}.Database/

### 需求变更管理（持续性流程）

**适用场景**：服务开发过程中所有新增需求、功能调整、需求变更

### 变更流程

```
发现新需求 → 评估影响 → 更新对应服务的根目录 spec.md（projects/项目名/{ServiceName}/spec.md） → 获得确认 → 继续开发
```

### 变更规范

1. **第一时间更新对应服务的根目录 spec.md**

   - 新增 API → 补充到"API 接口清单"和"接口设计规范"
   - 调整功能 → 更新对应的"业务流程"和"数据结构"
   - 变更实体 → 更新"数据库设计"和"数据模型"
2. **变更影响评估**

   - 评估对现有 API 的影响
   - 评估是否需要调整数据库结构
   - 评估是否需要调整其他服务
3. **确认后再开发**

   - 对应服务的根目录 spec.md 更新完成后需获得确认
   - 确认后方可进行代码开发

---

## 🏗️ 阶段二：项目框架生成与验证

**前置条件**：对应服务的 spec.md（`projects/项目名/{ServiceName}/spec.md`）已完成

### 项目生成范围

**目标**：使用 `net-microservice-generator` 技能生成标准化的微服务项目结构

### MVP 开发原则

- **标准化**：严格遵循公司项目模板和规范
- **可验证**：确保项目可以编译和启动
- **迭代基础**：为后续功能开发奠定基础

### MVP 验证内容

1. 项目结构是否符合标准规范
2. 各服务项目能否正常编译
3. 数据库连接配置是否正确
4. 是否需要调整对应服务的根目录 spec.md（`projects/项目名/{ServiceName}/spec.md`）

**⚠️ 验证通过后**：进入阶段三，基于对应服务的根目录 spec.md 进行详细功能开发。

## 📋 阶段三：详细功能开发

**前置条件**：

1. 对应服务的 spec.md（`projects/项目名/{ServiceName}/spec.md`）已完成
2. 项目框架已生成并验证通过
3. 基础架构已确认

### 开发规范

1. 所有开发工作必须基于服务根目录 spec.md 中的设计和功能描述
2. 新增或调整功能时，必须先更新对应服务的根目录 spec.md
3. Controller 的功能描述必须在服务根目录 spec.md 中维护和更新

### 开发流程

```
创建实体模型 → 创建 Fluent API 配置 → 创建 Controller → 实现 API 接口 → 注册配置 → 测试验证
```

---

## ✅ 阶段四：完整服务验证

### 验证内容

1. 项目可编译且正常启动
2. API 接口功能完整验证
3. 数据库操作正确性验证
4. Swagger 文档完整性验证
5. 整体服务的集成验证

### 验证流程

```
编译项目 → 启动服务 → Swagger 文档检查 → API 接口测试 → 数据库验证 → 验证通过
```

### 说明

- 使用 Swagger UI 进行 API 接口测试
- 确保所有接口正常返回预期结果
- 验证数据库操作（增删改查）的正确性
- 验证认证授权机制是否生效
- 确保代码与需求文档一致
- 确保没有占位代码或未实现的功能

---

## 📋 工作流程总结

```
1️⃣ 需求分析
   ↓
2️⃣ 生成各服务根目录 spec.md（projects/项目名/{ServiceName}/spec.md）（服务级完整功能说明书）
   ↓
3️⃣ 项目框架生成（使用 net-microservice-generator）
   ↓
4️⃣ MVP 框架验证
   ↓
5️⃣ 详细功能开发（遵循 net-api-efcore-developer）
   ↓
6️⃣ 完整服务验证
   ↓
7️⃣ 交付成果
```

**核心原则回顾**：

- 各服务 spec.md（`projects/项目名/{ServiceName}/spec.md`）是唯一权威文档
- 所有开发工作必须基于对应服务 spec.md
- 需求变更必须先更新对应服务 spec.md
- 代码与文档保持一致
- 严格遵循 .NET 微服务开发规范
