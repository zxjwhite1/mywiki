这份关于 **Cline Skills (技能)** 的指南已为您整理并翻译为 Markdown 格式。我保留了所有技术细节、加载机制说明以及文件目录结构，确保内容的完整性。

---

# Cline Skills：扩展特定任务能力的模块化指令集

**Skills (技能)** 是用于扩展 Cline 在特定任务中能力的模块化指令集。每个技能都封装了详细的指导、工作流和可选资源，Cline 仅在与你的请求相关时才会加载它们。

你可以安装多个技能，而 Cline 只会加载它需要的那个。例如，部署技能会一直处于休眠状态，直到你询问有关部署的问题。与始终开启的 Rules（规则）不同，技能是**按需加载**的，因此在处理无关任务时不会消耗上下文。

> **注意**：Skills 目前是一项实验性功能。请在 `Settings` → `Features` → `Enable Skills` 中开启。

---

## 1. 技能的工作原理

技能采用**渐进式加载**以实现效率最大化：

|**级别**|**加载时机**|**Token 成本**|**内容**|
|---|---|---|---|
|**元数据 (Metadata)**|始终加载（启动时）|每个技能约 100 词|YAML Frontmatter 中的名称和描述|
|**指令 (Instructions)**|当技能被触发时|5k 词以下|`SKILL.md` 的正文内容与指导说明|
|**资源 (Resources)**|根据需要加载|实际上无限制|通过 `read_file` 访问的捆绑文件或执行的脚本|

当你发送消息时，Cline 会看到可用技能及其描述的列表。如果你的请求与某个技能的描述匹配，Cline 会通过 `use_skill` 工具激活它，并从 `SKILL.md` 加载完整指令。

---

## 2. 技能的结构

每个技能都是一个包含带有 YAML Frontmatter 的 `SKILL.md` 文件的目录。

**目录结构示例：**

```Plaintext
my-skill/
├── SKILL.md          # 必填：主要指令
├── docs/             # 可选：补充文档
│   └── advanced.md
└── scripts/          # 可选：工具脚本
    └── helper.sh
```

**SKILL.md 示例：**


```Markdown
---
name: my-skill
description: 简要描述该技能的作用及适用场景。
---

# My Skill (我的技能名称)

技能激活后，供 Cline 遵循的详细指令。

## 步骤
1. 首先，执行此操作
2. 然后，执行彼操作
3. 进阶用法请参考 [advanced.md](docs/advanced.md)
```

**必填字段：**

- **name**：必须与目录名称完全一致。
    
- **description**：告诉 Cline 何时使用该技能（最多 1024 字符）。
    

---

## 3. 创建技能

1. **打开 Skills 菜单**：点击 Cline 面板底部模型选择器左侧的“天平”图标，切换到 **Skills** 选项卡。
    
2. **创建新技能**：点击 “New skill...” 并输入名称（如 `aws-deploy`）。Cline 会创建一个包含模板文件的目录。
    
3. **编写指令**：
    
    - 更新 `description` 字段以明确触发条件。
        
    - 在正文中添加详细指令。
        
    - 可选：在 `docs/`、`templates/` 或 `scripts/` 子目录中添加支持文件。
        

> **提示**：将最重要的信息放在 `SKILL.md` 的顶部。Cline 是按顺序阅读文件的。使用清晰的二级标题（如 `## Error Handling`）方便 Cline 快速扫描相关章节。

---

## 4. 编写高效的 SKILL.md

### 命名规范

技能名称应使用小写字母并以连字符分隔（**kebab-case**），且具有描述性。

- **推荐**：`aws-cdk-deploy`, `pr-review-checklist`, `database-migration`
    
- **避免**：`aws`（太模糊）, `my_skill`（使用了下划线）, `DeployToAWS`（使用了大驼峰）
    

### 编写有效的描述 (Description)

描述决定了触发时机。模糊的描述会导致技能无法在预期时激活。

- **优秀示例**：`使用 CDK 将应用程序部署到 AWS。在部署、更新基础设施或管理 AWS 资源时使用。`
    
- **欠佳示例**：`帮助处理 AWS 相关事务。`
    

### 保持专注

保持 `SKILL.md` 在 5k Token 以内。如果内容过多，请拆分到 `docs/` 目录中。Cline 只有在需要时才会加载引用的文件。

---

## 5. 存储位置与优先级

技能可以存储在全局或项目工作区中。

- **项目技能 (Project Skills)**：
    
    - `.cline/skills/` (**推荐**)
        
    - `.clinerules/skills/`
        
- **全局技能 (Global Skills)**：
    
    - `~/.cline/skills/` (macOS/Linux)
        
    - `C:\Users\用户名\.cline\skills\` (Windows)
        

> **优先级说明**：当全局技能和项目技能重名时，**全局技能具有优先级**。这允许你保留通用的全局技能，同时在项目中使用特定技能供全队共享。

---

## 6. 捆绑支持文件

技能可以包含 Cline 仅在需要时才访问的额外文件。

### `docs/` (文档)

用于存放对于 `SKILL.md` 来说过于详细或仅在特定情况下才相关的信息。

- 进阶配置选项、边缘情况排查指南、API 模式引用等。
    

### `templates/` (模板)

用于技能创建配置文件、样板代码或结构化文档时。

- Terraform 配置、Docker Compose、`.env.example` 模板等。
    

### `scripts/` (脚本)

用于需要一致行为的确定性操作。

- 校验（Linting）、数据处理（解析、转换）、复杂计算（成本估算）。
    
- **优势**：脚本非常节省 Token，因为只有脚本的**输出**会进入上下文，而脚本本身的源代码不会。

---

## 7. 实用示例：数据分析技能

创建一个名为 `data-analysis/` 的目录，并编写以下 `SKILL.md`：



``` Markdown
---
name: data-analysis
description: 分析数据文件并生成见解。当处理需要探索、清理或可视化的 CSV、Excel 或 JSON 数据文件时使用。
---

# 数据分析

分析数据文件时，请遵循以下工作流：

## 1. 理解数据
- 读取文件样本以了解其结构。
- 识别列类型和数据质量问题。
- 记录任何缺失值或异常。

## 2. 询问澄清问题
在深入分析前，询问用户：
- 他们正在寻找哪些具体见解？
- 是否有已知的质量问题？
- 期望的输出格式是什么？

## 3. 执行分析
使用 pandas 进行数据处理：

import pandas as pd
df = pd.read_csv("data.csv")
print(df.describe())

```
对于可视化，根据复杂度优先选择 matplotlib 或 seaborn。


>技能将 Cline 从一个通用的助手转变为精通你业务领域的专家。建议从一个你经常重复的任务开始，测试并迭代描述，直到它能稳定触发。
