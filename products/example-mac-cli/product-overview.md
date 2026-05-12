> 本文件对应 **Phase 1（产品初始化）Step 3 输出**。
> 上半部分是"输入需求文档（简版）"，下半部分是"输出初始化报告"。
> 通过这种对比，你可以看到框架如何将原始需求转化为结构化测试资产。

---

# 第一部分：输入需求文档（简版）

```
# Mac CLI Toolkit v1.0

## 概述
Mac CLI Toolkit 是 macOS 平台上的命令行系统管理工具集，提供常用系统管理命令的统一入口。

## 功能模块

### 1. 文件管理（file）
- 复制文件：cp <src> <dst> [--recursive] [--force]
- 移动文件：mv <src> <dst> [--force]
- 删除文件：rm <path> [--recursive] [--force]
- 搜索文件：find <path> --name <pattern> [--type f/d] [--size +-N] [--mtime +-N]
- 修改权限：chmod <mode> <path>
- 创建链接：ln <target> <link> [--symbolic]

### 2. 进程管理（process）
- 进程列表：ps [--user <name>] [--sort cpu/mem]
- 进程树：pstree [--pid <pid>]
- 发送信号：kill <signal> <pid>
- 资源监控：top [--interval <sec>] [--count <n>]
- 优先级管理：nice/renice <priority> <command/pid>

### 3. 网络诊断（network）
- Ping：ping <host> [--count <n>] [--timeout <sec>]
- Traceroute：traceroute <host> [--max-hops <n>]
- 端口扫描：portscan <host> [--port <range>]
- DNS查询：dnsquery <domain> [--type A/AAAA/MX]
- 接口信息：ifinfo [--interface <name>]

### 4. 系统信息（system）
- 硬件信息：hardware-info [--category cpu/memory/disk]
- 版本信息：os-version [--detail]
- 日志查看：logview [--since <time>] [--level error/warn/info]
- 用户会话：user-sessions [--active]
- 磁盘使用：disk-usage <path> [--human-readable]
```

---

# 第二部分：产品初始化报告

## 产品概述

| 属性 | 内容 |
|------|------|
| 产品名称 | Mac CLI Toolkit |
| 产品类型 | 命令行工具（CLI） |
| 目标用户 | macOS 系统管理员、开发者、高级用户 |
| 核心价值 | 提供统一的命令行入口，简化 macOS 系统管理操作 |
| 技术栈 | Bash/Zsh, Swift, Foundation, Network.framework, IOKit |

## 行业分类与合规信息

| 维度 | 分类 |
|------|------|
| 所属行业 | 企业服务/SaaS（工具类软件） |
| 业务区域 | 全球（macOS 全球发行） |
| 用户类型 | B 端为主（系统管理员、开发者），兼有 C 端高级用户 |

### 适用法规清单

| 法规名称 | 适用理由 | 关键要求 | 涉及模块 |
|---------|---------|---------|---------|
| 《网络安全法》 | 网络诊断模块涉及网络探测和数据传输 | 网络活动日志记录、用户授权、数据加密传输 | NET |
| 《数据安全法》 | 系统信息模块收集硬件/软件配置数据 | 数据分类分级、用户数据处理授权、数据出境合规 | SYS |
| 《个人信息保护法》 | 系统日志和用户会话可能包含个人信息 | 个人信息收集告知、最小必要原则、删除权 | SYS, PROC |
| ISO 27001 | 企业级工具的信息安全管理要求 | 访问控制、审计日志、安全开发流程 | 全模块 |

### 合规测试重点领域

- **数据收集透明性**：系统信息模块收集的数据范围需明确告知用户
- **日志安全**：操作日志和系统日志不可包含明文敏感信息
- **网络安全边界**：网络诊断功能不可被滥用为攻击工具（防止作为跳板）
- **权限最小化**：工具运行所需权限应符合最小必要原则
- **数据删除机制**：用户应能清除工具生成的缓存和日志

## 模块结构

| 模块ID | 模块名称 | 描述 | 依赖模块 | 功能点数量 |
|--------|---------|------|---------|-----------|
| FILE | 文件管理 | 文件复制/移动/删除/搜索/权限/链接管理 | 无 | 18 |
| PROC | 进程管理 | 进程列表/树状显示/信号发送/资源监控/优先级 | 无 | 15 |
| NET | 网络诊断 | Ping/路由追踪/端口扫描/DNS查询/接口信息 | 无 | 15 |
| SYS | 系统信息 | 硬件信息/OS版本/日志查看/用户会话/磁盘使用 | 无 | 15 |

## 模块间依赖关系

```
FILE ─── 独立模块，无外部依赖
PROC ─── 独立模块，依赖系统进程 API
NET ──── 独立模块，依赖 Network.framework
SYS ──── 独立模块，依赖 IOKit 和系统框架
```

> 四个模块彼此独立，无数据流依赖。每个模块通过调用 macOS 系统 API 实现功能。

## 功能点清单

### FILE 模块：文件管理

| FP ID | 描述 | 输入条件 | 预期行为 | 异常处理 |
|-------|------|---------|---------|---------|
| FP-FILE-001 | 单文件复制 | 源文件存在、目标路径合法 | 文件完整复制到目标路径 | 源文件不存在→报错；目标路径无写入权限→报错 |
| FP-FILE-002 | 多文件复制 | 多个源文件+目标目录 | 所有文件复制到目录 | 任一文件失败→继续复制其余并汇总错误 |
| FP-FILE-003 | 递归目录复制 | 源目录+--recursive标志 | 目录及其所有内容递归复制 | 目录不存在→报错；深层文件无权限→跳过并告警 |
| FP-FILE-004 | 强制覆盖复制 | 目标文件已存在+--force标志 | 直接覆盖目标文件 | 无特殊异常 |
| FP-FILE-005 | 非强制复制遇已存在 | 目标文件已存在、无--force | 提示确认是否覆盖 | 用户取消→跳过 |
| FP-FILE-006 | 单文件移动 | 源文件存在、目标路径合法 | 文件移动到目标路径 | 跨设备移动→先复制后删除 |
| FP-FILE-007 | 文件删除 | 文件存在、有写入权限 | 文件被删除移到废纸篓 | 文件不存在→报错 |
| FP-FILE-008 | 递归删除目录 | 目录+--recursive标志 | 目录及所有内容删除 | 目录非空且无--recursive→拒绝 |
| FP-FILE-009 | 强制删除 | --force标志 | 跳过确认直接删除 | 无确认提示 |
| FP-FILE-010 | 按名称搜索文件 | --name pattern | 返回匹配路径列表 | 无匹配→空结果 |
| FP-FILE-011 | 按类型过滤搜索 | --type f/d | 仅返回文件或目录 | 非法类型参数→报错 |
| FP-FILE-012 | 按大小过滤搜索 | --size +10M | 返回大于指定大小的文件 | 大小格式非法→报错 |
| FP-FILE-013 | 按修改时间过滤 | --mtime -7 | 返回7天内修改的文件 | 时间格式非法→报错 |
| FP-FILE-014 | 八进制权限修改 | chmod 755 <file> | 权限修改为rwxr-xr-x | 文件不存在→报错 |
| FP-FILE-015 | 符号权限修改 | chmod u+x <file> | 为用户添加执行权限 | 符号模式语法错误→报错 |
| FP-FILE-016 | 创建硬链接 | ln target link | 创建硬链接指向target | target不存在→报错；跨设备→报错 |
| FP-FILE-017 | 创建符号链接 | ln -s target link | 创建符号链接 | target不存在仍可创建（悬空链接） |
| FP-FILE-018 | 文件信息查看 | 文件路径 | 显示类型/大小/权限/修改时间 | 文件不存在→报错 |

### PROC 模块：进程管理

| FP ID | 描述 | 输入条件 | 预期行为 | 异常处理 |
|-------|------|---------|---------|---------|
| FP-PROC-001 | 显示所有进程 | 无参数 | 列出系统所有进程 | 无 |
| FP-PROC-002 | 按用户筛选进程 | --user <name> | 仅显示该用户进程 | 用户不存在→空列表 |
| FP-PROC-003 | 按CPU排序 | --sort cpu | 按CPU使用率降序排列 | 无 |
| FP-PROC-004 | 按内存排序 | --sort mem | 按内存使用率降序排列 | 无 |
| FP-PROC-005 | 进程树显示 | pstree | 树状显示进程父子关系 | --pid指定pid不存在→报错 |
| FP-PROC-006 | 终止进程 | kill -TERM <pid> | 发送SIGTERM终止进程 | pid不存在→报错；无权限→报错 |
| FP-PROC-007 | 强制终止进程 | kill -KILL <pid> | 发送SIGKILL强制终止 | 同上 |
| FP-PROC-008 | 暂停进程 | kill -STOP <pid> | 发送SIGSTOP暂停进程 | 同上 |
| FP-PROC-009 | 继续进程 | kill -CONT <pid> | 发送SIGCONT继续进程 | 同上 |
| FP-PROC-010 | 实时资源监控 | top | 每1秒刷新显示资源占用TOP进程 | 无 |
| FP-PROC-011 | 自定义刷新间隔 | top --interval 5 | 每5秒刷新 | interval≤0→使用默认值 |
| FP-PROC-012 | 指定刷新次数 | top --count 10 | 刷新10次后自动退出 | count≤0→持续刷新 |
| FP-PROC-013 | 低优先级启动 | nice -n 10 <command> | 以较低优先级运行命令 | 权限不足→使用默认优先级 |
| FP-PROC-014 | 修改运行中进程优先级 | renice -n 10 <pid> | 修改指定进程优先级 | pid不存在→报错；权限不足→报错 |
| FP-PROC-015 | 查看进程详细信息 | ps --pid <pid> | 显示指定进程的完整信息 | pid不存在→报错 |

### NET 模块：网络诊断

| FP ID | 描述 | 输入条件 | 预期行为 | 异常处理 |
|-------|------|---------|---------|---------|
| FP-NET-001 | 基本Ping测试 | ping <host> | 持续发送ICMP探测包 | 主机不可达→超时报错 |
| FP-NET-002 | 指定Ping次数 | ping --count 4 | 发送4个包后停止 | count≤0→使用默认值 |
| FP-NET-003 | 指定Ping超时 | ping --timeout 5 | 每5秒无响应判定超时 | timeout≤0→使用默认值 |
| FP-NET-004 | 基本路由追踪 | traceroute <host> | 显示到目标的每一跳 | 目标不可达→显示最后一跳 |
| FP-NET-005 | 指定最大跳数 | traceroute --max-hops 30 | 最多30跳后停止 | hops≤0或>255→使用默认值 |
| FP-NET-006 | 端口扫描（单端口） | portscan <host> --port 80 | 检查指定端口是否开放 | 主机不可达→报错 |
| FP-NET-007 | 端口范围扫描 | portscan <host> --port 1-1024 | 扫描范围内的所有端口 | 范围格式错误→报错 |
| FP-NET-008 | 多端口扫描 | portscan <host> --port 80,443,8080 | 扫描指定列表端口 | 同上 |
| FP-NET-009 | A记录DNS查询 | dnsquery <domain> --type A | 返回IPv4地址 | 域名不存在→NXDOMAIN |
| FP-NET-010 | AAAA记录DNS查询 | dnsquery <domain> --type AAAA | 返回IPv6地址 | 无AAAA记录→空结果 |
| FP-NET-011 | MX记录DNS查询 | dnsquery <domain> --type MX | 返回邮件交换记录 | 无MX记录→空结果 |
| FP-NET-012 | 显示所有网络接口 | ifinfo | 列出所有接口基本信息 | 无 |
| FP-NET-013 | 指定接口详情 | ifinfo --interface en0 | 显示指定接口详细信息 | 接口不存在→报错 |
| FP-NET-014 | Ping不可达主机 | ping 10.255.255.1 | 超时报错并显示丢包率 | 目标不可达 |
| FP-NET-015 | 扫描保留端口 | portscan <host> --port 0 | 端口0非法→报错 | 端口范围1-65535 |

### SYS 模块：系统信息

| FP ID | 描述 | 输入条件 | 预期行为 | 异常处理 |
|-------|------|---------|---------|---------|
| FP-SYS-001 | 显示CPU信息 | hardware-info --category cpu | 显示型号/核心数/频率 | 无 |
| FP-SYS-002 | 显示内存信息 | hardware-info --category memory | 显示总量/使用量/类型 | 无 |
| FP-SYS-003 | 显示磁盘信息 | hardware-info --category disk | 显示磁盘型号/容量 | 无 |
| FP-SYS-004 | 默认显示全部硬件 | hardware-info | 显示CPU+内存+磁盘摘要 | 无 |
| FP-SYS-005 | 显示OS版本 | os-version | 显示版本号+构建号 | 无 |
| FP-SYS-006 | 显示详细OS信息 | os-version --detail | 显示版本+构建+内核+启动时间 | 无 |
| FP-SYS-007 | 查看最近系统日志 | logview | 显示最近N条日志 | 无 |
| FP-SYS-008 | 按时间过滤日志 | logview --since "2024-01-01" | 显示指定时间后的日志 | 时间格式错误→报错 |
| FP-SYS-009 | 按级别过滤日志 | logview --level error | 仅显示错误级别日志 | 级别参数非法→报错 |
| FP-SYS-010 | 显示活跃用户会话 | user-sessions --active | 显示当前登录的用户 | 无 |
| FP-SYS-011 | 显示所有用户会话 | user-sessions | 显示所有登录历史 | 无 |
| FP-SYS-012 | 查看指定目录磁盘使用 | disk-usage <path> | 显示目录总大小 | 路径不存在→报错 |
| FP-SYS-013 | 可读格式显示 | disk-usage <path> --human-readable | 以K/M/G单位显示 | 同上 |
| FP-SYS-014 | 查看根目录磁盘使用 | disk-usage / | 显示根目录总大小 | 无 |
| FP-SYS-015 | 查看网络状态信息 | ifinfo --statistics | 显示收发数据包统计 | 无 |

## 初始风险评估

| 风险项 | 风险等级 | 描述 |
|--------|---------|------|
| 命令注入 | **高** | CLI工具直接接受用户输入参数，未做充分转义可能导致命令注入 |
| 权限滥用 | **高** | 文件管理、进程管理模块涉及系统级操作，权限控制不当可导致越权 |
| 网络探测被滥用 | **中** | 端口扫描、Ping等功能可能被用于网络攻击探测 |
| 敏感信息泄露 | **中** | 系统信息模块可能暴露硬件序列号、用户账户名等敏感信息 |
| 日志数据隐私 | **中** | 操作日志可能记录用户路径、文件名等个人信息 |

## 初始化元数据

| 属性 | 内容 |
|------|------|
| 初始化日期 | 2026-05-12 |
| 需求文档版本 | v1.0 |
| 模块总数 | 4 |
| 功能点总数 | 63 |
| 适用法规数 | 4 |
