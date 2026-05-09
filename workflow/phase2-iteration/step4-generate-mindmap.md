# 阶段二·第四步：生成思维导图

## 目标

将测试点转换为 Markmap 兼容的思维导图格式。

## Markmap 语法

Markmap 使用 Markdown 标题层级表示思维导图的层级结构：

```markdown
# 根节点（中心主题）
## 一级分支
### 二级分支
#### 三级分支
- 叶子节点（测试点）
- 叶子节点
```

## 处理流程

1. 以产品名+版本号为根节点
2. 功能测试、非功能测试和**合规测试**为一级分支
3. 模块名为二级分支
4. 功能名为三级分支
5. 测试点为叶子节点

## 输出格式

```markdown
# [产品名] v[版本号] 测试点

## 功能测试
### [模块A]
#### [功能A-1]
- AUTH-EQP-001：[P0] 验证有效邮箱格式被接受 [来源: FP-AUTH-001]
- AUTH-EQP-002：[P1] 验证无效邮箱格式被拒绝 [来源: FP-AUTH-001]
- AUTH-BVA-001：[P1] 验证密码长度边界值 [来源: FP-AUTH-002]
- AUTH-DCS-001：[P0] 验证登录组合条件 [来源: FP-AUTH-001]
- AUTH-STT-001：[P0] 验证连续5次失败锁定 [来源: FP-AUTH-003] [回归]
#### [功能A-2]
...
### [模块B]
...

## 非功能测试
### 性能测试
- SYS-PERF-001：[P0] 验证页面首屏加载不超过[阈值] [来源: FP-AUTH-001]
- ORDR-PERF-001：[P1] 验证[数量]用户并发操作时系统稳定 [来源: FP-ORDR-001]
### 安全测试
- SYS-SEC-001：[P0] 验证未登录访问被拒绝 [来源: FP-AUTH-001]
- AUTH-SEC-001：[P0] 验证水平越权被拦截 [来源: FP-AUTH-001]
- SRCH-SEC-001：[P1] 验证搜索框XSS攻击被转义 [来源: FP-SRCH-001]

## 合规测试
### [法规名称]
- AUTH-CMP-001：[P0] 测试点描述（引用法规） [来源: FP-AUTH-001]
- USR-CMP-001：[P1] 测试点描述（引用法规） [来源: FP-DATA-001]
```

## 输出保存

- 保存到 `my-mind-maps/<product-name>-v<version>-test-points.md`
- 同时将测试点完整清单保存到 `products/<product-name>/iterations/<version>/test-points.md`

## 渲染提示

Markmap 可通过以下方式渲染：
- CLI: `npx markmap <file.md>`
- VS Code 扩展: `markmap` 插件
- 在线: https://markmap.js.org/repl
