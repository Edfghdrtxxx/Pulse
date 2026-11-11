## Related issue

None. I searched the issues and pull requests and found nothing on this.

## Problem

Token spend reads only Claude Code's transcripts on this Mac (`~/.claude/projects`). The Claude subscription also pays for chats, Cowork and other Claude products, and those leave no local records. For anyone who uses them, the Claude amount in Token spend is lower than the real total, and nothing on the page says so.

Example: one week where Anthropic's own usage page shows Claude Code 92%, Chats 0%, Cowork 3%, Claude Duet 5%, Other 0%. Token spend shows $1,200 for Claude Code, while all products together come to about $1,304.

## Changes

The reply Pulse already fetches from `api/oauth/usage` includes `seven_day_breakdown.rows`, this week's usage split by product (`key`, `display_name`, `percent` 0–100). Other usage monitors also read this field. This PR:

- **Reads the split** (`ClaudeProductShare`), only for the CLI's own login on the endpoint route, because that is the account whose transcripts Token spend counts. The last split is kept in `UserDefaults`.
- **Adds an "All Claude products" group** under Total, in the combined view and in Claude Code's own view: the Claude Code amount divided by Claude Code's share, one row per product, labelled "Estimate", with a caption giving the measured amount, the share and the day the split was read.
- **Adds a setting**, Settings → Token spend → "Estimate all Claude products", on by default.
- **Reads the field defensively.** It is undocumented. With no split, no Claude Code row, a zero share or no priced Claude Code amount, nothing is shown. Rows without a key or a valid percentage are skipped. Shares are taken against the sum of the rows, because rounded rows can add up to 99 or 101.
- Strings in all six languages (`check-localization.sh` passes), and docs in `token-spend.md`, `testing.md` and `providers/claude-code.md`.

## Proof

- `./Scripts/check-localization.sh`: passes.
- New `ClaudeProductShareTests` with an anonymized fixture: parsing, odd rows, the division, rounding, the stored round trip and the setting's default.
- **Not built or run on macOS.** The change was written in a Linux environment without Swift, so `swift build` and `swift test` have not been run. Please build it and run the tests before merging.

These are **mockups** drawn from the code with the example figures above, **not screenshots of the running app**:

| Before | After |
|---|---|
| ![Before (mockup)](https://github.com/Edfghdrtxxx/Pulse/raw/claude/pr-assets-token-spend/token-spend-before.png) | ![After (mockup)](https://github.com/Edfghdrtxxx/Pulse/raw/claude/pr-assets-token-spend/token-spend-after.png) |

## Limitations

- `seven_day_breakdown` is undocumented and may change.
- The percentages are shares of the weekly limit, not of tokens, so the estimate assumes other products cost about what Claude Code does for the same share.
- The split is always this week's, whatever span is selected.
- Claude Code sessions in the cloud or on another Mac count in Anthropic's share but leave no local transcript, so the estimate runs a little low for people who use them.

## Checklist

- [x] I searched existing issues and pull requests for duplicates.
- [x] The change is focused and contains no unrelated work.
- [x] I updated the relevant documentation.
- [ ] I ran the checks required by `CONTRIBUTING.md`. (Localization check only; build and tests not run, see Proof.)
- [x] I did not claim anything as verified without testing it appropriately.
- [ ] I reviewed and take responsibility for any AI-assisted code or text.

🤖 Generated with [Claude Code](https://claude.com/claude-code)
