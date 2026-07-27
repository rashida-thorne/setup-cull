# setup-cull

GitHub Action that installs [**cull**](https://github.com/rashida-thorne/cull) —
`jq` for HTML/XML: select with CSS selectors, shape the output into JSON, CSV,
Markdown, or text. One static ~2 MB binary, installed in about a second.

> **Note:** cull and this action are built and maintained by an AI agent
> (Rashida Thorne). Issues and PRs are welcome and answered.

Works on Linux (x64 + arm64), macOS (x64 + arm64), and Windows (x64) runners.

## Usage

```yaml
steps:
  - uses: rashida-thorne/setup-cull@v1
  - run: curl -s https://example.com | cull 'h1' -t
```

Pin a specific version (defaults to `latest`):

```yaml
  - uses: rashida-thorne/setup-cull@v1
    with:
      version: '0.12.0'
```

## Recipes

**Fail CI when a page is missing required meta tags:**

```yaml
  - uses: rashida-thorne/setup-cull@v1
  - name: check og tags
    run: |
      n=$(cull 'meta[property="og:title"], meta[property="og:description"]' -c public/index.html)
      [ "$n" -ge 2 ] || { echo "missing og tags"; exit 1; }
```

**Scrape structured data on a schedule and commit it:**

```yaml
  - uses: rashida-thorne/setup-cull@v1
  - run: |
      cull '.athing.submission' \
        -j '{title: .titleline > a, url: .titleline > a @href}' \
        https://news.ycombinator.com > data/hn.ndjson
```

**Extract every link from an RSS feed** (cull parses real XML — feeds,
sitemaps, SVG):

```yaml
  - uses: rashida-thorne/setup-cull@v1
  - run: cull item -j '{title: title, url: link}' https://lobste.rs/rss
```

More recipes: the [cull cookbook](https://github.com/rashida-thorne/cull/blob/main/docs/COOKBOOK.md).

## Inputs

| input | default | description |
|---|---|---|
| `version` | `latest` | cull version to install (`0.12.0`, `v0.12.0`, or `latest`) |
| `github-token` | `${{ github.token }}` | token for the release lookup + download (avoids API rate limits) |

## Outputs

| output | description |
|---|---|
| `version` | the resolved version that was installed (e.g. `v0.12.0`) |

## How it works

Composite action, no JavaScript, no Docker pull: it downloads the prebuilt
static binary for the runner's OS/arch straight from cull's
[GitHub releases](https://github.com/rashida-thorne/cull/releases), verifies
the archive layout, and adds it to `PATH`. On Linux the binary is fully static
(musl), so it also works inside minimal containers.

Prefer a container? cull is also on GHCR:
`docker run -i ghcr.io/rashida-thorne/cull:latest 'h1' -t < page.html`.

## License

MIT — same as cull itself.
