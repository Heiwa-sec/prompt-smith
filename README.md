# prompt-smith

prompt-smith is a Claude Skill that aims to turn a rough prompt into a stronger one. You give it a half-formed request, it asks a few short clarifying questions, and it then writes one optimized prompt built from Anthropic's published prompting guidance: a clear role and context, direct instructions, output constraints, a verification cue, and explicit permission to say "I don't know". The finished prompt is sent to Claude as your actual request, so you get a result and a prompt you can reuse. It is an aim, not a guarantee that the result is better. It is written for Claude Desktop and Claude Code.

## Contents

- [How it works](#how-it-works)
- [Install for Claude Desktop](#install-for-claude-desktop)
- [Install for Claude Code](#install-for-claude-code)
- [Repo layout](#repo-layout)
- [Usage](#usage)
- [How to trigger it](#how-to-trigger-it)
- [Customization](#customization)
- [Troubleshooting](#troubleshooting)
- [Known uncertainties](#known-uncertainties)
- [Limitations](#limitations)
- [Security](#security)
- [Attribution and license](#attribution-and-license)

## How it works

The skill follows five steps. The full rules live in `SKILL.md`.

1. Read the request. Identify the core task, the implied audience, the expected output, and what context is missing.
2. Ask clarifying questions. Ask 3 to 6 numbered questions, each answerable in one line, always covering desired output format, intended audience, and hard constraints, plus goal-related questions when the goal is unclear. The skill never skips this step, even when the request looks complete. If you tell it to proceed without questions, or answer only some of them, it proceeds and lists its assumptions explicitly in an Assumptions paragraph.
3. Write the improved prompt. It builds the prompt with role and context, specific instructions that start with action verbs and end with a reasoning cue such as "verify your answer before responding", 1 to 3 input/output examples when they help, and output constraints (the format first, then one or two negative constraints), with each section wrapped in XML tags such as `<context>`, `<instructions>`, `<examples>`, and `<output_format>`. The prompt is written as flowing paragraphs, not bullet points, unless your task itself needs ordered steps. The exact rules, including how large pasted documents are placed and the dry-run default for file-changing tasks, are defined in `SKILL.md`.
4. Build in uncertainty permission. Every improved prompt explicitly allows Claude to say "I don't know" or to flag missing information rather than guess. The skill reads the uncertainty guidance together with the question patterns before writing the prompt in Step 3, and this step checks that the section fits the task type.
5. Return the result. It shows the final prompt in a fenced block, ready to paste, then an Assumptions paragraph only when it had to assume something, then a short note on what changed and why. It then sends that prompt to Claude as your actual request, executes it, and ends with one closing line inviting you to say what was off. Execution is the default, with exactly two exceptions: you asked for the prompt only, or it is a reusable or system prompt with no input supplied. The exact order, the priority rules for the note, and the large-document handling are defined once, in Step 5 of `SKILL.md`.

## Install for Claude Desktop

Anthropic's Help Center documents ZIP upload for claude.ai and does not mention the Claude Desktop app separately. This section assumes Claude Desktop follows claude.ai, which is unverified.

The skill is a folder, and the documented upload takes it as a ZIP file. The folder must be the top-level entry inside the ZIP, so the ZIP contains `prompt-smith/SKILL.md`, not a bare `SKILL.md` at the ZIP root.

Before you start, read the caveat at the end of this section. Menu names, plan requirements, and the upload flow can differ by app version and plan.

1. Get the folder. Either clone the repo (this works only once the repo has been published under heiwa-sec) or use a copy you already have.

   ```
   git clone https://github.com/heiwa-sec/prompt-smith.git ~/projects/prompt-smith
   ```

2. Zip it from the parent directory so the folder name is the first path segment in the archive. The `-X` option leaves out extra file attributes such as owner IDs, and the `-x` options keep macOS metadata, git history and the repo's CI files out of the ZIP.

   ```
   cd ~/projects
   zip -rX prompt-smith.zip prompt-smith -x "*.DS_Store" -x "prompt-smith/.git/*" -x "prompt-smith/.github/*"
   ```

3. Check the ZIP. Every listed path should start with `prompt-smith/`, and `prompt-smith/SKILL.md` should be present.

   ```
   unzip -l prompt-smith.zip
   ```

4. Make sure code execution is on. Anthropic's Help Center says skills need it, and that it is under Settings > Capabilities (labelled "Code execution and file creation" in some versions and "Create and edit files" in the updated layout). On Team and Enterprise plans, the Help Center is reported to say an owner must enable code execution first (and, on Enterprise, skills), under Organization settings > Plugins & skills > Policy tab. The exact toggle names were not confirmed.

5. Upload the ZIP. Anthropic's Help Center describes this path for claude.ai: Customize > Skills, then the "+" button, then "+ Create skill", then "Upload a skill". The older platform overview page still says Settings > Features, which looks stale. Any other menu path is unconfirmed, so use whichever menu your app version actually shows. Note that the `description` in this skill's `SKILL.md` is longer than 200 characters. A summary of the Help Center reported a 200-character limit (not read verbatim), while the platform docs state 1,024. If the upload is rejected over the description, shorten the description to 200 characters or fewer (dropping the trigger phrases) and zip again.

6. Turn the skill on. The Help Center says to toggle it on in the skills list after upload. Then start a new chat and try one of the phrasings under [How to trigger it](#how-to-trigger-it).

What was verified and what was not:

- Documented for claude.ai in Anthropic's docs, read but not tested in the app and not confirmed for Claude Desktop: the ZIP layout with the skill folder as the archive's top-level entry (this matches how Anthropic's own packaging script builds archives), the requirement for code execution, and the Customize > Skills upload flow as the currently documented one.
- Not verified: that the menus above match your version of Claude Desktop, that your plan includes skills, and that the uploader accepts this ZIP unchanged. The author has not uploaded this skill to a live Claude Desktop or claude.ai account. Confirm every step against Anthropic's current documentation before relying on it:
  - https://support.claude.com/en/articles/12512198-how-to-create-custom-skills
  - https://support.claude.com/en/articles/12512180-use-skills-in-claude
  - https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview

One more point from the docs that matters here: Anthropic's docs say custom skills do not sync across claude.ai, the API, and Claude Code, with one documented exception: signed-in Claude Code downloads claude.ai skills to `~/.claude/skills/synced/` (download only, and the folder name is reserved; stated in the Claude Code docs, not run against a live Claude Code). Install the skill separately for Claude Code. Whether the Desktop app shares skills with claude.ai is unverified.

## Install for Claude Code

Claude Code reads skills from folders. The folder name should match the `name` field, which is `prompt-smith`.

Personal install (available in all your projects), by copying a local folder:

```
mkdir -p ~/.claude/skills/prompt-smith
cp -R ~/projects/prompt-smith/. ~/.claude/skills/prompt-smith/
rm -rf ~/.claude/skills/prompt-smith/.git ~/.claude/skills/prompt-smith/.github
```

The trailing `/.` copies the folder contents into the target, so running the commands again replaces the files instead of nesting a second folder. The last line removes the copied git history and the repo's CI files, which are not part of the skill. Skip it if the folder you copied has neither.

Personal install by cloning directly into the skills directory. This works only once the repo has been published under heiwa-sec, and it fails if the target folder already exists and is not empty. The clone leaves a `.git` folder inside the skill folder, which is harmless but unnecessary.

```
git clone https://github.com/heiwa-sec/prompt-smith.git ~/.claude/skills/prompt-smith
```

Project install (shared with everyone who uses the repo, if you commit it). Run these from the root of your project, after cloning or otherwise obtaining the skill folder at `~/projects/prompt-smith`:

```
mkdir -p .claude/skills/prompt-smith
cp -R ~/projects/prompt-smith/. .claude/skills/prompt-smith/
rm -rf .claude/skills/prompt-smith/.git .claude/skills/prompt-smith/.github
```

The last line removes the copied git history so the skill does not become a nested repository inside your project, and removes the CI files. Skip it if the folder you copied has neither.

Check the install:

```
ls ~/.claude/skills/prompt-smith/SKILL.md
```

What Anthropic's Claude Code docs say (https://code.claude.com/docs/en/skills), as summarized when this README was written. Everything below is stated in the Claude Code docs and was not run against a live Claude Code; see the list after this one:

- Skills load in two ways. Claude can load one automatically when your request matches its description, and you can invoke it directly as a slash command named after the skill, here `/prompt-smith`.
- Claude Code watches the skill directories, so adding or editing a skill under `~/.claude/skills/` or a project `.claude/skills/` should be picked up in the current session without a restart. The documented exception is a top-level skills directory that did not exist when the session started. In that case run `/reload-skills`.
- If you are signed in with a claude.ai account, the docs say Claude Code downloads your claude.ai skills into `~/.claude/skills/synced/`. That is download only, edits there can be overwritten, and the folder name `synced` is reserved, so do not install this skill there by hand.
- The docs describe `/doctor` as a checkup of your Claude Code setup.

What was not verified:

- The author has not run this skill in Claude Code. Everything above comes from reading the docs.
- The `/reload-skills` command, the `/doctor` command, the file-watching behavior, and the `~/.claude/skills/synced/` download behavior. They are stated in the Claude Code docs and were not run against a live Claude Code.
- Whether text typed after `/prompt-smith` on the same line is passed to the skill as your rough prompt. The Claude Code skills docs say arguments are appended as an ARGUMENTS line holding the typed text when a skill has no placeholder; this is stated in the docs and was not run against a live Claude Code. If it is not picked up, send the rough prompt as your next message.
- Behavior on Claude Code versions other than the documentation as it stood when this repo was written.

## Repo layout

The repo root is the skill folder. `SKILL.md` and `reference/` are what the skill needs; the other files are project housekeeping. There is exactly one `SKILL.md`.

```
github.com/heiwa-sec/prompt-smith
├── SKILL.md                                   skill entry point: frontmatter, the 5-step workflow, index of reference files, worked example
├── README.md                                  install steps, usage, troubleshooting, attribution (this file)
├── SECURITY.md                                how to verify downloads, built-in protections, private vulnerability reporting
├── LICENSE                                    MIT License
├── .gitignore                                 keeps archives, keys and OS files out of the repo
├── .github/                                   CI integrity check, CODEOWNERS, Dependabot (not part of the skill)
└── reference/
    ├── role-and-context.md                    giving Claude a role and the context a prompt needs
    ├── instructions-and-action-verbs.md       explicit, direct instructions led by action verbs
    ├── output-format-and-constraints.md       output format and constraints; say what to do, match prompt style to output style
    ├── reasoning-cues.md                      reasoning and verification cues, when to ask for step-by-step thinking
    ├── obsessive-specificity.md               numbers, names, scope, audience, definitions of done
    ├── xml-structure.md                       XML tags for sections, naming, nesting, consistency
    ├── examples-in-prompts.md                 relevant, diverse examples wrapped in example and examples tags
    ├── uncertainty-handling.md                permission to say "I don't know", flagging missing info, quote-grounding
    ├── iteration.md                           refining the improved prompt after the first run
    ├── long-context.md                        long documents first, instructions after them, quote-grounded answers
    ├── question-patterns-and-template.md      clarifying-question patterns and the improved-prompt template
    └── worked-examples.md                     three worked examples: blog writing, coding, analysis
```

`SKILL.md` is the only file Claude loads on trigger. The files in `reference/` are linked from it and are read only when needed.

## Usage

Each example below shows the same shape: a rough prompt, the skill's questions, your one-line answers, then the improved prompt and a short note. In real use the prompt arrives in a fenced block, and after the note Claude treats the improved prompt as your request and carries it out, then ends with one line inviting you to say what was off, set off after a horizontal rule. To keep the transcripts short, the fence, the execution output and the closing line are omitted below. The improved prompts are in prose with XML tags, because prompt style can influence answer style, and matching the two is how the author tries to get prose answers.

### Example 1: customer support reply

```
You: reply to angry customer about late order

Claude: A few quick questions before I write the prompt.
1. What happened (how late, and the cause)?
2. What can you offer (a new delivery date, a discount, nothing)?
3. Who is the reader (first-time buyer, repeat customer)?
4. What output format and length do you need (email body only, or with a subject line; under 150 words)?
5. Any hard constraints (calm tone, no refund promises, a required sign-off)?

You:
1. Five days late because of a warehouse delay.
2. New delivery Friday, 15 percent off their next order.
3. Repeat customer, this is their second message about it.
4. Plain text with a subject line, under 150 words.
5. Calm and apologetic. Do not promise a refund. Sign off as Fernwood Goods Support Team.

Claude:

<context>You are a customer support writer for Fernwood Goods, an online shop. You are replying to a repeat customer whose order is five days late because of a warehouse delay. This is their second message about the problem.</context>

<instructions>Write a reply email that apologizes specifically, explains the cause in one sentence, gives Friday as the new delivery date, and offers 15 percent off their next order as an apology. Acknowledge that this is their second message. Keep the tone calm and apologetic, and keep the body under 150 words. Before you respond, check that the email includes the new date and the discount and stays under the word limit.</instructions>

<output_format>Return plain text with a subject line first, then the email body, signed "Fernwood Goods Support Team". Do not promise a refund. Apart from the missing-information note described below, add no commentary before or after the email.</output_format>

<uncertainty>If a detail the email needs is missing, such as the customer's name or an order number, leave it out of the email and name what is missing in one line after the email. Do not invent details.</uncertainty>

What changed and why: your three-word request became a prompt with the facts, the reader, a word limit, and a fixed format. I added a check step so the date and discount are not forgotten, and permission to flag gaps so the email does not contain invented order details.
```

### Example 2: SQL query

```
You: write a query to find my best customers

Claude: A few quick questions before I write the prompt.
1. What is the schema (database, tables, columns)?
2. How do you define "best" (total spend, order count, recent activity)?
3. What time window and result size (for example, last 12 months, top 20)?
4. Who will use this query (you, an analyst, a dashboard)?
5. What output format do you want (SQL only, or SQL plus a short explanation)?
6. Any hard constraints (read-only, specific SQL dialect features)?

You:
1. PostgreSQL. customers(id, name, email) and orders(id, customer_id, total_cents, created_at, status).
2. Total spend on orders where status = 'completed'.
3. Last 12 months, top 20.
4. Just me, running it by hand in psql.
5. SQL in a code block, then two sentences on assumptions.
6. One read-only SELECT statement.

Claude:

<context>You are a senior data analyst who writes careful PostgreSQL. I will run the query by hand in psql. The database has a customers table with columns id, name, and email, and an orders table with columns id, customer_id, total_cents, created_at, and status. The value 'completed' in the status column marks a finished order, and total_cents holds the order total in cents.</context>

<instructions>Write one read-only SELECT statement that returns the 20 customers with the highest total spend on completed orders created in the last 12 months. Join customers to orders, sum total_cents, and sort from highest spend to lowest. Return the customer id, name, email, and total spend. Before you respond, check that the query filters on status and on the date window and returns at most 20 rows.</instructions>

<output_format>Return the SQL in a single fenced code block, followed by two sentences stating the assumptions you made. Use only standard PostgreSQL. Write no statement that changes data, and add no other commentary.</output_format>

<uncertainty>If a table or column you need is not in the schema above, say which one is missing instead of guessing its name. If the question is ambiguous in a way the schema cannot resolve, state the interpretation you chose in your two assumption sentences.</uncertainty>

What changed and why: "best customers" now has a definition, a time window, and a row limit, so the query is checkable. The schema is in the prompt, so Claude does not invent column names, and the output format fixes the shape of the answer. No example is included because the output format already describes the answer.
```

### Example 3: study plan

```
You: help me study for my exam

Claude: A few quick questions before I write the prompt.
1. What exam and deadline (for example, organic chemistry final in six weeks)?
2. How many hours per week can you study?
3. Where are you strong and where are you weak?
4. Is this plan only for you, or will a tutor or study group use it?
5. What format do you want the plan in?
6. Any hard constraints (days off, free resources only)?

You:
1. Organic chemistry final, six weeks.
2. Eight hours a week.
3. Strong on nomenclature, weak on reaction mechanisms.
4. Only for me.
5. One paragraph per week.
6. No study on Sundays, free resources and my textbook only.

Claude:

<context>You are an experienced organic chemistry tutor. I am a student with six weeks until my final exam and eight hours a week to study, Monday through Saturday. I am strong on nomenclature and weak on reaction mechanisms, and I can use only free resources and my own textbook.</context>

<instructions>Create a six-week study plan that gives reaction mechanisms most of the study time, since that is where I am weak, and keeps nomenclature review brief, since I am strong there. For each week, state the goal and say how the eight hours should be spent. Before you respond, check that every week fits within eight hours and no study falls on a Sunday.</instructions>

<output_format>Write six paragraphs of flowing prose, one per week, each starting with "Week 1:", "Week 2:", and so on. Use plain text with no tables or bullet points. Begin directly with Week 1 and end after Week 6.</output_format>

<uncertainty>I have not told you my course syllabus. If you are unsure which mechanisms my course covers, name the ones you assumed and mark them as assumptions rather than presenting them as the syllabus. Do not recommend specific websites or books unless you are confident they exist and are free.</uncertainty>

What changed and why: the plan now has a time budget, a weakness to target, a fixed shape, and a self-check against the constraints. The uncertainty section stops Claude from presenting a guessed syllabus as fact.
```

### Example 4: proceeding without questions

If you say to skip the questions, the skill proceeds and lists its assumptions explicitly, so you can see and correct them.

```
You: Write a cover letter for a marketing coordinator job. Don't ask me questions, just go.

Claude:

<context>You are an experienced career coach who writes cover letters for early-career marketing candidates. The letter is for a marketing coordinator role at a mid-sized company, and the reader is a hiring manager whose name is not known. I have not given you my résumé, so you know nothing about my real employers, results, or skills.</context>

<instructions>Write a one-page cover letter of about 300 words. Open with a direct statement of interest in the role, connect the typical duties of the role to the strengths an early-career candidate can honestly claim, and close with a short request for an interview. Keep the tone professional and warm. Before you respond, check that the letter contains no claim about my background that I did not supply.</instructions>

<output_format>Return the letter in plain text with a greeting and a sign-off. Apart from the missing-information note described below, add no commentary before or after the letter.</output_format>

<uncertainty>Do not invent employers, job titles, achievements, or numbers. Where the letter would normally contain one, write around it, and after the letter list in one short paragraph the personal details I should supply so you can strengthen it.</uncertainty>

Assumptions: I assumed a one-page letter of about 300 words in plain text. I assumed the reader is a hiring manager at a mid-sized company whose name I do not know, and a professional but warm tone. I assumed an early-career candidate, because no experience level was given. I assumed no employers, achievements, or numbers may be invented, since none were provided.

What changed and why: the request now has a length, an audience, a tone, and a rule against fabricated experience. The assumptions above are the points to correct if they are wrong.
```

## How to trigger it

In Claude Desktop, the skill is meant to load when your request matches its description. Phrasings that fit that purpose include:

- "Improve this prompt: write me a thing about dogs for my blog"
- "Make this prompt better before you run it"
- "Turn this rough request into a good prompt"
- "Help me write a better prompt for summarizing a long report"
- "Make me a prompt for writing cold emails" or "Write a prompt that does this"
- "Optimize my prompt", "tighten up my prompt", or "rewrite my prompt"
- "Make my system prompt clearer" or "fix my system prompt"
- "Prompt engineering help"
- "Use the prompt-smith skill on this: summarize this report for my boss"

In Claude Code you can also invoke it directly with `/prompt-smith`, then give your rough prompt. See the note in [Install for Claude Code](#install-for-claude-code) about whether text after the command is passed through.

These phrasings are examples, not a tested list. Whether automatic loading fires depends on the `description` field in `SKILL.md` and on how many other skills are installed. In Claude Code, the slash command is the documented direct invocation. Mentioning the skill by name in a message may help Claude choose it, but this is untested. The author did not find a documented direct invocation for Claude Desktop (unverified). A bare task such as "write a script that renames my photos" is not meant to trigger the skill unless you invoke it by name. The description also excludes cases where you want the email, poem or summary itself written instead of a prompt for it, and it excludes explaining prompt concepts or creating Claude skills. As summarized by the author, not verified here, Anthropic's skill-writing guidance also notes that Claude consults skills mainly for tasks it cannot easily do directly, so a trivial one-line request may not trigger any skill.

## Customization

Everything is plain Markdown, so you can change it with a text editor.

- Edit a reference file to change one technique. For example, edit `reference/output-format-and-constraints.md` to add your preferred format rules. These files are read only when `SKILL.md` points Claude at them.
- Add house-style rules. Create a new file such as `reference/house-style.md` and add one line to the reference index in `SKILL.md` that links to it with a relative path. A house-style rule can be a plain sentence, for example: "Every prompt for the finance team states the reporting currency and the fiscal year in the context section." Keep links one level deep from `SKILL.md`. As summarized by the author from Anthropic's pages and not verified here, Claude may read only part of a file that is reached through a chain of links.
- Keep the workflow consistent. The five steps, the 3 to 6 numbered questions, the rule that the questions step is never skipped, the proceed-without-questions rule, the Step 3 structure, the uncertainty permission, and the Step 5 behavior are repeated across `SKILL.md`, this README, and the reference files. If you change one, change all of them.
- Keep `SKILL.md` under about 500 lines and start any reference file longer than 100 lines with a table of contents. Both come from Anthropic's skill best-practices guidance.
- Do not add unsupported frontmatter keys if you plan to upload to Claude Desktop. Anthropic's docs say the portable keys are `name`, `description`, `license`, `compatibility`, `metadata`, and `allowed-tools`, and that other keys make upload fail. Claude Code accepts more keys, but they are not portable.
- Keep the `name` field and the folder name identical, and keep exactly one `SKILL.md` in the folder.
- After editing, Claude Code is documented to pick up changes on its own (stated in the Claude Code docs; not run against a live Claude Code, see the reload notes above). For Claude Desktop, re-zip the folder and upload it again.

## Troubleshooting

The skill does not trigger.

- In Claude Desktop (assumed to match claude.ai, which was not checked), confirm the skill is toggled on and that code execution is enabled under Settings > Capabilities. Then name the skill in your message, which may help but is untested.
- In Claude Code, confirm that `~/.claude/skills/prompt-smith/SKILL.md` or `.claude/skills/prompt-smith/SKILL.md` exists and that the folder name matches the `name` field. If the skills directory did not exist when the session started, the docs say to run `/reload-skills`. Try the slash command `/prompt-smith` to separate a loading problem from a description-matching problem. The docs also point to `/doctor` for diagnosing skills. These commands are stated in the docs and not run against a live Claude Code (see Install for Claude Code).
- If you have many skills installed, Claude Code's docs say descriptions can be shortened or dropped to fit a budget, which weakens automatic matching. The slash command should still work because skill names are always listed, but this was not tested.
- If the frontmatter in `SKILL.md` is not valid YAML, for example after an edit that adds a colon followed by a space to an unquoted description, the Claude Code docs indicate the skill loads with empty metadata, so it cannot match on its description. Keep the description a valid YAML string, either quoted or a block scalar (as `SKILL.md` does with `>-`), so a colon followed by a space cannot break parsing.

The questions feel like too many.

- The skill asks 3 to 6 questions and each one is meant to be answerable in one line. Answer in a few words each, or reply "proceed without questions" and the skill will proceed and list its assumptions explicitly. It will not skip the questions step on its own, even for a request that looks complete.

The output is too long.

- Answer the output-format question with a concrete limit, such as "under 150 words" or "two paragraphs", so the limit is written into the improved prompt. After a run, ask for a shorter version and say which part to cut. Model behavior varies, including how verbose a model is by default, so check current docs. See `reference/iteration.md` and `reference/output-format-and-constraints.md`.

The improved prompt invents details.

- Check that the `<uncertainty>` section is present. If a fact matters, supply it in your answers instead of leaving Claude to infer it. See `reference/uncertainty-handling.md`.

## Known uncertainties

These are the details the author could not verify. They are listed here so nothing in this README reads as stronger than the evidence.

- The skill has not been tested in a live Claude Desktop, claude.ai, or Claude Code session. Nothing was uploaded or run.
- The exact Claude Desktop menu path for skills. Anthropic's Help Center describes Customize > Skills and places the code-execution toggle under Settings > Capabilities. An older platform page says Settings > Features. Which one your app shows was not confirmed, and no other path is claimed here.
- The Help Center does not mention the Claude Desktop app separately. This README assumes it follows claude.ai.
- Which Team and Enterprise setting enables skills. The Help Center is reported to tie the Organization settings > Plugins & skills > Policy tab path to code execution, with Enterprise owners enabling skills first. The exact toggle names were not confirmed.
- Plan availability. The Help Center lists Free, Pro, Max, Ultra, Team, and Enterprise, while an older platform page lists Pro, Max, Ultra, Team, and Enterprise. Confirm for your plan.
- The `description` length limit that the claude.ai uploader enforces. A summary of the Help Center reported 200 characters (not read verbatim), while the platform docs and the Agent Skills specification say 1,024. Which applies at upload was not confirmed. This skill's description is 909 characters and is not known to pass a 200-character limit. If an upload is rejected over the description, shorten it to 200 characters or fewer (some trigger phrases will have to go).
- Whether the uploader requires the ZIP file name to equal the folder name, whether a `.skill` extension is accepted there, and any maximum ZIP size. The Help Center describes `.zip` and gives no size limit.
- How to read the Help Center's wording that the ZIP should contain "the skill folder as its root (not a subfolder)". The author followed the layout that Anthropic's own packaging script produces, with the skill folder as the top-level entry in the archive.
- Whether Claude Code passes text typed after `/prompt-smith` to the skill, and whether the `/reload-skills` and `/doctor` commands, the file-watching behavior, and the `~/.claude/skills/synced/` download behavior work as the Claude Code docs state. Those items are stated in the docs and not run against a live Claude Code.
- Whether Prompt Cowboy itself asks clarifying questions. No public source the author could open documents that. The workflow in this repo is its own design. Prompt Cowboy's tagline is "Turn lazy prompts into great ones", and that tagline is all this repo refers to; the actual workflow of that product is unverified.
- Some quoted guidance comes from Anthropic's docs as summarized by the author when this repo was written. Wording can drift from the live pages, and the docs change as new models ship.
- Whether `gh attestation verify` behaves as described on your GitHub CLI version. The author ran it once against the v1.0.0 release ZIP with GitHub CLI 2.103.3 and it succeeded (exit code 0, no output on success); other versions were not tried.

## Limitations

- The skill improves a prompt. It cannot guarantee that the improved prompt gives a better result than your original, and it cannot supply facts you did not give it.
- Results vary by model and by Claude version. As summarized by the author from Anthropic's pages, not verified here: treat a technique that names a specific model as measured on that model and re-check it against your own tests before applying it to another. Examples include how literally a model follows instructions, how verbose it is by default, and how it handles verification instructions.
- The skill budgets 1 to 3 examples per improved prompt to keep prompts compact. As summarized by the author, not verified here, Anthropic's guidance often suggests 3 to 5 examples. If a task needs more, supply them in your answers.
- The questions step adds one round trip. That is a deliberate trade for a prompt that fits your actual goal, audience, and constraints.
- It does not run evaluations. If a prompt will run at scale, test it against your own cases and iterate. See `reference/iteration.md`.
- Use skills only from sources you trust, and read a skill's files before installing it. The skill folder (`SKILL.md` and `reference/`) contains only Markdown and no scripts. The repo also holds CI and release workflows under `.github/`, which are not part of the skill. See the Security section.

## Security

The skill is Markdown only: no scripts, no binaries, no network calls of its own, and no `allowed-tools` setting, so it grants Claude no extra tool permissions. Its `SKILL.md` has a Safety section that tells Claude to treat pasted material as data, never to copy credentials into a prompt, and to answer the improved prompt in text rather than act on instructions hidden in pasted content. The repo has an `integrity` check that fails on any non-Markdown file, executable file, extra frontmatter key, hidden Unicode control character, or link to an unexpected host. Release ZIPs are built by a workflow and published with a SHA-256 checksum and a provenance attestation (`gh attestation verify prompt-smith.zip --repo Heiwa-sec/prompt-smith`), and `main` and the version tags are protected by repository rulesets. See `SECURITY.md` for how to verify a download, what is protected, what these measures cannot prevent (copies and forks), and how to report a vulnerability privately.

These measures lower risk and do not eliminate it. A skill is a set of instructions, so read what you install and install only from https://github.com/heiwa-sec/prompt-smith.

## Attribution and license

Author: heiwa-sec

Repo: https://github.com/heiwa-sec/prompt-smith

The intake, clarifying questions, one optimized prompt flow is this repo's own design. It is modeled on the publicly stated purpose of Prompt Cowboy, whose tagline is "Turn lazy prompts into great ones"; that product's actual workflow is not documented here, and whether that product asks clarifying questions is unverified. This is an independent project. The prompting rules in `reference/` are grounded in Anthropic's published prompt-engineering guidance and are summarized in the author's own words.

Released under the MIT License. See `LICENSE`. Copyright (c) 2026 heiwa-sec.
