# 结构化文档格式

本文件定义功能现状文档的文件组织与字段格式。SKILL.md 描述四个转换工作流，本文件描述它们的产物格式。

## 目录结构

一个功能的所有结构化文档组成文档组：

```
docs/
├── project-manifest.yml              # 项目级清单（仅跨功能依赖时使用）
├── <feature>/                        # 功能目录
│   ├── manifest.yml                  # 功能级清单
│   ├── 1-template-download.md        # section 内容
│   └── 2-data-import.md
└── shared/                           # 跨功能共享 section
    └── name-to-code-conversion.md
```

`file` 字段指向承载该 section 内容的 `.md` 文件；没有 `file` 字段的子 section，内容写在最近的带 `file` 的父 section 的 `.md` 中。只写当前任务涉及的 section，不为占位创建空文件。

## 功能级 manifest

```yaml
title: "批量调拨"
type: plan                 # analysis | design | plan
created: 2026-11-24

sections:
  - id: "2"
    title: "数据导入"
    type: task
    status: done
    code_ref: "src/main/java/com/example/service/BatchImportService.java"
    file: "2-data-import.md"
    children:
      - id: "2.1"
        title: "importData"
        type: task
        status: done
        depends_on: ["2.2"]
        references: ["shared#name-to-code"]
        code_ref:
          file: "src/main/java/com/example/service/BatchImportService.java"
          method: "importData"
```

## 字段说明

| 字段 | 必填 | 说明 |
|---|---|---|
| id | 是 | 唯一标识，跨文档引用用 `文档id#section_id` |
| title | 是 | 简短标题 |
| type | 是 | overview / detail / requirement / design / task |
| status | 是 | pending / in_progress / done / blocked |
| code_ref | 否 | 对应的代码位置（文件或方法） |
| children | 否 | 子 section 列表，可任意深度嵌套 |
| file | 否 | 该 section 内容文件（相对于功能目录） |
| depends_on | 否 | 依赖的 section id，决定翻译顺序 |
| references | 否 | 引用的 section，翻译时需加载其内容 |

状态含义：`pending` 待翻译；`in_progress` 翻译中；`done` 校验通过；`blocked` 校验多轮未通过，需人工处理。

## code_ref 格式

| 格式 | 含义 |
|---|---|
| `"path/to/File.java"` | 整个文件对应这个 section |
| `{ file: "...", method: "xxx" }` | 文件中某个方法 |
| `{ file: "...", class: "Xxx", method: "xxx" }` | 文件中某个类的某个方法 |

## section 文件内容

每个 section 的 `.md` 用注释标记包裹，Markdown 标题层级表达嵌套：

```markdown
# 批量调拨 — 数据导入

<!-- ═══ SECTION: 2 | 数据导入 | status: done ═══ -->
<!-- DEPENDS_ON: none -->
<!-- REFERENCES: none -->

## 2 数据导入

用户上传 Excel 文件，系统解析并导入数据。

<!-- ═══ SECTION: 2.1 | importData | status: done ═══ -->
<!-- DEPENDS_ON: 2.2 -->

### 2.1 importData

解析 Excel 并逐行导入。

##### 实现细节
- [x] 使用 EasyExcel 读取文件，逐行解析
- [x] 空行过滤：跳过所有字段为空的行
- [x] 名称→编码转换失败时标记为校验错误，不中断导入
- [x] 所有行解析完成后批量插入数据库

##### 已知边界条件
- [x] 编码转换失败的行归入错误列表
- [x] 数据库插入部分失败时，已成功的行回滚

<!-- ═══ END: 2.1 ═══ -->

<!-- ═══ END: 2 ═══ -->
```

「实现细节」清单是翻译的核心：每条必须是具体、可验证的点，粒度到「AI 不知道就会出错」的层级。「已知边界条件」记录空值、并发、失败处理等场景。

## 关系与跨文档引用

| 关系 | 含义 | 翻译时的处理 |
|---|---|---|
| depends_on | 必须等对方完成 | 拓扑排序决定翻译顺序 |
| references | 引用对方内容 | 加载对方 section，逐条对照 |

跨文档引用格式为 `{文档id}#{section_id}`，如 `shared#name-to-code`。

## 项目级清单

存在跨功能依赖时创建 `project-manifest.yml`，记录功能列表与功能间的依赖、引用关系：

```yaml
title: "进销存系统"
features:
  - id: "batch-transfer"
    dir: "batch-transfer"
    title: "批量调拨"
    status: done

  - id: "batch-adjust"
    dir: "batch-adjust"
    title: "批量调整"
    status: pending
    depends_on: ["batch-transfer"]

cross_references:
  - from: "batch-adjust#1.1"
    to: "batch-transfer#2.2.1"
    type: references
    note: "批量调整复用批量调拨的导入校验逻辑"
```

功能间关系在功能级 manifest 的 section 上用 `depends_on` / `references` 表达；项目级清单只记录跨功能依赖与引用，不重复功能内容。

## 按需加载

section 较多时不必整篇读取，用 grep 定位后只读需要的块：

```bash
grep -rn "SECTION: 2.2.1" docs/batch-transfer/
```
