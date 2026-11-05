# Question patterns and template

This is the short working sheet for two steps. Use the first half in Step 2 to write the clarifying questions, and the second half in Step 3 to assemble the improved prompt. Read uncertainty-handling.md together with this file, before Step 3, so the uncertainty section can be written while the prompt is assembled. For the reasoning behind each section of the template, follow the links at the end.

## Clarifying question patterns

Format: a numbered list of 3 to 6 questions, one per line, each answerable in one line. Fewer than 3 leaves gaps; more than 6 turns intake into a form. You may add an unnumbered last line, such as 'Reply "ok" to accept all my suggested defaults.' A reply of "ok" turns those defaults into confirmed answers.

The mandatory three, in every question list, even when the request looks complete:

1. Output format. Example: "What format should the answer take (for example, a 300-word email, a table, a Python function)?" Length and tone belong here or in question 3, not in a fourth mandatory question.
2. Audience. Example: "Who will read or use this (for example, new customers, your manager, other engineers)?"
3. Hard constraints. Example: "What must be true or avoided (for example, under 200 words, no jargon, no new dependencies, only the facts I pasted)?"

Goal-related additions (use when the goal is unclear; they fill the remaining 0 to 3 slots):

- What should the reader do or decide after reading this (for example, sign up, approve the budget)?
- What does a finished result look like (for example, a working function, a one-page summary)?
- What source material should be used (the pasted text only, or general knowledge too)?
- What time period, scope, or version matters (for example, this quarter versus last, Python 3.12)?
- What is being offered or discussed (product, offer, topic)? Use it for generate-from-scratch requests such as a cold email or a product description.
- Will you run this once now, or reuse it with new inputs? Use it when it is unclear whether the user wants a result or a reusable prompt.
- Who will use this prompt, and who reads what it produces? Use it when the two may differ.
- Code task: which language and operating system should it target (for example, Python 3 standard library on macOS, bash, PowerShell)? For a personal script, the audience question can shrink to a one-word confirmation folded into this one.
- Reusable prompt: what will you paste in each time (for example, prospect name, company, role), and what stays the same (your offer, your sign-off)? Required for a reusable prompt, and it takes one of the 3 to 6 slots. For a reusable prompt the audience question names the reader of the output (for example, the email recipient), not the person who runs the prompt.
- Will it touch real files, and how (for example, dry run first, back up first, never overwrite)? Use it for any script, command, or repo task that renames, moves, overwrites, or deletes files, in any environment. If the answer sets no rule, apply the safety default in instructions-and-action-verbs.md (Rule 9).

How to write a question the user can answer in one line:

- Ask one topic per question. Related sub-options in parentheses are fine ("tone and length" inside one format question), but split unrelated "and also" questions, or drop the weaker half. The blog example in SKILL.md is a documented exception: its first and third questions bundle related options to stay short.
- Put two to four concrete options in parentheses, so the user can pick one or copy one. Never more than four options per question, so it stays answerable in one line.
- Do not ask what the request already states or what the conversation already shows. If the request looks complete, still ask the mandatory three, phrased as a confirmation with your reading in the parentheses (for example, "Audience: I read this as new dog owners; correct?"). The step is never skipped.

Priority when slots run out: the mandatory three first, then the highest-impact gap, then the rest. Fold secondary items (output language, "only the pasted text") into the hard-constraints question as parenthetical options, within the four-option cap. The prompt targets Claude by default, and that default is never listed as an assumption; ask about the target only when the request mentions another model or tool. If something still does not fit, record it as an assumption instead of asking a seventh question.

Intake is one round. When the user says to proceed without questions: do not wait. Choose a reasonable default for the mandatory three and for anything else missing, write the prompt, and list every assumption explicitly in a paragraph labeled Assumptions, placed after the prompt and before the note on what changed.

When the user answers only some questions: use the answers given, treat each unanswered question as an assumption, and list those assumptions in the same Assumptions paragraph. If an answer is vague or declined, write it as exactly as the answer allows and list the gap there too. Do not ask again and do not hold the prompt back. When every question was answered, or the user accepted the offered defaults with "ok", there is no Assumptions paragraph, except for items the questions never covered and for any skill safety default the user did not confirm.

## Improved-prompt template

An annotated structure. Each tag below shows what its content is, described in words. The order of sections is: long documents, then context, then short input tags, then instructions, examples, output_format, and uncertainty. Leave out any part that does not apply (no documents, no inputs, no examples). Input size decides the layout: a pasted input longer than about 500 words goes first, inside named document tags, with the instructions after it (see long-context.md); shorter input goes in a named tag after the context.

```
<context>
Two to four sentences of plain prose. The first sentence is a one-sentence role for Claude ("You are ..."). Then the situation: who the user is, who will read the result, and any fact Claude cannot guess. Give the reason behind any constraint that is not obvious.
</context>

<input tag named for what it holds, only when short material is pasted>
The pasted text exactly as the user gave it.
</input tag>

<instructions>
One to three short paragraphs of plain prose. Start sentences with action verbs (write, compare, extract, rewrite, recommend) and name the exact scope: counts, lengths, sections, names. Use numbered steps only when the task itself needs a fixed order. If the task has a definition of done, state it near the end. The last sentence is the reasoning cue: one sentence naming what Claude should check before responding, such as verifying the answer against the stated constraints.
</instructions>

<examples>
Include only when an example would fix the format, tone, or an edge case. One to three examples, each wrapped in an example tag holding an input and an output, covering different cases. Omit the whole section when no example helps. The user may supply more examples; keep them all.
</examples>

<output_format>
Say positively what to return: the format, the length, the structure, what comes first. Then one or two short negative constraints (at most two), such as no preamble and no commentary after the result. Write this section in the style you want back.
</output_format>

<uncertainty>
One or two sentences giving Claude explicit permission to say "I don't know" or to flag missing or unclear information instead of guessing, aimed at this task's likely weak points. Required in every improved prompt.
</uncertainty>
```

Template rules:

- Every improved prompt uses these tags and flowing prose inside them, because prose prompts tend to steer Claude toward prose output. This applies to generated prompts only; this file may use lists.
- The role is the first sentence of the context. The compact blog example in SKILL.md is the one documented exception; every real prompt opens its context with a role sentence.
- The reasoning cue is the last sentence of the instructions. The uncertainty section is separate and never omitted.
- Do not add tasks, deliverables, or constraints that the user's answers did not confirm. For a simple one-step task, use a one-sentence verification cue with no extra thinking instructions and no examples, and keep the role, the instructions, the output format with its negative constraints, and the uncertainty section.
- A reusable prompt (input not supplied) carries named empty input tags, and its uncertainty section says what to do when a tag is empty or lacks the key fields: ask for the input in one line and do not write the output.
- Step 5, in order: (1) the finished prompt in a four-backtick fence; (2) an "Assumptions:" paragraph only when the user skipped the questions, answered only some, or answered vaguely or declined, listing those gaps plus any skill safety default (Rule 9) the user did not confirm, and never the Claude target; (3) a note of at most five sentences, giving first the one to three changes that mattered most, then, when they apply, that a long document was left unchanged or moved and that examples were invented, then anything else only if room remains, and naming the target only when the request named another model or tool; (4) the execution of the prompt as the user's actual request; (5) one closing line inviting feedback, set off after a horizontal rule or blank line and not governed by the prompt's own no-commentary constraint.
- The prompt is not run in two cases: the user asked for the prompt only (including a prompt for a target other than Claude), or it is a reusable or system prompt whose input is missing. The order is then the prompt, Assumptions if any, the note, one line saying the prompt was not run and why (in the reusable case, plus one line saying what to paste into the empty tags), and the closing line.
- A pasted document up to about 2,000 words is reprinted in full and unchanged inside its tag in the prompt block. Above that, or when the input is an attached file, the fence carries the named empty document tag plus one line telling the user to paste the original (or naming the file or path), and the execution uses the original already in context.

## Related files

role-and-context.md, instructions-and-action-verbs.md, output-format-and-constraints.md, reasoning-cues.md, examples-in-prompts.md, uncertainty-handling.md, xml-structure.md, long-context.md, worked-examples.md.
