# Structured LLM Execution Framework — Copyright & Provenance Static Contract

## 1. 任务定位

本合同管理 **Structured Constraint-Driven LLM Execution Framework** 的正式化、作者身份固化、版本冻结、引用与长期 provenance（来源追踪）工作。

当前中文正文位于：

- `structured-llm-execution-framework/structured-llm-execution-framework-zh.md`

本任务的目标不是阻止他人学习、借鉴或独立使用 Static / Runtime / History 等抽象思想；目标是建立一条足够清晰、公开、可验证的证据链，使外部读者能够确认：该框架的具体文章、结构化论述、案例组织和正式版本由作者 **Yeming Dai** 创作、整理、公开并持续维护。

## 2. 长期目标

最终应形成一个可以用于简历、直博申请、实习申请、学术或工程引用的独立成果，至少具备：

1. 独立、语义清晰的 canonical GitHub repository；
2. 保留且可追溯到 `smter6626/sharable` 中的早期公开开发历史；
3. 冻结的中文正式版；
4. 与中文版语义一致的英文正式版；
5. 明确的作者、版本、首次公开年份、canonical repository、copyright notice 与引用元数据；
6. 可引用的固定版本和 GitHub Release/tag；
7. Zenodo 或同等级持久化存档与 DOI；
8. DOI、GitHub、PDF/Markdown 与 citation metadata 之间一致的双向引用；
9. 如项目所有者认为值得，再进行美国版权登记，作为额外法律证据层。

## 3. 版权与作者身份边界

### 3.1 要保护和固化的对象

重点固化：

- 文章的具体文字表达；
- 章节组织与结构；
- Static / Runtime / History 在本文中的系统化定义和组合论述；
- 执行循环、风险分层和案例分析的具体表达；
- Case A / Case B 的选材、组织与解释；
- 中英文正式版本；
- 人类对 LLM 辅助内容所做的选择、编辑、事实核验、重组和最终表达。

### 3.2 不试图垄断的对象

本任务不以版权手段阻止：

- 他人理解或讨论文中的思想；
- 他人独立采用 Static / Runtime / History 类似机制；
- 他人用自己的语言描述相似的软件工程或 LLM 执行方法；
- 合理引用、评论、批评或学术讨论。

版权与 DOI 的主要作用是强化 **authorship / priority / provenance**，而不是把抽象思想变成排他性技术权利。

## 4. Canonical Repository 原则

文章应从当前杂项仓库 `smter6626/sharable` 拆分到独立仓库。默认目标仓库名称：

- `smter6626/structured-llm-execution-framework`

除非项目所有者明确改变命名，否则新仓库应成为后续正式版本、英文版、Release、DOI 和引用元数据的唯一 canonical repository。

### 4.1 迁移必须保留 provenance

不得为了“干净”而重写、删除或伪造 `sharable` 中已有的历史证据。

迁移后必须能够清楚表达：

- 文章最初在 `smter6626/sharable` 中形成并公开迭代；
- 之后因项目成熟而拆分到独立仓库；
- 原仓库保留早期 commits，作为迁移前 provenance；
- 新仓库从明确的迁移节点开始成为 canonical source。

### 4.2 旧链接处理

独立仓库完成并验证前，不删除当前中文正文。

迁移完成后，可以将 `sharable` 中原正文替换为简短的迁移说明和 canonical repository 链接，但只有在：

- 新仓库内容已完整；
- 新旧链接关系已验证；
- 迁移 provenance 已记录；
- 项目所有者明确同意；

之后才能执行。

## 5. 正式版本原则

正式版本应采用显式版本号，例如 `v1.0`。

`v1.0` 至少应包含：

- 中文 Markdown；
- 英文 Markdown；
- 中文固定版 PDF；
- 英文固定版 PDF；
- `CITATION.cff`；
- copyright / rights statement；
- canonical repository 与版本信息；
- DOI（若 DOI 已完成注册或预留）；
- 对应 Git tag / GitHub Release。

正式版本冻结后，不应静默修改同一版本的发布资产。后续实质内容修改应进入新版本，例如 `v1.1` 或 `v2.0`。

## 6. Copyright notice 与许可原则

当前默认策略是：

```text
Copyright © 2026 Yeming Dai.
All rights reserved.
```

在项目所有者明确决定开放许可前，不擅自加入 CC BY、CC BY-SA、CC BY-NC-ND 或其他开放内容许可证。

如果未来改变许可策略，必须由项目所有者明确授权，并记录：

- 新许可名称和版本；
- 生效范围；
- 是否覆盖中英文版本；
- 是否覆盖旧版本；
- 与 Zenodo metadata / GitHub Release / PDF / Markdown 的同步方式。

## 7. 英文版原则

英文版应以冻结后的中文正式稿为事实和结构基础，不应在翻译过程中悄然形成另一套方法论。

允许：

- 学术英语润色；
- 为英语读者调整句法；
- 保持术语一致前提下改善可读性；
- 对无法直译的概念做必要解释。

不得未经确认：

- 改变框架核心定义；
- 改变 Case A / Case B 的事实；
- 增删会改变方法论边界的主张；
- 把中文版未声称的内容扩写成新的正式结论。

中英文正式版在作者、版本、DOI、canonical repository、copyright notice 和核心事实数字上必须一致。

## 8. DOI 与持久化存档原则

DOI 的目的包括：

- 固化一个可引用的正式版本；
- 建立公开、时间可验证的作者与版本 metadata；
- 让简历、申请材料、论文、博客和其他项目能够稳定引用；
- 与 Git history 共同形成 provenance 证据链。

默认路线为 Zenodo，但在正式执行前应核对当时最新的 Zenodo DOI、版本、许可和 GitHub 集成规则。

不得在正文仍处于明显草稿状态时提前 Publish 最终 DOI record。

可在正式发布前使用 DOI reservation，以便将 DOI 回填到最终 PDF、Markdown 和 citation metadata，再正式发布。

## 9. 美国版权登记原则

美国版权登记是 **可选的增强层**，不是 GitHub / DOI 正式发布的前置条件。

如执行：

- 必须使用申请时的最新 U.S. Copyright Office 规则；
- 必须如实处理 LLM / generative AI 辅助创作情况；
- 不得把纯 AI 生成且缺乏人类原创性的内容错误声明为完全由人类独立创作；
- 应以文章中真实存在的人类选择、编辑、组织、事实核验、重写与最终表达为版权主张基础；
- 申请前应再次核对作品发表状态、版本、deposit copy、AI disclosure 和申请类型。

本仓库文档不是法律意见；如果出现真实侵权争议或高价值法律风险，应考虑咨询合格律师。

## 10. Git 与证据安全

整个任务必须保护现有 Git provenance。

未经项目所有者明确授权，不得：

- force push；
- rebase / rewrite 已公开历史；
- 删除或移动作为早期 provenance 的已有 commits；
- 删除 `sharable` 中的早期文章历史；
- 删除正式 Release 资产后无痕替换；
- 移动已发布的正式 tag；
- 伪造作者时间、提交时间、首次公开时间或 DOI metadata。

每个 meaningful state transition 应在 Runtime 中记录对应 commit、tag、Release、DOI record 或其他可定位证据。

## 11. 验收标准

本任务达到主要完成状态，至少需要满足：

- [ ] 独立 canonical repository 已建立并公开可访问；
- [ ] 新仓库明确链接回 `sharable` 的早期 provenance；
- [ ] `sharable` 的既有 Git history 保持可追溯；
- [ ] 中文 v1.0 内容冻结并通过最终审校；
- [ ] Copyright notice 与 rights statement 明确；
- [ ] `CITATION.cff` 正确且可解析；
- [ ] 英文 v1.0 与中文版完成事实和术语一致性审校；
- [ ] 中英文固定 PDF 已生成并核对；
- [ ] 正式 Git tag / GitHub Release 建立；
- [ ] Zenodo DOI 已注册并指向正确作者和版本；
- [ ] DOI 已回填到 GitHub、Markdown、PDF 和 citation metadata；
- [ ] 从 DOI / Release / repository 任一入口都能定位到 canonical work；
- [ ] 正式发布资产与记录未出现作者、版本、日期、许可或链接冲突。

美国版权登记若未执行，不影响上述主要任务判定为完成；应记录为 optional follow-up。

## 12. 非目标

本任务当前不负责：

- 为该方法申请专利；
- 申请商标；
- 阻止他人独立实现类似思想；
- 将本文包装成正式同行评审论文投稿；
- 为了“证明原创”而删除或隐藏真实的 LLM 辅助过程；
- 修改 AudioShifter 或 Case A 科研仓库的既有技术实现。
