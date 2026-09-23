# 原生运行与覆盖

## 三层配置

1. `config/routing.yaml` 是 Skill 的持久化行为策略和默认值权威来源。
2. `C:\Users\asus\.codex\astra_orchestrator.config.toml` 是 CLI 主代理启动 profile，只负责 Astra 的原生模型与推理档位，不承载自定义路由字段。
3. 当前任务覆盖由 Astra 解析，并在真实派发时映射到原生 `spawn_agent` 参数；临时覆盖不写回上述文件。

修改 `routing.yaml` 的持久化默认值后，新调用应重新读取它。若同时修改 root 默认值，还要同步修改 CLI profile，避免 CLI 启动值分叉。文件修改不会改变已经运行的主代理或 Worker；主代理模型/档位改变需要用相应选择创建新会话，客户端配置变更通常需要完整重载。

## 主代理

CLI 使用：

```powershell
codex -p astra_orchestrator
```

该 profile 原生提交 `model = "gpt-6-astra"` 和 `model_reasoning_effort = "medium"`。临时改变主代理档位可在启动时覆盖：

```powershell
codex -p astra_orchestrator -c 'model_reasoning_effort="high"'
```

桌面客户端不会自动继承某次 CLI 的 `-p` 选择。请在新建桌面任务时使用客户端可见的模型与 reasoning 选择器选择 Astra 和相应档位，再显式调用 Skill。Skill 文本不能切换已运行线程的主模型。

## Worker 派发

当前原生协作工具直接暴露 `task_name`、`model`、`reasoning_effort` 和 `fork_turns`。因此不创建会写死默认档位的独立 Worker TOML；每次派发才是 Worker 原生配置的实际提交点。示意映射：

```text
task_name: astra_orch_sol_math_01
model: gpt-6-sol
reasoning_effort: xhigh
fork_turns: a supported non-all context mode
```

Terra 与 Luna 同理。`task_name` 是本 Skill 专属的可追踪命名，不是模型证据。若调用 schema 或宿主允许值拒绝用户指定值，原样报告错误，不静默替换。

普通执行默认 Sol=`gpt-6-sol`、Terra=`gpt-5.6-terra`、Luna=`gpt-6-luna`，均为 `medium`。审计从独立的 `audit` 配置解析，默认新建 `gpt-6-sol`、`high`、`fork_turns=none` 的只读子代理，不复用实施线程。审计也占用宿主子代理名额，并且必须在被审计产物稳定后开始。

## 覆盖示例

- “这次 Astra high”：只改变新启动/已由客户端明确设置的主代理；执行 Worker 保持各自解析值，审计仍为 high。
- “算法设计用 Sol xhigh”：该 Task Packet 解析为 Sol/xhigh，并在对应 spawn 中显式提交；不改变独立审计档位。
- “所有代理都用 high”：主代理需以 high 新建/选择会话，每个 Worker spawn 均提交 high。
- “全部子任务交给 Sol”：Worker model 全部解析为 `gpt-6-sol`，主代理仍为 Astra；审计仍须新建独立线程。
- “所有执行代理 medium”：只覆盖执行 Worker，审计保持 high；“所有代理 medium”则明确覆盖审计在内的档位。
- “禁用 Luna”或“代码用 Terra，数学用 Sol”：改变当前路由选择，不改未指定角色档位。
- “只用 Astra”：`delegation.enabled = false` 作为当前覆盖，不派发 Worker 或审计员；如实说明独立审计未执行。
- “并发最多两个”：当前有效并发上限为用户值与宿主上限中的更严格者。
- “允许自动调整推理档位”：只有此时才进入自动档位模式，并仍限于用户范围和宿主支持值。

## 身份记录

分别记录：`requested`（用户或默认策略提出）、`resolved`（覆盖后准备使用）、`submitted`（实际原生参数/配置绑定）、`actual`（宿主实际报告）。不得把前三级复制成 `actual`。宿主未提供时写 `UNKNOWN`，并保留线程 ID、原生任务名和配置来源。
