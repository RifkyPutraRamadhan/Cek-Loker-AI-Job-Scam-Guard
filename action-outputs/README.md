# Action Outputs

This directory is designated for storing the raw generated responses (outputs) produced by the AI agent (Bob) during the testing execution phase.

## Directory Structure
- `/image`: Contains AI outputs generated from image-based inputs (e.g., job vacancy posters).
- `/text`: Contains AI outputs generated from text-based inputs (e.g., job descriptions, chat messages, or emails).
- `/url`: Contains AI outputs generated from web link inputs.

## Notes
- Files within this directory are typically auto-generated or overwritten each time the testing script is executed.
- These output files serve as the actual results to be compared against the ground truth located in the `expected-outputs` directory.