# 2026-05-31 — Repo About Section Rewrite

## Goal

Rewrite the GitHub **About** section (description, website, topics) so a non-technical reader understands what the project is and what makes it distinctive — using the judgment-first lens ("prove the decision, not the output").

## What was decided

**The defining decision to lead with: zero AI calls.** Despite being a tool *about* AI automation, the app makes no LLM calls — every tool recommendation and starter prompt is generated instantly client-side by rule-based logic. That is the thing that surprises someone who assumes an "AI automation tool" phones home to a model. The full machinery (static site, single file, vanilla JS) was pushed to topics rather than the prose.

## What changed (GitHub metadata only — no code)

**Description (298 chars), before:**
> Fork-and-go tool for documenting automations. 8-step wizard → process playbook, tool recommendations, starter AI prompts. Deploy to Netlify in minutes.

**After:**
> Document an automation before you build it — even if you have never built one. Answer 8 questions and get a structured playbook with tool picks and starter prompts. The twist: it makes zero AI calls. Every suggestion is generated instantly in your browser, so it is free to run and yours to fork.

**Website:** `https://automate.demandai.studio` (confirmed, unchanged).

**Topics:** kept the original 8 (`ai`, `ai-tools`, `automation`, `netlify`, `no-code`, `supabase`, `template`, `workflow`) and added 4 machinery tags: `client-side`, `single-file`, `static-site`, `vanilla-javascript`.

**Applied via:** `gh repo edit --description ... --homepage ... --add-topic ...`, then verified with `gh repo view --json`.

## Notes / flags

- Repo is **public**, and the README "Deploy to Netlify" button forks it. New description and topics are clean of client-specific or personal context — nothing needed scrubbing.
- No commits, no pushes — repo metadata edit only.
