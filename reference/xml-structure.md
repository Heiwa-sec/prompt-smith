# XML structure

This file covers how to wrap the parts of an improved prompt in XML tags: which tags this skill uses, how to name them, when to nest them, and how to refer to them from the instructions. Read it when the prompt has two or more input tags or nested tags, or when the user asks why the prompt contains tags; otherwise the template in question-patterns-and-template.md is enough.

## Contents

1. Rules
2. Standard tags used by this skill
3. Before and after
4. Apply in this skill

## Rules

### 1. Give each kind of content its own tag

Rule: wrap context, instructions, examples, input data, and output requirements in separate tags rather than mixing them in one block of text.

Why it works: Anthropic's guidance says XML tags help Claude parse complex prompts unambiguously, especially when a prompt mixes instructions, context, examples, and variable input. Wrapping each type of content in its own tag can reduce misinterpretation, for instance a pasted email being read as an instruction.

How to apply: after drafting the content of each section, put it inside the matching tag from the standard list below. Anything the user pasted in, such as an email, a log, or an article, goes in its own tag and never inside the instructions.

### 2. Use consistent, meaningful tag names

Rule: choose descriptive names, and use the same name every time the same kind of content appears, within one prompt and across prompts you write for the same user.

Why it works: Anthropic recommends consistent, descriptive tag names. Anthropic's docs do not publish a canonical list of tag names, so there is no required vocabulary. The point is that the name tells Claude, and the next human reader, what the content is. A tag called <data1> says nothing; <customer_message> says what it holds.

How to apply: prefer the standard tags below for structural sections. For user content, name the tag after what the content is, in lowercase with underscores, such as <bug_report> or <meeting_transcript>. Do not rename a tag partway through a prompt.

### 3. Nest tags only when the content has a natural hierarchy

Rule: nest when one thing contains several of another thing, such as documents inside a documents tag, or examples inside an examples tag. Otherwise keep tags flat.

Why it works: nesting mirrors real structure, which helps Claude tell where each item starts and stops. Extra nesting with no real hierarchy adds noise. The docs give no numeric limit on nesting depth, so treat two levels as a practical guide and not as a rule from Anthropic.

How to apply: use <examples> containing one or more <example> tags. For multiple documents use <documents>, each inside <document index="1"> with <source> and <document_content> subtags, as described in long-context.md.

### 4. Refer to tags by name in the instructions

Rule: when the instructions rely on a section, name it: "Using the contract in the contract tag, ..." or "Follow the format in the output_format tag."

Why it works: it ties the instruction to the exact content and removes the chance that Claude treats the wrong block as the subject.

How to apply: after writing the instructions, check that each tag that holds input data is mentioned by name at least once.

### 5. Do not use a tag as a place for stray text

Rule: text that belongs to no section should either be placed into the right section or removed.

Why it works: untagged fragments recreate the ambiguity the tags were meant to remove.

How to apply: after assembling, read the prompt once from top to bottom and confirm every sentence sits inside a tag, or is a short framing sentence the user explicitly wants.

### 6. Match the prose style inside the tags to the output you want

Rule: tags give structure, but the text inside them should still be flowing paragraphs in the improved prompts this skill generates, with ordered steps only where the user's task itself needs them.

Why it works: Anthropic notes that the formatting style of a prompt can influence the style of the response. Prose inside the tags can steer Claude toward prose output. See output-format-and-constraints.md.

How to apply: write each section as a short paragraph of plain sentences. Do not turn a section into a bullet list.

## Standard tags used by this skill

The structural tags below are this skill's own convention, chosen for consistency. They are not an Anthropic-mandated list.

Core tags, used in most improved prompts:

- context: a one-sentence role for Claude first, then who the user is, the situation, the audience, and why the task matters
- instructions: what Claude must do, led by action verbs
- examples and example: one to three input/output samples, each in an example tag inside an examples tag
- output_format: the exact shape of the answer first, then one or two short negative constraints
- uncertainty: the permission to say "I don't know" or to flag missing information (Step 4, detailed in uncertainty-handling.md)

Optional tags, added when the task needs them:

- documents, document, source, document_content: long input material, with an index attribute on each document (long-context.md)
- quotes: word-for-word extracts that the answer must be grounded in
- input: variable data the instructions operate on, when no more specific name fits
- answer: the final result, when the user needs it separated from other text so it can be extracted. Do not add a `thinking` tag that asks Claude to write its reasoning out: Anthropic documents that this may be declined on its newest models (see reasoning-cues.md). Prefer a plain verification cue in the instructions.

## Before and after

Domain: contract review.

Before:

```text
Here is a vendor contract: Term. This agreement renews automatically for successive 12-month terms unless cancelled 90 days before renewal. Fees. Early termination requires payment of 50 percent of the remaining fees. Tell me what is risky. Be careful. I'm a small business owner, not a lawyer.
```

After. Answers given: the user owns a small landscaping company, is not a lawyer, and is about to sign this vendor agreement; the output is a plain-language risk review for the user alone, under 500 words. The contract text is a short sample, and it is identical before and after.

```text
<context>You are a contracts analyst who explains agreements to non-lawyers. I own a small landscaping company and I am not a lawyer. I am about to sign the vendor agreement in the contract tag below and I need to know which clauses could cost me money or limit my options.</context>

<contract>
Term. This agreement renews automatically for successive 12-month terms unless cancelled 90 days before renewal. Fees. Early termination requires payment of 50 percent of the remaining fees.
</contract>

<instructions>Read the agreement in the contract tag and identify the clauses that create financial or legal risk for my company, including the automatic renewal and the early termination fee. Write one short paragraph per clause that names the clause, quotes the exact sentence that creates the risk, and explains in plain language what it could cost me. Finish with a single paragraph that tells me which clause I should ask the vendor to change first and why. Before you respond, check that every quote you give appears word for word in the contract.</instructions>

<output_format>Write in plain paragraphs at a level a non-lawyer can follow, and keep the whole answer under 500 words. Do not use headings, and do not add legal citations.</output_format>

<uncertainty>If a clause is ambiguous or you cannot tell what it means, say so and quote it, rather than guessing. This is an analysis and not legal advice, so state plainly where I should ask a lawyer.</uncertainty>
```

What changed: the pasted contract moved into its own tag so it cannot be mistaken for an instruction, the vague "be careful" became the two clauses to examine and a quoting requirement, and the user's role became context.

## Apply in this skill

In Step 3, assemble the prompt in the one order used across this skill: pasted input longer than about 500 words first, in named document tags (see long-context.md), then context, then shorter input in named tags (like the contract tag above), then instructions, then examples if used, then output_format, then uncertainty. The template is in question-patterns-and-template.md. In Step 4, the uncertainty tag is mandatory in every improved prompt. In Step 5, show the full tagged prompt inside a four-backtick fenced block so the user can paste it (long-context.md sets when a very long document is not reprinted), and if room remains in the five-sentence change note, mention that the tags are there to separate content types and not as decoration.
