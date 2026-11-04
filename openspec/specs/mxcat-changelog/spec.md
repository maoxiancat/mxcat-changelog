# mxcat-changelog Specification

## Purpose

规定 Agent 在用户要求更新项目 CHANGELOG 时，如何选定文件、安排版本节，以及如何写入版本号、日期、条目类型、scope 和描述。本规格从现有技能正文反推，描述当前必须遵守的行为。

## Requirements

### Requirement: 仅在用户要求时更新

Agent MUST 仅在用户明确要求更新 CHANGELOG 时写入更新文档。代码或其他文件变更完成之后，Agent MUST NOT 自行更新 CHANGELOG。

#### Scenario: 用户要求更新

- **WHEN** 用户要求更新 CHANGELOG
- **THEN** Agent 按本规格写入本次更新

#### Scenario: 代码变更后未要求更新

- **WHEN** 代码或其他文件已经变更，且用户没有要求更新 CHANGELOG
- **THEN** Agent 不修改 CHANGELOG

### Requirement: 只约束本次新增

无特殊要求时，本规格 MUST 只约束本次新增的 CHANGELOG 内容。Agent MUST NOT 改写更新文档里已经存在的标题、日期、条目或空行。

#### Scenario: 补写新版本节

- **WHEN** 用户要求更新 CHANGELOG，且没有要求改动已有内容
- **THEN** Agent 只新增本次需要的版本节和条目
- **AND** 已有版本节保持原样

#### Scenario: 用户要求改动已有内容

- **WHEN** 用户明确要求修改已有 CHANGELOG 内容
- **THEN** Agent 按该要求修改对应的已有内容

### Requirement: 沿用项目已有风格

目标项目已经有明确 CHANGELOG 风格时，Agent MUST 沿用该风格更新。项目风格没有覆盖的部分，Agent MUST 使用本规格的格式与内容规则。

#### Scenario: 项目已有明确风格

- **WHEN** 目标项目的 CHANGELOG 已经形成明确风格
- **THEN** 本次新增内容沿用该风格

#### Scenario: 项目没有既定风格

- **WHEN** 目标项目没有可沿用的 CHANGELOG 风格
- **THEN** 本次新增内容使用本规格的格式与内容规则

### Requirement: 只更新 CHANGELOG 文件

执行 CHANGELOG 更新时，Agent MUST 只修改 CHANGELOG 文件。Agent MUST NOT 为此创建、移动或删除 git tag，也 MUST NOT 做与这次更新无关的其他操作。

#### Scenario: 写入预测版本

- **WHEN** Agent 为新版本节预测出版本号并写入标题
- **THEN** 该版本号只出现在更新文档的版本节标题中
- **AND** 仓库中的 git tag 不被创建、移动或删除

### Requirement: CHANGELOG 文件位置

目标项目中已有 CHANGELOG 文件时，Agent MUST 沿用该文件。没有 CHANGELOG 文件时，Agent MUST 在项目根目录创建。

#### Scenario: 已有 CHANGELOG

- **WHEN** 项目中已经存在 CHANGELOG 文件
- **THEN** Agent 更新该文件

#### Scenario: 没有 CHANGELOG

- **WHEN** 项目中不存在 CHANGELOG 文件
- **THEN** Agent 在项目根目录创建 CHANGELOG 并写入本次内容

### Requirement: 文档结构

新增 CHANGELOG MUST 使用唯一的一级标题 `# 更新文档`。每个版本节 MUST 使用二级标题 `## X.Y.Z（YYYY-MM-DD）`，其中 `X.Y.Z` 为版本号，日期放在全角括号 `（）` 内。全文版本节 MUST 自上而下按发布日期递减排列。

版本节中的条目 MUST 使用以下两种形态之一：

- shortcode：`- :shortcode: (scope) 描述`
- Conventional Commits 文字类型：`- type(scope): subject`

同一批新增内容 MUST NOT 混用这两套形态。条目可以不写 scope；写了 scope 时 MUST 使用半角括号。节点之间 MUST 保留一个空行，标题和其内容之间 MUST 保留一个空行。

#### Scenario: 新增一份短码风格的版本节

- **WHEN** 本次新增使用 shortcode，版本为 `0.2.0`，日期为 `2026-11-02`，且包含带 scope 与不带 scope 的条目
- **THEN** 新增内容符合下列结构

```md
# 更新文档

## 0.2.0（2026-11-02）

- :sparkles: (content) 增加场景内容
- :wrench: 更新工程配置
```

#### Scenario: 多个版本节按日期排列

- **WHEN** 更新文档同时存在多个版本节
- **THEN** 发布日期较新的版本节排在较旧的版本节之上

### Requirement: 版本号形态与比较

版本号 MUST 形如 `X.Y.Z`。比较版本前 MUST 去掉前缀和后缀（例如 `v`），再按 `X`、`Y`、`Z` 的数字比较。

确定版本号时，Agent MUST 先读取仓库提交和已有 tag。读取 tag 时 MUST 使用：

```bash
git tag --list
git for-each-ref --sort=v:refname --format '%(refname:short) %(creatordate:short)' refs/tags
```

已记录版本是 `# 更新文档` 下各节标题 `## X.Y.Z（YYYY-MM-DD）` 中的 `X.Y.Z`。其中最高的一个是已记录版本。还没有版本节时，已记录版本为空。

#### Scenario: 带前缀的 tag 参与比较

- **WHEN** 仓库 tag 为 `v0.2.0`，更新文档已记录版本为 `0.1.0`
- **THEN** 比较时把该 tag 视为 `0.2.0`
- **AND** 该 tag 高于已记录版本

#### Scenario: 尚无版本节

- **WHEN** `# 更新文档` 下还没有版本节
- **THEN** 已记录版本为空

### Requirement: 用户给定版本号

用户已经给出 `X.Y.Z` 时，新版本节标题 MUST 使用该版本号。Agent MUST NOT 再为该节预测版本号。

#### Scenario: 用户指定版本

- **WHEN** 用户给出版本号 `1.4.0`
- **THEN** 本次新版本节标题使用 `1.4.0`
- **AND** Agent 不根据提交预测另一个版本号

### Requirement: 按 tag 补全缺失版本

高于已记录版本的 tag MUST 先补进更新文档，之后才允许预测。低于或等于已记录版本的 tag MUST NOT 补入。每个待补 tag MUST 单独成节，并按版本从低到高写入。标题版本号 MUST 是去掉前缀和后缀后的 `X.Y.Z`。日期 MUST 使用该 tag 的 `creatordate`，并取本地日历日期。

#### Scenario: tag 超前于更新文档

- **WHEN** 已记录版本为 `0.1.0`，且存在更高的 tag `v0.2.0` 与 `v0.3.0`
- **THEN** Agent 先补入 `0.2.0` 节，再补入 `0.3.0` 节
- **AND** 两节日期分别是对应 tag 的本地日历创建日期

#### Scenario: tag 不超过已记录版本

- **WHEN** tag 去掉前缀和后缀后低于或等于已记录版本
- **THEN** Agent 不为该 tag 补版本节

### Requirement: 预测下一版本

预测的起点 MUST 是补全之后的最高版本。没有 tag、也没有已记录版本时，起点 MUST 是 `0.0.0`。预测 MUST 只看起点之后的提交和尚未提交的改动。

同一批改动 MUST 只抬一档，且同一档只加一。加哪一档取决于这些改动的范围和量：范围是改动波及的面有多广，量是这个范围内改动有多少。范围和量对得上更高一档时 MUST 取更高档，判断顺序是 `X`、`Y`、`Z`。

- `X` 加一：影响的范围广，量也大；`Y` 和 `Z` 归零
- `Y` 加一：影响的范围限于一部分，量居中；`Z` 归零
- `Z` 加一：影响的范围窄，量小

`1.0.0` 是特殊节点。已有 tag 尚未达到 `1.0.0` 时，预测结果 MUST NOT 达到 `1.0.0`。

#### Scenario: 没有任何版本起点

- **WHEN** 仓库没有 tag，更新文档也没有已记录版本
- **THEN** 预测从 `0.0.0` 起算

#### Scenario: 小范围少量改动

- **WHEN** 起点为 `0.2.0`，且起点之后的改动范围窄、量小
- **THEN** 预测版本为 `0.2.1`

#### Scenario: 局部中等改动

- **WHEN** 起点为 `0.2.1`，且起点之后的改动范围限于一部分、量居中
- **THEN** 预测版本为 `0.3.0`

#### Scenario: 大范围大量改动仍不能跨过 1.0.0

- **WHEN** 已有 tag 都低于 `1.0.0`，起点为 `0.4.0`，且起点之后的改动范围广、量大
- **THEN** 预测结果不到达 `1.0.0`

### Requirement: 发布日期

用户没有指定日期时，本次新版本节的日期 MUST 使用写入 CHANGELOG 当天的本地日历日期。补全既有 tag 时，日期 MUST 使用该 tag 的 `creatordate` 所对应的本地日历日期。用户指定了日期时，标题 MUST 使用用户指定的日期。

#### Scenario: 未指定日期

- **WHEN** 用户没有指定发布日期，且本次写入发生在本地日期 `2026-11-02`
- **THEN** 新版本节标题中的日期为 `2026-11-02`

#### Scenario: 用户指定日期

- **WHEN** 用户指定发布日期为 `2026-10-30`
- **THEN** 新版本节标题中的日期为 `2026-10-30`

### Requirement: 条目类型选择

条目类型 MUST 是 shortcode 或 Conventional Commits 文字类型，两套 MUST NOT 混写。项目已经使用其中一种时，Agent MUST 沿用那一种。项目两种都没有使用时，Agent MUST 使用 shortcode。

#### Scenario: 项目已使用 Conventional Commits 文字类型

- **WHEN** 目标 CHANGELOG 已经使用 `feat`、`fix` 这类 Conventional Commits 文字类型
- **THEN** 本次新增条目继续使用 Conventional Commits 文字类型
- **AND** 不写入 shortcode

#### Scenario: 项目没有既有类型

- **WHEN** 目标 CHANGELOG 尚未使用 shortcode 或 Conventional Commits 文字类型
- **THEN** 本次新增条目使用 shortcode

### Requirement: shortcode 词表

选用 shortcode 时，类型 MUST 取自下表，并写成 `:shortcode:`：

| Shortcode | 含义 | 使用场景 |
| --- | --- | --- |
| `:sparkles:` | 新增功能 | 新增功能、页面、组件等 |
| `:bug:` | 缺陷修复 | 修复功能异常 |
| `:boom:` | 破坏性变更 | 变更已发布的接口、行为或配置，既有用法无法沿用 |
| `:memo:` | 文档变更 | 修改文档，不改变程序行为 |
| `:art:` | 格式调整 | 调整空白、缩进、标点等排版，不改变代码逻辑 |
| `:recycle:` | 代码重构 | 调整内部实现与结构，不改变对外行为 |
| `:zap:` | 性能优化 | 降低耗时、减少资源占用或提升吞吐 |
| `:white_check_mark:` | 测试相关 | 新增、修正或补充测试，不改变程序行为 |
| `:package:` | 构建相关 | 调整构建脚本、编译配置、依赖安装或打包工具链 |
| `:construction_worker:` | CI 配置 | 调整持续集成配置与流水线脚本 |
| `:wrench:` | 杂项维护 | 依赖、工具升级与维护及其他不影响源码语义的事务 |
| `:rewind:` | 回退变更 | 撤销先前提交所引入的改动 |
| `:lipstick:` | 界面样式 | 调整颜色、间距、字体等外观，不改变交互行为 |
| `:fire:` | 代码移除 | 移除功能、模块或不再维护的文件 |
| `:tada:` | 项目初始化 | 仓库初次建立、工程初始化或首次提交 |
| `:twisted_rightwards_arrows:` | 合并分支 | 合并其他分支，或在解决冲突后形成的合并提交 |
| `:bookmark:` | 版本发布 | 更新版本号、创建发布标签或撰写发布说明 |
| `:ambulance:` | 紧急修复 | 紧急处理已发布环境中的故障，需快速上线 |
| `:construction:` | 尚未完成 | 功能尚未完成时的阶段性提交 |
| `:label:` | 类型变更 | 修改静态类型、接口或声明，不改变运行时逻辑 |
| `:see_no_evil:` | 忽略规则 | 新增或更新忽略规则文件，排除生成物、密钥或本地文件 |
| `:card_file_box:` | 数据库变更 | 调整表结构、迁移脚本或数据查询 |

#### Scenario: 新增功能使用 shortcode

- **WHEN** 本次变更是新增功能，且当前使用 shortcode
- **THEN** 条目类型为 `:sparkles:`

### Requirement: Conventional Commits 文字类型词表

选用 Conventional Commits 文字类型时，类型 MUST 取自下表：

| 类型 | 含义 | 使用场景 |
| --- | --- | --- |
| `feat` | 新增功能 | 新增功能、页面、组件等 |
| `fix` | 缺陷修复 | 修复功能异常 |
| `docs` | 文档变更 | 修改文档，不改变程序行为 |
| `style` | 格式调整 | 调整空白、缩进、标点等排版，不改变代码逻辑 |
| `refactor` | 代码重构 | 调整内部实现与结构，不改变对外行为 |
| `perf` | 性能优化 | 降低耗时、减少资源占用或提升吞吐 |
| `test` | 测试相关 | 新增、修正或补充测试，不改变程序行为 |
| `build` | 构建相关 | 调整构建脚本、编译配置、依赖安装或打包工具链 |
| `ci` | CI 配置 | 调整持续集成配置与流水线脚本 |
| `chore` | 杂项维护 | 依赖、工具升级与维护及其他不影响源码语义的事务 |
| `revert` | 回退变更 | 撤销先前提交所引入的改动 |
| `init` | 项目初始化 | 仓库初次建立、工程初始化或首次提交 |
| `merge` | 合并分支 | 合并其他分支，或在解决冲突后形成的合并提交 |
| `release` | 版本发布 | 更新版本号、创建发布标签或撰写发布说明 |
| `hotfix` | 紧急修复 | 紧急处理已发布环境中的故障，需快速上线 |
| `wip` | 尚未完成 | 功能尚未完成时的阶段性提交 |
| `types` | 类型变更 | 修改静态类型、接口或声明，不改变运行时逻辑 |

#### Scenario: 缺陷修复使用 Conventional Commits 类型

- **WHEN** 本次变更是缺陷修复，且当前使用 Conventional Commits 文字类型
- **THEN** 条目类型为 `fix`

### Requirement: scope

scope MUST 标识变化落在哪一块。它 MUST 是一段不含空白的全小写英文标识，可以含连字符，并放在半角括号里。Agent MUST NOT 按文件路径拼接 scope，也 MUST NOT 在一个条目里并列多个范围。

#### Scenario: 单一范围

- **WHEN** 变更落在内容区块
- **THEN** 条目写成 `(content)` 这种单一全小写标识

#### Scenario: 拒绝多个范围

- **WHEN** 一次变更同时涉及认证和接口
- **THEN** 条目不写成 `(auth,api)`

### Requirement: 描述

描述 MUST 使用简体中文，并以精确的语言说明新增的功能或造成的修改。

#### Scenario: 描述新增功能

- **WHEN** 条目记录一项新增功能
- **THEN** 描述为简体中文，并写明新增了什么

#### Scenario: shortcode 条目示例

- **WHEN** 使用 shortcode 记录内容与技能相关变更
- **THEN** 条目可以写成：

```md
- :sparkles: (content) 增加场景用的植物、技能、书、画框内容，以及作品预览图和技能图标
- :wrench: (skills) 加入代码规范与提交技能
```

#### Scenario: Conventional Commits 条目示例

- **WHEN** 使用 Conventional Commits 文字类型记录加载行为变更
- **THEN** 条目可以写成：

```md
- feat(loading): 等场景就绪后再结束加载
```
