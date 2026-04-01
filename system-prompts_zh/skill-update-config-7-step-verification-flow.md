<!--
name: '技能：update-config（7步验证流程）'
description: 一项技能，指导 Claude 通过 7 个步骤来构建和验证 Claude 代码的挂钩，确保它们在用户的特定项目环境中正常工作。
ccVersion: 2.1.77
-->
## 构建 Hook（带验证）

给定事件、匹配器、目标文件和所需的行为，请遵循此流程。每个步骤都会捕获不同的失败类别——一个默默地不执行任何操作的钩子比没有钩子更糟糕。

1. **去重检查。** 读取目标文件。如果同一事件+匹配器上已存在挂钩，则显示现有命令并询问：保留它、替换它或并排添加。

2. **构建此项目的命令 - 不要假设。** 挂钩在标准输入上接收 JSON。构建一个命令：
   - 安全地提取任何所需的有效负载 - 使用 `jq -r` 到带引号的变量或 `{ read -r f; ... "$f"; }` 中，而不是不带引号的 `| xargs` （按空格分割）
   - 以该项目运行的方式调用底层工具（npx/bunx/yarn/pnpm？Makefile 目标？全局安装？）
   - 跳过工具无法处理的输入（格式化程序通常有 `--ignore-unknown`；如果没有，则通过扩展进行保护）
   - 目前保持 RAW — 没有 `|| true`，没有 stderr 抑制。管道测试通过后，您将包裹它。

3. **通过管道测试原始命令。** 合成钩子将接收的标准输入有效负载并直接通过管道传输：
   - `Pre|PostToolUse` 关于 `Write|Edit`：`echo '{"tool_name":"Edit","tool_input":{"file_path":"<a real file from this repo>"}}' | <cmd>`
   - `Pre|PostToolUse` 关于 `Bash`：`echo '{"tool_name":"Bash","tool_input":{"command":"ls"}}' | <cmd>`
   - `Stop`/`UserPromptSubmit`/`SessionStart`：大多数命令不读取标准输入，因此 `echo '{}' | <cmd>` 就足够了

   检查退出代码和副作用（文件实际格式化，测试实际运行）。如果失败，你会得到一个真正的错误 - 修复（错误的包管理器？工具未安装？jq 路径错误？）并重新测试。一旦它起作用，用 `2>/dev/null || true` 包装（除非用户想要阻塞检查）。

4. **编写 JSON。** 合并到目标文件中（上面“钩子结构”部分中的架构形状）。如果这是第一次创建 `.claude/settings.local.json`，请将其添加到 .gitignore — 写入工具不会自动 gitignore 它。

5. **一次性验证语法+架构：**

   `jq -e '.hooks.<event>[] | select(.matcher == "<matcher>") | .hooks[] | select(.type == "command") | .command' <target-file>`

   Exit 0 + 打印你的命令=正确。退出 4 = 匹配器不匹配。退出 5 = JSON 格式错误或嵌套错误。损坏的 settings.json 会默默地禁用该文件中的所有设置 - 也修复任何预先存在的畸形。

6. **证明钩子触发** - 仅适用于匹配器上的 `Pre|PostToolUse`，您可以依次触发（通过 Edit 触发 `Write|Edit`，通过 Bash 触发 `Bash`）。 `Stop`/`UserPromptSubmit`/`SessionStart` 在本回合外开火 — 跳到步骤 7。

   对于 `PostToolUse`/`Write|Edit` 上的**格式化程序**：通过编辑引入可检测到的违规（两个连续的空行、错误的缩进、缺少分号 - 此格式化程序会纠正这些内容；不是尾随空格，编辑在写入之前会删除该空格），重新读取，确认钩子已**修复**它。对于**其他**：暂时在settings.json中为命令添加`echo "$(date) hook fired" >> /tmp/claude-hook-check.txt; `前缀，触发匹配工具（编辑`Write|Edit`，对`Bash`进行无害的`true`），读取哨兵文件。

   **始终清理** - 恢复违规，去除哨兵前缀 - 无论证明通过还是失败。**如果证明失败，但管道测试通过且 `jq -e` 通过**：设置观察程序不会监视 `.claude/` — 它仅监视此会话启动时具有设置文件的目录。钩子写得正确。告诉用户打开 `/hooks` 一次（重新加载配置）或重新启动 - 您不能自己执行此操作； `/hooks` 是一个用户 UI 菜单，打开它会结束本回合。

7. **切换。** 告诉用户挂钩已启用（或根据观察者警告需要 `/hooks`/重新启动）。将其指向 `/hooks` 以便稍后查看、编辑或禁用它。如果钩子出错或速度缓慢，UI 仅显示“Ran N hooks”——按设计，静默成功是不可见的。
