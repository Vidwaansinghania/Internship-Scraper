# How it works

**[Overview](../README.md#overview)** · **[How it works](how-it-works.md)** · **[Install](../README.md#install)** · **[Usage](../README.md#usage)** · **[Limits](../README.md#limits)** · **[Roadmap](../README.md#roadmap)**

---

The full instructions are in [SKILL.md](../SKILL.md). This page gives the shape of a run and the reasons behind the less obvious choices.

## The six steps

| Step | What happens |
|---|---|
| 1. Ask | One message with ten questions. Role and region are required; the rest have defaults. If one of the two required answers is missing, it asks once more. |
| 2. Build links | A LinkedIn and an Indeed search URL from the answers: keywords, location, radius, job type, date window and, on LinkedIn, work arrangement. |
| 3. Collect | Load each page, read its text, and run one read-only script in the page that returns every card as role, company, location, date and id. Page through to the result limit or five pages. |
| 4. Filter and dedupe | Drop full-time and off-field roles, apply the user's filters, merge repeats across sites, and match against a tracker spreadsheet if the user names one. |
| 5. Show | One summary line, then one table. Anything that stopped early is listed under it. |
| 6. Export | Only after a yes. Write a JSON file, run `scripts/export_xlsx.py`, and read the workbook back to check the row count. |

## Why it is built this way

<details>
<summary>One script per page, not one link at a time</summary>

A search page holds up to 25 cards. Reading each link separately means one tool call per posting. A single in-page query returns the whole page and keeps the number of page actions low, which also fits the rule to browse the way a person would.
</details>

<details>
<summary>Indeed's selector</summary>

The obvious selector, any link containing `jk=`, returned eight links for two real postings on the test page: salary pages, company pages and a hidden placeholder card with a made-up key. The skill selects `.job_seen_beacon a[data-jk]` and ignores cards with no height.
</details>

<details>
<summary>Dates</summary>

LinkedIn cards carry an absolute date in a `time` element, so no relative text needs converting. Indeed shows no date to a logged-out reader, so the skill records the window the user set ("Within 7 days") instead of inventing a date.
</details>

<details>
<summary>Borderline titles are kept</summary>

A strict title match would drop a "Co-Op Staff Accountant" from a "finance intern" search. The skill drops roles that are clearly wrong and flags the doubtful ones in the Notes column, so the user makes the call.
</details>

<details>
<summary>Tracker match on company</summary>

Role cells in a hand-kept tracker are long and uneven, so an exact company and role match almost never fires. The skill matches on company and reports the tracker's role beside it, for example `Company tracked: Acme Corp (Finance Analyst Co-op)`.
</details>

## What the script checks

`scripts/export_xlsx.py` writes two sheets and then reopens the file.

- Text that begins with `=`, `+`, `-` or `@` is stored as text, so a posting title can't run as an Excel formula.
- Only `http` and `https` links become hyperlinks.
- A `posted` value in `YYYY-MM-DD` becomes a date cell. Other text stays text.
- If the output name is taken, it writes `name (2).xlsx` and leaves the old file alone.
- If the saved row count differs from the input, it exits with an error.
