# Langflow Component Prompts

`cek-loker-ai-job-scam-guard-component.md` is the core system prompt for the Agent component inside Langflow. It owns the main reasoning engine logic, which includes:
- Multimodal data extraction (processing both text and job poster images).
- Strict execution of verification tools (Notion database, Tavily Search, VirusTotal).
- Mathematical risk scoring based on the predefined job scam rubric.
- Conditional automated actions (generating Google Docs reports and drafting Gmail emails for high-risk cases).

This is distinct from `../bob/agent-instructions.md`, which only tells the IBM Bob frontend how to communicate with the user, handle file uploads, and delegate tasks to the Langflow tool without recalculating the final scores.