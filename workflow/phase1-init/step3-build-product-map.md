# 阶段一·第三步：构建产品功能地图

## 目标

将第二步提取的模块结构整理为完整的产品知识库，保存到 `products/` 目录。

## 处理流程

1. 整理产品概述
2. 组织模块结构（排列顺序按业务逻辑，非按文档章节）
3. 构建模块依赖关系图
4. 识别初始风险领域
5. 保存到产品目录

## 输出

创建以下文件：

### `products/<product-name>/product-overview.md`
使用 `templates/product-init-report.md` 模板，填写完整的产品信息。

### `products/<product-name>/modules/`
每个模块一个文件，按业务逻辑组织：
- `module-<name>.md`：模块功能点详情

### `products/<product-name>/modules-index.md`
模块索引文件，包含所有模块的概览和依赖关系。

## 完整性检查

核查清单：
- [ ] 所有模块都已提取
- [ ] 模块间依赖关系已识别
- [ ] 外部系统集成点已标注
- [ ] 初始风险领域已评估
- [ ] 产品概述完整
- [ ] 功能点无遗漏
