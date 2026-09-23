# 手动检查清单

以下是行为验证场景，清单本身不表示已经执行。请只用临时目录、无破坏性的只读或小文件任务；仅运行当前任务授权范围内的检查。

- 普通任务不写 `$astra-orchestrator`：预期不触发该 Skill。
- 显式调用 `$astra-orchestrator`：预期读取策略，由 Astra 先拆分任务、标注依赖和所有权，再按难度逐项路由。
- 分别给出低歧义扫描、一般实现、困难推导：预期依次路由 Luna、Terra、Sol，而不是按字数或耗时路由。
- 不写覆盖：预期执行 Sol=`gpt-6-sol`、Terra=`gpt-5.6-terra`、Luna=`gpt-6-luna`；Astra 与执行 Worker 默认 `medium`，独立审计为 `gpt-6-sol`、`high`。
- 指定“这次 Astra high”：预期只改变主代理；执行 Worker 仍为 `medium`，审计保持 `high`。
- 指定“算法设计 Sol xhigh”：预期对应 spawn 的 `model` 为 `gpt-6-sol`、`reasoning_effort` 为 `xhigh`；若宿主不支持则报告真实错误。
- 指定“禁用 Luna”：预期不派发 Luna，任务在允许角色间重新路由。
- 指定“只用 Astra”：预期没有 Worker 或审计员派发，并说明默认独立审计未执行。
- 指定“并发最多两个”：预期同时活动的 Worker 不超过 2，写任务的 `owned_paths` 互不重叠。
- 让两个任务请求同一写路径：预期串行、合并或改为一个只读任务，不并发写入。
- 检查运行记录：预期区分 `requested/resolved/submitted/actual`；宿主未报告真实模型或档位时为 `UNKNOWN`。
- 模拟一次模型调用失败：预期不静默 fallback，不把其他模型声称为原模型。
- 完成混合任务：预期独立工作并行、依赖任务串行；整合完成且写入结束后，用新线程和 `fork_turns=none` 启动 Sol High 只读审计；实施代理不能充当审计员。
- 审计指出真实缺陷：预期 Astra 判断并协调修复，对变更执行针对性验证；影响旧审计结论时交给原审计员复核，审计员不直接修改文件。
- 指定“所有执行代理 medium”：审计仍为 high；明确指定“所有代理 medium”时审计档位随用户覆盖。
- 核心数学建模：预期保留全局 Astra/Ultra 与阶段确认边界，不按一般复杂任务默认改成 Sol。

这些预期行为需要用户实际运行后才能确认；清单本身不是测试结果。
