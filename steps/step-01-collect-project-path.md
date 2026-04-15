# Step 1: 收集项目路径

## MANDATORY EXECUTION RULES:

- 🛑 NEVER proceed without a valid project path
- ✅ ALWAYS verify the path exists and contains .c/.h files
- 📋 Check for existing profiles to support incremental update

## YOUR TASK:

获取待扫描的项目根目录路径，验证路径有效性，检查是否有已有画像。

## EXECUTION:

### 1. 获取项目路径

如果用户未提供项目路径，提示：

"**📂 特性画像生成 — 项目路径**

请提供需要扫描的 C 代码工程根目录路径：

示例：`D:\workspace\my-project`"

**HALT — 等待用户提供项目路径。**

### 2. 验证路径

- 确认路径存在
- 确认路径下包含 .c 或 .h 文件
- 如路径无效或无 C 文件，提示用户重新输入

### 3. 检查已有画像

检查 `{project-root}/_feature-profiles/` 目录是否存在：

**如果存在**，提示用户：

"**🔄 检测到已有画像**

目录 `{project-root}/_feature-profiles/` 已存在。

选择操作：
1. **全量重新生成**（覆盖已有画像）
2. **增量更新**（仅更新有变更的特性）
3. **取消**"

**HALT — 等待用户选择。**

**如果不存在**，直接进入下一步。

### 4. 设置输出目录

- 输出目录：`{project-root}/_feature-profiles/`
- 创建 `features/` 子目录

### 5. 进入下一步

Read fully and follow: `./steps/step-02-scan-project.md`
