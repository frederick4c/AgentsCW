# Writing Style Context

This file describes Fred's report-writing style from an unrelated medical-imaging report. Use it for style only; do not copy subject matter, claims, or phrasing.

## Overall Voice

- Write in a clear, direct academic style.
- Prefer plain explanations over ornate prose.
- Keep paragraphs compact and purposeful.
- Sound analytical rather than dramatic.
- Use British spelling where natural: `normalised`, `artefact`, `optimisation`, `modelling`.
- Use first-person plural sparingly for observed results, e.g. "we can see", but default to impersonal academic phrasing.
- Avoid rhetorical flourishes, jokes, and over-qualified hedging.

## Typical Paragraph Shape

Fred's paragraphs often follow this pattern:

1. State the method, comparison, or result.
2. Refer to the figure/table or metric.
3. Explain the qualitative trend.
4. Name the tradeoff or practical consequence.

For this report, a good paragraph should therefore move quickly from evidence to interpretation:

- what was run or inspected;
- what the evidence shows;
- why it happened;
- why it matters for GRPO, adaptive planning, or the coursework requirement.

## Sentence Tendencies

Useful sentence patterns:

- "In contrast, ..."
- "This suggests that ..."
- "This comes at the cost of ..."
- "Overall, ..."
- "The main tradeoff is ..."
- "This is more effective when ..."
- "These results show that ..."

Avoid overusing the same connector in consecutive sentences. The style is steady, but should not become repetitive.

## Evidence Style

- Put figures and tables to work; reference them close to the claim they support.
- Give concrete numbers when available.
- Combine qualitative and quantitative evidence rather than presenting either alone.
- Prefer simple comparisons: better/worse, more robust/less robust, sharper/smoother, stable/collapsed.
- When a result is negative, state it plainly and explain the likely mechanism.

For Part I, this means:

- use exact accuracies, format rates, empty-response counts, KL behaviour, and checkpoint steps;
- explain the tradeoff between reward optimisation, format compliance, numerical correctness, and stability;
- do not let reward curves substitute for external evaluation.

For Part II, this means:

- tie each critique to a concrete implementation detail;
- state the failure mode, why it matters, and what the proposed change trades off.

## Comparison Style

Fred often compares methods by assigning each a context where it works best. Use this balanced structure:

- method A is better under one condition;
- method B is better under another;
- neither is universally better;
- the important issue is the tradeoff.

For this report, avoid pretending a variant is successful just because it was principled. A strong Fred-style discussion can say that RLOO was theoretically motivated but failed empirically, then diagnose why.

## Conclusions

Conclusions should be short and integrative:

- summarise the main result;
- connect it to the mechanism;
- mention the practical implication.

Avoid ending sections with vague future work. If mentioning an improvement, make it concrete and name its cost.

## LaTeX Tendencies

- Use `\Cref{...}` for figure/table references when available.
- Keep captions descriptive, not decorative.
- Use compact tables with direct metric labels.
- Avoid long bullet lists in the final report unless page economy makes them clearly better.
- In prose, define acronyms once and then use them consistently.
