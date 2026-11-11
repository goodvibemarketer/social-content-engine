---
name: social-posts
description: Writes the author's LinkedIn posts, X posts and Substack Notes from a trend report (one per graphic and channel), in their voice, with the right CTA for each channel. Use in the content engine after trend-report, or when asked to draft social posts from a brief or article.
license: Complete terms in LICENSE.txt
---

# Social posts

Writes one trend's posts from `report.md`, in the author's voice (`config/engine.yaml` → `author`), one per draft slot in `config/engine.yaml` → `drafts` (ten with the default slots: four LinkedIn, three X, three Substack Notes). The content engine runs this once per trend, three trends a week. Output goes to `posts.json` (for Buffer) and `posts.md` (for reading).

## Before writing

1. Read `report.md`. It is the only source of facts. Nothing goes in a post that isn't in it.
2. Read `references/voice.md` in full, then `references/samples.md` (the author's best posts, ranked). Write like the top three there, not like a LinkedIn template. **Learned from edits** (at the end of `voice.md`) wins over everything else in this Skill: those are the author's own corrections.
3. Read `config/engine.yaml` → `cta` for each channel's call to action.
4. Read `.claude/skills/content-engine/references/performance-rules.md` → **Confirmed** (rules the author agreed to from their own numbers, e.g. a length target). Ignore the Proposed and Rejected sections.
5. If this run has already written posts for an earlier trend, read their `posts.md` (sibling run folders from the same date). This week's openings must not repeat.

## The posts

| Key | Channel | Goes with | Length | Shape |
| --- | --- | --- | --- | --- |
| `linkedin_infographic` | LinkedIn | the infographic GIF | 90–160 words | News hook (≤ 12 words) → a number → the turn → the author's view → CTA |
| `linkedin_carousel` | LinkedIn | the carousel PDF | 70–130 words | A different hook from the infographic post → why swipe → CTA |
| `linkedin_sketch` | LinkedIn | the sketch | 60–120 words | The sketch's one idea in a line → why it matters to the reader → the author's view → CTA |
| `linkedin_notebook` | LinkedIn | the notebook page | 70–130 words | "My notes on…" framing → the two or three points worth saving → CTA |
| `x_infographic` | X | the infographic (PNG) | ≤ 270 characters | One sharp claim + one fact. No link: the source goes in the reply (`first_comment`) |
| `x_sketch` | X | the sketch | ≤ 270 characters | The sketch's punchline, in words. Source in the reply |
| `x_notebook` | X | the notebook page | ≤ 270 characters | One point from the notes + why it matters. Source in the reply |
| `substack_infographic` | Substack Notes | the infographic GIF | 60–150 words | Conversational, peer to peer: open on the news or the number itself (never "Something I noticed this week") → CTA |
| `substack_sketch` | Substack Notes | the sketch | 40–100 words | The idea in plain words → one practical close → CTA |
| `substack_notebook` | Substack Notes | the notebook page | 60–150 words | What the notes boil down to → the author's take → CTA |

**Ten posts, ten openings.** The posts for one trend share the report's controlling idea, but each comes at it from a different door: the number, the turn, the practical how-to, the contrarian view, a question the data answers. No two posts in this run (or in the week's earlier trends) start with the same words, the same hook type on the same channel, or the same lead number. Each post describes its own graphic, not the infographic.

Only write the keys that appear in `config/engine.yaml` → `drafts`.

## CTAs

- **LinkedIn (`cta.linkedin`, type follow_save).** End every LinkedIn post with `cta.linkedin.lines`, with `{author.name}` and `{author.linkedin_url}` filled in from the config, each on its own line, after a blank line. No comment keyword, no lead-magnet offer.
- **Substack Note (`cta.substack`).** End with `cta.substack.line` (type `subscribe`), or no CTA line (type `none`). No link needed; Notes show a subscribe button.
- **X (`cta.x`).** End with `cta.x.line`, or nothing if type `none`. Set every X post's `first_comment` to "Source: {main source title} {url}" from `report.md` → Sources: it becomes the reply.
- **LinkedIn sketch and notebook first comments.** `first_comment` is "Source: {main source title} {url}" from `report.md` → Sources.
- **Never add a CTA of your own.** No extra subscribe line, free-download offer or "more every week" plug beyond what `config/engine.yaml` says: repeated every week, they read as false.

## Rules

- **Facts only from `report.md`,** and keep their numbers exact. A post may leave a fact out. It may not round it, stretch it or add one.
- **One idea per post:** the report's controlling idea, through its chosen angle.
- **The first line earns the second.** Specific beats clever: a number, a named change, a claim someone could disagree with. A question opener only if the next line answers it with data ("Is the CMO role dying? The data seems to suggest so."). Never "In today's world", "Let's talk about", or "Here's the thing".
- **Spelling per `author.language`.** No hashtags. No emoji except the ➕ and 🔖 in the LinkedIn CTA lines, and one 👇 if pointing to the comments.
- **No links in LinkedIn post bodies** (they cut reach), except the profile link in the CTA. If a source matters, write "link in the first comment" and put the URL in `first_comment`.
- **No invented experiences or beliefs.** Never give the author a memory, a past belief or a story they haven't told ("I'd have bet on…", "Last week I tried…"). Their view comes from `author.expertise` and the report's take, stated as a view.
- **The voice guide wins.** Follow `voice.md` → Learned from edits and Used to, now don't, even where this table says otherwise.
- **Pass every post through the no-ai-slop Skill** (edit mode, preserve voice) before saving, and leave alone anything `voice.md` lists under "Keep, even if it looks like AI". Re-check lengths afterwards.

## Output

`posts.json`:
```json
{
  "linkedin_infographic": {"text": "...", "alt_hooks": ["...", "..."], "first_comment": "" },
  "linkedin_carousel":    {"text": "...", "alt_hooks": ["...", "..."], "first_comment": "", "document_title": "≤ 60 characters, shown on the carousel post"},
  "linkedin_sketch":      {"text": "...", "alt_hooks": ["...", "..."], "first_comment": "Source: ..." },
  "linkedin_notebook":    {"text": "...", "alt_hooks": ["...", "..."], "first_comment": "Source: ..." },
  "x_infographic":        {"text": "...", "first_comment": "Source: ..."},
  "x_sketch":             {"text": "...", "first_comment": "Source: ..."},
  "x_notebook":           {"text": "...", "first_comment": "Source: ..."},
  "substack_infographic": {"text": "..."},
  "substack_sketch":      {"text": "..."},
  "substack_notebook":    {"text": "..."}
}
```

`posts.md`: each post under its key as a heading, with the alternative hooks listed under it, so the author can swap a hook in Buffer.
