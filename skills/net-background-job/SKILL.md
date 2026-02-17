---
name: net-background-job
description: 根据业务需求生成符合BackgroundRunner规范的后台定时任务代码，包含任务配置、条件检查及核心逻辑实现。
---
# 概述

BackgroundRunner 提供了完整的任务生命周期管理、异常处理和日志记录功能。

# 主要特性

- 循环执行任务，可配置休眠间隔
- 支持 ASP.NET Core 依赖注入
- 内置异常捕获和日志记录
- 支持两种日志记录方式（标准日志和后台专用日志）
- 支持优雅停止和取消操作
- 可配置任务执行条件

# 类结构

## 核心属性

| 属性      | 类型    | 默认值          | 说明                             |
| --------- | ------- | --------------- | -------------------------------- |
| SleepTime | int     | 3600000 (1小时) | 每次任务执行后的休眠时间（毫秒） |
| Name      | string  | null            | 任务名称，用于日志标识           |
| Check     | bool    | -               | 抽象属性，判断是否执行任务       |
| Logger    | ILogger | -               | 标准日志记录器                   |

## 核心方法

| 方法         | 类型     | 说明                           |
| ------------ | -------- | ------------------------------ |
| WorkAsync    | 抽象方法 | 需要子类实现，定义具体任务逻辑 |
| ExecuteAsync | 重写方法 | 后台服务主执行逻辑             |
| StartAsync   | 重写方法 | 服务启动时的日志记录           |
| StopAsync    | 重写方法 | 服务停止时的日志记录           |

# 使用步骤

## 1. 创建自定义任务类

任务类继承 BackgroundRunner 并实现抽象成员，示例：

```csharp
using Microsoft.Extensions.Logging;

public class MyBackgroundTask : BackgroundRunner
{
    private readonly IMyService _service;

    public MyBackgroundTask(ILogger<MyBackgroundTask> logger, IMyService service) 
        : base(logger)
    {
        _service = service;
        Name = "我的后台任务";
        SleepTime = 60000; // 1分钟执行一次
    }

    public override bool Check
    {
        get
        {
            // 根据业务逻辑判断是否执行任务
            return DateTime.Now.Hour >= 9 && DateTime.Now.Hour <= 18;
        }
    }

    public override async Task WorkAsync(CancellationToken cancellationToken)
    {
        // 实现具体任务逻辑
        Logger.LogInformation("开始执行任务");
        await _service.DoWorkAsync(cancellationToken);
        Logger.LogInformation("任务执行完成");
    }
}
```

## 2. 注册服务

在 Program.cs 或 Startup.cs 中注册：

```csharp
builder.Services.AddHostedService<MyBackgroundTask>();
```

# 注意事项

- 任务会在应用启动后自动开始
- 任务在应用关闭时优雅停止
- 即使发生异常，任务也会继续循环执行
- 如需永久停止任务，需要重新部署或修改 Check 属性
- 确保在 WorkAsync 中正确处理异步操作
