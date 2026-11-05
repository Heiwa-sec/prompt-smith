# Instructions and action verbs

This file covers how to write the `<instructions>` section: explicit, direct instructions that begin with action verbs, state their scope, and tell Claude to act rather than only advise. Read it when the request is phrased as a hint or a question ("can you...", "could you suggest...") but the user wants work done, or when the task renames, moves, overwrites or deletes files (Rule 9).

## Contents

1. Rules (1 to 9, including agentic and file-changing tasks)
2. Before and after: a coding example
3. Apply in this skill

## Rules

### Rule 1: Lead each instruction with an action verb

Anthropic's prompting guidance says Claude responds well to clear, explicit instructions and, as summarized by the author, that recent models are trained for precise instruction following; model behavior varies. An instruction such as "Write", "Summarize", "Refactor", "Classify", or "List" tells Claude exactly what operation to perform. A noun phrase such as "a summary of the report" leaves the operation implied.

How to apply it: find the core task from Step 1 and rewrite it as a sentence whose first word is a verb. If the user wrote "a function for dates", the instruction becomes "Write a function that...". Prefer concrete verbs (rewrite, extract, compare, draft) over vague ones (handle, deal with, look at).

### Rule 2: Ask for action, not suggestions, when action is wanted

The guidance gives a direct contrast. "Can you suggest some changes to improve this function?" makes Claude propose changes. "Change this function to improve its performance" makes Claude make them. Questions phrased as politeness get answers phrased as advice.

How to apply it: decide from the intake whether the user wants the work done or wants options. If the work, use an imperative. If the user wants ideas or a plan only, say so in the same sentence: "Propose three approaches and stop; do not start implementing until I choose one." Do not guess the mode; if it is unclear, ask in Step 2.

### Rule 3: Request extra effort explicitly

Anthropic's guidance says that if you want Claude to do more than the task strictly requires, you must request it rather than rely on the model to infer it from a vague prompt. "Create an analytics dashboard" gets a basic result. "Create an analytics dashboard. Include as many relevant features and interactions as possible and go beyond the basics" gets a fuller one.

How to apply it: only add a request for extra scope if the user wants it, based on their answers. The opposite also holds: if the user wants a minimal change, say so ("Change only the date-parsing function and leave the rest of the file as it is").

### Rule 4: State the scope explicitly

Anthropic's Sonnet 5 guidance, as summarized by the author and not verified here, says that model follows instructions literally and may not silently generalize an instruction from one item to another or infer requests you did not make. Other models may differ: model behavior varies, so check Anthropic's current documentation and your own tests. If an instruction should apply broadly, say so, for example "Apply this formatting to every section, not just the first one."

How to apply it: wherever the user's request involves several items, files, sections, or cases, state whether the instruction covers all of them. Name the boundary: which files, which section, which time range.

### Rule 5: Use ordered steps only when order matters

Anthropic's guidance recommends numbered lists when the order or completeness of steps matters, and the skill's own rule is that generated prompts use flowing paragraphs except where the user's task itself requires ordered steps. A data-migration procedure or an installation guide is such a task. A request to write a report is not.

How to apply it: if the deliverable is a procedure the reader will follow in sequence, number the steps in the instructions. Otherwise express the same content as connected sentences using words like "first", "then", and "finally".

### Rule 6: Prefer a goal over a recipe when the model can plan

Anthropic's guidance notes that Claude's own reasoning frequently exceeds what a human would prescribe. Tell Claude what a good result looks like and let it choose the method, unless the user requires a particular method. See `reasoning-cues.md`.

### Rule 7: Keep each deliverable in its own paragraph

If the request bundles unrelated deliverables, give each its own paragraph and say how the outputs should be separated. Suggest splitting the request into separate prompts only when the user wants to inspect or reuse each intermediate output.

### Rule 8: Agentic and repo-level tasks (Claude Code)

When the task is a change across a repository or a multi-step job that Claude will carry out with tools, the instructions need four things that a single-function request does not. This rule draws on Anthropic's agentic prompting guidance as summarized by the author; check the current docs. Write each as a sentence of prose inside `<instructions>`, only when the user's answers support it.

Acting rather than suggesting: say plainly whether Claude should make the changes or only propose them ("Make the changes in the repository; do not only describe them"). Stop condition: say when to keep going and when to stop and report ("Keep working until the tests pass, and stop and report back if a step would delete data or needs a decision I have not given you"). Minimal scope: say that Claude should not add features, tests, files, or documentation that were not asked for, and should change only what the task needs. Verification: name the check Claude should run before it reports, such as a test command the user supplied; if the user gave none, do not invent one. Confirmation before destructive actions: tell Claude to ask before any action that is hard to reverse, such as deleting files, force-pushing, or dropping data.

### Rule 9: Scripts that change or delete files

When the task is a script, command, or repo change that renames, moves, overwrites, or deletes files, in any environment and not only in Claude Code, add a safe default in `<instructions>`, not in `<uncertainty>`, unless the user's answers set a different rule: "Make the script do a dry run by default, never overwrite or delete an existing file unless I pass an explicit apply or overwrite option, and tell me if a name collision could occur." This is this skill's own safety default, not Anthropic guidance. If the user did not confirm it (they skipped the files question, answered vaguely, or told you to proceed without questions), list it in the Assumptions paragraph, for example "I assumed the script does a dry run by default and never overwrites or deletes a file (this skill's own safety default), so remove that if you want otherwise." For a direct file action instead of a script, the variant is to list the planned changes and ask before applying them. The `<uncertainty>` section keeps the usual code-task permission (say what you cannot tell and name any assumption you made) and also tells Claude to flag a possible name collision instead of choosing silently. Step 2 should have asked whether the task touches real files and how (see question-patterns-and-template.md).

## Before and after: a coding example

Before, as the user typed it:

```
can you look at this function that parses dates and see if it could be better
```

After. Answers given: the code is Python 3.11; parse_date runs on every row of user-supplied CSV files, some with more than a million rows, and the dates arrive in inconsistent formats; the signature must stay the same; the function should be faster and should also accept dates written like 2024/03/05; the user wants a two or three sentence explanation of what made the original slow and what changed.

```
<context>
You are a senior Python engineer. I maintain a Python 3.11 data-import tool. The function parse_date below runs on every row of user-supplied CSV files, some with more than a million rows, and it is slow. The dates arrive in inconsistent formats. Other modules call parse_date by name, so its signature must stay the same.
</context>

<code>
def parse_date(text):
    for fmt in ("%Y-%m-%d", "%d/%m/%Y", "%m/%d/%y"):
        try:
            return datetime.strptime(text.strip(), fmt).date()
        except ValueError:
            continue
    return None
</code>

<instructions>
Rewrite parse_date from the code tag so that it runs faster on large inputs and also accepts dates written like 2024/03/05. Keep the function name, parameters, and return type exactly as they are. Apply the change to this one function only and leave every other function in the file untouched. After the rewrite, explain in two or three sentences what made the original slow and what you changed. Before you respond, check that the function name, parameters, and return type match the original and that every format the original accepted still parses.
</instructions>

<output_format>
Return the rewritten function in a single Python code block, followed by the short explanation in plain prose. Do not rewrite any other function, and add no commentary before the code block.
</output_format>

<uncertainty>
If the original function relies on behavior you cannot determine from the code shown, say what is unclear and which assumption you made, rather than guessing.
</uncertainty>
```

What changed: a question ("could it be better") became an imperative with a measurable meaning of better; the scope was fixed to one function; the new date format was named instead of left implicit; the request for an explanation was bounded to two or three sentences; and the closing check names what to compare against the original. The instructions use paragraphs because the task is not a sequential procedure.

## Apply in this skill

In Step 1, mark every place the request uses a question form or a noun phrase where the user wants action. In Step 2, ask a goal question when "better" or "improve" has no measurable meaning (for example, "What would make the result good enough for you?"). In Step 3, write the `<instructions>` section as imperative sentences with explicit scope, ending with the reasoning cue. Keep numbered steps only if the user's task is itself a procedure. For a task that renames, moves, overwrites, or deletes files, apply Rule 9 in Step 3 and list the default under Assumptions when the user did not confirm it. In Step 5, if room remains in the five-sentence note, mention which vague phrases you replaced with specific verbs and scope.

Related files: `obsessive-specificity.md`, `output-format-and-constraints.md`, `reasoning-cues.md`, `role-and-context.md`.
