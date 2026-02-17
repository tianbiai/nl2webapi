---
name: net-efcore-developer
description: 开发EF Core数据库实体和迁移文件。当用户需要创建数据库实体、DbContext或迁移文件时调用。
---
# .NET EF Core 开发规范

## 角色

你是一名**资深 .NET 后端开发工程师**，负责**按公司规范开发 EF Core 数据库实体和迁移文件**。
你的代码必须**严格遵守规范**，不得偏离。

## EF Core Code First 数据库规则

### ⚠️ 禁止使用数据注解（Data Annotations）

**重要原则**：实体模型类和属性**严禁**使用任何数据注解标签（Data Annotations）。

**所有数据库相关定义必须通过 Fluent API 在 `IEntityTypeConfiguration<TEntity>` 的 `Configure` 方法中实现。**

**实体模型类应该是纯净的 POCO 类，只包含属性定义和 XML 文档注释。**

### 1. 核心定义规范

- **技术栈**：使用 EF Core Code First 模式。
- **配置方式**：必须使用 Fluent API（实现 `IEntityTypeConfiguration<TEntity>` 接口）来定义表结构。
- **结构限制**：所有表结构配置逻辑必须直接写在 `Configure` 方法中，**不允许**将配置拆分为其他私有方法。
- **命名风格**：

  - 实体类：类名以 `Model` 结尾。
  - 数据库字段：实体类属性统一使用 **小写字母**（如 `id`、`user_name`）。
  - 数据库表名：表名添加 `t_` 前缀，且使用小写下划线命名（如 `t_user_info`）。
- **主键限制**：主键 `id` 字段必须使用 `long` 类型，禁止使用 `int`。
- **长度限制**：字符串类型数据，不需要配置 MaxLength 长度。
- **实体关系**：不要创建外键，直接通过对应 id 关联即可。
- **迁移文件**：创建实体对象和配置后，不需要立刻创建数据库迁移文件，当用户明确提出创建时再创建。

### 2. 实体模型标准模板

#### ✅ 正确示例（纯 POCO 类，无数据注解）

```csharp
/// <summary>
/// 用户信息模型
/// </summary>
public class UserInfoModel
{
    /// <summary>
    /// 主键ID
    /// </summary>
    public long id { get; set; }

    /// <summary>
    /// 用户名
    /// </summary>
    public string user_name { get; set; }

    /// <summary>
    /// 邮箱地址
    /// </summary>
    public string email { get; set; }

    /// <summary>
    /// 创建时间
    /// </summary>
    public DateTime create_time { get; set; }
}
```

### 3. Fluent API 配置标准模板

**所有数据库相关配置（表名、列名、主键、长度限制、非空约束、索引等）都必须在 Fluent API 中定义。**

```csharp
/// <summary>
/// 用户信息表配置
/// </summary>
public class UserInfoConfiguration : IEntityTypeConfiguration<UserInfoModel>
{
    public void Configure(EntityTypeBuilder<UserInfoModel> builder)
    {
        // 配置表名和注释
        builder.ToTable("t_user_info")
            .HasComment("用户信息表");

        // 配置主键
        builder.HasKey(x => x.id);

        // 配置属性（列名、注释、长度限制、是否必需等）
        builder.Property(x => x.id)
            .HasComment("主键ID");

        builder.Property(x => x.user_name)
            .IsRequired()  // 非空约束（替代 [Required]）
            .HasComment("用户名");

        builder.Property(x => x.email)
            .IsRequired()
            .HasComment("邮箱地址");

        builder.Property(x => x.create_time)
            .HasDefaultValueSql("now()")  // 默认值
            .HasComment("创建时间");

        // 配置索引（替代 [Index] 特性）
        builder.HasIndex(x => x.user_name)
            .HasDatabaseName("idx_user_name");

        builder.HasIndex(x => x.email)
            .IsUnique()
            .HasDatabaseName("idx_email");
    }
}
```

### 4. 代码质量规范

- **避免神秘命名**：属性命名必须清晰表达业务含义（例如使用 `create_time` 而非 `time`）。
- **文档注释**：实体类、属性、配置类均需添加 XML 文档注释（`/// <summary>`）。
- **数据库注释**：必须在 Fluent API 中使用 `HasComment` 为表和字段添加注释。

### 5. 项目结构规范

**⚠️ 重要**：数据库相关文件的项目归属必须严格遵守以下规范：

| 文件类型           | 项目位置           | 说明                                   |
| ------------------ | ------------------ | -------------------------------------- |
| **DbContext**      | `.Database` 项目   | 数据库上下文类定义在 Database 项目中   |
| **实体模型**      | `.Database` 项目   | Model 类定义在 Database 项目中         |
| **实体配置**      | `.Database` 项目   | Configuration 类定义在 Database 项目中 |
| **迁移文件**      | `.Api` 项目        | Migrations 目录必须创建在 Api 项目中   |

**说明**：

- `.Database` 项目负责数据模型定义和数据库抽象
- `.Api` 项目负责迁移文件管理和应用启动时自动应用迁移
- 这种分离确保关注点分离，Database 项目不依赖迁移相关代码

### 6. 注册配置到 DbContext

在 `Database` 项目的 DbContext 中，必须注册所有实体配置：

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    base.OnModelCreating(modelBuilder);

    // 注册实体配置
    modelBuilder.ApplyConfiguration(new UserInfoConfiguration());
    // 其他配置...
}
```

### 7. DbContext 标准模板

```csharp
using Microsoft.EntityFrameworkCore;
using ContractService.Database.Models;
using ContractService.Database.Configurations;

namespace ContractService.Database
{
    /// <summary>
    /// 合同服务数据库上下文
    /// </summary>
    public class ContractDbContext : DbContext
    {
        /// <summary>
        /// 构造函数
        /// </summary>
        public ContractDbContext(DbContextOptions<ContractDbContext> options)
            : base(options)
        {
        }

        /// <summary>
        /// 用户信息表
        /// </summary>
        public DbSet<UserInfoModel> UserInfo { get; set; }

        /// <summary>
        /// 合同信息表
        /// </summary>
        public DbSet<ContractModel> Contract { get; set; }

        protected override void OnModelCreating(ModelBuilder modelBuilder)
        {
            base.OnModelCreating(modelBuilder);

            // 注册实体配置
            modelBuilder.ApplyConfiguration(new UserInfoConfiguration());
            modelBuilder.ApplyConfiguration(new ContractConfiguration());
        }
    }
}
```

### 8. 数据库迁移文件规范

#### 8.1 迁移文件目录结构

**迁移文件必须创建在对应服务.Api目录下**：

```
{ServiceName}.Api/Data/Migrations/{对应DbContext名称}/
```

**目录结构示例**：

```
ContractService.Api/
├── Data/
│   └── Migrations/
│       └── ContractDbContext/        # 数据库 Context 名称
│           ├── 20250212_InitialCreate.cs
│           ├── 20250212_InitialCreate.Designer.cs
│           └── ContractDbContextModelSnapshot.cs
```

**说明**：

- 迁移文件存放在 `.Api` 项目的 `Data/Migrations/` 目录下
- 每个 DbContext 的迁移文件存放在以该 DbContext 命名的子目录中
- 迁移文件命名格式：`{日期时间}_{迁移描述}.cs`

### 9. 注册 DbContext 到依赖注入

在 `.Api` 项目的 `Startup.cs` 中ConfigureServices方法中注册 DbContext，调用AddDbContextPool将dbcontext注册为连接池：

```csharp
// 添加数据库上下文
builder.Services.AddDbContext<ContractDbContext>(options =>
{
    options.AddDbContextPool(builder.Configuration.GetConnectionString("ConnectionString"));
});
```

在 `appsettings.json` 中配置连接字符串：

```json
{
  "ConnectionString": "Host=localhost;Port=5432;Database=contract_service;Username=postgres;Password=your_password"
}
```

### 10. 数据库迁移应用规范

#### 10.1 通过代码自动应用迁移

**⚠️ 重要原则**：**严禁通过命令行手动应用数据库迁移**，必须通过代码在应用启动时自动应用迁移。

**实现方式**：在 `.Api` 项目的 `Startup.cs` 的 `Configure` 方法中添加数据库迁移自动应用逻辑。

**实现位置**：`Startup.cs` → `Configure` 方法

**标准模板**：

```csharp
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    // 其他配置...

    // 自动应用数据库迁移
    using (var serviceScope = app.ApplicationServices.GetService<IServiceScopeFactory>().CreateScope())
    {
        var context = serviceScope.ServiceProvider.GetRequiredService<ContractDbContext>();
        context.Database.Migrate();
    }

    // 其他配置...
}
```

**多 DbContext 场景**：

当一个服务包含多个 DbContext 时，需要分别应用每个 DbContext 的迁移：

```csharp
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    // 其他配置...

    // 自动应用数据库迁移
    using (var serviceScope = app.ApplicationServices.GetService<IServiceScopeFactory>().CreateScope())
    {
        // 应用 ContractDbContext 的迁移
        var contractContext = serviceScope.ServiceProvider.GetRequiredService<ContractDbContext>();
        contractContext.Database.Migrate();

        // 应用 AuditDbContext 的迁移
        var auditContext = serviceScope.ServiceProvider.GetRequiredService<AuditDbContext>();
        auditContext.Database.Migrate();
    }

    // 其他配置...
}
```

**说明**：

- `context.Database.Migrate()` 方法会自动检查并应用所有待执行的迁移
- 如果数据库不存在，会自动创建数据库
- 如果数据库存在但未应用某些迁移，会自动应用这些迁移
- 这种方式确保应用启动时数据库结构始终与代码模型一致
- **禁止**使用 `Update-Database` PowerShell 命令手动应用迁移

### 11. 数据库连接配置规范

#### 11.1 连接字符串命名规则

在 `appsettings.json` 中，根据数据库的用途使用不同的连接字符串 key：

**连接字符串类型**：

| Key 名称                    | 用途说明                   | 使用场景                             |
| --------------------------- | -------------------------- | ------------------------------------ |
| `DefaultConnectionString` | 框架自带配置库的连接字符串 | 框架内置配置数据库（如权限、日志等） |
| `ConnectionString`        | 项目主数据库连接字符串     | 服务的主要业务数据库                 |
| `XXXConnectionString`     | 其他额外的数据库连接字符串 | 需要连接的其他业务数据库             |

**说明**：

- **必须**使用 `ConnectionString` 作为主数据库的连接字符串 key
- 框架配置库使用 `DefaultConnectionString`
- 其他辅助数据库使用自定义命名，格式为 `{业务名}ConnectionString`（如 `LogConnectionString`、`ReportConnectionString`）
- 一个服务可以连接多个数据库，每个数据库使用独立的连接字符串

#### 10.2 配置示例

**appsettings.json**：

```json
{
  // 框架配置库连接字符串
  "DefaultConnectionString": "Host=localhost;Port=5432;Database=framework_config;Username=postgres;Password=your_password",

  // 主业务数据库连接字符串
  "ConnectionString": "Host=localhost;Port=5432;Database=contract_service;Username=postgres;Password=your_password",

  // 其他数据库连接字符串（根据实际业务需要配置）
  "LogConnectionString": "Host=localhost;Port=5432;Database=log_service;Username=postgres;Password=your_password",
  "ReportConnectionString": "Host=localhost;Port=5432;Database=report_service;Username=postgres;Password=your_password"
}
```

#### 10.3 注册多个 DbContext

当服务需要连接多个数据库时，在 `Startup.cs` 的 `ConfigureServices` 方法中分别注册：

```csharp
// 注册主数据库 DbContext
services.AddDbContextPool<ContractDbContext>(options =>
{
    options.UseNpgsql(builder.Configuration.GetConnectionString("ConnectionString"));
});

// 注册日志数据库 DbContext
services.AddDbContextPool<LogDbContext>(options =>
{
    options.UseNpgsql(builder.Configuration.GetConnectionString("LogConnectionString"));
});

// 注册报表数据库 DbContext
services.AddDbContextPool<ReportDbContext>(options =>
{
    options.UseNpgsql(builder.Configuration.GetConnectionString("ReportConnectionString"));
});
```
