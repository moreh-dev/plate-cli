# plate-cli

Command-line client for the **Plate** publishing hub.

## Notes

- `plate` is a single **Python 3** script. `install.sh` downloads it to `~/.local/bin`, verifies it
  against the release `SHA256SUMS`, and puts that dir on your `PATH` (zsh / bash / fish; macOS & Linux).
- The CLI keeps itself up to date automatically.

## Advanced search

`plate search --syntax=v2 'memory OR "cache latency"'` opts into the new search contract.
Without `--syntax=v2`, existing search commands keep their existing meaning. An older server
is explicitly refused for v2 requests instead of displaying results that ignored new conditions.

Repeat `--owner=email` and `--type=html` / `--type=pdf` to include multiple owners or formats;
`--exclude-owner=email` always excludes that owner. Use `--space-id` or `--folder-id` for a
location (folders include descendants), `--since=YYYY-MM-DD` / `--until=YYYY-MM-DD` for
inclusive Asia/Seoul modification dates, and `--title-only` to search titles only.
`--phrase`, `--exclude`, `--sort`, and `--period=today|7|30|90` are also available in v2.

Continuation output contains a complete `--cursor` command with the same filters and frozen
dates. If the results or access change, run the first-page query again. `--json` still prints
only the hits array; partial coverage and continuation notices go to stderr.
