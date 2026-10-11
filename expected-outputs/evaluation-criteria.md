# Evaluation Criteria for AI Job Scam Guard

This document outlines the methodology and metrics used to evaluate the performance of the AI Job Scam Guard. The evaluation compares the AI-generated results (`action-outputs`) against our verified ground truth (`expected-outputs`).

## 1. Core Evaluation Metrics

To ensure the AI agent performs accurately and reliably, we measure its outputs based on the following key metrics:

*   **Risk Level Accuracy:** The AI must correctly classify the overall risk of the job vacancy into the predefined categories: `Low`, `Medium`, or `High Risk`. 
*   **Scam Indicator Match Rate:** The AI must successfully identify the specific red flags present in the input (e.g., requests for upfront payment, use of unofficial free email domains, unrealistic salary offers, urgency/pressure tactics). 
*   **False Positives (FP):** Occurs when the AI flags a legitimate job posting as a scam. While undesirable, this is considered a safer error than a False Negative.
*   **False Negatives (FN):** Occurs when the AI fails to detect a clear scam indicator and classifies a fraudulent job as safe. **This is a critical failure.**

## 2. Pass / Fail Criteria

Each test case execution is graded based on strict criteria to determine if the AI's prompt and logic are functioning as intended.

### ✅ PASSED
A test case is marked as **Passed** if it meets **ALL** of the following conditions:
1.  **Accurate Risk Level:** The predicted overall risk level exactly matches the expected risk level.
2.  **Critical Indicators Detected:** The AI successfully identifies at least 80% of the critical scam indicators listed in the expected output (especially any indication of financial extortion or data theft).
3.  **No Hallucinations:** The AI does not invent information or red flags that are not present in the provided input (image/text/url).

### ❌ FAILED
A test case is marked as **Failed** if **ANY** of the following occur:
1.  **Incorrect Risk Level:** Misclassifying a `High Risk` scam as `Low Risk` (Critical Failure) or vice versa.
2.  **Missed Critical Indicators:** Failing to detect explicit scam elements (e.g., missing a clause that asks candidates to transfer money for training).
3.  **Format Violation:** The AI fails to output the result in the requested JSON/Markdown structure defined in the system prompt.

## 3. How to Conduct the Evaluation

1. Run the test script to generate the AI responses in the `action-outputs` directory.
2. Open the corresponding file in `expected-outputs`.
3. Compare the detected indicators and the final risk assessment.
4. Document any discrepancies to refine the AI's system prompt (Prompt Engineering) in future iterations.