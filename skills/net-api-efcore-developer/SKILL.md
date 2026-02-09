---
name: net-api-efcore-developer
description: 开发.NET API接口和EF Core数据库实体。当用户需要创建Controller、API接口或数据库实体时调用。
---
# .NET API 和 EF Core 开发规范

## 角色

你是一名**资深 .NET 后端开发工程师**，负责**按公司规范开发标准化 API 接口和 EF Core 数据库实体**。
你的代码必须**严格遵守规范**，不得偏离。

## API 开发规范

### 1. 路由定义规范

API 接口路径必须使用 `api` 开头，并使用 Controller 名称（通常去掉 Controller 后缀）作为后续路径。

### 2. Controller 属性配置

在定义 Controller 类时，需包含以下标准属性：

- **`[Route("api/manager/agent")]`**：定义接口路径。必须以 `api` 开头。
- **`[ApiController]`**：标识为 API 控制器，这是默认的接口属性。
- **`[ApiExplorerSettings(GroupName = "manager")]`**：定义 Swagger 文档页面的分组名称。
- **`[SwaggerTag("智能体管理")]`**：定义 Swagger 文档页面显示的标签/描述。
- **`[Authorize]`**：标识需要认证授权。通过 `Policy` 参数可指定具体的认证方式：

  - **`[Authorize(Policy = "Logon")]`**：使用 Bearer Token 进行认证（用户登录）
  - **`[Authorize(Policy = "Basic")]`**：通过 Basic 请求头进行认证
  - **`[Authorize]`**（不设置 Policy）：同时支持 Bearer Token 和 Basic Auth
- **无需认证**：两种方式
  - 不设置 `[Authorize]`（需确保没有上级的 `[Authorize]`）
  - 使用 `[AllowAnonymous]`（覆盖上级的 `[Authorize]`）

### 3. Controller 标准模板

```csharp
/// <summary>
/// 智能体管理控制器
/// </summary>
[Route("api/manager/agent")]
[ApiController]
[ApiExplorerSettings(GroupName = "manager")]
[SwaggerTag("智能体管理")]
[Authorize(Policy = "Logon")]
public class AgentController : ControllerBase
{
    // API 接口实现
}
```

### 4. Controllers 目录组织规范

在 `.Api` 项目中，Controllers 文件夹的组织遵循以下规范：

#### 4.1 目录结构

```
{ServiceName}.Api/
├── Controllers/              # 默认 Controllers 文件夹
│   ├── XxxController.cs
│   └── YyyController.cs
├── [Category1]Controllers/   # 分类 Controllers 文件夹（与 Controllers 平级）
│   └── ZzzController.cs
├── [Category2]Controllers/   # 分类 Controllers 文件夹（与 Controllers 平级）
│   └── AaaController.cs
├── Program.cs
└── appsettings.json
```

#### 4.2 组织规则

- **默认文件夹**：`Controllers/` 是默认文件夹，用于存放通用的 Controller
- **分类文件夹命名规则**：`{分类}+Controllers`（如：`WebControllers`、`MobileControllers`、`AdminControllers`）
- **平级组织**：所有 Controllers 文件夹（包括默认的和分类的）都与 Api 项目根目录平级
- **禁止嵌套**：不要在 Controllers 文件夹内再创建子文件夹

#### 4.3 常见分类方式

根据业务特点选择合适的分类方式：

1. **按客户端类型分类**（推荐）：
   - `Controllers/` - 默认，通用接口
   - `WebControllers/` - Web 端接口
   - `MobileControllers/` - 移动端接口
   - `AdminControllers/` - 管理后台接口

2. **按业务模块分类**：
   - `Controllers/` - 默认，通用模块
   - `UserControllers/` - 用户相关模块
   - `OrderControllers/` - 订单相关模块
   - `ProductControllers/` - 商品相关模块

3. **按功能类别分类**：
   - `Controllers/` - 默认，通用功能
   - `QueryControllers/` - 查询类接口
   - `CommandControllers/` - 命令类接口
   - `ReportControllers/` - 报表类接口

**选择合适的分类方式可以提高代码的可维护性和可读性。**

## EF Core Code First 数据库规则

### ⚠️ 禁止使用数据注解（Data Annotations）

**重要原则**：实体模型类和属性**严禁**使用任何数据注解标签（Data Annotations），包括但不限于：

- ❌ `[Column("xxx")]` - 列名映射
- ❌ `[MaxLength(50)]` / `[MinLength]` - 长度限制
- ❌ `[Required]` - 非空约束
- ❌ `[Key]` - 主键标识
- ❌ `[DatabaseGenerated]` - 生成策略
- ❌ `[Table("xxx")]` - 表名映射
- ❌ `[ForeignKey]` / `[InverseProperty]` - 外键关系
- ❌ `[Index]` - 索引定义
- ❌ 其他任何 `System.ComponentModel.DataAnnotations` 或 `System.ComponentModel.DataAnnotations.Schema` 命名空间下的特性

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
- **长度限制**：字符串类型数据，默认不需要配置 MaxLength 长度。
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
    /// 创建时间
    /// </summary>
    public DateTime create_time { get; set; }
}
```

#### ❌ 错误示例（使用数据注解 - 严禁使用）

```csharp
/// <summary>
/// 用户信息模型
/// </summary>
[Table("t_user_info")]  // ❌ 禁止使用
public class UserInfoModel
{
    /// <summary>
    /// 主键ID
    /// </summary>
    [Key]  // ❌ 禁止使用
    [Column("id")]  // ❌ 禁止使用
    public long Id { get; set; }

    /// <summary>
    /// 用户名
    /// </summary>
    [Required]  // ❌ 禁止使用
    [MaxLength(50)]  // ❌ 禁止使用
    [Column("user_name")]  // ❌ 禁止使用
    public string UserName { get; set; }

    /// <summary>
    /// 创建时间
    /// </summary>
    [Column("create_time")]  // ❌ 禁止使用
    public DateTime CreateTime { get; set; }
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
            .HasMaxLength(50)  // 长度限制（替代 [MaxLength(50)]）
            .HasComment("用户名");

        builder.Property(x => x.email)
            .IsRequired()
            .HasMaxLength(100)
            .HasComment("邮箱地址");

        builder.Property(x => x.create_time)
            .HasDefaultValueSql("CURRENT_TIMESTAMP")  // 默认值
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

**通过 Fluent API 可以实现的所有配置包括：**

- `ToTable("表名")` - 配置表名
- `HasComment("注释")` - 表/字段注释
- `HasKey(x => x.id)` - 配置主键
- `Property(x => x.field)` - 配置属性
- `HasColumnName("列名")` - 配置列名（替代 [Column]）
- `IsRequired()` - 配置非空约束（替代 [Required]）
- `HasMaxLength(n)` - 配置最大长度（替代 [MaxLength]）
- `HasDefaultValueSql("SQL表达式")` - 配置默认值
- `HasIndex(x => x.field)` - 配置索引（替代 [Index]）
- `IsUnique()` - 配置唯一索引
- `HasDatabaseName("索引名")` - 配置索引名称

### 4. 代码质量规范

- **避免神秘命名**：属性命名必须清晰表达业务含义（例如使用 `create_time` 而非 `time`）。
- **文档注释**：实体类、属性、配置类均需添加 XML 文档注释（`/// <summary>`）。
- **数据库注释**：必须在 Fluent API 中使用 `HasComment` 为表和字段添加注释。

### 5. 注册配置到 DbContext

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

## 工作流程

### 创建 API 接口时：

1. 确认 Controller 的业务领域和路由前缀
2. 应用标准属性配置（Route、ApiController、ApiExplorerSettings、SwaggerTag、Authorize）
3. 实现 API 接口方法
4. 添加必要的 XML 文档注释

### 创建数据库实体时：

1. 创建以 `Model` 结尾的实体类
2. 添加属性（使用小写命名）
3. 创建实现 `IEntityTypeConfiguration<T>` 的配置类
4. 在 `Configure` 方法中使用 Fluent API 定义表结构
5. 添加完整的 XML 文档注释和数据库注释
6. 在 DbContext 中注册配置

### 集成缓存功能（可选）：

**当业务实体需要缓存支持时**（如字典数据、配置信息、高频查询数据），使用 `net-cache-use` 技能为该实体添加完整的缓存功能：

**适用场景：**

- 字典数据（如角色、权限、配置项等参考数据）
- 高频查询的业务实体（如用户信息、组织架构）
- 需要批量查询优化的场景
- 需要提升 API 响应速度的场景

**缓存集成流程：**

1. 判断实体是否需要缓存支持（根据查询频率和数据变更频率）
2. 如需缓存，调用 `net-cache-use` 技能
3. 该技能会生成完整的缓存代码（View、RedisHandler、CacheManager）
4. 在 API 的 Service 层中注入和使用缓存接口

**调用示例：**

```
需要为实体 "Department"（部门）添加缓存功能，请使用 net-cache-use 技能生成缓存代码。
- 实体名称：Department
- 主键类型：long
- 数据库表名：t_department
- TTL：24小时
```

## 输出要求

- 代码必须符合上述规范模板
- 命名必须清晰、一致
- 必须包含完整的文档注释
- 不得省略必要的配置步骤
- 不得擅自添加未要求的额外功能
