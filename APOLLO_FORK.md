# Apollo fork operations

This fork carries fixes that Apollo needs before they ship upstream. It is built into the stdio image of [apolloio/mcp-proxy](https://github.com/apolloio/mcp-proxy) and backs the shared `notion` MCP release. Keep the patch set small and drop each patch once upstream releases it.

## Patches

| Patch | Upstream | Drop when |
|-------|----------|-----------|
| Add `Notion-Version` header parameter to `POST /v1/comments` (`create-a-comment`) | [#328](https://github.com/makenotion/notion-mcp-server/issues/328), [#329](https://github.com/makenotion/notion-mcp-server/pull/329) | #329 is released |

Do not work around missing `Notion-Version` headers with a global `OPENAPI_MCP_HEADERS` entry. A global header overrides the per-operation defaults, so the page-markdown endpoints (pinned to `2026-03-11`) would get the wrong version.

## Branches

`main` mirrors upstream and is never modified. Apollo patches live on `mgarapich/aieng-663-comment-notion-version`, based on upstream `v2.5.2`. This keeps upstream syncs to a fast-forward of `main`.

## Deployment

`build/Dockerfile.stdio` in apolloio/mcp-proxy clones this repository at a pinned commit on the patch branch (`NOTION_MCP_REF`), builds it, and installs the packed tarball globally. To deploy a change, push it to the patch branch, bump `NOTION_MCP_REF` to the new commit in a mcp-proxy PR, then roll out the `notion` release.

## Syncing upstream

```bash
git remote add upstream https://github.com/makenotion/notion-mcp-server.git  # once
git fetch upstream --tags
git switch main && git merge --ff-only upstream/main && git push origin main
git switch -c apollo-<upstream-tag> <upstream-tag>
git cherry-pick <patch commits from the current patch branch>
npm ci && npm test && npm run build
git push -u origin apollo-<upstream-tag>
```

Use a new branch per upstream base instead of rebasing the current one: mcp-proxy pins a commit on it, and rewriting the branch would make that commit unreachable. Skip any patch upstream now includes, check that every operation in `scripts/notion-openapi.json` still declares `Notion-Version` (`notion-version-coverage.test.ts` enforces this), then bump `NOTION_MCP_REF` and update this file.
