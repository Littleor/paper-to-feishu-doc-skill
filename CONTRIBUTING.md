# 贡献指南

欢迎改进论文解读、阅读版式和飞书文档编辑流程。

## 修改位置

- 技能范围与关键约束：`skills/lark-paper-blog/SKILL.md`
- 阅读与写作偏好：`skills/lark-paper-blog/references/writing-style.md`
- 取证、插图、写入和核验流程：`skills/lark-paper-blog/references/workflow.md`
- 安装、依赖和使用入口：`README.md`

将操作细节放在对应参考文件中，避免在多个文件复制同一套规则。保持 `lark-paper-blog` 调用名稳定，不使用个人绝对路径、账号或真实私有文档作为示例。

## 验证修改

1. 通过 Skills CLI 的只读发现功能确认能找到技能：

   ```bash
   npx skills add . --list
   ```

2. 检查相对链接是否可解析，以及示例参数是否与已安装 `lark-cli` 的帮助和参考文档一致。若有 `skill-creator` 的验证器，可对 `skills/lark-paper-blog` 运行其 `quick_validate.py`。
3. 对影响文档行为的修改，使用自己有权修改的测试文档。分别检查空白文档与已有正文场景，确认书签、原图、评论、公式和实验数字得到保留。
4. 在 PR 中说明修改解决的问题、验证方式与未验证范围。安装发现和静态验证不能替代真实飞书写入验证。

不要为演示而在他人的文档执行写入，也不要提交论文下载、API 响应、凭证或真实私有文档链接。

## 提交与 PR

提交信息使用英文与 Conventional Commits，例如：

```text
feat: support a new paper source workflow
fix: preserve image blocks during section updates
docs: clarify Feishu CLI dependencies
```

只提交与修改有关的文件。欢迎通过 Issue 提供脱敏后的复现步骤，通过 PR 提交可独立检查的改动。
