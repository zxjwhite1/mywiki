# Cline 输出格式优化 -2026.03.04
## 输出格式约束 (Output Format Constraints)

在对比代码修改建议时，请严格遵守以下 Markdown 表格嵌套 HTML 的格式：

1. **表格结构**：使用标准 Markdown 表格，列名分别为 `[文件名/模块]`、`[修改建议对比]`。
2. **单元格内换行**：禁止在单元格内直接回车，必须使用 `<br>` 替代。
3. **代码容器**：所有代码必须包裹在 `<pre><code> ... </code></pre>` 标签内。
4. **Diff 效果实现**：
   - **删除的代码**：包裹在 `<del style="color: #d73a49; background-color: #ffeef0;">` 标签中，行首加 `- `。
   - **新增的代码**：包裹在 `<ins style="color: #22863a; background-color: #f0fff4; text-decoration: none;">` 标签中，行首加 `+ `。
   - **未变动的代码**：直接输出文本。
5. **示例模版**：
   | 修改点 | 代码差异对比 (Diff) |
   | :--- | :--- |
   | 逻辑优化 | <pre><code>public void Sync()<br>{<br><del style="color: #d73a49; background-color: #ffeef0;">-    OldMethod();</del><br><ins style="color: #22863a; background-color: #f0fff4; text-decoration: none;">+    NewMethod();</ins><br>}</code></pre> |

请注意：不要使用 Markdown 的三个反引号 (```) 在表格单元格内。
