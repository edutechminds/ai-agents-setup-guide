# How to Talk to Your AI Coding Agents

## A Practical Prompting Guide for Pi and Hermes

**Companion to the [AI Coding Agents Setup Guide](AI-Coding-Agents-Setup-Guide.md)**

You installed the agents. You're staring at a blinking cursor. Now what?

This guide teaches you how to write prompts that get good results, how to recover when things go wrong, and how to use each agent's unique features. It includes real examples you can try right now.

---

## TABLE OF CONTENTS

1. The #1 mistake beginners make
2. The five rules of good prompts
3. Worked example: Build a project with Pi from scratch
4. Pi-specific techniques
5. Hermes-specific techniques
6. Hermes learning from Pi: the skill workflow
7. Prompt templates you can copy
8. When the agent gets it wrong

---

## PART 1: The #1 Mistake Beginners Make

The most common mistake is being too vague.

```
❌ BAD:  "make me a website"
❌ BAD:  "fix my code"
❌ BAD:  "write a python script"
```

These prompts give the agent almost nothing to work with. It has to guess what you want, and it will guess wrong. You'll get generic boilerplate that doesn't match your needs.

The fix is simple: **tell the agent what you'd tell a coworker sitting next to you.**

```
✅ GOOD: "Create a Python CLI tool that takes a CSV file as input and outputs
         a summary of each column — name, type, min, max, and number of missing
         values. Use argparse for the CLI. Keep it in a single file."
```

You don't need to be an expert to write good prompts. You just need to be specific about what you want.

---

## PART 2: The Five Rules of Good Prompts

### Rule 1: State What You Want Built

Be concrete. Name the language, the format, the purpose.

```
❌ "Help me with a script"
✅ "Write a bash script that backs up the ~/projects folder to a timestamped
   tar.gz file in ~/backups"
```

### Rule 2: Give Context About What Already Exists

If you have existing code, tell the agent where to look and what matters.

```
✅ "Look at the files in this directory. The main app is in app.py.
   I want to add a /health endpoint that returns JSON with the current
   timestamp and the app version from setup.py."
```

Pi's tools (read, bash) will let it explore your codebase — but pointing it in the right direction saves time and tokens.

### Rule 3: Mention Constraints

Anything you care about — keep it simple, use a specific library, stay under a file size, avoid dependencies — say it upfront.

```
✅ "Create a REST API with Flask. Use SQLite for storage, no ORM — just raw
   SQL. Keep it in a single file under 200 lines. Include error handling
   for invalid inputs."
```

### Rule 4: Say What You DON'T Want

Agents love to over-engineer. If you want something simple, say so.

```
✅ "Build this as a simple script. Don't create a class hierarchy or use
   design patterns. Just functions. No external dependencies beyond the
   standard library."
```

### Rule 5: Break Big Tasks Into Steps

Don't ask for an entire application in one prompt. Build incrementally.

```
Step 1: "Create a basic Flask app with a single /hello endpoint."
Step 2: "Add a SQLite database with a 'users' table."
Step 3: "Add a POST /users endpoint that creates a user."
Step 4: "Add a GET /users endpoint that lists all users."
Step 5: "Add input validation and error handling."
```

Each step is small enough that the agent can get it right, and you can verify before moving on. This is how experienced developers work with AI agents — small steps, constant verification.

---

## PART 3: Worked Example — Build a Project With Pi

Let's build a real tool from scratch. Follow along in Pi.

### Step 1: Set Up the Project

Open Pi inside the VM and type:

```
Create a new directory called ~/projects/csv-inspector and set up a basic
Python project structure with a main.py file, a README.md, and a
requirements.txt. Don't install anything yet — just create the files.
The tool will be a CLI that analyzes CSV files and prints a summary.
```

**What Pi does:** Uses its `bash` tool to create directories and its `write` tool to create files. You'll see each tool call in the TUI.

**What you do:** Read what Pi created. Does the structure look right? If so, continue. If not, tell it what to change.

### Step 2: Build the Core Feature

```
In main.py, write a function called analyze_csv that takes a file path,
reads the CSV with pandas, and returns a dictionary where each key is a
column name and each value is a dict with: dtype, count, nulls, unique
values, min (if numeric), max (if numeric), and a sample of 3 values.

Add a main() function that uses argparse to accept a file path argument,
calls analyze_csv, and prints the results in a readable table format
using tabulate. Add both pandas and tabulate to requirements.txt.
```

**What you do:** After Pi writes the code, test it:

```
Run: pip install -r requirements.txt && python main.py --help
```

You can type this as your next prompt and Pi will run it for you.

### Step 3: Test With Real Data

```
Create a sample CSV file called test_data.csv with 5 columns: name (text),
age (integer with some missing values), salary (float), department (text
with repeated values), and start_date (dates). Include about 10 rows with
some intentional missing values. Then run main.py on it and show me the
output.
```

### Step 4: Fix Issues

If the output doesn't look right, be specific about what's wrong:

```
❌ "That's wrong, fix it"
✅ "The date column is showing as 'object' dtype instead of datetime.
   Parse the start_date column as datetime during analysis. Also,
   the table is too wide for the terminal — switch from tabulate's
   'grid' format to 'simple' format."
```

### Step 5: Use Pi's Branching to Try Alternatives

This is Pi's killer feature. Say you want to try a different output format:

1. Press **Ctrl+B** or type `/tree` to open the session tree
2. You can see every message in your conversation as a branching tree
3. Navigate back to Step 3 (before you chose the table format)
4. Press Enter to branch from that point
5. Now try: "Instead of a table, output the summary as formatted JSON"

You now have **two branches** — one with table output, one with JSON output. You can switch between them without losing either. This is like git branches but for conversations.

### Step 6: Polish

```
Add a --output flag to main.py that accepts "table" (default) or "json".
Add a --columns flag that accepts a comma-separated list of column names
to analyze (default: all columns). Update the README with usage examples.
```

**What you just learned:**
- Start with structure, then build features one at a time
- Test after each step
- Be specific when something is wrong
- Use branching to explore alternatives without losing work

---

## PART 4: Pi-Specific Techniques

### Steering Messages (Redirect While Pi is Working)

If Pi is in the middle of writing code and you realize you forgot something, press **Enter** to send a steering message:

```
Also add type hints to all functions
```

Pi will see this after it finishes the current tool call and adjust its approach. You don't have to wait for it to finish and redo everything.

### Follow-Up Messages (After Pi Finishes)

Press **Alt+Enter** to queue a message that delivers only after Pi completes all its tool calls:

```
Now run the tests and fix any failures
```

### Reading Existing Code

When you `cd` into a project and start Pi, give it the lay of the land:

```
Read the README and the main source files in this project. Tell me what
this project does, how it's structured, and what dependencies it uses.
Then suggest three improvements you'd make.
```

### Multi-File Edits

Pi excels at changes that touch multiple files:

```
Rename the 'user' module to 'account' everywhere — update the filename,
all imports, all references in tests, and the README. Show me a summary
of every file you changed.
```

### Using Skills

Pi supports skills — reusable instruction sets that teach it specific workflows:

```
# List available skills
/skills

# If you've installed skills via packages:
pi install npm:@some/skill-package
```

You can also create project-specific skills by adding markdown files to a `.pi/skills/` directory in your project.

---

## PART 5: Hermes-Specific Techniques

### Research Mode

Hermes has web search, browser tools, and can synthesize information from multiple sources:

```
Research the top 5 Python testing frameworks in 2026. For each one, tell
me: what makes it unique, how many GitHub stars it has, whether it supports
async, and give me a one-line install command. Present this as a comparison
table.
```

### Memory — The Long Game

Hermes remembers things across sessions. Use this deliberately:

```
"Remember that my main project is a Flask API called csv-inspector. It uses
SQLite and is in ~/projects/csv-inspector. I prefer pytest for testing and
black for formatting."
```

Next time you open Hermes, it already knows this context. You can say:

```
"Help me add authentication to my project."
```

And Hermes knows you mean csv-inspector, Flask, SQLite — without you repeating it.

### Skill Creation — Hermes Learns From Experience

After completing a complex task (typically 5+ tool calls), Hermes can save the approach as a reusable skill. This is one of its most powerful features.

```
"Create a skill from what we just did — document the steps for setting up
a new Flask project with SQLite, pytest, and a basic CRUD API."
```

Hermes saves this as a structured markdown file in `~/.hermes/skills/`. Next time you (or anyone using your Hermes) need to set up a Flask project, Hermes loads the skill instead of figuring it out from scratch.

### Cron / Scheduled Tasks

Hermes can run tasks on a schedule through its messaging gateway:

```
"Every morning at 9am, check if there are any new issues on my GitHub repo
edutechminds/ai-agents-setup-guide and send me a summary on Telegram."
```

This requires the messaging gateway to be set up (`hermes gateway setup`).

### Subagent Delegation

For complex tasks, Hermes can spin up isolated subagents:

```
"Research three different CSS frameworks, and for each one, create a small
demo HTML page. Use subagents to work on each framework in parallel."
```

---

## PART 6: Hermes Learning From Pi — The Skill Workflow

This is the strategy that makes both agents more powerful together.

### The Pattern

1. **Solve it in Pi first.** Use Pi for hands-on coding. Figure out the right approach through trial, error, and branching.

2. **Document it in Hermes.** Once you have a working approach, tell Hermes to create a skill from it.

3. **Reuse it forever.** Next time you or Hermes encounters a similar task, the skill is loaded automatically.

### Example Walkthrough

**In Pi** — you build a project, figure out the right testing setup, discover that pytest with fixtures works best for your Flask app, learn that you need to use a test database, etc. This takes 30 minutes of back-and-forth.

**Then in Hermes:**

```
I just figured out a good pattern for testing Flask apps with SQLite.
Create a skill called "flask-sqlite-testing" that documents these steps:

1. Use pytest with conftest.py for fixtures
2. Create a test database in /tmp that's destroyed after each test session
3. Use Flask's test_client() for API endpoint testing
4. Structure tests as: test_models.py, test_routes.py, test_integration.py
5. Always test both success cases and error cases for each endpoint
6. Use factories instead of raw SQL for creating test data

Include the actual code patterns for the conftest.py fixtures and a sample
test file.
```

**Next time** you start a new Flask project and ask Hermes for testing help, it loads this skill automatically and applies your preferred patterns without you explaining them again.

### Building a Personal Skill Library

Over time, you accumulate skills for:
- How you set up new projects
- Your preferred folder structures
- Testing patterns that work for your stack
- Deployment checklists
- Code review criteria
- Common bug fix patterns

This is Hermes's "learning loop" — it genuinely gets better at helping you the longer you use it.

---

## PART 7: Prompt Templates You Can Copy

### Starting a New Project

```
Create a new [language] project in ~/projects/[name]. Set up:
- A basic project structure with src/ and tests/ directories
- A README.md with a description and setup instructions
- A [package manager] config file with [specific dependencies]
- A .gitignore appropriate for [language]
- Initialize a git repo with an initial commit

The project will be a [brief description of what it does].
```

### Understanding Existing Code

```
Read all the files in this directory. Give me:
1. A one-paragraph summary of what this project does
2. The main entry point and how to run it
3. The key modules and what each one is responsible for
4. Any obvious issues or code smells you notice
5. The testing situation — are there tests? Do they pass?
```

### Adding a Feature

```
I want to add [feature description] to this project.

Requirements:
- [specific requirement 1]
- [specific requirement 2]
- [constraint or limitation]

Don't modify [specific files/areas to leave alone].
Add tests for the new feature.
Show me what you plan to change before making edits.
```

### Fixing a Bug

```
There's a bug: [describe what happens]
Expected behavior: [describe what should happen]
Steps to reproduce: [how to trigger the bug]

Look at [relevant file/function] and figure out what's wrong.
Fix it and add a test that would have caught this bug.
```

### Code Review

```
Review the changes I made to [file(s)]. Check for:
- Bugs or logic errors
- Security issues
- Performance concerns
- Code style consistency with the rest of the project
- Missing error handling
- Missing edge cases

Be direct — tell me what's wrong, not just what's good.
```

### Research (Hermes)

```
Research [topic]. I need to make a decision about [specific choice].

Compare the top [N] options. For each one, tell me:
- What it does well
- What it does poorly
- How active the community/maintenance is
- A one-line install/setup command
- Whether it works with [my specific stack/constraints]

Give me a clear recommendation at the end with reasoning.
```

### Creating a Skill (Hermes)

```
Create a skill called "[name]" that documents the process we just
completed. Include:
- When to use this skill (trigger conditions)
- Prerequisites and setup steps
- The actual procedure, step by step
- Common pitfalls and how to avoid them
- Verification steps to confirm it worked
- Code snippets for the key parts
```

---

## PART 8: When the Agent Gets It Wrong

It will happen. Here's how to handle it.

### The Code Doesn't Work

Don't just say "it's broken." Give specifics:

```
❌ "That doesn't work"
✅ "When I run main.py with the test CSV, I get this error:
   TypeError: unsupported operand type(s) for +: 'int' and 'str'
   on line 42. The 'age' column has mixed types because of null values."
```

### The Agent is Going in the Wrong Direction

Stop it early. Press **Escape** in Pi to abort, then redirect:

```
"Stop — you're overcomplicating this. I don't need a class-based
architecture. Just write simple functions. Start over with main.py
as a single flat script with no classes."
```

### The Agent Changed Things You Didn't Ask For

Be explicit about scope:

```
"You modified app.py, models.py, and utils.py. I only asked you to
change the /users endpoint in app.py. Revert the changes to models.py
and utils.py."
```

In Pi, you can also use `/tree` to go back to before the change and try again.

### The Agent is Hallucinating (Making Up APIs/Libraries)

This happens more with smaller local models. Verify:

```
"Are you sure the 'pandas.auto_summarize()' function exists? Show me
the pandas documentation for it. If it doesn't exist, use only real
pandas functions to accomplish the same thing."
```

### You're Going in Circles

Sometimes the agent keeps trying the same broken approach. Break the loop:

```
"We've tried this approach three times and it keeps failing on the same
issue. Let's try a completely different approach. Instead of [what it
was trying], let's [alternative approach]. Start fresh."
```

### When to Start Over vs When to Fix

- **Fix** if the problem is small and specific (wrong variable name, missing import, logic error in one function)
- **Start over** if the architecture is wrong (wrong library choice, wrong data structure, fundamentally wrong approach)

In Pi, starting over is easy — use `/tree` to branch from an earlier point, or `/new` for a completely fresh session.

---

## Summary: The Workflow That Works

1. **Plan in Hermes** — research, compare options, make architectural decisions
2. **Build in Pi** — write code step by step, test after each step, branch to explore
3. **Document in Hermes** — create skills from successful patterns
4. **Repeat** — each cycle makes both agents smarter about how you work

The agents aren't magic. They're tools that amplify your thinking. The better you get at directing them, the more powerful they become. Start with small, specific tasks. Build confidence. Scale up.

---

*Last updated April 2026. Additions and corrections welcome via Issues or PRs.*
