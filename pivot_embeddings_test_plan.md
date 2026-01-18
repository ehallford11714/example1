# STELLA Pivot Embeddings — Step-by-Step Test Plan

This plan operationalizes tasks 1–45 as a sequential execution checklist to determine whether pivot embeddings improve base reasoning in LLMs. Each step lists required inputs, actions, and concrete outputs. Run all benchmarks with 3–5 seeds unless labeled as a smoke test.

## A. Evaluation Harness & Reproducibility (1–8)

1) **Freeze the baseline configuration**
   - Inputs: model version(s), decoding params, prompt templates, scoring rules.
   - Actions:
     - Create `baseline_config.json` with all frozen settings.
     - Run baseline evaluations for each benchmark once to create the scoreboard.
   - Outputs:
     - `baseline_config.json`
     - `baseline_scoreboard.csv`

2) **Build a single unified evaluation harness**
   - Inputs: benchmark loaders, pre/post-processing, pivot toggle flag.
   - Actions:
     - Implement a single CLI `eval_runner` with a `--pivot` flag.
     - Define `run_manifest` schema (YAML/JSON).
   - Outputs:
     - `eval_runner` CLI
     - `run_manifest.schema.json`

3) **Determinism + multi-seed protocol**
   - Inputs: seed list, batching policy, retry policy, max attempts.
   - Actions:
     - Write `seed_policy.md` covering seed list (3–5), retry logic, batching.
     - Emit per-seed logs for each run.
   - Outputs:
     - `seed_policy.md`
     - `logs/run_<benchmark>_<seed>.jsonl`

4) **Paired statistical testing module**
   - Inputs: per-item correctness for baseline and pivot-on.
   - Actions:
     - Implement `stats_reporter.py` for paired bootstrap and McNemar tests.
     - Generate a sample report for a benchmark.
   - Outputs:
     - `stats_reporter.py`
     - `sample_stats_report.md`

5) **Cost/latency instrumentation**
   - Inputs: model API usage, timing, memory metrics.
   - Actions:
     - Record tokens in/out, wall time per item, peak memory, pivot overhead.
     - Aggregate into summary report.
   - Outputs:
     - `cost_metrics.jsonl`
     - `aggregate_summary.md`

6) **Golden run regression tests (smoke tests)**
   - Inputs: subset of 50–200 items per benchmark.
   - Actions:
     - Create `golden_subset` datasets.
     - Add CI step to compare against expected ranges.
   - Outputs:
     - `golden_subset/`
     - `ci/golden_run_check.yml`

7) **Result schema standardization**
   - Inputs: desired result fields.
   - Actions:
     - Define `results_schema.json`.
     - Add a validator to check output consistency.
   - Outputs:
     - `results_schema.json`
     - `results_validator.py`

8) **Repro bundle packaging**
   - Inputs: configs, commands, hashes, environment details.
   - Actions:
     - Define `run_bundle` structure spec.
     - Package a bundle per run.
   - Outputs:
     - `run_bundle_spec.md`
     - `run_bundle_<run_id>.zip`

## B. Pivot Library Definition & Controls (9–18)

9) **Pivot count sweep (K sweep)**
   - Actions:
     - Run K ∈ {0, 4, 8, 16, 32, 64} on representative benchmarks.
     - Plot accuracy vs K.
   - Outputs:
     - `accuracy_vs_K.png`
     - `k_sweep_report.md`

10) **Placebo control: random pivots**
    - Actions:
      - Generate random pivot vectors or random soft prompts matched in size.
      - Compare against real pivots.
    - Outputs:
      - `placebo_random_report.md`

11) **Placebo control: shuffled pivots**
    - Actions:
      - Shuffle real pivots across tasks.
      - Measure accuracy changes.
    - Outputs:
      - `placebo_shuffled_report.md`

12) **Domain-specific pivot sets**
    - Actions:
      - Build pivot libraries for math, code, science, reading, general knowledge.
      - Evaluate each domain.
    - Outputs:
      - `domain_pivot_library/`
      - `domain_pivot_report.md`

13) **Pivot construction method ablation**
    - Actions:
      - Compare clustering of question embeddings, hidden states, human labels, retrieval-derived.
      - Record best method per benchmark family.
    - Outputs:
      - `pivot_method_comparison.md`

14) **Pivot quality ranking / filtering**
    - Actions:
      - Rank pivots by dev-set lift.
      - Compare top-N vs bottom-N vs mixed.
    - Outputs:
      - `pivot_quality_curve.png`
      - `pivot_quality_report.md`

15) **Pivot compression / quantization test**
    - Actions:
      - Quantize (fp16/int8) and/or PCA-reduce pivots at fixed K.
      - Compare accuracy retention.
    - Outputs:
      - `accuracy_retention_vs_compression.png`
      - `compression_report.md`

16) **Pivot stability across time / versions**
    - Actions:
      - Use pivots trained on one model version and test on another.
    - Outputs:
      - `transfer_matrix.csv`
      - `pivot_transfer_report.md`

17) **Pivot update frequency ablation**
    - Actions:
      - Compare static vs adaptive pivots updated every 10/50/100 queries.
    - Outputs:
      - `adaptation_benefit_vs_drift.md`

18) **Pivot abstain / null option**
    - Actions:
      - Allow router to choose “no pivot” below confidence threshold.
      - Compare to always-on.
    - Outputs:
      - `abstain_report.md`

## C. Injection Mechanism Ablations (19–28)

19) **Prompt-only vs vector injection**
    - Actions:
      - Compare textual prompts vs vector/soft prompt injection.
    - Outputs:
      - `prompt_vs_vector_report.md`

20) **Injection location sweep (layer/site)**
    - Actions:
      - Inject at embedding, early, mid, late, all layers.
    - Outputs:
      - `layer_site_heatmap.png`
      - `injection_site_report.md`

21) **Additive vs gated injection**
    - Actions:
      - Compare additive vs gated injection (fixed vs learned gate).
    - Outputs:
      - `gated_injection_report.md`

22) **Attention-bias vs residual-add**
    - Actions:
      - Compare attention-biasing vs residual addition.
    - Outputs:
      - `mechanism_comparison.md`

23) **Pivot strength sweep (alpha)**
    - Actions:
      - Evaluate α ∈ {0.1, 0.25, 0.5, 1, 2, 4}.
    - Outputs:
      - `alpha_sensitivity.png`
      - `alpha_report.md`

24) **Single pivot vs mixture-of-pivots**
    - Actions:
      - Compare single pivot vs mixture strategies (softmax, top-k, sparsemax).
    - Outputs:
      - `routing_vs_content_report.md`

25) **Late-binding pivots**
    - Actions:
      - Inject pivots after first-pass outline vs early-only vs both.
    - Outputs:
      - `late_binding_report.md`

26) **Router input ablation**
    - Actions:
      - Route using embedding similarity, router mini-model, heuristic tags, retrieval signals.
    - Outputs:
      - `router_accuracy_report.md`

27) **Router error sensitivity**
    - Actions:
      - Inject 10/25/50% routing noise.
    - Outputs:
      - `router_noise_degradation.png`
      - `router_noise_report.md`

28) **Interference tests with competing pivots**
    - Actions:
      - Provide two conflicting pivots; evaluate dominance and gating resolution.
    - Outputs:
      - `conflict_resolution_report.md`

## D. Benchmark Suite Runs (29–36)

29) **Reasoning core suite**
    - Benchmarks: MMLU (or MMLU-Pro), ARC-Challenge, BBH.
    - Outputs:
      - `reasoning_core_report.md`
      - category-level deltas.

30) **Math suite**
    - Benchmarks: GSM8K + MATH (strict exact-match).
    - Outputs:
      - `math_suite_report.md`

31) **Coding suite**
    - Benchmarks: HumanEval and/or MBPP.
    - Outputs:
      - `coding_suite_report.md`

32) **Truthfulness / hallucination suite**
    - Benchmarks: TruthfulQA (or similar).
    - Outputs:
      - `truthfulness_report.md`

33) **Reading comprehension / QA suite**
    - Benchmarks: RACE-like or similar.
    - Outputs:
      - `reading_comprehension_report.md`

34) **Long-context robustness suite**
    - Benchmarks: long-context or distractor-heavy.
    - Outputs:
      - `long_context_report.md`

35) **OOD generalization test**
    - Actions:
      - Build pivots from one benchmark family and test on another.
    - Outputs:
      - `ood_generalization_report.md`

36) **Low-resource / few-shot sensitivity**
    - Actions:
      - Compare zero-shot, few-shot, many-shot with pivots on/off.
    - Outputs:
      - `fewshot_sensitivity_report.md`

## E. Analysis, Diagnostics, and Safety Checks (37–45)

37) **Item-level win/loss attribution**
    - Outputs:
      - `pivot_flip_report.md`

38) **Subpopulation analysis**
    - Outputs:
      - `subgroup_delta_table.csv`
      - `subgroup_report.md`

39) **Calibration analysis**
    - Outputs:
      - `calibration_report.md`

40) **Consistency under paraphrase**
    - Outputs:
      - `paraphrase_consistency_report.md`

41) **Adversarial / distractor stress tests**
    - Outputs:
      - `distractor_robustness_report.md`

42) **Contamination/memorization sanity check**
    - Outputs:
      - `leakage_audit.md`

43) **Token-budget sensitivity**
    - Outputs:
      - `budget_tradeoff.png`
      - `budget_tradeoff_report.md`

44) **Latency/accuracy trade-off analysis**
    - Outputs:
      - `pareto_frontier.png`
      - `latency_accuracy_report.md`

45) **Final ablation matrix & conclusion**
    - Outputs:
      - `final_report.md`
      - `run_manifests/`

## Execution Notes

- Run each task in order, carrying forward artifacts into subsequent steps.
- For each benchmark, use 3–5 seeds and report mean, std, and paired bootstrap CIs. Use McNemar for paired correctness shifts.
- Do not claim improvements without statistical significance and cross-seed consistency.
- Always include placebo controls (random/shuffled pivots) to rule out trivial gains.

