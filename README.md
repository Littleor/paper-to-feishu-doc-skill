# Paper to Feishu Doc Skill

将学术论文整理成图文并茂的中文飞书技术博客。基于官方论文核对方法与实验，插入论文原图，并直接更新指定的飞书 Docx / Wiki 文档。

An Agent Skill for turning research papers into illustrated Chinese technical articles in Feishu, with source verification and targeted document edits.

**技能调用名：`lark-paper-blog`** · **许可证：[MIT](LICENSE)**

## 能做什么

- 从论文书签、arXiv 链接或其他官方论文来源开始，整理动机、方法、训练、实验、消融和局限。
- 在书签后加入“动机 / 方法 / 核心贡献”三点高亮概览。
- 用分点说明、编号流程和实验表格降低阅读负担，让公式与实现细节紧跟对应模块。
- 从官方 HTML 或源文件中获取关键原图，目视检查后上传为飞书内部资源，并配中文图注。
- 按 Block 局部更新已有文档，保留标题、书签、评论、图片及其他有价值资源。
- 重新读取文档，核验结构、图片、公式、实验数字与写入结果。

这是供 AI Agent 使用的指令型 Skill，需要具备终端、联网检索和图片查看能力的 Agent 执行。它本身不运行后台服务，也不提供独立的论文转换 CLI。

## 安装

### 1. 准备飞书依赖

安装并配置 [飞书官方 CLI](https://github.com/larksuite/cli)，并安装配套 Agent Skills：

```bash
npx @larksuite/cli@latest install
npx skills add larksuite/cli -g
```

按官方引导完成应用配置、用户登录和文档权限授权。已有可用配置时不必重建。至少需要 `lark-doc`；它涉及认证时会使用 `lark-shared`。

本 Skill 使用 `--as user` 访问用户指定的目标文档。目标文档需要可读写，论文正文需要可访问。

本项目的文档读写流程已在 `lark-cli 1.0.93` 上使用；这不是最低版本声明。CLI 更新后，以对应版本的 Skill 参考及 `--help` 为准。

### 2. 安装本 Skill

仓库内容发布后，可以通过 [Skills CLI](https://github.com/vercel-labs/skills) 安装：

```bash
npx skills add Littleor/paper-to-feishu-doc-skill --skill lark-paper-blog -g
```

安装器会让你选择目标 Agent。移除 `-g` 可按项目安装。

也可以从本地 checkout 安装，适合尚未发布或正在维护的版本：

```bash
cd paper-to-feishu-doc-skill
npx skills add . --skill lark-paper-blog -g
```

如果你的 Agent 不使用 Skills CLI，将 [`skills/lark-paper-blog`](skills/lark-paper-blog) 整个目录放入该 Agent 支持的技能目录。安装后按宿主要求刷新或重新加载技能。

### 3. 检查依赖是否可读取

```bash
lark-cli --version
lark-cli docs +fetch --help
lark-cli docs +update --help
lark-cli skills read lark-doc
```

若未安装独立的 `lark-doc` 文件，Agent 也可通过 CLI 内置的 `skills read` 读取对应操作说明，例如：

```bash
lark-cli skills read lark-doc references/lark-doc-fetch.md
```

本仓库不捆绑飞书官方技能副本，避免它们与用户安装的 CLI 版本脱节。

## 使用示例

在支持技能调用的 Agent 中输入：

```text
用 $lark-paper-blog 整理这个飞书文档里书签对应的论文：
<你的飞书 Wiki 或 Docx 链接>
```

如果论文与目标文档分别提供：

```text
用 $lark-paper-blog 解读这篇论文：<论文官方链接>
写入：<目标飞书文档链接>
保留已有标题和资源，重点解释核心机制与实验结论。
```

对已有解读进行可读性优化：

```text
按 $lark-paper-blog 的阅读偏好调整这篇文档：<目标文档链接>
在书签下加三点概览，把说明分点、流程编号、实验结果改成表格。
```

只有论文链接时，Agent 还需要明确交付的目标文档；它不会自行选择其他人的文档。给出具体论文版本、语言、篇幅或排版要求时，以这次请求为准。

## 默认成品结构

```text
已有标题
论文书签
三点高亮概览：动机 / 方法 / 核心贡献
动机
方法总览与核心模块
  原图、公式解释和实现补充就近放置
训练 / 数据
实验与消融
结论与局限
来源与版本信息
```

结构会按论文实际贡献调整。通常选择 2～4 张有解释价值的论文原图，普通方法论文通常约 3000～6000 个中文字符；这些都是默认偏好，不是硬性配额。

## 仓库结构与维护

```text
skills/lark-paper-blog/
├── SKILL.md                    # 入口、适用范围与关键约束
├── agents/openai.yaml          # Agent 界面展示信息
└── references/
    ├── writing-style.md        # 中文写作与阅读版式
    └── workflow.md             # 论文取证、局部写入、插图与验证
```

维护时修改仓库内的 Skill，并在本地安装后检查实际效果。安装副本与仓库可能不是同一份文件：若使用复制安装，修改仓库后需重新安装；若使用符号链接，可直接跟踪仓库中的修改。提交前查看差异，只提交本次修改。

贡献流程见 [CONTRIBUTING.md](CONTRIBUTING.md)。

## 数据与授权边界

- 仅修改用户指定的目标文档；无法访问论文或缺少写权限时停止写入。
- 不用整篇覆盖代替局部编辑，不因调整版式而删除既有资源或评论。
- 临时论文、图片、草稿与 API 响应放在任务的 `work/` 中，不提交到本仓库。
- 飞书凭证与登录状态由官方 CLI 管理，不写入 Skill、Git 仓库或生成文档。
- 本项目的 MIT 许可证适用于仓库原创内容；论文正文、原图和外部依赖仍遵循各自的许可与使用条件。

## License

[MIT](LICENSE) © 2026 Littleor
