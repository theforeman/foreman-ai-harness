---
name: foreman-presentation-generator
description: >-
  Generate a Foreman-branded Reveal.js slide deck and speaker notes from one or
  more PRs, a Jira ticket, or free-text context. Use this skill when asked to
  create a presentation, demo deck, or slides for a sprint demo, team talk, or
  community talk about a Foreman change. Requires the `gh` CLI for PR inputs.
---

# Foreman Presentation Generator

Generate a self-contained HTML slide deck in the Foreman visual identity, plus
companion speaker notes with anticipated Q&A. For team presentations, sprint
demos, and community talks.

The full slide-design rules and the HTML / speaker-notes templates live in
[reference.md](reference.md) — follow them exactly when generating a deck.

## Inputs

- **PR URLs or numbers** (one or more): e.g., `608` or `https://github.com/theforeman/foremanctl/pull/608`
- **Jira ticket IDs** (optional): e.g., `SAT-12345` — extract summary and acceptance criteria
- **Free-text context** (optional): topic description when no PR/Jira is involved
- **Presenter name** (optional): defaults to `git config user.name`

## Outputs

Both saved in the current working directory:

1. `<topic>-presentation.html` — self-contained Reveal.js deck (opens in any browser)
2. `<topic>-presentation-notes.md` — slide-by-slide talking points and anticipated Q&A

## Workflow

### Phase 1: Gather context

For each PR, fetch details:

```bash
gh pr view <URL> --json title,body,author,state,commits
gh pr diff <URL>
```

For Jira tickets, ask the user to paste the summary/description if the Jira API
is not available. For free-text context, use it directly as the narrative basis.

### Phase 2: Identify the narrative

Structure the content as a story: **Problem** (what was broken/missing — use real
error messages) → **Root cause** (why — before/after diagrams help) → **Solution**
(what changed — key snippets, config, architecture) → **Takeaways**.

Focus on the technical narrative. Avoid PR-level metadata (file counts, lines
changed, commits, merge dates) and testing/validation steps (CI results,
reviewer names, build IDs). The audience cares about the topic, not the dev
process — that belongs in the PR description.

### Phase 3: Design the slide outline

Aim for 6-10 slides: title (teal) → problem → root cause / before-after (1-2) →
solution (1-3) → takeaways → thank you + questions (teal). **Present the outline
to the user before generating HTML.**

### Phase 4: Generate HTML

Use the HTML template in [reference.md](reference.md). Every content slide MUST
follow the `slide-title` + `slide-body` structure, and every slide MUST fit the
viewport without scrolling. See the Slide Design Rules in the reference.

### Phase 5: Generate speaker notes

Use the notes template in [reference.md](reference.md): slide-by-slide sections
headed by the slide title, talking points as natural prose (not bullets), and an
"Anticipated Questions & Answers" section grouped by topic.

### Phase 6: Iterate

Ask the user to open the HTML in a browser and adjust: split slides whose content
overflows, shorten code blocks that pass the right margin, and verify every slide
uses `slide-title` + `slide-body`.

## Example Usage

```
/foreman-presentation-generator https://github.com/theforeman/foremanctl/pull/608
/foreman-presentation-generator 608 654 662
/foreman-presentation-generator "Migration from foreman-installer to foremanctl"
/foreman-presentation-generator SAT-12345
```

## Limitations

- Cannot access authenticated Jira — user must paste ticket details
- Foreman logo loads from `theforeman.org` — requires internet when presenting
- Large PRs may need the user to specify which changes to focus on
- The generated deck is a starting point — always review in browser and iterate
