> 本文件对应 **Phase 2（迭代测试）Step 3+4 输出**。
> 展示一次完整迭代测试的输出格式和内容覆盖面。
> 包含：变更分析 → 影响范围 → 测试点（7种技术） → 探索式测试 → 思维导图 → 追溯矩阵 → 统计

---

# Mac CLI Toolkit v1.0 迭代测试计划

## 元数据

| 属性 | 内容 |
|------|------|
| 产品名称 | Mac CLI Toolkit |
| 迭代版本 | v1.0 |
| 需求文档 | Mac CLI Toolkit v1.0 需求规格说明书 |
| 生成日期 | 2026-05-12 |
| 基于产品 | example-mac-cli（v1.0 初始化基线） |

---

## 一、变更概览

| 变更类型 | 变更内容 | 涉及模块 | 功能点引用 |
|---------|---------|---------|-----------|
| 新增 | 批量重命名功能：batch-rename 命令，支持通配符模式匹配、dry-run预览、递归子目录 | FILE | FP-FILE-010（搜索）扩展 |
| 新增 | 进程保护模式：--protected 阻止对系统关键进程（PID<100）的 kill/renice 操作 | PROC | FP-PROC-006/007/008/014 安全增强 |
| 修改 | Ping超时策略：默认超时从无限等待改为10s，新增 --wait 自定义超时选项 | NET | FP-NET-001/002/003 行为变更 |
| 新增 | 系统信息导出功能：--export json/csv，支持硬件信息和系统日志的数据导出 | SYS | FP-SYS-001/002/003/007 扩展 |

---

## 二、影响范围分析

| 变更内容 | 直接模块 | 受影响模块 | 影响等级 | 回归策略 | 最低覆盖 |
|---------|---------|-----------|---------|---------|---------|
| 批量重命名 | FILE | — | 中 | 关键路径回归 | P0+P1 |
| 进程保护模式 | PROC | — | 高 | 全量回归 | P0+P1+P2 |
| Ping超时策略修改 | NET | — | 高 | 全量回归 | P0+P1+P2 |
| 系统信息导出 | SYS | — | 中 | 关键路径回归 | P0+P1 |

---

## 三、测试点清单

### 3.1 FILE 模块 — 批量重命名功能

##### 新增：batch-rename 基础功能

- FILE-EQP-001：[P0] 验证 batch-rename 使用 `*.txt` 通配符匹配当前目录下所有 txt 文件并重命名 [来源: FP-FILE-010]
- FILE-EQP-002：[P1] 验证 batch-rename 通配符无匹配文件时给出提示且不报错 [来源: FP-FILE-010]
- FILE-EQP-003：[P2] 验证 batch-rename 匹配到隐藏文件（`.` 开头）时默认排除、--all 可包含 [来源: FP-FILE-010]
- FILE-EQP-004：[P0] 验证 batch-rename 使用 `--pattern` 和 `--replace` 参数替换文件名中的指定字符串 [来源: FP-FILE-010]
- FILE-EQP-005：[P1] 验证 batch-rename 对包含空格的路径正确处理 [来源: FP-FILE-010]
- FILE-EQP-006：[P1] 验证 batch-rename 对包含特殊字符（`!@#$%^`）的文件名正确处理 [来源: FP-FILE-010]

##### 新增：batch-rename 边界与批量控制

- FILE-BVA-001：[P1] 验证 batch-rename 批量 0 个文件（空匹配）时静默退出返回码 0 [来源: FP-FILE-010]
- FILE-BVA-002：[P1] 验证 batch-rename 批量 1 个文件时可正常执行 [来源: FP-FILE-010]
- FILE-BVA-003：[P2] 验证 batch-rename 批量 100 个文件时全部成功 [来源: FP-FILE-010]
- FILE-BVA-004：[P2] 验证 batch-rename 批量 101 个文件时触发批量限制提示 [来源: FP-FILE-010]
- FILE-BVA-005：[P1] 验证 rename 结果文件名长度上限为 255 字符（254/255/256） [来源: FP-FILE-010]

##### 新增：batch-rename 选项组合

- FILE-DCS-001：[P1] 验证 `--dry-run` 模式显示预览但不实际执行重命名 [来源: FP-FILE-010]
- FILE-DCS-002：[P1] 验证 `--dry-run` + `--recursive` 组合预览递归目录的重命名结果 [来源: FP-FILE-010]
- FILE-DCS-003：[P2] 验证 `--overwrite`（覆盖同名文件）与 `--skip`（跳过同名文件）两种冲突策略的选择 [来源: FP-FILE-010]

##### 新增：batch-rename 安全

- FILE-SEC-001：[P0] 验证 batch-rename 拒绝包含路径遍历（`../../`）的替换模式 [来源: FP-FILE-010]
- FILE-SEC-002：[P1] 验证 batch-rename 对无写入权限目录中的文件不执行重命名并报错 [来源: FP-FILE-010]
- FILE-SEC-003：[P2] 验证 batch-rename 不允许重命名为已存在的系统文件（如 `/etc/passwd`） [来源: FP-FILE-010]

##### 新增：batch-rename 性能

- FILE-PERF-001：[P2] 验证 batch-rename 递归处理 10000 个文件时完成时间不超过 30s [来源: FP-FILE-010]
- FILE-PERF-002：[P2] 验证 batch-rename 深层嵌套目录（10层以上）可正确处理不超时 [来源: FP-FILE-010]

---

### 3.2 PROC 模块 — 进程保护模式

##### 新增：protected 模式基础行为

- PROC-EQP-001：[P0] 验证 `kill --protected` 拒绝终止 PID<100 的系统关键进程 [来源: FP-PROC-006]
- PROC-EQP-002：[P0] 验证 `kill --protected` 允许终止 PID≥100 的普通用户进程 [来源: FP-PROC-006]
- PROC-EQP-003：[P1] 验证 `renice --protected` 拒绝修改 PID<100 进程的优先级 [来源: FP-PROC-014]
- PROC-EQP-004：[P1] 验证 `kill -STOP --protected` 允许暂停普通进程但拒绝暂停系统进程 [来源: FP-PROC-008]

##### 新增：保护模式状态转换

- PROC-STT-001：[P0] 验证受保护进程处于"运行中"→ 发送 SIGTERM（--protected）→ 拒绝并提示"受保护进程" [来源: FP-PROC-006]
- PROC-STT-002：[P0] 验证非保护进程处于"运行中"→ 发送 SIGTERM（--protected）→ 正常终止 [来源: FP-PROC-006]
- PROC-STT-003：[P1] 验证受保护进程处于"已暂停"（SIGSTOP）→ 发送 SIGCONT（--protected）→ 允许继续（继续操作不视为危险） [来源: FP-PROC-009]
- PROC-STT-004：[P1] 验证用户进程 → 优先级调整为极高（nice -20）→ 权限不足被拒绝的状态转换 [来源: FP-PROC-013]

##### 新增：保护模式边界

- PROC-BVA-001：[P1] 验证 PID=99（保护范围内）的进程被拒绝操作 [来源: FP-PROC-006]
- PROC-BVA-002：[P1] 验证 PID=100（保护范围边界）的进程可被操作 [来源: FP-PROC-006]
- PROC-BVA-003：[P2] 验证 PID=101（保护范围外）的进程可被操作 [来源: FP-PROC-006]

##### 新增：进程管理安全

- PROC-SEC-001：[P0] 验证普通用户尝试通过指定 PID=1（launchd）并绕过 --protected 标志无法终止该进程 [来源: FP-PROC-006]
- PROC-SEC-002：[P1] 验证非 root 用户通过 sudo 方式运行工具时，protected 模式仍对系统进程生效 [来源: FP-PROC-007]
- PROC-SEC-003：[P1] 验证普通用户无法通过发送 SIGKILL 绕过 protected 限制 [来源: FP-PROC-007]

##### 新增：进程管理性能

- PROC-PERF-001：[P1] 验证 5000 个进程同时运行时的 ps 列表加载时间 <3s [来源: FP-PROC-001]
- PROC-PERF-002：[P2] 验证启用 --protected 时额外安全检查不增加命令执行时间（增加 ≤100ms） [来源: FP-PROC-006]

---

### 3.3 NET 模块 — Ping超时策略调整

##### 修改：默认超时行为变更

- NET-EQP-001：[P0] 验证 ping 未指定 --wait 时默认超时为 10s [来源: FP-NET-003]
- NET-EQP-002：[P1] 验证 ping 指定 --wait 5 时超时为 5s [来源: FP-NET-003]
- NET-EQP-003：[P1] 验证 ping 指定 --wait 0 时超时无限（与原行为一致） [来源: FP-NET-003]
- NET-EQP-004：[P1] 验证 ping --count 与 --wait 组合使用时两个参数互不影响 [来源: FP-NET-002/003]

##### 修改：超时边界值

- NET-BVA-001：[P1] 验证 ping 超时边界 —wait 1（最小值）时可达主机正常返回 [来源: FP-NET-003]
- NET-BVA-002：[P1] 验证 ping 超时边界 —wait 1（最小值）时不可达主机正确超时 [来源: FP-NET-003]
- NET-BVA-003：[P2] 验证 ping 超时边界 —wait 3600（最大值）时正常 [来源: FP-NET-003]
- NET-BVA-004：[P1] 验证 ping 参数 --wait -1（非法）被拒绝并提示有效范围 [来源: FP-NET-003]
- NET-BVA-005：[P2] 验证 ping 超时边界：目标响应时间 9.9s/10.0s/10.1s 时的超时判定正确性 [来源: FP-NET-003]

##### 新增：性能测试—网络响应

- NET-PERF-001：[P1] 验证 ping 对本地主机（127.0.0.1）的响应时间 <1ms [来源: FP-NET-001]
- NET-PERF-002：[P2] 验证 ping 丢包率统计在 0%/50%/100% 时显示正确 [来源: FP-NET-001]
- NET-PERF-003：[P2] 验证 dnsquery 的解析响应时间 <500ms（主流域名） [来源: FP-NET-009]

##### 新增：网络安全

- NET-SEC-001：[P0] 验证 ping 拒绝 ICMP 广播地址（如 255.255.255.255）以防止 Smurf 攻击 [来源: FP-NET-001]
- NET-SEC-002：[P1] 验证 portscan 对 localhost 以外的扫描记录操作日志 [来源: FP-NET-006]
- NET-SEC-003：[P1] 验证 portscan 在 1 分钟内对同一主机的扫描次数上限为 10 次（防滥用） [来源: FP-NET-008]
- NET-SEC-004：[P2] 验证 traceroute 对内网 IP 段（10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16）的追踪需用户确认 [来源: FP-NET-004]

---

### 3.4 SYS 模块 — 系统信息导出功能

##### 新增：export 基础功能

- SYS-EQP-001：[P0] 验证 `hardware-info --export json` 输出合法 JSON 格式 [来源: FP-SYS-004]
- SYS-EQP-002：[P1] 验证 `hardware-info --export csv` 输出合法 CSV 格式（标题行+数据行） [来源: FP-SYS-004]
- SYS-EQP-003：[P1] 验证 `logview --since "24h ago" --export json` 导出指定时间范围内的日志为 JSON [来源: FP-SYS-007]
- SYS-EQP-004：[P1] 验证 `disk-usage / --export csv --human-readable` 导出可读格式数据 [来源: FP-SYS-014]
- SYS-EQP-005：[P2] 验证 --export 不支持的格式（如 --export xml）时给出提示并列出支持格式 [来源: FP-SYS-004]

##### 新增：export 边界

- SYS-BVA-001：[P1] 验证 logview --export json 导出的 0 条日志时输出空数组 `[]` [来源: FP-SYS-007]
- SYS-BVA-002：[P2] 验证 logview --export json 导出 10000 条日志时文件大小不超过 50MB [来源: FP-SYS-007]
- SYS-BVA-003：[P2] 验证 hardware-info --export json 含特殊字符（如 CPU 名称中的 ©™）时 JSON 正确转义 [来源: FP-SYS-004]

##### 新增：系统信息安全

- SYS-SEC-001：[P0] 验证 hardware-info --export json 导出的数据**不包含**用户个人身份信息（如当前登录用户名、用户全名） [来源: FP-SYS-004]
- SYS-SEC-002：[P1] 验证 hardware-info --export json 导出的数据**不包含**设备序列号 [来源: FP-SYS-001]
- SYS-SEC-003：[P1] 验证 logview --export json 导出的日志中密码/密钥等敏感字段被自动脱敏 [来源: FP-SYS-007]
- SYS-SEC-004：[P2] 验证导出的 JSON/CSV 文件默认权限为 600（仅创建者可读写） [来源: FP-SYS-004]

##### 新增：合规测试

- SYS-CMP-001：[P0] 验证用户首次使用 export 功能时展示数据收集范围说明并请求确认 [来源: FP-SYS-004] [个人信息保护法]
- SYS-CMP-002：[P1] 验证 logview --since --export 导出的日志中包含的时间、用户信息遵循最小必要原则 [来源: FP-SYS-007] [个人信息保护法]
- SYS-CMP-003：[P1] 验证 hardware-info 输出的硬件序列号已做哈希脱敏处理 [来源: FP-SYS-001] [数据安全法]
- SYS-CMP-004：[P2] 验证用户可通过工具命令清除本地缓存的操作记录和导出文件 [来源: FP-SYS-004] [个人信息保护法]

---

## 四、安全测试（SEC）汇总

上文中已分散到各模块的 SEC 测试点：

| 测试点ID | 描述 | 风险等级 | 涉及模块 |
|---------|------|---------|---------|
| FILE-SEC-001 | [P0] 验证 batch-rename 拒绝路径遍历替换模式 | 高 | FILE |
| FILE-SEC-002 | [P1] 验证 batch-rename 无写入权限时报错 | 中 | FILE |
| FILE-SEC-003 | [P2] 验证 batch-rename 不允许重命名为系统文件 | 中 | FILE |
| PROC-SEC-001 | [P0] 验证普通用户无法绕过 --protected 终止 PID=1 | 高 | PROC |
| PROC-SEC-002 | [P1] 验证 sudo 下 protected 模式仍对系统进程生效 | 高 | PROC |
| PROC-SEC-003 | [P1] 验证 SIGKILL 无法绕过 protected 限制 | 高 | PROC |
| NET-SEC-001 | [P0] 验证 ping 拒绝 ICMP 广播地址防 Smurf 攻击 | 高 | NET |
| NET-SEC-002 | [P1] 验证 portscan 跨主机扫描记录操作日志 | 中 | NET |
| NET-SEC-003 | [P1] 验证 portscan 扫描频率限制 10次/分钟 | 中 | NET |
| NET-SEC-004 | [P2] 验证 traceroute 内网段追踪需确认 | 低 | NET |
| SYS-SEC-001 | [P0] 验证 export 不导出用户个人身份信息 | 高 | SYS |
| SYS-SEC-002 | [P1] 验证 export 不导出设备序列号 | 高 | SYS |
| SYS-SEC-003 | [P1] 验证 log export 密码/密钥脱敏 | 高 | SYS |
| SYS-SEC-004 | [P2] 验证导出文件权限 600 | 中 | SYS |

---

## 五、性能测试（PERF）汇总

| 测试点ID | 描述 | 预期阈值 | 场景等级 |
|---------|------|---------|---------|
| FILE-PERF-001 | [P2] 验证 batch-rename 递归处理 10000 文件 | <30s | P2 |
| FILE-PERF-002 | [P2] 验证 batch-rename 深层嵌套目录不超时 | 正常完成 | P2 |
| PROC-PERF-001 | [P1] 验证 5000 进程时 ps 加载 | <3s | P1 |
| PROC-PERF-002 | [P2] 验证 --protected 性能开销 | ≤100ms 额外 | P2 |
| NET-PERF-001 | [P1] 验证 ping 127.0.0.1 响应时间 | <1ms | P1 |
| NET-PERF-002 | [P2] 验证丢包率统计正确性 | 0%/50%/100% | P2 |
| NET-PERF-003 | [P2] 验证 dnsquery 解析时间 | <500ms | P2 |

---

## 六、合规测试（CMP）汇总

| 测试点ID | 描述 | 引用法规 | 涉及模块 |
|---------|------|---------|---------|
| SYS-CMP-001 | [P0] 首次 export 展示数据收集范围并请求确认 | 个人信息保护法 | SYS |
| SYS-CMP-002 | [P1] 日志导出遵循最小必要原则 | 个人信息保护法 | SYS |
| SYS-CMP-003 | [P1] 硬件序列号哈希脱敏 | 数据安全法 | SYS |
| SYS-CMP-004 | [P2] 用户可清除本地缓存和导出文件 | 个人信息保护法 | SYS |

---

## 七、探索式测试（ET）建议

| Session | Charter | 时间盒 | 关注点 |
|---------|---------|-------|--------|
| ET-001 | 探索 batch-rename 在复杂目录结构下的行为，使用含特殊字符、超长路径、隐藏文件的混合目录 | 60min | 文件名编码、路径深度、通配符与隐藏文件的交互 |
| ET-002 | 探索 protected 模式的边界情况，包括多进程竞争、fork后子进程的保护继承、进程名变更后的保护状态 | 60min | 进程保护边界、竞态条件、fork语义 |
| ET-003 | 探索网络模块在弱网/断网环境的统一错误处理，ping/traceroute/portscan/dnsquery 在各类异常网络下的表现一致性 | 60min | 错误信息一致性、超时行为统一性、退出码规范 |
| ET-004 | 探索 export 功能的异常场景：同时 --export 和 --watch 冲突参数、导出到不可写路径、导出超大日志文件时的中断恢复 | 45min | 参数冲突、权限异常、大文件中断恢复 |

---

## 八、思维导图（Markmap 格式）

```markdown
# Mac CLI Toolkit v1.0 测试点

## 功能测试
### FILE 模块
#### batch-rename 基础功能
- FILE-EQP-001：[P0] 验证 `*.txt` 通配符匹配正常
- FILE-EQP-002：[P1] 验证通配符无匹配时提示
- FILE-EQP-003：[P2] 验证隐藏文件默认排除
- FILE-EQP-004：[P0] 验证 pattern+replace 替换
- FILE-EQP-005：[P1] 验证含空格路径
- FILE-EQP-006：[P1] 验证特殊字符文件名
#### batch-rename 边界控制
- FILE-BVA-001：[P1] 验证批量 0 文件
- FILE-BVA-002：[P1] 验证批量 1 文件
- FILE-BVA-003：[P2] 验证批量 100 文件
- FILE-BVA-004：[P2] 验证批量超限 101 文件
- FILE-BVA-005：[P1] 验证文件名上限 255 字符
#### batch-rename 选项组合
- FILE-DCS-001：[P1] dry-run 预览
- FILE-DCS-002：[P1] dry-run + recursive
- FILE-DCS-003：[P2] overwrite vs skip 策略
### PROC 模块
#### protected 模式基础
- PROC-EQP-001：[P0] 拒绝终止 PID<100
- PROC-EQP-002：[P0] 允许终止 PID≥100
- PROC-EQP-003：[P1] 拒绝修改受保护进程优先级
- PROC-EQP-004：[P1] 允许暂停普通进程
#### protected 状态转换
- PROC-STT-001：[P0] 受保护进程 TERM 被拒
- PROC-STT-002：[P0] 非保护进程 TERM 正常
- PROC-STT-003：[P1] 受保护进程 CONT 允许
- PROC-STT-004：[P1] 优先级调整权限拒绝
#### protected 边界
- PROC-BVA-001：[P1] PID=99 被拒绝
- PROC-BVA-002：[P1] PID=100 可操作
- PROC-BVA-003：[P2] PID=101 可操作
### NET 模块
#### 超时策略变更
- NET-EQP-001：[P0] 默认超时 10s
- NET-EQP-002：[P1] --wait 5 超时 5s
- NET-EQP-003：[P1] --wait 0 无限等待
- NET-EQP-004：[P1] --count + --wait 组合
#### 超时边界
- NET-BVA-001：[P1] --wait 1 可达主机
- NET-BVA-002：[P1] --wait 1 不可达主机
- NET-BVA-003：[P2] --wait 3600 最大值
- NET-BVA-004：[P1] --wait -1 非法
- NET-BVA-005：[P2] 9.9s/10.0s/10.1s 边界
### SYS 模块
#### export 基础功能
- SYS-EQP-001：[P0] 导出 JSON 合法
- SYS-EQP-002：[P1] 导出 CSV 合法
- SYS-EQP-003：[P1] 日志导出 JSON
- SYS-EQP-004：[P1] disk-usage 导出 CSV
- SYS-EQP-005：[P2] 不支持格式提示
#### export 边界
- SYS-BVA-001：[P1] 0 条日志导出空数组
- SYS-BVA-002：[P2] 10000 条日志文件大小
- SYS-BVA-003：[P2] 特殊字符 JSON 转义

## 非功能测试
### 安全测试
#### FILE 安全
- FILE-SEC-001：[P0] 拒绝路径遍历模式
- FILE-SEC-002：[P1] 无权限报错
- FILE-SEC-003：[P2] 拒绝重命名为系统文件
#### PROC 安全
- PROC-SEC-001：[P0] 无法绕过 protected
- PROC-SEC-002：[P1] sudo 下仍生效
- PROC-SEC-003：[P1] SIGKILL 无法绕过
#### NET 安全
- NET-SEC-001：[P0] 拒绝广播地址
- NET-SEC-002：[P1] 跨主机扫描记录日志
- NET-SEC-003：[P1] 扫描频率限制
- NET-SEC-004：[P2] 内网追踪需确认
#### SYS 安全
- SYS-SEC-001：[P0] 不导出用户身份信息
- SYS-SEC-002：[P1] 不导出设备序列号
- SYS-SEC-003：[P1] 日志密钥脱敏
- SYS-SEC-004：[P2] 导出文件权限 600
### 性能测试
#### FILE 性能
- FILE-PERF-001：[P2] 10000 文件 <30s
- FILE-PERF-002：[P2] 深层目录不超时
#### PROC 性能
- PROC-PERF-001：[P1] 5000 进程 <3s
- PROC-PERF-002：[P2] protected 开销 ≤100ms
#### NET 性能
- NET-PERF-001：[P1] ping 本地 <1ms
- NET-PERF-002：[P2] 丢包率统计正确
- NET-PERF-003：[P2] dnsquery <500ms
### 合规测试
#### 个人信息保护法
- SYS-CMP-001：[P0] 首次 export 数据范围告知
- SYS-CMP-002：[P1] 日志导出最小必要
- SYS-CMP-004：[P2] 用户清除缓存权
#### 数据安全法
- SYS-CMP-003：[P1] 序列号脱敏

## 探索式测试
### ET-001：复杂目录 batch-rename
### ET-002：protected 边界探索
### ET-003：弱网环境统一错误处理
### ET-004：export 异常场景恢复
```

---

## 九、需求追溯矩阵

| 功能点 | 测试点覆盖 | 覆盖率 |
|-------|-----------|--------|
| FP-FILE-010 | FILE-EQP-001~006, FILE-BVA-001~005, FILE-DCS-001~003, FILE-SEC-001~003, FILE-PERF-001~002 | 100% |
| FP-PROC-006 | PROC-EQP-001~002, PROC-STT-001~002, PROC-BVA-001~003, PROC-SEC-001~003, PROC-PERF-002 | 100% |
| FP-PROC-007 | PROC-SEC-003 | 100% |
| FP-PROC-008 | PROC-EQP-004 | 100% |
| FP-PROC-009 | PROC-STT-003 | 100% |
| FP-PROC-013 | PROC-STT-004 | 100% |
| FP-PROC-014 | PROC-EQP-003 | 100% |
| FP-PROC-001 | PROC-PERF-001 | 100% |
| FP-NET-001 | NET-PERF-001~002 | 100% |
| FP-NET-002 | NET-EQP-004 | 100% |
| FP-NET-003 | NET-EQP-001~004, NET-BVA-001~005 | 100% |
| FP-NET-004 | NET-SEC-004 | 100% |
| FP-NET-006 | NET-SEC-002 | 100% |
| FP-NET-008 | NET-SEC-003 | 100% |
| FP-NET-009 | NET-PERF-003 | 100% |
| FP-SYS-004 | SYS-EQP-001~005, SYS-BVA-003, SYS-SEC-001~002, SYS-CMP-001/004 | 100% |
| FP-SYS-007 | SYS-EQP-003, SYS-BVA-001~002, SYS-SEC-003, SYS-CMP-002 | 100% |
| FP-SYS-014 | SYS-EQP-004 | 100% |
| FP-SYS-001 | SYS-SEC-002, SYS-CMP-003 | 100% |

**未覆盖功能点**：0（全功能点覆盖率 ≥ 1 个测试点）

---

## 十、测试点统计

### 按测试技术分布

| 技术 | 数量 | 占比 |
|------|------|------|
| EQP（等价类划分） | 18 | 32.1% |
| BVA（边界值分析） | 13 | 23.2% |
| DCS（决策表） | 3 | 5.4% |
| STT（状态转换） | 4 | 7.1% |
| SEC（安全测试） | 14 | 25.0% |
| PERF（性能测试） | 7 | 12.5% |
| CMP（合规测试） | 4 | 7.1% |
| **合计** | **56** | **100%** |

### 按优先级分布

| 优先级 | 数量 | 占比 |
|--------|------|------|
| P0 | 12 | 21.4% |
| P1 | 27 | 48.2% |
| P2 | 17 | 30.4% |
| **合计** | **56** | **100%** |

### 按模块分布

| 模块 | 数量 | 占比 |
|------|------|------|
| FILE | 16 | 28.6% |
| PROC | 13 | 23.2% |
| NET | 13 | 23.2% |
| SYS | 14 | 25.0% |
| **合计** | **56** | **100%** |

### 正反向比率

| 类型 | 数量 | 占比 |
|------|------|------|
| 正向测试 | 22 | 39.3% |
| 反向测试 | 34 | 60.7% |
| **合计** | **56** | **100%** |

---

## 十一、完成标准检查

| 标准 | 检查结果 |
|------|---------|
| P0 测试点覆盖率 100%（12/12） | ✅ |
| P1 测试点通过率 ≥95% | 待执行 |
| P2 测试点通过率 ≥90% | 待执行 |
| 无未关闭的 P0 缺陷 | 待执行 |
| ET Session 完成率 ≥80%（4次中完成3次以上） | 待执行 |
