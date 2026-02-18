---
name: net-efcore-developer
description: 开发EF Core数据库实体和迁移文件。当用户需要创建数据库实体、DbContext或迁移文件时调用。
---
# .NET EF Core 开发规范

## 角色

你是一名**资深 .NET 后端开发工程师**，负责**按公司规范开发 EF Core 数据库实体和迁移文件**。

## 核心规范

### 禁止使用数据注解

**实体模型类必须是纯净的 POCO 类**，所有数据库配置通过 Fluent API 实现。

### 命名规范

| 类型       | 规范                        | 示例              |
| ---------- | --------------------------- | ----------------- |
| 实体类     | 以 `Model` 结尾             | `UserInfoModel`   |
| 属性/字段  | 小写字母                    | `user_name`       |
| 表名       | `t_` 前缀 + 小写下划线      | `t_user_info`     |
| 主键       | 必须使用 `long` 类型        | `public long id`  |

### 其他规则

- 字符串类型**不需要**配置 MaxLength
- **不创建外键**，通过对应 id 关联
- 迁移文件仅在用户明确要求时创建

## 实体模型模板

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

## Fluent API 配置模板

```csharp
/// <summary>
/// 用户信息表配置
/// </summary>
public class UserInfoConfiguration : IEntityTypeConfiguration<UserInfoModel>
{
    public void Configure(EntityTypeBuilder<UserInfoModel> builder)
    {
        builder.ToTable("t_user_info")
            .HasComment("用户信息表");

        builder.HasKey(x => x.id);

        builder.Property(x => x.id).HasComment("主键ID");

        builder.Property(x => x.user_name)
            .IsRequired()
            .HasComment("用户名");

        builder.Property(x => x.email)
            .IsRequired()
            .HasComment("邮箱地址");

        builder.Property(x => x.create_time)
            .HasDefaultValueSql("now()")
            .HasComment("创建时间");

        builder.HasIndex(x => x.user_name).HasDatabaseName("idx_user_name");
        builder.HasIndex(x => x.email).IsUnique().HasDatabaseName("idx_email");
    }
}
```

## 项目结构规范

| 文件类型      | 项目位置         |
| ------------- | ---------------- |
| DbContext     | `.Database` 项目 |
| 实体模型      | `.Database` 项目 |
| 实体配置      | `.Database` 项目 |
| 迁移文件      | `.Api` 项目      |

### 迁移文件目录

```
{ServiceName}.Api/Data/Migrations/{DbContext名称}/
├── 20250212_InitialCreate.cs
├── 20250212_InitialCreate.Designer.cs
└── ContractDbContextModelSnapshot.cs
```

## DbContext 模板

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
        public ContractDbContext(DbContextOptions<ContractDbContext> options) : base(options) { }

        /// <summary>
        /// 用户信息表
        /// </summary>
        public DbSet<UserInfoModel> UserInfo { get; set; }

        protected override void OnModelCreating(ModelBuilder modelBuilder)
        {
            base.OnModelCreating(modelBuilder);
            modelBuilder.ApplyConfiguration(new UserInfoConfiguration());
        }
    }
}
```

## DbContext 注册

在 `Startup.cs` 的 `ConfigureServices` 方法中：

```csharp
// 注册主数据库
services.AddDbContextPool<ContractDbContext>(options =>
    options.UseNpgsql(builder.Configuration.GetConnectionString("ConnectionString")));

// 注册其他数据库（如需要）
services.AddDbContextPool<LogDbContext>(options =>
    options.UseNpgsql(builder.Configuration.GetConnectionString("LogConnectionString")));
```

## 连接字符串配置

在 `appsettings.json` 中：

```json
{
  "DefaultConnectionString": "框架配置库连接字符串",
  "ConnectionString": "主业务数据库连接字符串",
  "LogConnectionString": "其他数据库连接字符串（按需配置）"
}
```

| Key 名称              | 用途               |
| --------------------- | ------------------ |
| `DefaultConnectionString` | 框架内置配置库 |
| `ConnectionString`        | 服务主业务数据库 |
| `{业务名}ConnectionString` | 其他业务数据库 |

## 迁移自动应用

**禁止命令行手动迁移**，在 `Startup.cs` 的 `Configure` 方法中自动应用：

```csharp
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    // ...

    using (var serviceScope = app.ApplicationServices.GetService<IServiceScopeFactory>().CreateScope())
    {
        // 单 DbContext
        serviceScope.ServiceProvider.GetRequiredService<ContractDbContext>().Database.Migrate();

        // 多 DbContext（如需要）
        serviceScope.ServiceProvider.GetRequiredService<AuditDbContext>().Database.Migrate();
    }

    // ...
}
```

## 视图模型与原生 SQL

### 视图模型

用于查询数据库视图、JOIN 多表、聚合统计等场景。命名：`{Entity}View`。

```csharp
/// <summary>
/// 用户及部门信息视图模型
/// </summary>
public class UserWithDeptView
{
    public long id { get; set; }
    public string user_name { get; set; }
    public string department_name { get; set; }
}
```

### 原生 SQL 查询

```csharp
// 单条查询
public async Task<UserView> GetById(long id)
{
    var sql = @"SELECT * FROM public.t_user WHERE id = {0}";
    return await _dbcontext.Database.SqlQueryRaw<UserView>(sql, id).AsNoTracking().FirstOrDefaultAsync();
}

// 列表查询
public async Task<List<UserView>> GetList()
{
    var sql = @"SELECT * FROM public.t_user";
    return await _dbcontext.Database.SqlQueryRaw<UserView>(sql).AsNoTracking().ToListAsync();
}

// 批量查询（PostgreSQL ANY）
public async Task<List<UserView>> GetByIds(List<long> ids)
{
    var sql = @"SELECT * FROM public.t_user WHERE id = ANY(@ids)";
    return await _dbcontext.Database.SqlQueryRaw<UserView>(sql, new NpgsqlParameter("ids", ids)).AsNoTracking().ToListAsync();
}

// 多表 JOIN
public async Task<List<UserWithDeptView>> GetUserWithDepartment()
{
    var sql = @"SELECT u.id, u.user_name, d.name as department_name
                FROM public.t_user u
                LEFT JOIN public.t_department d ON u.department_id = d.id";
    return await _dbcontext.Database.SqlQueryRaw<UserWithDeptView>(sql).AsNoTracking().ToListAsync();
}

// 字典查询
public async Task<Dictionary<long, UserView>> GetDic()
{
    var list = await _dbcontext.Database.SqlQueryRaw<UserView>(@"SELECT * FROM public.t_user").AsNoTracking().ToListAsync();
    return list.ToDictionary(f => f.id, f => f);
}
```

### Model 与 View 的区别

| 特性         | 实体模型（Model）    | 视图模型（View）       |
| ------------ | -------------------- | ---------------------- |
| 用途         | 数据库表完整映射     | 查询结果轻量级投影     |
| 后缀         | `Model`              | `View`                 |
| 配置         | 需要 Fluent API      | 无需配置               |
| 操作         | 支持增删改查         | 仅支持查询             |
