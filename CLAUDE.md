# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Shape

This is a **single-file static web app** — `index.html` (~2,500 lines) contains all HTML, CSS, and JavaScript. There is no build step, no package manager, no framework, and no local dev server config. To work on it, edit `index.html` and open it in a browser (or use any static server, e.g. `python3 -m http.server`).

External dependencies are loaded from CDN at runtime only:
- Google Fonts (Fraunces, Barlow, Syne)
- `@supabase/supabase-js` v2 (UMD build) — optional, only used for submission logging

There are no tests, no linter config, no CI. Deployment is Netlify-direct from the repo root — pushing to `main` redeploys.

## Architecture in One Page

The app is an 8-step wizard that collects information about an automation idea, then renders a "playbook" with tool recommendations and starter AI prompts. All logic is **client-side and rule-based** — there are no LLM calls.

Three sections to know in `index.html`:

1. **Wizard markup** (~lines 1124–1280): eight `<div class="wizard-step" data-step="N">` blocks, one per question. Step 4 is special — it renders a dynamic list of sub-steps the user adds.
2. **Script block** (~line 1400 onward): contains state, validation, navigation, Supabase submission, output generation, and idea generator.
3. **Customization comments**: search for `<!-- CUSTOMIZE:` to find every brand-specific element (4 spots: title/meta, nav, footer CTA, copyright).

### Key JS landmarks (line numbers approximate — verify before editing)

- `SUPABASE_URL` / `SUPABASE_ANON_KEY` (~1406): placeholder credentials. App falls back to logging-disabled mode if these are still the `YOUR_*` placeholders.
- `state` object (~1418): wizard state — current step, collected answers, process steps array.
- `nextStep(n)` (~1507): per-step validation gate. Each step has bespoke validation rules (min word counts, tool-name detection, email format). Adding a step means extending this switch.
- `addProcessStep()` / `collectStepData()` (~1642, 1699): step 4 dynamic list management.
- `submitGate()` (~1782): collects name+email, writes one row to Supabase `automation_submissions` (schema in README), reveals output. Always reveals output even if Supabase write fails — logging is best-effort.
- `recommendTools(step)` + `TOOL_RATIONALE` (~1862, 1885): rule-based engine that maps keywords + judgment level + trigger type to tool suggestions. Edit `TOOL_RATIONALE` and the matching logic together.
- `generateStarterPrompt(step)` (~1956): template-based prompt generation per step.
- `buildOutput()` (~2011): assembles the final playbook HTML from collected state.
- `IDEA_DATA` (~2179): the 21 pre-loaded role-based idea seeds (3 per role × 7 roles) for the idea generator.

### Data flow

```
user input → state object → validation per step → email gate →
  Supabase insert (best-effort, anon role) → buildOutput() → DOM render
```

`localStorage` is used **only** to detect returning users so they can skip the email gate. No analytics, no tracking, no third-party scripts beyond the CDN libs above.

## When Editing

- **Adding a wizard step**: update the markup block, extend `state`, add a case to `nextStep()` validation, update `updateProgress()` denominator, and extend `buildOutput()` / `buildSupabasePayload()` / the Supabase table schema in README.
- **Changing tool recommendations**: edit `TOOL_RATIONALE` and the keyword-matching branches inside `recommendTools()` together — they reference each other by name.
- **Changing colors/fonts**: all CSS variables live in `:root` near the top of `<style>`. Don't introduce new color values inline.
- **Touching Supabase logic**: the table uses anon-role insert-only RLS. Never widen permissions in the README SQL without explaining why. The client never reads from Supabase.
- **HTML escaping**: anything user-supplied that lands in the output must go through `escHtml()` (~line 2168). The output section uses `innerHTML`, so this is the XSS boundary.

## Deployment Notes

- Netlify auto-deploys from `main` on push. No `netlify.toml` or build command — it serves the repo root as-is.
- The "Deploy to Netlify" button in README forks the public repo, so anything committed here is public. Never commit real Supabase credentials. The placeholder strings in `index.html` are intentional — users replace them in their own fork.
