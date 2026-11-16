# plate-cli

Command-line client for the **Plate** publishing hub.

## Notes

- `plate` is a single **Python 3** script. `install.sh` downloads it to `~/.local/bin`, verifies it
  against the release `SHA256SUMS`, and puts that dir on your `PATH` (zsh / bash / fish; macOS & Linux).
- The CLI keeps itself up to date automatically.

## Search

**`plate search` uses search v2 by default.** Earlier versions of this CLI needed `--syntax=v2`
for it and otherwise kept the legacy meaning; that is no longer so. What changed for a command
without `--syntax`:

- The query is search syntax: space-separated terms must all match, `memory OR "cache latency"`
  matches either side, a quoted phrase stays together, `-draft` excludes a term, `( )` groups, and
  `author:NAME` / `folder:NAME` narrow by owner or folder. Text that was literal before can now mean
  something else or be refused (an unclosed `"` or `(`). For literal text put `\` before a `"`,
  `(`, `)` or a leading `-` (`\(draft\)`, `\-O3`); write the words OR/AND in lower case or quote them
  (`"OR"`) — `\OR` searches for a backslash; or search the whole text as a phrase:
  `plate search '' --phrase '<text>'`. A syntax error exits 1; the message says how to write it.
- Pages continue with the printed `next page: plate search --syntax=v2 … --cursor=…` command (filters
  kept, dates frozen), not `--offset`. If the results or access change, run the first-page command again.
  The `showing K of N matches` line is printed on every page; the list is complete when there is no
  `next page:` line, the count does not say `at least`, and no `body coverage: pending=…` line appeared.
- `--json` still prints only the hits array, and every key a hit had is still there (`format`,
  `bodyStatus` and `page`/`pageId` are added). The order, `score` and `snippet` follow v2's ranking, and
  `subPath` is empty when the title, folder and owner alone satisfy the query. The count, coverage and
  next-page lines go to stderr; neither mode prints the server's `total`/`truncated`/`complete`.

Legacy search keeps the old meaning: `--syntax=v1`, or any of `--author`, `--folder`, `--offset`
(which only legacy search has). With those options and no `--syntax` the command prints exactly what
it printed before, plus one `plate:` note on stderr; its printed `--offset` continuation keeps working.
Options of both kinds together (`--author` with `--owner`, `--offset` with `--cursor`) are refused.

Repeat `--owner=email` and `--type=html` / `--type=pdf` to include multiple owners or formats;
`--exclude-owner=email` always excludes that owner. Use `--space-id` or `--folder-id` for a
location (folders include descendants), `--since=YYYY-MM-DD` / `--until=YYYY-MM-DD` for
inclusive Asia/Seoul modification dates, and `--title-only` to search titles only.
`--phrase`, `--exclude`, `--sort`, and `--period=today|7|30|90` are also available in v2.

A server without search v2 is refused, not silently searched with v1: the command makes one
search request, prints nothing on stdout, and says to add `--syntax=v1` (unless a v2-only option was
given). The once-a-day update check can add one request for the CLI itself before that. Exit status: 0 when the search ran
(also with no or partial results), 1 when it was refused or failed, 2 for an option or value the CLI refuses.

An installed CLI gets this version on its daily automatic check, on `plate self-update`, or when it
first sees `--syntax=v1` (it tries one update before calling the option unknown). Until then it keeps
the old default, and `--syntax=v1` exits 2. The automatic paths are skipped in CI (unless
`PLATE_SELF_UPDATE=1`), with `PLATE_NO_SELF_UPDATE=1`, without saved credentials, when the server
cannot be reached, or when the installed file cannot be replaced; run `plate self-update` (it falls
back to this repository's release when the credential no longer works) or reinstall.

## Google Sheets connections

Sheets connections can appear in ordinary listings and searches. Search covers their Plate title and
description, not spreadsheet cells. `get` and `fetch` use the existing read permissions; fetched
connection data contains a link configuration, not a copy or backup of the Google spreadsheet.

Create and edit connections in the Plate web UI. CLI rename, source editing, adding a version and
restoring an old content version cannot change a connection. Trash recovery and existing lifecycle
commands retain their normal permissions. This CLI does not add a `--type sheets` filter.
