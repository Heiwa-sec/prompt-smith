# Role and context

This file covers how to give Claude a role and how to supply the context a prompt needs: who is asking, who the output is for, why the task exists, and what background Claude cannot see. Read it when the user could not say who they are, who reads the result, or why the task exists, and the role and context need real work.

## Contents

1. Rules
2. Before and after: a support-team example
3. Apply in this skill

## Rules

### Rule 1: Open the prompt with a one-sentence role

State who Claude should act as, in the specific field the task belongs to, as the first sentence of `<context>`. Anthropic's prompting guidance says a role set in the system prompt focuses Claude's behavior and tone, and that even a single sentence makes a difference. Anthropic's guidance is about system prompts. The prompts this skill writes are pasted as a user message, and the skill applies the same one-sentence role there by extension. The role does not need a biography. "You are a senior customer-support writer for a B2B software company" is more specific than "You are a helpful assistant"; the author's rationale, not Anthropic's claim, is that a specific role narrows vocabulary, tone, and judgment.

How to apply it: pick the role from the profession that would normally do the task, add the domain, and stop at one sentence unless the task is a character or persona. For persona work, Anthropic's consistency guidance recommends detailed personality, background, and traits, plus common scenarios with expected responses. Write the scenarios and expected responses as connected sentences, not a bulleted list. Only expand the role that far when the user's task is a persona.

### Rule 2: Describe the situation, not only the task

A bare task leaves Claude to guess the setting. Treat Claude as a brilliant new colleague who lacks context on your norms and workflows. Write down who the user is, what the work feeds into, and what already exists. Put this in a `<context>` section so it stays separate from the instructions.

How to apply it: take every fact the user gave you in the intake and clarifying answers that describes the situation (their business, their reader, the decision the output supports) and move it into `<context>`. Do not leave situational facts scattered through the instructions.

### Rule 3: Explain the why behind a requirement

Anthropic's guidance notes that giving the motivation behind an instruction lets Claude generalize from it. "Never use ellipses" is a rule Claude must follow blindly. "The reply will be read aloud by a text-to-speech engine, so avoid ellipses because it cannot pronounce them" lets Claude generalize from the reason; for instance, it may also avoid other characters a speech engine cannot read (the author's illustration, not Anthropic's example).

How to apply it: for each hard requirement the user states, ask whether a short reason exists. If the user gave a reason, include it. If they did not and the reason is not obvious, ask in Step 2 ("What will the output be used for?") instead of inventing one.

### Rule 4: Name the audience and their starting knowledge

The audience decides vocabulary, depth, and what can be skipped. Say who reads the output and what they already know. "Readers are finance managers who know accounting terms but have not seen this tool" is concrete. "Business users" is not.

How to apply it: audience is one of the three questions Step 2 always covers. If the user answers vaguely, write the audience in the prompt as exactly as their answer allows and add an assumption to the Assumptions paragraph rather than sharpening it with invented detail.

### Rule 5: Supply the material, do not assume Claude has it

Claude cannot see the user's files, previous conversations, or company policies unless they are in the prompt. If the task depends on a document, data set, or policy, either tell the user to attach it or include it. When the material is long, follow `long-context.md` for placement.

How to apply it: if the request refers to "the report" or "our style guide" and nothing is attached, ask for it in Step 2, or in Step 3 write the uncertainty permission so Claude flags the missing material instead of guessing (Step 4 then checks that it does). See `uncertainty-handling.md`.

### Rule 6: Keep context relevant and short

Context earns its place only if it changes the output. Drop company history, the user's full story, and anything Claude would not act on. Longer is not better here; the test is whether removing a sentence would change the result.

## Before and after: a support-team example

Before, as the user typed it:

```
reply to this angry customer about the late shipment
```

After, as the improved prompt would read once the user answered the clarifying questions. Answers given: the store sells office supplies to small businesses; the order shipped four days later than promised and is now due next Tuesday; the customer is a small-business owner; agents may offer free expedited shipping on the next order; no refund over 50 dollars without manager approval; the reply is an email under 150 words in a calm, direct tone, signed by the support team.

```
<context>
You are a senior customer-support writer for an online store that sells office supplies to small businesses. I lead the support team, and a customer has written in angry because their order shipped four days later than promised. The reply goes out signed by my support team, and the customer is a small-business owner. Our policy lets agents offer free expedited shipping on the next order, but any refund over 50 dollars needs a manager's approval, so the reply should not promise one.
</context>

<customer_message>
Order 4471 was promised for Monday and it still has not arrived. I run a print shop and I am about to miss a client deadline because of this. I want to know what you are going to do about it.
</customer_message>

<instructions>
Write a reply to the customer's message in the customer_message tag that acknowledges the delay once, in plain words, states that the order shipped four days later than promised and is now due next Tuesday, and offers free expedited shipping on their next order. Keep the reply under 150 words and write it in a calm, direct tone, as one professional to another. Before you respond, check that the reply stays under 150 words and makes no promise of a refund.
</instructions>

<output_format>
Return only the email body, beginning with the greeting and ending with the sign-off from the support team, in short paragraphs of plain text. Do not promise a refund, and add no commentary outside the email.
</output_format>

<uncertainty>
If the customer's message leaves out the order number, say what is missing and ask for it in the reply instead of inventing one. If you are unsure whether a promise is within the policy above, leave it out.
</uncertainty>
```

What changed: the role, the situation, the shipping facts, and the policy limit moved into `<context>` and the instructions, with the role first; the pasted message sits above the instructions; a verification sentence closes the instructions; the audience and tone became concrete; and the missing-information case got explicit permission. The text inside `<customer_message>` is the user's own pasted material, shown here as a sample.

## Apply in this skill

In Step 1, note which of role, situation, reason, audience, and material the request is missing. In Step 2, turn the largest gaps into one-line questions. In Step 3, open `<context>` with the one-sentence role, and fill `<context>` only with facts the user actually gave. In Step 3, tie any missing material to the `<uncertainty>` section, and in Step 4 check that it does. Never invent context the user did not provide; if the user chose to proceed without questions, list the context you assumed in the Assumptions paragraph that follows the prompt.

Related files: `instructions-and-action-verbs.md`, `obsessive-specificity.md`, `uncertainty-handling.md`, `long-context.md`.
