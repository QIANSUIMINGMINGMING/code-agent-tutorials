# Tutorial 03 Part 1 - Maya PDF Codex probe

This file records the compact evidence used by the Maya case in the tutorial.
It is a reproducible teaching probe, not a transcript from the historical
FlexEva collaboration.

## Paper identity

- FlexEva citation: the `maya2025` entry in FlexEva's `reference.bib`,
  entry `maya2025`.
- Citation target: arXiv `2503.20191`.
- Local PDF: `output/references/maya-arxiv-2503.20191/2503.20191.pdf`.
- PDF metadata title: *Maya: Optimizing Deep Learning Training Workloads using
  GPU Runtime Emulation*.
- PDF pages: 21.
- PDF SHA-256:
  `93d81a8ebcf460aa5b0bf08752e2ff110f894898e7f6cf45f26693c99ea21e23`.

## Probe runtime

- Date: 2026-07-15.
- Codex CLI: `0.144.4`.
- Model reported by the CLI: `gpt-5.6-sol`.
- Working directory contained only `2503.20191.pdf`.
- The agent made no project edits. It created a temporary extracted-text file
  at `/tmp/maya-paper.txt`.

## Exact failed sample prompt

```text
当前目录只有论文 2503.20191.pdf。请阅读它，并为“从零复现 Maya-style evaluator”给出足够直接编码的最小设计；重点包括 worker trace dedup 与 simulator。不要联网，不要请求澄清，也不要修改文件。若论文没有给出细节，请按常见实现补齐。保持简短，最终答复不超过 600 个汉字。
```

## Actual PDF-reading tool calls

The first call performed a broad extraction and search:

```bash
pdftotext -layout 2503.20191.pdf - \
  | rg -n -i -C 4 'evaluator|simulator|dedup|trace|worker'
```

The result was large, so Codex made a second, targeted extraction:

```bash
pdftotext -layout 2503.20191.pdf /tmp/maya-paper.txt && \
rg -n -i -C 12 \
  'Dynamic Worker Deduplication|worker dedup|4\.2|4\.4|Trace Collat|Event-driven' \
  /tmp/maya-paper.txt
```

The paper-related tool result made these facts model-visible:

- Unmodified training code is executed through a virtual device runtime.
- Per-worker traces are collated, annotated with predicted runtimes, and passed
  to an event-driven simulator.
- Worker dedup starts by profiling all workers during the first iteration and
  comparing rolling hashes of operation sequences.
- A key model-visible sentence says that duplicate detection terminates
  redundant workers while profiling continues only for unique ranks.
- The collator later reconstructs the full execution from the profiled ranks.
- The simulator uses resource queues, synchronization wait maps, and a priority
  queue for operation timing.

The PDF bytes themselves were not inserted as text. The model received the
selected `pdftotext` stdout as tool output in a later sampling request.

## Automatic completion returned by Codex

The final answer mixed the extracted facts with the following concrete design
choices that the prompt authorized Codex to fill in:

```text
Op{rank,host,device,stream,seq,kind,args,cpu_gap,duration,deps}

去掉 rank、地址、句柄等易变值；保留 op 类型、shape、dtype、stream 角色、
collective 类型及组内相对 rank。

哈希相同时再逐项比较；记录 rank -> representative。

memcpy = latency + bytes / bw
collective = alpha + beta * bytes * topology_factor

evaluate(traces, cluster)
  -> {iteration_time, peak_memory, compute_time,
      communication_time, timeline}

首测覆盖：串行流、双流重叠、跨流 event、collective 晚到、重复 worker 展开。
```

These items are shown without a correctness judgment in the tutorial so the
speaker can explain the failure boundary orally.
