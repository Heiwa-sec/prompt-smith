# Obsessive specificity

This file covers how to remove ambiguity from a rough prompt by replacing vague words with numbers, names, scope, audience, and a definition of done. Read it when the request uses vague quantity or quality words or lacks names, scope, or a definition of done.

## Contents

1. Rules
2. Before and after
3. Apply in this skill

## Rules

### 1. Replace every vague quality word with something measurable

Rule: words like "short", "good", "professional", "soon", "a few", and "detailed" must become a number, a range, or a named reference.

Why it works: Anthropic's guidance describes Claude as a brilliant new employee who lacks context on your norms. A new employee given "write a short reply" has to guess what short means. A stated length removes the guess. The test Anthropic suggests is to show the prompt to a colleague with minimal context; if they would be confused, Claude will be too.

How to apply: scan the user's draft for adjectives and quantifiers. For each one, take the number or reference from the user's answer to a clarifying question. If the user gave no answer, either ask in Step 2 or list your assumption in the Assumptions paragraph that follows the prompt.

### 2. Name the things instead of pointing at them

Rule: write the actual product, file, customer segment, library, or document name wherever the draft says "it", "this", "the thing", or "my project".

Why it works: Claude sees only what is in the prompt. Pronouns that referred to something in the user's head refer to nothing for Claude.

How to apply: expand every pronoun and shorthand. If the referenced thing is long, such as a document, include it in the prompt inside its own tag (see xml-structure.md) and refer to it by that tag name.

### 3. State the scope explicitly, including how far an instruction reaches

Rule: say which items an instruction covers and which it does not. If a rule applies to every item, say "every" and name the set.

Why it works: Anthropic's Sonnet 5 guidance, as summarized by the author and not verified here, says that model follows instructions literally and does not silently generalize an instruction from one item to another; model behavior varies, so check current docs. A formatting rule written for the first section may not be applied to the rest unless the prompt says so. With a literal reader, a vague threshold such as "only report important issues" can be followed faithfully and lower recall.

How to apply: write "apply this to every section, not just the first" when that is the intent. For filters, give a concrete threshold ("report any issue that could cause data loss or an incorrect result") instead of a vague word like "important".

### 4. Name the audience and what they already know

Rule: state who will read or use the output, their level of expertise, and what they will do with it.

Why it works: audience determines vocabulary, depth, and what can be skipped. It also supplies motivation, and Anthropic observes that Claude generalizes from an explanation of why something matters.

How to apply: take the audience from the answer to the audience question in Step 2. Put it in the context section, for example "The readers are support agents with two weeks of experience who will paste this into a customer reply."

### 5. Write a definition of done

Rule: end the instructions with a testable description of a complete answer (how many items, what must be present, what must be absent, what format it ends in), placed just before the reasoning cue, which stays the last sentence.

Why it works: a definition of done gives Claude something to check against, which pairs with the verification cue described in reasoning-cues.md.

How to apply: ask "how will you know this output is right?" in Step 2 when the goal is unclear. In Step 3, convert the answer into one or two sentences near the end of the instructions, followed by the reasoning cue.

### 6. Ask for more only when you mean it

Rule: if the user wants depth or extras beyond the literal task, request them explicitly.

Why it works: Anthropic's guidance says that to get more than the task strictly requires, you must ask for it rather than rely on the model to infer it from a vague prompt.

How to apply: if the user's draft says "make a dashboard" and the answers show they want filters and charts, write those into the instructions by name.

## Before and after

Domain: customer support.

Before (the user's rough prompt):

```text
reply to this customer who is mad about their order being late, keep it short and nice
```

After (the improved prompt, with specifics filled in from the user's answers). Answers given: the store sells ceramic cookware and agents paste replies into email; the customer is Dana Whitfield, who ordered a 10-inch skillet on May 2, promised by May 9, not arrived as of May 14; the order number is 48213 and the carrier shows it delayed at a regional hub with a new estimated delivery of May 18; offer a full refund or a free expedited replacement; under 90 words, warm and plain, signed as the Support Team. The customer message is a sample.

```text
<context>You are a customer-support writer for an online store that sells ceramic cookware. Our agents paste replies into email. The customer, Dana Whitfield, ordered a 10-inch skillet on May 2, and it was promised by May 9 but has not arrived as of May 14. Her message is in the customer_message tag below.</context>

<customer_message>Your site said delivery by May 9. It is May 14 and nothing. I want to know where my order is or I want my money back.</customer_message>

<instructions>Write an email reply to Dana from the support team. Open by apologizing once, in a single sentence, for the delay. Tell her the order number is 48213 and that the carrier shows it as delayed at a regional hub, with a new estimated delivery of May 18. Offer a full refund or a free expedited replacement, whichever she prefers, and ask her to reply with her choice. Keep the whole reply under 90 words, use a warm and plain tone, and sign off as the Support Team. The reply is complete when it contains the apology, the new date, both options, and the request for her choice. Before you respond, check that each of those four elements is present and that the reply stays under 90 words.</instructions>

<output_format>Return the email body as plain text, beginning with the greeting and ending with the sign-off. Do not add a subject line or any commentary around the email, with one exception: if a fact is missing, write a single line before the email that begins "Missing:" and names it.</output_format>

<uncertainty>If any fact in this request is missing or unclear, such as the order number or the carrier status, do not invent it; name it in the single "Missing:" line before the email and leave it out of the email.</uncertainty>
```

What changed: "late" became dates, "short" became under 90 words, "nice" became a tone plus a single apology, and the unstated goal became a list of required elements written as a sentence.

## Apply in this skill

In Step 2, use this file to choose questions: any vague word in the draft (length, tone, deadline, quantity) is a candidate for a one-line question, and the three mandatory topics (output format, audience, and hard constraints) are always included. In Step 3, put the resolved numbers and names into the context and instructions sections. In Step 4, the uncertainty section covers the specifics the user could not supply, so Claude flags them instead of guessing. When the user told you to proceed without questions, fill the gaps with explicit assumptions and list each one in the Assumptions paragraph that follows the final prompt.
