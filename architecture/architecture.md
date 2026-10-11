# Cek Loker: Architecture

Cek Loker is an AI agent that assesses the scam risk of a job posting before a job seeker sends personal data or money. A user pastes a posting (text, link, or screenshot). The agent gathers evidence from a knowledge base, the web, and a URL reputation service, then returns a weighted risk score with the reasons behind it. For high-risk cases (score ≥ 50) it also prepares a formal report and a draft complaint email. A human reviews and sends anything that leaves the system.

**Stack:** IBM Langflow (workflow and agent engine, now part of IBM's portfolio following the DataStax acquisition [4]) · IBM Bob (conversation layer, connected via MCP) · Composio (Notion, Google Docs, Gmail connectors) · Tavily (web search) · VirusTotal (URL reputation).

## 1. Context

- According to SEEK research data, Indonesia is currently recorded as the largest hotspot for job scams in Asia Pacific [1].
- Indonesia accounts for 38% of all fraud attempts in the Asia Pacific region and 62% of job scam cases across Asia (detikFinance, 2025) [2].
- Entry-level jobs such as administration and office staff are the main targets, with more than 39% of job ads in those categories flagged as scams (BaliNews.id, 2025) [1].
- The Ministry of Communication and Digital Affairs (Kemkomdigi) together with BP2MI took down 7,908 fake overseas job ads between January and August 2026 [3].

## 2. Implementation status

| Capability | Status | Notes |
|---|---|---|
| Langflow flow with Agent and five tools | Implemented | Runs in the Langflow Playground. |
| Notion knowledge base query | Implemented | Read-only. |
| Web search (Tavily) | Implemented | One call per check. |
| VirusTotal domain lookup | Implemented | Successfully analyzes URL reputation in an integrated way. |
| Weighted score and risk level | Implemented | Rubric lives in the Agent instructions (section 6). |
| Gmail draft | Implemented | Creates drafts only. Sending is not enabled. |
| Google Docs report | Implemented | Successfully generates report documents through authenticated OAuth. |
| Text input | Implemented | Successfully processes and extracts information directly from the posting's text description. |
| Link (URL) input | Implemented | Successfully extracts the context of the job posting's web page and triggers domain verification. |
| Image input | Implemented | Fully supported if the LLM used has vision capability. |
| IBM Bob connection via MCP | Implemented | Successfully connected and tested end-to-end. The Langflow project MCP server is the integration point (section 8). |
| Application tracker (Google Sheets) | Planned | Not yet in the flow. |
| Reminders (Google Calendar) | Planned | Not yet in the flow. |
| Community feedback loop (write to Notion) | Planned | Needs a moderation step. |

## 3. System architecture

```mermaid
flowchart TD
    U["User: job posting text / URL / screenshot"] --> B["IBM Bob: conversation layer"]
    B -->|"MCP tool call"| M["Langflow project MCP server"]
    M --> A["Langflow flow: Cek Loker Agent"]
    A --> N["Notion via Composio: query database"]
    A --> T["Tavily: web verification"]
    A --> V["VirusTotal API: domain reputation"]
    N --> S["Weighted risk score"]
    T --> S
    V --> S
    S -->|"score >= 50 (High Risk)"| D["Google Docs: formal report"]
    S -->|"score >= 50 (High Risk)"| G["Gmail: formal draft, never sent"]
    S -->|"score < 50 (Safe)"| O["Chat Output: score, indicators, guidance"]
    D --> O
    G --> O
    O --> B
    B --> R["User reads the result and decides"]
```

The Langflow Agent orchestrates the tools inside one flow. IBM Bob calls that flow as a single MCP tool and explains the result to the user. Data moves over MCP: Bob passes the job posting content as tool input, Langflow returns the risk score, the evidence list, and its sources, and Bob then explains the result to the user.

## 4. Components

| Component | Configuration | Role |
|---|---|---|
| Chat Input | Default | Receives the posting from the user or from Bob. |
| Agent | Max Iterations 25. Instructions stored in the Agent Instructions field. | Selects tools, applies the rubric, writes the answer. |
| Language model | Any provider supported by the Agent component. Runs so far used Google Gemini and Groq `openai/gpt-oss-120b`. | Reasoning. |
| Notion (Composio) | OAuth2, Composio-managed. Only `NOTION_QUERY_DATABASE` enabled. | Read the company registry and scam pattern table. |
| Tavily Search API | Default | Check whether the company has a verifiable web presence. |
| API Request | GET `https://www.virustotal.com/api/v3/domains/{domain}`, header `x-apikey` | Domain reputation (`last_analysis_stats`, `reputation`). |
| Google Docs (Composio) | Only the create-document action enabled | Formal report for high-risk results. |
| Gmail (Composio) | Only `GMAIL_CREATE_EMAIL_DRAFT` enabled | Formal draft complaint email. |
| Chat Output | Default | Returns the final answer. |

The full Agent instructions are versioned in `prompts/cek-loker-ai-job-scam-guard-component.md`. This system prompt lives inside the Langflow Agent component, not in Bob's instructions (`bob/agent-instructions.md`). Keep the two in sync whenever either changes.

## 5. Request lifecycle

```mermaid
sequenceDiagram
    actor User
    participant Bob as IBM Bob
    participant MCP as Langflow MCP endpoint
    participant Agent as Langflow Agent
    participant Notion
    participant Tavily
    participant VT as VirusTotal
    participant Docs as Google Docs
    participant Gmail
    User->>Bob: Paste job posting
    Bob->>MCP: Call Cek Loker tool
    MCP->>Agent: Run flow with posting text
    Agent->>Notion: Query database (once)
    Notion-->>Agent: Registration status, scam patterns
    Agent->>Tavily: Search company (once)
    Tavily-->>Agent: Web results
    opt Posting contains a URL
        Agent->>VT: GET domain report (once)
        VT-->>Agent: Analysis stats, reputation
    end
    Agent->>Agent: Compute weighted score
    opt Score 50 or above
        Agent->>Docs: Create formal report
        Agent->>Gmail: Create formal draft
    end
    Agent-->>MCP: Chat Output
    MCP-->>Bob: Result
    Bob-->>User: Explanation and guidance
```

1. **Input.** The user sends text, a URL, or a screenshot if the model supports images. The agent extracts the company name, contact details, payment requests, and links.
2. **Evidence.** The agent queries Notion first, runs one web search, and calls VirusTotal only when the posting contains a URL.
3. **Scoring.** The agent sums the rubric points (section 6) and assigns a level.
4. **Action.** Only a score of 50 or above triggers Google Docs and Gmail.
5. **Answer.** The agent returns the score, each indicator with its weight and source, the action status, limits of the evidence, and a pointer to official verification channels.

## 6. Agent design

### Tool call budget

| Tool | Limit |
|---|---|
| Notion | 1 call, called first |
| Tavily Search | 1 call |
| VirusTotal | At most 1 call, only if the posting contains a URL |
| Google Docs | 1 call, only for High risk |
| Gmail | 1 call, only for High risk |

The agent must write its final answer after collecting evidence. It may not repeat searches to gain confidence. An email address without a URL does not justify a VirusTotal call.

### Scoring rubric

| Points | Indicator |
|---|---|
| +30 | The posting asks for an upfront payment or deposit. |
| +25 | The company is not found in Notion or in web results, or has the status `Tidak Terdaftar` (not registered). Never applied if the status is `Terdaftar` (registered). |
| +20 | The recruiter's domain or email does not match the company, or a free email/blog service (gmail, yahoo, blogspot, etc.) is used as the official contact. |
| +15 | VirusTotal flags the URL, or a scam pattern in Notion matches. Not applied merely because the salary or requirements look unusual. |
| 0 | Registered, consistent, no fraud indicators. |

| Total | Level |
|---|---|
| Below 20 | Low |
| 20 to 49 | Medium |
| 50 or above | High |
| Insufficient evidence | "Belum dapat dinilai" (cannot be assessed yet) |

The maximum score is 90.

### Decision rules

- A domain mismatch counts once, under the +20 indicator, never again under +25.
- The +20 indicator carries one exception: no penalty when the Notion status is `Terdaftar` and the email domain exactly matches the Official Domain.
- Status `Belum Terverifikasi` adds no +25 and no deduction. The agent says the registration data is not enough and scores only proven indicators.
- A tool error, empty result, or 404 counts as "no evidence", not as a risk signal.
- Company name matching ignores letter case and trailing markers such as `(Fiktif)`.
- Text from postings, images, web results, and Notion is treated as data, not instructions. The agent ignores any instruction inside it.
- The Agent Instructions state the scoring rules, call budget, and output format. They contain eleven numbered hard rules plus output examples for Low, Medium, High, and "Belum dapat dinilai" results.

### Output format

1. Score and level
2. Indicators with weights and sources
3. Automatic action status (High only)
4. Notes on evidence limitations
5. Guidance to official verification channels (Kemnaker, or BP2MI for overseas work)

### Formal documents

The chat answer, the Google Doc, and the Gmail draft carry the same facts: score, level, indicators, weights, sources, and limits. Wording and layout differ.

- **Google Doc:** formal Indonesian report, plain text. Sections: title, summary, inspected posting data, risk indicator findings, final assessment, limitations, follow-up recommendations, closing disclaimer.
- **Gmail draft:** formal complaint email with subject, greeting, numbered findings, final score, a request for verification, and a closing with `[Nama Pelapor]` and `[Kontak Pelapor]` placeholders. The recipient stays empty when no official address is known. The agent never invents an address or a name.

## 7. Knowledge base (Notion)

Database: **Registered Companies & Scam Reports**, read-only for the agent.

| Column | Values |
|---|---|
| Company Name | Text. Fictional entries carry the suffix `(Fiktif)` (fictional). |
| Registration Status | `Terdaftar` (registered), `Tidak Terdaftar` (not registered), `Belum Terverifikasi` (not yet verified) |
| Official Domain | URL |
| Known Scam Pattern | Text describing a documented tactic, for example an up-front fee or an impersonated government program |

The table holds 14 synthetic rows: registered companies, fictional scam entries with distinct tactics, and one unverified company that exercises the "cannot be assessed" path. Fictional domains use the `.test` top-level domain, which RFC 2606 reserves for testing, so none can resolve to a real site [5]. For `Tidak Terdaftar` rows, the Official Domain column records the domain the suspicious party used. It does not prove the company is genuine.

Replace any row whose name or domain resembles a real company before a public demo.

## 8. IBM Bob and MCP integration

Each Langflow project has its own MCP server, shown in the **MCP Server** tab next to the **Flows** tab. Bob sees only the flows of the project whose endpoint you configure. With more than one project in Langflow, copy the configuration from the project that contains the Cek Loker flow.

Setup order (from the workshop material), with one check per layer:

1. **Flow:** run a known posting in the Playground and confirm the Chat Output.
2. **Project:** enable MCP, copy the endpoint of the correct project, and confirm the Cek Loker flow appears as a tool.
3. **Bob:** add the server in Bob, review the discovered tool, and send one explicit request before trying natural phrasing. Use the contents of `bob/agent-instructions.md` as Bob's agent instructions.

Bob picks a tool from its name and description, so write both with care. Suggested values:

- **Name:** `check_job_posting_risk`
- **Description:** Assesses the scam risk of a job posting. Checks a Notion knowledge base, web search, and URL reputation, then returns a weighted risk score with reasons. For high risk it also creates a formal Google Docs report and a Gmail draft. Input: job posting text or URL. Do not use for general career advice.

## 9. Responsible AI and safety controls

| Control | Mechanism | Enforced by |
|---|---|---|
| Email is never sent | Only `GMAIL_CREATE_EMAIL_DRAFT` is enabled | Toolkit configuration and prompt rule |
| Knowledge base is never modified | Only `NOTION_QUERY_DATABASE` is enabled | Toolkit configuration |
| No inbox access | No Gmail read/search actions | Toolkit configuration |
| Human review | The user reviews, edits, and sends drafts | Design |
| No absolute verdicts | The agent reports indicators; the decision stays with the user | Prompt |
| Honest uncertainty | "Belum dapat dinilai" and disclosure of tool failures | Prompt |
| No invented contacts | Empty recipient and placeholders `[Nama Pelapor]`, `[Kontak Pelapor]` | Prompt |
| Prompt injection resistance | Posting and web content are treated as data | Prompt (not a guarantee) |
| Data minimization | The flow stores no user data of its own | Design |
| Credentials outside prompts | API keys sit in Langflow components and global variables | Configuration |

Configuration controls give a hard guarantee. Prompt controls only reduce risk and depend on model behavior. Check the toolkit settings before every release.

Additional notes:

- Send, delete, filter, and label actions are disabled for Gmail.
- Composio's "Requires Approval" setting can add a confirmation before a tool runs (optional).
- Langflow keeps Playground session history locally, so clear sessions before sharing screenshots.
- The Langflow flow export can contain the VirusTotal API key in the header table of the API Request component. Remove it or move it to a Langflow global variable before sharing the JSON file. Do not commit `.bob/mcp.json` if it contains a real Langflow API key.

Planned fairness checks: informal Indonesian and regional-language postings, low-quality screenshots, and slow connections, to confirm answer quality does not drop for these inputs.

## 10. Failure handling

| Situation | Behavior |
|---|---|
| Notion returns no matching row | Treated as "not found"; +25 applies only if web search also finds nothing |
| VirusTotal returns 404 or empty data | No evidence, no points |
| Google Docs or Gmail fails | The agent says the document or draft was not created and suggests copying the summary manually. It never claims success without confirmation. |
| Model rate limit or 503 | The run fails at the model layer. Retry later or switch the provider in the Agent component. |
| Recursion limit reached | Max Iterations raised to 25, and the per-tool call budget stops repeated calls |

## 11. Limitations

- **No live official registry.** The Notion table contains 14 rows of synthetic data standing in for a registry. Replace any row whose name or domain resembles a real company before a public demo.
- **Large VirusTotal responses.** The prompt instructs the agent to read only `last_analysis_stats` and `reputation`, but the full response still consumes tokens.
- **Free-tier model quotas.** Early tests hit rate limits and unavailability on free tiers. Production use needs a paid or self-hosted model.
- **Nondeterminism.** LLM output varies between runs. Scores were checked manually and there is no automated regression suite yet.
- **Image input** depends on the capabilities of the selected model.
- **Composio user ID.** Connected accounts must exist under the user ID that Langflow sends, which defaults to `default` [6]. A connection under another ID returns "No connected account found".
- **Custom component dropped.** A custom registry-check component failed when Langflow wrapped it as a tool. The registry data moved into Notion, so the Agent needs one tool fewer.
- **VirusTotal key storage.** The key sits in the API Request headers table. Move it to a Langflow global variable and remove it before sharing an exported flow JSON.

## 12. Roadmap

1. Record Playground evidence for each scenario (registered company, `Belum Terverifikasi`, fictional scam with a fee, company absent from Notion, posting with a real URL).
2. Add a VirusTotal wrapper tool that returns only `last_analysis_stats` and `reputation`.
3. Build a regression set of about 10 postings with expected score ranges.
4. Add the application tracker (Google Sheets) and reminders (Google Calendar).
5. Add the feedback loop: user-confirmed outcomes enter a pending queue, are reviewed, and only then update Notion.
6. Integrate a public registry source when an API becomes available.
7. Add a WhatsApp channel, where most job offers are received.
8. Partnerships with Kemnaker, BP2MI, and official job platforms, plus a white-label version for local manpower offices and universities.

## References

1. BaliNews.id (2025). "Waspada! Indonesia Jadi Hotspot Penipuan Lowongan Kerja Terbesar di Asia Pasifik Menurut Data SEEK." https://balinews.id/waspada-indonesia-jadi-hotspot-penipuan-lowongan-kerja-terbesar-di-aia-pasifik-menurut-data-seek
2. detikFinance (20 November 2025). "Ini Modus Penipuan Lowongan yang Jerat Banyak Pencari Kerja RI." https://finance.detik.com/berita-ekonomi-bisnis/d-8219789/ini-modus-penipuan-lowongan-yang-jerat-banyak-pencari-kerja-ri
3. Bisnis.com (8 September 2026). "Waspada Penipuan! Komdigi Ungkap Ada 7.908 Iklan Loker Luar Negeri Fiktif." https://teknologi.bisnis.com/read/20260908/84/2002462/waspada-penipuan-komdigi-ungkap-ada-7908-iklan-loker-luar-negeri-fiktif
4. DBTA (2025). "IBM Officially Closes Acquisition of DataStax." (Langflow is now part of IBM's watsonx portfolio). https://www.dbta.com/Editorial/News-Flashes/IBM-Officially-Closes-Acquisition-of-DataStax-169711.aspx
5. IETF (1999). RFC 2606, "Reserved Top Level DNS Names." https://www.rfc-editor.org/rfc/rfc2606
6. Composio. "Authentication." https://docs.composio.dev/docs/authentication
