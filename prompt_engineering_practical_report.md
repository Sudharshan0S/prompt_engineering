# Prompt Engineering Practical Assignment

## Zero-Shot vs Few-Shot Prompting, Chain-of-Thought Prompting, and Structured Output

**Prepared by:** \[Your Name\]\
**Course / Batch:** \[Course or Batch\]\
**Date of experiment record:** 6 November 2026\
**Model used for the recorded examples:** ChatGPT GPT-6\
**Testing environment:** ChatGPT conversation interface\
**API / JSON mode:** Not used for the examples in this report

------------------------------------------------------------------------

## 1. Introduction

Prompt engineering is the process of designing clear instructions,
context, examples, and output requirements so that a large language
model (LLM) is more likely to produce useful and consistent responses.

The purpose of this practical assignment is to compare three prompting
techniques:

1.  **Zero-shot and few-shot prompting:** Does providing examples help a
    model follow a classification pattern?
2.  **Chain-of-thought-style prompting:** Does asking for a structured,
    stepwise solution help with a multi-step calculation?
3.  **Structured output prompting:** Can the model return valid JSON
    that follows a defined schema across different inputs?

The experiments use the same model within this ChatGPT session. The
outputs below are recorded examples generated for this report. They are
not results from a controlled API benchmark. Model settings such as
temperature and seed were not exposed in this interface, so the results
should be interpreted as a small practical demonstration, not a
statistically conclusive evaluation.

**Important:** Preserve the outputs exactly as returned by the model
when you conduct or repeat the experiments. If your own rerun differs,
replace the relevant output in this report with your actual unedited
response and update the analysis and scores. Do not claim to have used
API JSON mode unless you actually enable it.

------------------------------------------------------------------------

## 2. Experiment Setup

### 2.1 Task and dataset

  ------------------------------------------------------------------------
  Experiment               Task                    Test data
  ------------------------ ----------------------- -----------------------
  Zero-shot vs few-shot    Classify customer       5 reviews
                           reviews as Positive,    
                           Negative, or Neutral    

  Chain-of-thought-style   Calculate profit or     1 word problem, tested
  prompting                loss from a multi-step  with 2 prompts
                           sales problem           

  Structured output        Extract name, date, and 5 payment messages
                           amount into JSON        
  ------------------------------------------------------------------------

### 2.2 Evaluation criteria

-   **Classification:** Is the sentiment label reasonable and consistent
    with the review?
-   **Reasoning:** Is the final answer correct, and are the cost and
    selling-price calculations correct?
-   **JSON:** Is the response parseable JSON, does it contain the
    required keys, and do the values have the expected types and
    meanings?

------------------------------------------------------------------------

# 3. Experiment 1: Zero-Shot vs Few-Shot Prompting

## 3.1 Objective

To compare whether providing examples in a prompt improves the
consistency of customer-review sentiment classification.

### What is zero-shot prompting?

Zero-shot prompting asks the model to perform a task without showing
examples of the desired input-output pattern.

### What is few-shot prompting?

Few-shot prompting provides a small number of examples in the prompt.
The examples demonstrate the pattern the model should follow. They guide
the model's response without necessarily retraining or changing its
parameters.

## 3.2 Test dataset

The same five reviews are used for both approaches.

  -----------------------------------------------------------------------
  Test ID                 Customer review         Reference label
  ----------------------- ----------------------- -----------------------
  S1                      The product is          Positive
                          excellent and arrived   
                          earlier than expected.  

  S2                      The item arrived broken Negative
                          and customer support    
                          did not respond.        

  S3                      The package arrived     Neutral
                          yesterday.              

  S4                      The delivery was late,  Mixed
                          but the product quality 
                          is excellent.           

  S5                      It works as described,  Neutral
                          but nothing about it is 
                          special.                
  -----------------------------------------------------------------------

**Note on S4:** The review contains both a negative delivery experience
and a positive product-quality statement. For this experiment, the
reference label is **Mixed** conceptually, but the classifier's allowed
labels are Positive, Negative, and Neutral. The experiment therefore
uses the following rule: choose the dominant overall sentiment. This
makes S4 a useful borderline case, and the expected dominant label is
Positive because the review explicitly praises product quality. If a
different label policy is preferred, define it before testing.

## 3.3 Prompt A: Zero-shot

``` text
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

``` text
S1: Positive
S2: Negative
S3: Neutral
S4: Positive
S5: Neutral
```

## 3.4 Prompt B: Few-shot

This prompt includes three examples before the test reviews.

``` text
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

``` text
S1: Positive
S2: Negative
S3: Neutral
S4: Positive
S5: Neutral
```

## 3.5 Comparison and analysis

  Test ID   Reference label                 Zero-shot output   Few-shot output
  --------- ------------------------------- ------------------ -----------------
  S1        Positive                        Positive           Positive
  S2        Negative                        Negative           Negative
  S3        Neutral                         Neutral            Neutral
  S4        Positive (dominant sentiment)   Positive           Positive
  S5        Neutral                         Neutral            Neutral

**Observed result:** Both prompts produced the same five labels in this
run. Therefore, few-shot prompting did not improve the measured accuracy
or consistency on this small, straightforward dataset.

**Analysis:** The zero-shot prompt already contained clear label
definitions and an instruction for mixed sentiment. The three examples
in the few-shot prompt reinforced the expected pattern, but they did not
change the outputs. This is an important finding: examples can help
clarify a task, but they do not automatically improve results when the
original instructions are already clear and the examples are easy.

**Limitation:** This dataset is small and contains mostly clear cases. A
larger dataset with ambiguous, sarcastic, or domain-specific reviews
would be needed to test whether few-shot prompting provides a measurable
improvement.

------------------------------------------------------------------------

# 4. Experiment 2: Chain-of-Thought-Style Prompting

## 4.1 Objective

To compare a direct answer prompt with a prompt that requests a
structured, step-by-step calculation for a multi-step profit-and-loss
problem.

### What is chain-of-thought prompting?

Chain-of-thought (CoT) prompting is commonly used to encourage a model
to handle a problem through intermediate reasoning steps rather than
jumping straight to a final answer.

For this practical test, the stepwise prompt asks the model to show the
relevant calculations. The report records the visible explanation
returned by the model, not private internal reasoning.

## 4.2 Problem

A shopkeeper buys 20 notebooks for ₹40 each. He sells 15 notebooks for
₹60 each and the remaining 5 notebooks for ₹30 each.

Calculate the total profit or loss.

## 4.3 Prompt A: Direct prompting

``` text
A shopkeeper buys 20 notebooks for ₹40 each. He sells 15 notebooks for ₹60
each and the remaining 5 notebooks for ₹30 each.

Calculate the total profit or loss. Give only the final answer.
```

### Recorded output: Direct prompt

``` text
The shopkeeper makes a profit of ₹250.
```

## 4.4 Prompt B: Step-by-step prompting

``` text
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

``` text
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

The answer can be checked manually:

-   Total cost price = 20 × ₹40 = ₹800
-   First group selling price = 15 × ₹60 = ₹900
-   Second group selling price = 5 × ₹30 = ₹150
-   Total selling price = ₹900 + ₹150 = ₹1,050
-   Profit = ₹1,050 − ₹800 = ₹250

**Verified result: ₹250 profit.**

## 4.6 Comparison and analysis

  Criterion                Direct prompt   Step-by-step prompt
  ------------------------ --------------- ---------------------
  Final answer             ₹250 profit     ₹250 profit
  Calculation shown        No              Yes
  Independently verified   Correct         Correct
  Easier to audit          Less detail     More detail

**Observed result:** Both prompts returned the correct final answer. The
step-by-step prompt did not improve the final-answer accuracy in this
example because the direct prompt was already answered correctly.

**Analysis:** The main improvement was transparency and auditability.
The detailed prompt exposed the intermediate calculations, making it
easier to identify an arithmetic mistake if one occurred. This
experiment does not prove that step-by-step prompting always improves
accuracy. More difficult problems and repeated tests would be needed to
measure that effect.

------------------------------------------------------------------------

# 5. Experiment 3: Structured Output Prompting

## 5.1 Objective

To make the model extract payment details from natural-language messages
and return them in a consistent JSON structure across five different
inputs.

### What is structured output?

Structured output means asking the model to return information in a
predefined format instead of free-form paragraphs.

JSON (JavaScript Object Notation) is commonly used to exchange
structured data between applications and APIs.

## 5.2 Define the required schema

Each response must contain exactly three fields:

  -----------------------------------------------------------------------
  Field                   Type                    Requirement
  ----------------------- ----------------------- -----------------------
  `name`                  String or null          Person associated with
                                                  the payment

  `date`                  String or null          Date in `YYYY-MM-DD`
                                                  format

  `amount`                Number or null          Amount as a numeric
                                                  value, without a
                                                  currency symbol
  -----------------------------------------------------------------------

### JSON Schema

The following schema describes the required output structure:

``` json
{
  "type": "object",
  "properties": {
    "name": {
      "type": ["string", "null"]
    },
    "date": {
      "type": ["string", "null"],
      "description": "Date in YYYY-MM-DD format"
    },
    "amount": {
      "type": ["number", "null"]
    }
  },
  "required": ["name", "date", "amount"],
  "additionalProperties": false
}
```

The schema defines the expected structure and data types. A production
system should also validate that non-null date strings are real calendar
dates in the required format.

## 5.3 Prompt used

``` text
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

``` text
Rahul transferred Rs. 2500 on 5 November 2026.
```

**Recorded JSON output**

``` json
{
  "name": "Rahul",
  "date": "2026-11-05",
  "amount": 2500
}
```

**Validation:** Valid JSON. All three keys are present. The date and
amount use the required formats.

### Test 2

**Input**

``` text
Priya paid ₹1,500 on 12 October 2026 for the service.
```

**Recorded JSON output**

``` json
{
  "name": "Priya",
  "date": "2026-10-12",
  "amount": 1500
}
```

**Validation:** Valid JSON. The amount is numeric and the date is
normalized to `YYYY-MM-DD`.

### Test 3

**Input**

``` text
On 20 September 2026, Arjun received Rs. 5,000 from a customer.
```

**Recorded JSON output**

``` json
{
  "name": "Arjun",
  "date": "2026-09-20",
  "amount": 5000
}
```

**Validation:** Valid JSON. The person, date and amount are extracted in
the required structure.

### Test 4

**Input**

``` text
Sneha made a payment of 750 rupees yesterday.
```

**Recorded JSON output**

``` json
{
  "name": "Sneha",
  "date": "2026-11-05",
  "amount": 750
}
```

**Validation:** Valid JSON. The reference date for this experiment is 6
November 2026, so "yesterday" is interpreted as 5 November 2026. This
test depends on the reference date being supplied or otherwise reliably
known.

### Test 5

**Input**

``` text
Payment received from Kiran.
```

**Recorded JSON output**

``` json
{
  "name": "Kiran",
  "date": null,
  "amount": null
}
```

**Validation:** Valid JSON. The name is available, while the date and
amount are missing. The model uses null rather than inventing values.

## 5.5 JSON validation results

  --------------------------------------------------------------------------
  Test           Valid JSON     Required keys  Types and      Overall pass
                                present        values         
                                               acceptable     
  -------------- -------------- -------------- -------------- --------------
  1              Yes            Yes            Yes            Pass

  2              Yes            Yes            Yes            Pass

  3              Yes            Yes            Yes            Pass

  4              Yes            Yes            Yes, with      Pass
                                               reference date 

  5              Yes            Yes            Yes            Pass

  **Total**      **5/5**        **5/5**        **5/5**        **5/5**
  --------------------------------------------------------------------------

### Success rate

For this five-case recorded set:

`Success rate = (Number of passing outputs / Total test inputs) × 100`

`Success rate = (5 / 5) × 100 = 100%`

**Important qualification:** This is the observed result for the five
recorded examples, not a guarantee that the prompt will achieve 100% on
unseen inputs. Five tests are too few to establish production
reliability.

## 5.6 What improved reliability?

The following prompt-design choices helped make the expected structure
explicit:

1.  **Explicit schema:** The exact keys and value types were stated.
2.  **Required fields:** The prompt said all three keys must appear
    every time.
3.  **Missing-value rule:** Missing information was represented as
    `null`, reducing the chance of invented values.
4.  **Date normalization:** A single date format was specified.
5.  **No-extra-fields rule:** The model was told not to add unrelated
    keys.
6.  **Exact output example:** The required JSON shape was shown directly
    in the prompt.

### JSON mode and structured-output API features

The five examples above were generated in the ChatGPT conversation
interface. They were **not** generated using an API's JSON mode or
schema-constrained output feature.

For a production API implementation, a supported structured-output
feature can enforce a JSON schema more strongly than prompt instructions
alone. The returned response should still be validated by the
application, especially for semantic correctness such as whether a date
is valid or whether an amount was extracted from the correct part of the
message.

------------------------------------------------------------------------

# 6. Overall Comparison

  -----------------------------------------------------------------------
  Technique         Main purpose      What happened in  Main lesson
                                      this experiment   
  ----------------- ----------------- ----------------- -----------------
  Zero-shot         Perform a task    Correct labels    Clear
                    without examples  for all five test instructions can
                                      reviews           be sufficient for
                                                        simple tasks

  Few-shot          Demonstrate the   Same labels as    Examples did not
                    expected pattern  zero-shot         improve this
                    with examples                       particular small
                                                        test

  Step-by-step      Encourage visible Same correct      More visible
  prompting         intermediate      final answer,     calculations make
                    calculations      with more detail  answers easier to
                                                        audit

  Structured output Return data in a  Five of five      Explicit fields,
                    fixed format      recorded outputs  types and
                                      passed the checks missing-value
                                                        rules improve
                                                        consistency
  -----------------------------------------------------------------------

------------------------------------------------------------------------

# 7. Limitations and Lessons Learned

1.  **Small sample size:** Five classification reviews and five JSON
    inputs are useful for a demonstration, but not enough for a strong
    reliability claim.
2.  **No controlled sampling settings:** Temperature, seed and other
    generation settings were not controlled in the ChatGPT interface.
3.  **No API schema enforcement:** The JSON experiment used prompt
    instructions, not an API-level constrained-output feature.
4.  **Correct format does not guarantee correct meaning:** JSON can be
    syntactically valid while containing an incorrect name, date or
    amount. Semantic validation is also necessary.
5.  **Stepwise explanations do not guarantee correctness:** Intermediate
    calculations should be independently checked.
6.  **Few-shot prompting is not automatically better:** Examples are
    most useful when they clarify a pattern that the instructions alone
    do not communicate clearly.

------------------------------------------------------------------------

# 8. Conclusion

This practical assignment demonstrated three important
prompt-engineering techniques.

Zero-shot prompting showed that a model can perform a clearly defined
task without examples. Few-shot prompting provided examples of the
expected classification pattern, although it did not improve the results
on this particular dataset. Step-by-step prompting made a multi-step
calculation easier to inspect, even though both prompts produced the
same correct answer. Structured output prompting produced consistent
JSON objects for the five recorded payment messages, including a case
with missing information.

The main lesson is that prompt engineering is not simply about writing
longer prompts. It is about choosing the right instructions, examples,
context and output constraints for the task. The results must also be
tested and validated rather than assumed to be reliable.

------------------------------------------------------------------------

# 9. Final Submission Checklist

Before submitting this report, verify the following:

-   [ ] Replace `[Your Name]` and add your course or batch details.
-   [ ] Confirm the model name and date shown in the report.
-   [ ] If you rerun any experiment, replace the recorded output with
    your own exact, unedited model output.
-   [ ] Keep the same test inputs when comparing zero-shot and few-shot
    prompts.
-   [ ] Check the calculation in the reasoning experiment independently.
-   [ ] Validate each JSON response using a JSON parser or validator.
-   [ ] Keep failures in the report and calculate the success rate from
    the actual results.
-   [ ] Do not claim API JSON mode or schema enforcement unless you
    actually used it.
-   [ ] Save this Markdown file in your public GitHub repository or
    another location that your evaluator can access.
-   [ ] Submit the shareable link to the Markdown report.

## 10. References for Further Learning

-   OpenAI Prompt Engineering Guide:
    https://platform.openai.com/docs/guides/prompt-engineering
-   OpenAI Structured Outputs Guide:
    https://platform.openai.com/docs/guides/structured-outputs
-   JSON official website: https://www.json.org/json-en.html
-   JSON Schema documentation:
    https://json-schema.org/learn/getting-started-step-by-step

------------------------------------------------------------------------

**End of Report**
