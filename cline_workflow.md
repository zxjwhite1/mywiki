这份关于 **Cline Workflows (工作流)** 的指南已为您整理并翻译为 Markdown 格式。我保留了所有代码示例、工具语法以及操作步骤，确保内容的完整性。

---

# Cline Workflows：使用 Markdown 文件自动化重复任务

**Workflows (工作流)** 是定义了一系列步骤的 Markdown 文件，用于引导 Cline 完成重复性或复杂的任务。只需在聊天框输入 `/` 紧跟工作流文件名即可调用（例如：`/deploy.md`）。

部署项目、搭建新环境、执行发布检查清单：这些任务通常需要记住几十个步骤，按正确顺序运行命令，并手动更新文件。只要错了一步，可能就要花一个小时来调试。**Workflows 将这些多步骤过程转化为一个命令。** 输入 `/release.md`，Cline 就能处理版本升级、运行测试、更新变更日志、提交、打标签并推送。你只需要审查并批准即可。

---

## 1. 工作流结构

工作流是一个包含标题和步骤的 Markdown 文件。文件名即是命令：`demo-workflow.md` 通过 `/demo-workflow.md` 调用。

**demo-workflow.md 示例：**

Markdown

````
# 演示工作流
简要描述该工作流实现的目标。

## 步骤 1：检查前置条件
验证环境是否准备就绪。查找所需的工具和依赖项。

## 步骤 2：运行构建
执行构建命令：
```bash
npm run build
````

## 步骤 3：验证结果

检查构建是否成功完成，并报告任何问题。

````

**步骤描述的深度可以灵活调整：**
* **高层级描述**：“运行测试套件并修复任何失败项”，让 Cline 自行决定如何达成目标。
* **具体描述**：当你需要精准控制时，使用 XML 工具语法或确切的命令。

---

## 2. 创建工作流
1.  **打开 Workflows 菜单**：点击 Cline 面板底部模型选择器左侧的“天平”图标，切换到 **Workflows** 选项卡。
2.  **创建新文件**：点击 “New workflow file...” 并输入文件名（如 `deploy`）。文件将以 `.md` 扩展名创建。
3.  **编写工作流**：添加标题和带编号的步骤。描述每个步骤应完成的任务。

> **专家技巧：从已完成的任务创建工作流。** 在完成一项以后需要重复的任务后，直接告诉 Cline：“为我刚才完成的过程创建一个工作流。”Cline 会分析对话，识别步骤并生成工作流文件。你积累的上下文瞬间变成了可复用的自动化工具。

---

## 3. 调用与管理
* **调用**：在聊天输入框输入 `/` 即可看到可用工作流。支持自动补全（例如输入 `/rel` 会匹配 `release-prep.md`）。
* **执行**：Cline 会按顺序执行每一步，在需要时暂停等待你的批准。你可以通过拒绝某个步骤随时停止工作流。
* **切换开关**：每个工作流都有一个开关，允许你控制哪些工作流显示在 `/` 菜单中。

---

## 4. 存储位置
工作流可以存储在两个位置：

* **工作区工作流 (Workspace Workflows)**：存放在项目根目录的 `.clinerules/workflows/`。适用于项目特有的自动化，如部署脚本或团队共享的设置程序。
* **全局工作流 (Global Workflows)**：存放在系统的 Cline Workflows 目录中。适用于跨项目的个人生产力工作流。

**全局目录位置：**
| 操作系统 | 默认位置 |
| :--- | :--- |
| **Windows** | `Documents\Cline\Workflows` |
| **macOS** | `~/Documents/Cline/Workflows` |
| **Linux/WSL** | `~/Documents/Cline/Workflows` |

*注：重名时，工作区工作流优先级更高。*

---

## 5. 工作流可以调用的资源
工作流可以将自然语言指令与具体的工具调用结合使用。

### 自然语言
直接编写纯文本指令，Cline 会理解并确定使用哪些工具：
```markdown
## 步骤 1：检查未提交的更改
查看 git 状态。如果有未提交的更改，询问是继续还是中止。
````

### Cline 内置工具 (XML 语法)

为了精确控制，使用 Cline 的内置工具：

XML

```
<execute_command>
  <command>npm run test</command>
  <requires_approval>false</requires_approval>
</execute_command>

<ask_followup_question>
  <question>部署到生产环境还是测试环境？</question>
  <options>["Production", "Staging", "Cancel"]</options>
</ask_followup_question>
```

### CLI 工具

引用你机器上安装的任何命令行工具（git, npm, docker, curl 等）。

### MCP 工具

如果你连接了 **MCP 服务器**，可以使用 `use_mcp_tool` 语法：

XML

```
<use_mcp_tool>
  <server_name>github-server</server_name>
  <tool_name>create_release</tool_name>
  <arguments>{"tag": "v1.2.0", "name": "Release v1.2.0"}</arguments>
</use_mcp_tool>
```

---

## 6. 编写高效工作流的建议

- **从简单开始**：先写自然语言步骤，只有在需要保证特定行为时才添加 XML 工具调用。
    
- **明确决策点**：如果步骤需要用户输入，请明确说明：“询问用户部署到生产环境还是预发环境。”
    
- **包含失败处理**：告诉 Cline 出错时怎么办：“如果测试失败，显示错误并停止工作流。”
    
- **保持专注**：一个工作流只做一件事。将复杂过程拆分为多个独立的可运行工作流。
    
- **版本控制**：将工作流提交到 Git 仓库，以便团队共享和审查。
    

> **⚠️ 安全提醒**：工作流以你的权限执行。在运行之前请务必审查，尤其是来自外部来源的工作流。

---

## 7. 示例：发布准备 (Release Preparation)

该工作流自动化了繁琐的发布前检查。它会验证目录是否干净、运行测试和构建、提示版本升级，并根据近期提交生成变更日志。

**release-prep.md**

Markdown

```
# 发布准备
通过运行测试、构建和更新版本信息来准备新版本。

## 步骤 1：检查工作目录是否干净
<execute_command>
<command>git status --porcelain</command>
</execute_command>
如果有未提交的更改，询问是继续还是先将它们 stash。

## 步骤 2：运行测试套件
<execute_command>
<command>npm run test</command>
</execute_command>
如果任何测试失败，停止工作流并报告错误。

## 步骤 3：构建项目
<execute_command>
<command>npm run build</command>
</execute_command>
验证构建是否无错完成。

## 步骤 4：询问新版本号
<ask_followup_question>
<question>新版本号应该是？</question>
<options>["Patch (x.x.X)", "Minor (x.X.0)", "Major (X.0.0)", "Custom"]</options>
</ask_followup_question>

## 步骤 5：更新版本
将 `package.json` 中的版本号更新为用户指定的版本。

## 步骤 6：生成变更日志条目
<execute_command>
<command>git log --oneline $(git describe --tags --abbrev=0)..HEAD</command>
</execute_command>
利用这些提交记录为新版本编写变更日志条目。
```

使用 `/release-prep.md` 调用它，Cline 就会引导你完成每一个步骤。

---

**下一步：你想让我为你当前的某个具体流程（比如配置一个新的 Docker 环境或自动化 Git 分支清理）生成一个工作流草案吗？**
