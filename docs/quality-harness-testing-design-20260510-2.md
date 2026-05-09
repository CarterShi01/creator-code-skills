# Quality Harness Test Design for AI Coding

本文描述一套适用于 AI 辅助编码项目的测试体系设计原则与 Quality Harness 分层验证模型。

这是一套通用方法论，适用于任何使用 Claude Code 或类似 AI coding agent 进行开发的工程项目。

---

## 1. 背景：为什么 AI 编码时代测试变得更重要

### 传统开发流程

在 AI 辅助编码出现之前，开发者手动编写代码，CI 在合并前执行确定性验证：

```
developer writes code
→ local test
→ PR
→ CI
→ review
→ merge
```

### AI 辅助开发流程

引入 AI coding agent 之后，流程发生了根本性变化：

```
human defines goal
→ AI coding agent reads context
→ AI coding agent edits code
→ AI coding agent runs local verification
→ human reviews diff
→ PR CI runs deterministic verification
→ merge
```

新增了一层：**AI 本地自我验证（local verification）**。

这一层的目的是让 AI 在提交人工审查之前，自己先发现明显的错误，减少 review 负担。

但这一层**不替代 CI**。

### 关键变化

AI coding agent 进入了开发循环，但它不应成为正确性的最终裁判。

AI 可以生成代码、修复失败、运行测试、汇报结果，但判断一次改动是否可接受，仍然依赖**确定性命令**的结果。

AI 编码不减少对测试的需求，反而提高了测试的重要性——因为 AI 生成的改动必须由确定性验证命令来评判。

> **核心原则：没有经过验证的 AI 输出进入代码库（No unverified AI output enters the codebase）。**

---

## 2. 核心质量原则

AI coding agent 可以：

- **生成代码**（Generate code）
- **修复代码**（Repair code）
- **解释代码**（Explain code）
- **建议测试**（Suggest tests）
- **运行测试**（Run tests）
- **汇总测试结果**（Summarize results）

但只有**确定性命令（deterministic commands）**能决定一次改动是否可被接受。

### 角色模型

```
AI coding agent  =  coder + junior reviewer + repair assistant
CI               =  deterministic judge
Human            =  product owner + architect + final reviewer
```

### 关键原则

- **不能让 AI 既是球员又是裁判。** AI 运行验证是好的，但不能只凭 AI 的自我判断宣布任务完成。
- **验证必须是可执行的（executable）。** 写在文档或 prompt 里的规则是有用的指导，但指导不等于约束。
- **重要质量规则应最终成为脚本、测试、Hooks 或 CI 检查。** 只有机器可以执行的规则，才能被稳定执行。
- **任务不因 AI 宣布完成而完成。** 任务完成的标准是相关的验证命令通过，或者失败原因已明确报告。

---

## 3. 什么是 Quality Harness？

**Quality Harness** 是一组文档、脚本、测试、Hooks、Skills 和 CI 门控的集合，使 AI 生成的改动**安全、可审查、可重复、可验证**。

Quality Harness 不是产品逻辑，而是围绕产品的**工程基础设施**。

一个好的 Quality Harness 应能回答：

- AI 是否理解了项目的边界？
- AI 是否只修改了目标文件？
- AI 是否运行了相关检查？
- 生成的代码是否仍然通过 typecheck 和测试？
- 改动是否违反了架构约束？
- 文档或接口契约是否需要更新？
- 这次改动是否可以安全地被审查和合并？

---

## 4. 分层验证模型

一个成熟的 AI 编码质量体系应该包含多个层次。

### Layer 1：AI 本地验证

AI coding agent 在完成代码编辑后，应运行**最小相关验证命令**。

示例命令：

```bash
npm run verify:quick
```

可选范围：
- typecheck
- smoke tests
- 受影响的 unit tests
- 快速验证脚本

**目的：**
- 立即捕获明显失败。
- 减少 review 负担。
- 避免 AI "只写不验"的行为。
- 让 AI agent 对基本验证负责。

### Layer 2：Hook-based 轻量级检查

Hooks 可以在特定文件变更时自动触发轻量级检查。

示例：
- TypeScript 文件变更后：运行 typecheck + smoke tests。
- 文档或 harness 文件变更后：运行 harness 验证。
- 危险命令执行前：要求确认或阻断命令。

**重要原则：**
- Hooks 应保持轻量，提供快速反馈。
- Hooks 不是最终验证门控。
- Hooks 有价值，因为它们是确定性的，而自然语言指令只是建议性的。

### Layer 3：任务完成验证（Task-completion verification）

AI coding agent 在宣布任务完成前，必须运行相关验证命令。

适用场景：

| 场景 | 推荐命令 |
|---|---|
| 小型实现改动 | `verify:quick` |
| 架构或 harness 改动 | `verify:harness` |
| 功能完成 / 准备 PR | `verify` |

AI 应报告：
1. 运行了哪条命令。
2. 是否通过。
3. 如果失败，什么失败了。
4. 为修复失败做了什么改动。
5. 修复后是否重新运行并通过。

### Layer 4：PR CI

CI 在干净的远程环境中运行。

**为什么 CI 仍然必要：**
- 本地验证可能受本地状态影响（未提交的文件、本地环境变量、本地安装的包等）。
- CI 验证代码库在从零安装后是否仍可工作。
- CI 对人工编写的代码和 AI 生成的代码执行相同的标准。
- AI 生成的代码不应享有任何特殊豁免。

示例 CI 流程：

```bash
npm ci
npm run verify
```

区别说明：

- `npm ci`：基于 lock 文件进行干净的依赖安装。
- `npm run verify`：运行项目定义的确定性验证套件。

### Layer 5：Branch protection 与 required status checks

CI 工作流可以运行，但不会自动阻断合并。只有将其配置为 **required status check** 并启用 **branch protection**，才能成为合并门控。

**推荐分阶段做法：**
- 首先添加 CI，但不要求它通过才能合并（soft CI）。
- 观察 CI 是否稳定。
- 修复依赖安装问题、环境问题和不稳定测试（flaky tests）。
- 待 CI 稳定后，配置 branch protection，将 CI 升级为硬性合并门控（hard CI）。

这在 AI 辅助项目中尤其重要，因为初期的 Quality Harness 可能还在演进中。

### Layer 6：人工审查

人工审查不可替代。

人类应审查：
- 产品意图是否正确。
- 架构权衡是否合理。
- 测试是否有意义。
- AI 是否修改了过多文件。
- 验证范围是否充分。
- 实现是否符合原始目标。

CI 可以验证正确性边界，但**设计判断仍由人类负责**。

---

## 5. 测试层设计

推荐的通用测试目录结构：

```
tests/
  smoke/
  unit/
  contract/
  harness/
  integration/
  e2e/
```

### Smoke Tests（冒烟测试）

**目的：** 验证最重要的代码路径能基本运行。

适合作为早期项目的第一验证层。应快速、稳定，捕获"系统明显损坏"的问题。

典型示例：
- 核心函数可以被导入并调用。
- 基本对象可以被创建。
- 最简工作流可以运行而不抛出错误。
- CLI 命令可以成功启动。

Smoke Tests 不需要覆盖所有边界情况。

### Unit Tests（单元测试）

**目的：** 验证函数级别的详细行为。

覆盖：正常输入、边界情况、非法输入、错误处理。

典型示例：
- 输入字符串 trim 行为
- 参数校验逻辑
- 默认值处理
- 错误场景抛出正确的 error
- 纯粹的转换逻辑（transformation logic）

Unit Tests 应靠近代码，运行速度快。

### Contract Tests（契约测试）

**目的：** 验证公共模块的对外接口契约和 exported 类型。

保护其他模块或用户依赖的 API。在模块化架构和 AI 辅助代码库中尤其重要——AI 可能意外地重命名字段或改变返回结构。

典型示例：
- exported 函数名称保持稳定。
- 返回对象结构保持稳定。
- 公共错误行为保持稳定。
- 命令行输出格式保持稳定。
- API 请求/响应 schema 保持稳定。

Contract Tests 不仅适用于外部 API，也适用于保护内部模块边界。

### Harness Tests（规则验证测试）

**目的：** 验证项目规则，而非产品行为。

将架构约束和协作规则从"书面建议"变成"可执行约束"。在 AI coding agent 参与修改代码库时尤为重要。

典型示例：
- 某些目录不能从被禁止的层导入。
- 必要的文档文件必须存在。
- Skill 文件必须遵循预期结构。
- 生成文件不应出现在源码目录中。
- 公共契约变更必须附带文档更新。
- 依赖边界规则必须被执行。

**Harness Tests 与产品测试的核心区别：**
- 产品测试问：产品行为是否正确？
- Harness Tests 问：开发过程是否遵守了项目规则？

### Integration Tests（集成测试）

**目的：** 验证模块之间的交互。

适合在多个模块真实可用之后引入。应测试有意义的边界，而非每个内部细节。

典型示例：
- parser + validator 的组合行为。
- storage + service 的数据流。
- API 层 + 业务逻辑的集成。
- 工作流步骤 A + 步骤 B 的串联。

Integration Tests 应在真正的模块协作存在后再添加。

### E2E Tests（端对端测试）

**目的：** 验证完整的用户侧工作流。

成本最高，最容易脆化。不适合作为早期项目的第一测试层。

典型示例：
- 用户创建输入 → 系统处理 → 生成输出。
- CLI 命令运行完整工作流。
- Web UI 流程成功完成。
- 外部集成在真实场景下正常工作。

E2E Tests 应在工作流足够稳定后引入。

---

## 6. Minimal-first 设计原则

**不要在项目初期过度设计测试体系。**

第一版应极小：
- typecheck
- 一两个 smoke tests
- 基本的 test scripts
- 各测试层的 README 占位文件

但**结构应反映未来的意图**。

### 推荐初始结构

```
tests/
  README.md              ← 说明测试体系分层意图
  smoke/
    README.md
    basic.smoke.test.ts  ← 唯一实现的测试
  unit/
    README.md            ← 占位，说明未来用途
  contract/
    README.md            ← 占位
  harness/
    README.md            ← 占位，说明规则验证用途
  integration/
    README.md            ← 占位
  e2e/
    README.md            ← 占位
```

**说明：**
- 初期只需实现 smoke tests。
- 其他目录可以是占位结构。
- 占位目录有价值，因为它们**传递意图**，而不是假装功能已存在。
- 避免过度工程化，同时保持清晰的长期质量模型。

---

## 7. 推荐的 npm scripts 设计

```json
{
  "scripts": {
    "typecheck": "tsc --noEmit",
    "test": "vitest run",
    "test:smoke": "vitest run tests/smoke",
    "test:unit": "vitest run tests/unit",
    "test:contract": "vitest run tests/contract",
    "test:harness": "vitest run tests/harness",
    "verify:quick": "npm run typecheck && npm run test:smoke",
    "verify:harness": "npm run typecheck && npm run test:harness",
    "verify": "npm run typecheck && npm run test"
  }
}
```

### 各 script 用途

| Script | 用途 |
|---|---|
| `verify:quick` | AI 本地快速反馈，适用于小型实现改动 |
| `verify:harness` | 架构、文档、Skills、Hooks、规则类改动后运行 |
| `verify` | 任务完成前 / 准备 PR 时运行，也是 CI 的入口命令 |

**原则：**
- Scripts 应**稳定且确定性（stable and deterministic）**。
- AI agent 运行和开发者运行，结果应一致。
- 不依赖隐式的本地环境状态。
- 命名约定本身有价值：AI agent 需要可预测的命令接口。

**非 TypeScript 项目：** 将 `typecheck`/`test` 替换为对应生态的等价命令，但保持相同的命名约定。

---

## 8. 推荐的分阶段落地计划

### Step 1：最小测试基础

**目标：** 让项目可测试。

任务：
- 添加 typecheck 配置（如 `tsconfig.json`）。
- 添加轻量级 test runner（如 Vitest）。
- 添加第一个 smoke test。
- 添加 `verify:quick` 和 `verify` scripts。
- 添加各测试层的 README 占位文件。
- 记录测试策略文档。

**不要做：**
- 添加重型 E2E 框架。
- 添加复杂的集成测试。
- 在本地验证稳定之前添加 CI。
- 过早添加严格的架构检查。

**交付标准：** `npm run verify:quick` 通过。

### Step 2：轻量级 Harness 验证

**目标：** 让项目规则可执行。

任务：
- 添加基本的架构边界测试（architecture boundary tests）。
- 添加文档存在性检查（documentation presence tests）。
- 添加 Skill 或 agent 配置的格式检查（如适用）。
- 添加依赖边界检查（如适用）。
- 添加 `verify:harness` script。
- 初期保持规则宽松。

**不要做：**
- 过早强制要求未来的占位文件存在。
- 将未来模块的缺失视为构建失败。
- 如果简单扫描就够用，不要构建基于 AST 的复杂分析器。
- 把 harness tests 当成产品测试来设计。

**交付标准：** `npm run verify:harness` 通过。

### Step 3：工作流与 CI 集成

**目标：** 从本地验证升级为仓库级质量门控。

任务：
- 向 agent 指令文件（如 `AGENTS.md`）添加 Verification Policy。
- 记录 Hook 策略文档（可先以文档形式存在，暂不实现）。
- 添加 CI 工作流（在 pull_request 和主分支 push 时触发）。
- 初期以 soft CI 运行（CI 结果可见但不阻断合并）。
- 待 CI 稳定后配置 branch protection 和 required status checks。

**关键说明：**
- CI 配置可以在成为硬性合并门控之前就存在。
- CI 工作流运行不自动阻断合并。
- 阻断合并需要 branch protection + required status checks。
- 分阶段做法避免因不成熟的检查意外锁死所有 PR。

**交付标准：** PR CI 通过，branch protection 生效后可作为合并门控。

---

## 9. CI vs 本地验证

| 维度 | 本地验证 | CI |
|---|---|---|
| 执行者 | 开发者或 AI agent | GitHub / GitLab / CI 平台 |
| 时机 | 开发过程中 | PR 或 push 时 |
| 反馈速度 | 快 | 较慢 |
| 环境 | 可能依赖本地状态 | 干净环境 |
| 价值 | 快速修复循环 | 验证仓库完整性 |
| 合并门控 | 否 | 可以是 |

典型命令对比：

```bash
# 本地运行
npm run verify:quick
npm run verify

# CI 运行
npm ci
npm run verify
```

**说明：**
- `npm ci` 从干净的 lock 文件状态安装依赖，不依赖本地缓存。
- `npm run verify` 在本地和 CI 中应保持相同行为。
- 相同的 `verify` 命令应可同时用于本地和 CI，这是脚本设计稳定性的重要标志。

---

## 10. Soft CI vs Hard CI

### Soft CI

- CI 自动运行。
- CI 结果在 PR 上可见。
- CI 失败向 reviewer 发出警告。
- CI 失败**不阻断合并**。

### Hard CI

- CI 被配置为 required status check。
- Branch protection 要求其通过。
- CI 失败**阻断合并**。

### 推荐方式

1. 先以 Soft CI 运行。
2. 稳定依赖安装和测试。
3. 排除 flaky checks。
4. 再将 CI 升级为 Hard CI 门控。

这在 AI 辅助项目中尤其重要，因为初期的 Quality Harness 可能还在演进，过早强制 Hard CI 会导致不必要的阻塞。

---

## 11. Verification Policy 模板

以下政策块可直接复制到 agent 指令文件（如 `AGENTS.md`）中：

```markdown
## Verification Policy

For any implementation change, the AI coding agent must run the smallest relevant
verification command before finishing.

Preferred commands:
- `npm run verify:quick` for small local implementation changes.
- `npm run verify:harness` for architecture, documentation, skills, hooks, or harness changes.
- `npm run verify` before completing a task or preparing a PR.

The AI coding agent must report:
1. What command was run.
2. Whether it passed.
3. If it failed, what failed.
4. What was changed to fix it.
5. Whether the verification command was rerun successfully.

Do not claim a change is complete unless the relevant verification command has passed,
or clearly state why verification could not be run.

No unverified AI output enters the codebase.
```

**政策的核心作用：**
- 给 AI agent 明确的"任务完成标准"，而不是编辑完代码就宣布 done。
- 使验证行为成为工作流的一部分，而非事后补救。
- 提供足够的灵活性（允许说明为何无法运行验证），而不是死板地要求每次都必须有 passing output。

---

## 12. Hook 策略模板

Hooks 适合用于**确定性的轻量级强制检查**。

### 推荐 Hook 场景

| 触发条件 | 推荐动作 |
|---|---|
| 源码文件编辑后 | 运行 `verify:quick` |
| 测试文件变更后 | 运行相关测试 |
| 架构 / harness 文档变更后 | 运行 `verify:harness` |
| 危险命令执行前 | 要求确认 |
| 依赖变更前 | 要求显式批准 |

### 重要原则

- Hooks 应保持快速。
- Hooks 不替代 CI。
- 除非项目极小，否则不应在每次小编辑后运行全量测试套件。
- 完整验证属于任务完成时和 CI，不属于每次文件保存。
- **手动验证流程稳定后，再引入 Hooks 自动化。** 不要在流程未被人工验证之前就自动化它。

---

## 13. 常见错误与避免方法

以下是 AI 辅助项目中常见的质量体系设计错误：

**不要做：**

- **将 AI 的自我评估视为验证。** AI 说"通过了"不等于真的通过了。
- **让 AI 在没有运行命令的情况下宣布任务完成。** 完成声明必须附带验证证据。
- **在项目行为稳定之前引入大型测试框架。** 框架应跟随需求增长，而非超前于需求。
- **过早强制要求未来的占位文件存在。** 让检查规则随项目成熟而收紧。
- **以 E2E Tests 作为第一验证层。** E2E 是最脆、最贵的层，应最后引入。
- **在 CI 稳定之前就配置 Hard CI。** 未成熟的 Hard CI 会阻塞所有 PR。
- **让测试依赖本地环境状态。** 测试必须在干净环境可重复运行。
- **给 AI 生成的代码更容易的合并路径。** AI 生成的代码与人工编写的代码应遵守相同标准。

---

## 14. 设计总结

### 验证层分工

| 层次 | 执行者 | 性质 | 阶段 |
|---|---|---|---|
| AI 本地验证 | AI coding agent | 即时反馈 | 开发中 |
| Hook-based 检查 | 自动触发 | 轻量级补充 | 开发中 |
| 任务完成验证 | AI coding agent | 完成证明 | 任务结束时 |
| PR CI | CI 平台 | 确定性门控 | PR 时 |
| Branch protection | GitHub / GitLab | 合并门控 | 稳定后 |
| 人工审查 | Human | 设计判断 | 全程 |

### 测试层分工

| 层次 | 验证对象 | 优先级 |
|---|---|---|
| Smoke Tests | 核心路径能运行 | 最高，最先实现 |
| Unit Tests | 函数级详细行为 | 高 |
| Contract Tests | 公共接口契约 | 高（模块化项目） |
| Harness Tests | 项目规则与架构约束 | 高（AI 辅助项目） |
| Integration Tests | 模块间交互 | 中（后期引入） |
| E2E Tests | 完整用户工作流 | 低（最后引入） |

### 最终原则

传统 CI 不被移除。  
AI coding 增加了更早的本地验证层。  
Hooks 自动化轻量级检查。  
任务完成验证让 AI 汇报验证证据。  
PR CI 保持确定性的仓库级验证门控地位。  
Branch protection 可在 CI 稳定后将其升级为硬性合并门控。  
Harness Tests 将项目规则转化为可执行约束。  
人工审查保持对设计判断的最终责任。  

> **每一次 AI 生成的改动，都必须能被确定性命令所验证。**  
> **Every AI-generated change must be verifiable by deterministic commands.**
