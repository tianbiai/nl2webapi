# [服务名称]

> ⚠️ **强制要求**：在编写本服务 spec.md 之前，必须先完整阅读 `skills/` 目录下的相关技能文档，并严格遵循其中的最佳实践和指导原则。

## 📚 适用的技能文档清单

在编写本服务规格说明书时，已阅读并遵循以下技能文档：

### 按工作流程顺序（强制阅读）

- [ ] `dotnet-architect/SKILL.md` ⭐⭐⭐ - .NET 架构设计和代码审查（需求分析、技术选型、架构决策）
- [ ] `net-microservice-generator/SKILL.md` ⭐⭐⭐ - .NET 微服务项目架构和生成规范（项目框架生成）
- [ ] `net-api-efcore-developer/SKILL.md` ⭐⭐⭐ - .NET API 和 EF Core 开发规范（API 和数据库开发）

### 特定功能开发（按需阅读）

- [ ] `net-database-bulkcopy/SKILL.md` ⭐⭐ - PostgreSQL 批量数据操作规范（大数据量导入/更新/同步场景）
- [ ] `net-background-job/SKILL.md` ⭐⭐ - 后台循环任务开发规范（定时任务、数据同步等场景）

---

## 📋 业务与功能

### 1.1 服务概述

> 简要说明该服务的核心定位、解决的问题、业务价值

**服务名称**：[ServiceName]

**服务职责**：

- [职责1]
- [职责2]
- [职责3]

**依赖服务**：

- IdentityService - 认证授权服务
- [其他依赖服务]

### 1.2 核心目标

> 该服务的核心业务目标

[在此描述核心目标...]

### 1.3 功能模块清单

> 列出该服务包含的所有功能模块及其优先级

- **P0 [模块1]**：[功能描述]
- **P0 [模块2]**：[功能描述]
- **P1 [模块3]**：[功能描述]
- **P2 [模块4]**：[功能描述]

### 1.4 业务流程要点

> 关键的业务流程、事务边界、数据一致性要求

- [流程1]：[步骤1] → [步骤2] → [步骤3]
- [流程2]：[步骤1] → [步骤2] → [步骤3]

### 1.5 Controller 功能描述

> ⚠️ **重要**：本节列出该服务的所有 Controller 及其功能描述。每次新增、修改或删除 Controller 时都必须更新本节内容。

#### [Controller1 名称]

**Controller 名称**：`[Controller1Name]Controller`

**路由前缀**：`api/[module]/[resource]`

**功能描述**：

- [功能1]
- [功能2]
- [功能3]

**认证方式**：

- [ ] `Bearer Token` (Policy: "Logon")
- [ ] `Basic Auth` (Policy: "Basic")
- [ ] Bearer 和 Basic 都支持（不设置 Policy）
- [ ] 无需认证
  - 方式1：不设置 `[Authorize]`（需确保没有上级的 `[Authorize]`）
  - 方式2：使用 `[AllowAnonymous]`（覆盖上级的 `[Authorize]`）

**Swagger 配置**：

- **GroupName**：`[group-name]`
- **SwaggerTag**：`[中文描述]`

**API 接口清单**：

| 方法 | 路由 | 功能 | 优先级 | 认证 |
| ---- | ---- | ---- | ------ | ---- |
| POST | `/api/[module]/[resource]/xxx` | 创建资源、更新资源、删除资源 | P0 | ✅ |
| GET | `/api/[module]/[resource]` | 获取资源 | P0 | ✅ |

#### [Controller2 名称]

**Controller 名称**：`[Controller2Name]Controller`

**路由前缀**：`api/[module]/[resource]`

**功能描述**：

- [功能1]
- [功能2]
- [功能3]

**认证方式**：

- [ ] `Bearer Token` (Policy: "Logon")
- [ ] `Basic Auth` (Policy: "Basic")
- [ ] Bearer 和 Basic 都支持（不设置 Policy）
- [ ] 无需认证
  - 方式1：不设置 `[Authorize]`（需确保没有上级的 `[Authorize]`）
  - 方式2：使用 `[AllowAnonymous]`（覆盖上级的 `[Authorize]`）

**Swagger 配置**：

- **GroupName**：`[group-name]`
- **SwaggerTag**：`[中文描述]`

**API 接口清单**：

| 方法 | 路由 | 功能 | 优先级 | 认证 |
| ---- | ---- | ---- | ------ | ---- |
| POST | `/api/[module]/[resource]/xxx` | 创建资源、更新资源、删除资源 | P0 | ✅ |
| GET | `/api/[module]/[resource]` | 获取资源 | P0 | ✅ |

---

## 📊 数据模型设计

### 2.1 数据库表清单

> 列出该服务涉及的所有数据库表

| 表名         | 说明   | 主键类型 | 备注   |
| ------------ | ------ | -------- | ------ |
| `t_table1` | [说明] | `long` | [备注] |
| `t_table2` | [说明] | `long` | [备注] |
| `t_table3` | [说明] | `long` | [备注] |

### 2.2 数据访问策略

> 数据访问模式和策略

- **ORM**：Entity Framework Core (Code First)
- **数据库**：PostgreSQL
- **连接管理**：连接池配置
- **事务策略**：[说明]
- **并发控制**：[乐观锁 / 悲观锁]

### 2.3 数据库字段配置规范

> ⚠️ **必须遵循**：本项目严格遵循 Fluent API 配置规范，所有数据库配置通过 `IEntityTypeConfiguration<TEntity>` 接口实现。

**核心原则**：

1. **禁止使用数据注解**：实体模型类严禁使用任何数据注解标签（Data Annotations）
   - ❌ `[Table]`、`[Column]`、`[Key]`、`[Required]`、`[MaxLength]`、`[Index]` 等
   - ✅ 所有配置通过 Fluent API 在 `Configure` 方法中实现

2. **命名规范**：
   - **实体类**：类名以 `Model` 结尾（如 `UserInfoModel`）
   - **数据库字段**：实体类属性统一使用小写字母（如 `id`、`user_name`、`create_time`）
   - **数据库表名**：表名添加 `t_` 前缀，使用小写下划线命名（如 `t_user_info`）

3. **主键限制**：
   - 主键 `id` 字段必须使用 `long` 类型，禁止使用 `int`

4. **字符串长度配置**：
   - **默认规则**：字符串类型默认**不需要配置** MaxLength，由数据库自动处理
   - **特殊情况**：仅在业务明确要求限制长度时才配置 `HasMaxLength(n)`

5. **实体关系**：
   - 不创建外键关系，直接通过对应 id 字段关联即可

**配置示例**：

```csharp
public class UserInfoConfiguration : IEntityTypeConfiguration<UserInfoModel>
{
    public void Configure(EntityTypeBuilder<UserInfoModel> builder)
    {
        // 配置表名和注释
        builder.ToTable("t_user_info").HasComment("用户信息表");

        // 配置主键
        builder.HasKey(x => x.id);

        // 配置属性（列名、注释、长度限制、是否必需等）
        builder.Property(x => x.id).HasComment("主键ID");

        builder.Property(x => x.user_name)
            .IsRequired()  // 非空约束
            .HasMaxLength(50)  // 仅在需要时配置长度
            .HasComment("用户名");

        builder.Property(x => x.create_time)
            .HasDefaultValueSql("CURRENT_TIMESTAMP")
            .HasComment("创建时间");

        // 配置索引
        builder.HasIndex(x => x.user_name).HasDatabaseName("idx_user_name");
    }
}
```

---

## 🏗️ 架构与设计

> ⚠️ **必须遵循**：本部分设计必须严格基于 `dotnet-architect/SKILL.md` 中的架构设计原则和最佳实践。

### 3.1 项目结构

> 该服务的标准项目结构

```
{ServiceName}.slnx
├── {ServiceName}.Api/            # 表示层
│   ├── Controllers/              # 默认 Controllers 文件夹
│   │   ├── XxxController.cs
│   │   └── YyyController.cs
│   ├── [Category1]Controllers/   # 分类 Controllers 文件夹（与 Controllers 平级）
│   │   └── ZzzController.cs
│   ├── [Category2]Controllers/   # 分类 Controllers 文件夹（与 Controllers 平级）
│   │   └── AaaController.cs
│   ├── Program.cs
│   └── appsettings.json
└── {ServiceName}.Database/       # 数据层
    ├── Models/                   # 实体模型
    ├── Configurations/           # Fluent API 配置
    └── [ServiceName]DbContext.cs
```

**Controllers 目录组织说明**：

- `Controllers/` 是默认文件夹，用于存放通用的 Controller
- 其他分类的 Controllers 文件夹命名规则：`{分类}+Controllers`（如：WebControllers、MobileControllers、AdminControllers）
- 所有 Controllers 文件夹（包括默认的和分类的）都与 Api 项目根目录平级
- 常见分类方式：
  - 按客户端类型：Controllers（默认）、WebControllers、MobileControllers、AdminControllers
  - 按业务模块：Controllers（默认）、UserControllers、OrderControllers、ProductControllers
  - 按功能类别：Controllers（默认）、QueryControllers、CommandControllers、ReportControllers
- 选择合适的分类方式可以提高代码的可维护性和可读性

### 3.2 分层架构

> ⚠️ **必须遵循 ASP.NET Core 核心原则**：
>
> - **关注点分离**：不同职责分离到不同层
> - **依赖倒置**：高层模块不依赖低层模块，都依赖抽象
> - **单一职责**：每个类只有一个改变的理由
> - **开闭原则**：对扩展开放，对修改关闭

**标准分层**：

- **Api 层**：Controller、路由、认证授权
- **Database 层**：实体模型、Fluent API 配置、DbContext

### 3.3 依赖注入配置

> 服务生命周期和注册策略

- **Transient**：[轻量级无状态服务]
- **Scoped**：[DbContext、UnitOfWork、业务服务]
- **Singleton**：[配置、缓存、HTTP 客户端]

---

## 🔐 安全与认证

### 4.1 认证机制

> 认证方式和实现策略

- **认证方式**：[Bearer Token / Basic Auth / 两者都支持 / 无需认证]
- **Token 来源**：[HTTP Header]
- **认证服务**：[IdentityService]

### 4.2 授权策略

> 使用ThirdNet框架自定义授权。

### 4.3 数据验证

> 输入验证和业务规则验证。不使用[Data Annotations / FluentValidation]，通过自定义代码验证。

### 4.4 安全防护

> 常见安全威胁防护措施

- [ ] **SQL 注入防护**：[使用 ORM 参数化查询]
- [ ] **XSS 防护**：[输入过滤 / 输出编码]
- [ ] **HTTPS 强制**：[生产环境强制 HTTPS]
- [ ] **速率限制**：[防止暴力攻击]
- [ ] **敏感数据加密**：[加密算法 / 密钥管理]

---

## 🚀 性能与优化

### 5.1 缓存策略

> 按需创建缓存，参考缓存技能 `net-cache-use`

### 5.2 异步处理

> 异步编程模式和最佳实践

- **异步原则**：All the way down
- **取消令牌**：支持操作取消
- **超时控制**：[超时策略]

### 5.3 数据库优化

> 查询优化和索引策略

- **查询优化**：
  - [ ] 避免 N+1 查询
  - [ ] 使用投影（Select）
  - [ ] 分页查询
  - [ ] 延迟加载 vs 预加载
- **索引策略**：
  - `[索引1]`：[字段] - [原因]
  - `[索引2]`：[字段] - [原因]

### 5.4 批量操作

> 大数据量操作策略（如需要）

- [ ] 使用 `net-database-bulkcopy` 进行批量导入
- [ ] 批量更新策略
- [ ] 数据同步方案

---

## ⏰ 后台任务

### 6.1 后台任务清单

> 列出该服务的所有后台任务（如需要）

- **[Task1]**：[任务描述] - [执行频率]
- **[Task2]**：[任务描述] - [执行频率]

### 6.2 任务配置

> 后台任务的配置和实现要求

- **框架**：BackgroundRunner
- **注册方式**：`builder.Services.AddHostedService<[TaskName]>()`
- **异常处理**：内置异常捕获和日志
- **停止策略**：优雅停止

---

## 🧪 测试策略

### 7.1 测试范围

> 单元测试、集成测试、端到端测试

- **单元测试**：[覆盖率和测试工具]
- **集成测试**：[测试场景和工具]
- **API 测试**：[Swagger UI / Postman]

### 7.2 关键测试场景

> 必须测试的业务场景

- [ ] [场景1]：[测试描述]
- [ ] [场景2]：[测试描述]
- [ ] [场景3]：[测试描述]

---

## 🚢 部署与运维

### 8.1 环境配置

> 环境变量和配置管理

- **配置源**：appsettings.json / 环境变量
- **环境清单**：
  - **Development**：[开发环境配置]
  - **Staging**：[预发布环境配置]
  - **Production**：[生产环境配置]

### 8.2 日志与监控

> 日志记录和监控策略

- **日志框架**：内置 ILogger
- **日志级别**：
  - **Information**：一般信息
  - **Warning**：警告信息
  - **Error**：错误信息
  - **Critical**：严重错误
