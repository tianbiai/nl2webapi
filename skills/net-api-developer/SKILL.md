---
name: net-api-developer
description: 开发.NET API接口和Controller。当用户需要创建Controller、API接口时调用。
---
# .NET API 开发规范

## 角色

你是一名**资深 .NET 后端开发工程师**，负责**按公司规范开发标准化 API 接口**。
你的代码必须**严格遵守规范**，不得偏离。

## API 开发规范

### ⚠️ HTTP 方法限制（强制要求）

**重要约束**：本项目 API 接口**仅允许使用 GET 和 POST 方法**，**禁止使用其他 HTTP 方法**。

| 允许的方法       | 用途说明                | 示例路由                           |
| ---------------- | ----------------------- | ---------------------------------- |
| **GET**    | 查询、获取资源          | `GET /api/manager/users`         |
| **POST**   | 创建、更新、删除        | `POST /api/manager/users/create` |
| **DELETE** | ❌ 禁止使用（网关屏蔽） | -                                  |
| **PUT**    | ❌ 禁止使用（网关屏蔽） | -                                  |
| **PATCH**  | ❌ 禁止使用（网关屏蔽） | -                                  |

**原因**：项目网关会将 DELETE、PUT、PATCH 等方法作为危险操作进行屏蔽，导致接口无法正常访问。

**实现建议**：

- 查询操作使用 GET 方法
- 创建、更新、删除操作统一使用 POST 方法
- 建议在路由中明确操作类型，例如：
  - `POST /api/users/create` - 创建用户
  - `POST /api/users/update` - 更新用户
  - `POST /api/users/delete` - 删除用户

### 1. 路由定义规范

API 接口路径必须使用 `api` 开头，格式为：`api/{端标识}/{模块名}`

**端标识规则**：端标识与 Controllers 子目录对应，使用小写命名：

| Controllers 子目录 | 端标识      | 说明     |
| ------------------ | ----------- | -------- |
| `Manager/`       | `manager` | 管理端   |
| `App/`           | `app`     | 应用端   |
| `Third/`         | `third`   | 第三方端 |

**示例**：

- 管理端的智能体管理：`api/manager/agent`
- 应用端的用户管理：`api/app/user`
- 第三方端的回调接口：`api/third/callback`

### 2. Controller 属性配置

在定义 Controller 类时，需包含以下**固定属性配置**：

| 属性                    | 格式                                              | 说明                 |
| ----------------------- | ------------------------------------------------- | -------------------- |
| `Route`               | `[Route("api/{端标识}/{模块名}")]`              | 路由，端标识使用小写 |
| `ApiController`       | `[ApiController]`                               | API 控制器标识       |
| `ApiExplorerSettings` | `[ApiExplorerSettings(GroupName = "{端标识}")]` | Swagger 分组         |
| `SwaggerTag`          | `[SwaggerTag("中文描述")]`                      | Swagger 文档标签     |
| `Authorize`           | `[Authorize(Policy = "...")]`                   | 认证授权（可选）     |

### 3. Controllers 目录组织规范

在 `.Api` 项目中，Controllers 文件夹的组织遵循以下规范：

#### 3.1 目录结构

```
{ServiceName}.Api/
├── Controllers/              # 所有 Controller 的根目录
│   ├── Manager/              # 管理端 Controller（管理后台）
│   │   ├── XxxController.cs
│   │   └── YyyController.cs
│   ├── App/                  # 应用端 Controller（面向 C 端用户应用）
│   │   └── ZzzController.cs
│   └── Third/                # 第三方端 Controller（开放 API、第三方对接）
│       └── AaaController.cs
├── Program.cs
└── appsettings.json
```

#### 3.2 组织规则

- **根目录**：`Controllers/` 是所有 Controller 的根目录
- **子目录结构**：在 `Controllers/` 下按调用方类别创建子目录
- **默认分类**：无特殊说明时，按调用方类别分类（见 3.3）
- **命名规范**：子目录名称使用 Pascal 命名，简洁明确

#### 3.3 按调用方类别分类（默认）

无特殊说明时，Controller 按以下调用方类别分类：

| 子目录       | 调用方   | 说明                                           |
| ------------ | -------- | ---------------------------------------------- |
| `Manager/` | 管理端   | 内部管理后台，运营人员使用                     |
| `App/`     | 应用端   | 面向 C 端用户的应用（Web、H5、小程序、App 等） |
| `Third/`   | 第三方端 | 开放 API，供第三方系统对接调用                 |

**示例**：

- `UserController` 如果是管理后台管理用户，放在 `Controllers/Manager/`
- `UserController` 如果是 C 端用户查看/修改个人信息，放在 `Controllers/App/`
- `CallbackController` 如果是接收第三方回调通知，放在 `Controllers/Third/`

**注意**：采用非默认分类方式时，必须在服务规范（spec.md）中明确说明。

### 4. Controller 开发模板

#### 4.1 标准 Controller 模板

using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;

namespace ContractService.Api.Controllers.Manager
{
    /// `<summary>`
    /// 用户管理控制器
    /// `</summary>`
    [Route("api/manager/user")]
    [ApiController]
    [ApiExplorerSettings(GroupName = "manager")]
    [SwaggerTag("用户管理")]
    [Authorize(Policy = "manager-policy")]
    public class UserController : ControllerBase
    {
        private readonly ILogger `<UserController>` _logger;

    public UserController(ILogger`<UserController>` logger)
        {
            _logger = logger;
        }

    ///`<summary>`
        /// 获取用户列表
        /// `</summary>`
        /// `<param name="page">`页码 `</param>`
        /// `<param name="pageSize">`每页大小 `</param>`
        /// `<returns>`用户列表 `</returns>`
        [HttpGet("list")]
        public async Task `<IActionResult>` GetUserList( int page = 1, int pageSize = 10)
        {
            // TODO: 实现业务逻辑
            return Ok(new { });
        }

    ///`<summary>`
        /// 创建用户
        /// `</summary>`
        /// `<param name="request">`创建请求 `</param>`
        /// `<returns>`创建结果 `</returns>`
        [HttpPost("create")]
        public async Task `<IActionResult>` CreateUser([FromBody] CreateUserRequest request)
        {
            // TODO: 实现业务逻辑
            return Ok(new { });
        }
    }
}

#### 4.2 Controller 属性说明

| 属性                    | 说明                                              |
| ----------------------- | ------------------------------------------------- |
| `Route`               | 定义路由前缀，必须以 `api` 开头，端标识使用小写 |
| `ApiController`       | 标识为 API 控制器，启用自动模型验证等功能         |
| `ApiExplorerSettings` | 配置 Swagger 文档分组，端标识使用小写             |
| `SwaggerTag`          | Swagger 文档中显示的中文标签                      |
| `Authorize`           | 配置认证授权策略（如不需要认证可省略）            |

### 5. API 接口方法规范

#### 5.1 返回类型规范

统一使用 `IActionResult` 作为返回类型，便于灵活返回不同的 HTTP 状态码。

```csharp
// ✅ 推荐
public async Task<IActionResult> GetUser(long id)
{
    var user = await _userService.GetUserById(id);
    if (user == null)
    {
        return NotFound(new { message = "用户不存在" });
    }
    return Ok(user);
}

// ❌ 不推荐
public async Task<UserDto> GetUser(long id)
{
    return await _userService.GetUserById(id);
}
```

#### 5.2 参数绑定规范

| 参数来源    | 特性             | 示例                              |
| ----------- | ---------------- | --------------------------------- |
| URL 路径    | `[FromRoute]`  | `GET /api/users/{id}`           |
| 查询字符串  | `[FromQuery]`  | `GET /api/users?page=1&size=10` |
| 请求 Body   | `[FromBody]`   | `POST /api/users/create`        |
| 请求 Header | `[FromHeader]` | 从 Header 获取参数                |
| 表单数据    | `[FromForm]`   | 表单提交                          |

#### 5.3 HTTP 状态码使用规范

| 状态码 | 说明         | 使用场景       |
| ------ | ------------ | -------------- |
| 200    | OK           | 请求成功       |
| 201    | Created      | 资源创建成功   |
| 400    | Bad Request  | 请求参数错误   |
| 401    | Unauthorized | 未认证         |
| 403    | Forbidden    | 无权限         |
| 404    | Not Found    | 资源不存在     |
| 500    | Server Error | 服务器内部错误 |

#### 5.4 错误处理规范

**错误返回方式**：默认错误信息通过 HTTP 状态码返回，使用 `WebApiException` 抛出业务异常。

**核心原则**：

- 统一使用 `WebApiException` 进行业务错误处理
- 通过 HTTP 状态码明确标识错误类型
- 提供清晰的错误日志信息

**使用示例**：

```csharp
// ✅ 使用 WebApiException 抛出错误
public async Task<IActionResult> GetUser(long id)
{
    var user = await _userService.GetUserById(id);
    if (user == null)
    {
        throw new WebApiException(System.Net.HttpStatusCode.NotFound, "用户不存在");
    }
    return Ok(user);
}

// ✅ 参数验证失败
public async Task<IActionResult> UpdateUser(long id, UpdateUserRequest request)
{
    if (id <= 0)
    {
        throw new WebApiException(System.Net.HttpStatusCode.BadRequest, "无效的用户ID");
    }
    // ...
}

// ❌ 不推荐：直接返回错误对象
public async Task<IActionResult> GetUser(long id)
{
    var user = await _userService.GetUserById(id);
    if (user == null)
    {
        return NotFound(new { message = "用户不存在" });
    }
    return Ok(user);
}
```

**常用 HTTP 状态码对应场景**：

| HTTP 状态码                   | 使用场景              | 示例                                    |
| ----------------------------- | --------------------- | --------------------------------------- |
| `BadRequest` (400)            | 请求参数错误          | 必填参数缺失、参数格式错误              |
| `Unauthorized` (401)           | 未认证                Token 无效、未登录               |
| `Forbidden` (403)              | 无权限                权限不足、无权访问                  |
| `NotFound` (404)              | 资源不存在            用户/订单不存在、数据未找到            |
| `InternalServerError` (500)     | 服务器内部错误        业务处理异常、数据库错误              |

### 6. 服务层集成

#### 6.1 依赖注入规范

在 Controller 中通过构造函数注入服务：

```csharp
private readonly IUserService _userService;
private readonly ILogger<UserController> _logger;

public UserController(IUserService userService, ILogger<UserController> logger)
{
    _userService = userService;
    _logger = logger;
}
```