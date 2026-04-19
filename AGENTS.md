# Cursor agent context — AZ-305 exam prep

This workspace is for **studying for the Microsoft Azure AZ-305: Designing Microsoft Azure Infrastructure Solutions** certification. Treat every task here as supporting that goal: learning, retention, and exam readiness—not production deployment or unrelated coding unless the user asks.

## Exam focus (keep content aligned)

- Identity, governance, and monitoring (Entra ID, RBAC, policies, landing zones, monitoring design).
- Data storage and integration (storage accounts, databases, data movement, resilience).
- Business continuity and disaster recovery (backup, replication, recovery objectives).
- Infrastructure design (compute, networking, hybrid, migration patterns, scalability, security).

When creating or editing notes, tie concepts to **design decisions**, **trade-offs**, and **when to choose what**—that is how AZ-305 is framed.

## Q&A and follow-up questions (`*-q&a.md`)

When the user asks **extra questions**, **clarifications**, **drill prompts**, or **anything beyond the main study notes** for a given topic, capture the **question and answer** in a dedicated file so it stays scoped and easy to review.

- **Naming:** `<topic>-q&a.md` in the **same folder** as that topic’s main notes. The `<topic>` stem should match the main file (for example `design-governance.md` → `design-governance-q&a.md`; `log-monitor.md` → `log-monitor-q&a.md`).
- **When to create:** Create the matching `*-q&a.md` the **first time** a follow-up for that topic appears; **append** new Q&A entries there afterward—do **not** scatter one-off Q&A across random files or only in chat.
- **Format:** Short **Q** / **A** blocks (or dated entries). Keep answers consistent with this doc: plain language first, exam-relevant contrasts, and cite or defer to Microsoft Learn when facts are version-sensitive.

## How to write notes in this repo

- **Explain simply first**, then add detail. Assume the reader is learning, not reviewing a job runbook.
- **Define terms** on first use (acronyms, Azure product names, exam jargon).
- **Use structure**: short headings, bullets, and optional “In plain terms” or “Remember for the exam” callouts where helpful.
- **Prefer clarity over density**: one idea per paragraph or bullet; avoid walls of unexplained lists.
- **Use examples** (hypothetical scenarios, small diagrams in text/mermaid when it helps) so abstract topics feel concrete.
- **Highlight contrasts** (e.g., Active–Active vs Active–Passive, RTO vs RPO, when to use Private Endpoint vs Service Endpoint) so distinctions stick.
- **Stay accurate**: if unsure about a fact or a service limit, say so and suggest verifying in Microsoft Learn; do not invent exam specifics.

## Tone

Supportive and patient—like a good study guide. No unnecessary jargon; when jargon is required, explain it.

## What to avoid

- Vague bullet dumps without explanation.
- Production-only deep dives that skip the “why” and “when” needed for the exam.
- Unrelated frameworks or certifications unless the user connects them to AZ-305.
