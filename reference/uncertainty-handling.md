# Uncertainty handling

This file is the home of Step 4, and it is read together with question-patterns-and-template.md before Step 3, so the section can be written while the prompt is assembled. It covers how to give Claude explicit permission to say "I don't know", to flag missing information, to ask for clarification, and to ground claims in quotes when documents are involved, along with phrasing variants for different task types. Read it every time you write an improved prompt (before Step 3), because every improved prompt must contain an uncertainty section, and read it again when the user complains that an answer contained invented facts.

## Contents

1. Rules
2. Phrasing variants by task type
3. Before and after
4. Apply in this skill

## Rules

### 1. Always grant the permission explicitly

Rule: every improved prompt contains an uncertainty section that tells Claude it may say "I don't know" or flag missing information instead of guessing. No improved prompt ships without it, including short ones.

Why it works: Anthropic's hallucination guidance says that explicitly giving Claude permission to admit uncertainty can reduce false information.

How to apply: add an uncertainty tag as the last section of the prompt, and write it in a sentence or two that fits the task (see the variants below). Do not paste the same sentence into every prompt when a task-specific one is clearer.

### 2. Tell Claude what to do when it is unsure, not only that it may be unsure

Rule: pair the permission with an action: say it does not know, name what is missing, quote the ambiguous part, or ask one specific question.

Why it works: a bare "don't guess" leaves Claude without a way to finish the task. An alternative action gives it a correct path. This follows the broader guidance to tell Claude what to do instead of what not to do (output-format-and-constraints.md).

How to apply: choose the action that matches the user's situation. If the user can answer follow-ups, allow a question. If the output goes straight into a pipeline or a document, specify how a gap should appear in the output, for example the word "unknown" in the relevant field.

### 3. Allow Claude to ask for clarification only when it helps

Rule: allow a clarifying question in the uncertainty section when the user will read the answer and can reply. Do not allow it when the output is consumed by another program or when the user asked for a one-shot result.

Why it works: Step 2 of this skill already resolves most ambiguity before the prompt is written. A leftover gap that matters is better surfaced than guessed.

How to apply: write "If a missing detail would change the answer, ask me one specific question before continuing" for interactive tasks, and "State the missing detail in one sentence and continue with a clearly labeled assumption" when the work should not stop.

### 4. Ground claims in quotes when documents are involved

Rule: when the prompt includes documents or pasted source material, ask Claude to extract the relevant word-for-word quotes first and base the answer only on them, and to say so if no quote supports a point.

Why it works: Anthropic recommends extracting direct quotes before performing the task for long documents (it cites above 20k tokens), then answering from the quotes. A related technique is to have Claude draft an answer, find a supporting quote for each claim, and retract any claim it cannot support. These techniques reduce hallucination but do not eliminate it, so critical facts still need human checking.

How to apply: put the documents above the question (long-context.md), add a sentence asking for quotes first, and state the fallback such as "If no quote supports an answer, say that the documents do not contain it."

### 5. Restrict the knowledge source when the user wants only the supplied material

Rule: if the answer must come from the provided text alone, say so.

Why it works: Anthropic lists external knowledge restriction ("only use information from the provided documents") among its techniques for reducing hallucination. Without it, Claude may blend general knowledge with the document.

How to apply: ask the user in Step 2 whether outside knowledge is acceptable. If not, add the restriction to the instructions and mirror it in the uncertainty section.

### 6. For code and files, require investigation before claims

Rule: in tasks about existing code or files, tell Claude to read the relevant material before answering and not to describe code it has not opened.

Why it works: Anthropic provides an agentic sample block that says never to speculate about code that has not been opened and to give grounded answers. This applies where Claude has file access, as it does in Claude Code.

How to apply: include a sentence such as "Open and read the files before describing them, and tell me if a file you need is missing". Do not add it for tasks where no files are available.

### 7. Keep the permission calm and proportionate

Rule: use plain wording. Avoid capital letters and absolute threats such as "NEVER guess under any circumstances".

Why it works: Anthropic notes that some recent models respond strongly to system prompts (the author's summary names Opus 4.5 and 4.6, not verified here), and that emphatic wording such as "CRITICAL: You MUST" can cause overtriggering of tool use. Plain wording avoids the risk.

How to apply: write "If you are unsure, say so" and not "You MUST NEVER answer unless certain".

## Phrasing variants by task type

Use one of these as a starting point and adapt the nouns to the task. All are written as prose, matching the improved prompts this skill generates.

Factual questions: "If you are not confident in a fact, say 'I don't know' or say how confident you are, rather than guessing. Leave out any detail you cannot support."

Analytical tasks: "If the information given is not enough to reach a conclusion, say what is missing and what you would need, and then give the best conclusion the available information supports, labeled as provisional."

Code tasks: "If you cannot tell how a function or file behaves from what I have given you, say so and name the file or detail you need instead of inventing an interface. Mark any assumption you make in a comment."

Code that changes or deletes real files or data: the uncertainty section keeps the usual code-task permission (say what you cannot tell and name the assumption you made) and also tells Claude to flag a possible name collision or any case it cannot resolve, for example "If a name collision could occur, say so instead of choosing silently." The dry-run and no-overwrite default is a behavior rule, not an uncertainty rule, so it goes in `<instructions>` (see instructions-and-action-verbs.md, Rule 9), and it is listed under Assumptions when the user did not confirm it.

Anthropic's own phrasing, usable as a variant for any report or analysis, quoted as it appears in the guidance: 'If you're unsure about any aspect or if the report lacks necessary information, say "I don't have enough information to confidently assess this."'

Mixed summary and risk flagging over a document: "Summarize each section with a clause or section citation, and for any clause you flag as risky, quote the sentence behind it in at most 15 words. If the document does not cover something a reader would expect, say so instead of filling the gap." The summary part uses citations and only the flagged risks use short quotes.

Creative tasks: "If a detail about my brand, characters, or setting is missing and it matters, make a clearly labeled choice or ask me one question, and do not present an invented fact as if it came from me."

Document question answering: "Answer only from the documents above. First extract the exact quotes that bear on the question. If no quote supports an answer, reply that the documents do not contain it."

Reusable prompt with an input tag: "If the prospect_details tag is empty, or lacks the company name or the role, ask me for the missing items in one line and do not write the email." Name the actual tag and the fields that the output cannot be written without. Every reusable prompt's uncertainty section names its empty-input behavior.

Data and numbers: "If a figure is not in the data I provided, write 'not provided' in its place. Do not estimate a number unless I ask for an estimate, and label it as an estimate if you give one."

## Before and after

Domain: answering questions about a company policy document.

Before:

```text
here is our PTO policy, what does it say about carrying over days?
```

After. Answers given: the user is answering an employee's question about carrying over unused paid time off, using only the handbook section below; the employee asks how many unused days can be carried into next year and when they expire; the answer is two or three sentences of plain prose. The handbook text is a three-sentence sample. Quote extraction pays off on long documents (Anthropic cites above 20k tokens), so on a source this short it is shown only to illustrate the form, and citing the sentence relied on would do. For the same reason the document-first layout here only demonstrates the layout; a source this short would normally sit in a named tag after the context.

```text
<documents>
<document index="1">
<source>Employee handbook, section 6: Paid Time Off</source>
<document_content>Full-time employees accrue 1.5 days of paid time off per month. Unused days may be carried into the next calendar year up to a maximum of 5 days. Carried days expire on March 31.</document_content>
</document>
</documents>

<context>You are an HR assistant answering an employee's question using only the handbook section above.</context>

<instructions>First extract the exact sentences from the document that address carrying over unused paid time off. Then answer the employee's question using only those quotes, in two or three sentences, stating the maximum number of days and the expiry date if the document gives them. The employee asks how many unused days can be carried into next year and when they expire. Before you respond, confirm that each quote appears word for word in the document and that your answer matches the quotes.</instructions>

<output_format>Begin with the quotes in a quotes tag, then give the answer in plain prose. Do not add headings, and do not add advice beyond what the handbook says.</output_format>

<uncertainty>If the document does not state something the employee asked, say that the handbook section does not cover it instead of assuming a rule. If a quote is ambiguous, show it and explain the ambiguity.</uncertainty>
```

What changed: the document sits above the question, the answer is tied to extracted quotes, the knowledge source is limited to the supplied text, and the uncertainty section names the specific fallback.

## Apply in this skill

Step 3 writes the uncertainty section, using this file and the template read beforehand. Step 4 is a required check before Step 5: confirm that an uncertainty section exists and that its wording fits the task type (writing, question answering, summary, code, files, reusable prompt, data). If the user said to proceed without questions, the uncertainty section matters more, because the assumptions you listed are exactly the points Claude should flag if they turn out to be wrong. In the Step 5 note, if room remains within the five-sentence limit, tell the user in one sentence that the prompt allows Claude to say "I don't know" or to flag gaps, and that this lowers but does not remove the chance of errors, so important facts should still be checked. The assumptions you listed belong in the Assumptions paragraph, not in the note.
