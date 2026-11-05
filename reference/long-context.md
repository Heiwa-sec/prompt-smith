# Long context

This file covers prompts that include long documents or large pasted inputs: contracts, reports, transcripts, data exports, several files at once. Read it when the user's input includes long documents or several files (see the Reference index in SKILL.md). It sets where the document goes, how to tag it, where the question goes, and how to make Claude ground its answer in quotes. The quote-grounding rule also supports the uncertainty permission required by Step 4, so read uncertainty-handling.md alongside it.

## Contents

- Rules (1 to 6, including the 500-word and 2,000-word thresholds)
- Before and after
- Skeleton
- Apply in this skill

## Rules

### 1. Put long documents at the top

Rule: place a pasted input longer than about 500 words near the top of the improved prompt, inside named document tags, above the context, instructions, and examples. A shorter pasted input goes in a named tag (for example, a contract tag) after the context.

Why it works: Anthropic's prompting guidance says this placement can improve performance and applies it to inputs of roughly 20,000 tokens or more. Claude reads the material first and then meets the task with the whole input already in view.

How to apply: when the user pastes a document longer than about 500 words together with a question, move the document to the very start of the improved prompt. The 500-word threshold is this skill's own rule, chosen because the user rarely knows Anthropic's much larger figure and the reordering is cheap. The quote-extraction step in rule 4 is separate and does not apply to every input. If the material is a file Claude can open or is too long to paste, name the file and tell Claude to read it in full before answering.

Reprint size: in the Step 5 prompt block, reprint a pasted document in full and unchanged inside its tag up to about 2,000 words, so the block is complete and paste-ready, and say in the note that the document was left unchanged and moved to the top. Above about 2,000 words (or for an attached file) the note says instead that the document is not reprinted, that the tag is left empty for the user to paste into, and that the original was used unaltered for this run. Above about 2,000 words, or when the input is an attached file, do not reprint it, because that doubles the output and risks silent drift in the copy. Instead the fence carries the named empty document tag (for example, an empty document_content tag) plus one line telling the user to paste the original between the tags, or naming the attached file or path; the execution in this turn uses the original that is already in context. Never put an unnamed stand-in line where the document belongs. When the input is a file, name it: in Claude Code, give the path and tell Claude to read the file in full before answering; in Claude Desktop, refer to the attached file by name (for example, "the attached file contract.pdf") and tell Claude to read it in full. The user must attach the file again when reusing the prompt in a new conversation.

### 2. Wrap each document in tags, with structure when there are several

Rule: wrap a single document in one descriptive tag. For several documents, use a documents tag around everything, one document tag per item with an index attribute, and inside each a source tag and a document_content tag.

Why it works: tags let Claude tell where one input ends and the next begins, and which text is data and which is instruction. The source tag lets Claude and the reader cite a document by name. Tag naming rules are in xml-structure.md.

How to apply: use the file name, title, or date of each item as its source. Keep the original text unedited inside document_content.

### 3. Put the query and instructions at the end

Rule: after the documents, give context, then instructions, then the output format and the uncertainty section. The instructions come after the documents; output_format and uncertainty may follow them.

Why it works: As summarized by the author from Anthropic's published guidance (not verified here), putting the query after the documents can improve response quality, especially with complex inputs that span several documents; check current docs for any figures. Anthropic's finding concerns the query coming after the documents, not the query being the final tokens of the prompt, and this skill did not test whether trailing sections dilute it.

How to apply: when rewriting a prompt that starts with the question, split it. The question moves after the documents as an instruction with an action verb; the documents move to the top. Let the reasoning cue at the end of the instructions name the question again.

### 4. Ask for quotes first, then the answer, when the task calls for it

Rule: for question answering, extraction, risk-flagging, and fact checks over long prose, tell Claude to extract the word-for-word passages relevant to the task into a quotes section, and then write the answer using only those quotes, referring to them by number. For a summary, do not ask for a quotes block, because the relevant passages are the whole document and the output would double in length; ask instead for a clause or section reference on each point and cap any quote, for example "quote only the sentence behind each key obligation, at most 15 words". For tables and short inputs, quote extraction does not apply; ask Claude to cite the rows or lines it relies on. For a mixed task such as a summary plus flagged risks, use clause or section citations for the summary part and short quotes (at most 15 words) only for the flagged risks.

Why it works: Anthropic's hallucination guidance recommends quote extraction for long documents because it makes Claude locate evidence before it concludes, and it can filter out unrelated text. Anthropic scopes it to long documents (it cites above 20k tokens). The skill's own judgment, not Anthropic's claim, is that copying a whole table into a quotes block adds length and risks omissions.

How to apply: name the tags for the two parts (quotes, then answer) so the user can see the evidence next to the conclusion. Anthropic cautions that these techniques reduce hallucination but do not eliminate it, so the user should still check critical claims.

### 5. Give a defined result for the case with no evidence

Rule: tell Claude what to write when no relevant quote exists, and let it retract any claim it cannot support with a quote.

Why it works: without a stated fallback, Claude has no instruction for the case with no evidence. The fallback is the uncertainty permission from Step 4 applied to documents.

How to apply: add a sentence such as "If no passage is relevant, write 'No relevant quotes found.' and say what is missing." Keep it inside the uncertainty section as well.

### 6. Restrict the answer to the provided material when the user wants that

Rule: when the user says to rely only on the documents, say so in plain words, and forbid outside knowledge for the facts of the case.

Why it works: Anthropic lists restricting Claude to the supplied documents as a hallucination control. Without it Claude may mix general knowledge into a document-specific answer.

How to apply: ask in a hard-constraints question (see question-patterns-and-template.md) whether outside knowledge is allowed, and write the answer into the instructions.

## Before and after

Domain: contract review.

Before (rough prompt, question first, document untagged):

```
Is my vendor allowed to raise the price? Contract: 7.1 The platform fee is 3,900 per month. 7.2 Provider may adjust the fee once per contract year on 60 days written notice, by no more than 5 percent of the then-current fee. Their email: Starting 1 March the platform fee will be 4,200 per month.
```

After (documents first, tagged, instructions last, quote grounding, uncertainty). Answers given: the user is a small-business owner who is not a lawyer; the question is whether the announced increase is allowed; use only the two documents; answer in plain prose under 150 words. The two documents are a deliberately small illustration that uses the document-first layout only to demonstrate it; at this size the contract would sit in a tag after the context, and a real contract would be much longer.

```
<documents>
<document index="1">
<source>master_services_agreement_excerpt.txt</source>
<document_content>7.1 The platform fee is 3,900 per month. 7.2 Provider may adjust the fee once per contract year on 60 days written notice, by no more than 5 percent of the then-current fee.</document_content>
</document>
<document index="2">
<source>vendor_email_december.txt</source>
<document_content>Starting 1 March the platform fee will be 4,200 per month.</document_content>
</document>
</documents>

<context>You are a contracts analyst helping a small-business owner who is not a lawyer. The owner wants to know whether a price increase announced by a vendor is allowed by the agreement.</context>

<instructions>First, find the passages in the two documents that bear on whether the increase is permitted and copy them word for word into numbered entries inside <quotes> tags. If no passage is relevant, write "No relevant quotes found." Then, using only those quotes, answer inside <answer> tags: state whether the announced fee is within the limit the agreement sets, cite each quote by number, and show the arithmetic. Before responding, recompute the percentage increase from the two fee figures and confirm that your answer matches it.</instructions>

<output_format>Return the <quotes> block first and the <answer> block second. Write the answer as plain prose in no more than 150 words. Do not use headings, and do not add commentary outside the two blocks.</output_format>

<uncertainty>If the documents do not contain enough information to decide, say which fact is missing instead of guessing. Do not rely on general knowledge of contract law for any statement about these documents.</uncertainty>
```

## Skeleton

The layout of any long-context improved prompt, described section by section:

```
<documents>
One document tag per input, numbered with an index attribute. Inside each, a source tag naming the file or title, and a document_content tag holding the full original text. A single document may use one tag named for what it is, such as <transcript>. Use this layout for input longer than about 500 words; shorter input goes in a named tag after the context.
</documents>

<context>
A one-sentence role ("You are ..."), then who the user is, who will read the result, and why the question matters.
</context>

<instructions>
Where quote grounding fits (rule 4): first, extract the passages relevant to the question, word for word, into numbered entries inside <quotes> tags, and write a fixed phrase when none exist; then answer using only those quotes, citing them by number. Otherwise: do the task and cite a clause, section, or row for each point. The final sentence is the reasoning cue naming the question and what to check before responding.
</instructions>

<output_format>
The order of the two parts, the length, the style, and one or two short negative constraints (at most two).
</output_format>

<uncertainty>
Permission to say what is missing or "I don't know" instead of guessing, and a ban on filling gaps from outside knowledge when the user asked for that.
</uncertainty>
```

## Apply in this skill

In Step 2, ask whether the answer must come only from the pasted material; that answers rule 6. In Step 3, when the request involves pasted material, use the skeleton above: documents first, then context, then the instructions and the rest of the standard order. Use the quotes step only where rule 4 says it fits. In Step 4, the no-evidence fallback of rule 5 is the uncertainty section. In Step 5, reprint the document in full inside its tag up to about 2,000 words (above that, or for an attached file, use the named empty tag and the paste line described in rule 1), and say in the note, as a mandatory item, that the document text was left unchanged and moved to the top (or, above the size limit, that it was not reprinted and the tag is left for the user to paste into). The full worked analysis case is in worked-examples.md.
