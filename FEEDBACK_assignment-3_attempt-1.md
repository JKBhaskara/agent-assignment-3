## Grade: 81 / 100

**Assignment:** E-Commerce Supply Chain Manager (n8n)  
**Attempt:** 1 of 2  ·  **Graded:** 2026-07-18  ·  Commit `9e659a5`

### Score breakdown
| Criterion | Max | Earned | Notes |
|-----------|-----|--------|-------|
| mp_1 | 8 | 2 | Master Planner system prompt is byte-identical to the untouched scaffold: the required output shape incl. next_subgoal is only present as the original '# TODO' comment block, and the meta-instruction 'Add explicit instructions to ALWAYS reply with raw JSON only' was never fulfilled (the model is not actually told to emit raw JSON for the downstream JSON.parse). Schema field is nominally visible so the Switch could route, but no student work was done here. No verifiable demo. Low partial. (`workflows/supply-chain-manager-starter.json (Master Planner Agent node) vs rubrics/assignment-3/reference/workflows/supply-chain-manager-starter.json`) |
| mp_2 | 10 | 0 | MP-2 requires 1-2 worked HTN decomposition examples in the system prompt. None were added — the '# TODO [medium] ... Provide 1-2 worked examples' comment remains unmodified. analysis.md discusses classical-vs-LLM boundaries but contains no HTN decomposition example. No credit. (`workflows/supply-chain-manager-starter.json (Master Planner Agent node); analysis.md`) |
| df_1 | 6 | 6 | movingAverage() correctly averages the trailing 3-month window via slice(-3), reduces to a sum, and divides by recent.length (so <3 months averages what exists); empty series returns 0. Correct. (`custom-nodes/demand-forecast.js:66-72`) |
| df_2 | 8 | 8 | seasonalIndex() filters rows to the target month-of-year, computes monthMean/overallMean, and guards empty series and zero overall mean (returns 1.0). Matches the specified month-avg / overall-avg definition. Correct. (`custom-nodes/demand-forecast.js:92-106`) |
| df_3 | 4 | 4 | Fully written prompt takes the numeric forecast row in and emits schema-conformant JSON {sku, revised_forecast, delta_pct, reasoning, context_signals_used}, with sensible override thresholds (<=10% light evidence, larger needs justification, non-negative). Complete. (`workflows/supply-chain-manager-starter.json (Forecast Context Adjuster node)`) |
| eoq_1 | 8 | 8 | eoq() implements Q* = round(sqrt(2*D*S/H)) and guards D<=0, H<=0, S<=0 -> 0. Closed-form correct. (`custom-nodes/eoq-optimizer.js:68-71`) |
| eoq_2 | 10 | 8 | detectViolations() correctly catches the viral spike (recent-3 mean > 2.5x prior mean) and the dying/declining SKU (last-3 mean < 0.5x first-3 mean), plus low_velocity and long_lead_time. Both boundary cases are caught via demand trend and analysis.md defends the choices. Held below full: detection uses only sales-trend signals and never cross-references inventory position (on_hand vs reorder_point), which is the nuance the rubric flags for viral-below-reorder / dying-far-above-reorder. (`custom-nodes/eoq-optimizer.js:103-139; analysis.md (section 2)`) |
| eoq_3 | 4 | 4 | Prompt emits a per-flag action with an explicit rule for each violation (viral_spike->emergency reorder > EOQ, declining->markdown/hold, low_velocity->small fixed qty, long_lead_time->conservative/safety stock) and a JSON schema {sku, recommended_action, recommended_qty, reasoning}. Complete. (`workflows/supply-chain-manager-starter.json (Inventory Exception Handler node)`) |
| sp_1 | 14 | 14 | Prompt scores suppliers on all four dimensions with explicit weights (reliability 0.35, lead time 0.25, cost 0.20, quality 0.20) on a 0-100 rubric, with a score_breakdown per dimension and a tier. analysis.md defends the weighting with a coherent DTC-e-commerce business rationale. Full credit. (`workflows/supply-chain-manager-starter.json (Supplier Performance Monitor node); analysis.md (section 3)`) |
| lg_1 | 6 | 6 | Prompt reasons over hard constraints first (deadline, weight, perishability, region match) then soft preferences (cost, time slack, reliability), with a tie-break favoring more time slack. Well specified. (`workflows/supply-chain-manager-starter.json (Logistics Coordinator (LLM) node)`) |
| lg_2 | 10 | 10 | pickCheapestFeasible() filters options against all five hard constraints (origin+dest region, transit<=deadline, max_weight>=weight, perishable support) then reduces to the minimum total_cost_usd; returns null when infeasible. Correct greedy planner. (`custom-nodes/classical-logistics.js:60-77`) |
| lg_3 | 2 | 2 | Prompt instructs use_classical_fallback: true only when the request is purely numeric with no soft-constraint judgment, otherwise false — enabling the downstream IF node to route to the classical planner. Present. (`workflows/supply-chain-manager-starter.json (Logistics Coordinator (LLM) node)`) |
| fn_1 | 10 | 9 | Student completed this beyond the reference stub (which returned executed_subgoal: 'TODO'): detects the executed_subgoal from item fields, builds a business-readable key_findings array, and sets next_recommended_subgoal via a rotation map to close the HTN loop. Strong, complete code. Held one point below full because the required end-to-end demo touching >=3 branches is not provided (analysis.md only lists an intended, unrecorded walkthrough). (`workflows/supply-chain-manager-starter.json (Final Output node)`) |
| Integrity deduction | — | 0 | Provided files unmodified |
| **Total** | **100** | **81** | |

### What went well
- All four classical algorithms are correct and defensively guarded: movingAverage (demand-forecast.js:66), seasonalIndex (demand-forecast.js:92), eoq (eoq-optimizer.js:68), and pickCheapestFeasible (classical-logistics.js:60).
- Supplier Performance Monitor is a standout: explicit 4-dimension weights (reliability 0.35 / lead time 0.25 / cost 0.20 / quality 0.20), a per-dimension score_breakdown and tiering, backed by a business-grounded defense in analysis.md section 3.
- Final Output was genuinely implemented (not left as the 'TODO' stub): it infers the executed subgoal, produces human-readable key_findings, and feeds next_recommended_subgoal back for an iterative HTN loop.
- The four completed LLM prompts (forecast adjuster, exception handler, supplier monitor, logistics coordinator) all emit clean, well-specified JSON schemas with sensible per-flag / hard-vs-soft-constraint rules.

### What to improve (actionable)
- The Master Planner Agent node was left as the unmodified scaffold — the required output schema and the 'reply with raw JSON only' directive still exist only as '# TODO' comments, so MP-1 lost most of its credit and MP-2 (worked HTN examples) received none. Rewrite the system prompt so the model itself is directed to emit the {plan_id, reasoning, subgoals, next_subgoal} JSON and add 1-2 ordered decomposition examples.
- detectViolations() keys entirely off sales-trend signals; it never consults the inventory position (on_hand vs reorder_point). Combining demand trend with inventory state would more precisely separate 'viral spike below reorder point' from 'dying SKU far above reorder point', the exact boundary the rubric targets.
- No demo video was recorded — analysis.md only sketches the intended walkthrough — leaving the end-to-end >=3-branch behavior (needed for full MP-1 and FN-1 credit) unverifiable.
- The 'Run metrics' section of analysis.md is described in generic terms (cost 'around $0.05-$0.10', latency 'dominated by model calls') without actual observed numbers from a real execution; concrete measured figures would strengthen it.

### Automated checks
- ✅ All required files implemented
- ✅ Provided files unmodified
- ✅ 0/0 output artifacts committed
- ✅ Reflection 1010 words

### Resubmission
You may resubmit **once**. Push fixes to this repo, then notify the instructor; we'll re-grade as **Attempt 2 (final)**. This is attempt 1 of 2.

---
*Graded automatically with Claude Code against the course rubric. Questions → contact the instructor.*


---
<sub>🔎 **Autograder record** — attempt 1 of 2 · graded at commit `9e659a5` · delivered 2026-07-18T20:42:39Z. Commits pushed to `main` after this timestamp are treated as a resubmission.</sub>
