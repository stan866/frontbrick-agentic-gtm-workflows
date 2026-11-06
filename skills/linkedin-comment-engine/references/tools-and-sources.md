# Tools, sources and fallbacks

Tool names below are the MCP tool names. Your client may add a prefix (for example `mcp__Blitz__people_enrich` or `mcp__Snitchfeed__snitchfeed_get_usage`). Use whichever prefix is loaded. If tools are deferred, load them with tool search first.

## Required connectors

| Connector | Tools used | Why |
|---|---|---|
| Snitchfeed | `snitchfeed_get_usage`, `snitchfeed_get_linkedin_profile_comments`, `snitchfeed_extract_linkedin_post_comments`, `snitchfeed_list_linkedin_comments_scrapes`, `snitchfeed_get_linkedin_profile_posts`, `snitchfeed_search_linkedin_posts` | What creators comment on, and who comments on a post |
| Apify | `fetch-actor-details`, `call-actor`, `get-actor-run`, `get-dataset-items` | Batch-pull posts from many profiles or post URLs |
| BlitzAPI | `account_key_info`, `people_enrich`, `company_enrich` | Real job title and company size for each sampled commenter |
| Taplio | `find_posts_to_comment`, `list_comment_feeds`, `search_inspiration`, `get_post`, `create_comment_draft`, `commit_comment_draft`, `get_comment_draft` | Extra posts, and scheduling approved comments |
| Google Sheets | `get_values`, `update_values`, `append_values`, `batch_clear_values`, `get_spreadsheet` | Sources, drafts, history |
| Slack | `slack_send_message` | Daily heads-up to the team |

Network: if Claude runs behind a network allowlist, `r2.snitchfeed.com` must be allowed (Snitchfeed stores comment CSVs there).

## Discovery tools (primary → fallback)

| Job | Primary | Fallback |
|---|---|---|
| A person's recent comments | `snitchfeed_get_linkedin_profile_comments` (10 credits/page) | None. If it fails, stop and alert the owner (Hard rule 4). Never skip Phase 1. |
| A person's recent posts | Apify `supreme_coder/linkedin-post` | `snitchfeed_get_linkedin_profile_posts` |
| Who commented on a post | `snitchfeed_extract_linkedin_post_comments` (1 credit/comment, min 100), then download the CSV at `download_url` | Apify `scraping_solutions/linkedin-posts-engagers-likers-and-commenters-download` ONLY when Snitchfeed credits are too low. A failed CSV download is NOT a reason to switch (Hard rule 4). |
| Person and company data | BlitzAPI `people_enrich` + `company_enrich` | None. Headline only for single people BlitzAPI can't find. |
| More posts | Taplio `find_posts_to_comment`, `search_inspiration`; Snitchfeed `snitchfeed_search_linkedin_posts` | None |

## Apify

- Call `fetch-actor-details` before the first run of an actor in a session to get its input schema.
- `supreme_coder/linkedin-post` input: `urls` (full URLs: profile URLs like `https://www.linkedin.com/in/name`, or post URLs like `https://www.linkedin.com/feed/update/urn:li:activity:123`), `limitPerSource`, `scrapeUntil` (date), `deepScrape: true`, `numComments`, `numLikes`. Short forms like `activity:123` return nothing.
- Batch many URLs into one run. Start with `waitSecs: 0`, poll `get-actor-run`, then read with `get-dataset-items` using `fields` to keep output small (`url,authorName,postedAtISO,numLikes,numComments,text,activityDescription,inputUrl`).
- `activityDescription` like "X reposted this" marks a repost. Skip those unless the person added text.

## Snitchfeed

- `snitchfeed_get_linkedin_profile_comments` returns `comments[]` with `text`, `post_url`, `comment_url`, `date_posted`. The true parent post is in `comment_url` (`urn:li:comment:(activity:ID,...)` or `(ugcPost:ID,...)`). Use that ID, not `post_url`.
- A creator with many comments on one post in a day is usually replying on their own post. Drop those.
- `snitchfeed_extract_linkedin_post_comments(post_url, max_comments)` returns a `download_url` only (no comments inline).

### CSV downloads

- Download the CSV with `curl` into a scratch folder and parse every row. Columns: `Author Name`, `Author Headline`, `Profile URL`, `Comment`, `Posted At`.
- If the download is refused (proxy 403 / connection rejected), the domain is not allowed. Stop and alert the owner: "Add r2.snitchfeed.com to the allowed domains." Do not fetch it through another service and do not switch scrapers.
- Scrapes are saved in Snitchfeed. Once access is fixed, re-download from `snitchfeed_list_linkedin_comments_scrapes` instead of paying for a new scrape.

## BlitzAPI (person + company enrichment), mandatory

Every commenter sample in Phase 1 and Phase 3 goes through BlitzAPI. Never score threads from headlines alone. If the tools are missing or a call errors, retry once, then stop and alert the owner (Hard rule 4).

- `account_key_info` (0 records): run once in Phase 0. Must return `"valid": true` with records remaining.
- **Step 1, person:** `people_enrich` with `person_linkedin_url` (1 record). Returns headline, location and full job history. The job with `job_is_current: true` gives the real title and `company_linkedin_url`.
- **Step 2, company (always):** `company_enrich` with that `company_linkedin_url` (1 record). Returns industry, type, size, LinkedIn employee count, HQ and domain. This decides strong vs partial in `icp-scoring.md`.
- **Speed:** BlitzAPI allows about 50 requests per second. Send all person calls for a thread as one batch of parallel calls, then all company calls as a second batch. Never one by one.
- **Cache:** keep `enriched_people.json` and `enriched_companies.json` for the run so nobody is looked up twice.
- If `found` is false or a person has no current company, use their headline for that one person only and note it.

## Taplio

- `create_comment_draft(post_urn, content)` then `commit_comment_draft(id)`. Commit means scheduled, not live.
- `get_post` turns a LinkedIn URL into a post URN when parsing the URL fails.
- `get_comment_draft` confirms status later. `cancel_comment_draft` stops it before it posts.
- One comment per account per post. `COMMENT_ALREADY_POSTED` means skip.
- Max comment length: 1,250 characters.

## Google Sheets

- Read with `get_values`. To add drafts, find the first empty row in column A and write with `update_values` on that exact range. `append_values` can land at row 1001 if the dropdown covers 1,000 rows; if it does, move the rows up and clear the copies.
- `get_spreadsheet` with `fields: ["sheets.properties.sheetId","sheets.properties.title"]` gives the tab `gid` for links.

## Cost guardrails

- Snitchfeed comment scrapes are the expensive step. Cap per run is in `setup.md` (default 16 posts, about 1,600 credits). Check `snitchfeed_get_usage` first.
- Apify: one batched run per source list. Cap posts per profile at 5.
- BlitzAPI: 1 record per person and 1 per company found. Cache to avoid repeats.
- Report credit use per tool in the closing summary.
