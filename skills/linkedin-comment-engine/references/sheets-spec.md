# Sheets the skill needs (build them if they don't exist)

The skill uses two Google Sheets. If the owner doesn't have them yet, the agent creates them in Setup mode with exactly this layout. Tab names and headers must match, because the skill reads them by name.

## Sources sheet: "LinkedIn Comment Engine - Sources"

| Tab | Header row | Columns |
|---|---|---|
| `Fetch Their Latest Comments` | 1 | A `LinkedIn Profile URL`, B `Name` (optional), C `Notes` (optional) |
| `ICP To Engage With` | 1 | A `Full Name`, B `LinkedIn Profile URL`, C `Company`, D `Job Title`, E `Notes` (optional) |
| `Priority Engagement List` | 1 | A `Rank`, B `Name`, C `Company`, D `LinkedIn Profile URL`, E `Notes` (optional) |
| `Weekly watchlist` | 1 | Same columns as `Priority Engagement List` |

What goes in each tab:
- **Fetch Their Latest Comments:** 5-15 creators in the owner's space whose comments show where the conversation is happening. The skill follows the posts they comment on.
- **ICP To Engage With:** real people who match the owner's ICP and post on LinkedIn. The skill comments on their posts directly. Start with 20+.
- **Priority Engagement List:** creators whose own posts attract the owner's ICP, ranked 1 = best.
- **Weekly watchlist:** more creators, used when the day needs more posts.

The agent can suggest people for these tabs, but only from the owner's ICP input and the owner's confirmation. Never fill them with guesses.

## Output sheet: "LinkedIn Comment Engine - Output"

| Tab | Header row | Columns |
|---|---|---|
| `Drafts` | 1 | A `Date MM/DD/YYYY`, B `LinkedIn Post`, C `Comment`, D `Approval Stage`, E `Edit Notes for Claude` |
| `Comments we left on posts` | 1 | A `LinkedIn Post`, B `Comment` |

- Column D in `Drafts` is a dropdown with exactly these values: `Draft/Pending Approval`, `Approved`, `Modify`, `Declined/Drop`, `Scheduled`.
- Freeze row 1 in every tab. Wrap text in `Drafts` column C.
- `Comments we left on posts` starts empty, or with the owner's own past comments if they want to speed up voice learning.

## Setup mode (first run only)

Trigger: "set up the LinkedIn comment engine", or any run where `setup.md` has no sheet IDs.

1. Ask the owner where their ICP and expertise inputs are (see `your-inputs.md`). Do not continue without them.
2. Create both sheets with the Google Sheets or Google Drive connector, using the layout above. Add the dropdown with data validation on `Drafts!D2:D`.
3. Write the new sheet IDs, the `Drafts` tab link (with its `gid`) and the input locations into `setup.md`. If the skill files can't be edited (for example an installed skill), give the owner the values to paste in.
4. Ask the owner for the Slack channel, and check every connector listed in `tools-and-sources.md` with one cheap call each (for example `snitchfeed_get_usage`, `account_key_info`, Taplio `get_me`).
5. Tell the owner what to fill in next: creators and ICP people in the Sources sheet.
