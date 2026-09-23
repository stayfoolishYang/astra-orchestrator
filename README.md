# Astra Orchestrator 使用说明

这是一个仅在显式调用时启用的用户级通用 Codex Skill。Astra 先按难度与依赖拆分任务，通过原生多代理接口派给 GPT-6 Sol、Terra 或 GPT-6 Luna；整合后新建 GPT-6 Sol High 只读审计子代理，处理发现并完成验证后交付。

默认分工：低难度且低风险的明确任务交给 `gpt-6-luna`，一般工程任务交给 `gpt-5.6-terra`，复杂算法、架构与故障分析交给 `gpt-6-sol`；执行 Worker 均为 `medium`。核心数学建模保留全局 Astra/Ultra 路由。独立审计使用 `gpt-6-sol`、`high`，与实施代理分离。默认最多三个并行子代理（含审计），有依赖或共享写入的任务按顺序执行。

## 安装与启动

- Skill：`C:\Users\asus\.codex\skills\astra-orchestrator`
- CLI 主代理 profile：`C:\Users\asus\.codex\astra_orchestrator.config.toml`
- 标准调用：在任务中明确写 `$astra-orchestrator`。
- CLI 以 Astra medium 启动：`codex -p astra_orchestrator`
- 桌面端：新建任务时在客户端模型/推理选择器中选择 `gpt-6-astra` 和 `medium`，然后显式调用 `$astra-orchestrator`。CLI profile 不会自动控制桌面端，Skill 文本也不会切换已经运行的主代理。

没有显式调用时，本 Skill 不触发。

## 常用临时覆盖

CLI 临时把主代理改为 high：

```powershell
codex -p astra_orchestrator -c 'model_reasoning_effort="high"'
```

当前任务可直接用自然语言指定 Worker、档位和并发；Astra 会把每个 Worker 的解析结果显式映射到原生派发参数，而不是只改文字角色：

```text
$astra-orchestrator
按任务难度与依赖拆分工作，使用默认执行路由，整合后由新建 Sol High 子代理独立审计。
```

```text
$astra-orchestrator
这次 Astra high，复杂算法设计 Sol xhigh，常规模块实现 Terra medium，独立审计保持默认。
```

```text
$astra-orchestrator
只使用 Astra，不开启子代理。
```

```text
$astra-orchestrator
禁用 Luna，并发最多两个，其他保持默认。
```

主代理档位变化只有在客户端选择或新会话启动参数中才真正生效。Worker 覆盖在每个 `spawn_agent` 调用中通过 `model` 与 `reasoning_effort` 真正提交。宿主不支持某档位时应报告错误，不会静默替代。

## 持久化设置

持久化路由、角色、档位、并发或委派默认值时，编辑 `config/routing.yaml`；它是 Skill 行为配置的权威来源。若改变 `root.model` 或 `root.reasoning`，同时更新 `C:\Users\asus\.codex\astra_orchestrator.config.toml` 的同名值，确保 CLI profile 与策略一致。不要把 `routing.yaml` 的自定义字段复制到 Codex `config.toml`。

修改 Skill 行为配置后，在下一次显式调用中重新读取。修改 CLI profile 后新建 CLI 会话。桌面端模型设置按客户端模型选择器执行，必要时完整重启客户端加载 Skill/元数据变更。任何文件变化都不会热切换已运行代理。

临时措辞如“这次”只作用于当前范围。只有明确说“以后默认如此”或“保存到配置”才持久化。说“恢复默认”会清除当前适用范围的覆盖。

## 三种状态不要混淆

- 路由计划：根据策略得到 `resolved`，尚未派发。
- 真实派发：原生调用已经提交，记录在 `submitted`。
- 实际运行信息：只记录宿主报告的 `actual`；宿主未暴露时为 `UNKNOWN`。

自定义任务名或 Worker 自称某模型不能证明实际身份。默认不做模型失败后的自动 fallback。

## 移除

关闭正在使用该 Skill 的会话后，仅删除：

- `C:\Users\asus\.codex\skills\astra-orchestrator`
- `C:\Users\asus\.codex\astra_orchestrator.config.toml`

这不会修改或移除 `sol-luna-orchestrator`、其他 Skill、全局默认模型、认证、服务商或项目配置。删除前如有自定义内容，请自行备份。

手动行为检查见 [`MANUAL_CHECKS.md`](MANUAL_CHECKS.md)。
