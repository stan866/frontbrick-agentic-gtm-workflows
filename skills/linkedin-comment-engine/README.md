# LinkedIn Comment Engine

Get seen by the right people on LinkedIn every day, without spending hours scrolling.

## What it does for you

Good comments are one of the easiest ways to grow on LinkedIn. When you leave a smart comment on a busy post, everyone reading that post sees your name. Many of them click your profile. Some follow you. Over time, the right people start to know who you are.

The hard part is doing it every day. You have to find the right posts, check who is reading them, and write something worth reading. That takes hours.

This skill does that work for you:

1. **It finds the right posts.** Every day it looks for 20-30 fresh posts where your buyers are reading.
2. **It checks who is in the comments.** It looks up the people commenting and keeps only posts where your buyers actually show up.
3. **It writes a draft comment for each post.** The comment adds a real idea from your own experience. No "Great post!" filler, no selling, no links.
4. **You approve every comment.** Drafts land in a Google Sheet. You (or your team) mark each one Approved, Modify or Declined.
5. **It schedules only what you approved.** Approved comments get scheduled on your LinkedIn account. Comments you want changed get rewritten and sent back to you for another look.

The result: more of the right people see your name every day, your profile gets more visits, and your posts reach more people over time.

## Why a human in the loop matters

AI can find posts and write drafts fast. But every comment goes out under your name, so a person checks each one first.

- Nothing posts without your OK.
- You can edit any draft right in the sheet before you approve it.
- Every comment you approve gets saved, and the skill learns your style from those saved comments.

This is what keeps the quality high. You get the speed of AI and the judgment of a person.

## Works for one person or a whole team

- **On your own:** you review the sheet once a day, mark your choices, and you're done.
- **As a team:** one person sets it up. Teammates review drafts in the shared sheet and leave notes. A daily Slack message tells everyone when new drafts are ready.

It also works if a team runs comments for a founder or an executive. The drafts are written in that person's voice and go out from their account.

## How a day looks

| When | What happens | Who |
|---|---|---|
| Morning | Claude finds posts, checks the audience, drafts 20-30 comments, adds them to the sheet and posts a Slack message | Claude (on a schedule) |
| Anytime | Mark each row Approved, Modify (with a note) or Declined/Drop | You or your team |
| Later in the day | Claude schedules the approved comments, rewrites the Modify rows and saves what went out | Claude (on a schedule) |

## What you need

### 1. Claude

A Claude plan that supports skills and connectors, like the Claude app (with Cowork) or Claude Code.

### 2. Tools and connectors

| Tool | What it's used for | How to connect |
|---|---|---|
| **Snitchfeed** | Sees what people comment on, and pulls the list of people commenting on a post | Connector (MCP) |
| **Apify** | Pulls recent posts from a list of LinkedIn profiles | Connector (MCP) |
| **BlitzAPI** | Looks up each commenter's real job title and company size | Connector (MCP) |
| **Taplio** | Schedules approved comments on your LinkedIn account, and finds extra posts if needed | Connector (MCP) |
| **Google Sheets** | Your source lists, your drafts and your history of approved comments | Connector (MCP) |
| **Slack** | Daily heads-up when drafts are ready | Connector (MCP) |

Snitchfeed, Apify, BlitzAPI and Taplio need their own paid accounts.

If Claude runs in the cloud with a network allowlist, add `r2.snitchfeed.com` to the allowed domains. That's where Snitchfeed stores the commenter lists.

### 3. Your own ICP and expertise

This skill comes with the process and the tools. It does not come with anyone's business details. You bring two things, in any format you like (Google Doc, PDF, Notion page, a file in your Claude Project, or just notes):

- **Your ICP:** who your buyers are. Job titles, company types and company size.
- **Your expertise:** the topics you know, your opinions, the steps you follow, and any numbers you're happy to share in public.

The skill reads these every day. It only comments on your topics, only counts your buyers, and only uses facts you gave it. It never fills gaps with guesses. See [`references/your-inputs.md`](references/your-inputs.md) for exactly what to include.

Your voice comes from you too. Every comment you approve is saved, and the skill learns how you write from those.

### 4. Two Google Sheets (Claude can build them)

You don't have to build these by hand. Say **"Set up the LinkedIn comment engine"** and Claude creates both sheets with the right tabs and columns:

- **Sources sheet:** creators to follow, the buyers whose posts you want to comment on, and backup lists.
- **Output sheet:** the `Drafts` tab where you approve comments, and the history of comments that went out.

Then you add the people you want to watch. The exact layout is in [`references/sheets-spec.md`](references/sheets-spec.md).

## How to run it

Say to Claude:

- **"Set up the LinkedIn comment engine"** the first time. Claude asks for your ICP and expertise, builds your sheets and checks your tools.
- **"Draft today's comments"** to find posts and write drafts.
- **"Process today's drafts"** to schedule what you approved and rewrite what you marked Modify.

Or set both as daily scheduled tasks: drafting in the morning, processing in the afternoon.

## The rules it always follows

- 20-30 drafts a day. It keeps searching until it has at least 20 good ones.
- Only posts on your topics, where you can show real expertise.
- Only posts where your buyers are actually reading.
- Prefers busy posts (50+ likes and comments).
- Never mentions people by name. Never pitches, never links.
- Never makes up numbers or results.
- Never schedules anything you didn't approve.
- If a tool breaks, it stops and tells you instead of cutting corners.

## What's in this folder

| File | What it is |
|---|---|
| `SKILL.md` | The exact steps Claude follows |
| `references/setup.md` | Your settings: sheet links, Slack channel, where your inputs live |
| `references/your-inputs.md` | What you need to provide: your ICP and your expertise |
| `references/sheets-spec.md` | The sheets Claude builds for you, and the first-time setup |
| `references/voice-guide.md` | How comments should sound |
| `references/icp-scoring.md` | How it decides if a post's audience is right |
| `references/tools-and-sources.md` | Exact tools, settings and costs |
