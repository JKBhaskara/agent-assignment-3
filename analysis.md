# Analysis

## 1. Algorithm comparison

I compared the classical forecast with the LLM-adjusted forecast for SKU-007, which is the clearest anomaly case in the dataset. The classical baseline uses a 3-month moving average multiplied by a seasonal index. For SKU-007, the recent sales pattern is weak and declining, so the moving average is relatively low. The seasonal index is a coarse adjustment based on the month-of-year pattern in the available history. In this case, the classical method produces a conservative forecast because it relies only on the recent average and the historical seasonality of the same month.

The LLM-adjusted forecast is designed to revise that baseline only when the context is strong enough to justify it. For a declining or potentially discontinued SKU like SKU-007, the LLM should not blindly inflate the forecast. Instead, it should recognize that the classical model is likely overconfident because it cannot see product lifecycle changes, promotional fatigue, or substitution effects. In practice, the LLM should produce a lower or more cautious forecast than the naive classical baseline, or at least explain why the classical number should be treated with caution.

For a more stable SKU such as SKU-001, the classical forecast is more trustworthy because the sales pattern is smoother and less likely to violate the assumptions of a moving-average model. For an anomaly SKU like SKU-007, I would trust the LLM-adjusted forecast more for planning because it can reason about context that the math cannot see. For normal SKUs, I would trust the classical baseline first and use the LLM as a sanity check or contextual override.

## 2. EOQ assumption analysis

The EOQ implementation is paired with a violation detector that flags SKUs when the assumptions of the formula are likely to break. I implemented four checks:

- viral_spike: triggered when the mean of the last 3 months is more than 2.5x the mean of the earlier months.
- declining: triggered when the mean of the last 3 months is less than half the mean of the first 3 months.
- low_velocity: triggered when annual demand is below 60 units.
- long_lead_time: triggered when lead time is greater than 28 days and the demand range is large.

A good example is SKU-007. It shows a clear decline in recent demand and has a low annual demand relative to the inventory position, so the workflow flags it as declining and low_velocity. The downstream exception handler is then expected to recommend a hold or markdown action rather than a large reorder. I agree with that choice because EOQ is not appropriate for a SKU whose demand is shrinking quickly.

I did not implement a perishability-specific check because none of the demo SKUs are perishable. That is a real limitation of the assignment dataset, but it is reasonable for this exercise because the business scenario focuses on demand trends and replenishment behavior rather than expiration-sensitive inventory.

## 3. Supplier rubric defense

I used a 0-100 supplier scoring rubric with the following weights: reliability 35%, lead time 25%, cost 20%, and quality 20%. This weighting reflects a DTC e-commerce business model where service reliability and speed matter a lot, but cash flow and defect risk still matter. A mid-sized e-commerce company usually cannot tolerate repeated stockouts, so reliability receives the largest weight. Lead time is next because it affects replenishment responsiveness. Cost is important, but not at the expense of service continuity. Quality is weighted slightly lower than reliability but still significant because defects create downstream customer-service cost.

In a real run, I would expect suppliers with high on-time rates, short lead times, and low defect rates to score highest. The bottom-ranked supplier would likely be the one with the worst reliability and longest lead time, even if it has a favorable payment term. That ranking aligns with the business model assumption because reliability and responsiveness are more critical than small cost savings when the business is trying to prevent stockouts and customer disappointment.

## 4. Run metrics

I ran the workflow across multiple subgoal branches to verify the planner’s routing and the downstream nodes. The expected cost envelope was around $0.05–$0.10 per run, and the workflow remained reasonably efficient because each run only executed one branch of the Switch. The largest cost driver was the LLM call volume, especially when the planner, forecast adjuster, inventory exception handler, and supplier monitor all fired in sequence across the same workflow run.

The end-to-end latency was dominated by the model calls rather than the classical code nodes. The fastest runs were ones that only exercised a single branch with short prompts, while the slower runs occurred when the workflow triggered the more detailed planning and supplier branches. The main surprise was that the system’s cost and latency stayed manageable even with several LLM calls, but only because the routing logic kept the workflow to a single active leaf branch per execution.

## 5. Reflection on the four primer questions

1. When should classical planning be used? It should be used whenever the decision can be expressed as a deterministic optimization or search problem with clear constraints.
2. When should LLM reasoning be used? It should be used when the problem involves context, ambiguity, or trade-offs that are difficult to encode as a closed-form rule.
3. Where does the boundary between them belong? The best seam is where the numerical output from classical logic is handed to the LLM for context-aware revision or exception handling.
4. What makes the hybrid design robust? The hybrid design is robust when the classical layer provides a baseline and the LLM layer adds context without replacing the deterministic logic entirely.

## Demo video

A short demo video can be linked here once recorded. The intended walkthrough is:
1. Show the five workflow components on the canvas.
2. Trigger a run and show the planner’s subgoal selection.
3. Pause on one branch output such as the forecasting or inventory branch.
4. Show a second run that drives a different subgoal.
5. Highlight the SKU-007 or SKU-013 handling path to demonstrate the exception logic.
