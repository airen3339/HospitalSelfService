# HospitalSelfService
医院自助打印机程序**面向医院门诊场景的自助单据打印终端** — 患者自助完成「身份识别 → 查询单据 → 确认版式 → 打印出纸」全流程，无需在收费窗口排队。
<div align="center">

# 医院自助打印机程序

### Hospital Self-Service Print Terminal

**面向医院门诊场景的自助单据打印终端** — 患者自助完成「身份识别 → 查询单据 → 确认版式 → 打印出纸」全流程，无需在收费窗口排队。

[![build](https://img.shields.io/badge/build-Debug%20%2F%20Release-0%20errors%20%200%20warnings-2ea44f)]()
[![test](https://img.shields.io/badge/tests-98%20passed-2ea44f)]()
[![framework](https://img.shields.io/badge/.NET-Framework%204.7.2-512BD4)]()
[![lang](https://img.shields.io/badge/C%23-8.0-239120)]()
[![db](https://img.shields.io/badge/database-SQLite%20%2F%20SQL%20Server%20%28one%20switch%29-003366)]()
[![license](https://img.shields.io/badge/license-MIT-blue)]()
[![platform](https://img.shields.io/badge/platform-Windows-0078D6)])

</div>

---

## 📖 文档导航

| 我是… | 我想看… |
| :--- | :--- |
| 🏥 **院方决策者** | [产品宣传册](HospitalSelfService/docs/产品宣传册.md) — 痛点、价值、部署形态、实施流程 |
| 🔧 **实施 / 交付人员** | [功能说明文档](HospitalSelfService/docs/功能说明文档.md) — 20 模块 197 功能点、配置手册、故障排查、验收清单 |
| 👨‍💻 **开发 / 维护人员** | 本文档 — 架构、构建、测试、API |
| 📊 **项目管理** | [功能计划进度文档](医院自助打印机程序_功能计划进度文档.md) — WBS、进度表、风险登记 |

---

## 🎯 这是什么

医院门诊高峰期，收费窗口压力大，而很多单据只是「打印一份」—— 这类需求本不该占用人工。

本系统把这条路径完全自动化：

```
患者自助终端                          服务端                        院内系统
┌──────────────────┐            ┌─────────────────┐          ┌───────────────┐
│ 首页选择业务       │            │                 │          │               │
│       ↓           │            │                 │          │               │
│ 刷身份证 / 就诊卡   │── 查询 ──→ │  适配器 + 熔断   │── 取数 ─→│  HIS / EMR    │
│ 或手输号码         │            │                 │          │  LIS / PACS   │
│       ↓           │            │                 │          │  票据平台      │
│ 单据列表           │            │                 │          │               │
│ （含已打印次数）    │            │                 │          │               │
│       ↓           │            │                 │          │               │
│ 版式预览           │── 打印 ──→ │  模板 + 排版     │── 回写 ─→│  打印次数      │
│       ↓           │            │  幂等 + 次数控制 │          │               │
│ 物理出纸 + 留痕     │            │                 │          │               │
└──────────────────┘            └─────────────────┘          └───────────────┘
```

**支持单据类型**：医疗门诊收费票据 · 门诊病历 · 检验报告单 · 检查报告 · 通用凭证

---

## ✨ 核心能力

### 患者侧

| 能力 | 说明 |
| :--- | :--- |
| 身份识别 | 身份证读取、就诊卡、二维码、纸质单据条码、手动输入（含含 X 的大数字键盘） |
| 会话管理 | 全局倒计时自动重置、10 秒提醒、打印中禁止强制退出（防票据滞留） |
| 单据查询 | 发票 / 门诊病历 / 检验报告 / 检查报告 / 通用单据，列表直显「已打印次数」 |
| 版式预览 | 所见即所得，渲染位图与实际打印共用同一套排版代码 |
| 隐私保护 | 身份证号、手机号、姓名按规则脱敏；界面遮蔽防旁观窥视 |
| 适老化 | 大字号、高对比、语音提示与延长会话时间 |

### 医生 / 收费侧

工作站端独立程序：按门诊号 + 就诊时间区间检索病历，支持多条件筛选、补打授权确认、批量打印。补打须填写原因并留痕。

### 打印引擎

- **模板与代码解耦** —— 版式以 JSON 描述页面元素，业务人员可自行调整，变更不触发发版
- **所见即所得** —— 预览与打印共用同一个排版器，不存在「预览好看、打印错位」
- **明细表跨页分页** —— 长处方自动续页并重复表头
- **流式签章区** —— 中文大写金额等超宽内容自动独占整行，不跨列重叠
- **可插拔后端** —— 内置 GDI+ 实现零依赖可编译；接入 FastReport 只需实现 `IReportBackend`

### 可靠性

| 机制 | 作用 |
| :--- | :--- |
| 幂等防重 | 同一单据在时间窗内只允许成功打印一次，连点被拦 |
| 次数控制 | 单据默认最多打印 2 份，超限提示到窗口办理 |
| 熔断降级 | 上游接口连续失败自动熔断，降级为演示数据而不是卡死患者 |
| 失败重试 | 仅失败任务可重试，最多 3 次，全程留痕 |
| 任务作废 | 作废不计入有效打印次数，但保留审计记录 |
| 单据锁定 | 打印全程加锁，杜绝并发导致重复出纸 |

### 可审计

登录、查询、打印提交、成功、失败、重试、作废、配置变更、设备告警 —— 每一个关键动作都落库。日志同时写文件，便于事后追溯与纠纷举证。

---

## 🛠 技术栈

| 层次 | 选型 | 说明 |
| :--- | :--- | :--- |
| 客户端 | C# 8.0 / .NET Framework 4.7.2 / WinForms | 保留 Windows 打印驱动与读卡器生态兼容性 |
| 报表 | FastReport .NET + 内置 GDI+ 后端 | 与院内既有系统一致；内置后端保证零依赖可编译 |
| 服务端 | ASP.NET Web API / Windows 服务宿主 | 同一解决方案内统一管理 |
| 数据库 | **SQLite（默认）/ SQL Server 2012+**，单参数切换 | 24 张核心表，两套建表脚本结构一致 |
| 缓存队列 | Redis / RabbitMQ（可选） | 单机版可省略 |
| 日志 | NLog | 分级落盘 |
| 依赖管理 | NuGet PackageReference | |
| 构建 | Visual Studio 2019 / MSBuild 16.x | |

---

## 🏗 系统架构

```
                    ┌─────────────────────────────┐
                    │      表现层（双端）           │
                    │  Client.Kiosk   Client.Station│
                    └──────────────┬──────────────┘
                                   │
                    ┌──────────────▼──────────────┐
                    │       编排层 HSP.Service     │
                    │  AppBootstrap（单一条装配路径）│
                    │  PrintJobService（打印编排）   │
                    └──────┬───────────────┬───────┘
                           │               │
          ┌────────────────▼──┐      ┌─────▼──────────────────┐
          │  能力层            │      │  数据集成层             │
          │  Print.FastReport  │      │  Adapter.HIS / EMR     │
          │  Print.Imaging     │      │  Adapter.LIS / PACS    │
          │  Device            │      │  Adapter.Invoice       │
          └────────────────────┘      │  Adapter.MedicalInsurance│
                                     │  Adapter.Auth          │
                                     └────────────┬───────────┘
                                                  │
                    ┌─────────────────────────────▼───────────┐
                    │           核心层 HSP.Core               │
                    │  Models / Common / DataAccess /         │
                    │  Interfaces / Logging / Session /       │
                    │  Configuration / Exceptions             │
                    └─────────────────────────────────────────┘

     服务端：Server.Api（REST）    服务宿主：Server.Host（Windows 服务）
```

**分层约束**（静态脚本 `csrefs.py` 校验，17 个工程引用闭包完整）：

- 核心层不依赖任何上层
- 打印层与适配器层并列，互不感知
- 客户端只依赖 `HSP.Service`，不直接依赖具体适配器

---

## 📁 工程结构

```
医院自助打印机程序/
├── README.md                          # 本文件
├── LICENSE                            # MIT
├── 医院自助打印机程序_功能计划进度文档.md
│
└── HospitalSelfService/
    ├── HospitalSelfService.sln        # VS2019 解决方案（17 个工程）
    ├── Directory.Build.props          # 统一编译属性
    ├── NuGet.config
    │
    ├── src/
    │   ├── HSP.Core/                  # 核心：模型 / 配置 / 日志 / 脱敏 / 会话 / 数据访问
    │   ├── HSP.Service/               # 编排：应用引导 + 打印任务控制
    │   ├── HSP.Adapter.HIS/           # HIS 适配器（Db / Rest / Demo 三模式）
    │   ├── HSP.Adapter.EMR/           # 电子病历适配器
    │   ├── HSP.Adapter.LIS/           # 检验系统适配器
    │   ├── HSP.Adapter.PACS/          # 影像系统适配器（DICOM 胶片参数）
    │   ├── HSP.Adapter.Invoice/       # 电子票据适配器（含扫码验真）
    │   ├── HSP.Adapter.MedicalInsurance/  # 医保身份核验与支付构成
    │   ├── HSP.Adapter.Auth/          # 统一认证适配器
    │   ├── HSP.Device/                # 设备层：自检 / 读卡器 / 出纸传感器
    │   ├── HSP.Print.FastReport/      # 报表引擎：模板 / 排版 / 渲染
    │   ├── HSP.Print.Imaging/         # 影像渲染（PDF / TIFF / JPG）
    │   ├── HSP.Client.Kiosk/          # 自助机端（WinForms）
    │   ├── HSP.Client.Station/        # 工作站端（WinForms）
    │   ├── HSP.Server.Api/            # Web API
    │   └── HSP.Server.Host/           # Windows 服务宿主
    │
    ├── test/
    │   └── HSP.SmokeTest/             # 端到端冒烟测试（98 项断言）
    │
    ├── templates/                     # 报表模板（JSON 版式定义）
    │   ├── Invoice/INV-OUTPATIENT.json       # 医疗门诊收费票据
    │   ├── MedicalRecord/REC-OUTPATIENT.json  # 门诊病历
    │   ├── Report/RPT-LAB.json               # 检验报告单
    │   └── Common/CMN-GENERAL.json           # 通用凭证
    │
    ├── database/
    │   ├── schema/                    # 建表脚本（24 张表 × 2 种数据库）
    │   ├── migration/                 # 迁移脚本
    │   └── seed/                      # 种子数据
    │
    ├── deploy/                        # 部署脚本（安装 / 自助机 / 升级）
    ├── web/                           # 管理后台前端
    ├── tools/                         # 辅助工具（TplGen / LogAnalyzer）
    └── docs/                          # 项目文档
```

---

## 🚀 快速开始

### 环境要求

- Windows 10 / 11（64 位）
- Visual Studio 2019，勾选「**.NET 桌面开发**」工作负载
- .NET Framework 4.7.2 开发包

### 构建

```powershell
cd HospitalSelfService

# 还原依赖
msbuild HospitalSelfService.sln /t:Restore

# 编译（Debug / Release 均为 0 错误 0 警告）
msbuild HospitalSelfService.sln /t:Rebuild /p:Configuration=Debug
msbuild HospitalSelfService.sln /t:Rebuild /p:Configuration=Release
```

推荐使用 VS2019 自带的 MSBuild 16.x：

```powershell
$msbuild = "C:\Program Files (x86)\Microsoft Visual Studio\2019\Enterprise\MSBuild\Current\Bin\MSBuild.exe"
cd HospitalSelfService
& $msbuild HospitalSelfService.sln /t:Rebuild /p:Configuration=Release
```

> 也可直接用 `dotnet build HospitalSelfService.sln`，或用 VS 打开 `.sln` 按 F5。

### 运行

```powershell
# 自助机端
.\src\HSP.Client.Kiosk\bin\Release\HSP.Client.Kiosk.exe

# 工作站端
.\src\HSP.Client.Station\bin\Release\HSP.Client.Station.exe

# 服务宿主（控制台模式便于调试）
.\src\HSP.Server.Host\bin\Release\HSP.Server.Host.exe --console
```

### 🎁 开箱即跑

默认使用 **SQLite**，首次运行自动建库建表（24 张表），**无需安装 SQL Server、无需手工执行任何 SQL**，数据文件落在程序目录 `data/hsp.db`。

数据库也不可用时自动降级为内存仓储 + 内置演示数据源（取自真实业务场景），「查询 → 预览 → 打印」全流程依然走得通。

---

## 🧪 运行测试

项目内置端到端冒烟测试，以真实业务链路验证而非仅验证编译：

```powershell
.\test\HSP.SmokeTest\bin\Release\HSP.SmokeTest.exe
```

```
============================================================
  医院自助打印机程序 — 端到端冒烟测试
============================================================
---- 1. 应用装配（AppBootstrap） -----------------------------------
  [PASS] AppBootstrap.Initialize 返回成功
  [PASS] 演示模式判定为假（SQLite 就绪）
  [PASS] 适配器注册数量 >= 5
---- 6. 打印编排（幂等 / 次数控制 / 状态） ------------------------
  [PASS] 首次打印任务完成  (Completed)
  [PASS] 捕获到实际出纸页  (1 页)
  [PASS] 重复打印被幂等拦截（M10-02）
  [PASS] 打印次数上限生效（M10-03）
...
---- 9. 数据库双库切换（38 项） -----------------------------------
  [PASS] SQL Server 脚本不做任何改写
  [PASS] 真实打印历史 SQL 翻译正确（LIMIT 在 ORDER BY 之后）
  [PASS] 自动建表执行成功；建表后 sys_user 存在（回归曾被漏表）
  [PASS] TOP (@top) 分页在 SQLite 下正确返回 3 棵
  [PASS] 唯一键重复占用成功拦截
  [PASS] 两份脚本表名集合一致
============================================================
  测试结果: 通过 98 项，失败 0 项
============================================================
```

| 配置 | 结果 | 退出码 |
| :--- | :--- | :--- |
| Debug | 通过 98 项，失败 0 项 | 0 |
| Release | 通过 98 项，失败 0 项 | 0 |

覆盖：应用装配、模板解析、列表查询、详情取数、预览渲染、打印编排、幂等防重、次数上限、任务作废、异常路径、脱敏校验、会话管理、日志落盘，以及 **38 项数据库双库切换验证**（方言翻译、连接串判别、提供程序工厂，以及在真实 SQLite 文件上完成建库建表 → 写入 → 查询 → 幂等防重 → 分页 → 事务 → 重复建表的完整链路）。

渲染产物输出为 PNG，便于人工核对版式。

### 无打印机环境下如何验证打印链路

测试通过 `ReportEngine.SetOutputAction(...)` 注入位图捕获器：反射取出 `PrintPage` 事件后备委托，按「份数 × 页数」逐页触发，传入离屏 `Graphics` —— 版式解析、数据绑定、分页、页脚绘制全部真实执行，仅不提交打印队列。物理出纸需在装有打印机驱动的机器上另行验证。

---

## ⚙️ 配置说明

配置位于各客户端的 `App.config`，全部配置项由 `ConfigProvider` 统一读取（**唯一配置入口**，代码中不散落 `ConfigurationManager.AppSettings[...]` 调用）：

### 常用配置

```xml
<appSettings>
  <!-- 服务端接口地址 -->
  <add key="Hsp.ApiBaseUrl" value="http://127.0.0.1:8080/api/" />
  <!-- 院区名称（显示在头栏） -->
  <add key="Hsp.HospitalName" value="XX市第一人民医院" />
  <!-- 会话超时（秒）；适老化模式建议 180 -->
  <add key="Hsp.SessionTimeoutSeconds" value="120" />
  <!-- 单据最大打印次数 -->
  <add key="Hsp.MaxPrintCount" value="2" />
  <!-- 幂等时间窗（秒），防连点 -->
  <add key="Hsp.IdempotentWindowSeconds" value="60" />
  <!-- 敏感信息脱敏开关 -->
  <add key="Hsp.MaskEnabled" value="true" />
</appSettings>
```

### 完整配置项

| 配置键 | 默认值 | 说明 |
| :--- | :--- | :--- |
| `Hsp.ApiBaseUrl` | `http://127.0.0.1:8080/api/` | 服务端接口地址 |
| `Hsp.ApiTimeoutSeconds` | `10` | 接口调用超时 |
| `Hsp.DefaultHospitalCode` | `H001` | 默认院区编码 |
| `Hsp.HospitalName` | — | 院区名称（头栏显示） |
| `Hsp.HospitalEnglishName` | — | 院区英文名称 |
| `Hsp.TerminalNo` | `KIOSK` | 终端编号，用于心跳与日志定位 |
| `Hsp.FooterNotice` | 咨询台提示语 | 页脚提示语 |
| `Hsp.SessionTimeoutSeconds` | `120` | 会话超时（秒） |
| `Hsp.AccessibleMode` | `false` | 适老化模式（大字号/高对比/延长会话） |
| `Hsp.MaxPrintCount` | `2` | 单据最大打印次数 |
| `Hsp.IdempotentWindowSeconds` | `60` | 幂等时间窗（秒） |
| `Hsp.MaskEnabled` | `true` | 敏感信息脱敏开关 |
| `Hsp.MaxReadRetry` | `3` | 读卡重试次数 |
| `Hsp.TakePaperTimeoutSeconds` | `30` | 取纸等待超时（秒） |
| `Hsp.DatabaseProvider` | `SQLite` | **`SQLite` / `SqlServer`，切换数据库的唯一开关** |
| `Hsp.ConnectionString` | `data\hsp.db` | 连接串（SQLite 为文件路径） |
| `Hsp.AutoInitSchema` | `true` | 启动时自动建表（幂等，可重复执行） |
| `Hsp.SchemaDirectory` | `database\schema` | 建表脚本目录 |
| `Hsp.TemplateRoot` | `templates` | 报表模板根目录 |

---

## 🧾 报表模板

模板为 JSON 描述的版式定义，与代码完全解耦 —— 业务人员可自行调整，不触发发版。

```jsonc
// templates/Invoice/INV-OUTPATIENT.json（节选，字段名与代码一一对应）
{
  "PageWidthMm": 210,
  "PageHeightMm": 140,
  "SubTitle": "XX医院",
  "Title": "医疗门诊收费票据（电子）",
  "QrCodeField": "QrCodeContent",      // 二维码内容取自数据的该字段
  "ShowSerialAtFooter": true,          // 页脚打印流水号
  "Fields": [                          // 字段区
    { "Label": "票据代码", "Path": "InvoiceCode" },
    { "Label": "开票日期", "Path": "IssueDate", "Format": "yyyy年MM月dd日" },
    { "Label": "姓名",     "Path": "PatientName" }
  ],
  "Table": {                           // 明细区（自动跨页并重复表头）
    "Title": "收费明细",
    "SourcePath": "Items",
    "Columns": [
      { "Header": "项目名称", "Path": "ItemName", "Width": 90 },
      { "Header": "金额（元）", "Path": "Amount", "Width": 70, "Align": "Right" }
    ]
  },
  "Signature": {                        // 签章区（超宽内容自动独占整行）
    "Fields": [ { "Label": "合计", "Path": "AmountText", "Bold": true } ],
    "Texts": [ "收款人", "复核人" ]
  }
}
```

版式可声明：页面尺寸（毫米）、标题区、字段区（标签 + 数据路径 + 格式 + 跨列数）、
明细表（列宽、对齐、跨页重复表头）、签章区、二维码占位、页脚流水号。

模板字段定义见 [`BuiltInReportBackend.cs`](HospitalSelfService/src/HSP.Print/HSP.Print.FastReport/BuiltInReportBackend.cs)。

**新增单据类型只需增加一份模板 JSON，无需改动任何代码。**

---

## 🗄 数据库

**SQL Server 与 SQLite 双库支持，切换只需改一个配置项。**

```xml
<!-- SQLite（默认）：单机版 / 开发测试，首次运行自动建表 -->
<add key="Hsp.DatabaseProvider" value="SQLite" />
<add key="Hsp.ConnectionString" value="data\hsp.db" />
<add key="Hsp.AutoInitSchema" value="true" />

<!-- SQL Server：多院区生产，改这三行即可 -->
<add key="Hsp.DatabaseProvider" value="SqlServer" />
<add key="Hsp.ConnectionString"
     value="Server=localhost;Database=HSP;Integrated Security=SSMS;" />
<add key="Hsp.AutoInitSchema" value="false" />
```

### 实现方式

`IDbProvider` 抽象 + 方言翻译，**业务 SQL 只写一份**：

| 差异点 | SQL Server | SQLite | 处理方式 |
| :--- | :--- | :--- | :--- |
| 结果集限行 | `SELECT TOP n` | `SELECT … LIMIT n` | 自动改写**关键字与位置**（LIMIT 须在 ORDER BY 之后） |
| 空值聚合 | `ISNULL(x, y)` | `IFNULL(x, y)` | 自动替换（列名含 `top` 等关键字时不做误替换） |
| 参数对象 | `SqlParameter` | `SQLiteParameter` | 统一为 `DbParam`，运行时由 Provider 落地 |
| 异常基类 | `SqlException` | `SQLiteException` | 统一按 `DbException` 捕获 |
| 唯一键冲突 | 受影响行数 0 | 抛约束异常 | `ExecuteNonQuery` 识别后返回 0（**幂等防重依赖此语义**） |
| 建表类型 | `NVARCHAR`/`DATETIME2`/`IDENTITY` | `TEXT`/`CURRENT_TIMESTAMP`/`AUTOINCREMENT` | 两份脚本，同构校验 |

> 业务代码里没有任何一处数据库专有类型引用，`SqlParameter` 只出现在 `DbProvider.cs` 内部。

### 24 张核心表

| 域 | 表 |
| :--- | :--- |
| 系统（8） | `sys_hospital` `sys_terminal` `sys_user` `sys_role` `sys_user_role` `sys_permission` `sys_dictionary` `sys_config` |
| 配置（2） | `cfg_business_entry` `cfg_interface` |
| 业务（8） | `biz_patient` `biz_visit` `biz_invoice` `biz_invoice_item` `biz_medical_record` `biz_prescription_item` `biz_report` `biz_report_item` |
| 打印（4） | `prt_template` `prt_job` `prt_job_log` `prt_idempotent` |
| 运维（2） | `audit_log` `dev_alarm` |

```powershell
# SQL Server：手工建库
sqlcmd -S localhost -d HSP -i database\schema\01_create_tables.sql

# SQLite：无需手工执行，程序首次运行自动建表
# 如需手工执行：
sqlite3 data\hsp.db < database\schema\01_create_tables_sqlite.sql
```

> 两份脚本由同一份源生成，`verify_schema.py` 校验表名与列结构完全一致（已有回归测试守护）。

关键设计：

- 身份证号加密存储，另设 `id_card_hash` 索引列支持按证件号等值检索
- `prt_idempotent` 存幂等键并带过期时间，兼作防重与清理依据
- `prt_job` + `prt_job_log` 完整留痕，支持事后追溯

---

## 🌐 Web API

| 方法 | 路由 | 说明 |
| :--- | :--- | :--- |
| GET | `api/document/list/{documentType}` | 单据列表查询 |
| GET | `api/document/detail/{documentType}/{documentId}` | 单据详情 |
| POST | `api/print/submit` | 提交打印任务 |
| GET | `api/print/job/{jobId}` | 任务状态实时回显 |
| POST | `api/print/retry/{jobId}` | 重试失败任务 |
| POST | `api/print/cancel/{jobId}` | 作废任务 |
| GET | `api/system/health` | 健康检查 |
| GET | `api/system/config` | 下发配置 |
| POST | `api/system/heartbeat` | 终端心跳 |

统一返回 `ApiResult`，全局异常过滤器把领域异常翻译为可读提示，**不向客户端泄漏堆栈**。

---

## 💡 设计取舍

几个值得说明的决策：

**打印层命名空间用 `HSP.Printing.*` 而非 `HSP.Print.*`**
—— 后者与 `System.Drawing` 存在前缀遮蔽，导致 `Font`、`PrintDocument` 等类型解析失败。改名彻底消除歧义。

**适配器注册表放在 `HSP.Core.Interfaces` 而非某个适配器工程**
—— 打印编排与客户端都要解析适配器。若注册表寄生在某个适配器内，每加一个适配器就要改动上游引用链，违反分层约束。

**提供内存仓储与演示数据源**
—— 保证院方接口未就绪时系统仍能完整跑通，而不是退化成「点了没反应」的假实现。演示模式下票据验真、医保核验明确标注「不构成真伪/参保结论」，避免误导患者。

**出纸动作抽象为可注入委托**
—— 便于接入静默打印、批次打印、PDF 直出等策略，也使无打印机环境下的渲染链路可被完整测试。

**演示数据严格遵守业务规则**
—— 演示用的身份证号通过 GB 11643 校验位验证。数据不合规会让演示自相矛盾（证件号被自己的校验判为非法）。

---

## 📊 当前进度

| 阶段 | 状态 |
| :--- | :--- |
| 需求调研与方案设计 | ✅ 完成 |
| 系统设计与工程搭建 | ✅ 完成 |
| 核心平台与打印引擎 | ✅ 完成 |
| 客户端界面（自助机 / 工作站） | ✅ 完成 |
| 数据源适配器（7 类） | ✅ 完成 |
| Web API 与服务宿主 | ✅ 完成 |
| 数据库双支持（SQLite / SQL Server） | ✅ 完成 |
| 编译验证（Debug / Release） | ✅ 0 错误 0 警告 |
| 运行验证（98 项端到端断言） | ✅ 全部通过 |
| 管理后台前端（`web/`） | 🟡 进行中 |
| 数据库生产联调 | ⬜ 待院方环境 |
| 真实接口对接 | ⬜ 待院方接口 |
| 物理出纸与硬件对接 | ⬜ 待现场环境 |

代码规模：**17 个工程 / 63 个 C# 源文件 / 26 个命名空间 / 约 1.56 万行**。

---

## 🗺 路线图

- [x] 参考系统业务闭环复现
- [x] 工程化重建（分层、异常、日志、审计）
- [x] 打印防重与次数控制
- [x] 隐私脱敏与适老化
- [x] 端到端测试宿主（98 项断言）
- [x] 数据库双支持（SQLite / SQL Server 单参数切换）
- [ ] 管理后台前端（`web/hsp-admin`）
- [ ] 接入 FastReport 正式后端（实现 `IReportBackend` 即可）
- [ ] 灰度升级与自动回滚
- [ ] 设备远程监控与告警推送
- [ ] 集成 CI（自动编译 + 测试 + 制品归档）

---

## 🤝 参与贡献

欢迎贡献代码、模板、文档与问题反馈。

1. **Fork** 本仓库并创建特性分支
2. 保持 `Directory.Build.props` 中的语言版本与编码约定
3. **提交前务必双配置编译并运行冒烟测试**：

```powershell
cd HospitalSelfService
msbuild HospitalSelfService.sln /t:Rebuild /p:Configuration=Debug
msbuild HospitalSelfService.sln /t:Rebuild /p:Configuration=Release
.\test\HSP.SmokeTest\bin\Release\HSP.SmokeTest.exe
```

4. 新增单据类型请一并补充模板 JSON 与冒烟测试断言
5. 提交信息遵循 [Conventional Commits](https://www.conventionalcommits.org/)
6. 保持分层依赖不被绕过 —— 引用链校验会确保跨工程引用闭包完整性

### 贡献新单据模板

在 `templates/` 下按单据类型建目录，放一份 JSON，并保证字段名与数据源 DTO 对得上：

```jsonc
// templates/YourType/YOU-XXX.json
{
  "PageWidthMm": 210,
  "PageHeightMm": 140,
  "Title": "你的单据标题",
  "SubTitle": "XX医院",
  "Fields":  [ { "Label": "姓名", "Path": "PatientName" } ],
  "Table":   { "SourcePath": "Items", "Columns": [
                 { "Header": "项目", "Path": "ItemName", "Width": 90 } ] },
  "Signature": { "Texts": [ "经办人" ] }
}
```

模板加载路径与默认目录在 `ConfigProvider` 中配置（`Hsp.TemplateRoot`）。

---

## ❓ 常见问题

<details>
<summary><b>编译报「找不到 MSBuild」怎么办？</b></summary>

用 VS2019 安装目录下的完整路径调用，或直接用 VS 打开 `.sln`。
本项目需要 .NET Framework 4.7.2 开发包，勾选「.NET 桌面开发」工作负载即可。
</details>

<details>
<summary><b>没有 SQL Server 能跑起来吗？</b></summary>

能。默认走 SQLite，首次运行自动建库建表，数据落在 `data/hsp.db`。
连 SQLite 都不可用时会降级为内存仓储 + 演示数据源，界面流程仍可完整走通。
</details>

<details>
<summary><b>没有打印机怎么验证打印？</b></summary>

运行冒烟测试即可。测试注入了离屏位图捕获器，版式解析、数据绑定、分页、
页脚绘制全部真实执行，只是不提交打印队列；渲染结果会输出为 PNG 可人工核对。
</details>

<details>
<summary><b>演示模式下的数据是真的吗？</b></summary>

演示数据取自真实业务场景，身份证号通过 GB 11643 校验位验证。
但票据验真与医保核验在演示模式下返回的是模拟结论，界面已明确标注
「不构成真伪/参保结论」，不可用于实际业务判断。
</details>

<details>
<summary><b>接入新的院内系统要改多少代码？</b></summary>

只需新增一个适配器实现 `IQueryAdapter` 接口并注册，业务流程无需改动。
现有 HIS / EMR / LIS / PACS / 票据 / 医保 / 认证共 7 类适配器均可作为模板。
</details>

---

## 📄 许可证

[MIT](LICENSE) © 2026 HospitalSelfService Contributors

第三方组件与 `参考/` 素材的版权归各自作者所有，详见 [LICENSE](LICENSE) 附表。
