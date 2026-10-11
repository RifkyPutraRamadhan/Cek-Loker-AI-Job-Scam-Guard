# Sample Inputs

This directory houses the testing datasets (test cases) provided to the AI Job Scam Guard for analysis. These datasets simulate various forms of job vacancy scams, as well as legitimate job postings for comparative baseline purposes.

## Directory Structure
- `/image`: Contains visual media such as screenshots, brochures, or job vacancy posters.
- `/text`: Contains text-based job descriptions (e.g., copied WhatsApp messages, SMS, or emails).
- `/url`: Contains a list of URLs linking to job portal websites or fraudulent registration forms.

## Guidelines for Adding New Test Data
1. Select the appropriate sub-directory based on your test case media type (`image`, `text`, or `url`).
2. Save the file using a consistent naming convention (e.g., `poster-cek-loker-[id]-[risk-level].jpg`).
3. **Important:** Whenever a new sample input is added, ensure that its corresponding counterpart is created in the `expected-outputs` directory to allow for proper validation of the test case.