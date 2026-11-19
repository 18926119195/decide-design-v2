# decide-design-v2 入库指南

这里保存了把书块坐标、递归补缺、名词合并和拓扑召回串起来的协作设计，以及已有名词表评测的代码。先查意图和实际运行路径的交叉表，再决定维护哪一条链。审阅基线为 [main · e61d9de](https://github.com/18926119195/decide-design-v2/commit/e61d9de9ef27b838b94a2102db449a40d162feec)。

## 意图、证据与接线

| 会话节点 | 设计控制量 | 当前入口与结论 |
| --- | --- | --- |
| 早期评测和 9 月 14 日分批讨论 | 从 S₀ 转入遮罩残文，围绕两个遮罩定组，并最终能够导出质量监督结果。 [脱敏导出](docs/chat-export/decide-design-chat-29c55109.md) · [9 月 14 日原件](9.14的聊天记录) | Web 的 [`/api/run`](src/server/main.ts) 和 [CLI](scripts/eval-decide.ts) 走 [runNounTableMaterial](src/harness/runNounTableMaterial.ts)。现有监督、缓存、输出分别从 [superviseS0S1](src/harness/superviseS0S1.ts) 和 [monitor](src/harness/buildMonitorExport.ts) 查。 |
| 9 月 16 日入库的聊天归纳开头 | 书块与 pure text 坐标分层。S₁ 找到子串之后用原文确定偏移，再把间隙继续切小。 [归纳 L1-L74](https://github.com/18926119195/decide-design-v2/blob/e61d9de9ef27b838b94a2102db449a40d162feec/%E8%81%8A%E5%A4%A9%E5%BD%92%E7%BA%B3#L1-L74) | [pureTextTable](src/core/pureTextTable.ts)、[nounS0New](src/core/nounS0New.ts) 和 [nounS1Fill](src/core/nounS1Fill.ts) 有代码。补缺代码有默认最大深度 3。 |
| 归纳中段确认连续合并和链边界，末段追查模块入库 | 同形词去重后仍保留出现坐标，合并结果进入拓扑链。 [归纳 L74-L156](https://github.com/18926119195/decide-design-v2/blob/e61d9de9ef27b838b94a2102db449a40d162feec/%E8%81%8A%E5%A4%A9%E5%BD%92%E7%BA%B3#L74-L156) · [模块追查 L2924](https://github.com/18926119195/decide-design-v2/blob/e61d9de9ef27b838b94a2102db449a40d162feec/%E8%81%8A%E5%A4%A9%E5%BD%92%E7%BA%B3#L2924-L2939) | [合并](src/core/nounMerge.ts)、[拓扑链](src/core/topologyChain.ts) 和 [runFullPipeline](src/core/pipeline.ts) 在源码中。当前无 CLI 或 Web 调用指向该编排，导入示例实测因缺少 `src/core/batchUtil.ts` 退出。 |

[ARCHITECTURE_v2.md](ARCHITECTURE_v2.md) 是看结构的入口。新同事可用 [相邻主快照](https://github.com/18926119195/decide-design-main/commit/092d4646a083d21aa5eef0f44f43a0187512c349) 比对。v2 的 HEAD 提交只加《聊天归纳》，两个快照的代码差异不能靠仓库名推断。

## 先跑已有的基线

```sh
npm run smoke
npm run test:s0
mkdir -p output
CORPUS_PATH=/path/to/book.docuverse.json LLM_API_KEY= npm run test:full
```

本次用 Node 24.22.1 复跑。前两条通过，S₀ 内置短样例输出 0 段。`test:full` 默认语料缺失，提供 126 字的一段合成书包并创建 `output/` 后通过，其中一条内置查询召回 1 段，其余两条词汇无匹配。`test:full` 测的是 [旧的段落召回实验](scripts/test-complete-pipeline.ts)。递归编排在本次导入失败，真模型和端到端切换均待验证。

## 按反馈闭环推进

1. **恢复模块加载**。以固定 BookIndex 和注入的 Mock 模型恢复 `runFullPipeline` 可运行，顺手校对 [示例](src/core/examples.ts) 向纯文表提供的数据类型。确认输入书块 ID、可接受响应格式和调用预算后再立基线。
2. **验证状态转移**。覆盖命中两端、内部两点、单个命中、没有命中、重复同形、模型返回书上没有的词。每轮记录父子区间、偏移、通过和丢弃数、深度及停止原因。目标是 0 个越出父区间的命中，所有被接收位置都能切回原字串，零新增时该支停止，整条调用受预算限制。区间或坐标错误先修补缺，拓扑顺序错误先查合并产物。
3. **比较召回后再决定切换**。固定语料、问题、gold、模型与预算，比较旧 harness 和新编排的 P/R、命中页、调用数、超时和失败原因。由设计负责人确认语义歧义、偏移定义与允许折损，由评测负责人签验收。若未达事前门限，保留现行入口并把最小失败输入连回会话和代码。

## 方法依据

本指南把总体设计落实为边界、目标和任务接口，把工程控制中的反馈落实为观测、判据和纠偏。维护者用原始协作记录、代码和固定样本共同核验定性判断与量化结果，据此决定下一轮行动。[《工程控制论》](https://openlibrary.org/books/OL49218199M/Engineering_cybernetics) · [《组织管理的技术——系统工程》](https://doi.org/10.13255/j.cnki.jusst.2011.06.020) · [《一个科学新领域——开放的复杂巨系统及其方法论》](https://cnki.istiz.org.cn/kcms/detail/detail.aspx?dbcode=CPFD&dbname=cpfd1999&filename=ZGXC199008001001)
