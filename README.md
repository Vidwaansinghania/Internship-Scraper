# Internship Scraper

An AI skill that searches LinkedIn and Indeed for internship and co-op postings, shows them in a table, and exports them to Excel when you ask.

Version 20261021

**[Overview](#overview)** · **[How it works](docs/how-it-works.md)** · **[Install](#install)** · **[Usage](#usage)** · **[Limits](#limits)** · **[Roadmap](#roadmap)**

---

## Overview

Checking two job sites by hand for the same search is slow and easy to get wrong: the filters differ, Indeed mixes full-time roles into internship searches, and the same posting turns up on both sites. This skill does the search for you and removes the repeats.

It works with any capable AI model. The skill is a plain Markdown prompt (`SKILL.md`) in the open Agent Skills format plus one Python script, so nothing in it is tied to one vendor. The agent needs a browser it can drive: load a page, read its text, and run a read-only script in the page.

Before it searches, the skill asks for the role and region, which have no default, and for filters with defaults: work arrangement, internship or co-op, how recent, target term, keywords to require or exclude, sites, result limit and export folder. It reads the postings, drops full-time and off-field roles, merges repeats, and shows one table. It exports to Excel only after you say yes.

It never applies, saves a job, messages a recruiter or edits your tracker.

## Install

| Where you run it | How |
|---|---|
| An agent that reads skills folders (Claude Code, OpenAI Codex, Gemini CLI, GitHub Copilot, Cursor and the others listed at [agentskills.io](https://agentskills.io)) | Clone this repo into the agent's skills folder as `internship-sweep`, for example `git clone https://github.com/Vidwaansinghania/Internship-Scraper.git ~/.claude/skills/internship-sweep`. Each agent documents its own folder. |
| A chat assistant without skills or a browser | Not suited. The skill depends on loading the job sites and reading them. |

You also need Python 3 and `pip install openpyxl` for the Excel export.

## Usage

Ask your agent:

```
Run an internship sweep.
```

It replies with a short form. Answer with the numbers you want to change:

```
1. finance intern
2. Vancouver, BC, 40 km
6. Winter 2027
```

The table that comes back has these columns: role, company, location, work mode, term, posted date, whether you already track the company, source and link. Say "export" and you get `Internship sweep YYYY-MM-DD.xlsx` with a Results sheet (filterable, live links) and a Search sheet (your settings and run notes).

To export from a saved sweep by hand:

```bash
python scripts/export_xlsx.py sweep.json "internship-sweeps/Internship sweep 2026-10-21.xlsx"
```

The JSON format is at the top of [scripts/export_xlsx.py](scripts/export_xlsx.py). The script never overwrites a file, and it reads the workbook back and fails if the row count is wrong.

## Limits

- **Site terms.** LinkedIn and Indeed restrict automated access in their terms of use. The skill is built to read like a person browsing: logged out, a few pages, a pause between loads, and no workaround if a site shows a sign-in wall or a CAPTCHA. You are responsible for how you use it.
- **Markup changes.** The search parameters and page selectors were tested on 2026-10-21. Both sites change them without notice. When one breaks, the skill is told to say so and fall back to reading the page text.
- **Logged-out views.** Indeed shows no posting date when you are not signed in, so those rows read "Within N days" from your date filter. LinkedIn ignored its date sort for logged-out results.
- **Quality of match.** The filter judges titles and snippets. It keeps borderline titles and flags them instead of dropping them, so skim the Notes column.

## Roadmap

- Pay, where a posting shows it.
- A second pass that opens each posting to fill in term and work arrangement.
- More sites, such as a company career page or a university job board.

## Licence

Apache-2.0. See [LICENSE](LICENSE). Anyone who uses or builds on this work must keep the [NOTICE](NOTICE) file and credit Vidwaan Singhania.
