---
name: sunp-code-style
description: >
  审查代码是否符合风格规范，按项目与文件自动识别语言触发对应审查规则，输出分级问题清单。
  覆盖 Java、TypeScript、JavaScript、Python 与语言无关通用项；只审查、不修改代码。
  Use when 需要检查代码风格、命名、注释、结构与一致性，或做提交前自查与代码评审。
  Triggers: '代码风格审查', '代码风格', '代码规范', '风格检查', '代码注释审查', '命名规范', '代码自查'.
license: MIT
metadata:
  language: zh-CN
  audience: all-developers
---

# 代码风格审查

## 用途与边界

审查代码是否符合风格规范，输出可定位、可执行的问题清单；只读，不改代码、不实施重构。以文件扩展名识别语言并逐文件套用规则，项目配置仅作辅助判定。

只处理风格层：格式、命名、注释、结构、一致性。逻辑正确性、功能缺陷、安全与性能问题归 `sunp-code-logic`，本 skill 点到即止、不展开。

## 与相邻 skill 的边界

- `sunp-doc-check` 审查文档表达层；本 skill 审查代码风格层
- 代码逻辑审查（场景 5）另立 skill，本 skill 不替代

## 输入解析

调用时若随 skill 附带了文字（如 `/sunp-code-style src/OrderService.java`），视为待审查的文件或目录路径；路径明确时直接读取，含糊时按流程先问清楚。没有附带文字时同样先问。

## 流程

复制下列清单逐项推进：

```
- [ ] 确认审查对象：文件或目录路径；没有先问
- [ ] 识别语言：以文件扩展名逐文件判定，必要时用项目配置辅助，加载对应规则
- [ ] 逐项审查，记录位置、表现、依据、改法与级别
- [ ] 按级别排序问题清单输出，只报确实存在的问题
- [ ] 事实或逻辑疑点单列，不计入风格问题
```

## 审查依据

通用规则见 [`references/general.md`](references/general.md)。语言规则：

- [`references/java.md`](references/java.md)
- [`references/typescript.md`](references/typescript.md)
- [`references/javascript.md`](references/javascript.md)
- [`references/python.md`](references/python.md)

语言判定：以文件扩展名为第一依据，逐文件判定并套用对应规则：

| 扩展名 | 语言 |
|---|---|
| `.java` | Java |
| `.ts`、`.tsx` | TypeScript |
| `.js`、`.jsx` | JavaScript |
| `.py` | Python |

项目配置（`pom.xml`、`build.gradle*`、`tsconfig.json`、`package.json`、`pyproject.toml`、`setup.py`、`requirements.txt`）只作辅助：文件无扩展名或需盘点项目语言构成时用来推断，不覆盖扩展名的判定结果。

边界处理：

- 审查对象为单文件时，直接按扩展名判定
- 配置文件本身不套语言规则，只套通用规则
- 无法识别的文件只套通用规则，并在交付说明中列出未识别文件

## 输出

```
👉 结论
一句话给出总体判断：风格是否达标、主要问题是什么。

👉 问题清单（按级别排序）
1. [必须改] <位置>：<问题> → 依据：<规范或约定>；建议：<改法>
2. [建议改] <位置>：<问题> → 依据：<规范或约定>；建议：<改法>
3. [Nit] <位置>：<问题> → 建议：<改法>

👉 交付说明
识别到的语言、审查覆盖范围、未识别或未审查的文件及原因、事实或逻辑疑点。
```

级别定义：必须改（违反硬性规范或项目约定）、建议改（影响可读性或一致性但可接受例外）、Nit（偏好性优化，可不改）。无问题且无疑点时写「未发现风格问题」并停止，不追加无关建议。
