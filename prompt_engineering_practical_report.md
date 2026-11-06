# Prompt Engineering Practical Assignment

## Zero-Shot vs Few-Shot Prompting, Step-by-Step Prompting, and Structured Output

**Prepared by:** Sudharshan S  
**Course:** Generative AI and Agentic AI  
**Date of experiment record:** 6 November 2026  
**Model used for the recorded examples:** ChatGPT GPT-6  
**Testing environment:** ChatGPT conversation interface  
**API / JSON mode:** Not used

---

## 1. Introduction

Prompt engineering is the process of designing clear instructions, relevant context, examples, and output requirements so that a large language model (LLM) is more likely to produce useful and consistent responses.

This practical assignment demonstrates three prompt-engineering techniques:

1. **Zero-shot and few-shot prompting:** Compare classification with and without examples.
2. **Step-by-step prompting:** Compare a direct answer with a response that shows the calculations for a multi-step problem.
3. **Structured output prompting:** Extract information from natural-language messages and return it in a consistent JSON structure.

The examples in this report were recorded in a ChatGPT conversation. They are a small practical demonstration, not a controlled benchmark. Temperature, seed, and other generation settings were not controlled. The report does not claim that API-level JSON mode or schema-constrained output was used.

**Reporting note:** The prompts and outputs below are retained as recorded. The five JSON objects were checked with a JSON parser and basic key, type, and date-format checks while preparing this report. If you repeat the experiments and obtain different outputs, preserve your own exact outputs and update the analysis accordingly.

---

## 2. Experiment Setup

### 2.1 Tasks and test data

| Experiment | Task | Test data |
|---|---|---|
| Zero-shot vs few-shot | Classify customer reviews as Positive, Negative, or Neutral | 5 reviews |
| Step-by-step prompting | Calculate profit or loss from a multi-step sales problem | 1 word problem, 2 prompts |
| Structured output | Extract name, date, and amount into JSON | 5 payment messages |

### 2.2 Evaluation criteria

- **Classification:** Is the label reasonable under the stated classification rule?
- **Calculation:** Is the final answer correct, and are the intermediate calculations correct?
- **JSON:** Can the response be parsed as JSON? Does it contain exactly the required keys and acceptable value types and formats?

---

# 3. Experiment 1: Zero-Shot vs Few-Shot Prompting

## 3.1 Objective

To compare whether providing examples in a prompt changes the results of customer-review sentiment classification.

### Zero-shot prompting

Zero-shot prompting asks the model to perform a task without providing examples of the desired input-output pattern.

### Few-shot prompting

Few-shot prompting provides a small number of examples inside the prompt. These examples demonstrate the pattern the model should follow. They guide the response without necessarily retraining or changing the model's parameters.

## 3.2 Test dataset

The same five reviews are used for both approaches.

| Test ID | Customer review | Expected label under the experiment's rule |
|---|---|---|
| S1 | The product is excellent and arrived earlier than expected. | Positive |
| S2 | The item arrived broken and customer support did not respond. | Negative |
| S3 | The package arrived yesterday. | Neutral |
| S4 | The delivery was late, but the product quality is excellent. | Positive |
| S5 | It works as described, but nothing about it is special. | Neutral |

**Classification rule for S4:** The review contains both a negative delivery experience and a positive product-quality statement. Since the available labels do not include Mixed, the prompt asks the model to choose the dominant overall sentiment. For this test, the expected label is Positive because the review explicitly praises the product quality. This rule is stated before comparing the outputs.

## 3.3 Prompt A: Zero-shot

```text
Classify each customer review as Positive, Negative, or Neutral.

Use the overall sentiment of the review. If the review contains both positive
and negative statements, choose the dominant overall sentiment.

Return only the label for each review, using the test ID.

S1: The product is excellent and arrived earlier than expected.
S2: The item arrived broken and customer support did not respond.
S3: The package arrived yesterday.
S4: The delivery was late, but the product quality is excellent.
S5: It works as described, but nothing about it is special.
```

### Recorded output: Zero-shot

```text
S1: Positive
S2: Negative
S3: Neutral
S4: Positive
S5: Neutral
```

## 3.4 Prompt B: Few-shot

This prompt includes three examples before the test reviews.

```text
You are a customer-review sentiment classifier.

Classify each review as Positive, Negative, or Neutral.
Use the overall sentiment. If both positive and negative statements appear,
choose the dominant overall sentiment.

Follow these examples:

Example 1
Review: "The headphones sound amazing. I am very happy with the purchase."
Label: Positive

Example 2
Review: "The package was damaged and the product stopped working."
Label: Negative

Example 3
Review: "The order was placed on Monday and delivered on Wednesday."
Label: Neutral

Now classify these reviews:

S1: The product is excellent and arrived earlier than expected.
S2: The item arrived broken and customer support did not respond.
S3: The package arrived yesterday.
S4: The delivery was late, but the product quality is excellent.
S5: It works as described, but nothing about it is special.

Return only the test ID and its label.
```

### Recorded output: Few-shot

```text
S1: Positive
S2: Negative
S3: Neutral
S4: Positive
S5: Neutral
```

## 3.5 Comparison and analysis

| Test ID | Expected label | Zero-shot output | Few-shot output |
|---|---|---|---|
| S1 | Positive | Positive | Positive |
| S2 | Negative | Negative | Negative |
| S3 | Neutral | Neutral | Neutral |
| S4 | Positive | Positive | Positive |
| S5 | Neutral | Neutral | Neutral |

**Observed result:** Both prompts produced the same five labels. Under the stated expected labels, both approaches achieved 5/5 correct classifications in this small test.

**Analysis:** The zero-shot prompt already included clear label options and a rule for mixed-sentiment reviews. The three examples in the few-shot prompt reinforced the pattern, but did not change the output. This shows that examples do not automatically improve results when the instructions are already clear and the test cases are straightforward.

**Limitation:** The dataset is small and contains mostly clear examples. A stronger follow-up experiment could include sarcasm, unclear wording, and reviews with conflicting opinions. Those additional cases should be tested and reported separately rather than counted as results in this experiment.

---

# 4. Experiment 2: Step-by-Step Prompting

## 4.1 Objective

To compare a direct-answer prompt with a prompt that requests a structured, step-by-step calculation for a multi-step profit-and-loss problem.

### What is step-by-step prompting?

Step-by-step prompting asks the model to present the intermediate calculations or stages needed to solve a problem. In this report, the recorded explanation is the visible calculation returned by the model. It is not a record of private internal reasoning.

## 4.2 Problem

A shopkeeper buys 20 notebooks for ₹40 each. He sells 15 notebooks for ₹60 each and the remaining 5 notebooks for ₹30 each.

Calculate the total profit or loss.

## 4.3 Prompt A: Direct prompting

```text
A shopkeeper buys 20 notebooks for ₹40 each. He sells 15 notebooks for ₹60
each and the remaining 5 notebooks for ₹30 each.

Calculate the total profit or loss. Give only the final answer.
```

### Recorded output: Direct prompt

```text
The shopkeeper makes a profit of ₹250.
```

## 4.4 Prompt B: Step-by-step prompting

```text
A shopkeeper buys 20 notebooks for ₹40 each. He sells 15 notebooks for ₹60
each and the remaining 5 notebooks for ₹30 each.

Solve this problem step by step. Show the calculation for:
1. Total cost price
2. Selling price of the first 15 notebooks
3. Selling price of the remaining 5 notebooks
4. Total selling price
5. Final profit or loss

Present the final answer clearly.
```

### Recorded output: Step-by-step prompt

```text
Step 1: Calculate the total cost price.
20 × ₹40 = ₹800

Step 2: Calculate the selling price of the first 15 notebooks.
15 × ₹60 = ₹900

Step 3: Calculate the selling price of the remaining 5 notebooks.
5 × ₹30 = ₹150

Step 4: Calculate the total selling price.
₹900 + ₹150 = ₹1,050

Step 5: Calculate the profit or loss.
Profit = Total Selling Price − Total Cost Price
Profit = ₹1,050 − ₹800 = ₹250

Final Answer: The shopkeeper makes a profit of ₹250.
```

## 4.5 Independent verification

- Total cost price = 20 × ₹40 = ₹800
- Selling price of the first 15 notebooks = 15 × ₹60 = ₹900
- Selling price of the remaining 5 notebooks = 5 × ₹30 = ₹150
- Total selling price = ₹900 + ₹150 = ₹1,050
- Profit = Total selling price − Total cost price = ₹1,050 − ₹800 = ₹250

**Verified result: ₹250 profit.**

## 4.6 Comparison and analysis

| Criterion | Direct prompt | Step-by-step prompt |
|---|---|---|
| Final answer | ₹250 profit | ₹250 profit |
| Intermediate calculations shown | No | Yes |
| Final answer independently verified | Correct | Correct |
| Ease of checking the calculation | Lower | Higher |

**Observed result:** Both prompts returned the correct final answer. Step-by-step prompting did not improve final-answer accuracy in this example because the direct prompt was already answered correctly.

**Analysis:** The main benefit was transparency. The detailed response exposed each calculation, making it easier for a person to check the result and locate a possible arithmetic error. This single example does not prove that step-by-step prompting always improves accuracy. More difficult problems and repeated tests would be needed to measure that effect.

---

# 5. Experiment 3: Structured Output Prompting

## 5.1 Objective

To extract payment details from natural-language messages and return them in a consistent JSON structure across five inputs.

### What is structured output?

Structured output means returning information in a predefined format instead of a free-form paragraph. JSON (JavaScript Object Notation) is commonly used to exchange structured data between applications and APIs.

## 5.2 Define the required structure

Each response must contain exactly three fields.

| Field | Expected type | Requirement |
|---|---|---|
| `name` | String or null | Person associated with the payment |
| `date` | String or null | Date in `YYYY-MM-DD` format |
| `amount` | Number or null | Numeric amount without a currency symbol |

### JSON Schema

```json
{
  "type": "object",
  "properties": {
    "name": {
      "type": ["string", "null"]
    },
    "date": {
      "type": ["string", "null"],
      "pattern": "^\\d{4}-\\d{2}-\\d{2}$"
    },
    "amount": {
      "type": ["number", "null"]
    }
  },
  "required": ["name", "date", "amount"],
  "additionalProperties": false
}
```

The schema defines the required keys and value types. The date pattern checks the expected text format, but a separate date check is still needed to reject impossible dates such as 2026-02-30.

## 5.3 Prompt used

```text
You are an information extraction assistant.

Extract the payment details from the input message and return one valid JSON
object with exactly these three keys:

{
  "name": "string or null",
  "date": "YYYY-MM-DD string or null",
  "amount": "number or null"
}

Rules:
1. Return JSON only. Do not add explanations or Markdown code fences.
2. Include all three keys in every response.
3. Use the date format YYYY-MM-DD.
4. Return the amount as a number, without currency symbols or commas.
5. If a value is missing or cannot be determined from the message, use null.
6. Do not invent missing information.
7. Do not add any extra keys.

Input:
[INSERT PAYMENT MESSAGE HERE]
```

## 5.4 Test inputs and recorded outputs

### Test 1

**Input**

```text
Rahul transferred Rs. 2500 on 5 November 2026.
```

**Recorded JSON output**

```json
{
  "name": "Rahul",
  "date": "2026-11-05",
  "amount": 2500
}
```

### Test 2

**Input**

```text
Priya paid ₹1,500 on 12 October 2026 for the service.
```

**Recorded JSON output**

```json
{
  "name": "Priya",
  "date": "2026-10-12",
  "amount": 1500
}
```

### Test 3

**Input**

```text
On 20 September 2026, Arjun received Rs. 5,000 from a customer.
```

**Recorded JSON output**

```json
{
  "name": "Arjun",
  "date": "2026-09-20",
  "amount": 5000
}
```

### Test 4

**Input**

```text
Sneha made a payment of 750 rupees yesterday.
```

**Recorded JSON output**

```json
{
  "name": "Sneha",
  "date": "2026-11-05",
  "amount": 750
}
```

**Date interpretation:** The experiment record is dated 6 November 2026, so "yesterday" is interpreted as 5 November 2026. For repeatable testing, a production prompt should explicitly supply the reference date rather than relying on the model to infer the current date.

### Test 5

**Input**

```text
Payment received from Kiran.
```

**Recorded JSON output**

```json
{
  "name": "Kiran",
  "date": null,
  "amount": null
}
```

The name is available, but the date and amount are missing. The model uses `null` rather than inventing values.

## 5.5 JSON validation results

The five recorded JSON objects were parsed and checked for:
- Valid JSON syntax.
- Exactly the three required keys.
- Acceptable value types for `name`, `date`, and `amount`.
- The required date text format when a date is present.

| Test | JSON parsing | Required keys | Value types and date format | Result |
|---|---|---|---|---|
| 1 | Pass | Pass | Pass | Pass |
| 2 | Pass | Pass | Pass | Pass |
| 3 | Pass | Pass | Pass | Pass |
| 4 | Pass | Pass | Pass | Pass |
| 5 | Pass | Pass | Pass | Pass |
| **Total** | **5/5** | **5/5** | **5/5** | **5/5** |

**Success rate**

Success rate = (Passing outputs ÷ Total test inputs) × 100

Success rate = (5 ÷ 5) × 100 = **100%**

**Important qualification:** This is the result for these five recorded examples only. It does not guarantee 100% accuracy on unseen messages. The checks confirm JSON syntax and basic structure, not that every extracted fact is semantically correct. The relative-date test also depends on the stated reference date.

## 5.6 What helped produce consistent JSON?

1. **Explicit keys:** The prompt named the exact fields required.
2. **Value types:** It specified strings, numbers, and `null`.
3. **Required fields:** It instructed the model to include all three keys every time.
4. **Missing-value rule:** It instructed the model to use `null` instead of guessing.
5. **Date normalization:** It specified one date format.
6. **No-extra-fields rule:** It prohibited additional keys.
7. **Output example:** It showed the expected JSON shape directly.

### JSON mode and structured-output API features

The five examples were generated in the ChatGPT conversation interface. They were not generated using an API's JSON mode or schema-constrained output feature.

In a production application, a supported structured-output feature can enforce a schema more strongly than prompt instructions alone. The application should still validate the returned data, especially semantic details such as whether a date is a real calendar date and whether an amount was extracted from the correct part of the message.

---

# 6. Overall Comparison

| Technique | Main purpose | Result in this experiment | Main lesson |
|---|---|---|---|
| Zero-shot | Perform a task without examples | 5/5 expected labels | Clear instructions can be sufficient for simple tasks. |
| Few-shot | Show examples of the expected pattern | Same five labels as zero-shot | Examples did not change the result on this small dataset. |
| Step-by-step prompting | Show intermediate calculations | Same correct final answer, with more detail | Visible calculations make results easier to audit. |
| Structured output | Return data in a fixed format | 5/5 recorded JSON objects passed basic checks | Explicit keys, types, and missing-value rules improve structural consistency. |

---

# 7. Limitations and Lessons Learned

1. **Small sample size:** Five reviews and five payment messages are suitable for a basic demonstration, but not enough to establish broad reliability.
2. **No controlled generation settings:** Temperature, seed, and other generation settings were not controlled in the ChatGPT interface.
3. **No API schema enforcement:** The JSON experiment used prompt instructions, not an API-level constrained-output feature.
4. **Valid JSON does not guarantee correct meaning:** A response can be syntactically valid while containing an incorrect name, date, or amount. Semantic validation is also necessary.
5. **Step-by-step explanations do not guarantee correctness:** Intermediate calculations must still be checked.
6. **Few-shot prompting is not automatically better:** Examples are most useful when they clarify a pattern that the instructions alone do not communicate clearly.
7. **Relative dates need context:** Terms such as "yesterday" should be interpreted against an explicitly supplied reference date for repeatable results.

---

# 8. Conclusion

This practical assignment demonstrated three important prompt-engineering techniques.

Zero-shot prompting showed that a model can perform a clearly defined classification task without examples. Few-shot prompting supplied examples of the expected pattern, although it did not change the results on this particular dataset. Step-by-step prompting made a multi-step calculation easier to inspect, even though both prompts produced the same correct final answer. Structured output prompting returned consistent JSON objects for five payment messages, including one message with missing information.

The main lesson is that prompt engineering is not simply about writing longer prompts. It is about selecting suitable instructions, examples, context, and output constraints for the task. Model responses should be tested and validated rather than assumed to be reliable.

---

# 9. Final Submission Checklist

- [ ] Confirm that your name, course, and date are correct.
- [ ] Confirm that the model name and testing environment match the experiments you actually performed.
- [ ] Preserve the exact prompts and raw outputs from your own experiment runs.
- [ ] Use the same test inputs when comparing zero-shot and few-shot prompting.
- [ ] Check the profit calculation independently.
- [ ] Validate JSON using a parser and check keys, types, and date values.
- [ ] Keep failures in the report and calculate the success rate from actual results.
- [ ] Do not claim API JSON mode or schema enforcement unless you actually used it.
- [ ] Save the Markdown file in your public GitHub repository or another location accessible to the evaluator.
- [ ] Submit the shareable link to the report.

---

# 10. References for Further Learning

- OpenAI Prompt Engineering Guide: https://platform.openai.com/docs/guides/prompt-engineering
- OpenAI Structured Outputs Guide: https://platform.openai.com/docs/guides/structured-outputs
- JSON official website: https://www.json.org/json-en.html
- JSON Schema documentation: https://json-schema.org/learn/getting-started-step-by-step

---

**End of Report**
