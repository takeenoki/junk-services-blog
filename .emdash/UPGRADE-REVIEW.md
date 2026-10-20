# EmDash 1.2.0 → 1.3.1 review

Reviewed 2026-10-20 (Asia/Tokyo), using the refreshed `upgrading-emdash` skill and all 73 entries in `UPGRADE.md`.

## Scope and source changes

- Both installed direct packages, `emdash` and `@emdash-cms/cloudflare`, are 1.3.1 (`npm ls --depth=0`, WSL Node 24.19.0).
- Existing updater changes to package files and skills were preserved. No application source/configuration changes are required by these releases.
- The site uses Cloudflare Workers, D1 (`DB`, session `auto`), R2 (`MEDIA`), standard EmDash UI and content helpers, and the standard scheduled handler.
- No new features, seed changes, content edits, migrations or deployments were performed.

## Release assessment

Numbers refer to `UPGRADE.md`; each entry is included below.

| Entries | Assessment |
| --- | --- |
| 1 | Config descriptor import fix is included in the dependency update; Wrangler remains installed. |
| 2, 3, 4, 13, 14, 18 | No project sandbox/Playground/external-auth/custom-provider/Node S3/custom plugin-index configuration requires adjustment. No numeric role mapping is configured. |
| 5, 15, 31, 59 | Migration behavior/diagnostics improved. No new core migrations in this upgrade; database status must still be checked for earlier pending migrations. |
| 6, 10, 16, 19, 21, 22, 24, 26, 27, 28, 34, 35, 36, 37, 39, 40, 41, 42, 43, 44, 46, 47, 48, 49, 50, 51, 56, 57, 60, 61, 62, 63, 64, 73 | Built-in runtime, admin, CLI, search, menu, comments, seed, editing and maintenance fixes are supplied by the updated packages; no application API changes required. Existing link flags/data were not rewritten. |
| 7, 8 | CLI URL and refresh-header fixes are automatic; no project client/interceptor calls need changing. |
| 9 | D1 schema-write fix is automatic; no schema creation work is needed for the upgrade. |
| 11, 25 | Built-in authorization fixes are automatic; no project authorization overrides found. |
| 12, 23 | Existing `emdash/ui` Image usage receives rendering/type fixes automatically. |
| 17 | Project locales are simply `ja`; no custom path/code aliases need adjustment. |
| 20 | No template reads legacy `meta.caption`. No content was reselected or republished to backfill captions. Generated `emdash-env.d.ts` is not hand-edited; refresh from the running upgraded site when schema types are next generated. |
| 29, 30, 32, 33 | Built-in editor/SEO features require no site changes. AI-training settings left unchanged; no custom ContentEditor restore callback exists. |
| 38 | D1 session is already `auto`. Experimental `coalesce` is optional; not adopted as part of this upgrade. |
| 45, 53, 54, 55, 58 | No custom plugin storage pagination, scoped MCP calls, in-process adapter, private route-metadata calls or registry gallery implementation to update. |
| 52, 67, 68, 69, 70, 71, 72 | WordPress import fixes do not require site code changes. No importer was run and no previously imported data was deleted or rewritten. |
| 65 | Installed adapter and core versions match (1.3.1). |
| 66 | Project queries use ordinary taxonomy filters and published_at ordering; no unsupported `{ in: ... }` filter found. |

## Verification and database

- `npm run typecheck`: passed (18 files; 0 errors, 0 warnings, 0 hints), using WSL Node 24.19.0.
- `npm run build`: passed, server output, EmDash 1.3.1. Vite emitted a chunk-size warning for minified chunks over 500 kB; no unrelated code-splitting refactor was introduced.
- No formatter or test script is defined; the relevant project commands are the two above.
- Initial Windows commands could not resolve the Linux-installed Astro executable. Initial WSL validation stalled with a WSL connection timeout; the user approved restarting Ubuntu, after which validation was rerun sequentially. No source workaround or dependency reinstall was needed.

Confirmed remote D1: `junk-services-blog-site`, ID `09162a01-372f-4210-8450-08723aad435b`, binding `DB`, account `46a4414ab9070e1e4126ba3b136ab91c`, top-level environment.

`emdash migrate --status --account-id 46a4414ab9070e1e4126ba3b136ab91c --wrangler-config wrangler.jsonc` exited 0. Target fingerprint: `3af409a6a038c66bfb703721fb8649dce8190da895aa14978e3aa7159ac5bf95`. Applied through `091_redirect_artifacts`; pending: none; unknown applied: none; executed: none. The user explicitly authorized passing the existing Wrangler OAuth credential in memory for this read-only check. The temporary helper was removed; no credential was printed or saved to the project.

The generated manifest contains migrations through `091_redirect_artifacts`; the updater reports no additions versus 1.2.0. The remote status check independently confirms that the target database is up to date. No upgrade-specific recovery point or migration application is needed for this release when that state remains unchanged.

Do not start/deploy the upgraded application if pending migrations are reported. Before any authorized database-changing action, create a restorable database backup, a separate media backup, preserve encryption keys and the matching application artifact, and obtain approval. Do not use a JSON export as the recovery point.

## Production deployment

The user authorized production deployment on 2026-10-20. `npm exec -- wrangler deploy` succeeded using the existing validated `dist/server/wrangler.json` artifact, without rebuilding.

- URL: https://junk-serives-blog.r32952507.workers.dev
- Worker version: `d7d5ca3e-45f7-435b-80ba-d148bc11f5ed`
- D1/R2 bindings and the every-minute maintenance trigger were deployed successfully.
- Post-deployment `emdash migrate --check` exited 0 against the same target fingerprint: pending none, unknown applied none, executed none.
- GET checks returned HTTP 200 for home, posts list, an existing post detail, search (`q=and`), RSS, admin entry and an existing media file (`image/png`, 44,102 bytes).
- No content edits, republishing, settings changes, deletions or migration applications were performed. Temporary verification files were removed.
- Remaining optional manual verification: authenticated admin sign-in, saving/editing content and encrypted plugin settings, if used. These were not exercised because they require an authenticated session or modify existing data.
