# Reasoning cues

This file covers the reasoning cue that closes every improved prompt: the "verify your answer before responding" check, when to ask Claude to think step by step, how to separate the final answer from other text with an answer tag, and when to leave heavy reasoning cues out. Read it only for math, code, or multi-step analysis, where the cue needs more than the one verification sentence; otherwise the one-sentence cue in the template is enough.

## Contents

1. Rules
2. Model-specific cautions
3. Before and after: a financial-analysis example
4. Apply in this skill

## Rules

### Rule 1: End with a verification cue

Anthropic's guidance recommends asking Claude to check its work before finishing, with a line such as "Before you finish, verify your answer against [test criteria]." It says this can catch errors, especially for coding and math. The cue is cheap, works on short prompts, and does not require Claude to write out its reasoning.

How to apply it: make the verification sentence the last sentence of `<instructions>` (after any definition of done), and make it specific to the task. The workflow requires this cue in every improved prompt, so it stays; for some models it may be less needed (see Model-specific cautions), so keep it short and specific. "Verify your answer before responding" is the minimum. Better: "Before you respond, check that every figure in your summary appears in the report and that the percentages add up." The specific form tells Claude what to check.

### Rule 2: Prefer a general thinking cue over a prescribed procedure

Anthropic's guidance says a prompt such as "think thoroughly" often produces better reasoning than a hand-written step-by-step plan, because Claude's reasoning frequently exceeds what a human would prescribe. When thinking may be off, prefer wording such as "reason through", "consider", or "evaluate" over "think" (see Model-specific cautions). Do not invent a reasoning recipe the user did not ask for. If the task is hard, ask Claude to work through the problem carefully and then check the result.

How to apply it: write one sentence such as "Reason through the problem carefully, then verify your answer before responding." Keep the method open unless the user's domain requires a particular method (for example, a required accounting procedure).

### Rule 3: Request step-by-step thinking for multi-step problems

Step-by-step reasoning helps when the task needs several dependent steps: arithmetic, logic puzzles, comparing options against criteria, debugging, planning. It adds little to retrieval, rewriting, or short factual questions.

How to apply it: when Step 1 shows the task has dependent steps, add one general reasoning line, such as "Reason through the problem carefully before you answer" or "Work through the calculation before you give the final figure." Add an explicit step-by-step instruction only when the user says thinking is off; where thinking is built in (see below), the general line is enough.

### Rule 4: Separate the final answer from the rest with tags, not a written-out reasoning transcript

The default is to rely on the model's built-in thinking plus the verification cue, and to ask for no written-out reasoning. Anthropic describes a manual fallback for when thinking is off: ask Claude to think through the problem before it answers and to put the final answer in a tag such as `<answer>` so you can extract it. The author's summary of Anthropic's pages, not verified here, is that on some of its newest models a prompt that asks the model to write out its reasoning, for example in `<thinking>` tags, may be declined. Model behavior varies; check current docs for the model in use, since this may change.

How to apply it: use the manual fallback only when the user says thinking is off or the target is an older model, and the user needs a clean final answer that other text would clutter. Name the tag in `<output_format>`: "Give the final result inside <answer> tags." Do not ask for reasoning inside `<thinking>` tags. If the user wants to see why, ask for a short explanation or the key evidence after the answer ("After the answer, give a short paragraph naming the evidence you relied on"). Choose consistent tag names; see `xml-structure.md`.

### Rule 5: Keep reasoning cues out of simple tasks

Rewriting a sentence, answering a factual question, formatting a list, or translating a paragraph does not need a reasoning cue beyond the single verification sentence. Extra cues add length without improving the result and may cause Claude to over-explain.

How to apply it: if the task is one step and low risk, use a one-sentence verification cue with no extra thinking instructions. Keep the role, the instructions, the `<output_format>` with its negative constraints, and the `<uncertainty>` section as the workflow requires; drop only the extra reasoning cues and the examples. Do not pad the prompt to look thorough.

### Rule 6: Add reflection after tool use only when tools are involved

Anthropic's guidance suggests asking Claude to reflect on the quality of tool results and decide the next step before proceeding. This applies to agentic work in Claude Code. For a chat prompt that uses no tools, leave it out.

## Model-specific cautions

Anthropic's current guidance says where a technique names a specific model, treat it as measured on that model and re-check it against your own tests before applying it to another. Every per-model statement below is as summarized by the author from Anthropic's pages, not verified here; model behavior varies and may change, so check current docs.

1. Anthropic's documentation is summarized by the author as describing thinking as on by default or adaptive on several of the newest models, where Claude decides how much to think from the effort setting and the difficulty of the question. For such models a long manual chain-of-thought instruction is often unnecessary; rely on the built-in thinking and keep the verification cue. The same summary says that asking the model in the prompt to think less does not reliably reduce its thinking, and that the effort setting does; that is a setting the user controls in their tool.
2. The author's summary is that some of the newest models may decline a prompt that asks them to write out their reasoning, for example in `<thinking>` tags. Anthropic's manual chain-of-thought-in-tags fallback is for when thinking is off. If you do not know which model the user will run, use the verification cue and a general reasoning line, and ask for a short explanation after the answer instead of visible reasoning.
3. The author's summary is that one model (Opus 5) verifies its work well without being asked, and that verification instructions carried over from other prompts can cause over-verification there. The verification cue stays because the workflow requires it; keep it to one short, specific sentence.
4. The author's summary is that one model (Opus 4.5) with thinking disabled is sensitive to the word "think", and that "consider", "evaluate", or "reason through" work instead. When thinking may be off, use "reason through" as the portable wording.

The skill does not know which model the user is running. Use the portable wording above: a general reasoning cue and a specific verification sentence.

## Before and after: a financial-analysis example

Before, as the user typed it:

```
is this startup worth investing in, here are the numbers
```

After. Answers given: the user is an individual angel investor with limited time; the check size is 25,000 dollars; the question is whether the numbers justify moving on to further due diligence; the output is a recommendation memo for the user alone; the figures are in the prompt.

```
<context>
You are an experienced venture analyst. I am an individual angel investor considering a 25,000 dollar investment in an early-stage software startup. I have limited time and want to know whether the numbers justify moving on to further due diligence. The founders' figures are below.
</context>

<financials>
Monthly recurring revenue is 18,000 dollars, up from 11,000 dollars six months ago. Customer acquisition cost is 2,400 dollars and average customer lifetime is 20 months at 150 dollars per month. The company holds 210,000 dollars in cash and spends 35,000 dollars per month net.
</financials>

<instructions>
Assess whether the unit economics, the revenue trend, and the runway in the financials tag support moving on to further due diligence, comparing each against what is typical for an early-stage software company, and state your overall judgment in one sentence. Reason through the problem carefully before you answer. Before you respond, verify that every number you cite matches the figures I gave and that your arithmetic is correct.
</instructions>

<output_format>
Write a one-page memo in plain prose with three short paragraphs: what the numbers show, the main risks they reveal, and your recommendation, stating the key figures you relied on inside the paragraphs. Do not add headings, and do not add commentary outside the memo.
</output_format>

<uncertainty>
If the figures are incomplete, or a benchmark you would normally rely on is one you cannot confirm, say so plainly and tell me which additional number you would need, rather than guessing.
</uncertainty>
```

What changed: the vague question became a defined judgment, the method stayed a general cue instead of a hand-written recipe, the key figures are stated in the memo so the user can check them, and the closing verification sentence names what to check. If the user needs the final answer separated from other text, see Rule 4.

## Apply in this skill

In Step 3, always add the reasoning cue the workflow requires as the last sentence of `<instructions>`, in its simplest form ("verify your answer before responding"), and make it specific to the task when you can. Add the general reasoning line ("Reason through the problem carefully before you answer") when Step 1 shows dependent steps, and an explicit step-by-step instruction only when the user says thinking is off. Add an `<answer>` tag only when the user needs the final result separated from other text. In Step 4, pair the cue with the uncertainty permission so that a failed check results in a flagged gap rather than a guess. In Step 5, mention whether you added heavy reasoning cues and why only if room remains in the five-sentence note.

Related files: `uncertainty-handling.md`, `instructions-and-action-verbs.md`, `output-format-and-constraints.md`, `iteration.md`.
