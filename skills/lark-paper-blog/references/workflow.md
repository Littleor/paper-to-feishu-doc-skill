# 执行流程

## 1. 读取目标飞书文档

先看目录，再根据现状选择范围：

```bash
lark-cli docs +fetch --as user --doc '<wiki-or-docx-url>' --scope outline --max-depth 4
lark-cli docs +fetch --as user --doc '<wiki-or-docx-url>' --detail full
```

仅有 `<title>`、论文 `<bookmark>` 和空 `<p>` 时，记录空段落 Block ID。已有正文时，定位相关章节，不要整篇覆盖。任何既有 `<img>`、`<source>`、`<whiteboard>`、引用和评论都视为需保护资源。

## 2. 研究论文

优先级：出版方正式版本（若用户指定）→ arXiv 当前版本 HTML/PDF/TeX source → 项目页。技术事实只依赖第一方来源。

对 arXiv：

1. 从摘要页确认标题、作者、版本号、修订日期和摘要。
2. 阅读 HTML/PDF 的方法、实验、消融、结论和必要附录。
3. 记录核心公式、训练数据、比较基线、统计口径及局限。
4. HTML 渲染不完整时读取 PDF 或 TeX source，不从搜索摘要猜测。

## 3. 选择和获取原图

先从官方 HTML 定位 caption 与资源路径，例如：

```bash
curl -fsSL 'https://arxiv.org/html/<paper-id>v<version>' \
  | rg -n -C 2 'Architecture|Overview|Framework|Training|Data Pyramid'
```

HTML 中的相对路径拼接到同一官方基址。若 HTML 没有高质量资源，从 arXiv TeX source 中定位 `\includegraphics`；SVG/PDF/EPS 在上传前转换为 PNG。所有下载放在当前任务的 `work/` 下，检查：

- 来自论文自身而非 Related Work 中转载的第三方图片；
- 清晰、方向正确、没有裁切或空白页；
- PNG/JPEG/GIF/WebP 且不超过飞书限制；
- 与正文某个章节有直接解释关系。

下载后用本地图像查看工具目视检查。不要只凭文件名上传。

## 4. 写正文

新建文档遵循 `lark-doc` 的创建工作流及其返回的草稿路径；编辑已有文档时，在任务 `work/` 目录生成局部 XML 草稿。保留目标文档的 `<title>` 和 `<bookmark>`，按 [写作风格](writing-style.md) 在论文链接后放三点高亮概览，再展开正文；已有概览则局部更新，避免重复插入。可先用 `--dry-run` 验证 XML：

```bash
lark-cli docs +update --as user --doc '<url>' \
  --command block_replace --block-id '<empty-paragraph-id>' \
  --content @./work/paper-blog.xml --doc-format xml --dry-run
```

空文档可将唯一空段落 `block_replace` 为全部正文。已有内容使用 `str_replace`、`block_replace` 或 `block_insert_after` 做局部更新；除非用户明确要求完全重建且无保留价值，否则不使用 `overwrite`。

## 5. 插入图片到对应章节

`docs +media-insert` 只能追加到文末。需要精确位置时，先获取最新标题 Block ID，再使用：

```bash
lark-cli docs +update --as user --doc '<url>' \
  --command block_insert_after --block-id '<heading-id>' \
  --content '<img path="@./work/figures/architecture.png" width="800" caption="图 1｜……"/>'
```

宽图通常设置 700～800 px，竖图按可读性缩小。使用本地 `path` 上传为飞书资源，不直接热链 arXiv。每次结构性写入后重新 fetch，随后才使用新的定位继续插图。

## 6. 验证

至少验证三类结果：

```bash
# 目录与层级
lark-cli docs +fetch --as user --doc '<url>' --scope outline --max-depth 4

# 关键章节和图片
lark-cli docs +fetch --as user --doc '<url>' --scope section \
  --start-block-id '<heading-id>' --detail full

# 关键结论或数字
lark-cli docs +fetch --as user --doc '<url>' --scope keyword \
  --keyword '核心方法|关键数字|结论' --context-before 1 --context-after 1
```

确认：

- CLI 返回 `ok: true`、`result: success` 且无 warnings；
- 标题、书签、目录层级和正文顺序正确；
- 版式符合 [写作风格](writing-style.md)：论文链接后有简短的动机 / 方法 / 核心贡献高亮列表，无受众和版本日期式开场；多层说明用无序列表，流程用编号，实现补充就近独立放置；
- 每张图存在 `src`/Token、中文 caption 和正确位置；
- 公式、列表、百分比、比较口径与论文一致；
- 没有暴露飞书内部下载 URL、凭证或临时授权参数。
