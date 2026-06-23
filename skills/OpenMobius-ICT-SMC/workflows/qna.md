# Workflow: Concept Q&A

For text-based ICT/SMC questions when **no chart is attached**.

## When this workflow applies

- User asks "what is X" / "how to identify Y" / "how to enter Z"
- Topics on ICT / SMC / Wyckoff / VSA / Order Flow / Price Action
- User mentions specific terms (FVG / OB / Killzone / CISD / OTE / Liquidity / Inducement / Breaker / IFVG, etc.)
- User asks about trading strategy, market structure, risk management

## When NOT to use

- User attached a chart → switch to `analyze.md`
- General greetings / unrelated topics → no skill needed

## Special case: data-source / freshness questions

If the user's question is about the data source / pipeline / freshness
/ upstream vendors — e.g. "数据从哪来 / 数据源 / data source / where is
this data from / 你用什么数据 / 实时吗 / 怎么取的数据" — **do NOT call
`kb_retrieve.py`**. Instead, respond using the canonical data-source
disclosure from `SKILL.body.md` § "Data source disclosure".

If the conversation has already produced an API response in this turn
(klines / indicators / chart), substitute its `freshness` block and
`exchange`/`market`/`symbol` fields into the template.

If no API call has been made yet, answer:

> 本对话尚未发起行情数据请求；如果你接下来问某个资产的行情，数据将通过
> **Mobius Quant API** (`api.mobiusquant.ai`) 获取。具体上游来源（Mobius
> 内部接入哪些交易所 / 数据供应商）skill 无法核实，详情见
> [mobiusquant.ai](https://www.mobiusquant.ai/)。

Then stop. Do NOT add SMC analysis or chart in response to a data-source
question alone.

## Steps

### Step 1: Retrieve relevant cards

Extract the core concepts from the user's question (prefer English technical terms), then run:

```bash
.venv/bin/python scripts/kb_retrieve.py "<query>" --top-k 5
```

Variants:

```bash
# Case-only retrieval
kb_retrieve.py "BTC reversal liquidity sweep" --type case --top-k 5

# School filter
kb_retrieve.py "smart money concepts market structure" --school ICT --top-k 5
```

### Step 2: Synthesize the answer

The tool returns markdown-formatted cards with: `Term` / `Aliases` / `School` / `Definition` / `Identification rules` / `Trading implication` / `Common mistakes` / `Related concepts`.

**Strict requirements**:

1. **Anchor every claim to the knowledge base** — cite specific rule numbers
2. **No vague generalities** — give concrete identification steps, entry points, stop placements
3. **If retrieval is insufficient** — say "knowledge base does not explicitly cover X, but concept Y may be relevant"
4. **Link related concepts** — when discussing FVG, mention PD Array / OTE / CISD relations
5. **Match user's language** — Chinese question → Chinese answer; English → English (technical terms stay English per shared rules)

## Query optimization tips

- **Use English technical terms** for best retrieval (knowledge base is English):
  - "如何识别市场反转" → retrieve `"market structure shift trend reversal"`
  - "止损放哪" → retrieve `"stop loss placement swing point"`
- Join multiple concepts with spaces to let vector search match related clusters
- For case queries, use concrete features: `"4H FVG liquidity sweep entry"`

## Examples

### Example 1 — Concept question

User: "什么是 Fair Value Gap，怎么交易"

Action:
```bash
kb_retrieve.py "Fair Value Gap how to trade entry" --top-k 5
```

Response (in Chinese, technical terms in English):
- Precise FVG definition (three-candle non-overlap pattern)
- 3 identification rules (specific bullish/bearish criteria)
- Entry strategy (wait for CISD confirmation + entry at OTE 0.62-0.79 + stop below swept low)
- Common mistakes (5 concrete pitfalls, citing the knowledge base)

### Example 2 — School overview

User: "ICT 是什么流派，它的核心方法论是什么"

Action:
```bash
kb_retrieve.py "ICT methodology smart money concepts" --school ICT --top-k 8
```

Response:
- Positioning of ICT (Inner Circle Trader)
- 4-5 core tools (OB / FVG / Liquidity Sweep / Killzone)
- Typical workflow: HTF bias → PD Array → CISD → entry → stops/targets
- Common misapplications

### Example 3 — Case query

User: "找一个 BTC 在 FVG 反转的真实案例"

Action:
```bash
kb_retrieve.py "BTC bitcoin Fair Value Gap reversal entry" --type case --top-k 3
```

Response: Extract 1-2 most relevant cases' `analysis_steps` + `lessons`.

## Output format

Standard prose answer (no JSON, no special structure). The 5-section format with auto-annotation is for chart analysis only — Q&A is free-form structured prose with clear sections like:

```markdown
## 定义 / Definition
<from card>

## 识别规则 / Identification Rules
1. <rule 1>
2. <rule 2>
...

## 交易意义 / Trading Implication
<from card>

## 常见错误 / Common Mistakes
- <mistake 1>
- <mistake 2>

## 相关概念 / Related
<linked terms>
```

Adapt section headers to fit the question type (e.g. for a strategy question, lead with "Strategy" rather than "Definition").
