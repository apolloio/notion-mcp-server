# Apollo fork operations

This fork carries fixes that Apollo needs before they ship upstream. It is built into the stdio image of [apolloio/mcp-proxy](https://github.com/apolloio/mcp-proxy) and backs the shared `notion` MCP release. Keep the patch set small and drop each patch once upstream releases it.

## Patches

| Patch | Upstream | Drop when |
|-------|----------|-----------|
| Add `Notion-Version` header parameter to `POST /v1/comments` (`create-a-comment`) | [#328](https://github.com/makenotion/notion-mcp-server/issues/328), [#329](https://github.com/makenotion/notion-mcp-server/pull/329) | #329 is released |

Do not work around missing `Notion-Version` headers with a global `OPENAPI_MCP_HEADERS` entry. A global header overrides the per-operation defaults, so the page-markdown endpoints (pinned to `2026-03-11`) would get the wrong version.

## Deployment

Merging here does not change the running MCP. `build/Dockerfile.stdio` in apolloio/mcp-proxy clones this repository at a pinned commit (`NOTION_MCP_REF`), builds it, and installs the packed tarball globally. To deploy a change, bump `NOTION_MCP_REF` to the merged `main` commit in a mcp-proxy PR, then roll out the `notion` release.

## Syncing upstream

Add the upstream remote once:

```bash
git remote add upstream https://github.com/makenotion/notion-mcp-server.git
```

For each upstream bump, use a short-lived branch and PR. Do not rewrite `main`:

```bash
git fetch upstream --tags
git switch main
git pull --ff-only origin main
git switch -c sync-upstream-YYYYMMDD
git merge --no-ff upstream/main
npm ci && npm test && npm run build
```

Before merging, remove any patch from the table above that upstream now includes, and check that every operation in `scripts/notion-openapi.json` still declares `Notion-Version` (`notion-version-coverage.test.ts` enforces this).
