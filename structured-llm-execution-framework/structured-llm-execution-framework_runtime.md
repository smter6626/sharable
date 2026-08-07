# Structured LLM Execution Framework — Copyright & Provenance Runtime

## Current Status

```text
ACTIVE — copyright/provenance formalization task initialized; dedicated repository migration is the current active step
```

当前 canonical copy 仍位于：

- `smter6626/sharable/structured-llm-execution-framework/structured-llm-execution-framework-zh.md`

目标独立仓库：

- `smter6626/structured-llm-execution-framework`

截至本 Runtime 创建前，GitHub 仓库搜索未发现该目标仓库已经存在，因此迁移尚未开始。

---

# done

- 2026-08：中文文章已在 `smter6626/sharable` 中持续公开迭代，并形成 Case A（长期科研工程）与 Case B（AudioShifter 软件发布工程）两个实际案例。
- 2026-08：已明确本任务的核心目的不是阻止他人借鉴抽象思想，而是强化文章及框架论述的 authorship / priority / provenance，使其可以作为直博、实习和其他职业/学术材料中的可验证成果。
- 2026-08：已决定保留 `sharable` 的既有 Git commit 历史，不通过 history rewrite、force push 或删除早期 commits 的方式“清理”来源证据。
- 2026-08：已决定将成熟后的框架文章从杂项 `sharable` 仓库拆分为独立 canonical repository；后续英文版、正式版本、GitHub Release、DOI 和 citation metadata 原则上都应落在独立仓库。
- 2026-08：当前默认 rights strategy 为 `Copyright © 2026 Yeming Dai. All rights reserved.`；在 owner 明确改变许可前，不主动授予 CC 或其他开放内容许可。
- 2026-08：已确定主要正式化路线为：独立仓库迁移 → 中文 v1.0 冻结与版权/引用元数据 → 英文版 → 中英文联合审校 → 固定 PDF → Git tag / GitHub Release → Zenodo DOI → DOI 回填与双向链接 → 可选美国版权登记。
- 2026-08-07：创建长期合同 `structured-llm-execution-framework_static.md`，commit `8d5e706c6f258cc10813274e7f765008e1fb5756`。Static 明确了 provenance、安全边界、版权目标、正式版本、DOI、英文版、Git 安全和可选美国版权登记的长期规则。
- 2026-08-07：查询 GitHub，未发现 `smter6626/structured-llm-execution-framework` 已存在；因此目标 repo 名称当前未发现冲突，但真正创建前仍需执行一次即时确认。

---

# active step

## Step 1 — 拆分并建立独立 canonical repository

### 目标

把当前已经成熟的框架文章从杂项仓库 `smter6626/sharable` 拆分到独立仓库：

```text
smter6626/structured-llm-execution-framework
```

本步骤只完成 **repository identity + provenance migration**。不要提前执行英文翻译、DOI 发布、正式 v1.0 Release 或美国版权登记。

### 当前事实来源

执行前必须重新读取：

1. `structured-llm-execution-framework_static.md`
2. `structured-llm-execution-framework_runtime.md`
3. `structured-llm-execution-framework-zh.md`
4. `sharable` 中与该文章相关的 Git history / commits

不得依赖聊天记忆替代仓库事实。

### 迁移原则

1. **旧历史不动**
   - 不 rewrite `sharable` history；
   - 不 force push；
   - 不删除已有文章 commits；
   - 不伪造“新仓库从一开始就是原始开发仓库”的叙事。

2. **新仓库是 continuation，不是假装原始起点**
   - README 必须明确说明该项目最初在 `smter6626/sharable` 中形成；
   - 提供旧文章路径或历史入口；
   - 说明因项目成熟、需要独立版本/引用/DOI 管理而拆分；
   - 新仓库从迁移节点开始成为 canonical repository。

3. **第一轮迁移只复制当前可验证内容**
   初始仓库至少应包含：

   ```text
   README.md
   structured-llm-execution-framework-zh.md
   ```

   是否在本步骤同步复制 Static / Runtime，由执行时根据仓库管理便利性决定；如果复制，必须明确它们是 internal execution-control documents，不是文章主体或 DOI deposit 的默认 public deliverable。

4. **不要在迁移时改变文章正文**
   - 迁移 commit 的首要目标是身份拆分与 provenance；
   - 正文内容应与 `sharable` 当前 canonical copy 一致；
   - 如果发现正文确实需要修改，应先记录差异并在迁移完成后作为下一状态单独处理，不要把内容编辑混入迁移证据。

5. **不要现在给仓库添加开放源代码许可证**
   - 当前文章默认 `All rights reserved`；
   - 不因为 GitHub 创建仓库界面提供 license template 就选择 MIT、Apache、GPL、CC 等许可；
   - copyright / rights metadata 将在中文 v1.0 冻结步骤正式加入。

### 推荐执行方式

优先使用可审计的 CLI / GitHub API：

```bash
gh repo create smter6626/structured-llm-execution-framework --public
```

具体命令可根据 GitHub CLI 当前行为调整，但必须满足：

- owner = `smter6626`；
- repo name = `structured-llm-execution-framework`；
- visibility = public；
- default branch = `main`；
- 不自动套用不需要的 license；
- 不导入或重写 `sharable` 的整个杂项历史。

推荐采用“新仓库从当前文章快照开始 + README 明确链接原始历史”的 provenance 模式，而不是使用 filter-repo 把 `sharable` 的全部历史加工成一条看似原生的新仓库历史。原因是：原始 commits 保留在原仓库本身就是更直接的历史证据，迁移节点则形成第二段清晰的 provenance。

### 新仓库 README 最低信息

README 至少应包含：

- 项目正式名称；
- 一句话框架摘要；
- 中文正文入口；
- `Author: Yeming Dai`；
- 明确的 provenance note，例如：

  ```text
  This project was originally developed and iterated in the author's
  `smter6626/sharable` repository before being split into this dedicated
  repository in August 2026. Earlier development history remains preserved
  in the original repository.
  ```

- 原 `sharable` 中文文章或历史入口链接；
- 当前状态仍为 pre-v1.0 / formalization in progress，不虚构 DOI 或版权登记状态。

### 验收条件

Step 1 只有在以下条件全部满足后才能标记 COMPLETE：

- [ ] `smter6626/structured-llm-execution-framework` 已公开创建；
- [ ] default branch 为 `main`；
- [ ] 中文文章已迁移，内容与迁移前 `sharable` 当前版本一致，或差异已逐项解释；
- [ ] README 明确作者与 provenance；
- [ ] README 能从新仓库定位回旧 `sharable` 历史；
- [ ] `sharable` 中的早期 commits 仍存在且未重写；
- [ ] 未错误添加开放许可证；
- [ ] 未提前声称 DOI、v1.0、Copyright Office registration 已完成；
- [ ] 新仓库可以被视为后续正式版本的 canonical repository；
- [ ] 迁移 commit SHA 被记录到本 Runtime 的 `done`；
- [ ] 完成后将 Step 2 设为唯一 active step。

### 失败 / 阻塞处理

- 如果目标 repo 名称被占用或已存在：停止，不自行改名；报告实际状态，由 owner 决定复用还是更名。
- 如果迁移时发现 `sharable` 当前正文与预期不一致：停止正文编辑，仅记录差异。
- 如果 GitHub 权限不足：记录 blocker，不绕过账户或权限控制。
- 如果 Git 工作区或远端状态不清楚：先检查，禁止 force push / reset / history rewrite。

---

# next steps

以下仅记录方向。除 Step 1 完成后被提升为 active 的步骤外，不提前写成详细执行手册。

## Step 2 — 冻结中文 v1.0 候选与作者/版权元数据

方向：对迁移后的中文正文做最后事实、术语和结构审校；加入作者、版本、canonical repository、copyright notice、rights statement，并建立 `CITATION.cff`。此时仍不正式打 v1.0 tag。

## Step 3 — 制作英文正式版

方向：以冻结后的中文候选版为唯一事实源进行英文翻译与学术英语润色；保持框架定义、Case A / Case B 和事实数字一致。

## Step 4 — 中英文联合一致性验收

方向：交叉检查标题、作者、版本、术语、链接、案例事实、数字、rights metadata 和 citation metadata，形成 v1.0 release candidate。

## Step 5 — 生成固定版 PDF 与发布资产

方向：生成中英文固定 PDF，验证字体、链接、分页、作者信息、版本和版权信息；生成 SHA-256 或等价资产校验记录。

## Step 6 — Git v1.0 tag 与 GitHub Release

方向：从干净且已验收的 commit 创建不可移动的正式 `v1.0` tag 和 GitHub Release；正式资产不得在发布后静默替换。

## Step 7 — Zenodo DOI

方向：核对届时 Zenodo 最新规则，建立 deposit，优先 reserve DOI → 回填 Markdown/PDF/CITATION → 最终 Publish；确保 creator、版本、发布日期、rights、repository 与文件一致。

## Step 8 — DOI 回写与 citation closure

方向：把正式 DOI 回写 GitHub README、中文/英文文档、PDF、`CITATION.cff` 等入口，并验证从 DOI、Release 和 repository 任何一端都能定位到同一正式成果。

## Step 9 — 可选：美国版权登记

方向：仅在 owner 确认要做时启动；届时重新核对 U.S. Copyright Office 最新规则、发表状态、deposit copy、AI-assisted authorship disclosure 和申请类型。该步骤不阻塞主要任务完成。

---

# stop condition

主要任务可判定为：

```text
PASS — v1.0 authorship/provenance package publicly frozen, citable, DOI-backed, and cross-linked
```

前提是 Static 的主要验收标准全部通过。

美国版权登记若未执行，应标记为 `OPTIONAL / NOT STARTED`，不得因此把主要任务判定为 PARTIAL。
