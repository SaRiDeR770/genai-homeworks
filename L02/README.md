# HW2 Support Message Classifier: Zero-Shot vs Few-Shot

## Goal

Check how examples (few-shot) change the answer of an LLM classifier

Two things are checked for every answer: correct category and pure JSON format without extra text

The prompt was run on purpose on different Claude models to see if it works the same on each of them

## Prompt

**Step 1 Zero-Shot (no examples)**

```
<role>You are an e-commerce support triage classifier.</role>
<task>Determine category: DELIVERY, PAYMENT or RETURN.</task>
<rules>Respond ONLY with pure JSON: {"category": "..."}. No preamble.</rules>
Customer: "<test phrase>"
```

**Step 2 Few-Shot (3 balanced examples, one per class)**

```
<role>You are an e-commerce support triage classifier.</role>
<task>Determine category: DELIVERY, PAYMENT or RETURN.</task>
<rules>Respond ONLY with pure JSON: {"category": "..."}. No preamble.</rules>
<examples>
Customer: "Where is my driver?" -> {"category": "DELIVERY"}
Customer: "Double billing on card" -> {"category": "PAYMENT"}
Customer: "Teapot arrived broken" -> {"category": "RETURN"}
</examples>
Customer: "<test phrase>"
```

## Test inputs

| # | Phrase | Expected |
|---|---|---|
| 1 | Hello! Order #412 is delayed by 30 minutes, where is the courier? | DELIVERY |
| 2 | I noticed two identical charges for the same transaction on my credit card | PAYMENT |
| 3 | The package was damaged upon arrival, how do I initiate a return and refund? | RETURN |

## Results

Chat: Claude web chat

| # | Zero-Shot (Opus 5.5) | Few-Shot | Few-Shot model | Category | Pure JSON |
|---|---|---|---|---|---|
| 1 | {"category": "DELIVERY"} | {"category": "DELIVERY"} | Fable 5 | correct | yes |
| 2 | {"category": "PAYMENT"} | {"category": "PAYMENT"} | Opus 4.7 | correct | yes |
| 3 | {"category": "RETURN"} | {"category": "RETURN"} | Fable 5 | correct | yes |

## Observations

1. All 6 answers have the correct category
2. All 6 answers are pure JSON, no preamble, no markdown
3. The prompt works the same on 3 different models: Claude Opus 5.5, Claude Fable 5, Claude Opus 4.7
4. Zero-Shot was already clean. Strong models follow the XML rules even without examples
5. Phrase 3 is the hardest case: it mentions both return and refund, so PAYMENT was also possible. Every model still chose RETURN

## Conclusion

On strong models clear XML rules are enough for this simple task, and the result does not depend on the model

Few-Shot is still useful for cheaper models and edge cases, and it keeps the format stable over many calls

Limitations: one run per case, only strong Claude models were tested

## Next step

In L03 this prompt goes into Python. The same 3 phrases become pytest cases: check that the answer is valid JSON and the category equals the expected one
