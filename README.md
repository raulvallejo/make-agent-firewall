# Agent Firewall — Make AI Agent

A reusable security layer for Make AI agents, built on the **OWASP Top 10 for LLM Applications 2025**.

Drop it in front of any Make workflow to screen every incoming message before it reaches your main agent. Featured in [Make's Agent Library](https://www.make.com/en/ai-agents-library).

---

## What it does

Every incoming message is analyzed and returns a structured JSON verdict:

```json
{
  "safe": false,
  "threat_detected": "prompt_injection",
  "owasp_ref": "LLM01:2025",
  "confidence": "high",
  "reason": "The input explicitly attempts to override agent instructions.",
  "recommended_action": "block",
  "sanitized_input": "Ignore your previous instructions and [REDACTED]"
}
```

Your Make Router acts on `recommended_action` in one step:

| Action | Meaning | What to do |
|--------|---------|------------|
| `allow` | Message is safe | Pass `sanitized_input` to your main agent |
| `flag_for_review` | Suspicious but uncertain | Alert via Slack/email, queue for human review |
| `block` | Malicious input detected | Return rejection, log the attempt |

---

## Threat categories (OWASP LLM Top 10 2025)

| Category | OWASP Ref | Default action |
|----------|-----------|----------------|
| Prompt injection — direct | LLM01:2025 | block |
| Prompt injection — indirect (hidden in pasted content) | LLM01:2025 | block |
| System prompt leakage attempt | LLM07:2025 | block |
| Social engineering & false authority | LLM06:2025 | block |
| Data exfiltration (API keys, credentials, configs) | LLM02:2025 | block |
| Scope violation | LLM01:2025 | block |
| Harmful content | LLM09:2025 | block |
| PII in input — auto-redacted | LLM02:2025 | flag |
| Resource abuse / denial-of-wallet | LLM10:2025 | flag / block |

---

## Test results

13 test cases — zero false positives. Full details in [test-cases.md](./test-cases.md).

| # | Threat | OWASP | Action |
|---|--------|-------|--------|
| T01 | — | — | ✅ allow |
| T02 | prompt_injection | LLM01 | 🔴 block |
| T03 | indirect_injection | LLM01 | 🔴 block |
| T04 | system_prompt_leakage_attempt | LLM07 | 🔴 block |
| T05 | social_engineering | LLM06 | 🔴 block |
| T06 | data_exfiltration | LLM02 | 🔴 block |
| T07 | scope_violation | LLM01 | 🔴 block |
| T08 | harmful_content | LLM09 | 🔴 block |
| T09 | pii_in_input | LLM02 | 🟡 flag + redact |
| T10 | resource_abuse | LLM10 | 🟡 flag |
| T11 | prompt_injection (meta) | LLM01 | 🔴 block |
| T12 | scope_violation | LLM01 | 🟡 flag |
| T13 | — | — | ✅ allow |

---

## Files

| File | Description |
|------|-------------|
| `system-prompt.txt` | Full agent system prompt — paste into Make AI Agent |
| `test-cases.md` | All 13 test cases with inputs and expected results |
| `scenario/blueprint.json` | Make scenario blueprint — import directly into Make |

---

## Setup

**1. Create the agent**
- In Make, create a new AI Agent
- Paste the contents of `system-prompt.txt` into the system prompt field
- Model: GPT-5.4 nano (fast, cheap, ideal for classification)
- Max tokens: 300 · Temperature: 0 · Thread history: 0 · Recursion limit: 1

**2. Import the scenario**
- In Make, create a new scenario
- Import `scenario/blueprint.json`
- Connect your webhook and Gmail/Slack credentials

**3. Full scenario structure**
```
Webhook (Input)
  └── Make AI Agents (Agent Firewall)
        └── JSON Parse (Parse Verdict)
              └── Router (Route by Action)
                    ├── allow          → Webhook Response (Return Safe Input)
                    ├── flag_for_review → Gmail Alert + Webhook Response
                    └── block          → Webhook Response (Return Block Response)
```

**4. Run the 13 test cases**
Use the inputs in `test-cases.md` to validate before going live.

---

## References

- [OWASP Top 10 for LLM Applications 2025](https://genai.owasp.org/llm-top-10/)
- [Make AI Agents documentation](https://help.make.com/make-ai-agents)
- [Make Agent Library](https://www.make.com/en/ai-agents-library)

---

## Author

Built by [Raul Vallejo](https://github.com/raulvallejo) — PM at Celonis, building in the agentic space.
