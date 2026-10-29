# Exported skills

Two Claude Code skills from my job hunting repo, copied out so they can be reused
in another project or shared.

| Skill | What it does |
|-------|--------------|
| `build-tailored-resume/` | Takes a master resume and a job description and produces a tailored resume as JSON, then renders it to .docx |
| `humanizer/` | Edits text to remove AI writing patterns while keeping the facts the same |

Each folder holds one `SKILL.md`. The YAML block at the top of the file (name and
description) is how Claude decides when to use the skill. The rest is the
instructions it follows.

## Installing

Copy a skill folder into one of these locations:

- `<your-project>/.claude/skills/` to use it in a single project
- `~/.claude/skills/` to use it in every project

```bash
# one project
mkdir -p ~/code/my-project/.claude/skills
cp -R build-tailored-resume humanizer ~/code/my-project/.claude/skills/

# everywhere
mkdir -p ~/.claude/skills
cp -R build-tailored-resume humanizer ~/.claude/skills/
```

Start a new Claude Code session afterwards. Both skills should show up when you
type `/`.

### Extra setup for build-tailored-resume

This skill renders the .docx with the `resume-skill` CLI, so the CLI has to be
installed first. Use the vendored copy in the main repo. It has a local patch that
renders `**bold**` in the summary and bullets, and the PyPI package doesn't have it.

```bash
pip3 install vendor/resume-skill-renderer   # run from the Job_Hunting repo root
resume-skill --help                         # check it's on PATH
```

If you're on another machine, copy `vendor/resume-skill-renderer/` over too and
install it from there. The upstream project is
github.com/SankaiAI/ats-optimized-resume-agent-skill (MIT).

To get a PDF, convert the .docx afterwards. In the main repo that's
`scripts/docx_to_pdf.sh`, which needs Microsoft Word installed.

## Using build-tailored-resume

Run `/build-tailored-resume`, or just ask for it ("tailor my resume for this job",
"resume for Stripe"). Give it:

- your master resume or career history (a file path is fine, e.g. `CAREER.md`)
- the job description (paste it or point to a file)
- the company name
- your name, email and phone, if they aren't in the master resume

Optional: career level, page length (one or two pages), tone, roles to emphasize or
play down.

It runs in fixed stages and won't skip any of them:

1. Intake: collects inputs and asks at most 3 questions at a time if something is missing
2. JD analysis: required skills, keywords, seniority
3. Strategy: what to emphasize, section order, page target (can also research the company and guess at the team)
4. Content tailoring: picks and rewrites bullets, then does its own humanization pass
5. ATS check: keyword coverage and parseability
6. Render: writes the resume JSON, validates it, renders the .docx
7. Final validation: checks for placeholders and duplicates, then reports the file path, the strategy and any gaps

It won't invent tools, metrics or experience. If the JD asks for something your
master resume doesn't show, it tells you instead of adding it.

Example:

```
Tailor my resume for this job. Master resume is CAREER.md, company is Acme.
<paste job description>
```

Commands you can run yourself on the JSON:

```bash
resume-skill validate --input cv.json
resume-skill render --input cv.json --output cv.docx
```

Fonts, sizes, margins and divider lines are set in
`vendor/resume-skill-renderer/src/rendering/config.py`.

## Using humanizer

Run `/humanizer`, or ask Claude to "humanize this" or "make this sound less like
AI". It works three ways:

- **Pasted text:** you get a draft, a list of patterns that remain, and the final rewrite.
- **A file:** name the file and it rewrites the prose in place. Code blocks, commands, paths, YAML and links are left alone.
- **Inside another task** (a PR description, a resume summary): it returns only the final text.

If you give it a sample of your own writing, it matches that voice (sentence length,
punctuation, word choice) and the sample wins over its own rules. For example, if
you use dashes, it keeps them.

What it looks for, strongest signals first: "not X but Y" contrasts, one-line
closers, staged run-ups, forced groups of three, dash overuse, inflated or sales
language, stock AI words, bold labels and chatbot leftovers. It never adds facts,
numbers or quotes that weren't in the original.

Example:

```
/humanizer
<paste text>
```

## Using them together

In the main repo the resume pipeline runs `build-tailored-resume` first and then
`humanizer` over the summary and bullets, since humanizer's rules are stricter.
If you use them that way, tell humanizer which formatting to keep. My resumes
bold skills and numbers on purpose, and humanizer would otherwise strip that as
decorative bold.
