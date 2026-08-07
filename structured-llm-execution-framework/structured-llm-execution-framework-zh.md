# 结构化约束驱动的 LLM 执行框架

> 通过文档分离与角色专职压缩 LLM 输出的不确定性

---

## 摘要

本文记录了一套在实际项目中迭代验证的 LLM 执行框架。其核心思路不是优化提示词的措辞，而是通过**文档职责分离**和**角色边界固定**，从结构层面压缩 LLM 在执行过程中的自由度——从而使输出的质量和可预测性都得到系统性提升，而不依赖于单次提示的质量。

---

## 一、问题来源

使用单一 LLM 执行多步骤任务时，有两类典型失效模式：

**幻觉扩散（Hallucination Drift）**：LLM 在上下文不足时倾向于自行填补细节。在多步骤任务中，早期步骤的幻觉会被后续步骤继承和放大，最终产出偏离原始需求。

**约束漂移（Constraint Drift）**：LLM 在执行动态任务时会无意识地修改原始限制。当任务描述和执行状态混在同一个上下文中时，"这件事应该怎么做"和"原来要求做什么"之间的边界会逐渐模糊。

早期版本使用单一复合文档同时记录项目目标、步骤计划和执行状态。实践中暴露了两个方向上的失效：

- **静态部分被污染**：LLM 在更新执行状态时会顺手修改原始需求描述，硬性约束悄然改变。
- **动态部分被过度锁死**：所有步骤在项目初期就被详细规划，但后续步骤依赖前置步骤的产出，过早锁定细节导致 LLM 只能基于假设推进，幻觉集中在这里爆发。

这两个问题的根源是同一个：**单一文档无法同时服务于"稳定"和"动态"两种相反的需求**。

---

## 二、设计原则

基于上述失效分析，框架的设计围绕四条原则展开：

**原则一：职责不可混用（Separation of Concerns）**
文档按更新频率和修改权限严格分类。描述"做什么"的文档与描述"现在做到哪"的文档物理隔离，LLM 对两者的修改权限不同。

**原则二：约束前置，细节延迟（Constraints First, Details Late）**
固定的限制在项目启动时一次性写入，后续步骤的实现细节在该步骤成为 active 之前不做详细规划。避免基于假设的提前规划。

**原则三：提示词应当简单（Prompts Should Be Dumb）**
框架的复杂度承载在文档结构上，而不是提示词上。理想状态下，触发 LLM 执行下一步的提示词应当简单到任何人都能写出来，例如："Codex 已完成，汇报如下……请去 repo 检查验证，判断是否推进。"这意味着框架的健壮性不依赖于提示技巧。

Case B 还意外验证了这一点：用户只如实描述了源码遗失、目标使用者不熟悉电脑操作、需要跨平台复刻等现实约束，并未主动要求“建立软件工程流程”；Static / Runtime 结构由 LLM 根据这些约束生成，后续执行自然演化成了一条可审计的软件交付链。

**原则四：工程强度应与任务风险匹配（Proportional Rigor）**
本框架不是要求所有任务使用相同强度的工程控制。文档数量、验收门、独立验证、Git 追踪和 Runtime 更新频率，应当与任务的失败成本、可逆性、持续时间、复现需求和状态复杂度相匹配。对代码实现、科研实验、数据处理、长期仓库维护等任务，如果错误会污染后续步骤、产生高计算成本或需要跨会话复现，应严格使用 Static / Runtime、明确 acceptance gate，并以仓库实际产物作为验证依据。对 PPT、演讲稿、一次性文案、低风险内容整理等任务，可以保留目标和关键约束，但减少状态文档、Git gate 和阶段化验收，以避免框架本身的管理成本超过任务收益。

因此，本框架的目标不是最大化工程化，而是**使用足够的结构压缩有实际风险的不确定性，并最小化达到所需可靠性的工程化成本**。

---

## 三、文档结构

框架使用两个核心文档（长期项目可选第三个），均托管于 Git 仓库。

### 3.1 Static 文档（稳定合同）

**职责**：描述固定不变的内容——项目目标、需求、约束条件、验收标准。

**更新时机**：仅当项目大方向或核心需求发生变更时更新，日常执行过程中不触碰。

**修改权限**：Static 不应在普通执行步骤中被顺手修改。LLM 只能在用户明确授权、任务合同确实变化、并且修改理由被写明时更新 Static。

**典型内容**：

```
## 项目目标
[一句话描述最终产出]

## 硬性约束
- [不可违反的限制，例如技术栈、格式要求、截止时间]

## 验收标准
- [每个可交付物的具体验收条件]

## 背景信息
[LLM 执行时需要的稳定上下文]
```

### 3.2 Runtime 文档（动态指令）

**职责**：追踪执行状态。记录已完成的步骤和结果，详细描述当前唯一 active 的步骤，给出下一步的模糊方向。

**更新时机**：当执行状态发生实质变化时更新，例如验收门通过或失败、blocker 出现或解除、active step 切换、阶段切换、计划被证据推翻。更新可以由用户手动确认，也可以由自动化 API 写回；关键是 Runtime 更新必须成为执行循环的一部分。

**关键设计**：active step 之后的步骤只保留模糊方向，不做详细规划。细节在该步骤成为 active 时才展开——此时前置步骤的产出已知，可以基于真实上下文规划，而不是基于假设。

**状态迁移证据（State Transition Evidence）**：对 meaningful Runtime 更新，建议同时记录触发这次状态变化的 commit、PR、job、artifact 或其他可定位证据，并用一两句话说明它对当前状态意味着什么。这里不是复制完整 Git history；Git history 负责回答“先后发生了什么”，Runtime 负责回答“这些变化使当前状态变成了什么”。

**决策覆写与失效传播（Decision Supersession）**：Runtime 不应只记录新的正确状态，还应在必要时明确声明哪些旧结论已经失效。在长期任务中，一个方案可能在早期证据下通过验收，随后又被新的实验、bug、资源测试或 owner clarification 推翻。此时仅记录新方案是不够的，因为旧 commit、旧 Runtime 或历史备注中仍可能存在 `PASS`、`validated`、`recommended` 等当时正确但现在已失效的结论。

当新证据推翻旧结论时，Runtime 应显式记录：

- 先前结论是什么；
- 什么新证据推翻了它；
- 当前结论是什么；
- 哪些旧记录、方案或假设被 `SUPERSEDED` / `DEPRECATED` / `INVALIDATED`；
- 对应的 commit、PR、job 或 artifact evidence。

这不是为了让 Runtime 变成第二份 Git log，而是建立**语义上的失效关系**。Git history 提供时间顺序，Runtime 提供当前语义。

**典型结构**：
```
## 已完成
- Step 1: [描述 + 结果摘要] ✓
- Step 2: [描述 + 结果摘要] ✓  [2026-06-20 14:23]

## Active Step
### Step 3: [详细描述]
- 具体目标：
- 输入：
- 预期输出：
- 验收条件：
- 已知依赖：

## State Transition
- Evidence: commit `abc1234` / job `12345678` / artifact `[path]`
- State change: [ACTIVE → COMPLETE / BLOCKED → ACTIVE / plan A → plan B]
- Meaning: [这份证据对当前执行状态意味着什么]

## Superseded Decisions（仅在需要时）
- Previous conclusion: [旧结论]
- New evidence: [推翻旧结论的证据]
- Current verdict: [当前有效结论]
- Supersedes: [旧 Runtime 结论 / commit / route / assumption]

## Next Steps（模糊）
- Step 4: [方向描述，细节待 Step 3 完成后确定]
- Step 5: [依赖 Step 4 产出，暂不展开]
```

### 3.3 History 文档（可选，长期项目）

**适用场景**：跨月的持续性项目，例如科研工作、长期产品开发。

**职责**：记录里程碑级别的完成事项、方向调整节点。不记录步骤细节（那是 Runtime 的职责）。

### 3.4 文档优先级与冲突处理

三个文档的职责不同，不存在单一文档永久覆盖其他文档的规则。执行前应按职责判断冲突来源：

- `STATIC_SPEC` 约束当前任务的目标、范围、硬性限制和验收标准；
- `RUN_STATE` 描述当前任务的最新执行状态、blocker、active step，以及已经被新证据明确废止的旧运行时结论；
- `History` 记录跨阶段、跨任务的长期状态变化和阶段收口。

若 `History` 显示某个任务已经阶段收口，而旧的 `RUN_STATE` 仍显示 active，则不得继续按旧 Runtime 执行，应报告冲突并要求用户确认是否新开任务、恢复旧任务，或更新 Runtime。

若 `STATIC_SPEC` 与 `RUN_STATE` 冲突，LLM 不得自行修正其中一个文档。只有在用户授权或明确的执行循环规则允许时，才能更新对应文档。

若当前 Runtime 明确写明某个旧运行时结论已被 `SUPERSEDED`、`DEPRECATED` 或 `INVALIDATED`，后续 LLM 不得因为旧 commit、旧 Runtime 快照或早期备注中出现 `PASS` / `validated` 等字样而重新采信该结论。除非出现新的证据重新打开该决策，否则显式的失效声明应作为当前运行时语义的一部分；它不能覆盖 Static 中仍然有效的硬性合同。

---

## 四、执行循环

```
Step 0  文档一致性预检
        LLM 同时读取 Static、Runtime，长期任务还需读取 History
        若三者在任务状态、目标、active phase 或 stop condition 上冲突
        不得自行选择继续执行
        必须报告冲突并等待用户或更高层文档裁决

Step 1  初始化
        LLM 分析任务需求，与用户对齐后生成 Static 和 Runtime 文档
        直接写入 Git 仓库

Step 2  生成执行提示
        LLM 根据当前 Runtime 中的 Active Step，给出 Codex 执行提示词

Step 3  Agent 执行
        Codex CLI 执行任务，自行校验 + smoke test
        验收后 push 到 Git 仓库

Step 4  LLM 验证
        LLM 直接访问仓库，验证 Step 3 的更新
        ├── 4.1 通过       → 推进，更新 Runtime，标记 Step i 完成，Step i+1 为 Active
        ├── 4.2 通过但有小问题 → 推进，Runtime 记录遗留问题
        ├── 4.3 拒绝验收   → LLM 分析原因，给出修复提示，返回 Step 3
        └── 4.4 需要用户介入 → 中断，等待用户决策后从 Step 2 继续，或验收+终止循环
```

**关于 Step 4 的三个设计决策：**

LLM 的验证基于 Git 仓库的实际产出，而不是 Codex 的汇报。Codex 自评存在盲区，LLM 独立拉取仓库内容做压力测试，这是两个不同视角的交叉验证。

4.3 的修复迭代记录为 Step i-a、Step i-b……保留完整修复轨迹，便于事后分析失效原因。

当 Step 4 导致 meaningful state transition 时，Runtime 应附上能够定位本次变化的 commit / job / artifact evidence；如果本次证据推翻了以前被接受的方案，还应同步写入 supersession 关系，而不是仅仅追加一个新的“当前方案”。

---

## 五、为什么这样有效

**约束空间压缩**：LLM 生成幻觉的根本原因是上下文不足时需要"猜"。Static 文档提供了不可动摇的地基，LLM 在执行时无需猜测"原始需求是什么"——它被写死在那里，且 LLM 不能在普通执行步骤中随意修改。

**局部上下文最大化**：每次执行时，LLM 的注意力集中在当前 Active Step 上，该 Step 的描述是所有步骤中最详细的。后续步骤的模糊性是刻意设计的，避免 LLM 基于假设提前做决策。

**双重验证消除自评盲区**：Codex 执行后自评，LLM 独立验证。两者使用相同的 Git 仓库但处于不同的执行角色，任何 Codex 的自我合理化都会在 LLM 的独立检查中暴露。

**显式失效传播减少历史歧义**：旧结论并不会因为出现了一个更新的 commit 就自动从 LLM 的上下文中消失。通过在 Runtime 中显式声明 supersession，框架把“旧证据曾经成立”和“旧结论现在仍然有效”区分开，降低模型从互相矛盾的历史记录中自行推断知识有效性的风险。

**按风险调节工程强度避免流程税**：严格控制只有在错误代价和状态复杂度足够高时才值得。对低风险任务减少文档和 gate，可以把框架本身的管理成本控制在合理范围；对高风险任务保持完整证据链，则保留可验证性和复现性。

**提示词简单化降低人为引入的变量**：当提示词足够简单，提示词本身就不再是质量瓶颈。框架的稳定性来自文档结构，而不是每次执行时的提示技巧。

---

## 六、Case Study

### Case A：从研究原型到论文、扩展实验与可复现交付

**任务类型**：长周期科研工程任务，覆盖 probabilistic circuits、optimal transport 与 PeTeR 的多轮实验执行、资源调度、结果审计、论文支撑和可复现交付。它不是一个从头到尾只有一份计划的单任务，而是一个研究主线下连续发生、边界不同的多个任务。

**时间跨度与阶段演化**：早期工作从 GCW / `fastcircuits` / old optimal-transport pipeline 开始，经过单文件端到端整合、DEBD HCLT runner 简化和 sample-DRO 执行，最终收敛为 `runDRO` reviewer-facing supplementary package。随后研究成果形成实际论文 **PeTeR: Post-Training Robustification of Probabilistic Circuits**，被 UAI 2026 Workshop on Tractable Probabilistic Modeling（TPM）接收并公开发布。TPM 阶段收口后，没有继续把旧 `runDRO` Runtime 无限扩写，而是为 AAAI full-length extension 新建独立任务：先执行 K=1/3/5 hyperparameter sweep，再执行 RLTPM K=3/K=5 GPU learning experiments。到 AAAI full-paper submission 后，production experiment execution 已再次收口，工作重心转向 appendix、supplementary、tables/charts 和外部可复现性整理。

**关键的文档分工**：`runDRO_STATIC_SPEC.md` 在任务后期把已经完成或取消的内部执行分支明确关闭，并把合同收敛为 standalone supplementary zip、单一 reviewer-facing runner、20 个 DEBD 数据集、完整 `fastcircuits/` 模块和 CSV 表格输出；`runDRO_RUN_STATE.md` 则只保存当时的真实当前状态——clean zip 和 clean delivery repository 已产生、远端检查通过、下一步只剩 handoff。TPM 阶段完成后，History 把 `runDRO` 和 workshop paper 一起标记为已完成历史，从而阻止旧 Runtime 被误当作仍然 active 的研究主线。

进入 AAAI 阶段后，相同结构被复用但没有复用旧合同。`peter_sweep_STATIC_SPEC.md` 只锁定 Adrian 指定的三条 sweep 命令、运行环境、结果边界和“不得为跑通而修改科学方法”等限制；对应 Runtime 从 Python 版本和依赖 blocker 一路记录到 production job 完成、3360/3360 个配置达到 `metrics.json` 或 `error.json` 终态，并在 Adrian 明确确认高 learning-rate 数值失败属于预期结果后完成 commit/push。这里的关键不是“让所有配置成功”，而是防止 Agent 把实验失败误判成必须修复的代码错误。

第二个 AAAI 任务进一步体现了 Runtime 的动态作用。`peter_rltpm_gpu_STATIC_SPEC.md` 固定 K=3/K=5、Python 3.11 和 Adrian 的 PyTorch/CUDA/Triton/PyJuice 运行栈，只把 `-j` 并发数留作允许通过证据调优的变量。Runtime 随执行持续吸收真实证据：Triton 因缺少 `Python.h` 失败后记录并验证 task-local header 修复；K=3 在 L40S 上验证 `j=8`、显存约 12 GiB；K=5 则通过实际 telemetry 证明 L40/L40S 不应默认承担 `j=8`，最终在完整 A100 80GB 上稳定运行，峰值显存约 47.8 GiB。最终 K=3 与 K=5 均达到 28/28 artifacts complete，并按批次审计、commit、push。Static 没有因为这些运行时发现而被污染成日志，Runtime 也没有把临时资源状态误写成永久科学约束。

**框架在这个案例中解决的核心问题**：长期科研项目的“下一步”会不断变化，但每个具体任务的科学边界又必须保持稳定。若使用一份不断增长的总计划，早期 runDRO 的 dataset、runner、Slurm 和 artifact 假设很容易污染后来的 PeTeR/AAAI 任务；反过来，如果每次只看最新 Runtime，又会失去“TPM 已收口、AAAI 已另开任务”的长期状态。这里通过 task-local Static/Runtime 与跨任务 History 的分层，把“研究主线连续”与“执行合同不连续”同时表达出来。

**结果**：该框架最终支撑的不再只是一个补充材料包，而是一条有实际论文产出的研究链：PeTeR workshop 论文完成、接收并公开；AAAI 延伸阶段的 3360 个 sweep 配置完成终态审计；RLTPM K=3/K=5 各 28/28 数据集实验完成并推送；production execution 收口后又能明确切换到 paper/supplementary reproducibility 工作，而不重新打开已经验收的旧任务。这个案例说明 Static / Runtime / History 的价值不只是“记录进度”，而是让一个跨数月、会反复改方向的科研项目仍然保持可验证的任务边界、可追溯的证据链和明确的停止条件。

---

### Case B：从自有 Windows 原型到可审计发布的 macOS 应用

**任务类型**：AudioShifter 最初是作者给母亲做的伴奏变调变速工具，早期 Windows 与 macOS 版本都能实际使用，但没有进入 Git，也没有采用系统化的软件工程流程。后来 macOS 源码遗失，只剩已经打包的 App；当需要进一步做移动端版本时，家中电脑性能有限，主要使用者又不熟悉电脑操作，原先依赖桌面端和手工维护的方式已经不再适合。于是项目重新建立公开仓库，先把可追溯的 Windows 历史实现作为只读参考，重新完成 macOS 工程化复刻，再以同一套产品合同为 Android 移植准备基础。这是一个边界清晰的单项目软件工程与发布任务，案例仓库为 [AudioShifter](https://github.com/smter6626/AudioShifter)；整个 macOS 阶段始终围绕“从自有历史原型重建并发布独立 macOS 应用”这一单一目标推进，没有 Case A 那种跨任务研究方向切换，因此采用 Static + Runtime + artifact verification，而没有额外 History，对应第七节的“严格”约束强度。

**时间跨度与阶段演化**：Runtime 记录的首次仓库盘点开始于 **2026-08-02 21:41 MST**；2026-08-03 完成行为合同、`behavior_spec.md`、`architecture_plan.md`、`verification_matrix.md`、模块化源码 MVP、真实音频/GUI 验收与 **130 项自动化测试**，随后推进到 PyInstaller 阶段。2026-08-04 完成独立 `.app` 构建、打包审计和受限环境下的四格式回验，并在同一天依次经历 alpha.1 → alpha.2 → alpha.3 三个 Release 候选迭代：alpha.1 的技术资产已经通过但因项目许可证未决停在 `PARTIAL`，owner 明确 GPL-3.0-or-later 与品牌边界后才继续推进，alpha.3 最终于 **2026-08-04 06:39 MST** 公开为 GitHub Pre-release。也就是说，从本轮 macOS 工程化复刻的首次仓库盘点，到完成需求合同、实现、测试、打包、依赖审计、许可证决策、Release 构建、非开发机验收并公开发布，实际经过约 **32 小时 58 分钟，约 33 小时**；最终测试全部通过。

**关键的文档分工**：[**`macos_rebuild_static.md`**](https://github.com/smter6626/AudioShifter/blob/main/docs/macos_rebuild_static.md) 锁定长期产品和平台合同，例如 Apple Silicon `arm64` only、不支持 Intel / Rosetta / `universal2`、不把 Developer ID 签名或 Apple 公证作为首阶段目标、四种输入格式、`-24~+24` 半音、`-95%~+400%` 相对变速、固定 44.1 kHz / 320 kbps 双声道 MP3 输出、Downloads 目录、不覆盖已有文件、源文件保护与取消清理等。它刻意没有提前声称“最低支持 macOS 27”，而是规定最低兼容版本必须由最终依赖、打包产物和实机验证决定；也没有在 owner 决策前擅自选定项目许可证，只规定二进制分发前必须解决 GPL 兼容路线和对应源码义务。实际验证到“仅 macOS 27.0 build `26A5378n` 被测试”、alpha.1 因许可证缺失被阻塞、owner 随后选择 GPL-3.0-or-later 等事实，都由 [**`macos_rebuild_runtime.md`**](https://github.com/smter6626/AudioShifter/blob/main/docs/macos_rebuild_runtime.md) 和专门法律文档在状态推进时记录，而不是反向污染早期 Static。源码、打包和 Release 的验收证据则分别落在 [`mvp_test_report.md`](https://github.com/smter6626/AudioShifter/blob/main/macos/mvp_test_report.md)、[`packaging_test_report.md`](https://github.com/smter6626/AudioShifter/blob/main/macos/packaging_test_report.md) 与 [`release_verification_v0.1.0-alpha.3.md`](https://github.com/smter6626/AudioShifter/blob/main/macos/release/release_verification_v0.1.0-alpha.3.md) 中。

**框架在这个案例中解决的核心问题**：与 Case A 的科学实验不同，这里的主要风险不是“实验失败会不会被误判成代码错误”，而是**开发机上能运行的代码、实际打包出的 `.app`、tag 对应源码和 GitHub 最终分发资产是否真的是同一个可复现产品**。开发环境中的 Homebrew、虚拟环境和本地动态库很容易让一个“本机可运行”的构建产生虚假安全感，因此每次阶段切换都要求由仓库产物和独立脚本给证据，而不是相信 Agent 的“已测试”汇报。`build_release_assets.sh` 从干净 tag 的 detached worktree 重新运行全部测试、重建 App、生成对应源码和 SHA-256；随后又从 GitHub Draft / Public Release 重新下载资产并针对下载副本重复审计，相当于把 Case A 的“LLM 独立验证优于 Agent 自评”映射成发布工程中的 artifact identity verification。alpha.1 在技术资产已经全部通过时仍因许可证决策缺失停在 `PARTIAL`，则对应执行循环中的 4.4：当 blocker 属于 owner policy 而不是技术实现时，Agent 必须停止并等待人类决策，而不是为了让流程变绿擅自补全合同。

**结果**：`v0.1.0-alpha.3` 最终达到 `165 passed, 0 failed, 0 skipped`，发布包审计覆盖 **75 个 thin-arm64 Mach-O、324 条动态引用和 20 个 `LC_RPATH`**，没有指向 Homebrew、虚拟环境或仓库的外部非系统运行时依赖；Release 同时提供 App ZIP、SHA-256 校验文件和可审计的 GPL 对应源码包。非开发 Apple Silicon Mac 完成 ZIP 下载、哈希校验、Gatekeeper 单应用放行、启动和实际使用验收；公开后又重新下载三项资产并确认字节数和摘要不变。这个案例说明“严格”档位并不需要机械增加 History：对一个边界稳定、但构建与发布一致性风险较高的软件任务，Static + Runtime + artifact verification 已足以形成完整证据链；与 Case A 的“完整”档位并列后，也直接展示了“工程强度应与任务风险匹配”不是抽象口号，而是两种不同任务结构下的实际选择。

---

## 七、适用边界与错误使用风险

本框架不是通用方法论，也不试图覆盖所有 LLM 使用场景。它适合的是可以被阶段化、可验证、可追踪的执行型任务，尤其适合计算机相关工作流，例如代码实现、实验脚本、数据处理、仓库审计、文档生成、批处理任务和科研工程支撑。这里的“适用”也不是二元判断：同一种任务可以根据失败成本和复现要求采用不同强度的框架。

### 适合的任务类型

- 有明确验收标准的工程任务，例如代码、文档、数据处理、实验脚本和仓库维护；
- 步骤之间有明确依赖关系的多阶段任务；
- 需要可追溯性的任务，其中 Git commit、logs、artifact metadata 和 History 文档共同提供审计轨迹；
- 需求可以在阶段边界被重新确认的科研工程任务，例如先 reconnaissance，再锁定合同，再实现，再验证。

### 工程约束强度建议

| 任务特征 | 建议约束强度 |
| --- | --- |
| 一次性、可逆、失败成本低 | 轻量：目标 + 关键约束即可 |
| 多步骤但容易人工检查 | 中等：Static + 简化 Runtime |
| 代码、实验、数据流水线 | 严格：Static + Runtime + artifact verification |
| 长期科研、昂贵实验、高复现要求 | 完整：Static + Runtime + History + Git evidence + acceptance gates |

PPT、演讲稿、一次性文案通常落在轻量或中等区间，因此可以主动降低工程化程度；代码和科研任务通常更接近严格或完整区间。但任务名称不是最终判断标准——例如一次性的低风险脚本可以轻量处理，而涉及生产数据迁移的短代码修改仍应使用高强度约束。

### 不适合的任务类型

- 单步骤、低风险、一次性的简单任务，如果完整框架的启动成本高于收益，则只保留必要约束而不应机械套用全部流程；
- 无法形成阶段性验收标准的开放式创意任务；
- 需要实时人类判断、审美判断或价值判断的任务；
- 外部状态高速变化且无法通过自动化 API 或强制 checkpoint 同步的任务。

### 错误使用风险

**风险一：把未确认需求写入 Static。**  
Static 只能固化已确认的目标、约束、输入和验收标准。若需求不明确，LLM 不应补全为稳定合同，而应显式写入 `UNKNOWN`、`TO_CONFIRM` 或 `REQUIRES_OWNER_DECISION`。否则 Static 会把早期幻觉固化成后续执行的“权威约束”。

**风险二：执行循环没有强制更新 Runtime。**  
Runtime 的状态滞后不是双文档机制本身的局限，而是执行循环没有把 Runtime 更新作为验收门的一部分。无论由用户手动触发，还是由 GitHub API 自动写回，只要 meaningful subtask 完成、blocker 出现或解除、active step 切换、证据推翻当前计划，就应更新 Runtime。

**风险三：只读取单一文档就继续执行。**  
LLM 每次进入任务前必须读取 Static 和 Runtime；长期项目还应读取 History。若多个文档在任务状态、目标、active phase 或 stop condition 上不一致，LLM 不应自行选择一个版本继续执行，而应报告冲突并等待裁决。

**风险四：把 Runtime 当作历史归档。**  
Runtime 是当前状态快照，不负责保存完整历史。完整演化轨迹应由 Git commits、History 文档、logs 和 artifact metadata 承担。Runtime 的目标是让下一次执行立即知道“现在做到哪、什么被阻塞、下一步是什么”。Runtime 中附带的 commit / evidence 和 supersession 信息只用于解释当前状态及其有效性，不应扩张成逐 commit 的 changelog。

**风险五：把内部控制系统暴露为最终交付物。**  
Static / Runtime / audit machinery 服务于执行控制，不等于最终产品形态。若项目目标是交付给外部使用者的脚本、README 或 public runner，最终交付物应保持简单、可读、可运行；内部验证框架只能作为支撑证据，不应污染 public-facing artifact。

**风险六：把框架强度固定化。**  
对低风险任务使用完整科研级控制会造成不必要的流程成本；对高风险任务过度简化又会失去可追踪性。应根据失败成本、可逆性、复现要求、任务持续时间和状态复杂度选择工程强度，而不是根据“是否使用 LLM”机械套用同一流程。

**风险七：只追加新结论，不废止旧结论。**  
当新证据推翻旧方案时，仅写入新的当前状态会让旧的 `PASS`、`validated` 或推荐记录继续作为歧义证据存在。Runtime 应明确标记被 supersede 的旧结论及其证据来源，使后续 LLM 不需要自行从 Git 时间线推断知识有效性。旧路线的实验和 commit 可以继续作为历史证据保留，但其“当前可用性”必须被显式撤销。

---

## 八、与相关工作的区别

**与 ReAct / Chain-of-Thought 的区别**：那些方法优化的是单次推理的质量，本框架解决的是多步骤任务中的状态管理和约束维护问题。两者不互斥，CoT 可以在单个 Step 内使用。

**与 AutoGPT / 类 Agent 框架的区别**：自主 Agent 框架倾向于最大化 LLM 的自主性。本框架的设计哲学相反——人类保留对 Static 合同变更的授权权、对 Runtime 更新的最终确认权或自动化规则设定权、对 4.4 情况的决策权。LLM 是执行者，不是最终决策者。

**与标准 SOP 文档的区别**：传统 SOP 是静态的，不追踪执行状态，也没有 LLM 参与的验证环节。本框架的 Runtime 文档是活的，每次状态变化都应留下可追溯记录，形成可审计的执行轨迹。

---

*最后更新：2026-08*