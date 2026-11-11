---
name: content-engine
description: Runs the weekly content machine end to end: trend radar, then for each of the top three trends a trend report, social posts, infographic (and an animated GIF of it), carousel, sketch and notebook page for each channel, hosts the media, and saves everything to Buffer as drafts. Use when asked to "run the content engine", "do this week's content", or by the weekly scheduled task.
license: Complete terms in LICENSE.txt
---

# Content engine

One run turns this week's top three trends in your field (`config/engine.yaml` → `trends_per_run`) into three ready-to-approve content packs, enough to fill the week. With the default `drafts`, each pack is ten drafts: an infographic for LinkedIn, X and Substack Notes, a carousel for LinkedIn, a hand-drawn sketch for LinkedIn, X and Substack, and a notebook page for LinkedIn, X and Substack, each with its own post. Everything sits in Buffer as **drafts**. Nothing is published. You approve in Buffer.

Usually this runs unattended on a schedule. **Never stop to ask a question.** Where a choice is needed, make the sensible one, write it down in the run's `README.md`, and carry on. If a step fails, finish what you can, record the failure, and still commit (step 9).

Settings live in `config/engine.yaml`. Brand lives in `design.md` and `brand/`. If `config/engine.yaml` still has a blank `author.name`, or `design.md` still contains `TODO`, stop and say: "Run setup first: say 'set up my content engine'." (In an unattended run, write that in the README, commit and finish.)

## Step 0: Set up the run

1. `TODAY=$(date -u +%F)`. Create `runs/$TODAY-radar/` for this week's radar. Each trend gets its own run folder in step 2.
2. Check tools: `python3 -c "import playwright, yaml, PIL"`. Install what's missing with `pip install --break-system-packages playwright pyyaml pillow`. Don't run `playwright install` if Chromium is preinstalled (`/opt/pw-browsers`).
3. Fonts: load the fonts named in `design.md`. Files in `brand/fonts/` load with `@font-face`; for a Google font missing from the machine, `npm i --prefix /tmp/fonts @fontsource/<font-name>` and load its woff2 files.
4. Check Buffer access: `python3 scripts/buffer.py orgs`. If it fails, run everything else and skip step 8, recording why.

## Step 0b: Learn from last week's edits

Run the **learn-from-edits** Skill (unattended mode): `python3 scripts/buffer.py edits --all`, then turn what the owner changed in earlier drafts into [proposed] rules in the voice guide, before this week's posts are written. If Buffer isn't connected yet, skip this and say so in the README.

## Step 0c: Performance review

Run the **performance-review** Skill (unattended mode): fetch last week's numbers, tag new posts, write the report, and add any newly *proven* pattern to `references/performance-rules.md` as **Proposed**. Then read that file's **Confirmed** section: those are the rules the owner has agreed to, and they apply to this run (which graphics, lengths, topics or Note types to favour). Proposed rules and early signals change nothing. If Buffer isn't connected yet, skip this and say so in the README.

## Step 1: Trend radar

Run the **ai-marketing-trend-radar** Skill in radar mode with its saved Settings, with these overrides for unattended runs:
- Skip the Setup offer.
- Skip publishing an artifact and sending files. Save the report as `radar.md` and the data as `radar-data.json` in `runs/$TODAY-radar/`. Building the PDF is optional; skip it if time or usage is tight.

## Step 2: Trend reports (top three)

Pick `trends_per_run` trends from `radar-data.json`, in rank order: highest `score` first (the trend-report Skill's 48-hour shelf-life rule decides only which one leads). Skip any trend that covers the same story as one already picked.

For each pick, create `runs/$TODAY-draft-N/`, copy `radar-data.json` into it, and run the **trend-report** Skill naming that trend. It writes `report.md`. Rename the folder to `runs/$TODAY-<trend-slug>/` (slug: 3–6 lowercase words, hyphenated). If a report says the trend fell apart on checking, delete that folder and take the next trend on the radar instead. If the radar runs out first, carry on with fewer packs and say so in the README.

**Steps 3 to 9 run once per trend, one trend at a time, in rank order:** finish trend 1 all the way to Buffer and commit it before starting trend 2, so a run that is cut short still leaves complete packs. Each trend's files stay in its own folder.

## Step 3: Posts

Run the **social-posts** Skill on this trend's `report.md`. It writes one post per draft slot to `posts.json` and `posts.md`, and reads the earlier trends' `posts.md` so this week's openings don't repeat.

## Step 4: Graphics

All graphics follow `design.md` (it overrides any footer or CTA rules inside the graphic Skills) and use `report.md` as their only source. Save into `media/`:

| File | Made with | CTA (from `design.md` → Channel CTAs, wording from `config/engine.yaml`) |
| --- | --- | --- |
| `infographic-linkedin.png` | **linkedin-infographic-builder** | `linkedin`: follow/save card from `design.md` |
| `infographic-linkedin.gif` | **infographic-animator**, from the LinkedIn infographic's HTML (step 4b) | Same as the PNG: the GIF's last frame is the PNG |
| `infographic-substack.png` | the same design, re-rendered | `substack`: subscribe card. Change only the CTA block, nothing else |
| `infographic-substack.gif` | **infographic-animator**, from the Substack infographic's HTML (step 4b) | Same as the Substack PNG |
| `carousel.pdf` | **linkedin-carousel-builder** (6–8 slides) | Last slide: `linkedin` follow/save slide (dark) |
| `carousel-cover.png` | slide 1 of the carousel, 1080×1350 | (Buffer's document thumbnail) |
| `sketch.png` | **sketch-explainer**, portrait (4:5), from `report.md` → "The shape of the argument" | none (the signature from its Make it yours block, if set) |
| `notebook.png` | **notebook-notes**, portrait (4:5), from `report.md` (the findings or the process) | none |

Build the infographic once as HTML with the CTA as a swappable block, and render it twice. Keep the HTML in `work/`. Skip any graphic that no draft slot uses. The sketch and the notebook page each take a different slice of the report from the infographic (the sketch: the one idea or contrast; the notebook: the findings, steps or rules), so one trend's graphics don't repeat each other. The same sketch and notebook image go to every channel that uses them.

**Variety.** Read `runs/graphics-log.json` (create it if missing). Pick the infographic's layout archetype, main chart type and hero illustration with the infographic Skill's Variety, Hero illustration and Chart chooser rules: not the same archetype as either of the last two runs, not the same main chart type as the last run. The trends in one week count as separate runs, so their infographics all differ, and so do their sketch layouts and notebook looks. Never copy a previous run's `work/*.html` as a starting point. When done, append `{"run": ..., "archetype": ..., "chart": ..., "hero": ..., "sketch_layout": ..., "notebook_look": ...}` to the log (don't repeat last week's sketch layout or notebook look). Same for the carousel: vary chart types across slides, and give the cover a line-art illustration.

**Headlines first.** Before any layout, write three headline options for the infographic and for the carousel cover, following `design.md` → Headlines. Pick the one that passes the stranger test and save all of them in `work/headlines.md`. Then write every carousel slide headline so each one names its subject.

**Check before moving on.** Look at every PNG and at the carousel pages. First, the two hard fails: anything above a headline (eyebrow, kicker, date line or label) on any graphic or slide, and any headline that doesn't name its subject in plain words. Fix those before anything else. Infographic, sketch and notebook PNGs must be exactly 2160×2700 (the 1080×1350 layout at 2×; see the infographic Skill's Output section); carousel slide PNGs and `carousel-cover.png` stay 1080×1350. Check type sizes on the infographics: nothing under 20px in the layout, body 22px+, small text in a medium weight, not regular. Fix overflow, cut-off text, a headline word orphaned on its own line, a missing hero illustration, a layout that repeats the last run's, a chart type that doesn't suit its data, a highlight that misses its words, or colours that don't follow `design.md` (highlighter `--highlight`; highlighted chart series and big numbers `--accent`). Every number on a graphic must appear in `report.md`.

## Step 4b: Animate the infographics

Run the **infographic-animator** Skill on `work/infographic-linkedin.html` once the PNG has passed its checks (animate the final design, never a draft). Use build mode unless its Make it yours block says loop. Write the timeline to `work/motion-linkedin.json`, render to `media/infographic-linkedin.gif` with `--frames-dir work/gif-frames`, and review the frames as that Skill says (frame 0 shows the headline and sign-off; reading order; the count ends on the printed number; last frame matches the PNG; your headshot or logo is in colour; ≤ 4.8 MB). Delete `work/gif-frames/` afterwards.

Then render the Substack version with the same timeline (only the CTA card differs): `work/infographic-substack.html` → `media/infographic-substack.gif` with **`--max-mb 2.4`** (Substack Notes reject GIFs over about 2.5 MB). Check its first and last frames the same way. X keeps its own image.

Don't repeat last week's opening move (if last week opened with a count-up, open with the illustration drawing); note it as `"motion"` in this run's `runs/graphics-log.json` entry. If a GIF can't be made or won't pass review after three rounds, skip it: that post goes out with the PNG (step 6 falls back), and the README says why.

## Step 5: Host the media

Buffer needs a public URL for every image and PDF. They go to your media repo (`config/engine.yaml` → `media.repo`), a public GitHub repo with GitHub Pages switched on.

If `../<media repo name>` isn't cloned next to this repo, clone it: `git clone https://github.com/<media.repo> ../<media repo name>`. (In a routine, add the media repo to the routine's repositories so it is already there.)

Then run `python3 scripts/publish_media.py runs/<run>` (add `--media-repo <path>` if the media repo was cloned somewhere other than next to this repo). It commits only `<path_in_repo>/<run>-<code>/` to the media repo's main branch, pushes, waits until GitHub Pages serves the files, and writes `media-urls.json`. If the files aren't live within the wait, record it, skip step 8, and finish.

## Step 6: Build drafts.json

For each channel and slot in `config/engine.yaml` → `drafts`, add one entry, with `"post": <the post key>` so the edits and performance scripts can tell the drafts apart:

- `text`: `posts.json[<post>].text`
- `first_comment`: `posts.json[<post>].first_comment` when it isn't empty. LinkedIn posts it as the first comment; on X `buffer.py` posts it as the reply in a two-post thread. Every X post's reply is "Source: {main source title} {url}" from `report.md` → Sources, so the post itself stays free of links.
- `asset`:
  - `infographic` → `{"type": "image", "url": <infographic-linkedin.png URL for linkedin, infographic-substack.png URL for x and substack>}`
  - `infographic_gif` → `{"type": "image", "url": <infographic-linkedin.gif URL for linkedin, infographic-substack.gif URL for substack>}`; if that GIF wasn't made this run, use the matching `infographic` PNG instead
  - `sketch` → `{"type": "image", "url": <sketch.png URL>}`
  - `notebook` → `{"type": "image", "url": <notebook.png URL>}`
  - `carousel` → `{"type": "document", "url": <carousel.pdf URL>, "thumbnail_url": <carousel-cover.png URL>, "title": posts.json.linkedin_carousel.document_title}`
  - `none` → `null`

Write it to `runs/<run>/drafts.json`.

## Step 7: Dry run

`python3 scripts/buffer.py drafts runs/<run>/drafts.json --dry-run`. Fix anything it rejects.

## Step 8: Send to Buffer

`python3 scripts/buffer.py drafts runs/<run>/drafts.json`. Every post is saved as a draft. Never change `saveToDraft` and never publish directly. If a channel id is missing from the config, record it and skip that channel.

If Buffer refuses a GIF, the script stops there. Switch that entry's asset to the PNG URL in `drafts.json` and run the same command again: drafts already in `buffer-result.json` are skipped, so nothing is sent twice. Record it in the README.

## Step 9: Write up and commit

`runs/<run>/README.md`:

```markdown
# {Trend name}, week of {TODAY}

**Status:** {Drafts in Buffer | Files only: <reason>}
**Controlling idea:** {from report.md}
**Why this trend:** {score, shelf life, angle}

## In Buffer
| Channel | Post | Asset | Buffer id |

## Check before approving
- First comments are set on the LinkedIn drafts and the X reply carries the source.
- {each figure on the graphics, with its source}
- {anything provisional, e.g. voice guide still waiting for samples}

## Performance
- {posts counted per channel; any newly proven pattern and its proposed rule ("confirm the performance rule about …" or "reject …"); two or three early signals, labelled as such}

## Voice rules learned from your edits
- {each rule added in Step 0b, with its before → after; or "None new". Say "confirm the edit rules" or "drop the rule about …"}

## Teach it your voice
Edit drafts in Buffer as you normally would: next week's run compares them with what it wrote and learns the rules. You can also tell Claude directly ("learn from my edits").

## Decisions made
- {every choice made without asking}

## Problems
- {failures, fallbacks, skipped steps; "None" if clean}
```

Delete `work/` scratch files over 5 MB. Then commit everything in `runs/<run>/` to this repo's `main` with the message `run: {TODAY} {trend name}` and push. Then go back to Step 3 for the next trend.

After the last trend, write `runs/$TODAY-radar/README.md`: the week's trends in rank order, each with its run folder, its Buffer status and draft count, and any trend skipped and why. Commit it with the message `run: {TODAY} radar` and push.

## Step 10: Report back

If a `SendUserMessage` or notification tool is available, send the owner a short summary: the week's trends, how many drafts each put in Buffer, anything to check, and links to the run folders on GitHub. Otherwise end with that summary as the final message.
