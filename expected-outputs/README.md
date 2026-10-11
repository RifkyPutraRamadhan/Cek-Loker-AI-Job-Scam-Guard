# Expected Outputs

This directory contains the ground truth or the expected baseline outputs from the AI for each corresponding test case.

## Purpose
The files within this directory serve as the primary benchmark against the generated results in the `action-outputs` directory. The main objective is to validate whether the AI successfully and accurately identifies job scam indicators.

## Directory Structure
- `/image`: Expected analysis results from image-based inputs.
- `/text`: Expected analysis results from text-based inputs.
- `/url`: Expected analysis results from web link inputs.

## Evaluation Criteria
A test case is considered **Passed** if the analytical points in the `action-outputs` (e.g., detection of unrealistic salaries, requests for money transfers, unofficial contact numbers, etc.) align accurately with the predefined criteria outlined in the `expected-outputs`.