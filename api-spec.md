# 三网科技驾驶舱大屏系统 - API 接口清单

**版本**：v1.0
**基础路径**：`/api/v1`
**日期**：2026-02-17

---

## 接口概览

| 模块 | 接口数量 | 说明 |
|------|---------|------|
| 公共接口 | 2 | 系统信息、心跳检测 |
| 核心指标 | 1 | 驾驶舱总览数据 |
| 智慧消防 | 3 | 消防设备监控 |
| 智慧养老 | 2 | 养老服务数据 |
| 智慧交通 | 2 | 交通监控数据 |
| 智慧园区 | 2 | 园区管理数据 |
| 报警中心 | 2 | 报警信息管理 |

---

## 通用说明

### 请求头

```
Content-Type: application/json
Authorization: Bearer {token}  // 可选，根据安全需求
```

### 响应格式

```json
{
  "code": 200,           // 状态码：200成功，其他为错误
  "message": "success",  // 响应消息
  "data": {},            // 响应数据
  "timestamp": 1708156800000  // 时间戳
}
```

### 错误码说明

| 错误码 | 说明 |
|-------|------|
| 200 | 成功 |
| 400 | 请求参数错误 |
| 401 | 未授权 |
| 403 | 禁止访问 |
| 404 | 资不存在 |
| 500 | 服务器内部错误 |
| 503 | 服务不可用 |

---

## 一、公共接口

### 1.1 获取系统信息

获取系统基础信息和配置。

**请求**
```
GET /api/v1/system/info
```

**响应**
```json
{
  "code": 200,
  "data": {
    "systemName": "三网科技物联网驾驶舱",
    "version": "1.0.0",
    "company": {
      "name": "浙江三网科技股份有限公司",
      "logo": "https://xxx.com/logo.png",
      "description": "国家级高新技术企业"
    },
    "modules": ["fire", "elderly", "traffic", "park"]
  }
}
```

### 1.2 心跳检测

检测服务是否正常运行。

**请求**
```
GET /api/v1/system/health
```

**响应**
```json
{
  "code": 200,
  "data": {
    "status": "healthy",
    "uptime": 86400,
    "timestamp": 1708156800000
  }
}
```

---

## 二、核心指标

### 2.1 获取驾驶舱总览数据

获取驾驶舱首页核心指标数据。

**请求**
```
GET /api/v1/cockpit/overview
```

**参数**
| 参数名 | 类型 | 必填 | 说明 |
|-------|------|------|------|
| area | string | 否 | 区域代码，默认全部 |

**响应**
```json
{
  "code": 200,
  "data": {
    "totalDevices": 356892,           // 设备总数
    "onlineRate": 96.8,               // 在线率（%）
    "todayAlarms": 156,               // 今日报警数
    "dataProcessed": "2.8TB",         // 数据处理量
    "deviceTrend": [                  // 设备趋势（近7天）
      { "date": "2026-02-11", "count": 355000 },
      { "date": "2026-02-12", "count": 355800 },
      { "date": "2026-02-13", "count": 356000 },
      { "date": "2026-02-14", "count": 356200 },
      { "date": "2026-02-15", "count": 356400 },
      { "date": "2026-02-16", "count": 356700 },
      { "date": "2026-02-17", "count": 356892 }
    ],
    "areaDistribution": [             // 区域分布
      { "area": "杭州", "deviceCount": 89000, "alarmCount": 45 },
      { "area": "宁波", "deviceCount": 72000, "alarmCount": 38 },
      { "area": "温州", "deviceCount": 65000, "alarmCount": 32 },
      { "area": "南京", "deviceCount": 58000, "alarmCount": 25 },
      { "area": "其他", "deviceCount": 72892, "alarmCount": 16 }
    ]
  }
}
```

---

## 三、智慧消防模块

### 3.1 获取消防设备状态

获取消防设备统计数据。

**请求**
```
GET /api/v1/fire/device-status
```

**参数**
| 参数名 | 类型 | 必填 | 说明 |
|-------|------|------|------|
| area | string | 否 | 区域代码 |

**响应**
```json
{
  "code": 200,
  "data": {
    "total": 12580,           // 设备总数
    "online": 12045,          // 在线数量
    "offline": 535,           // 离线数量
    "alarm": 12,              // 报警数量
    "onlineRate": 95.75,      // 在线率
    "deviceTypes": [          // 设备类型分布
      { "type": "烟感", "count": 5000, "percentage": 39.7 },
      { "type": "温感", "count": 3000, "percentage": 23.9 },
      { "type": "消防栓", "count": 2500, "percentage": 19.9 },
      { "type": "喷淋", "count": 1500, "percentage": 11.9 },
      { "type": "其他", "count": 580, "percentage": 4.6 }
    ]
  }
}
```

### 3.2 获取报警趋势

获取消防报警趋势数据。

**请求**
```
GET /api/v1/fire/alarm-trend
```

**参数**
| 参数名 | 类型 | 必填 | 说明 |
|-------|------|------|------|
| period | string | 否 | 时间周期：day(24小时)/week(7天)/month(30天)，默认day |
| area | string | 否 | 区域代码 |

**响应**
```json
{
  "code": 200,
  "data": {
    "period": "day",
    "trend": [
      { "time": "00:00", "count": 5 },
      { "time": "01:00", "count": 3 },
      { "time": "02:00", "count": 2 },
      { "time": "03:00", "count": 1 },
      { "time": "04:00", "count": 2 },
      { "time": "05:00", "count": 4 },
      { "time": "06:00", "count": 6 },
      { "time": "07:00", "count": 8 },
      { "time": "08:00", "count": 12 },
      { "time": "09:00", "count": 15 },
      { "time": "10:00", "count": 18 },
      { "time": "11:00", "count": 14 },
      { "time": "12:00", "count": 10 },
      { "time": "13:00", "count": 12 },
      { "time": "14:00", "count": 15 },
      { "time": "15:00", "count": 13 },
      { "time": "16:00", "count": 11 },
      { "time": "17:00", "count": 14 },
      { "time": "18:00", "count": 16 },
      { "time": "19:00", "count": 12 },
      { "time": "20:00", "count": 10 },
      { "time": "21:00", "count": 8 },
      { "time": "22:00", "count": 6 },
      { "time": "23:00", "count": 5 }
    ],
    "total": 198,
    "avgPerHour": 8.25
  }
}
```

### 3.3 获取区域分布

获取消防设备区域分布数据。

**请求**
```
GET /api/v1/fire/area-distribution
```

**响应**
```json
{
  "code": 200,
  "data": {
    "areas": [
      { "area": "商业区", "count": 3500, "alarmCount": 45, "level": "high" },
      { "area": "住宅区", "count": 5200, "alarmCount": 28, "level": "medium" },
      { "area": "工业区", "count": 2800, "alarmCount": 35, "level": "high" },
      { "area": "学校", "count": 1080, "alarmCount": 4, "level": "low" }
    ],
    "summary": {
      "highLevel": 2,
      "mediumLevel": 1,
      "lowLevel": 1
    }
  }
}
```

---

## 四、智慧养老模块

### 4.1 获取养老统计数据

获取养老服务核心统计数据。

**请求**
```
GET /api/v1/elderly/statistics
```

**参数**
| 参数名 | 类型 | 必填 | 说明 |
|-------|------|------|------|
| date | string | 否 | 日期，格式yyyy-MM-dd，默认今天 |

**响应**
```json
{
  "code": 200,
  "data": {
    "serviceCount": 2856,           // 今日服务次数
    "activeUsers": 12890,           // 活跃用户数
    "totalUsers": 25000,            // 总用户数
    "healthAlerts": 23,             // 健康预警数
    "satisfaction": 98.5,           // 满意度（%）
    "serviceTypes": [               // 服务类型分布
      { "type": "健康监测", "count": 1200, "percentage": 42.0 },
      { "type": "生活照料", "count": 800, "percentage": 28.0 },
      { "type": "精神慰藉", "count": 500, "percentage": 17.5 },
      { "type": "紧急救援", "count": 56, "percentage": 2.0 },
      { "type": "其他", "count": 300, "percentage": 10.5 }
    ]
  }
}
```

### 4.2 获取服务趋势

获取养老服务趋势数据。

**请求**
```
GET /api/v1/elderly/service-trend
```

**参数**
| 参数名 | 类型 | 必填 | 说明 |
|-------|------|------|------|
| months | int | 否 | 月数，默认6 |

**响应**
```json
{
  "code": 200,
  "data": {
    "trend": [
      { "month": "2025-09", "serviceCount": 45680, "newUsers": 520 },
      { "month": "2025-10", "serviceCount": 48920, "newUsers": 680 },
      { "month": "2025-11", "serviceCount": 52300, "newUsers": 750 },
      { "month": "2025-12", "serviceCount": 56780, "newUsers": 890 },
      { "month": "2026-01", "serviceCount": 54120, "newUsers": 620 },
      { "month": "2026-02", "serviceCount": 51800, "newUsers": 580 }
    ],
    "totalService": 309600,
    "avgMonthly": 51600
  }
}
```

---

## 五、智慧交通模块

### 5.1 获取交通统计数据

获取交通监控核心数据。

**请求**
```
GET /api/v1/traffic/statistics
```

**参数**
| 参数名 | 类型 | 必填 | 说明 |
|-------|------|------|------|
| date | string | 否 | 日期，默认今天 |

**响应**
```json
{
  "code": 200,
  "data": {
    "vehicleFlow": 125680,          // 今日车流量
    "congestionIndex": 2.3,         // 拥堵指数（1-5）
    "congestionLevel": "基本畅通",   // 拥堵等级描述
    "parkingUsage": 78.5,           // 停车场使用率（%）
    "parkingTotal": 15000,          // 停车位总数
    "parkingUsed": 11775,           // 已使用车位数
    "violations": 234,              // 今日违章数
    "accidents": 5                  // 今日事故数
  }
}
```

### 5.2 获取车流趋势

获取车流量趋势数据。

**请求**
```
GET /api/v1/traffic/flow-trend
```

**参数**
| 参数名 | 类型 | 必填 | 说明 |
|-------|------|------|------|
| date | string | 否 | 日期，默认今天 |

**响应**
```json
{
  "code": 200,
  "data": {
    "trend": [
      { "hour": "06:00", "count": 8500 },
      { "hour": "07:00", "count": 12000 },
      { "hour": "08:00", "count": 15000 },
      { "hour": "09:00", "count": 13000 },
      { "hour": "10:00", "count": 12000 },
      { "hour": "11:00", "count": 12500 },
      { "hour": "12:00", "count": 14000 },
      { "hour": "13:00", "count": 12500 },
      { "hour": "14:00", "count": 13000 },
      { "hour": "15:00", "count": 13500 },
      { "hour": "16:00", "count": 16000 },
      { "hour": "17:00", "count": 18000 },
      { "hour": "18:00", "count": 17500 },
      { "hour": "19:00", "count": 14000 },
      { "hour": "20:00", "count": 12000 },
      { "hour": "21:00", "count": 10000 },
      { "hour": "22:00", "count": 8000 }
    ],
    "peakHour": "17:00-18:00",
    "peakCount": 18000
  }
}
```

---

## 六、智慧园区模块

### 6.1 获取园区统计数据

获取园区管理核心数据。

**请求**
```
GET /api/v1/park/statistics
```

**参数**
| 参数名 | 类型 | 必填 | 说明 |
|-------|------|------|------|
| parkId | string | 否 | 园区ID，默认全部 |
| date | string | 否 | 日期，默认今天 |

**响应**
```json
{
  "code": 200,
  "data": {
    "energyConsumption": 1256.8,    // 今日能耗（kWh）
    "energyCost": 895.2,            // 今日能耗费用（元）
    "securityStatus": "normal",     // 安防状态：normal/alert
    "visitorCount": 3256,           // 今日访客数
    "employeeCount": 8500,          // 在职员工数
    "deviceHealth": 99.2,           // 设备健康度（%）
    "totalDevices": 1200,           // 园区设备总数
    "normalDevices": 1190,          // 正常设备数
    "faultDevices": 10              // 故障设备数
  }
}
```

### 6.2 获取能耗趋势

获取园区能耗趋势数据。

**请求**
```
GET /api/v1/park/energy-trend
```

**参数**
| 参数名 | 类型 | 必填 | 说明 |
|-------|------|------|------|
| period | string | 否 | 时间周期：week/month，默认week |
| parkId | string | 否 | 园区ID |

**响应**
```json
{
  "code": 200,
  "data": {
    "period": "week",
    "trend": [
      { "day": "周一", "value": 1150, "cost": 820 },
      { "day": "周二", "value": 1280, "cost": 912 },
      { "day": "周三", "value": 1190, "cost": 848 },
      { "day": "周四", "value": 1320, "cost": 940 },
      { "day": "周五", "value": 1250, "cost": 890 },
      { "day": "周六", "value": 980, "cost": 698 },
      { "day": "周日", "value": 850, "cost": 606 }
    ],
    "total": 8020,
    "avg": 1145.7,
    "peak": "周四"
  }
}
```

---

## 七、报警中心

### 7.1 获取报警列表

获取实时报警信息列表。

**请求**
```
GET /api/v1/alarm/list
```

**参数**
| 参数名 | 类型 | 必填 | 说明 |
|-------|------|------|------|
| module | string | 否 | 模块：fire/elderly/traffic/park，默认全部 |
| level | string | 否 | 级别：high/medium/low |
| status | string | 否 | 状态：pending/processing/resolved |
| page | int | 否 | 页码，默认1 |
| pageSize | int | 否 | 每页条数，默认20 |

**响应**
```json
{
  "code": 200,
  "data": {
    "total": 156,
    "page": 1,
    "pageSize": 20,
    "list": [
      {
        "id": "ALM20260217001",
        "module": "fire",
        "level": "high",
        "status": "pending",
        "title": "烟感报警",
        "location": "杭州大厦A座15层",
        "deviceCode": "DEV001",
        "deviceType": "烟感探测器",
        "content": "检测到烟雾浓度超标",
        "occurredAt": "2026-02-17 14:25:30",
        "createdAt": "2026-02-17 14:25:30"
      },
      {
        "id": "ALM20260217002",
        "module": "elderly",
        "level": "medium",
        "title": "健康预警",
        "location": "西湖区翠苑街道",
        "deviceCode": "DEV002",
        "deviceType": "健康监测手环",
        "content": "用户张XX心率异常",
        "occurredAt": "2026-02-17 14:18:15",
        "createdAt": "2026-02-17 14:18:15"
      }
    ]
  }
}
```

### 7.2 获取报警统计

获取报警统计汇总数据。

**请求**
```
GET /api/v1/alarm/summary
```

**参数**
| 参数名 | 类型 | 必填 | 说明 |
|-------|------|------|------|
| period | string | 否 | 时间周期：today/week/month，默认today |

**响应**
```json
{
  "code": 200,
  "data": {
    "total": 156,
    "byLevel": {
      "high": 23,
      "medium": 68,
      "low": 65
    },
    "byStatus": {
      "pending": 45,
      "processing": 38,
      "resolved": 73
    },
    "byModule": {
      "fire": 56,
      "elderly": 42,
      "traffic": 35,
      "park": 23
    },
    "trend": [
      { "hour": "00:00", "count": 5 },
      { "hour": "04:00", "count": 3 },
      { "hour": "08:00", "count": 12 },
      { "hour": "12:00", "count": 18 },
      { "hour": "16:00", "count": 25 },
      { "hour": "20:00", "count": 15 }
    ]
  }
}
```

---

## 八、WebSocket 实时推送

### 8.1 连接说明

**连接地址**
```
ws://{host}/api/v1/ws
```

**连接参数**
| 参数名 | 类型 | 必填 | 说明 |
|-------|------|------|------|
| token | string | 是 | 认证令牌 |

### 8.2 消息格式

**推送消息格式**
```json
{
  "type": "alarm",          // 消息类型：alarm/data/heartbeat
  "module": "fire",         // 模块
  "action": "create",       // 动作：create/update/delete
  "data": {},               // 数据内容
  "timestamp": 1708156800000
}
```

### 8.3 消息类型

| 类型 | 说明 | 推送频率 |
|------|------|---------|
| alarm | 报警信息 | 实时 |
| overview | 核心指标更新 | 5秒 |
| device | 设备状态变更 | 实时 |
| heartbeat | 心跳检测 | 30秒 |

---

## 九、数据字典

### 9.1 报警级别

| 值 | 说明 | 颜色 |
|----|------|------|
| high | 高危/紧急 | 红色 |
| medium | 中危/警告 | 橙色 |
| low | 低危/提示 | 蓝色 |

### 9.2 报警状态

| 值 | 说明 |
|----|------|
| pending | 待处理 |
| processing | 处理中 |
| resolved | 已解决 |

### 9.3 安防状态

| 值 | 说明 |
|----|------|
| normal | 正常 |
| alert | 警戒 |

### 9.4 模块代码

| 代码 | 说明 |
|------|------|
| fire | 智慧消防 |
| elderly | 智慧养老 |
| traffic | 智慧交通 |
| park | 智慧园区 |

---

## 十、附录

### 10.1 接口调用示例

**JavaScript/axios**
```javascript
import axios from 'axios'

const api = axios.create({
  baseURL: '/api/v1',
  timeout: 10000
})

// 获取核心指标
const getOverview = async () => {
  const { data } = await api.get('/cockpit/overview')
  return data
}

// 获取消防设备状态
const getFireDeviceStatus = async (area) => {
  const { data } = await api.get('/fire/device-status', { params: { area } })
  return data
}
```

### 10.2 Mock 数据配置

开发阶段可使用以下 Mock 配置：

```javascript
// src/api/config.js
export const API_CONFIG = {
  useMock: true,           // 是否使用 Mock 数据
  mockDelay: 300,          // Mock 响应延迟（ms）
  refreshInterval: 5000    // 数据刷新间隔（ms）
}
```

---

**文档版本**：v1.0
**最后更新**：2026-02-17
**维护人员**：前端开发组
