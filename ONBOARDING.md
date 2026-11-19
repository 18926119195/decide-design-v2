# decide-design-v2 入库指南

[一分钟定位](#一分钟定位) · [沿聊天时间线看灵感](#沿聊天时间线看灵感) · [从代码反查来由](#从代码反查来由) · [走读与维护记录](#走读与维护记录)

## 一分钟定位

这里保存了一次名词表与拓扑召回的集中设计。作者想把每个词在书上的真实位置留下来，让模型补足间隙，再沿各个出现点的邻词找到可用证据。现行独立评测与这套递归试验同在一仓，可从下表分辨。阅读基线为 [main · e61d9de](https://github.com/18926119195/decide-design-v2/commit/e61d9de9ef27b838b94a2102db449a40d162feec)。

## 沿聊天时间线看灵感

**如何读状态。** 已实现表示在所列版本找到与灵感相符的代码路径。部分实现表示已有其中一段。未实现表示已查的入口尚无对应实现。证据不足时标待确认。归档与设计文稿按资料本身的状态描述。

[早期导出](docs/chat-export/decide-design-chat-29c55109.md)有北京时间，[9 月 14 日记录](9.14的聊天记录)按行阅读。《聊天归纳》由 9 月 16 日提交加入，个别轮次无可独立核对的时间，以下保持该原件内部顺序。

| 编号与聊天原件 | 当时提出的思路 | 当前状态与代码证据 |
| --- | --- | --- |
| <a name="idea-i01"></a>I01 · 9 月 13 日 10:23–10:36，[会话 6–9](https://github.com/18926119195/decide-design-v2/blob/e61d9de9ef27b838b94a2102db449a40d162feec/docs/chat-export/decide-design-chat-29c55109.md?plain=1#L64-L177) | 规则词表与模型补词一起扩大候选，定位每次出现后以邻词袋选材料。 | **已实现于现行评测。** `/api/run` 和命令行调用 `runNounTableMaterial`，然后用人工标注评分。 [服务入口](src/server/main.ts#L310-L386) · [CLI](scripts/eval-decide.ts#L64-L101) |
| <a name="idea-i02"></a>I02 · 9 月 14 日，[L422–429](9.14的聊天记录#L422-L429) | 两个已有词的遮罩定义一组，组间并行，长组可在内部串联。 | **未落地为现行评测调度。** 现行监督的残文批次循环与这组规则不同。 [监督循环](src/harness/superviseS0S1.ts#L228-L304) |
| <a name="idea-i03"></a>I03 · 9 月 16 日入库，[归纳开头 L1–74](聊天归纳#L1-L74) | 将书块地址和全文偏移关联；补缺模型发现词，回到原文验证位置，并递归缩小剩余间隙。 | **部分实现。** 纯文表构建可独立运行，递归 S₁ 源码有深度上限；该编排导入因缺少 `src/core/batchUtil.ts` 失败。 [偏移表](src/core/pureTextTable.ts) · [递归补缺](src/core/nounS1Fill.ts) · [构表编排](src/core/pipeline.ts) |
| <a name="idea-i04"></a>I04 · 9 月 16 日入库，[归纳 L74–156](聊天归纳#L74-L156) | 限定连续合并的含义，并确定拓扑链在两端如何补位。合并后的词仍要能回查原出现点。 | **部分实现，未接入评测。** 合并和拓扑相关模块均已入库，现行材料选择器走独立的书块拓扑袋。 [合并](src/core/nounMerge.ts) · [拓扑链](src/core/topologyChain.ts) · [实际评测调用](src/harness/runNounTableMaterial.ts#L350-L459) |
| <a name="idea-i05"></a>I05 · 9 月 16 日入库，[归纳 L2924–2939](聊天归纳#L2924-L2939) | 作者确认纯文、S₀、S₁、合并和拓扑模块是否已经入库。 | **模块存在，完整运行待验证。** `test:full` 实际运行的是较早的段落实验。递归 `runFullPipeline` 只见于示例调用，不能用前者的结果证明。 [旧段落实验](scripts/test-complete-pipeline.ts) · [递归示例](src/core/examples.ts) |

相邻 [decide-design-main 的历史基线](https://github.com/18926119195/decide-design-main/commit/092d4646a083d21aa5eef0f44f43a0187512c349) 到这个版本，只增加了《聊天归纳》。[架构笔记](ARCHITECTURE_v2.md) 对照的是后几项意图。

## 从代码反查来由

- [runNounTableMaterial](src/harness/runNounTableMaterial.ts) 回看 [I01](#idea-i01)。先认清建表、问题勾选、命中定位与 decide 的现行路径。
- [superviseS0S1](src/harness/superviseS0S1.ts) 回看 [I02](#idea-i02)。它是现行的残文质量协作入口。
- [pureTextTable](src/core/pureTextTable.ts) → [nounS1Fill](src/core/nounS1Fill.ts) → [nounMerge](src/core/nounMerge.ts) → [topologyChain](src/core/topologyChain.ts) 回看 [I03](#idea-i03)–[I05](#idea-i05)。从 `pipeline.ts` 可看清这组模块的目标顺序。

## 走读与维护记录

第一次碰到“拓扑”这个词，在 [现行候选袋](src/core/bookNounIndex.ts#L483) 和 [递归拓扑链](src/core/topologyChain.ts) 各找一个构造函数，核对它们分别对应 [I01](#idea-i01) 和 [I04](#idea-i04)。Node 24 下 `smoke` 与 `test:s0` 通过，指定 126 字合成语料并预建 `output/` 后，旧段落实验的一道题召回 1 段。递归示例导入失败，真实模型未实跑。修改指南时记录运行的是哪一条链，再更新原编号所附的状态和提交。

**指南如何更新。** 以仓库和版本划定范围，从聊天提取原始目标，再用代码与产物校正对当时决策的理解。新证据出现时，回到原编号修订记录。这种读法借鉴了钱学森的系统工程和工程控制论。 [《组织管理的技术——系统工程》](https://doi.org/10.13255/j.cnki.jusst.2011.06.020) · [《工程控制论》](https://openlibrary.org/books/OL49218199M/Engineering_cybernetics)
