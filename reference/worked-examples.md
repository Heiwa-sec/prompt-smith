# Worked examples

This file holds three complete worked examples in different domains: blog writing, coding, and analysis. Each shows the rough input, the clarifying questions the skill asks in Step 2, and the improved prompt built in Step 3 with the uncertainty permission from Step 4. Read it when you need a model for the shape of a good question list or a finished prompt, especially the coding example for an embedded input and output pair and the analysis example for the long-context layout.

## Contents

- Example 1: blog writing (compact and full form)
- Example 2: coding
- Example 3: analysis
- Note on the compact blog example
- Apply in this skill

## Example 1: blog writing

Answers the user gives in this example: about 600 words in a warm tone, first-time dog owners, three headings and a closing call to action.

<example>
<input>write me a thing about dogs for my blog</input>
<clarifying_questions>
1. What length and tone do you want (for example, 600 words, friendly and conversational)?
2. Who is the audience (new dog owners, breed enthusiasts, general readers)?
3. Should it include headings, a call to action, or SEO keywords?
</clarifying_questions>
<improved_prompt>
<context>I write a pet-care blog for first-time dog owners. My readers want practical, trustworthy advice.</context>
<instructions>Write a 600-word blog post about choosing a first dog. Use a warm, conversational tone. Include three H2 headings and end with a one-sentence call to action inviting readers to comment with their own experience.</instructions>
<output_format>Return the post in Markdown. Do not add a title line or any commentary before or after the post.</output_format>
<uncertainty>If a fact about dog breeds is uncertain, omit it rather than guess.</uncertainty>
</improved_prompt>
</example>

Full form of the same blog prompt, with the role, the reasoning cue, and a fuller uncertainty clause that real prompts carry:

<example>
<improved_prompt>
<context>You are an experienced pet-care writer. I write a pet-care blog for first-time dog owners. My readers want practical, trustworthy advice.</context>
<instructions>Write a 600-word blog post about choosing a first dog. Use a warm, conversational tone. Include exactly three H2 headings and end with a one-sentence call to action inviting readers to comment with their own experience. Before responding, check that the post has exactly three H2 headings, is close to 600 words, and ends with the call to action, and revise it if it does not.</instructions>
<output_format>Return the post in Markdown, beginning directly with the first paragraph. Do not add a title line, and do not add any commentary before or after the post.</output_format>
<uncertainty>If a fact about dog breeds, costs, or health is uncertain, omit it or say that you are unsure, and say "I don't know" rather than guess.</uncertainty>
</improved_prompt>
</example>

## Example 2: coding

Answers the user gives in this example: Python 3 with only the standard library; the file is a UTF-8 CSV export of contacts with a header row and the columns name, email, and signup_date; "clean up" means trimming spaces, lowercasing emails, and removing duplicate rows, where a duplicate means every field matches after cleaning, so two rows that differ only in signup_date are both kept; the code is for the user alone; return the function with a short usage example; the original file must stay untouched.

<example>
<input>write a function to clean up my csv</input>
<clarifying_questions>
1. Which language and libraries should the function use (for example, Python with only the standard library, or Python with pandas)?
2. What does "clean up" mean for your file (for example, trim spaces, remove duplicate rows, fix date formats, fill missing values)?
3. Who will read or maintain this code (just you, a team, a beginner)?
4. What should I return (for example, only the function, or the function plus a short usage example)?
5. Any hard constraints, such as no new dependencies, a file too large for memory, or the original file must not change?
</clarifying_questions>
<improved_prompt>
<context>You are a senior Python developer helping me tidy a contacts export. The file is a UTF-8 CSV with a header row and the columns name, email, and signup_date. I am the only person who will maintain the code, so clear and short matters more than clever.</context>
<instructions>Write a Python 3 function that takes an input path and an output path. Read the input with the standard library csv module only. Trim leading and trailing spaces from every field, convert the values in the email column to lowercase, and drop any row that duplicates an earlier row once those changes are applied, and treat a row that differs in any field, even only signup_date, as not a duplicate. Keep the first occurrence, keep the original row order, leave empty values empty, and write the result to the output path without modifying the input file. The examples in the examples tag are illustrative; follow their format, not their content. Before responding, trace each example input through your function and confirm that it produces exactly the example output, and fix the function if it does not.</instructions>
<examples>
<example>
<input>
name,email,signup_date
 Ana Silva ,ANA@Example.com,2024-03-05
Ana Silva,ana@example.com,2024-03-05
Ben Ortiz,,2024-03-07
</input>
<output>
name,email,signup_date
Ana Silva,ana@example.com,2024-03-05
Ben Ortiz,,2024-03-07
</output>
</example>
<example>
<input>
name,email,signup_date
Ana Silva,ana@example.com,2024-03-05
Ana Silva,ana@example.com,2024-04-19
</input>
<output>
name,email,signup_date
Ana Silva,ana@example.com,2024-03-05
Ana Silva,ana@example.com,2024-04-19
</output>
</example>
</examples>
<output_format>Return one Python code block containing the function and a short usage example. After the block, add at most one sentence naming any assumption you made. Do not add explanations before the block.</output_format>
<uncertainty>If the example leaves a case unspecified, such as a row with fewer fields than the header, name the assumption you made in the sentence after the block instead of choosing silently. If you are unsure how a library function behaves, say you are unsure rather than present it as fact.</uncertainty>
</improved_prompt>
</example>

## Example 3: analysis

Answers the user gives in this example: a short written summary with one table; the readers are sales leadership, who will read it before the Q4 planning meeting; the question is why revenue moved between Q2 and Q3; compare the two quarters; use only the pasted data and no outside benchmarks; keep it under 250 words. The user pasted the table below and a note from the sales manager saying that South lost its two largest accounts in August when their contracts expired, and that no other region lost accounts in Q3. The table and the note are deliberately small. This example uses the document-first layout and the quote step only to demonstrate them; at this size the inputs would normally sit in named tags after the context, and the quote step earns its length only on longer documents.

<example>
<input>look at these sales numbers and tell me whats going on</input>
<clarifying_questions>
1. What format do you want back (for example, a one-paragraph summary, a short memo with a table, findings for a slide)?
2. Who will read the analysis (for example, only you, your sales team, executives)?
3. What decision or question is this for (for example, why revenue fell, which region to invest in)?
4. Which time period and comparison matter (for example, this quarter versus last, year over year)?
5. Any hard constraints, such as only using the figures I pasted, no outside benchmarks, or a length limit?
</clarifying_questions>
<improved_prompt>
<documents>
<document index="1">
<source>sales_by_region_q2_q3.csv</source>
<document_content>
region,quarter,revenue_usd,units
North,Q2,410000,820
North,Q3,452000,845
South,Q2,380000,760
South,Q3,301000,640
East,Q2,295000,590
East,Q3,330000,610
West,Q2,350000,700
West,Q3,348000,690
</document_content>
</document>
<document index="2">
<source>sales_manager_note.txt</source>
<document_content>South lost its two largest accounts in August when their contracts expired. No other region lost accounts in Q3.</document_content>
</document>
</documents>
<context>You are a sales analyst writing for the sales leadership team, who will read this before the Q4 planning meeting. They know the business but have not seen these figures.</context>
<instructions>First, copy word for word any sentence in the note in document 2 (sales_manager_note.txt) that bears on why revenue changed between Q2 and Q3 into numbered entries inside <quotes> tags. If no sentence is relevant, write "No relevant quotes found." Then write the analysis inside <analysis> tags, using only the CSV in document 1 (sales_by_region_q2_q3.csv) and those quotes. State whether total revenue moved between the quarters, name the regions that drove the change with their percentage changes and the CSV rows you relied on, and say what the note in document 2 explains and what it leaves unexplained. Before responding, recompute every total and percentage from the figures and confirm that each one matches what you wrote.</instructions>
<output_format>Return the <quotes> block first and the <analysis> block second. Write the analysis as short paragraphs of plain prose under 250 words (the limit covers the analysis only, not the quotes block or the table), followed by one small Markdown table of revenue by region and quarter. Do not use outside benchmarks or industry comparisons.</output_format>
<uncertainty>If the documents do not show why a change happened, write "The data does not show why" for that change and name the extra data that would settle it. Do not infer causes the documents do not state, and say "I don't know" rather than guess.</uncertainty>
</improved_prompt>
</example>

## Note on the compact blog example

The first blog example is the compact form: it shows only the core sections, has no role sentence, no reasoning cue, and a one-line uncertainty clause. The full form above adds them. A real output carries the role, the reasoning cue, a task-specific uncertainty clause, and, when helpful, one to three examples, as the coding and analysis examples do. The questions in the compact example bundle related options (length and tone; headings, call to action, or keywords) to stay short, which is a documented exception to the one-topic-per-question rule. The compact example is minimal in its questions too: it has no explicit hard-constraints question and only partly covers output format. A real question list always includes the three mandatory topics: output format, audience, and hard constraints.

## Apply in this skill

Use these as shapes, not as text to copy. In Step 2, match the question lists: 3 to 6 numbered one-line questions, with example options in parentheses, covering output format, audience, and hard constraints (the compact blog questions do not, by design), plus goal questions when the goal is unclear. In Step 3, match the improved prompts: XML section tags, a one-sentence role opening the context, flowing prose inside them, numbered steps only when the task needs an order, a reasoning cue as the last sentence of the instructions, and one or two negative constraints after the positive format. Put stray framing sentences in `<instructions>`, never loose inside `<examples>`. In Step 4, every example ends with an uncertainty section. For the document layout in example 3, see long-context.md (documents longer than about 500 words go first; shorter ones go in a named tag after the context); for the template and question rules, see question-patterns-and-template.md.
