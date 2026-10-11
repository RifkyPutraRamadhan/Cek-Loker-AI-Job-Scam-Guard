# Cek Loker — AI Job Scam Guard (Langflow)

- `Cek Loker — AI Job Scam Guard.json`: The complete and finalized Langflow project export. 

*(Note: If you have a template/starting point version, you can add `- Template.json` here as well).*

**Instructions for Use:**
Import this JSON file into your Langflow environment to replicate the Cek Loker backend logic. After importing, please recheck the node settings and input your credentials/API keys (such as Tavily, VirusTotal, and Google credentials) as these are typically cleared during the export process for security reasons.

**System Prompt Management:**
The core agent component system prompt is documented at `../prompts/cek-loker-ai-job-scam-guard-component.md`. It belongs strictly inside the Langflow Agent component, **not** in Bob's frontend agent instructions (`../bob/agent-instructions.md`). 

When modifying or updating the flow in the future, please ensure that the system prompt inside your Langflow workspace and the documented markdown prompt remain synchronized.