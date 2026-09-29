# 忽略与检索适配

文档核验：2026-09-22。只读正在处理的章节；此表不是所有版本的兼容承诺。先看本地版本、有效配置和实际搜索入口；发现新版本差异时只核验受影响机制，不反复检查所有厂商。无法证实时保留原设置并报告未验证。

## 选择原则

Git 忽略、搜索/索引过滤、原生读写工具权限、操作系统隔离是不同层。工具读取已忽略路径、终端命令和外部 MCP 是否受限必须分别判断。

处理范围以用户明确指定及项目现有配置为准；“本机安装了某 Agent”不代表应给当前仓库创建它的配置。只读获取版本，不触发交互式安装或联网升级。不得自动修改用户全局配置、组织策略或审批方式。

## Git

`.gitignore` 适合共享的非跟踪文件规则；它不会停止跟踪已有文件。停止跟踪属于独立操作，需要精确路径与单独授权。不要把仍需版本管理、仅不宜默认检索的文件加入 Git 忽略。

保留嵌套 `.gitignore`、否定规则及优先级。被整目录排除后，简单追加一个子文件 `!` 规则可能无法重新包含它。不要未经验证就在文件末尾追加管理块，可能覆盖用户原有例外。

本地私有 Git 排除可使用 Git 实际解析的 exclude 文件；通过 `git rev-parse --git-path info/exclude` 定位，必要时结合 `--git-common-dir` 核验，不硬编码 `.git/`。不同 worktree 可能共享它；只有用户要求本地排除并认可影响范围后才编辑。这是版本控制内部数据禁止写入规则的唯一窄例外，不授权改其他内部文件。

示例管理标记，不包含任何默认忽略模式：

```gitignore
# BEGIN project-hygiene
# 仅放经本项目证实必要的规则。
# END project-hygiene
```

仅在目标没有同等有效规则时新增，已有规则尽可能复用。不按文件名或扩展名一刀切地过滤整个 `Assets/`、`Packages/`、`ProjectSettings/`、`docs/`、`.agents/`、`.claude/` 或 `.codex/`。

## Codex

本次查阅的官方配置参考未确认通用 `.codexignore` 协议；不把这个文件名作为可靠实现。已有同名文件先确认是否被当前客户端、插件或脚本实际读取，不因未见官方支持而删除它。

优先维护正确的 `.gitignore`，但不要声称它约束所有 Codex 读取。实际使用 ripgrep 的入口遵循其过滤规则；若已有 `.ignore`/`.rgignore`，先检查使用者与优先级。它们是搜索工具规则，不是 Codex 的通用权限文件，也可能影响其他工具。必要时采用目标命令的 `-g` 或显式搜索目录，不新增自制“通用 Agent 忽略协议”。

没有已验证的原生过滤时，先确认真正生效的 `AGENTS.md` / `AGENTS.override.md` 及作用范围，再补一个短的默认检索约定；不同时堆叠重复规则，不改全局行为。原指令已覆盖时不再增加。

参考措辞，替换为确认过的项目路径：

> 检索先限相关源码目录，再查符号并读取必要片段；默认跳过已确认的缓存、构建输出和历史导出目录。任务确实需要被过滤内容时，只定向读取相关文件，不全仓关闭忽略；已有权限限制仍须遵守。

验收要区分：Git 匹配已测；搜索命令过滤已测；项目指令已加载；Codex 原生入口尚未验证。不要把前两项称为“Codex 已无法读取”。

## Claude Code

本次采用官方的项目设置与权限机制，不假设存在通用 `.claudeignore`。纯降噪优先使用正确的 Git 规则、限定检索范围，以及实际生效的 `CLAUDE.md` 中的短约定；不把“不默认读取”升级成“不许读取”。已有相同约定不重复增加。

只有用户明确要求强制限制原生文件工具时，才增量合并 `.claude/settings.json` 或经授权的本地设置中的 `permissions.deny`。示例仅示意语法，不是默认模板：

```json
{
  "permissions": {
    "deny": ["Read(/confirmed-private-output/**)"]
  }
}
```

写入前核验安装版本及路径锚点，特别是从仓库子目录启动的会话。当前文档中的 `Read` deny 也可能阻止相同路径的 Edit/Write；不能把所有缓存与输出目录无差别加进去。它并非覆盖任意子进程的系统级文件隔离。不要通过脚本或其他工具绕过已有 deny。

设置文件须为有效 JSON，不能写注释管理块。保留其他键和数组顺序，按实际规则语义最小合并；通过 `/status`、权限界面及无敏感内容的测试核对加载与效果。仅语法解析通过不足以证明已生效。

## Cursor

当前官方文档明确支持项目根 `.cursorignore`，采用 Git 风格模式，影响其 Agent、引用等入口；终端和 MCP 不因此获得同样的阻断。Cursor 也有 `.gitignore` 与默认排除规则，先检查再补充。

本次查阅的当前页面没有确认 `.cursorindexingignore`；仅当安装版本的文档或行为测试确认后，才维护这种索引专用文件。保留既有文件，不因当前参考页未列出就删除或迁移。

必须通过当前客户端的实际索引/发现/引用入口检验。`git check-ignore` 不会自动读取 `.cursorignore`；Git 语法测试不能代替 Cursor 生效验证。

## Gemini CLI

使用项目根 `.geminiignore`。它按近似 Git 忽略语义过滤支持该功能的工具及 `@` 文件入口，不影响 Git 自身，也不构成所有工具和子进程的访问隔离。

当前文档要求重启 Gemini CLI 会话后应用修改；未完成重启/测试则标记待生效。不要与 Gemini Code Assist 的 `.aiexclude` 混用。不要照抄文档中广泛过滤 Markdown 或 packages 的演示规则。

## Windsurf / Devin Desktop

当前 Windsurf 文档入口重定向到 Devin Desktop 文档：列出 `.devinignore`、兼容的 `.codeiumignore`，并说明 Agent 文件访问也尊重 `.windsurfignore`。因此先识别实际产品、版本和入口；保留已有受支持文件，不自动迁移，不一次生成三套。

区分索引过滤与文件访问限制，依据本地版本测试。不得把该文档描述直接推广为对所有终端/MCP 的系统级隔离。

## GitHub Copilot 与其他工具

GitHub 官方内容排除具有套餐、权限和入口限制；本次文档仍注明编辑器中的 Edit/Agent 模式不支持该内容排除。不要创建想象中的 `.copilotignore`，不要把 IDE 的 `files.exclude` 或 `search.exclude` 当作所有 Copilot 工具都遵守的安全策略。

其他 Agent 先查当前官方机制与本地版本。支持则适配，不支持或权限不足则给出限定检索约定和准确限制；不安装额外插件、MCP 或服务来凑覆盖率。

## 游戏项目的保留检查

仅在实际项目匹配时应用；优先复用现有领域技能，而不是照模板加入路径。

Unity：默认保留资源及配套 `.meta`、`Packages` 清单/锁文件和 `ProjectSettings`。删除资源不能只看 C# 文本引用，还要检查 GUID、序列化场景/Prefab、动态加载、Addressables 与构建入口。`Library` 等缓存应先判断重建成本，通常过滤即可。

其他引擎：先识别项目清单、资源依赖/导出配置与官方缓存目录。Godot、Defold 和 FairyGUI 的项目源文件、资源描述及生成后仍被运行/发布引用的产物，都不能仅凭“自动生成”认定可删。

## 验证步骤与命令

以下是供 Agent 选用的命令，不是自动执行脚本。先确认工作目录和路径，读取输出时限定范围并做聚合；脚本处理文件名用 NUL 分隔，不能按行拼接到 shell。不要对敏感文件做读取探针。

```sh
# 工作树边界及用户改动；捕获并汇总，不直接倾倒大输出。
git rev-parse --show-toplevel
git status --porcelain=v1 -z --untracked-files=normal

# Git 路径清单：限定实际相关目录。被忽略目录不在 --others 清单内。
git ls-files -z --cached --others --exclude-standard -- "path/to/scope"

# Git 规则来源与匹配；先替换示例路径。
git check-ignore -v -- "confirmed-output/example.tmp"

# 诊断已跟踪路径是否被规则命中；不改变跟踪状态。
git check-ignore -v --no-index -- "src/important-file"

# 检查已有管理块或规则产生的空白错误。
git diff --check
```

对 `git check-ignore`：退出码 0 表示存在忽略匹配，1 通常表示没有文件被忽略，128 是错误；使用 `-v` 时还须检查是否命中了以 `!` 开头的否定模式，不能只看有没有输出。用 `--no-index` 检查保护样本，避免已跟踪状态掩盖错误规则。

使用 ripgrep 时可按具体项目选择：

```sh
# 默认尊重适用的忽略规则；并非所有隐藏文件都被枚举。
rg --files "path/to/source"

# 有界读取；命中太多时缩小目录/模式，不反复打印同一批内容。
rg -n --max-count 20 --max-columns 200 --max-columns-preview \
  -g '!confirmed-generated/**' -- "target_symbol" "path/to/source"
```

`--max-count` 是每个文件上限，不是总输出上限；仍需工具层限制整体输出。`-g` 模式相对于执行目录，须测试实际路径。Git 只诊断 Git 的规则，不替其他 Agent 原生文件做匹配。

每个被修改的过滤入口至少验证一个应排除样本、一个应保留源码样本，以及适用的嵌套路径/否定例外。另选依赖锁文件、Agent 指令、资源元数据等保护样本。必要时在系统临时目录的隔离仓库验证纯模式语义，不改真实项目以“试一试”。新探针不得包含秘密，用完只清理本轮生成的探针。

同一搜索范围和入口有前后数据时，报告候选数量/工具输出变化；没有前值就不编造比例。测试环境、引擎编辑器或 Agent 无法运行时，清楚列出未验证项。重复合并相同规则应无新增 diff；保留未解决冲突，不靠覆盖配置达成幂等。

## 官方核验来源

这些是参考来源，不是每次运行的必读清单。

- Git 忽略：`https://git-scm.com/docs/gitignore`
- Git 忽略诊断：`https://git-scm.com/docs/git-check-ignore`
- Git 文件清单：`https://git-scm.com/docs/git-ls-files`
- Codex 配置：`https://developers.openai.com/codex/config-reference/`
- Codex 项目指令：`https://developers.openai.com/codex/guides/agents-md/`
- Claude Code 设置：`https://code.claude.com/docs/en/settings`
- Claude Code 权限：`https://code.claude.com/docs/en/permissions`
- Cursor：`https://cursor.com/docs/reference/ignore-file`
- Gemini CLI：`https://geminicli.com/docs/cli/gemini-ignore/`
- Windsurf 文档入口：`https://docs.windsurf.com/context-awareness/windsurf-ignore`
- 当前重定向：`https://docs.devin.ai/desktop/context-awareness/windsurf-ignore`
- GitHub Copilot：`https://docs.github.com/en/copilot/concepts/context/content-exclusion`
- ripgrep 作者指南：`https://github.com/BurntSushi/ripgrep/blob/master/GUIDE.md`
- Unity 元数据：`https://docs.unity3d.com/Manual/AssetMetadata.html`
