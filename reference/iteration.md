# Iteration

This file covers what happens after Step 5: how the skill invites follow-up refinement, how to diagnose which section of the improved prompt caused a bad result, how to apply the user's feedback, and how to keep a changelog note so the user can see what changed between versions. Read it when the user reports a problem with the first result and wants the prompt refined.

## Contents

1. Rules
2. Diagnosis table
3. Before and after
4. Apply in this skill

## Rules

### 1. Treat the first prompt as a first draft

Rule: close Step 5 with one line, set off after a horizontal rule or blank line and placed after the executed output (or after the not-run line, when the prompt was not run), saying that the prompt can be refined and asking the user to come back with what was wrong with the first result. The closing line is separate from the executed output and is not governed by the prompt's own no-commentary constraint.

Why it works: Anthropic's prompting overview frames prompt work as iterative and eval-driven: have clear success criteria, a way to test against them, and a first-draft prompt, then refine. A single rewrite cannot know every preference the user holds, and the first real output reveals them.

How to apply: write one sentence such as "If the result misses, tell me what was off and I will adjust the prompt." This line is the fifth and last part of Step 5 in SKILL.md. Keep it to one sentence and do not repeat the questions from Step 2.

### 2. Ask for feedback that points at a symptom

Rule: ask for the specific problem, not a general verdict. Useful forms are "what was missing", "what should not be there", "what did you expect instead", and a short quote of the offending part.

Why it works: "It is not good" gives nothing to change. A symptom maps to a section of the prompt, which makes the fix narrow.

How to apply: when feedback is vague, ask one question that offers the symptoms as a single line, for example "Was it too long, the wrong tone, missing something, or wrong in its facts?" This stays within the skill's one-line-answer pattern.

### 3. Diagnose which section failed before editing

Rule: map the symptom to one section of the prompt (context, instructions, examples, output_format, uncertainty) and change that section only.

Why it works: changing everything at once hides which edit helped, and can break parts that worked. Anthropic's guidance is to iterate against a subset of test cases, which depends on knowing what you changed.

How to apply: use the table below. State your diagnosis to the user in one sentence before showing the revised prompt. Keep one fixed sample input and re-run it after each edit, so the results of successive versions are comparable.

### 4. Raise specificity before adding length

Rule: when a result is off, first make the failing instruction more specific. Add new sections or extra emphasis only if specificity does not fix it.

Why it works: most misses come from an instruction that left room for a different reading (see obsessive-specificity.md). Anthropic's Sonnet 5 guidance, as summarized by the author and not verified here, says that model follows instructions literally, so a missing scope statement can be a cause; model behavior varies, so check Anthropic's current docs for the model in use. Emphatic wording such as "CRITICAL: You MUST" can cause overtriggering on some recent models.

How to apply: replace a vague phrase with a number or a named scope. Avoid adding capital letters or repeated warnings.

### 5. Keep a changelog note with every revision

Rule: every revised prompt comes with a short note that says what changed and why, plus anything that was deliberately left alone, within the same limit of at most five sentences as the Step 5 note. Name any section that changed as a side effect of the main edit.

Why it works: the user can see the effect of each edit, undo one if it was wrong, and learn which part of the prompt controls which behavior.

How to apply: label versions in the conversation, such as "Version 2", and write the changelog note in plain prose with the new prompt, in the same order as Step 5 (prompt, then note). If the user later asks to go back, restore the earlier text from the conversation.

### 6. Do not run Step 2 again

Rule: a revision is a loop back to Step 3, not a new intake. The 3 to 6 question rule governs the first intake only, and intake is one round. Reuse the answers already given.

Why it works: re-asking wastes the user's time and signals that their earlier answers were ignored.

How to apply: before changing anything, check whether the earlier answers already cover it. If the feedback shows that a core fact (audience, goal, or hard constraint) was wrong or missing, state the corrected assumption in the changelog note and revise. If one fact is still needed, ask it as a single line inside the note instead of opening a new question round.

### 7. Decide when to stop

Rule: stop revising when the user says the output is good enough, or when further edits are trading one flaw for another.

Why it works: Anthropic notes that not every failure is best solved by editing the prompt; for example, a different model may fix a latency or cost problem. Separately, chaining separate prompts for separate sub-tasks is a recognized technique when one prompt is doing too much, and it is mainly worth the extra step when you want to inspect the intermediate output. One chain is a self-correction pattern: a draft prompt produces a first result, and a second review prompt checks that result against stated criteria and refines it.

How to apply: if two revisions fail in the same way, say so and suggest splitting the task into two prompts instead of a third rewrite.

## Diagnosis table

- Wrong audience, tone, or level of expertise: edit the context section; add who is reading and what they know.
- The right topic but the wrong actions or missing parts: edit the instructions section; add the missing action verb or state the scope ("every section").
- Output has a right shape in content but wrong style: edit the examples section; add or replace an example, or remove one that was copied too closely.
- Output drifts into bullets, headings, or Markdown the user did not want, or into prose when they wanted structure: rewrite the prompt's own style to match the output you want, and restate the positive format line in `<output_format>`.
- Wrong length, format, or contains things the user did not want: edit the output_format section; state the exact format and the constraint in positive terms.
- Invented facts, or confident answers to questions the prompt did not support: edit the uncertainty section, and add quote-grounding if documents are involved.
- Sloppy reasoning or arithmetic errors: add or strengthen a verification cue (reasoning-cues.md).
- Claude mixed pasted material with instructions: restructure the tags (xml-structure.md).

## Before and after

Domain: pulling action items out of an email thread.

First prompt, and the user's feedback: the user ran it and said "It gave me a long recap of the thread, and I only wanted who does what by when."

Diagnosis: the instructions and output_format sections described a summary, not a list of owners and deadlines. The context (which already opened with a one-sentence role) and the uncertainty section were fine, so they stay.

Revised prompt (Version 2). Answers given earlier: the user coordinates a six-person logistics team and sends the follow-up email personally. The thread below is a sample.

```text
<context>You are an operations assistant for a six-person logistics team. I coordinate the team and send the follow-up email myself.</context>

<email_thread>
Priya: I will send the revised carrier contract to finance by Friday. Marcus: I can audit the Denver dock schedule, no date yet. Dana: someone needs to update the returns policy page by the 20th.
</email_thread>

<instructions>Read the thread in the email_thread tag and extract every commitment, including ones with no named owner or no date. Write each commitment as one sentence in the form: the person's name, what they will do, and the due date, writing "owner not stated" or "date not stated" in place of anything the thread leaves out. Before you respond, check that every commitment in the thread appears exactly once.</instructions>

<output_format>Return only the commitments, one sentence each, in the order they appear in the thread, beginning with the first commitment and ending after the last. Do not add a recap or an introduction.</output_format>

<uncertainty>If a commitment has no named owner or no due date, write "owner not stated" or "date not stated" in its place instead of guessing.</uncertainty>
```

Changelog note: the instructions now ask for commitments instead of a recap, because the first version asked for a summary, and they cover commitments without an owner or date. The output format is limited to commitments only. The context and the uncertainty section are unchanged from Version 1, which already told Claude to write "owner not stated" or "date not stated" rather than guess.

## Apply in this skill

Step 5 has five parts (in this order): the final prompt, the Assumptions paragraph when it applies, the short note on what changed and why, the execution, and one closing line inviting refinement, set off after a horizontal rule or blank line. In the two not-run exceptions the order is the prompt, Assumptions if any, the note, one line saying the prompt was not run and why (plus, for a reusable prompt, one line saying what to paste into the empty tags), and then the closing line. When the user returns, treat it as a loop back to Step 3, not a restart: diagnose the section, edit it, and keep Step 4 intact, since every revision must still contain the uncertainty section. Do not run Step 2 again; if the feedback shows that a core fact (audience, goal, or hard constraint) was wrong or missing, state the corrected assumption in the changelog note, or ask one line inside it. A revision follows the Step 5 order: the revised prompt, the Assumptions paragraph if any, the changelog note (within the limit of at most five sentences) after the revised prompt, the execution, and the closing line. Send the revised prompt to Claude as the user's request, the same way Step 5 does, unless one of the two exceptions in SKILL.md Step 5 applies (the user asked for the prompt only, including a target other than Claude, or a reusable or system prompt whose input is missing).
