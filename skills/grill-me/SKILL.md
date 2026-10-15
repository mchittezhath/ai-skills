---
name: grill-me
description: 'Grill the user relentlessly about a plan, decision, idea, document, or codebase using multiple-choice questions. Manual use only: run when the user explicitly invokes /grill-me.'
argument-hint: 'Optional: what to grill you on (a Confluence page, repo, HTML file, plan, or idea)'
disable-model-invocation: true
---

Interview the user relentlessly until you reach a shared understanding. Map this as a **design tree**: every decision branches into the decisions that hang off it.

## Input

The user may give you something to grill them on: a Confluence page, a repo, an HTML file, a plan, or a plain idea.

- If they gave input, read it first (fetch the page, explore the repo, open the file) so you can skip questions it already answers and focus on gaps, risks, and unstated assumptions.
- If they gave nothing, ask a single multiple-choice question about what to grill them on (for example: a plan or idea, a document, a repo or codebase), with an "Other" for free text. Don't start grilling until you know.

## Rounds

Work the tree in **rounds**. The **frontier** is every decision whose prerequisites are already settled: the questions you can ask _now_ without guessing at answers you haven't heard yet. Ask the frontier in one round, then wait for the answers before the next round.

Ask each round with the `AskUserQuestion` tool, as multiple-choice questions:

- Up to 4 questions per round. If the frontier is larger, ask the most foundational ones first and carry the rest into the next round.
- Each question has 2-4 distinct options with a short description of the trade-off. The user can always type their own answer through "Other".
- Put your recommended answer first and suffix its label with "(Recommended)", so accepting it is a single selection.
- Use a short `header` as the question title, and word the question so the first option is a natural "yes".
- Allow multiple selections only when the options are not mutually exclusive.

Each round the user answers reshapes the tree: settled decisions push the frontier outward and unblock questions that depended on them. Recompute the frontier and ask the next round. A question whose answer depends on another question still open in this round belongs to a _later_ round, not this one.

## Facts vs decisions

Finding _facts_ is your job, never the user's. When a frontier question needs a fact from the environment (filesystem, tools, documents, etc.), dispatch a sub-agent to find it; don't ask the user for anything you could look up yourself. Don't block on it: a running exploration is an unsettled prerequisite, so only the questions downstream of it wait for the sub-agent to report; ask the rest of the frontier now. The _decisions_ are the user's: put each to them and wait.

## Done

The session is done when the frontier is empty: every branch of the design tree visited, nothing left silently assumed. Summarise the decisions and ask the user to confirm you have reached a shared understanding. Do not act on it until they confirm.
