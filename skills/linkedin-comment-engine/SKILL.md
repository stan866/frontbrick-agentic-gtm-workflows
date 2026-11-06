---
name: linkedin-comment-engine
description: Daily LinkedIn commenting engine with a human in the loop. Draft mode finds 20-30 fresh LinkedIn posts where the account owner's ideal buyers (ICP) are reading (via Snitchfeed, Apify, BlitzAPI and Taplio), drafts comments in the owner's voice and logs them to a Google Sheet for review. Process mode reads the sheet's Approval Stage column, schedules Approved comments in Taplio, rewrites Modify rows from the edit notes, and skips Declined rows. Use this whenever someone asks for daily LinkedIn comments, an engagement run, "comments for today", "draft today's comments", "process today's drafts", "schedule approved comments", social selling, or commenting on ICP posts, even if they do not name the skill.
---

# LinkedIn Comment Engine

Goal: every day, 20-30 new comments on LinkedIn posts where the account owner's ICP is reading, each one adding the owner's own expertise. Comments are the product. Discovery is only there to pick the right posts.

Why this matters: a comment in the right thread puts the owner in front of buyers who never saw their posts. A generic comment does nothing, and a salesy one hurts. So posts are picked by audience quality, and comments are written like an operator who has done the work.

"The owner" means the person whose LinkedIn account posts the comments. "The team" means everyone who reviews drafts (it can be just the owner).

## Before the first tool call

Read these files:
- `references/setup.md`: the owner's settings (sheet IDs, Slack channel, where their inputs live, thresholds).
- `references/your-inputs.md`: what the owner must provide (ICP and expertise) and how to read it.
- `references/sheets-spec.md`: the exact sheets this skill uses, and Setup mode.
- `references/tools-and-sources.md`: exact tools, fallbacks and credit costs.
- `references/icp-scoring.md`: how commenters are picked and threads are scored.

Before drafting, also read `references/voice-guide.md`.

This skill ships with no ICP, no expertise and no sheets. Those come from the owner. If `setup.md` still has placeholder values, run Setup mode (in `sheets-spec.md`) when someone is there, or stop and alert the owner in a scheduled run.

## Hard rules (non-negotiable, read first)

These override anything else in this skill or its reference files.

1. **Never skip a step.** Run every phase and every numbered step in order. "Saving time", "saving credits" or "the result is probably good enough" are never reasons to skip. A step is only skipped when this skill says it is conditional (Phase 3 and Phase 4 run only when under 25 posts).
2. **BlitzAPI enrichment is mandatory, person AND company.** Every thread scored in Phase 1 and Phase 3 must have its commenter sample (up to 15 people) enriched through the BlitzAPI MCP: `people_enrich` for every person, then `company_enrich` for each person's current company. Never score a thread from LinkedIn headlines alone. Headlines are only allowed for individual people BlitzAPI returned no data for.
3. **Snitchfeed comment scrapes must be read in full.** After `snitchfeed_extract_linkedin_post_comments`, download the CSV from the `download_url` it returns (or from `snitchfeed_list_linkedin_comments_scrapes`) and use those commenters. Do not swap in another scraper because the download did not work.
4. **If a required tool or connection fails, stop and alert the owner.** This applies to BlitzAPI, Snitchfeed (calls or CSV download, for example a blocked `r2.snitchfeed.com`), Google Sheets and Slack. Retry once. If it still fails:
   - Do not continue that phase with a fallback, and do not log or post partial results as if the run were complete.
   - In a scheduled run, send a notification right away that names the tool, the exact error and what the owner needs to do (for example "reconnect BlitzAPI" or "add r2.snitchfeed.com to the allowed domains").
   - In a live session, say the same thing in chat and wait.
5. **Fallbacks are only for the cases this skill names.** The only automatic fallback is Snitchfeed credits running low (Phase 0 step 5). Every other failure follows rule 4.
6. **Never mention anyone's name in a comment.** No author names, no tagging, no names of people from the post. The comment talks to the idea, not the person.
7. **Every comment must show expertise to the ICP.** Only comment on posts where the owner can add real know-how on one of the `topics` from the owner's expertise input. No off-topic posts, even from ICP people.
8. **Don't stop below 20.** Keep working through the volume loop in "Targets and judgment calls" until there are at least 20 good drafts (aim 25, max 30). Tool failures still follow rule 4.
9. **Report honestly.** The closing summary and the Slack message must list any step that did not run as written, and why. Never call a run complete when a step was skipped.

## Pick the mode

- **Draft mode**: "find posts", "draft today's comments", "daily comments", or the morning scheduled run. Run Phases 0-6. Never schedule anything in Taplio in this mode.
- **Process mode**: "process today's drafts", "schedule approved", "handle the Modify ones", or the processing scheduled run. Skip discovery and go straight to "Process mode" at the bottom.
- **Setup mode**: "set up the LinkedIn comment engine", or any run where `setup.md` has no sheet IDs. Follow "Setup mode" in `references/sheets-spec.md`.
- If the request is unclear and someone is there, ask which one. In a scheduled run, follow the mode the task prompt names.

## The Drafts tab is the approval system

Output sheet, tab `Drafts`. The team reviews here, not in chat, because nobody else can open the session that made the drafts.

| Col | Header | Values |
|---|---|---|
| A | Date MM/DD/YYYY | Day the draft was made |
| B | LinkedIn Post | Post URL |
| C | Comment | The draft. A teammate may edit this text directly. |
| D | Approval Stage | `Draft/Pending Approval`, `Approved`, `Modify`, `Declined/Drop`, `Scheduled` |
| E | Edit Notes for Claude | What to change when D is `Modify` |

Use the status strings exactly as written; D is a dropdown. Read the header row first every run. If columns moved, follow the headers, not the letters.

When writing rows, write into the first empty rows right under the existing data (read column A to find it, then use `update_values` on that exact range). Do not rely on `append_values` alone: if the dropdown is set on many empty rows, append can land at the bottom of the sheet (for example row 1001). If that happens, move the rows up and clear the copies.

## Draft mode, in order

Create a task list with one item per phase. Keep a running candidate list in a local file (`candidates.json`) so nothing is lost between phases: post URL, author, author headline, post text, post date, likes, comments, source phase, ICP notes, priority score.

### Phase 0: Setup and dedupe (always)

1. Get today's date.
2. Read the owner's ICP and expertise inputs (locations in `setup.md`) and extract them into `inputs.json` as described in `references/your-inputs.md`. These drive commenter picking, scoring, topics and what comments can claim. If either can't be read, follow Hard rule 4. Never fill gaps with guesses.
3. Read the output sheet: tab `Comments we left on posts` (A:B) and tab `Drafts` (A:E). Build a set of post URLs already commented or drafted. Normalize URLs before comparing: strip query strings, and compare by the activity / ugcPost / share ID inside the URL, because the same post shows up under different URL shapes.
4. Read the full `Comments we left on posts` tab as the live style corpus. It grows every day and is the best signal of the owner's voice. The newest rows matter most.
5. Check Snitchfeed credits with `snitchfeed_get_usage`. If credits are too low for this run (see cost guardrails in the tools file), switch to the Apify fallbacks for the rest of the run and say so in the summary and the Slack message. If a Snitchfeed call fails with an error (not low credits), follow Hard rule 4.
6. Check that BlitzAPI is reachable: call `account_key_info`. It must return `"valid": true` with records remaining. If the tools are missing or the call fails, follow Hard rule 4 before any discovery.

### Phase 1: Follow the creators' comments (tab `Fetch Their Latest Comments`)

For each profile URL in column A:
1. `snitchfeed_get_linkedin_profile_comments` (page 1). Keep only comments made in the last 24 hours. Drop comments they left on their own posts (replies to their commenters).
2. Collect the parent posts they commented on. These are threads the right crowd is already in. Read the real post URN from each `comment_url` (the `urn:li:comment:(activity:...` or `(ugcPost:...` part), because the `post_url` field can point to the wrong ID.
3. Get each post's text, author, likes and comments (Apify `supreme_coder/linkedin-post`, all URLs in one batched run, as full `https://www.linkedin.com/feed/update/urn:li:...` URLs). Rank by likes + comments, applying the engagement priority in "Targets and judgment calls". Drop posts that are not on one of the owner's `topics`.
4. For the top posts (about 2 per creator), check who is in the thread: `snitchfeed_extract_linkedin_post_comments` with `max_comments: 100`, then download the CSV from `download_url` and read every row. If the download fails, follow Hard rule 4.
5. Pick up to 15 commenters per post by job title, not at random. Follow "Picking the 15 commenters" in `references/icp-scoring.md`. Then enrich those picks with the BlitzAPI MCP:
   a. `people_enrich` for each commenter's LinkedIn URL, all calls sent in parallel.
   b. `company_enrich` for each person's current company (`company_linkedin_url` from the current job), all calls sent in parallel.
   c. Grade each person with the real title plus company industry and size. This step is mandatory (Hard rule 2).
6. Score the thread with `references/icp-scoring.md` and keep only threads that pass.

This is directional. The point is to rank posts, not to build a lead list.

### Phase 2: Comment on ICP posts (tab `ICP To Engage With`)

These people ARE the ICP, so no engager check is needed. But only comment where the owner can show expertise. A post qualifies only if it is about one of the owner's `topics`.
1. Take all URLs from the `LinkedIn Profile URL` column.
2. Fetch their posts from the last 7 days with Apify `supreme_coder/linkedin-post` in one batched run (`limitPerSource: 5`, `scrapeUntil` = 7 days ago).
3. Drop reposts with no added text, posts already in the dedupe set, and every post that is not about one of the topics: personal life, jokes, holidays, job changes, funding or award announcements, pure product promos, webinar or event invites, and anything outside the topic list. Do not stretch a loose link to justify a comment.
4. One comment per person per day. If someone posted several times, pick the post where the owner can show the most expertise. When two posts are similarly strong, pick the one with more likes + comments.
5. Rank the remaining posts by topic fit, then by engagement. Posts under the engagement threshold are fine if the topic is a strong fit.

If Phase 2 alone yields more than 30 strong posts, keep the best 30.

### Phase 3: Fill the gaps (if under 25 after Phases 1-2)

Work down `Priority Engagement List` by Rank, then `Weekly watchlist` by Rank (layout in `sheets-spec.md`).
1. Fetch each person's latest posts with Apify `supreme_coder/linkedin-post` (all profiles in one run) or Snitchfeed `snitchfeed_get_linkedin_profile_posts`. Keep posts from the last 48 hours; early comments get seen more. Check posts above the engagement threshold first.
2. Run the same audience check as Phase 1 steps 4-6.
3. Stop when the total reaches 30 good posts.

### Phase 4: Taplio (if still under 25)

Use Taplio `find_posts_to_comment` (`source: for_you`, then each curated feed from `list_comment_feeds`) and `search_inspiration` with each of the owner's `topics`. Prefer posts under 48 hours old, above the engagement threshold, with real discussion in the comments.

### Phase 5: Draft

Load context once per run:
- `references/voice-guide.md` and the owner's expertise input (`claims` in `inputs.json`).
- The live `Comments we left on posts` tab (from Phase 0).
- Any `context_sources` the owner gave (newsletter, videos, blog, past posts), searched once per topic cluster, not per post.

For each post, write one comment. Each comment must:
- React to something specific in the post, not the topic in general.
- Add one thing the author did not say: a process the owner runs, a number the owner has stated, a counterpoint, or a sharp question.
- Sound like the owner (voice guide plus history tab). No em dashes, ever.
- Never mention anyone by name.
- Never pitch, never link, never say "DM me" or "book a call".
- Only use facts and numbers that appear in the owner's inputs, the history tab or the owner's own content. Never invent a result.

Vary length and openings across the batch. If five comments in a row start the same way, rewrite some.

Before logging, check every draft: no em dashes or en dashes, no links, no names, under 1,250 characters.

### Phase 6: Log drafts and notify

1. Write every draft to `Drafts`: A today's date (MM/DD/YYYY), B post URL, C comment, D `Draft/Pending Approval`, E empty. Then read the tab back to confirm the rows are where you expect.
2. Post one Slack message to the channel in `setup.md` (or the one the task prompt names). Use clean formatting: a bold title, the number of drafts, the split by source (creator threads, ICP posts, watchlist, Taplio), how many are above the engagement threshold, a link to the Drafts tab, and the reminder as a quote: "Mark each row Approved, Modify (+ note in column E), or Declined/Drop."
3. Stop. Draft mode never schedules. A comment goes out under the owner's name, so a human approves each one first.

Closing summary: drafts logged, skipped posts and why, credit use per tool, and any step that did not run as written.

## Process mode

Runs on request or on a schedule. Uses the Taplio connection of whoever runs it, which must post as the owner.

1. Read `Drafts` (all rows) and `Comments we left on posts` (column A, for dedupe).
2. Go row by row by column D:
   - **`Approved`**: schedule the text in column C exactly as it is now (a teammate may have edited it):
     a. Skip if the post URL is already in `Comments we left on posts`, and set D to `Scheduled`.
     b. Get the post URN from the URL (`activity-7510...` → `urn:li:activity:7510...`, `ugcPost-...` → `urn:li:ugcPost:...`, `share-...` → `urn:li:share:...`). If Taplio rejects it, call Taplio `get_post` with the URL and use the URN it returns.
     c. `create_comment_draft`, then `commit_comment_draft`. If Taplio returns `COMMENT_ALREADY_POSTED`, treat it as done.
     d. Add to `Comments we left on posts`: A post URL, B final comment. Only scheduled comments go there; it is the style training data.
     e. Set D to `Scheduled`.
   - **`Modify`**: rewrite column C following column E, keeping the owner's voice and the drafting rules. Write the new text to C, set D back to `Draft/Pending Approval`, and clear E. Never schedule a rewrite in the same run.
   - **`Modify` with an empty E**: leave it and list it in the summary as "needs a note".
   - **`Declined/Drop`**, **`Draft/Pending Approval`**, **`Scheduled`**: leave them alone.
3. If the team edited C and marked `Approved`, that edit is final. Do not "improve" it. If the edit has an em dash, schedule it anyway and mention it in the summary.
4. Post one short Slack update to the same channel: X scheduled (with Taplio's scheduled window), Y rewritten and waiting for review, Z Modify rows missing a note, any failures. Never say a comment is live; Taplio queues it.

If nothing is `Approved` or `Modify`, post nothing to Slack and end quietly.

## Targets and judgment calls

- Volume: aim for 25. Hard floor 20, hard cap 30. A run is not finished below 20.
- Volume loop (run in order until there are 20+ good drafts, then keep going toward 25 while good posts remain):
  1. Phase 1: go further down the ranked creator-thread list (more than 2 posts per creator), and fetch page 2 of their comments.
  2. Phase 3: go further down the Priority list and the full Weekly watchlist, and widen the window to 72 hours.
  3. Phase 4: every Taplio comment feed, then `search_inspiration` with each topic as its own query.
  4. Snitchfeed `snitchfeed_search_linkedin_posts` with topic keywords (last 48 hours), then run the audience check on the best results.
  5. Relax the engagement threshold before relaxing anything else.
  Never relax the topic rule, the audience check, the no-names rule or dedupe to hit the number. If every step is done and it is still under 20, report which sources ran and how many posts each produced, and alert the owner.
- Mix: at least half of the comments should be on posts by ICP people or on threads with strong ICP presence. Creator threads with no buyers in them are the first thing to cut.
- Freshness: creator and watchlist posts under 48 hours. ICP posts up to 7 days.
- Engagement priority: ideally at or above the threshold in `setup.md` (default 50 likes + comments combined). This is a strong preference, not a hard cutoff:
  - In Phases 1, 3 and 4, rank and pick posts above the threshold first. Use posts below it only when there are not enough good ones to reach 20, and only if the audience check passes.
  - A fresh post (under ~6 hours) with fast early traction can count, since it is still climbing.
  - Audience quality still comes first: a busy post with no buyers in the thread is cut; a smaller post full of the right people can stay.
- Skip a post when the owner has nothing real to add. A skipped post costs nothing; a filler comment costs credibility.
- If a tool fails, follow Hard rule 4. Low Snitchfeed credits are the only case where the run switches to the Apify fallback and keeps going.
