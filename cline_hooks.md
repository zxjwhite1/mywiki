这份关于 **Cline Hooks (钩子)** 的深度指南已为您整理并翻译为 Markdown 格式。我保留了所有代码逻辑、生命周期图示说明以及技术规范，确保内容的完整性。

---

# Cline Hooks：注入自定义逻辑以校验操作并引导决策

**Hooks (钩子)** 是在 Cline 工作流的关键时刻运行的脚本。由于它们在特定点执行，并具有一致的输入和输出，Hooks 通过强制执行护栏、校验和上下文注入，为 AI 模型的非确定性特征带来了“确定性”。你可以在操作执行前进行校验，监控工具使用情况，并塑造 Cline 的决策方式。

## 1. 你可以构建什么？

- **拦截风险操作**：在问题发生前停止操作（例如在 TypeScript 项目中阻止创建 `.js` 文件）。
    
- **自动化校验**：在文件保存前运行 linter（代码检查器）或自定义校验脚本。
    
- **安全合规**：阻止违反安全策略的操作。
    
- **审计监控**：跟踪所有活动以便进行分析或合规检查。
    
- **外部集成**：在合适的时机触发外部工具或服务。
    
- **动态上下文**：根据 Cline 正在执行的任务向对话中添加额外上下文。
    

---

## 2. 钩子类型

Cline 支持 8 种钩子类型，分别在任务生命周期的不同阶段运行：

|**钩子类型**|**运行时间**|
|---|---|
|**TaskStart**|开始一个新任务时|
|**TaskResume**|恢复一个被中断的任务时|
|**TaskCancel**|取消正在运行的任务时|
|**TaskComplete**|任务成功完成时|
|**PreToolUse**|Cline 执行工具之前（如 `read_file`, `write_to_file` 等）|
|**PostToolUse**|工具执行完成后|
|**UserPromptSubmit**|当你向 Cline 提交消息时|
|**PreCompact**|在 Cline 为了释放上下文空间而压缩对话历史之前|

``` mermaid
graph TD
    %% 节点定义
    Start((开始))
    NewOrResume{新任务或继续?}
    
    TaskStart[TaskStart <br/>任务开始]
    TaskResume[TaskResume <br/>任务恢复]
    
    subgraph ConversationCycle [对话循环]
        TaskActiveLoop((任务激活循环))
        UserPromptSubmit[UserPromptSubmit <br/>用户提交提示词]
        ClineProcessesContext[Cline 处理上下文]
        
        PreCompact[PreCompact <br/>压缩前]
        PreToolUse[PreToolUse <br/>使用工具前]
        ToolExecutes{工具执行}
        PostToolUse[PostToolUse <br/>使用工具后]
    end
    
    TaskComplete[TaskComplete <br/>任务完成]
    TaskCancel[TaskCancel <br/>任务取消]
    End((结束))

    %% 流程连接
    Start --> NewOrResume
    NewOrResume -- 新任务 --> TaskStart
    NewOrResume -- 继续 --> TaskResume
    
    TaskStart --> TaskActiveLoop
    TaskResume --> TaskActiveLoop
    
    TaskActiveLoop -- 用户发送消息 --> UserPromptSubmit
    UserPromptSubmit --> ClineProcessesContext
    
    ClineProcessesContext -. 达到上下文限制 .-> PreCompact
    PreCompact -.-> ClineProcessesContext
    
    ClineProcessesContext -- 决定使用工具 --> PreToolUse
    PreToolUse -- 允许 --> ToolExecutes
    PreToolUse -. 已取消 .-> ClineProcessesContext
    
    ToolExecutes --> PostToolUse
    PostToolUse --> ClineProcessesContext
    
    ClineProcessesContext -- 任务成功结束 --> TaskComplete
    TaskActiveLoop -- 用户取消任务 --> TaskCancel
    
    TaskComplete --> End
    TaskCancel --> End

    %% 样式美化
    style Start fill:#f9f,stroke:#333
    style End fill:#f9f,stroke:#333
    style NewOrResume fill:#e1d5e7,stroke:#9673a6
    style TaskStart fill:#ffb366,stroke:#d79b00
    style TaskResume fill:#ffb366,stroke:#d79b00
    style UserPromptSubmit fill:#ffb366,stroke:#d79b00
    style PreCompact fill:#ffb366,stroke:#d79b00
    style PreToolUse fill:#ffb366,stroke:#d79b00
    style PostToolUse fill:#ffb366,stroke:#d79b00
    style TaskComplete fill:#ffb366,stroke:#d79b00
    style TaskCancel fill:#ffb366,stroke:#d79b00
    style TaskActiveLoop fill:#dae8fc,stroke:#6c8ebf
    style ClineProcessesContext fill:#dae8fc,stroke:#6c8ebf
    style ConversationCycle fill:#fff2cc,stroke:#d6b656
```

该流程图展示了完整的钩子（Hook）生命周期：

- **入口 (Entry)**：当您开始一个任务时，首先运行的是 **TaskStart**（新任务）或 **TaskResume**（中断的任务）。
    
- **对话循环 (Conversation Cycle)**：每当您发送一条消息时，**UserPromptSubmit** 就会运行，随后 Cline 会处理您的请求。
    
- **工具执行 (Tool Execution)**：当 Cline 决定使用某个工具时，**PreToolUse** 会首先运行；如果获得允许，工具将执行，随后 **PostToolUse** 运行。
    
- **上下文管理 (Context Management)**：如果对话接近上下文限制，在进行截断处理前会运行 **PreCompact**。
    
- **出口 (Exit)**：任务以 **TaskComplete**（成功）或 **TaskCancel**（用户取消）结束。
    

图中**橙色节点**代表您可以注入自定义逻辑的钩子。随着对话的持续，该循环会不断重复。


---

## 3. 存储位置

钩子可以存储在全局或项目工作区中：

- **全局钩子**：`~/Documents/Cline/Hooks/`
    
- **项目钩子**：仓库中的 `.clinerules/hooks/`（可提交至版本控制）。
    

> **注意**：如果同一类型的全局钩子和项目钩子同时存在，两者都会运行。**全局钩子先执行，然后是项目钩子**。如果任一钩子返回 `cancel: true`，操作将停止。

---

## 4. 快速上手：你的第一个钩子

让我们创建一个简单的钩子，记录 Cline 读取或写入的每个文件。

### 脚本内容 (file-logger)

在你的钩子目录中创建一个名为 `file-logger` 的文件：

Bash

```
#!/bin/bash
# 将所有文件操作记录到 ~/cline-activity.log

INPUT=$(cat)
TOOL=$(echo "$INPUT" | jq -r '.preToolUse.tool')
FILE_PATH=$(echo "$INPUT" | jq -r '.preToolUse.parameters.path // "N/A"')

# 记录到文件
echo "$(date '+%H:%M:%S') - $TOOL: $FILE_PATH" >> ~/cline-activity.log

# 始终允许操作执行
echo '{"cancel":false}'
```

### 设置步骤

1. **创建文件**：保存上述脚本。
    
2. **赋予执行权限** (macOS/Linux)：运行 `chmod +x file-logger`。
    
3. **启用钩子**：在 Cline 的 **Hooks** 选项卡中，找到 `PreToolUse` 下的 `file-logger` 并打开开关。
    

> **Windows 用户须知**：Windows 上的钩子使用 PowerShell 执行，文件名必须为 `HookName.ps1`（例如 `PreToolUse.ps1`）。目前 Windows 暂不支持通过 UI 切换开关，只要文件存在即视为启用。

---

## 5. 工作原理：输入与输出

Hooks 是通过 **stdin (标准输入)** 接收 JSON，并通过 **stdout (标准输出)** 返回 JSON 的可执行脚本。

### 输入结构 (Input)

每个钩子都会收到一个包含通用字段和特定字段的 JSON 对象：

JSON

```
{
  "taskId": "abc123",
  "clineVersion": "3.17.0",
  "workspacePath": "/path/to/project",
  "preToolUse": {
    "tool": "write_to_file",
    "parameters": { "path": "src/config.ts", "content": "..." }
  }
}
```

### 输出结构 (Output)

钩子必须向 stdout 返回一个 JSON 对象：

JSON

```
{
  "cancel": false,
  "contextModification": "可选的注入文本",
  "errorMessage": "取消操作时显示的错误信息"
}
```

- **cancel**: 如果为 `true`，则停止当前操作（阻止工具执行或取消任务启动）。
    
- **contextModification**: 注入到对话中的额外上下文。例如：“注意：此文件是自动生成的，修改可能会被覆盖。”
    

---

## 6. 实用示例

### 强制执行 TypeScript 规范 (PreToolUse)

阻止在项目中创建 `.js` 文件：

Bash

```
#!/bin/bash
INPUT=$(cat)
TOOL=$(echo "$INPUT" | jq -r '.preToolUse.tool')
FILE_PATH=$(echo "$INPUT" | jq -r '.preToolUse.parameters.path // empty')

if [[ "$TOOL" == "write_to_file" && "$FILE_PATH" == *.js ]]; then
  echo '{"cancel":true,"errorMessage":"请在本 TypeScript 项目中使用 .ts 文件而非 .js"}'
  exit 0
fi
echo '{"cancel":false}'
```

### 工具使用审计 (PostToolUse)

记录工具执行是否成功及耗时：

Bash

```
#!/bin/bash
INPUT=$(cat)
TOOL=$(echo "$INPUT" | jq -r '.postToolUse.tool')
SUCCESS=$(echo "$INPUT" | jq -r '.postToolUse.success')
DURATION=$(echo "$INPUT" | jq -r '.postToolUse.durationMs')

echo "$(date -Iseconds) | $TOOL | success=$SUCCESS | ${DURATION}ms" >> ~/.cline-tool-log.txt
echo '{"cancel":false}'
```

### 任务启动时注入环境上下文 (TaskStart)

Bash

```
#!/bin/bash
INPUT=$(cat)
WORKSPACE=$(echo "$INPUT" | jq -r '.workspacePath')

if [[ -f "$WORKSPACE/.project-context" ]]; then
  CONTEXT=$(cat "$WORKSPACE/.project-context")
  echo "{\"cancel\":false,\"contextModification\":\"项目特定上下文: $CONTEXT\"}"
else
  echo '{"cancel":false}'
fi
```

---

## 7. 故障排除

- **钩子未运行？**
    
    - **macOS/Linux**: 检查是否执行了 `chmod +x`。
        
    - **Windows**: 确保文件名符合 `HookName.ps1` 规范，且 PowerShell 在 PATH 路径中。
        
    - **UI 检查**: 确保在“设置”中全局启用了 Hooks，且特定钩子的开关已打开。
        
- **输出无法解析？**
    
    - 确保输出是 **单行且合法** 的 JSON。
        
    - 调试信息应输出到 **stderr** (`>&2`)，不要占用 stdout。
        
- **意外拦截？**
    
    - 检查逻辑判断，并同时确认全局和项目钩子是否产生了冲突。
        

---

**下一步：你想让我为你编写一个具体的 PowerShell 版 (Windows) 钩子，还是为特定的工作流定制一个 PreToolUse 校验脚本？**
