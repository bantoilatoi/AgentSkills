# Usage examples

Human output is the default in a terminal; `--json` gives the machine envelope. Agents should
always pass `--json`. Command and flag reference: `skills/fab/references/commands.md`.

## Search

```bash
bantoilatoi-fab search "medieval village"
bantoilatoi-fab search "medieval village" --engine unreal --count 10 --json
bantoilatoi-fab search "stylized trees" --style stylized --listing-type 3d-model --sort rating
bantoilatoi-fab search "castle" --hydrate 5 --engine-version 5.4   # verify versions on the top 5
```

## Owned assets

```bash
bantoilatoi-fab library                      # page 1: title, link, engines, download command
bantoilatoi-fab library --page 2             # next page (100 per page; --limit up to 500)
bantoilatoi-fab library "cathedral"          # search titles, descriptions, tags and ids
bantoilatoi-fab library aa08c781             # by listing id, or its first segment
bantoilatoi-fab library --details            # fetch each entry's description for this page
bantoilatoi-fab library "castle" --engine-version 5.4 --json
bantoilatoi-fab search "castle" --owned-only
bantoilatoi-fab ownership 1a2b3c4d-… 5e6f7a8b-… --json
```

Fab's library listing repeats the title in place of a description, so real descriptions come
from each listing's page: automatically when 10 or fewer entries match, for a whole page with
`--details`. Each page ends with the command for the next one.

## Free assets

```bash
bantoilatoi-fab search "footsteps" --free permanent        # always free
bantoilatoi-fab search "footsteps" --free limited-time     # paid, currently 100% off
bantoilatoi-fab search "footsteps" --free either           # both (two searches)
```

## Find and recommend

```bash
bantoilatoi-fab find "dark gothic ruined castle environment" \
  --engine unreal --engine-version 5.4 --prefer-owned --prefer-free --json

bantoilatoi-fab recommend "rigged low-poly zombie with walk and attack animations" \
  --engine unity --feature rigged --max-price 30 --require-engine --json
```

## Inspect

```bash
bantoilatoi-fab inspect 1a2b3c4d-…
```

Listings show whether the account owns them, the price only when it does not, and the licences
they are offered under. FabCLI does not report licences, so they are established from Fab's
licence search filter, run alongside the page's own search, plus one to three searches for each
listing the page left open. Verdicts are cached for a week, so each listing is looked up once;
one run sends at most 200 licence searches and none after a rate limit, and a warning counts the
listings whose licence stayed unknown or is one the filter does not name.

## Download

```bash
bantoilatoi-fab download 1a2b3c4d-…                                  # newest version, this machine's platform
bantoilatoi-fab download 1a2b3c4d-… --out Content/Fab/Castle --dry-run --json
bantoilatoi-fab download 1a2b3c4d-… --out Content/Fab/Castle --engine-version 5.4 --json
```

When an owned asset ships several engine versions or platforms and none is given, `download` takes
the configured engine version if the asset ships it, else the newest, and this machine's platform
(Windows on Linux, where Fab publishes no Linux builds for content). A warning names each choice.

The response's `data.outputDir` is the absolute path written; `data.sidecar` points at
`bantoilatoi-fab.asset.json` inside it.

## Claim (approval required)

```bash
bantoilatoi-fab claim 1a2b3c4d-… --dry-run --json   # show the plan to the user
bantoilatoi-fab claim 1a2b3c4d-… --json             # exit 8, FAB_APPROVAL_REQUIRED
bantoilatoi-fab claim 1a2b3c4d-… --approve --json   # after the user agrees
```

## JSON in scripts

```bash
id=$(bantoilatoi-fab find "ruined castle" --engine unreal --json | jq -r '.data.candidates[0].asset.id')
bantoilatoi-fab download "$id" --out "Content/Fab/$id" --json | jq '.data.receipt'
```

Branch on the exit code first, then `error.code`: `skills/fab/references/errors.md`.

## Per-project defaults

```bash
bantoilatoi-fab config init --project     # writes .bantoilatoi-fab.toml
```

```toml
[defaults]
engine = "unreal"
engine-version = "5.4"

[download]
directory = "Content/Fab"
```

With that file, `bantoilatoi-fab find "castle"` already targets Unreal 5.4 and downloads land in
`Content/Fab/<listing id>`.
