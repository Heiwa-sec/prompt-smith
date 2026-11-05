# Examples in prompts

This file covers when to include input/output examples in an improved prompt, how to make them relevant and diverse, how to wrap them in tags, how many to use, when to leave them out, and how to build them when the user has none. Read it when the user supplied sample outputs, when the format is hard to describe and an example would fix it, or when the task is a reusable templated writing prompt such as cold emails or support replies.

## Contents

1. Rules
2. Before and after
3. Apply in this skill

## Rules

### 1. Use examples to show format, tone, and structure that words describe poorly

Rule: add an example when the output has a specific shape, voice, or label set that is easier to show than to describe.

Why it works: Anthropic describes examples as one of the most reliable ways to steer Claude's output format, tone, and structure, and says they are more effective than abstract instructions for constraining output.

How to apply: ask whether a sentence of description would leave room for two reasonable outputs. If yes, an example removes that room. If the instruction alone is unambiguous, skip the example.

### 2. Make examples relevant

Rule: each example must mirror the real use case: the same kind of input, the same level of difficulty, and the same output form.

Why it works: examples set a pattern for the output, so one from a different situation points at the wrong pattern.

How to apply: build examples from the user's real material when they supplied some. Otherwise use inputs that resemble what the user described in their answers to the Step 2 questions.

### 3. Make examples diverse

Rule: when you use more than one example, vary the inputs so they cover different cases, including at least one edge case when the task has one.

Why it works: Anthropic advises that examples should cover edge cases and vary enough that Claude does not pick up unintended patterns (for instance, the same length or the same label every time, which is the author's illustration).

How to apply: if the task is classification, show different labels; when the labels outnumber the example budget, the instructions must define every label so the examples do not have to cover them all. If the task is rewriting, show a short input and a long one. If a common failure exists, such as an input with missing information, include one example that shows the correct handling.

### 4. Wrap examples in tags

Rule: put each example inside an example tag, and put several examples inside one examples tag. Inside each example, label the input and the output with their own tags.

Why it works: tags let Claude tell the examples apart from the instructions and from each other. See xml-structure.md.

How to apply: use the nesting examples, then example, then input and output. Refer to the section from the instructions: "Match the format shown in the examples tag."

### 5. Keep to the skill's budget of 1 to 3 examples

Rule: this skill uses one to three examples per improved prompt. If the user supplies more, or asks for more, use theirs.

Why it works: Anthropic's guidance often suggests 3 to 5 examples (as summarized by the author, not verified here). This skill budgets fewer as a trade-off, so that the generated prompts stay compact and easy to review; fewer examples may cover fewer cases. The skill's rule is authoritative for prompts it writes. A user with a hard or high-volume task can still ask for more, and the extra examples are valid.

How to apply: default to two examples when examples help at all. Use one when the format is simple, three when the task has distinct cases that one pair cannot cover.

### 6. Leave examples out when they over-constrain

Rule: do not add examples to open-ended creative tasks where variety is the goal, and do not add them when the task is simple.

Why it works: this is the skill's own judgment, not Anthropic guidance. Anthropic describes examples as one of the most reliable ways to steer tone and structure, so for open-ended creative work a single example can anchor every output to its structure and wording. For a simple factual question, an example adds length without adding information.

For templated business writing (cold emails, support replies), include one example only if the user supplied one or the structure is hard to describe; otherwise describe the structure in the instructions, because a single invented example can anchor every email.

How to apply: for open-ended creative work, describe the qualities wanted (voice, length, constraints) and skip the sample, or use one clearly labeled loose illustration of tone only. If the user supplies a sample, use it.

### 7. Build illustrative examples when the user has none, and label them

Rule: when examples would help but the user has none, write one or two yourself, state in the instructions that they are illustrative, and tell Claude to use them as guides to format and not as content to reuse.

Why it works: without the label, Claude may copy the example's facts, names, or phrasing into the real answer. Anthropic notes that you can also ask Claude to generate more examples from an initial set, and to evaluate them for relevance and diversity.

How to apply: invent plausible inputs that clearly differ from the real task, say in the instructions "these examples are illustrative; follow their format, not their content" (never as a loose sentence inside the examples tag), and, if it fits within the five-sentence note, mention that the examples are invented so the user can replace them with real ones.

## Before and after

Domain: classifying support tickets.

Before:

```text
sort these support tickets into categories
```

After. This is a reusable prompt, so the tickets tag is delivered empty and the user pastes the tickets into it. Answers given: the user manages a software support queue; each ticket gets exactly one of five labels (billing, bug, feature_request, account_access, unclear); a mixed ticket takes the need the customer stated first; one label per line, no explanation. The three example tickets are invented illustrations.

```text
<context>You are a support-queue triage assistant. I manage a software support queue, and each incoming ticket must be tagged with exactly one category so it reaches the right team.</context>

<tickets>
</tickets>

<instructions>Read each ticket in the tickets tag and assign exactly one category: billing, bug, feature_request, account_access, or unclear. Use unclear when the ticket lacks enough information to categorize. Choose the category that describes the customer's main need, and if a ticket mixes two needs, choose the one the customer stated first. Return one line per ticket in the format shown in the examples tag; those examples are illustrative, so follow their format, not their wording. Before you respond, check that every label you return is one of the five allowed labels and that you returned exactly one line per ticket.</instructions>

<examples>
<example>
<input>The app crashes every time I open the settings page. It would also be nice to have a dark mode.</input>
<output>bug</output>
</example>
<example>
<input>I can't log in since I changed my email address, and the reset link never arrives.</input>
<output>account_access</output>
</example>
<example>
<input>hi</input>
<output>unclear</output>
</example>
</examples>

<output_format>Return one category label per line, in the same order as the tickets. Do not add numbering, and do not add any explanation.</output_format>

<uncertainty>If a ticket does not contain enough information to categorize, return unclear instead of guessing, and do not invent details the customer did not write. If the tickets tag is empty, ask me to paste the tickets in one line and do not invent any.</uncertainty>
```

What changed: the examples show a mixed-need ticket resolved by the first-stated rule, an account problem, and the near-empty edge case, the full label set including unclear is named in the instructions, and the illustrative label stops Claude from copying the sample text. Billing and feature_request are not shown; add an example for either if the model confuses them.

## Apply in this skill

In Step 3, decide on examples after you have the audience, format, and constraints from Step 2. Include one to three when the format needs showing, none when the task is creative or trivial. Wrap them in the examples and example tags, with each example's input and output in their own tags, in prose or a single line, not bullets. If you wrote the examples yourself, label them illustrative inside the instructions and say in the Step 5 note, as a mandatory item, that they were invented so the user can replace them with real ones. In Step 4, an edge-case example showing the "unclear" or "I don't know" path is an effective way to demonstrate the uncertainty permission; see uncertainty-handling.md.
