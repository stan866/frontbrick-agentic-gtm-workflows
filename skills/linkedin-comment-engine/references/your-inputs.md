# Your inputs (you provide these, the skill never makes them up)

The skill knows the process and the tools. It does not know your business. Two things must come from you, in your own words. Without them, the skill stops and asks.

They can be in any format: a Google Doc, a Google Sheet, a PDF, a Notion page, a Markdown file, files in a Claude Project, or notes you paste into the chat. Tell the skill where they are in `setup.md`.

## 1. Your ICP (who your buyers are)

Describe the people you want in front of. Include, in any order:

- **Job titles** of the people who buy or champion what you sell (and close variations).
- **Company types** (for example B2B SaaS, agencies, manufacturers).
- **Company size range** (employee count) that counts as a fit.
- **Who is NOT your buyer**, if it's not obvious (for example your competitors, or peers who do the same work as you).
- Optional: industries, regions, pains, buying triggers.

One doc with a few buyer types is enough. More detail means better picks.

## 2. Your expertise (what you can say)

This is where every comment gets its substance. Include:

- **Topics** you know well enough to comment on with real experience. Only posts on these topics get a comment.
- **Your takes:** the opinions you hold and why.
- **Your processes:** things you do step by step that others can learn from.
- **Numbers and results you're OK repeating in public.** The skill will only use numbers that appear in your inputs or your own content.
- Optional: links to your newsletter, YouTube channel, blog or past posts, so the skill can find your angle on a topic.

## 3. Your voice (grows on its own)

You don't need to write a style guide. Every approved comment is saved to the `Comments we left on posts` tab, and the skill learns your voice from it. To start faster, paste 10-20 of your own past LinkedIn comments into that tab.

## How the skill uses your inputs (instructions for the agent)

At the start of every Draft mode run:

1. Open the ICP input and the expertise input from the locations in `setup.md`, using whatever tool fits the format (Sheets, Docs, Drive, Notion, project files, a local file, or the chat).
2. Extract and keep for the run (in `inputs.json`):
   - `personas`: job titles and company types that count as buyers
   - `size_range`: min and max employee count
   - `not_icp`: who to exclude
   - `topics`: the list of topics allowed for comments
   - `claims`: takes, processes and numbers the owner has stated
   - `context_sources`: optional newsletter, video or blog sources
3. If either input is missing, empty, or can't be opened, stop and ask the owner for it (Hard rule 4). Never fill the gaps with guesses, examples or general knowledge.
4. If something important is unclear (for example no company size), ask once in a live session. In a scheduled run, use what is there, and list the missing piece in the summary and the Slack message.
