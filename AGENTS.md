# AGENTS.md

## Project

This repo has the spec for an AI code verification pipeline.

- `llm.txt` is the source of truth. Edit it first. Then update `README.md`.
- `README.md` is short. It tells the user how to set up and run the pipeline.
- Do not state that a command ran if it did not run. Mark it NOT RUN.

## Documentation rules

All text in `README.md`, `llm.txt`, comments, commit messages, and error messages must
follow ASD-STE100 Simplified Technical English (STE).

### Style: STE plus caveman

Write short. Cut filler. Say one thing, then stop.

- Keep STE grammar. Add caveman terseness on top.
- Drop filler words: "in order to", "please", "note that", "it is important", "basically".
- In list items, tables, and comments, you may drop articles. Keep them in procedure
  sentences if the meaning is not clear without them.
- Prefer a short fragment to a full sentence in a list. Write "Fail on exit 2." not "The script should fail when the exit code is 2."
- Do not drop a number, unit, flag, path, or rule id. Do not make the text unclear.
- Do not change a rule, a value, or a name when you shorten text.

### Sentences

- Write a maximum of 20 words in a procedure sentence.
- Write a maximum of 25 words in a descriptive sentence.
- Put one instruction in one sentence.
- Put the condition before the command. Write "If the file is missing, stop."
- Use the imperative for steps. Write "Run the script." Do not write "You should run the script."
- Write "Do not" for a prohibition. Do not use "avoid" or "never" alone.
- Put a warning or caution before the step it applies to.

### Paragraphs

- Write one topic in each paragraph.
- Write a maximum of 6 sentences in each paragraph.
- Use a list for 3 or more parallel items.
- Use a numbered list for steps that need an order.

### Words

- Use one word for one meaning. Use the same word for the same thing in all files.
  Examples: "finding" (not "issue" or "alert"), "scan" (not "check" or "review" for the run),
  "lock file" (not "pin file").
- Use the simple verb. Write "use", not "utilize". Write "start", not "initiate".
- Do not use a phrasal verb when a single verb exists. Write "remove", not "take out".
- Do not use a word as two parts of speech. Do not use "test" as a verb and a noun in one page.
- Use a technical name exactly as it is in the tool. Write `--diff-only`, not "the diff flag".
- Keep articles in procedure sentences (see Style).
- Do not use a noun cluster of more than 3 nouns. Break it up with a preposition.
- Do not use slang, idioms, jokes, or metaphors.
- Do not use marketing words. Examples: "powerful", "seamless", "robust", "blazing".
- Spell out an abbreviation at first use. Use the same abbreviation after that.

### Verbs and tenses

- Use the simple present for facts. Write "The script writes the file."
- Use the simple future only for a result that comes later.
- Use the active voice. Use the passive voice only if the actor is not known or not important.
- Do not use the -ing form as a verb or as a noun when a simple form is possible.
- Do not use the perfect tenses or the conditional "would" in a procedure.

### Facts

- Give a number and a unit. Write "7 days", not "a few days".
- Write a value, a path, or a flag in code format.
- Do not make a claim you cannot show. Write "The scan is offline" only if a test shows it.
- Write what the tool does not do. Do not hide a limit.

### Structure

- Start each file with its purpose in one sentence.
- Use short headings. Use the same heading names for the same content.
- Put a command in a code block. Put one command on each line.
- Do not repeat text between files. Link to the source file.

## Check before you finish

1. Count the words in each long sentence. Cut it if it is too long.
2. Search for these words and replace them: "just", "simply", "easily", "etc.", "and so on", "should", "may want to".
3. Search for the passive voice and for "-ing" verbs. Change them if you can.
4. Check that each term has one meaning in all files.
5. Run `grep -n "TODO\|<…>\|TBD" README.md`. The result must be empty.
   `llm.txt` names these words in its rules. Do not add them as open work items.

## Commit messages

- Write a subject of 50 characters or fewer, in the imperative. Write "Add CI detection".
- Add a body only if the reason is not clear. Write why, not what.
- Do not add attribution lines.
