# 15-minute presentation script (for the recorded Zoom)

Total target: 13–14 minutes, leaves margin. Record at 1280×720 or above, share the screen with
`REPORT.docx` open plus a terminal for the live demo.

---

## 0:00–1:30 — What this is (1.5 min)

- "This is a reproducibility study of GenericAgent, the paper behind the first item on the course list."
- One line on the system: nine atomic tools, a ~100-line loop, hierarchical memory where only a small
  index stays in the prompt.
- The target I chose: Section 4.3.2 / Table 5 — the condensed-memory ablation on SOP-Bench dangerous goods.
- Why this one: smallest self-contained experiment, deterministic local tools, released test set with labels.
- State the headline up front: "One result reproduces, one does not. The reported TSR gradient does not
  reproduce on the released protocol; the mechanism does, once I remove one tool."

## 1:30–3:30 — Setup (2 min)

- Show `README.md`: two repos, pandas, Linux VM, no GPU.
- Backbone swap: paper used GPT-5.4; I used DeepSeek-V4-Pro at temperature 0. Config is four lines in
  `mykey.py`. Say explicitly: "Results are not directly comparable because the underlying LLM differs;
  same tasks, same tools, same scoring script."
- Show `ga_sop_bench.py` and point at two imports: `agent_runner_loop` and `OrderFulfillmentManager`.
  "Nothing re-implemented — this is the authors' loop and the authors' tools."
- Memory blocks had to be reconstructed: the paper reports only token sizes. Mine measure 0/419/303/133
  against the reported 0/575/288/165.

## 3:30–5:00 — Metric and the oracle check (1.5 min)

- TSR = fraction of 20 rows matching the reference label. Five-way label space.
- The important step: run the four tools deterministically, in order, on every row, and compare with
  the labels. Show `oracle_check.py` output: 20/20.
- Two consequences, say them slowly:
  1. the task is fully determined by tool outputs, so no label noise can explain a gap;
  2. the majority class is 60%, far above the paper's No-Memory 13.87% — so a no-memory agent must be
     failing to produce a valid label, not just guessing the majority class.

## 5:00–7:30 — Baseline: the negative result (2.5 min)

- Show Table 1: all four configurations 100%, zero variance across three trials.
- Explain the three mechanisms: the decision tool implements the procedure; its schema enumerates the
  five labels; each tool description references the previous tool's outputs. Procedure inferable from
  schemas alone.
- Show the token column: memory is pure cost here — Full-Memory ~38% more tokens at identical accuracy.
- Be honest about the confound: "My first iteration restated the procedure in the task text. It also
  scored 100% everywhere. I only caught it by reading per-row tool traces — No-Memory was calling all
  four tools in the right order. That is what sent me to the oracle check."

## 7:30–10:30 — The modification (3 min)

- The change: delete `make_fulfillment_decision` from schema and dispatch. One tool, nothing else.
- The rationale: that tool holds the mapping the memory is supposed to provide.
- Show Table 2 and Figure 1. No-Memory 10.00% ± 0.00; Full 98.33 ± 2.36; Redundant and Condensed 95.00%.
- Compare verbally with the paper: 13.87 → 52.44 → 66.48 → 66.48. "No-Memory lands within three points
  of the paper; the memory conditions sit far higher, which I attribute to the backbone, not to my memory
  blocks."
- Failure mode, show Table 3: the no-memory agent emits `approved`, `reject`, `fulfil`, `review_required`
  — upstream statuses instead of labels. "It was still calling the tools correctly. It just had no
  vocabulary for the answer slot."
- The residual 5%: ORD008, low stock but urgent. `tools.py` has an urgent override; `sop.txt` never
  mentions it. So the SOP under-determines the task, and even a verbatim SOP cannot recover that row.
  This is the benchmark's property, not the agent's.

## 10:30–12:00 — What worked, what did not (1.5 min)

- Worked: importing the authors' loop; the oracle check; the modification as a single-tool ablation;
  keeping an exploratory directory so the failed protocol stayed auditable.
- Did not work: the released protocol cannot discriminate memory configurations; the reported TSR values
  are not multiples of 1/20, so the shipped 20 rows are a sample, not the evaluation set; my memory
  blocks are reconstructions with approximate sizes.
- One number I want to flag: Full-Memory beats Condensed by 3.33 points here, while the paper has Condensed
  ahead of Full by 14 points. There is no evidence for that ordering on 20 rows with a different backbone.

## 12:00–13:30 — Lessons for future users (1.5 min)

Three concrete recommendations:
1. Run the deterministic tool chain against the reference labels before believing any agent score.
   If the chain reproduces every label, the benchmark measures plumbing, not reasoning.
2. Publish prompt text and tool schemas verbatim. One sentence restating the procedure moved every
   configuration to the ceiling in my first iteration.
3. Ship the memory blocks. Reconstructing them from token counts preserves ordering but cannot settle
   whether wording or content drives the effect.

## 13:30–14:00 — Close

- "Reproducibility here was not a pass/fail check. The mechanism reproduces, the number does not, and
  the reason is in the artefact rather than in the agent. That is the finding."
- Point at the repo: five commits — harness, driver, baseline results, modification results, report.

---

### Live demo (optional, 60 s, do it between 7:30 and 8:30)

```bash
cd repro
python3 oracle_check.py                                     # 20/20, free
python3 ga_sop_bench.py --condition condensed_memory --rows 2 --mock   # offline plumbing
```

Do not run a paid call on camera unless you have checked the key is set.
