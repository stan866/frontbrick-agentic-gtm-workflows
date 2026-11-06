# Scoring a thread's audience (directional)

Source of truth: the owner's ICP input (see `your-inputs.md`), extracted into `inputs.json` at the start of the run: `personas`, `size_range`, `not_icp`. This file only defines the method. Never add job titles, company types or sizes that the owner did not give.

## Grades

**Strong match (2 points)**
- A job title that matches one of the owner's personas (or a clear equal, for example "Head of X" for "VP of X"), AND
- at a company type from the owner's ICP, within `size_range`.

**Partial match (1 point)**
- Right title, company outside `size_range`.
- One level below a persona title (for example manager level) at an ICP company.
- Right title at a company type close to, but not listed in, the owner's ICP.

**Not ICP (0 points)**
- Anyone on the owner's `not_icp` list.
- People who sell the same thing as the owner (peers and competitors).
- Students, job seekers, recruiters.
- The tool vendor's own staff commenting on their product's post.

## Picking the 15 commenters (not random)

The sample is chosen on purpose, from job titles, before any enrichment:

1. Take every unique commenter from the full Snitchfeed comment CSV (`Author Name`, `Author Headline`, `Profile URL`). Drop the post author, the author's replies, and obvious creators/influencers.
2. Read each person's headline (the job title line) and sort them against the owner's personas:
   - **Likely ICP:** the headline shows a persona title AND no Not-ICP signal.
   - **Maybe:** one level below a persona title, or a vague senior title with no clear company.
   - **Not ICP:** anything in the Not ICP grade above.
3. Fill the slots (default 15, see `setup.md`): all Likely ICP first, then Maybe. Never fill slots with Not ICP people just to reach the number. If there are fewer, enrich only those.
4. Enrich the picks with BlitzAPI (person, then company) and grade them using the real title and company size. Headlines often oversell, so the BlitzAPI result decides, not the headline.

## Thread score

Because the sample is picked from likely ICPs, do not compute density over the sample. Use:

- `confirmed_strong` and `confirmed_partial`: counts among the enriched picks, after BlitzAPI.
- `headline_icp_share = (Likely ICP + Maybe) / all unique commenters`.

Keep the thread if `confirmed_strong >= 3`, OR `headline_icp_share >= 20%` AND at least 1 confirmed strong match.

Rank kept threads by `confirmed_strong x 2 + confirmed_partial`, then by total engagement. When two threads score close, the one above the engagement threshold wins.

## Logging

- Cache enriched people and companies in `enriched_people.json` and `enriched_companies.json` for the run.
- Log the picks per thread in `candidates.json` (name, headline, why picked, BlitzAPI result, final grade) so the choice can be checked later.
- If BlitzAPI returns nothing for someone, grade that one person from their headline and mark it as such.
