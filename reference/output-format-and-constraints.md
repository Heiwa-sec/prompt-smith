# Output format and constraints

This file covers the `<output_format>` section of the improved prompt: how to specify the shape of the answer, how to phrase constraints so Claude follows them, how to reconcile the skill's requirement for one or two negative constraints with the advice to say what to do instead, how to match the prompt's own style to the style of output you want, and how to express length and tone in concrete terms. Read it when the user asked for a specific format, complained about bullets, markdown or length drift, or listed more than two prohibitions; otherwise the template in question-patterns-and-template.md is enough.

## Contents

1. Rules (1 to 7)
2. Reconciling positive and negative constraints
3. Before and after: a meeting-notes example
4. Apply in this skill

## Rules

### Rule 1: Specify the output format precisely

Name the format the answer should take: Markdown, plain text, JSON, a table, a code block, an email body. Name the parts it should contain and their order. Anthropic's guidance on output consistency says to define the output format precisely using JSON, XML, or custom templates, and notes that providing an example of the output you want is more effective than abstract instructions. For output that must conform to a strict JSON schema, Anthropic points to its Structured Outputs feature instead of prompt wording alone.

How to apply it: output format is one of the three topics Step 2 always covers. Turn the answer into a sentence in `<output_format>` that names the format, the sections, and where the answer begins and ends. If the user shows a sample of the output they want, reuse it as an example (see `examples-in-prompts.md`).

### Rule 2: Say what to do, then state the boundary

Anthropic's guidance says to tell Claude what to do instead of what not to do. "Do not use markdown in your response" is weaker than "Your response should be composed of smoothly flowing prose paragraphs." The author's rationale, not a quoted Anthropic claim, is practical: a positive instruction describes a target Claude can aim at, while a prohibition leaves the whole space of alternatives open.

How to apply it: for each thing the user does not want, first write the positive form of what they do want. Then, only if the boundary matters, add the short negative form. A short positive sample of the target style, a sentence or two written the way the answer should read, often works better than a prohibition; this is the author's judgment, and it fits Anthropic's advice on positive instructions and examples. The next section covers this in detail.

### Rule 3: Match the prompt's style to the output style

Anthropic's guidance notes that the formatting style of your prompt can influence the formatting style of Claude's response. If you want less Markdown in the output, remove Markdown from the prompt. A prompt full of bullets and bold headings tends to produce bullets and bold headings, and a prompt written in flowing paragraphs tends to produce flowing paragraphs; this is an aim, not a guarantee. This is the reason the skill writes every generated prompt in prose, using XML tags for structure and paragraphs for content.

How to apply it: write the content of each section in the style the user wants back. If they want a conversational essay, the prompt is a conversational paragraph. If they want a table, describe the table's columns in a sentence and show one example row. Reserve lists inside a prompt for tasks that are truly ordered procedures.

### Rule 4: Express length and tone concretely

"Short" and "professional" mean different things to different readers. Replace them with numbers and observable traits. Length: a word count, a sentence count, a paragraph count, or a bound ("under 150 words", "three paragraphs of four to five sentences"). Tone: name the register and the reader ("calm and direct, as one professional to another"; "warm and conversational, like a friendly neighbor"). Anthropic's guidance on verbosity also notes that newer models tend to be concise and conversational by default, so ask explicitly for summaries or detail when you want them. It adds that some models are more verbose by default and need an explicit request for conciseness. Per-model differences in verbosity (for instance, how verbose Opus 5 or Fable 5.1 are by default) are as summarized by the author from Anthropic's pages, not verified here: model behavior varies, so check the current docs for the model in use.

How to apply it: take the user's answer to the length and tone question and place it verbatim in concrete form. If the user gave no number, ask for one in Step 2; if they decline, list the assumed length in the Assumptions paragraph.

### Rule 5: Say where the answer starts and ends

Preambles such as "Here is your summary:" and closing offers such as "Let me know if you want changes" are common. Anthropic's guidance suggests a direct instruction to respond without preamble. Say where the output begins and what it contains, so there is nothing for Claude to add around it. Prefilled assistant responses are, as summarized by the author from Anthropic's pages and not verified here, no longer supported on the newest Claude models, so a direct instruction is the portable approach.

### Rule 6: Use XML tags to mark parts of the output when you need to extract them

If a program or the user needs to pull one part out of the answer, ask for that part inside a named tag. Anthropic's guidance suggests, for example, asking for prose sections inside a named tag. A concrete pattern for `<output_format>`: "Write the prose sections of your response inside <answer_prose> tags." Choose one tag name and reuse it. See `xml-structure.md`.

### Rule 7: When style steering still fails, write a detailed formatting paragraph

If the positive-first wording and the matched prompt style do not fix a formatting problem, Anthropic's guidance is to give detailed instructions for the formatting preference: a short paragraph saying when to use Markdown and when to use plain prose, and why. Anthropic's best-practices page also includes a ready-made block for cutting excess Markdown and bullets (tagged avoid_excessive_markdown_and_bullet_points); check the current page for its wording instead of retyping it from memory. Use this only after the simpler fixes, to keep the prompt short.

## Reconciling positive and negative constraints

The workflow's Step 3 requires output constraints that include both format and negative constraints. Anthropic's guidance prefers positive instructions. The two fit together by ordering and by size: write the positive form first, then add one or two negative constraints as short, hard boundaries, drawn from what the user said to avoid or from the failures that would cost most.

Example pairs, positive form first:

1. "Begin directly with the first paragraph of the post. Do not add commentary before or after it."
2. "Write in plain prose paragraphs. Do not use bullet points or headings."
3. "Return only the JSON object. Do not wrap it in a code fence or add explanation."
4. "Describe each item in one sentence of under 20 words. Do not add a second sentence."

Keep negative constraints short (one clause), few (one or two, never more than two), and specific (name the exact thing to avoid). When the user lists more than two prohibitions, turn the surplus into positive instructions in `<instructions>` (for example, "open with the prospect's specific situation" in place of "do not open with a generic greeting"). If a prohibition cannot be turned positive, fold the remaining ones into a single sentence that names them. Avoid shouting words in capitals; Anthropic notes that on some recent models emphatic wording such as "CRITICAL: You MUST" can cause overtriggering. Plain wording is safer.

## Before and after: a meeting-notes example

Before, as the user typed it:

```
summarize my meeting notes, keep it short and don't make it too formal
```

After. Answers given: the meeting is a weekly product-planning meeting for a team of eight; the readers are teammates who missed it; the summary goes into a team chat that does not render Markdown; it should be around 120 words. The notes in the prompt are a sample.

```
<context>
You are an experienced chief of staff. I run a weekly product-planning meeting for a team of eight. Teammates who missed it read my summary in a team chat, which displays plain text only and does not render Markdown. They want to know what was decided and who owes what, and they read it in under a minute.
</context>

<meeting_notes>
Priya: launch date moves from May 3 to May 17 because the payments review is not finished. Marco will confirm the new date with the partner by Friday. Dana asked whether the beta group should be widened; no decision. Priya will update the roadmap page next Tuesday.
</meeting_notes>

<instructions>
Summarize the meeting notes in the meeting_notes tag in about 120 words. Lead with the decisions that were made, then say who owns each follow-up and when it is due, and close with any question that is still open. Use a friendly, plain tone, the way a colleague would explain the meeting at a desk, and use first names as they appear in the notes. Before you respond, check that every owner and due date you mention appears in the notes.
</instructions>

<output_format>
Write two short paragraphs of plain text with no markup, beginning directly with the first decision and ending after the last follow-up or open question. Do not add a greeting or a title.
</output_format>

<uncertainty>
If the notes do not say who owns an action or when it is due, write "owner not stated" or "date not stated" instead of guessing, and mention it in the summary.
</uncertainty>
```

What changed: "short" became about 120 words, "not too formal" became a described register, and the chat's plain-text limitation became the stated reason for the format. The prompt itself contains no Markdown list, which supports a plain-text answer.

## Apply in this skill

In Step 2, always include the format question and fold length and tone into it or into the hard-constraints question, so the mandatory set stays at format, audience, and hard constraints. In Step 3, build `<output_format>` from the answers: format and parts first, concrete length and tone next, then one or two short negative constraints written after their positive forms. The reasoning cue is not part of this section; it is the last sentence of `<instructions>` (see `reasoning-cues.md`). Keep the prompt's own style consistent with the output style. In Step 5, if it fits within the five-sentence note, say which format and length choices came from the user and which you assumed; assumed ones also go in the Assumptions paragraph.

Related files: `instructions-and-action-verbs.md`, `xml-structure.md`, `examples-in-prompts.md`, `obsessive-specificity.md`.
