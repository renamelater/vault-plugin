# Notion Ingest

Read when the content to ingest is a Notion URL. Tool access and the no-tools branch: `../../references/notion.md` — read it first.

## Fetch and detect the shape

Fetch the URL once. The result reveals the **shape**, which picks the branch below: a **database** (schema and rows / data sources), a **page with children** (child-page links in the content), or a **plain page**.

## Plain page

The fetched markdown is the ingest content. Record the URL in `source:`. Rejoin `SKILL.md` at Step 2 (classify) and run the normal pipeline.

**Blank or property-only page** — database-row pages often carry no body: the substance lives in properties and file links. There is nothing to curate, so report what the page holds (properties, links, people) and ask whether to ingest that as a **stub** — a note whose content is the properties table and the links — or skip. Curation resumes only when a page has prose.

## Page with children: the gate

A pasted URL commits this ingest to exactly one page — the pasted one. Everything below it passes through the **gate**:

1. Ingest the pasted page itself as a plain page.
2. Count its direct children from the already-fetched content — titles only; grandchildren stay unfetched.
3. Report the count and a numbered list of titles. Ask which to include: specific numbers, none, or all. "All" is confirmed by restating the scale: "all 14 — that's 14 curated vault files."
4. Each selected child runs the full pipeline (`SKILL.md` Steps 2–12) as its own ingest. A child that turns out to have children of its own fires this gate again at its level.

Fan-out happens only through a numbered selection the user just made.

## Database

One curated file for the whole database — never one file per row:

- `## Schema` — the properties (name and type)
- `## Rows` — a curated markdown table of the rows worth keeping; the snippet rule from `SKILL.md` applies. State the total row count and what was left out.
- Type defaults to `note` unless the content clearly reads as another canonical type. Record the database URL in `source:`.

Rejoin `SKILL.md` at Step 4 (target folder) with that type.

## Done

Every selected item is filed and indexed, and the confirm step (`SKILL.md` Step 12) reports one line per file created.
