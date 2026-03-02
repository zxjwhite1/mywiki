这份关于 **.clineignore** 的指南已为您翻译并整理完毕。这是控制 Cline 上下文效率最直接的工具。

---

# .clineignore：控制项目访问权限

**.clineignore** 文件用于告知 Cline 在分析代码库时应跳过哪些文件和目录。它的工作原理与 `.gitignore` 完全一致：在项目根目录创建一个名为 `.clineignore` 的文件，添加你想要排除的文件模式，Cline 就会自动忽略它们。

## 1. 为什么它很重要？

如果没有 `.clineignore`，Cline 可能会将整个项目加载到上下文中，包括依赖项、构建产物和自动生成的文件。这会导致：

- **浪费 Token**：增加不必要的 API 开销。
    
- **增加成本**：模型处理的信息越多，费用越高。
    
- **稀释上下文**：无关信息会挤掉真正有用的代码片段，导致 Cline 的响应质量下降。
    

通过添加 `.clineignore`，你可以将初始上下文从 200k+ Token 削减到 50k 以下。这意味着**更快的响应速度**、**更低的成本**，以及能够更有效地使用体量较小、价格更低的模型。

---

## 2. 创建 .clineignore

在项目根目录创建文件并添加如下示例内容：

Plaintext

```
# 依赖项
node_modules/
**/node_modules/

# 构建输出
/build/
/dist/
/.next/
/out/

# 测试产物
/coverage/

# 环境变量
.env
.env.*

# 大型数据文件
*.csv
*.xlsx
*.sqlite

# 生成/压缩后的代码
*.min.js
*.map
```

### 匹配模式语法

|**模式**|**匹配范围**|
|---|---|
|`node_modules/`|匹配 `node_modules` 目录|
|`**/node_modules/`|匹配任何深度的 `node_modules` 目录|
|`*.csv`|匹配所有 CSV 文件|
|`/build/`|仅匹配项目根目录下的 `build` 目录|
|`*.env.*`|匹配类似 `.env.local`, `.env.production` 的文件|
|`!important.csv`|**例外**：不忽略此特定文件|

---

## 3. 排除建议

### 几乎总是需要排除的内容：

- **包管理器目录** (`node_modules/`, `vendor/`, `.venv/`)。
    
- **构建产物** (`dist/`, `build/`, `.next/`, `out/`)。
    
- **覆盖率报告** (`coverage/`)。
    
- **大型锁定文件**（如果文件极大：`package-lock.json`, `yarn.lock`）。
    

### 视情况排除的内容：

- **大型数据文件** (`.csv`, `.xlsx`, `.sqlite`, `.parquet`)。
    
- **二进制资源**（图片、字体、视频）。
    
- **生成的代码**（API 客户端代码、Protobuf 输出、压缩后的代码包）。
    
- **敏感环境变量** (`.env`, `.env.local`)。
    

### 保持可访问的内容：

- 你正在积极开发的**源代码**。
    
- Cline 需要理解的**配置文件** (`tsconfig.json`, `package.json`)。
    
- **文档**和 README 文件。
    
- **测试文件**（Cline 通常需要这些来获取上下文以编写新测试）。
    

---

## 4. 工作原理

当 Cline 扫描项目以构建上下文时，它会对比 `.clineignore`。匹配的文件将从以下场景中排除：

1. 开始任务时 Cline 看到的**文件列表**。
    
2. 对话过程中的**自动上下文收集**。
    
3. Cline 查找相关代码时的**搜索结果**。
    

> **提示**：你仍然可以使用 `@` 显式引用被忽略的文件。如果你输入 `@/node_modules/some-package/index.js`，尽管它在忽略列表中，Cline 依然会读取该特定文件。忽略规则仅控制**自动加载**，不限制**显式访问**。

---

## 5. 实用技巧

- **及早添加**：在项目初期就添加 `.clineignore`。从广泛排除开始，后续再根据需要缩小范围，比以后调试为什么上下文过载要容易得多。
    
- **监控 Token**：添加忽略文件后，检查任务顶部的 Token 使用情况，差异通常非常显著。
    
- **排查缺失**：如果 Cline 似乎找不到某个文件，请先检查它是否被你的忽略模式误伤了。
    
- **单文件独立性**：`.clineignore` 是独立于 `.gitignore` 的。某些文件虽然受 Git 跟踪（如大型测试固件），但如果对 Cline 没用，也应该加入 `.clineignore`。
    

---

### 相关功能

- **Cline Rules**：为 Cline 定义持久的指令。
    
- **Auto-Compact**：在长对话任务中自动压缩上下文。
    
- **Memory Bank**：跨会话上下文的结构化文档。
    

---

这是 Cline 定制化五个系统的最后一部分。既然你已经掌握了 **Rules, Skills, Workflows, Hooks, 和 .clineignore**，你想尝试针对你的具体项目配置其中哪一个吗？
