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

基于上述失效分析，框架的设计围绕三条原则展开：

**原则一：职责不可混用（Separation of Concerns）**
文档按更新频率和修改权限严格分类。描述"做什么"的文档与描述"现在做到哪"的文档物理隔离，LLM 对两者的修改权限不同。

**原则二：约束前置，细节延迟（Constraints First, Details Late）**
固定的限制在项目启动时一次性写入，后续步骤的实现细节在该步骤成为 active 之前不做详细规划。避免基于假设的提前规划。

**原则三：提示词应当简单（Prompts Should Be Dumb）**
框架的复杂度承载在文档结构上，而不是提示词上。理想状态下，触发 LLM 执行下一步的提示词应当简单到任何人都能写出来，例如："Codex 已完成，汇报如下……请去 repo 检查验证，判断是否推进。"这意味着框架的健壮性不依赖于提示技巧。

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
- `RUN_STATE` 描述当前任务的最新执行状态、blocker 和 active step；
- `History` 记录跨阶段、跨任务的长期状态变化和阶段收口。

若 `History` 显示某个任务已经阶段收口，而旧的 `RUN_STATE` 仍显示 active，则不得继续按旧 Runtime 执行，应报告冲突并要求用户确认是否新开任务、恢复旧任务，或更新 Runtime。

若 `STATIC_SPEC` 与 `RUN_STATE` 冲突，LLM 不得自行修正其中一个文档。只有在用户授权或明确的执行循环规则允许时，才能更新对应文档。

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

**关于 Step 4 的两个设计决策：**

LLM 的验证基于 Git 仓库的实际产出，而不是 Codex 的汇报。Codex 自评存在盲区，LLM 独立拉取仓库内容做压力测试，这是两个不同视角的交叉验证。

4.3 的修复迭代记录为 Step i-a、Step i-b……保留完整修复轨迹，便于事后分析失效原因。

---

## 五、为什么这样有效

**约束空间压缩**：LLM 生成幻觉的根本原因是上下文不足时需要"猜"。Static 文档提供了不可动摇的地基，LLM 在执行时无需猜测"原始需求是什么"——它被写死在那里，且 LLM 不能在普通执行步骤中随意修改。

**局部上下文最大化**：每次执行时，LLM 的注意力集中在当前 Active Step 上，该 Step 的描述是所有步骤中最详细的。后续步骤的模糊性是刻意设计的，避免 LLM 基于假设提前做决策。

**双重验证消除自评盲区**：Codex 执行后自评，LLM 独立验证。两者使用相同的 Git 仓库但处于不同的执行角色，任何 Codex 的自我合理化都会在 LLM 的独立检查中暴露。

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

### Case B：[项目名称]

**任务类型**：[类型]
**步骤数**：[N]
**关键节点**：[描述]
**结果**：[产出质量]

---

## 七、适用边界与错误使用风险

本框架不是通用方法论，也不试图覆盖所有 LLM 使用场景。它适合的是可以被阶段化、可验证、可追踪的执行型任务，尤其适合计算机相关工作流，例如代码实现、实验脚本、数据处理、仓库审计、文档生成、批处理任务和科研工程支撑。

### 适合的任务类型

- 有明确验收标准的工程任务，例如代码、文档、数据处理、实验脚本和仓库维护；
- 步骤之间有明确依赖关系的多阶段任务；
- 需要可追溯性的任务，其中 Git commit、logs、artifact metadata 和 History 文档共同提供审计轨迹；
- 需求可以在阶段边界被重新确认的科研工程任务，例如先 reconnaissance，再锁定合同，再实现，再验证。

### 不适合的任务类型

- 单步骤、低风险、一次性的简单任务，因为框架启动成本高于收益；
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
Runtime 是当前状态快照，不负责保存完整历史。完整演化轨迹应由 Git commits、History 文档、logs 和 artifact metadata 承担。Runtime 的目标是让下一次执行立即知道“现在做到哪、什么被阻塞、下一步是什么”。

**风险五：把内部控制系统暴露为最终交付物。**  
Static / Runtime / audit machinery 服务于执行控制，不等于最终产品形态。若项目目标是交付给外部使用者的脚本、README 或 public runner，最终交付物应保持简单、可读、可运行；内部验证框架只能作为支撑证据，不应污染 public-facing artifact。

---

## 八、与相关工作的区别

**与 ReAct / Chain-of-Thought 的区别**：那些方法优化的是单次推理的质量，本框架解决的是多步骤任务中的状态管理和约束维护问题。两者不互斥，CoT 可以在单个 Step 内使用。

**与 AutoGPT / 类 Agent 框架的区别**：自主 Agent 框架倾向于最大化 LLM 的自主性。本框架的设计哲学相反——人类保留对 Static 合同变更的授权权、对 Runtime 更新的最终确认权或自动化规则设定权、对 4.4 情况的决策权。LLM 是执行者，不是最终决策者。

**与标准 SOP 文档的区别**：传统 SOP 是静态的，不追踪执行状态，也没有 LLM 参与的验证环节。本框架的 Runtime 文档是活的，每次状态变化都应留下可追溯记录，形成可审计的执行轨迹。

---

*最后更新：2026-08*
