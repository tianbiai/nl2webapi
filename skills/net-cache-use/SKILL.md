---
name: net-cache-use
description: 为业务实体添加完整的缓存功能，生成符合三层架构的缓存代码，包括读取、刷新、批量查询等功能。
---
# 技能：缓存类库创建与使用

## 技能描述

根据项目缓存架构规范，为新的业务实体添加完整的缓存功能。该技能会生成符合三层架构（Interface、Cache、Data）的缓存代码，包括读取、刷新、批量查询等功能。

## 适用场景

当需要为新的数据库表或视图添加缓存功能时使用此技能：

- 添加新的业务实体的缓存支持
- 为现有实体添加缓存层
- 实现高性能的字典查询
- 支持批量查询优化

## 执行步骤

### 步骤 1：收集实体信息

在生成代码之前，需要收集以下信息：

**必需信息：**

1. **实体名称**（英文）：如 `User`、`Role`、`MyEntity`
2. **主键类型**：通常是 `long` 或 `string`
3. **数据库表名**：PostgreSQL 表名，如 `t_user`、`t_role`
4. **字段列表**：实体包含的所有字段及其类型

**可选信息：**
5. **唯一索引字段**：除主键外的唯一查询字段（如手机号、账号）
6. **TTL 时长**：缓存过期时间（10分钟、2小时、8小时、24小时）
7. **是否需要嵌套结构**：是否需要树形结构或复杂关联
8. **自定义查询条件**：是否需要 JOIN、WHERE 等复杂查询

### 步骤 2：生成缓存模型类

在 `View/` 目录下创建 `{Entity}Cache.cs`：

```csharp
// 模板：View/{Entity}Cache.cs
namespace FutureCommunity.Cache.View
{
    /// <summary>
    /// {实体中文名}缓存
    /// </summary>
    public class {Entity}Cache
    {
        /// <summary>
        /// 主键ID
        /// </summary>
        public {KeyType} id { get; set; }

        /// <summary>
        /// {字段说明}
        /// </summary>
        public {FieldType} {field_name} { get; set; }

        // ... 其他字段
    }
}
```

**命名规则：**

- 类名：`{Entity}Cache`（如 `UserCache`、`RoleCache`）
- 属性名：使用小写（遵循数据库命名）
- 必须有 `id` 属性（如果需要字典访问）

### 步骤 3：实现 RedisHandler 查询方法

在 `RedisHandler.cs` 中添加查询方法：

```csharp
#region {实体中文名}相关

/// <summary>
/// 获取单个{实体}
/// </summary>
public async Task<{Entity}Cache> Get{Entity}({KeyType} id)
{
    var dbcontext = new CacheViewDBContext(_dboption);
    var sql = @"SELECT * FROM public.t_{entity} WHERE id = {0}";
    return await dbcontext.Database
        .SqlQueryRaw<{Entity}Cache>(sql, id)
        .AsNoTracking()
        .FirstOrDefaultAsync();
}

/// <summary>
/// 获取{实体}字典
/// </summary>
public async Task<Dictionary<{KeyType}, {Entity}Cache>> Get{Entity}List()
{
    var dbcontext = new CacheViewDBContext(_dboption);
    var sql = @"SELECT * FROM public.t_{entity}";
    var list = await dbcontext.Database
        .SqlQueryRaw<{Entity}Cache>(sql)
        .AsNoTracking()
        .ToListAsync();
    return list.ToDictionary(f => f.id, f => f);
}

#endregion
```

**查询方式说明：**

- 使用 `Database.SqlQueryRaw<T>()` 执行原生 SQL 查询
- 使用 `{0}` 占位符传递参数，防止 SQL 注入
- 所有查询必须使用 `AsNoTracking()` 以提高性能
- 对于复杂查询（JOIN、WHERE 等），直接修改 SQL 语句即可

### 步骤 4：实现 CacheManager 读取方法

在 `CacheManager.cs` 的 Reader 部分添加：

```csharp
#region Reader - {实体中文名}相关

/// <summary>
/// 获取单个{实体}
/// </summary>
public async Task<{Entity}Cache> Get{Entity}Info({KeyType} id)
{
    string key = $"{entity}.{id}";
    return await GetSingle(key, () => reader.Get{Entity}(id), {TTL});
}

/// <summary>
/// 获取{实体}字典
/// </summary>
public async Task<Dictionary<{KeyType}, {Entity}Cache>> Get{Entity}Dic()
{
    var key = $"{entity}";
    return await GetSingle(key, reader.Get{Entity}List, {TTL});
}

/// <summary>
/// 批量获取{实体}
/// </summary>
public async Task<Dictionary<{KeyType}, {Entity}Cache>> Get{Entity}Info(List<{KeyType}> ids)
{
    string key = "{entity}.";
    var dic = await GetMultiple(
        ids.Distinct().Select(s => $"{key}{s}").ToArray(),
        func,
        {DateTimeTTL}
    );
    return dic.ToDictionary(f => {KeyTypeParse}(f.Key.Replace(key, "")), v => v.Value);

    async Task<IDictionary<string, {Entity}Cache>> func(string[] keys)
    {
        var ids = keys.Select(s => {KeyTypeParse}(s.Replace(key, ""))).ToList();
        var dbcontext = new CacheViewDBContext(viewDbOp);
        var sql = @"SELECT * FROM public.t_{entity} WHERE id = ANY({0})";
        var list = await dbcontext.Database
            .SqlQueryRaw<{Entity}Cache>(sql, ids)
            .AsNoTracking()
            .ToListAsync();
        return list.ToDictionary(f => $"{key}{f.id}");
    }
}

#endregion
```

**TTL 选择（根据数据特性）：**

- `_stime010` (10分钟)：临时会话数据
- `_stime2` (2小时)：外部 API Token
- `_stime8` (8小时)：用户相关数据（房屋、住户、积分）
- `_stime24` (24小时)：大部分参考数据（字典、配置、角色）

**KeyTypeParse：**

- `long`：`long.Parse`
- `string`：无需转换
- 其他：相应转换方法

### 步骤 5：实现 CacheManager 刷新方法

在 `CacheManager.cs` 的 Refresh 部分添加：

```csharp
#region Refresh - {实体中文名}相关

/// <summary>
/// 刷新单个{实体}
/// </summary>
public async Task Refresh{Entity}({KeyType} id)
{
    string key = $"{entity}.{id}";
    var info = await reader.Get{Entity}(id);
    await AddOrUpdate(key, info, {TTL});
}

/// <summary>
/// 刷新整个{实体}集合
/// </summary>
public async Task Refresh{Entity}()
{
    var key = $"{entity}";
    var dic = await reader.Get{Entity}List();
    await AddOrUpdate(key, dic, {TTL});
}

/// <summary>
/// 刷新特定{实体}（使用模型）
/// </summary>
public async Task Refresh{Entity}({Entity}Cache model)
{
    string key = $"{entity}.{model.id}";
    await AddOrUpdate(key, model, {TTL});
}

#endregion
```

**刷新场景：**

- 新增实体后：调用 `Refresh{Entity}(model)`
- 更新实体后：调用 `Refresh{Entity}(id)` 或 `Refresh{Entity}(model)`
- 删除实体后：调用 `RemoveSingle(key)` 或 `Refresh{Entity}()`
- 批量变更后：调用 `Refresh{Entity}()`

## 代码审查清单

完成缓存代码后，使用以下清单进行自查：

### 文件结构

- [ ] 在 `View/` 创建了 `{Entity}Cache.cs`
- [ ] 在 `RedisHandler.cs` 添加了查询方法
- [ ] 在 `CacheManager.cs` 添加了 Reader 和 Refresh 方法
- [ ] 在 `ICacheReader.cs` 添加了接口定义
- [ ] 在 `ICacheRefresh.cs` 添加了接口定义

### 代码质量

- [ ] 缓存键命名符合规范（小写、点号分隔）
- [ ] 使用了正确的 TTL（根据数据特性）
- [ ] 所有查询使用了 `AsNoTracking()`
- [ ] 多键场景更新了所有相关键
- [ ] 添加了完整的 XML 注释

### 性能优化

- [ ] 批量查询使用了 PostgreSQL `ANY` 数组参数（而非循环）
- [ ] 所有查询使用 `Database.SqlQueryRaw<T>()` 执行原生 SQL
- [ ] 大批量数据考虑了分批处理

## 使用示例

### 调用技能的方式

当需要添加新缓存实体时，按以下格式提供信息：

```
请添加缓存实体：
- 实体名称：Department
- 中文名称：部门
- 主键类型：long
- 数据库表名：t_department
- 字段列表：
  - name (string) - 部门名称
  - state (int) - 状态
  - add_time (DateTime) - 添加时间
- TTL：24小时
- 需要：单个查询、字典查询、批量查询
```

### 生成的代码使用

```csharp
// 在业务代码中使用
public class DepartmentService
{
    private readonly ICacheReader _cacheReader;
    private readonly ICacheRefresh _cacheRefresh;

    // 获取单个部门
    public async Task<DepartmentCache> GetDepartment(long id)
    {
        return await _cacheReader.GetDepartmentInfo(id);
    }

    // 获取所有部门字典
    public async Task<Dictionary<long, DepartmentCache>> GetAllDepartments()
    {
        return await _cacheReader.GetDepartmentDic();
    }

    // 批量获取部门
    public async Task<Dictionary<long, DepartmentCache>> GetDepartments(List<long> ids)
    {
        return await _cacheReader.GetDepartmentInfo(ids);
    }

    // 更新部门
    public async Task UpdateDepartment(DepartmentCache dept)
    {
        // 1. 更新数据库
        await _dbcontext.SaveChangesAsync();

        // 2. 刷新缓存
        await _cacheRefresh.RefreshDepartment(dept);
    }
}
```

## 注意事项

### ⚠️ 关于数据注解的特殊说明

**重要原则**：本项目严格禁止在实体模型中使用数据验证相关的注解（如 `[Required]`、`[MaxLength]`、`` `[Key]` 等），所有数据库配置必须通过 Fluent API 实现。

**例外情况**：`[DbBulk]` 特性是批量操作框架（`PostgresqlAsyncBulk`）的特殊要求，用于：

- 标记属性是否参与批量操作（`Ignore`）
- 指定数据库列名映射（`ColumnName`）
- 定义数据库字段类型（`Type`）
- 处理特殊类型（`UnknownType`）

`[DbBulk]` 特性**不是数据验证注解**，而是批量操作的元数据标记，可以安全使用。

### 必须遵循的规范

1. **命名规范**

   - 缓存键必须小写，使用点号分隔
   - 类名以 `Cache` 结尾
   - 属性名使用小写（与数据库一致）
2. **性能规范**

   - 必须使用 `AsNoTracking()`
   - 批量查询使用 IN 语句
   - 避免循环查询
3. **一致性规范**

   - 数据变更后立即刷新缓存
   - 多键场景更新所有相关键
   - 使用正确的 TTL
